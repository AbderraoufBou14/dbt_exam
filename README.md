# Mission : audit technique et plan de refactoring du moteur Raw Vault

Tu es un Data Engineer senior spécialisé en Python, PySpark, Hive et architecture logicielle.

Nous devons faire évoluer le module `src/Tools/raw_vault_tests/` pour compléter les contrôles fonctionnels, améliorer la maintenabilité et les performances, et partitionner les tables d'audit.

Le lead développeur a autorisé :
- L'implémentation des contrôles manquants.
- Le refactoring de `engine.py`, actuellement trop volumineux.
- Une modularisation raisonnable.
- L'optimisation des traitements.
- Le partitionnement de `audit_run`, `audit_result` et `audit_detail` par date d'exécution.

## Étape demandée : analyse uniquement

Examine le code et ses dépendances réelles, notamment :

- `engine.py`
- `controls.py`
- `analyzer.py`
- Les configurations JSON Raw Vault.
- Les tests automatisés existants.
- Les définitions et écritures des tables d'audit.
- Les scripts de lancement et le packaging si nécessaire.

## Travaux à réaliser

**1. Cartographie**

Identifier les responsabilités des fonctions existantes, leurs appels, les dépendances et les duplications de logique.

**2. Couverture fonctionnelle**

Comparer les contrôles existants aux spécifications disponibles.

Point déjà identifié : `MAS_HK_ABSENT_HUB` semble absent, alors qu'il est demandé dans l'Excel.

Vérifier aussi les autres exigences, notamment les comparaisons entre sources et SAT/MAS, sans inventer de règles.

**3. Refactoring**

Proposer une architecture simple, avec un nombre limité de modules spécialisés et une séparation claire entre configuration, logique de contrôle, orchestration et persistance.

**4. Performances**

Identifier les opérations Spark potentiellement coûteuses et les structures Python inutilement complexes.

Distinguer les optimisations démontrables des hypothèses nécessitant un benchmark.

**5. Partitionnement des audits**

Localiser les schémas Hive et les écritures Spark des trois tables.

Proposer un partitionnement journalier cohérent avec `run_id`, l'historique existant et les usages Hue/Impala.

Décrire les impacts DDL, migration, écriture, lecture, compatibilité et tests.

**6. Plan de développement**

Fournir un plan en étapes limitées, chacune comportant :
- Objectif.
- Fichiers concernés.
- Modifications nécessaires.
- Risques.
- Tests associés.
- Critères de validation.

## Contraintes

- Ne modifier aucun fichier.
- Ne lancer aucun job Spark.
- Ne modifier aucune table Hive.
- Ne pas installer de dépendances.
- Ne pas changer la version du package.
- Ne pas créer de commit.
- Ne pas inventer des fonctions ou schémas absents du dépôt.
- Préserver le comportement existant tant que le changement n'est pas explicitement décidé.

## Résultat attendu

Un rapport Markdown court mais techniquement précis, avec :

1. Cartographie de l'existant.
2. Matrice des contrôles présents/manquants.
3. Architecture cible proposée.
4. Opportunités d'optimisation classées par priorité.
5. Stratégie de partitionnement des audits.
6. Plan de modifications incrémentales.

Indique clairement les décisions qui doivent être validées avant de commencer le développement.
