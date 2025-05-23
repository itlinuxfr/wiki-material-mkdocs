---
tags:
  - Kubernetes
  - TrueNAS
  - ZFS
---

# Introduction

Pour comprendre comment sont provisionnés les volumes dans le Cluster Kubernetes sous TrueNAS, il faut comprendre quel est le backend officiel de stockage dans cette solution NAS.

## TrueNAS & OpenZFS

Qu'est-ce que ZFS ?

ZFS est un système de fichiers. C'est aussi un gestionnaire de volumes. Cela signifie qu'il gère le stockage depuis le disque jusqu'au système d'exploitation, remplaçant à la fois les systèmes de fichiers traditionnels comme FAT, NTFS, UFS ou ext4 et les solutions RAID traditionnelles comme les cartes RAID matérielles, le Drive Extender de Windows Home Server, les Storage Spaces de Windows 10, le GEOM de FreeBSD ou le Logical Volume Manager de Linux.

Vous avez peut-être entendu parler d'Oracle ZFS. ZFS a été développé chez Sun Microsystems par Matt Ahrens et Jeff Bonwick. Il a été open-source et porté sur d'autres systèmes d'exploitation. Après avoir acquis Sun, Oracle a pris la décision inhabituelle de fermer le développement interne du système d'exploitation OpenSolaris, y compris ZFS. Le code existant était toujours open-source, donc la plupart de l'équipe ZFS a quitté Oracle et le projet OpenZFS a été créé pour continuer le développement open-source de ZFS.

## Classe de stockage

Dans Kubernetes, on utilise des **StorageClass** ou **Classe de Stockage** en bon français. Cet objet permet de définir les informations nécessaire pour *réclamer* un espace de stockage. Il existe d'autre classe de stockage en fonction des fournisseurs, notamelent **Trident** pour NetAPP ou encore **Portworx** par PureStorage et d'autre encore.

On peut vérifier la classe de stockage par défaut via la commande :

```bash
sudo k3s kubectl get storageclass
```
On doit obtenir ceci :

```bash
NAME                              PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
openebs-zfspv-default (default)   zfs.csi.openebs.io   Delete          Immediate           true                   90d
```

On peut afficher au format **yaml** la classe de stockage pour avoir plus d'infos :

```yaml
apiVersion: v1
items:
- allowVolumeExpansion: true
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    annotations:
      storageclass.kubernetes.io/is-default-class: "true"
    creationTimestamp: "2024-03-31T17:50:06Z"
    name: openebs-zfspv-default
    resourceVersion: "18378374"
    uid: 95b00a4e-2e08-45e4-87b2-33d08e41e724
  parameters:
    fstype: zfs
    poolname: NAS79_VOL_MIR_02/ix-applications/default_volumes
    shared: "yes"
  provisioner: zfs.csi.openebs.io
  reclaimPolicy: Delete
  volumeBindingMode: Immediate
kind: List
metadata:
  resourceVersion: ""
```

Cette classe de stockage définit le pool ZFS configuré dans TrueNAS Scale pour les applications Kubernetes à utiliser, l'API va ensuite créer des volumes dynamiques en fonction des applications et des besoins.

- *storageclass.kubernetes.io/is-default-class: "true"*

 Permet de définir la classe par défaut

- *parameters.poolname: NAS79_VOL_MIR_02/ix-applications/default_volumes*

Permet de définir le pool ZFS pour tous les volumes Kube

## Les secrets

Pour communiquer avec notre TrueNAS et ZFS, il faut que Kubernetes est accès à l'API avec un compte et un mot de passe. 

Il existe un secret dans le namespace **kube-system** :

```bash
admin@scale:~$ sudo k3s kubectl get secret -n kube-system |grep -i truenas
```

```bash
ix-truenas.node-password.k3s   Opaque              1      228d
```

On peut obtenir le mot de passe en regardant de plus près le secret :

```yaml
apiVersion: v1
data:
  hash: "une-très-grande-chaine-de-caractères-en-base64"
immutable: true
kind: Secret
metadata:
  creationTimestamp: "2023-11-14T18:38:52Z"
  name: ix-truenas.node-password.k3s
  namespace: kube-system
  resourceVersion: "237"
  uid: ba5fb692-ffe8-4ff9-a831-23ee73616389
type: Opaque
```

Une envie de connaitre le mot de passe ?

Rien de plus simple : 

```bash
echo "une-très-grande-chaine-de-caractères-en-base64" | base64 -d
```

Et voilà !

## Les Customs Resources Definitions

