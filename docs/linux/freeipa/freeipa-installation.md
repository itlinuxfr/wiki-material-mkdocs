---
tags:
  - Linux
  - FreeIPA
  - Enterprise Linux 8
---

# Objectif

Le principe ici sera de montrer comment installer un serveur et un réplica de FreeIPA.

## Topologie FreeIPA

Pourquoi deux serveurs ? L'idée est d'offrir une résilence et une tolérance de panne en cas de crash d'un service FreeIPA (DNS, LDAP, PKI ou autre) ou bien d'un serveur.

Le principe est simple, le serveur primaire est installé en premier, il est maitre de tous les services, une fois terminé, le secondaire est installé à son tour et rejoint la topologie avec synchronisation en temps réel des objets LDAP.

## Pré-requis

Il faut se munir de deux serveurs avec un minimum de 4G de mémoire et 2 CPUs.
Si vous souhaitez utiliser le mode de cryptographie le plus élévé afin de sécurisé les serveurs, il faut le faire en avance de phase.

Mes serveurs sont configurés pour fonctionner avec FIPS. Cela ne peut être changé à chaud une fois les serveurs installés.

Si vous utilisez des DNS externes pour résoudre les noms de vos VM, alors pas de soucis, sinon ajouter les IP + FQDN dans le fichier **/etc/hosts** de chaque VM.

## Installation via Ansible

Depuis plusieurs version déjà, nous avons la possibilité de procéder à l'installation au travers de playbooks Ansible écrit par RedHat et la communauté afin de gérer plus facilement les différents composants.

Il suffira d'avoir un serveur Ansible ou bien le binaire Ansible sur votre poste local et de cloner le dépôt Git qui se trouve ici :

