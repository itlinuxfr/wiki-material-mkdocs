---
tags:
  - Linux
  - Apache
  - Enterprise Linux 8
  - Vagrant
---
# Objectifs :

- Création d’un Cluster sous Linux EL8 ou RockyLinux 8 avec pacemaker sans **Fencing**.
- Création d’une ressource LVM
- Création d’une IP Virtuelle partagée
- Création d’un service web mutualisé

Source : [RedHat Cluster Active-Passive HTTPD EL8 with HA](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_high_availability_clusters/assembly_configuring-active-passive-http-server-in-a-cluster-configuring-and-managing-high-availability-clusters)

## Environement :

Dans le cadre du homelab, nous allons crééer deux VMs via Vagrant avec le fichier suivant :

```terraform
Vagrant.configure("2") do |config|
  config.vm.box = "generic/rocky8"
  
  # Configuration du plugin disksize pour ajuster la taille du disque principal
  config.disksize.size = '40GB'

  config.vm.provider "virtualbox" do |hv|
    hv.cpus = 1
    hv.memory = 512  
  end

  config.vm.define "ha01" do |ha01|
    ha01.vm.network :public_network, ip: "192.168.1.151", bridge: "eno1"
    ha01.vm.hostname = "clusha01.lab.example.priv"
    # Configuration pour ajouter un disque supplémentaire, si nécessaire
    ha01.vm.disk :disk, name: "ha01-nfs", size: "10GB"
  end

  config.vm.define "ha02" do |ha02|
    ha02.vm.network :public_network, ip: "192.168.1.152", bridge: "eno1"
    ha02.vm.hostname = "clusha02.lab.example.priv"
    # Configuration pour ajouter un disque supplémentaire, si nécessaire
    ha02.vm.disk :disk, name: "ha02-nfs", size: "10GB"
  end
end
```

On aura donc deux machines avec une IP fixe et un disque de 10G

| Machine | IP | CPU | Mémoire | OS | Disque LVM |
| --- | --- | --- | --- | --- | --- |
| clusha01.lab.example.com | 192.168.1.151 | 1 | 512 Mb | RockyLinux 8.9 | 10G |
| clusha02.lab.example.com | 192.168.1.152 | 1 | 512 Mb | RockyLinux 8.9 | 10G |

## Prérequis OS

Activation du dépôt High Availability :

```bash
sudo dnf config-manager --set-enabled ha
```

Installation des paquets nécessaires :

```bash
dnf install -y pcs pacemaker
```

Activation des services au démarrage :

```bash
sudo systemctl enable --now pcsd.service
```

Ouverture des ports nécessaires :

```bash
sudo firewall-cmd --permanent --add-service=high-availability
sudo firewall-cmd --reload

```

Configuration de la résolution DNS en local (si besoin) :

```bash
# Ajouter ces deux lignes si vous n'avez pas de DNS sur chacun des deux noeuds :
192.168.1.151 clusha01.lab.example.priv clusha01
192.168.1.152 clusha02.lab.example.priv clusha02
```

## Configuration de Pacemaker

Changement du mot de passe pour le compte **hacluster** :

```bash
passwd hacluster
```

Authentification du compte **hacluster** sur les deux machines :

```bash
pcs host auth clusha01.lab.example.com clusha02.lab.example.com
# Retour de la commande :
Username: hacluster
Password: 
clusha02.lab.example.priv: Authorized
clusha01.lab.example.priv: Authorized
```

Création du Cluster sur les deux machines :