On peut afficher la liste de CRD pour OpenEBS :

```bash
admin@scale:~$ sudo k3s kubectl get crd -n openebs
NAME                                             CREATED AT
helmcharts.helm.cattle.io                        2023-11-14T18:38:50Z
helmchartconfigs.helm.cattle.io                  2023-11-14T18:38:50Z
addons.k3s.cattle.io                             2023-11-14T18:38:50Z
network-attachment-definitions.k8s.cni.cncf.io   2023-11-14T18:38:54Z
zfsbackups.zfs.openebs.io                        2023-11-14T18:38:56Z
zfsnodes.zfs.openebs.io                          2023-11-14T18:38:56Z
zfsrestores.zfs.openebs.io                       2023-11-14T18:38:56Z
zfssnapshots.zfs.openebs.io                      2023-11-14T18:38:56Z
zfsvolumes.zfs.openebs.io                        2023-11-14T18:38:56Z
volumesnapshotclasses.snapshot.storage.k8s.io    2023-11-14T18:38:54Z
volumesnapshotcontents.snapshot.storage.k8s.io   2023-11-14T18:38:54Z
volumesnapshots.snapshot.storage.k8s.io          2023-11-14T18:38:54Z
```

Si on regarde la ressource **zfsnodes**, on pourra voir notre NAS connecté et ses pools :

```yaml
# sudo k3s kubectl describe zfsnodes -A
Name:         ix-truenas
Namespace:    openebs
Labels:       <none>
Annotations:  <none>
API Version:  zfs.openebs.io/v1
Kind:         ZFSNode
Metadata:
  Creation Timestamp:  2023-11-14T18:42:13Z
  Generation:          309892
  Managed Fields:
    API Version:  zfs.openebs.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:ownerReferences:
          .:
          k:{"uid":"6a44f76f-c285-4e72-a604-47081266cd63"}:
      f:pools:
    Manager:    zfs-driver
    Operation:  Update
    Time:       2024-06-30T16:56:46Z
  Owner References:
    API Version:     v1
    Controller:      true
    Kind:            Node
    Name:            ix-truenas
    UID:             6a44f76f-c285-4e72-a604-47081266cd63
  Resource Version:  30619121
  UID:               22466e9a-cc0c-42c3-a21e-f6088e1737c1
Pools:
  Free:  2292652156Ki
  Name:  NAS79_VOL_MIR_01
  Uuid:  17172874061595888983
  Free:  2709244160Ki
  Name:  NAS79_VOL_MIR_02
  Uuid:  10077988922093906049
  Free:  9382090924512
  Name:  NAS79_VOL_VAULT_01
  Uuid:  16543390058926155067
  Free:  445038332Ki
  Name:  boot-pool
  Uuid:  6920887177742642366
Events:  <none>
```

## Utiliser les volumes

Comment le volume créé dans un pool ZFS est ensuite monté dans un conteneur ? 

Le montage d'un volume type **ix-application** se fait via le mode **hostPath**, **PVC** ou **NFS** en fonction de l'application et de sa Chart.

Quand le volume est créé dynamiquement par la classe de stockage lors de l'installation d'une nouvelle application, le backend zfs créé un nouveau volume et le monte sur le NAS lui même. On peut afficher les volumes déjà montés :

### Exemple avec hostPath

Type : **hostPath**

```bash
# admin@scale:~$ df -h |grep -i "ix-applications" |awk '{print $1}'

NAS79_VOL_MIR_02/ix-applications
NAS79_VOL_MIR_02/ix-applications/default_volumes
NAS79_VOL_MIR_02/ix-applications/k3s
NAS79_VOL_MIR_02/ix-applications/catalogs
NAS79_VOL_MIR_02/ix-applications/releases
NAS79_VOL_MIR_02/ix-applications/k3s/kubelet
NAS79_VOL_MIR_02/ix-applications/releases/prometheus
NAS79_VOL_MIR_02/ix-applications/releases/prometheus/charts
NAS79_VOL_MIR_02/ix-applications/releases/prometheus/volumes
NAS79_VOL_MIR_02/ix-applications/releases/prometheus/volumes/ix_volumes
NAS79_VOL_MIR_02/ix-applications/releases/prometheus/volumes/ix_volumes/data
NAS79_VOL_MIR_02/ix-applications/releases/prometheus/volumes/ix_volumes/config
NAS79_VOL_MIR_02/ix-applications/releases/grafana
NAS79_VOL_MIR_02/ix-applications/releases/grafana/charts
NAS79_VOL_MIR_02/ix-applications/releases/grafana/volumes
NAS79_VOL_MIR_02/ix-applications/releases/grafana/volumes/ix_volumes
NAS79_VOL_MIR_02/ix-applications/releases/grafana/volumes/ix_volumes/data
```

