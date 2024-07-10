---
tags:
  - Kubernetes
---

# Gestion des contextes
## Contexte kube

Au travers du binaire **kubectl**, nous pouvons configurer plusieurs contextes, c'est à dire plusieurs configuration qui décrivent la connexion à un cluster Kubernetes distant.

Pour gérer les contextes, il faut préfixer les commandes par ```kubectl config```.

Par exemple, si l'on souhaite afficher les contextes :

```bash
k config current-context
```

On peut afficher la liste des contextes déjà configurés :

```bash
k config get-contexts

CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default
```

Dans notre, nous sommes sur minikube donc c'est par défaut notre contexte et notre Cluster.

Par défaut c'est le fichier placé sous ```~/.kube/config```qui est pris en compte.

Il existe une variable globale qui permet de gérer les fichiers de configuration à prendre en compte : **KUBECONFIG**

```bash
export KUBECONFIG="/kubernetes/mon-super-cluster"

k config get-contexts

CURRENT   NAME                CLUSTER             AUTHINFO   NAMESPACE
          mon-super-cluster   mon-super-cluster   minikube   default
```

Si l'on souhaite changer de contexte, il suffit de passer par la commande : ```kubectl config use-context <contexte>```.

Création d'un nouveau contexte :

```bash
k config set-context minikube \
> --cluster=minikube \
> --user=minikube \
> --namespace=default
Context "minikube" created.
```

Ensuite, on choisi le nouveau contexte :

```bash
k config use-context minikube
Switched to context "minikube"
```

##  kubectx & kubens

Source : [Github](https://github.com/ahmetb/kubectx)

**kubens** permet de se déplacer au travers des différents namespaces plus facilement. C'est un script écrit en bash qui permet de simplier la gestion des namespaces.

**kubectx** permet de se déplacer au travers de tous les contextes configurés localement. 

Installation :

```bash
sudo apt install kubectx

sudo wget -O /usr/local/bin/kubens https://github.com/ahmetb/kubectx/blob/master/kubens
sudo chmod +x /usr/local/bin/kubens
```

Activation de l'autocomplétion :

```bash
cd /usr/share/bash-completion/completions
sudo vi kubectx.bash

# Insérer le code suivant :
_kube_contexts()
{
  local curr_arg;
  curr_arg=${COMP_WORDS[COMP_CWORD]}
  COMPREPLY=( $(compgen -W "- $(kubectl config get-contexts --output='name')" -- $curr_arg ) );
}

complete -F _kube_contexts kubectx kctx
```

Dans votre .bashrc, il faudra ajouter les lignes suivantes :

```bash
source /usr/share/bash-completion/completions/kubectx.bash
source /usr/share/bash-completion/completions/kubens.bash
```