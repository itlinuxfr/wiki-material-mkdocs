# Introduction à Helm

Helm a été introduit par la communauté afin de répondre à un besoin de simplification de l'installation des applications dans un cluster Kubernetes. Avant l'arrivée de Helm, les administrateurs devaient gérer le cycle de vie des fichiers de configuration YAML, leur personnalisation ainsi que l'intégration des produits eux mêmes.

Un cluster Kubernetes fonctionne à l'aide de nombreux produits (Elasticsearch, Kafka, Prometheus, Grafana, Nginx, etc.). Les maîtriser tous peut être chronophage.

Les charts Helm sont une aide précieuse pour s'affranchir des problématiques d'installaton initiale.

Sources :

- [Helm](https://helm.sh)
- Livre *Expert Kubernetes* par **Yanning Perré**

## Principe de fonctionnement

En version 3, Helm fonctionne à l'aide d'un client écrit en Go, se dernier d'appuie sur l'API de Kubernetes . Le programme se présente sous forme de binaire indépendant, comme l'est kubectl par exemple. Il ne nécessite pas de dépendance particulière.