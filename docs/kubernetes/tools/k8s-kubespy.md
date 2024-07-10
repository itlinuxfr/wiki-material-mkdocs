---
tags:
  - Kubernetes
---

# kubespy | Suivez-vos déploiement 

Source : [kubespy](https://github.com/pulumi/kubespy)

KubeSpy permet de suivre l'activité sur un Cluster Kubernetes lors d'un déploiement ou d'une ressource.

## Installation 

Télécharger le binaire en fonction de votre distribution puis :

```bash
# Installation sous Debian 12

sudo mv ./kubespy /usr/local/bin/

# Activation de l'autocomplétion 

kubespy completion bash > kubespy.bash
sudo mv kubespy.bash /usr/share/bash-completion/completions/

# Ajouter cette ligne dans votre .bashrc :

source /usr/share/bash-completion/completions/

```

## Manipulation

Si on lance kubespy avec l'option **trace** pour suivre l'état d'un déploiement, cela nous donnes :

```bash
kubespy trace deploy wordpress-example
[ADDED apps/v1/Deployment]  wordpress-homelab/wordpress-example
    Rolling out Deployment revision 1
    ✅ Deployment is currently available
    ✅ Rollout successful: new ReplicaSet marked 'available'

ROLLOUT STATUS:
- [Current rollout | Revision 1] [ADDED]  wordpress-homelab/wordpress-example-64d4bdfddc
    ✅ ReplicaSet is available [1 Pods available of a 1 minimum]
       - [Ready] wordpress-example-64d4bdfddc-7vcwt
```