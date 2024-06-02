---
tags:
  - Kubernetes
  - Minikube
---
# Introduction

Minikube est un outil qui permet de lancer un cluster Kubernetes localement sur votre machine. C'est idéal pour les développeurs qui souhaitent tester ou développer des applications Kubernetes sans avoir besoin de déploiement sur un vrai cluster. Minikube simule un environnement Kubernetes complet sur un PC ou Mac, facilitant l'apprentissage, le développement, et l'expérimentation avec Kubernetes.

# Installation

> Système d'exploitation : **Linux Ubuntu**
{.is-info}

Rien de plus simple pour installation minikube sur son poste :

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```