```bash
# Lancer la commande sur une seule des deux machines :
pcs cluster setup mon_cluster --start clusha01.lab.example.priv clusha02.lab.example.priv
# Le système nous retourne ceci :
o addresses specified for host 'clusha01.lab.example.priv', using 'clusha01.lab.example.priv'
No addresses specified for host 'clusha02.lab.example.priv', using 'clusha02.lab.example.priv'
Destroying cluster on hosts: 'clusha01.lab.example.priv', 'clusha02.lab.example.priv'...
clusha01.lab.example.priv: Successfully destroyed cluster
clusha02.lab.example.priv: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'clusha01.lab.example.priv', 'clusha02.lab.example.priv'
clusha01.lab.example.priv: successful removal of the file 'pcsd settings'
clusha02.lab.example.priv: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'clusha01.lab.example.priv', 'clusha02.lab.example.priv'
clusha01.lab.example.priv: successful distribution of the file 'corosync authkey'
clusha01.lab.example.priv: successful distribution of the file 'pacemaker authkey'
clusha02.lab.example.priv: successful distribution of the file 'corosync authkey'
clusha02.lab.example.priv: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'clusha01.lab.example.priv', 'clusha02.lab.example.priv'
clusha01.lab.example.priv: successful distribution of the file 'corosync.conf'
clusha02.lab.example.priv: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
Starting cluster on hosts: 'clusha01.lab.example.priv', 'clusha02.lab.example.priv'...
```

Désactivation de **stonith** qui est utile quand nous avons une 3ème machine dédiée au quorum :

```bash
pcs property set stonith-enabled=falseActivation des services du Cluster au démarrage :
```

```bash
pcs cluster enable --all
```

Cette commande n’est pas obligatoire dans le sens où si l’on souhaite un état différent au redémarage d’un des noeuds, par exemple que les services ne soient pas démarrés.

Afficher l’état actuel de notre Cluster :

```bash
pcs cluster status
# Résultat :
Cluster Status:
 Cluster Summary:
   * Stack: corosync (Pacemaker is running)
   * Current DC: clusha01.lab.example.priv (version 2.1.6-9.el8_9-6fdc9deea29) - partition with quorum
   * Last updated: Sat Feb 10 10:17:58 2024 on clusha01.lab.example.priv
   * Last change:  Sat Feb 10 10:17:56 2024 by root via cibadmin on clusha01.lab.example.priv
   * 2 nodes configured
   * 0 resource instances configured
 Node List:
   * Online: [ clusha01.lab.example.priv clusha02.lab.example.priv ]

PCSD Status:
  clusha01.lab.example.priv: Online
  clusha02.lab.example.priv: Online
```

Création d’une première sauvegarde pour le Cluster via la commande :

```bash
pcs config backup cluster_sauvegarde
```

Un fichier est alors créé à la racine de notre compte actuel (root) :

```bash
-rw-------. 1 root root 1839 Feb 10 10:21 cluster_sauvegarde.tar.bz2
```

## Configuration d’un Cluster Actif/Passif HTTPD

### Création d’un filesystem partagé LVM via pacemaker

Déclaration du **systemid** dans la configuration de **lvm** pour identifier chaque machine :

```bash
# vi /etc/lvm/lvm.conf
# Décommenter et modifier la ligne :
system_id_source = "uname"

# Résultat :
[root@clusha01 ~]# lvm systemid
  system ID: clusha01.lab.example.priv
```

Création du **vg_shared** et désactivation du montage automatique au démarrage de la machine :

(Chaque machine possède un disque de 10G sur /dev/sdb)

```bash
vgcreate --setautoactivation n vg_shared /dev/sdb

# Résultat :
Physical volume "/dev/sdb" successfully created.
  Volume group "vg_shared" successfully created with system ID clusha01.lab.example.priv

```

Création du lv_shared avec la taille total de notre vg_shared :

```bash
lvcreate -n lv_shared -l 100%FREE vg_shared

# Résultat :
Logical volume "lv_shared" created.
```

Création du système de fichier de type xfs sur notre lv_shared :

```bash
mkfs.xfs /dev/vg_shared/lv_shared
```

### Configuration de Apache HTTPD

Installation du paquet :

```bash
dnf install -y httpd wget
```

