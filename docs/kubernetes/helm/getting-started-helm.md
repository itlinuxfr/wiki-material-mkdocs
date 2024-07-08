---
tags:
  - Kubernetes
  - Minikube
  - Helm
---

# En Avant ! 
## Installation sous Linux
Pour l'installer sur un système Linux, rien de plus simple :

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Helm s'appuie sur la configuration du contexte actuelle pour la connexion à un cluster Kubernetes.

Cependant, vous pouvez librement définir des variables d'environnements afin de changer de contexte :
```bash
# helm --help
...
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config")                                |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                                                  |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                                             |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.                                      |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                                         |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                                                    |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                                          |
| $HELM_KUBEINSECURE_SKIP_TLS_VERIFY | indicate if the Kubernetes API server's certificate validation should be skipped (insecure)                |
| $HELM_KUBETLS_SERVER_NAME          | set the server name used to validate the Kubernetes API server certificate                                 |
...
```

## Ajout d'une source Helm

Dans la version 3 de Helm, il n'y a plus de source configurée par défaut, ce qui veut dire que nous ne pouvons pas effectuer de recherche pour une chart spécifique.

Nous allons ajouter la source du dépôt [Bitnami](https://charts.bitnami.com/bitnami)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

"bitnami" has been added to your repositories
```

On peut mettre à jour le dépôt via : ```helm repo update```

## Recherche d'une Chart

Maintenant que nous avons un dépôt officiel, nous pouvons rechercher une application :

```bash
helm search repo wordpress

NAME                   	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/wordpress      	22.4.18      	6.5.5      	WordPress is the world's most popular blogging ...
bitnami/wordpress-intel	2.1.31       	6.1.1      	DEPRECATED WordPress for Intel is the most popu...
```

On remarque plusieurs champs :

- ```NAME``` -> correspond au nom du chart
- ```CHART VERSION``` -> la version du chart
- ```APP VERSION``` -> ici, la version officielle de wordpress
- ```DESCRIPTION``` -> une descritpion du logiciel

## Installer une chart Helm

Nous allons installer bêtement la chart Helm pour Wordpress :

```bash
helm install wordpress-example bitnami/wordpress --create-namespace --namespace wordpress-homelab
```

Si tout ce passe bien, la commande nous renvois différentes informations, notament la méthode pour se connecter à wordpress avec les valeurs par défaut.

```bash
NAME: wordpress-example
LAST DEPLOYED: Mon Jul  8 16:55:56 2024
NAMESPACE: wordpress-homelab
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 22.4.18
APP VERSION: 6.5.5

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    wordpress-example.wordpress-homelab.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace wordpress-homelab -w wordpress-example'

   export SERVICE_IP=$(kubectl get svc --namespace wordpress-homelab wordpress-example --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace wordpress-homelab wordpress-example -o jsonpath="{.data.wordpress-password}" | base64 -d)

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```

Par exemple, en utilisant minikube et le chart Wordpress, avec la commande suivante nous pouvons exposé localement notre premier site :

```bash
k port-forward -n wordpress-homelab deployments/wordpress-example 8080:8080
```