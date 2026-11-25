Nous préparons une évolution des tables d'audit du moteur Raw Vault. Avant de prendre les décisions techniques, je veux un état des lieux précis de l'implémentation actuelle.

Les évolutions déjà validées fonctionnellement sont :
- Renommer `table_name` → `target_table_name`.
- Renommer `physical_table_name` → `target_physical_table_name`.
- Ajouter `source_table_name` pour identifier chaque source.
- Introduire le partitionnement des tables d'audit.

Le moteur comporte 13 contrôles. Nous conservons un contrôle par source, sans UNION, et les calculs fonctionnels doivent rester inchangés.

Le périmètre de ton investigation est **le code DF6 et les tables de l'environnement DEV uniquement**. Ne fais aucune hypothèse sur REC, PREX ou PROD.

### 1. Structure actuelle des tables d'audit

Pour `AUDIT_TEST`, `AUDIT_RUN`, `AUDIT_RESULT` et `AUDIT_DETAIL`, identifie :

- Les colonnes et leurs types exacts.
- Les clés logiques et relations entre les tables.
- Les formats physiques, emplacements HDFS et bases Hive configurées.
- Le partitionnement actuel, s'il existe.
- La façon dont les tables sont créées et initialisées.

Précise ce qui est confirmé par le code et ce qui a été réellement vérifié dans Hive DEV.

### 2. Fonctionnement des écritures Spark

Retrouve les fonctions exactes qui :
- Construisent les DataFrames d'audit.
- Créent les tables Hive.
- Insèrent les résultats.
- Gèrent les runs, la déduplication des contrôles et les détails d'anomalies.

Explique les méthodes d'écriture utilisées (`insertInto`, `saveAsTable`, SQL...), la correspondance des colonnes par nom ou par position et le comportement en cas de différence entre le schéma Spark et celui de Hive.

Indique si le moteur peut détecter ou corriger une incompatibilité de schéma.

### 3. Origine des métadonnées des contrôles

Pour les 13 contrôles, identifie comment sont déterminés :
- `table_name`
- `physical_table_name`
- `uniqueness_columns`
- Le nom de la table source lorsqu'elle existe.

Vérifie si le nom source est déjà disponible dans les objets Python et comment il pourrait être propagé jusqu'aux résultats.

Le nom des tables sources dans les JSON correspond au nom physique Hive : inutile d'inventer une distinction supplémentaire.

Examine aussi la déduplication d'`AUDIT_TEST` et le rattachement d'`AUDIT_DETAIL` aux résultats, particulièrement pour les objets multisources comme `instrument`.

### 4. Dépendances des colonnes actuelles

Recherche dans le dépôt toutes les utilisations de `table_name`, `physical_table_name` et des structures des tables d'audit.

Identifie les fichiers, fonctions, tests, requêtes SQL et composants d'analyse qui dépendent de ces noms.

Distingue les véritables dépendances des simples occurrences de texte.

### 5. Possibilités de modification des tables Hive DEV

À partir des DDL et du fonctionnement réel du moteur, explique :

- Si les tables existantes peuvent être renommées et enrichies avec `ALTER TABLE`.
- Si une recréation serait techniquement plus simple.
- Ce qui arriverait si nous déployions le nouveau code sans faire évoluer les tables existantes.
- Comment intégrer un partitionnement par date aux tables concernées.
- Les implications du partitionnement sur les écritures Spark, les lectures, les requêtes et les performances.
- Si l'ajout du partitionnement impose une recréation ou une migration particulière.

L'historique actuel en DEV n'a pas besoin d'être préservé. Les tables pourront être recréées si nous choisissons cette solution.

### 6. Rapport attendu

Je veux un rapport technique détaillé, mais sans code inutile, contenant :

1. Un tableau des quatre tables avec leurs schémas, formats, partitions et méthodes d'écriture.
2. Le parcours des métadonnées depuis les JSON jusqu'à `AUDIT_TEST`, `AUDIT_RESULT` et `AUDIT_DETAIL`.
3. Les fonctions et chemins de fichiers réellement concernés.
4. Les dépendances et risques de compatibilité identifiés.
5. Les possibilités techniques de renommage, d'ajout de colonne et de partitionnement.
6. Les informations manquantes et les vérifications DEV nécessaires pour choisir une stratégie fiable.

**Important : ne prends pas encore de décision à notre place et n'implémente rien.**

Ne modifie aucun fichier, aucune table Hive, aucune donnée et aucune configuration. Aucun commit, push, packaging ou déploiement.

Si tu ne peux pas interroger Hive DEV directement, indique-le clairement et fournis les requêtes SQL de lecture seule strictement nécessaires pour que je récupère les informations manquantes dans Hue.