Ouvertures des portes du parefeu nécessaires :

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --reload
```

Ajout d’une configuration pour **/etc/httpd/conf.d/status.conf** pour récupérer l’état du serveur Apache :

```bash
# Passer la commande :
cat <<-END > /etc/httpd/conf.d/status.conf
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
END
```

Monter le volume **lvm** dans **/var/www** pour la redondance des fichiers Apache, sur une seule machine :

```bash
mount /dev/vg_shared/lv_shared /var/www
mkdir -p /var/www/{html,cgi-bin,error}
restorecon -R /var/www
```

Ajouter une page web très basic au serveur Apache :

```bash
# Passer la commande suivante :
cat <<-END >/var/www/html/index.html
<html>
<body>Hello</body>
</html>
END
```

Enfin, démonter le volume :

```bash
umount /var/www
```

### Création des resources partagés via pacemaker

Création d’une première ressource de type système de fichier partagé pour notre partie Apache, avec le LV lv_shared que nous avons créé précédement :

```bash
# Sur une seule machine, taper les commandes :
pcs resource create lvm_shared ocf:heartbeat:LVM-activate vgname=vg_shared vg_access_mode=system_id --group apachegroup
# Ce qui nous donne :
[root@clusha01 ~]# pcs resource status
  * Resource Group: apachegroup:
    * lvm_shared	(ocf::heartbeat:LVM-activate):	 Started clusha01.lab.example.priv
```

Création des 3 ressources principales,

- Ip Virtuelle
- Système de Fichier
- Apache

```bash
pcs resource create fs_shared Filesystem device="/dev/vg_shared/lv_shared" directory="/var/www" fstype="xfs" --group apachegroup
pcs resource create VirtualIP IPaddr2 ip=192.168.1.249 cidr_netmask=24 --group apachegroup
syst
```

Si nous n’avons pas d’erreur, alors le résutlat doit être le suivant :

```bash
[root@clusha01 ~]# pcs status
Cluster name: mon_cluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: clusha01.lab.example.priv (version 2.1.6-9.el8_9-6fdc9deea29) - partition with quorum
  * Last updated: Sun Feb 11 20:03:21 2024 on clusha01.lab.example.priv
  * Last change:  Sun Feb 11 20:02:31 2024 by root via cibadmin on clusha01.lab.example.priv
  * 2 nodes configured
  * 4 resource instances configured

Node List:
  * Online: [ clusha01.lab.example.priv clusha02.lab.example.priv ]

Full List of Resources:
  * Resource Group: apachegroup:
    * lvm_shared	(ocf::heartbeat:LVM-activate):	 Started clusha01.lab.example.priv
    * fs_shared	(ocf::heartbeat:Filesystem):	 Started clusha01.lab.example.priv
    * VirtualIP	(ocf::heartbeat:IPaddr2):	 Started clusha01.lab.example.priv
    * Website	(ocf::heartbeat:apache):	 Started clusha01.lab.example.priv

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

Notre VIP (Virtual IP) est bien créée :

```bash
[root@clusha01 ~]# ip a |grep eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.1.151/24 brd 192.168.1.255 scope global noprefixroute eth1
    inet 192.168.1.249/24 brd 192.168.1.255 scope global secondary eth1
```

On peut essayer de joindre notre serveur Web avec la commande cURL :

```bash
curl -XGET http://192.168.1.249/index.html

# Retour :
<html>
<body>Hello</body>
</html>
```

Fracture volontaire d’un nœud pour prouver la résilience :

```bash
pcs node standby clusha01.lab.example.priv
# La VIP bascule normalement sur l'autre machine
Full List of Resources:
  * Resource Group: apachegroup:
    * lvm_shared	(ocf::heartbeat:LVM-activate):	 Started clusha02.lab.example.priv
    * fs_shared	(ocf::heartbeat:Filesystem):	 Started clusha02.lab.example.priv
    * VirtualIP	(ocf::heartbeat:IPaddr2):	 Started clusha02.lab.example.priv
    * Website	(ocf::heartbeat:apache):	 Started clusha02.lab.example.priv
```

Si on coupe le serveur distant :

```bash
Node List:
   * Node clusha01.lab.example.priv: OFFLINE (standby)
   * Online: [ clusha02.lab.example.priv ]

PCSD Status:
  clusha02.lab.example.priv: Online
  clusha01.lab.example.priv: Offline
```

Le service est toujours rendu par la machine en vie.