Ensuite une déclartion dans le type de déploiement Kubernetes est ajouté pour monter le volume, exemple avec une application Grafana basique :

```yaml
...
        volumeMounts:
        - mountPath: /mnt/directories/data
          name: data
...
      volumes:
      - hostPath:
          path: /mnt/NAS79_VOL_MIR_02/ix-applications/releases/grafana/volumes/ix_volumes/data
          type: ""
        name: data
      - emptyDir: {}
        name: tmp
...
```

Comme on peut le voir dans cet exemple, on ne passe par la déclaration d'un **PersistentVolumeClaim** mais par **hostPath**. En fonction du type d'application et du type de stockage nécessaire, la méthode de provisionnement de stockage diffère.

### Exemple avec un PVC

Ici, j'ai déployé une application via Chart pour qbittorrent, celle-ci nécessite du stockage pour garder sa configuration persistante au redémarrage.

Voici la composition du déploiement :

```yaml
...
      volumes:
      - name: config
        persistentVolumeClaim:
          claimName: qbittorrent-config
...
```

Ensuite, on peut afficher le PVC associé :

```bash
# sudo k3s kubectl get pvc qbittorrent-config -n ix-qbittorrent

NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS            AGE
qbittorrent-config   Bound    pvc-c58380a0-25d9-4080-a18f-e13b150edd13   5Gi        RWO            openebs-zfspv-default   130m
```

Voici sa composition :

```yaml
#sudo k3s kubectl get pv -n ix-qbittorrent pvc-c58380a0-25d9-4080-a18f-e13b150edd13 --output yaml
...
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: qbittorrent-config
    namespace: ix-qbittorrent
  csi:
    driver: zfs.csi.openebs.io
    fsType: zfs
    volumeAttributes:
      openebs.io/cas-type: localpv-zfs
      openebs.io/poolname: NAS79_VOL_MIR_02/ix-applications/default_volumes
      storage.kubernetes.io/csiProvisionerIdentity: 1719863288029-693-zfs.csi.openebs.io
    volumeHandle: pvc-c58380a0-25d9-4080-a18f-e13b150edd13
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: openebs.io/nodeid
          operator: In
          values:
          - ix-truenas
  persistentVolumeReclaimPolicy: Delete
  storageClassName: openebs-zfspv-default
  volumeMode: Filesystem
...
```
On peut aussi utiliser une ressource (CRD) de OpenEBS pour afficher le volume : 

```bash
# sudo k3s kubectl get zv -n openebs

NAME                                       ZPOOL                                              NODEID       SIZE         STATUS   FILESYSTEM   AGE
pvc-c58380a0-25d9-4080-a18f-e13b150edd13   NAS79_VOL_MIR_02/ix-applications/default_volumes   ix-truenas   5368709120   Ready    zfs          133m
```

```yaml
# sudo k3s kubectl get zv -n openebs pvc-c58380a0-25d9-4080-a18f-e13b150edd13 --output yaml

piVersion: zfs.openebs.io/v1
kind: ZFSVolume
metadata:
  creationTimestamp: "2024-07-02T16:45:58Z"
  finalizers:
  - zfs.openebs.io/finalizer
  generation: 2
  labels:
    kubernetes.io/nodename: ix-truenas
  name: pvc-c58380a0-25d9-4080-a18f-e13b150edd13
  namespace: openebs
  resourceVersion: "30837917"
  uid: 2716d6ac-7607-4dad-b51d-a095bbdf810d
spec:
  capacity: "5368709120"
  fsType: zfs
  ownerNodeID: ix-truenas
  poolName: NAS79_VOL_MIR_02/ix-applications/default_volumes
  shared: "yes"
  volumeType: DATASET
status:
  state: Ready
```
On peut vérifier le montage depuis le pod :

```bash
#sudo k3s kubectl -n ix-qbittorrent exec -it qbittorrent-6dbbbf9787-pl9q6 -- bash

bittorrent-6dbbbf9787-pl9q6:/config$ df -h /config
Filesystem                Size      Used Available Use% Mounted on
NAS79_VOL_MIR_02/ix-applications/default_volumes/pvc-c58380a0-25d9-4080-a18f-e13b150edd13
                          5.0G      5.6M      5.0G   0% /config
```