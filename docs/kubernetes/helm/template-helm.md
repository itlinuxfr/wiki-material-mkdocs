---
tags:
  - Kubernetes
  - Minikube
  - Helm
---

# Les templates Helm

Un template Helm est une base  de configuration variabilisé pour intéragir dynamiquement avec des valeurs indiquées par défauts ou par l'utilisateur.

Il faut voir ceci comme le fait de créer un modèle dyniquement pour configurer une application en fonction du client et de son besoin.

L'idée est de faire comme sous Ansible avec les templates Jinja2, variabiliser et conditionner toutes nos configurations YAML.

En passant par :

- Déploimement (ou DaemonSet, ReplicatSetn etc..)
- Stockage (PV/PVC)
- Ingress (Règle réseau)
- Compte de Service (SA)
- Exposition du service (SVC)

Bref, tous les élements nécessaires à une application