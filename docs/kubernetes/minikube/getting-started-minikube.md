---
tags:
  - Kubernetes
  - Minikube
---
# Installation

> Système d'exploitation : **Linux Ubuntu**
{.is-info}

Rien de plus simple pour installation minikube sur son poste :

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

## Premier cluster

Pour mettre en place son premier Cluster via minikube sur sa machine, en fonction des ressources, il suffit de lancer cette commande :

```bash
minikube start --cpus=2 --memory=2024
```

Attention, il faut au minimum 2 Coeurs ainsi que 1800Mo de mémoire.

Une fois quelques minutes passées, notre Cluster est prêt :

```bash
$kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
minikube       Ready    control-plane          11d   v1.30.0
```
## En avant !

L'accès au cluster est automatiquement configuré au travers du fichier kubeconfig se trouvant ici : **$HOME/.kube/config**

Pour plus de rapidité, nous pouvons créer un **alias k=kubectl** afin de réduire la taille de nos commandes.

On peut aussi activer l'autocomplétion pour la commande kubectl :

```bash
source <(kubectl completion bash)
# Ainsi que l'autocomplétion pour notre alias
complete -o default -F __start_kubectl k
```
Vous être maintenant prêt à manipuler Kube !