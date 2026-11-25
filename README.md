Nous passons à l'étape 2 du chantier V2 : **création physique des tables d'audit partitionnées et activation des écritures Spark**.

L'étape 1 est déjà implémentée : schémas V2, renommages, propagation de `source_table_name`, calcul de `run_date` et adaptation des modèles Python.

Travaille sur cette base existante. Ne refais pas l'étape 1 et ne crée aucune seconde implémentation du moteur.

## 1. Création des quatre tables V2

Adapter `ensure_audit_tables()` dans `audit_store.py` pour créer automatiquement, si elles n'existent pas, les tables suivantes dans la base d'audit configurée à l'exécution :

- `AUDIT_TEST_V2` : non partitionnée.
- `AUDIT_RUN_V2` : partitionnée par `run_date`.
- `AUDIT_RESULT_V2` : partitionnée par `run_date`.
- `AUDIT_DETAIL_V2` : partitionnée par `run_date`.

Conserver le format PARQUET et les conventions actuelles de création des tables.

Utiliser les schémas V2 définitifs préparés à l'étape 1, sans introduire de nouvelles colonnes inutiles.

Dans les tables partitionnées :
- `run_date` doit être de type STRING, au format `YYYY-MM-DD`.
- La colonne doit être déclarée dans `PARTITIONED BY`, et non en double dans la liste des colonnes ordinaires.
- Elle doit figurer à la bonne position dans les DataFrames destinés à l'écriture Spark.

**Les quatre tables historiques sans suffixe `_V2` doivent rester totalement intactes.**

Aucun DROP, ALTER, INSERT ou autre opération sur les anciennes tables.

## 2. Écritures Spark partitionnées

Adapter les écritures existantes pour alimenter exclusivement les tables V2.

Le moteur utilise actuellement `df.write.mode("append").insertInto(...)`, avec une correspondance des colonnes par position.

Vérifie et sécurise :
- L'ordre exact des colonnes entre les DataFrames Spark et les tables Hive.
- La présence et le type de `run_date`.
- La création et l'alimentation correctes des partitions quotidiennes.
- Le comportement des partitions dynamiques dans notre environnement Spark/Hive.
- La cohérence des écritures RUN, RESULT et DETAIL.

Attention : ne pas ajouter mécaniquement `.partitionBy("run_date")` à `insertInto()`. Utiliser la méthode compatible avec les tables Hive déjà déclarées partitionnées.

Conserver `run_date` calculée depuis `started_at` en UTC, identique dans les trois tables d'exécution, même lorsqu'un traitement franchit minuit.

Pour `AUDIT_TEST_V2`, conserver une écriture non partitionnée et la déduplication multisources préparée à l'étape 1.

## 3. Sécurité et cohérence des schémas

Avant les écritures, vérifier que les schémas V2 existants, s'ils sont déjà présents dans Hive, sont compatibles avec les schémas attendus.

Éviter particulièrement les écritures silencieusement décalées dues à `insertInto()`.

En cas d'incompatibilité, remonter une erreur explicite plutôt que modifier ou supprimer automatiquement les tables.

Les anciennes tables V1 ne doivent jamais être utilisées comme solution de repli.

Ne pas coder en dur la base DEV : conserver la base d'audit fournie par le KSH et les paramètres existants.

## 4. Préservation du fonctionnement métier

Ne modifier aucune règle des 13 contrôles.

Conserver :
- Les statuts OK/KO/ERROR.
- Les comptages et les anti-jointures actuels.
- Les calculs BK/HK/HDIFF.
- La logique un contrôle par source, sans UNION.
- Les valeurs de `uniqueness_columns` déjà validées.
- La propagation de `source_table_name` dans TEST, RESULT et DETAIL.
- Les limites existantes de génération des détails.
- Les interfaces `engine.py`, `controls.py` et KSH.

Aucune duplication du moteur, aucune optimisation Spark ou refactorisation hors sujet.

## 5. Tests nécessaires

Effectuer les tests locaux nécessaires pour vérifier :

1. Les DDL des quatre tables V2, dont les trois `PARTITIONED BY (run_date STRING)`.
2. La conformité exacte des schémas et de l'ordre des colonnes.
3. Les écritures dans des tables V2 partitionnées avec Spark local, si l'environnement de test le permet.
4. L'alimentation des partitions par date.
5. Deux exécutions sur deux jours différents produisant deux partitions distinctes.
6. Une exécution franchissant minuit, avec la même `run_date` dans RUN/RESULT/DETAIL.
7. La persistance correcte de `source_table_name`, y compris NULL pour les contrôles internes.
8. La déduplication d'`AUDIT_TEST_V2` sur plusieurs exécutions et plusieurs sources.
9. L'absence de modification des tables historiques.

Exécuter ensuite les suites de non-régression counting et analyzer.

Si un test local ne peut pas reproduire fidèlement Hive DEV, indiquer clairement cette limite.

## 6. Limites de cette étape

- Ne pas modifier le dashboard : il sera traité à l'étape 3.
- Ne pas modifier les anciennes tables Hive.
- Ne pas lancer de KSH/YARN ni de traitement réel sur le cluster.
- Ne pas procéder à une création manuelle des tables dans Hive DEV : leur création doit être prévue par le moteur au lancement du prochain traitement.
- Ne pas créer de mécanisme de migration des données historiques.

## 7. Bilan attendu

À la fin, fournis un bilan factuel et concis :

- Fichiers et fonctions modifiés.
- DDL définitifs des quatre tables V2.
- Méthode retenue pour les écritures partitionnées.
- Vérifications de compatibilité des schémas.
- Tests exécutés et résultats.
- Limites éventuelles à vérifier lors de la première exécution réelle DEV.

**Objectif final :** disposer d'un moteur prêt à créer et alimenter automatiquement les quatre tables `AUDIT_*_V2` lors de ma prochaine exécution KSH, sans aucune intervention sur les anciennes tables d'audit.