- [Ansible FreeIPA](https://github.com/freeipa/ansible-freeipa.git)

### Inventaire

Une fois le dépôt cloné, l'inventaire se trouve dans **inventaire/**. Dans notre contexte, nous allons utiliser le fichier **host.cluster**.

Voici un exemple de configuration pour une topologie à deux noeuds :

```bash
[ipaserver]
lnx0001.lab.it-linux.priv

[ipaserver:vars]
ipaserver_setup_ca=yes
ipaserver_setup_adtrust=yes
ipaserver_setup_kra=yes
ipaserver_setup_dns=yes
ipaserver_no_hbac_allow=yes
ipaserver_no_pkinit=no
ipaserver_no_ui_redirect=no
ipaserver_mem_check=no
ipaserver_random_serial_numbers=false
ipaserver_no_dnssec_validation=yes
ipaserver_enable_compat=yes
ipaserver_no_forwarders=yes

[ipareplicas]
lnx0002.lab.it-linux.priv

[ipareplicas:vars]
ipaclient_servers=lnx0001.lab.it-linux.priv
ipaclient_force_join=yes
ipareplica_setup_adtrust=yes
ipareplica_setup_ca=yes
ipareplica_setup_kra=yes
ipareplica_setup_dns=yes
ipareplica_no_dnssec_validation=yes
ipareplica_enable_compat=yes
ipareplica_no_forwarders=yes

[ipaclients]

[ipaclients:vars]
ipaclient_allow_repair=yes


[ipa:children]
ipaserver
ipareplicas
ipaclients

[ipa:vars]
ipaadmin_password=un_autre_mot_de_passe_complexe
ipadm_password=un_super_mot_de_passe_complexe
ipaserver_domain=lab.it-linux.priv
ipaserver_realm=LAB.IT-LINUX.PRIV
```

Explication des variables pour le serveur primaire **ipaservers**:

| Variable | Description | Valeur |
| :---        |    :----:   |          ---: |
| ipaserver_setup_ca | Utilisation de la PKI Interne | yes |
| ipaserver_setup_adtrust | Module d'intégration à l'AD | yes |
| ipaserver_setup_kra | Module Vault (Secret) | yes |
| ipaserver_setup_dns | Utilisation de la brique DNS avec Named | yes |
| ipaserver_no_hbac_allow | Ne pas utiliser la règle ALLOW_ALL par défaut | yes |
| ipaserver_no_pkinit | Utilisation de la PKI Interne | yes |
| ipaserver_no_ui_redirect | Direction Apache sur le nom FQDN | no |
| ipaserver_mem_check | Vérification du minimum requis en mémoire | no |
| ipaserver_no_dnssec_validation | Activer/Désactiver la vérification DNSSEC | yes |
| ipaserver_enable_compat | X | yes |
| ipaserver_no_forwarders | Configurer des DNS Globaux | yes |

On retrouvera presque les mêmes variables pour le **ipareplica**.

Dans un scénario de tolérance de panne et de haute disponibilité, l'idéal est de configurer les mêmes services entre le primaire et le secondaire. 

Dans certain contexte, un réplica peut être configuré uniquement pour certain service. Par exemple si on souhaite uniquement avec un réplica avec les services de base et le DNS, c'est possible.

L'option **hidden** permet aussi de configurer un serveur en mode caché, techniquement injoignable par les clients Linux de façon traditionnel avec SSSD.

Il restera les variables en bas de fichier pour définir les mots de passe et le nom du domaine LDAP et Kerberos.

### Lancement de l'installation

Une fois l'inventaire complété, les clés SSH configurées et les serveurs prêts, il suffit de passer cette commande :

```bash
ansible-playbook -i inventory/hosts.cluster install-cluster.yml
```

L'installation dure environ 20Minutes pour les deux serveurs. A ce stade, vous ne devriez pas rencontrer de problème particulier.

### Vérification de l'installation

En ligne de commande :

```bash
# ipactl status

Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
pki-tomcatd Service: RUNNING
smb Service: RUNNING
winbind Service: RUNNING
ipa-otpd Service: RUNNING
ipa-ods-exporter Service: STOPPED
ods-enforcerd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
ipa: INFO: The ipactl command was successful
```

Connexion via Kerberos :

```bash
# kinit admin
Password for admin@LAB.IT-LINUX.PRIV: 

# klist
Ticket cache: KCM:0
Default principal: admin@LAB.IT-LINUX.PRIV

Valid starting       Expires              Service principal
08/15/2024 19:01:54  08/16/2024 18:20:28  krbtgt/LAB.IT-LINUX.PRIV@LAB.IT-LINUX.PRIV
```

On peut afficher la configuration de FreeIPA avec les commandes :

```bash
# ipa config-show

  Maximum username length: 32
  Maximum hostname length: 64
  Home directory base: /home
  Default shell: /bin/sh
  Default users group: ipausers
  Default e-mail domain: lab.it-linux.priv
  Search time limit: 2
  Search size limit: 100
  User search fields: uid,givenname,sn,telephonenumber,ou,title
  Group search fields: cn,description
  Enable migration mode: False
  Certificate Subject base: O=LAB.IT-LINUX.PRIV
  Password Expiration Notification (days): 4
  Password plugin features: AllowNThash, KDC:Disable Last Success
  SELinux user map order: guest_u:s0$xguest_u:s0$user_u:s0$staff_u:s0-s0:c0.c1023$sysadm_u:s0-s0:c0.c1023$unconfined_u:s0-s0:c0.c1023
  Default SELinux user: unconfined_u:s0-s0:c0.c1023
  Default PAC types: MS-PAC, nfs:NONE
  IPA masters: lnx0001.lab.it-linux.priv, lnx0002.lab.it-linux.priv
  IPA master capable of PKINIT: lnx0001.lab.it-linux.priv, lnx0002.lab.it-linux.priv
  IPA CA servers: lnx0001.lab.it-linux.priv, lnx0002.lab.it-linux.priv
  IPA CA renewal master: lnx0001.lab.it-linux.priv
  IPA KRA servers: lnx0001.lab.it-linux.priv, lnx0002.lab.it-linux.priv
  IPA DNS servers: lnx0001.lab.it-linux.priv, lnx0002.lab.it-linux.priv
  IPA DNSSec key master: lnx0002.lab.it-linux.priv
```