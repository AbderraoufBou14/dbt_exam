# [DF6 - PREX] Raw Vault – Flux Financiers – Complément V2 – DF6DFM-547

Bonjour,

Dans le cadre de l'évolution du Datahub Finance de marché (DF6), merci de procéder à la livraison en **pré-exploitation (PREX)** du deuxième lot du HUB Flux Financiers.

Cette évolution consiste à ajouter **5 nouveaux MAS** alimentés par les sources SI3 Résultat de cession, Résultat de cession FIFO et Produits FI YTD DOPE, ainsi qu'à mettre en place **6 jobs Control-M** pour leurs alimentations récurrentes.

**Environnement physique : PREX**

## 1. Livraison du package applicatif DF6

Merci de livrer en PREX le package DF6 suivant, à partir de la version disponible et validée en RECETTE :

- Application : `bigdata-datahub_finance-df6-batch`
- Package : `df6-<VERSION_VALIDÉE>.tar.gz`
- Source : `<CHEMIN_SAS_DU_PACKAGE_RECETTE>`
- Environnement cible : PREX
- Serveur cible : `<SERVEUR_PREX_À_CONFIRMER>`

## 2. Initialisation des nouveaux objets Raw Vault

Merci de procéder à la création des nouveaux objets Hive et à leur chargement initial à l'aide du shell :

`PDDF6AJR0_RAW_VAULT.KSH`

Les traitements doivent être lancés **séquentiellement, sans parallélisation des LOAD**, car plusieurs alimentations écrivent dans le HUB commun Flux Financiers.

Date technique d'initialisation : `<DATE_VALIDÉE_AAAAMMJJ>`

### Résultat de cession quotidien

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t create -c raw -b resultat_cession_quot true`

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t load -c raw -b resultat_cession_quot true`

### Résultat de cession mensuel

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t create -c raw -b resultat_cession_mens true`

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t load -c raw -b resultat_cession_mens true`

### Résultat de cession FIFO

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t create -c raw -b resultat_cession_fifo_quot true`

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t load -c raw -b resultat_cession_fifo_quot true`

### Produits FI YTD DOPE quotidien

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t create -c raw -b produits_fi_ytd_dope_quot true`

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t load -c raw -b produits_fi_ytd_dope_quot true`

### Produits FI YTD DOPE mensuel

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t create -c raw -b produits_fi_ytd_dope_mens true`

`./PDDF6AJR0_RAW_VAULT.KSH -d <DATE> -t load -c raw -b produits_fi_ytd_dope_mens true`

**Attention :** le HUB `hrv_flux_financiers` existe déjà et est partagé avec les alimentations du premier lot. Ne pas supprimer ni réinitialiser les données existantes du HUB. Vérifier l'impact des opérations CREATE avant leur lancement.

## 3. Création des six boîtes Control-M

Merci de créer les six boîtes suivantes, conformément au dossier d'exploitation Control-M DF6 et au schéma d'ordonnancement joints ou référencés.

| Job DF6 | Condition de déclenchement (fin du job SI3) | Groupe KSH |
|---|---|---|
| `XDDF6AJR25_RESULTAT_CESSION_QUOT` | `XDSI3AJL21_RESCESEOD` | `resultat_cession_quot` |
| `XDDF6AJR25_RESULTAT_CESSION_INTRA` | `XDSI3AJL21RESCES_INTRA` | `resultat_cession_quot` |
| `XDDF6AJR25_RESULTAT_CESSION_MENS` | `XDSI3AJL22_RESCESEOM` | `resultat_cession_mens` |
| `XDDF6AJR26_RESULTAT_CESSION_FIFO_INTRA` | `XDSI3AJL23_RESCESFIFO_INTRA` | `resultat_cession_fifo_quot` |
| `XDDF6AJR27_PRODUITS_FI_YTD_DOPE_QUOT` | `XDSI3AJL24_PRDFINYTDQUOT` | `produits_fi_ytd_dope_quot` |
| `XDDF6AJR27_PRODUITS_FI_YTD_DOPE_MENS` | `XDSI3AJL25_PRDFINYTDMENS` | `produits_fi_ytd_dope_mens` |

Pour chaque boîte, utiliser les paramètres d'exécution récurrente suivants :

- PARM 1 : `-d date_ctrlm`
- PARM 2 : `-t load`
- PARM 3 : `-c raw`
- PARM 4 : `-b <GROUPE_FONCTIONNEL>`
- PARM 5 : `false`

Condition Control-M de sortie à reprendre conformément au dossier d'exploitation : `TOPJOB_XDDF6FIN`.

Les périodicités et autres caractéristiques de chaque boîte sont précisées dans les fiches Control-M.

## 4. Contraintes d'exécution et exclusions

Les nouveaux jobs alimentent le HUB partagé :

`hrv_flux_financiers`

Merci de prendre en compte les contraintes suivantes :

- Ne pas exécuter simultanément plusieurs chargements écrivant dans ce HUB.
- Prévoir les exclusions entre les six nouveaux jobs, ainsi qu'avec les alimentations existantes du premier lot : Dividendes, Échéanciers et Flux passés.
- Respecter les dépendances SI3 indiquées dans le dossier Control-M et dans le schéma draw.io.
- Respecter les périodicités et calendriers définis pour chaque traitement.

## 5. Mise au plan des boîtes Control-M

Après création et vérification des six boîtes, merci de procéder à leur mise au plan conformément au calendrier et aux conditions définis dans le dossier d'exploitation.

## 6. Vérifications et retour d'intervention

Merci de confirmer :

- La version applicative effectivement installée en PREX.
- La création et le chargement initial des cinq MAS.
- La bonne conservation des objets et données préexistants du HUB Flux Financiers.
- La création des six boîtes Control-M.
- La configuration des dépendances SI3 et des exclusions.
- La mise au plan des traitements.

En cas d'échec, merci de conserver les logs et de signaler les anomalies à l'équipe DF6 afin d'organiser les actions nécessaires.

**Documents de référence :**
- Dossier d'exploitation Control-M DF6 mis à jour.
- Schéma Control-M DF6 mis à jour.
- Référence du ticket Godzilla `DF6DFM-547`.

Merci.
