---
tags:
  - Kubernetes
---

# Kubernetes, briques internes

Nous allons voir au travers de minikube les différentes briques internes importantes, sachant que nous sommes sous Minikube, certaines briques sont différentes d'un Cluster K8S classique.

Vous trouverez la documentation officielle sur les briques ici : [Composants de Kubernetes](https://kubernetes.io/fr/docs/concepts/overview/components/)

Voici un schéma de principe des briques internes de Kube :

![Kubernetes Components](https://kubernetes.io/images/docs/components-of-kubernetes.svg)


## Espace de nom kube-system

Un namespace permet de scinder des élements à l'intérieur du Cluster, exemple avec le namespace **kube-system**, essentiel à tout cluster Kube.

Sur notre environnement, voici ce qu'il contient : 

```bash
k get pods -n kube-system

NAME                               READY   STATUS    RESTARTS       AGE
coredns-7db6d8ff4d-5jv58           1/1     Running   4 (11m ago)    30d
coredns-7db6d8ff4d-mrggh           1/1     Running   4 (11m ago)    30d
etcd-minikube                      1/1     Running   4 (11m ago)    30d
kindnet-4nbpw                      1/1     Running   4 (11m ago)    30d
kube-apiserver-minikube            1/1     Running   10 (10m ago)   30d
kube-controller-manager-minikube   1/1     Running   10 (10m ago)   30d
kube-proxy-lk8gl                   1/1     Running   4 (11m ago)    30d
kube-scheduler-minikube            1/1     Running   4 (11m ago)    30d
kube-vip-minikube                  1/1     Running   3 (11m ago)    28d
metrics-server-c59844bb4-9fxrl     1/1     Running   0              8m9s
storage-provisioner                1/1     Running   6 (11m ago)    30d
```

## Composants internes essentiels

### CoreDNS

CoreDNS est le composant qui gère en interne toute la partie résolution de nom au sein du Cluster Kube.

Pour connaitre la configuration initiale de CoreDNS, il suffit de regarder la ConfigMap associée :

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        hosts {
           192.168.59.1 host.minikube.internal
           fallthrough
        }
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2024-06-05T20:56:33Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "320"
  uid: 6e061255-abd1-4d91-a86b-bac04bf0fe29
```

CoreDNS est puissant par sa flexibilité lié aux plugins.

### etcd

Etcd est une base interne NoSQL qui permet de gérer des clés/valeurs. C'est cette base qui va servir au Cluster pour stocker les élements du Cluster Kubernetes.

En autre, c'est un composant critique d'un Cluster.

### Kindnet

Kindnet est utilisé principalement dans Minikube pour la gestion du réseau auprès des pods, c'est le pilote CNI (Container Network Interface).

Ce composant est donc lié à minikube, il n'est pas utilisé lors de la création d'un cluster K8S.

### APIServer

Le role du serveu d'API est d'accueillir toutes les requêtes au sein d'un Cluster Kube (ou minikube dans notre cas), notament lors de la création des différents objets tels que les namespaces, pods, déploiements, le stockage etc...

Dans notre cas, en mode **SingleNode**, le pod n'est démarré qu'une seule fois sur le noeud labélisé **master**/**control-plane**. Plus le cluster contient ce type de noeud, plus il y aura un scaling horizontal de ce type de pod afin d'élargir la résilience.

### Kube Proxy

Le proxy kube est un pod qui est démarré sur chacun des noeuds d'un Cluster, il permet de faire la liaison réseau au niveau des services kubernetes (svc)

Dans les grandes lignes :

- Redirection du trafic
- Equilibrage de charge
- Support des protocoles standards, TCP/UDP/SCTP
- Mise à jour dynamique (en terme de modification des svc)

Plusieurs modes de fonctionnement :

1. Userspace Mode : Redirige le trafic en passant par l'espace utilisateur. Ce mode est moins performant et est rarement utilisé.
2. iptables Mode : Utilise iptables pour rediriger le trafic au niveau du noyau Linux. C'est le mode le plus commun et offre de bonnes performances.
3. IPVS Mode : Utilise IP Virtual Server (IPVS) pour des performances encore meilleures que celles d'iptables, particulièrement adapté pour des environnements à grande échelle.

### Scheduler

Cette brique permet de définir où les pods seront positionnés en fonction de la configuration demandé par l'utilisateur au sein d'un déploiement, replicatset ou bien d'un daemon-set. 

### Controller Manager

C'est le composant qui permet d'établir un suivi permanent des composants du système.

### Kubelet

Kubelet est un agent démarré sur chaqune machine d'un Cluster qui permet de gérer certain composant Kubernetes, ils peuvent être sous le format d'un service systemd (daemon) ou en conteneur classique.

Lors du démarrage d'un noeud, Kubelet peut démarrer des pods grace à des fichiers de configuration yaml se trouvant sur le noeud.

Exemple avec minikube :
```bash
minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cd /etc/kubernetes/manifests/
$ ls -l
total 16
-rw------- 1 root root 2506 Jul  6 17:33 etcd.yaml
-rw------- 1 root root 3665 Jul  6 17:33 kube-apiserver.yaml
-rw------- 1 root root 2974 Jul  6 17:33 kube-controller-manager.yaml
-rw------- 1 root root 1464 Jul  6 17:33 kube-scheduler.yaml
```

Pour finir, Kubelet ne gère que les pods créés par Kubernetes, en autre, ceux utiles au Cluster.