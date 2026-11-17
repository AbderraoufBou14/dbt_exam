# DF6DFM-75 — Initialisation de session et cartographie technique

Nous reprenons les travaux sur le module `src/Tools/raw_vault_tests/` du projet DF6.

Deux documents ont été ajoutés au contexte de cette session :
- Un document de passation technique issu de l'analyse précédente.
- Un document de contexte applicatif DF6.

## 1. Ta mission pour cette première étape

Prends connaissance des deux documents, puis explore en **lecture seule** le code réellement disponible dans le workspace.

L'objectif est de construire une cartographie fiable des fichiers, fonctions, configurations et dépendances nécessaires pour les prochains développements.

**IMPORTANT :**
- Ne commence aucune implémentation.
- Ne modifie aucun fichier.
- Ne lance aucun refactoring.
- Ne crée aucun test ou script.
- Ne réalise aucun commit, push ou opération de déploiement.
- Ne lance aucun traitement Spark ou audit réel.
- Ne génère pas de nouveau rapport Markdown dans le dépôt.

Tu peux utiliser les commandes de consultation et d'inspection nécessaires.

## 2. Décisions techniques déjà retenues

Les points suivants ont été arbitrés après l'analyse précédente.

**D1 — MAS → HUB**

Le contrôle `MAS_HK_ABSENT_HUB` doit être conçu en considérant les différentes configurations MAS : mono-HUB, multi-HUB et avec CDK.

La comparaison directe entre HK MAS et HK HUB ne doit pas être généralisée sans vérification.

Examine les mécanismes réels de construction des clés dans les patterns Data Vault du projet et les métadonnées JSON.

Pour les cas complexes, identifie les solutions techniquement fiables, les données disponibles et les éventuelles limites.

**D2 — Partitionnement des audits**

Stratégie retenue : migration non destructive.

- Créer des tables V2 partitionnées.
- Reprendre l'historique existant.
- Vérifier l'intégrité des données migrées.
- Effectuer une bascule contrôlée après validation.

Les tables existantes `AUDIT_RUN`, `AUDIT_RESULT` et `AUDIT_DETAIL` doivent être préservées pendant cette phase.

**D3 — Colonne de partition**

Orientation retenue : `partition_date STRING`, format `yyyy-MM-dd`.

Vérifier sa compatibilité avec les conventions du projet, Spark, Hive et Impala avant implémentation.

**D4 — Ordre de développement**

1. Implémentation des contrôles fonctionnels manquants.
2. Refactoring progressif.
3. Partitionnement et migration des tables d'audit.
4. Optimisations complémentaires validées par mesures.

**D5 — Périmètre fonctionnel**

Le périmètre inclut :
- `MAS_HK_ABSENT_HUB`.
- Les cinq contrôles de comparaison avec les sources identifiés comme manquants dans la spécification.

Retrouve leurs définitions exactes dans les documents et configurations disponibles. Ne considère pas leurs identifiants proposés précédemment comme définitifs.

**D6 — Performances Spark**

Établir une référence de performance avant optimisation.

Ne pas modifier pour l'instant les paramètres Spark, notamment `spark.sql.autoBroadcastJoinThreshold`.

Les optimisations futures devront être justifiées techniquement et, lorsque possible, mesurées avant/après.

## 3. Contraintes d'architecture

Le module actuel fonctionne dans un écosystème comprenant GitLab, Spark/PySpark, Hive/Impala, HDFS, des configurations JSON et un lancement par KSH.

`src/Tools/raw_vault_tests/controls.py` est un point d'entrée utilisé par le lancement Spark/YARN.

**Il doit rester compatible avec l'existant.**

Le refactoring devra rester raisonnable : séparation des responsabilités, réduction des duplications, mais sans multiplication inutile de modules.

Les interfaces publiques, tests existants, écritures d'audit et mécanismes de packaging sont à préserver.

Le code réel fait autorité sur les hypothèses des documents. Si tu détectes une divergence, signale-la avec les références précises.

## 4. Cartographie attendue

Explore les endroits pertinents du dépôt pour identifier :

**A. Moteur d'audit**
- Points d'entrée.
- Orchestration.
- Fonctions de contrôle.
- Modèles et structures de données.
- Lecture des configurations.
- Gestion des résultats OK/KO/ERROR et des détails.

**B. Construction Data Vault**
- Code de génération des HK HUB, SAT, MAS et LINK.
- Traitement des BK, CDK, discriminants et hashdiff.
- Patterns de chargement et relations entre composants.
- Différences importantes entre MAS mono-HUB, multi-HUB et avec CDK.

**C. Métadonnées et sources**
- Configurations JSON des objets métier.
- Références entre tables.
- Mappings source → Raw Vault.
- Informations nécessaires aux contrôles source manquants.

**D. Persistance et exploitation**
- Création et écriture dans les tables d'audit.
- DDL existants.
- Dashboard et requêtes Impala.
- Scripts KSH, packaging et dépendances pouvant être affectés.

**E. Tests**
- Localisation des tests existants.
- Couverture actuellement disponible.
- Fixtures et mécanismes de simulation Spark.
- Points où ajouter les futurs tests sans casser l'existant.

**F. Git et environnement**
- Vérifier la branche active, attendue : `feature/test_75_qualité_de_données` (utiliser le nom réellement présent).
- Signaler les éventuelles modifications locales.
- Identifier la différence éventuelle entre le code source du dépôt et les versions installées dans `bin/`.

Évite toute exploration exhaustive des environnements virtuels ou dépendances volumineuses : inspecte uniquement les fichiers pertinents.

## 5. Format de ton retour

Produis une réponse concise et structurée :

**1. Compréhension générale**
Un résumé du rôle du module et de son fonctionnement actuel, en 5 à 10 lignes.

**2. Cartographie des fichiers**

Un tableau avec :
- Chemin du fichier.
- Responsabilité.
- Fonctions ou éléments importants.
- Utilité pour notre développement.
- Risque ou dépendance éventuelle.

Classe les fichiers en trois catégories :
- Essentiels.
- À consulter lors du développement.
- Secondaires.

**3. Cartographie du flux d'exécution**

Présente simplement la chaîne :

KSH → point d'entrée Python → chargement configurations → exécution contrôles → écriture audit → consultation Hue/dashboard.

**4. Points techniques sensibles**

Signale uniquement les points importants à surveiller : compatibilité, calcul des clés, mappings sources, tests, persistance et migration.

Distingue :
- Ce qui est confirmé par le code.
- Ce qui reste à vérifier.

**5. Préparation de la prochaine étape**

Identifie précisément les fichiers que tu devras examiner ou modifier pour commencer le contrôle `MAS_HK_ABSENT_HUB`.

Ne fournis pas encore de code.

## 6. Consignes de travail

Sois rigoureux, mais pragmatique.

Tu as accès directement au code : utilise cet avantage pour vérifier les faits et challenger les conclusions des documents précédents.

Ne considère pas les propositions d'architecture antérieures comme des obligations si tu identifies une solution plus simple ou plus fiable.

Évite de reproduire un audit technique complet déjà réalisé. Nous cherchons principalement une **carte de navigation du projet et une compréhension partagée**.

À la fin, arrête-toi et attends mon prochain feu vert avant toute implémentation.
