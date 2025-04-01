# Introduction

FreeIPA vise à fournir un système d'identité, de politique et d'audit (IPA) géré de manière centralisée3. Il utilise une combinaison de Fedora, 389 Directory Server, MIT Kerberos, NTP, DNS, l'IGC DogTag, SSSD et d'autres composants open-source libres. 

FreeIPA apporte des interfaces de gestion extensibles (CLI, interface utilisateur Web, XML-RPC et l'API JSONRPC), le SDK Python pour l'autorité de certification intégrée, et BIND avec un plug-in personnalisé pour le serveur DNS intégré. 

Chacun des principaux composants de FreeIPA fonctionne comme un projet gratuit / open-source préexistant. Le regroupement de ces composants dans une seule suite avec une interface de gestion complète est sous licence GPLv3, mais cela ne change pas les licences des composants.

Depuis la version 3.0.0, FreeIPA utilise Samba pour s'intégrer à Active Directory Microsoft par le biais de Cross Forest Trusts. FreeIPA prend en charge les ordinateurs Linux, Unix, Windows et Mac OS X. 

Sources: 

  - [Wikipedia](https://fr.wikipedia.org/wiki/FreeIPA)
  - [FreeIPA](https://www.freeipa.org/)

## Composants

| Composant               | Détails   |
| -----------             | ----------- |
| Enterprise Linux        | Système d'exploitation GNU / Linux        |
| 389 Directory Server    | Implémentation LDAP        |
| Kerberos 5 (MIT)        |  Authentification et SSO |
| Serveur HTTP Apache     | Interface utilisateur Web et Infrastructure logicielle |
| Python                  | Infrastructure logicielle |
| DogTab                  | Autorité de certification interne (PKI) |
| Samba/Winbind           | Intégration Active Directory |
| Named                   | Intégration DNS |
| PKCS#11                 | API Cryptographique pour FreeIPA |

Cette ensemble de services permet d'apporter une solution libre et opensource aux produits nativements intégrés côté Windows avec l'AD/DNS et les composants gravitants.

Le projet est développé par **RedHat**, le produit officiel est **RedHat Identity Management**.