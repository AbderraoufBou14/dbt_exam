Avant de passer à l'étape 2 du chantier V2, vérifie uniquement la logique de déduplication d'`AUDIT_TEST_V2`.

Analyse les configurations JSON réelles dans `parm/raw` pour déterminer si une même table source peut alimenter une même cible avec plusieurs mappings ou configurations distincts pour un même contrôle.

Vérifie si la clé actuelle `(test_id, component, target_physical_table_name, source_table_name)` risque de fusionner à tort deux définitions de contrôle différentes.

**Retour attendu :**
- Les éventuels cas concrets identifiés, avec les objets et configurations concernés.
- Le risque réel de déduplication incorrecte.
- Ta recommandation : conserver la clé actuelle ou proposer une correction minimale, uniquement si nécessaire.

Analyse uniquement, sans modification de code ni des tables Hive. Ne lance aucun traitement.
