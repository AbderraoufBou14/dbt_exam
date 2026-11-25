Nous commençons la première étape de l'évolution du moteur d'audit Raw Vault vers un nouveau modèle V2.

**Objectif : faire évoluer directement le modèle Python existant et la propagation des métadonnées, en préparant la structure définitive des futures tables d'audit V2.**

Cette étape ne comprend pas encore la création physique des tables Hive ni l'activation des écritures V2.

## 1. Modèle cible

Les quatre futures tables sont :

- `AUDIT_TEST_V2`
- `AUDIT_RUN_V2`
- `AUDIT_RESULT_V2`
- `AUDIT_DETAIL_V2`

Évolutions validées :

**Renommage :**
- `table_name` → `target_table_name`
- `physical_table_name` → `target_physical_table_name`

**Ajout :**
- `source_table_name` : identification de la table source utilisée par le contrôle.
- `run_date` : colonne technique de partitionnement quotidien, de type STRING au format `YYYY-MM-DD`.

Structure attendue :

- `AUDIT_TEST_V2` : nouveaux noms de colonnes + `source_table_name`, sans partitionnement.
- `AUDIT_RUN_V2` : schéma existant + `run_date`.
- `AUDIT_RESULT_V2` : nouveaux noms + `source_table_name` + `run_date`.
- `AUDIT_DETAIL_V2` : nouveaux noms + `source_table_name` + `run_date`.

Conserver les autres colonnes et leurs types, notamment les horodatages existants.

`run_date` doit être dérivée en UTC du `started_at` du run. Une même exécution doit avoir exactement la même `run_date` dans RUN, RESULT et DETAIL, y compris si elle se termine après minuit.

Préparer les définitions de schéma définitives pour l'étape suivante, sans créer physiquement les tables.

## 2. Traçabilité de la source

Faire évoluer les objets et fonctions existants pour transmettre correctement `source_table_name` jusqu'aux structures V2.

Règles fonctionnelles :

- Pour les contrôles Source → HUB/SAT/MAS/LINK, utiliser le nom de la table source provenant de la configuration réellement utilisée pour le contrôle.
- Les noms de tables sources déclarés dans les JSON correspondent aux noms physiques Hive : aucune distinction supplémentaire n'est nécessaire.
- Pour les contrôles internes au Raw Vault et les contrôles de comptage, `source_table_name` doit être NULL.
- `target_table_name` et `target_physical_table_name` continuent de désigner l'objet Raw Vault concerné par le contrôle.
- Conserver `SOURCE_BK_ABSENT_HUB.uniqueness_columns` avec les colonnes BK source, conformément à la dernière correction validée.

La propagation doit couvrir tous les contrôles Source → Vault concernés, et pas uniquement l'objet `instrument`.

Ne pas modifier les calculs des BK composites, les HASHDIFF, les anti-jointures, les comptages ou les règles fonctionnelles des 13 contrôles.

## 3. Cohérence entre TEST, RESULT et DETAIL

Préparer la logique V2 pour garantir une traçabilité cohérente :

**AUDIT_TEST_V2**
- Une définition de contrôle doit pouvoir être distinguée selon sa table source.
- Adapter la déduplication existante pour ne pas fusionner deux sources différentes.
- Gérer correctement les valeurs NULL de `source_table_name` pour les contrôles sans source.
- Vérifier le cas où une même source possède plusieurs configurations distinctes pour la même cible.

**AUDIT_RESULT_V2**
- Chaque résultat doit conserver la bonne table source.
- Ne pas modifier les statuts, les valeurs calculées ni la granularité actuelle : un résultat par source.

**AUDIT_DETAIL_V2**
- Chaque anomalie doit conserver la provenance source appropriée.
- Vérifier que les détails restent rattachables sans ambiguïté au bon résultat, y compris pour plusieurs sources générant des KO.
- Conserver le fonctionnement actuel de la génération des détails et ses limites existantes.

Réutiliser autant que possible les structures et fonctions déjà présentes. Ne pas introduire de mécanisme générique ou d'identifiant supplémentaire sans nécessité démontrée.

## 4. Évolution du code et conservation des tables historiques

**Distinction importante :**

Nous voulons conserver les anciennes tables Hive `AUDIT_TEST`, `AUDIT_RUN`, `AUDIT_RESULT` et `AUDIT_DETAIL` uniquement pour l'historique et la comparaison.

En revanche, **nous ne voulons pas conserver une version V1 du moteur Python**.

Consignes :

- Faire évoluer directement les modules et fonctions Python existants vers le nouveau modèle V2.
- Ne pas créer de moteur V2 parallèle, ni maintenir deux chemins d'écriture V1/V2.
- Ne pas conserver d'anciens modèles uniquement pour la rétrocompatibilité des tables historiques.
- Ne pas créer de nouveaux modules si les modules existants suffisent.
- Préserver les interfaces KSH, le point d'entrée Spark et le fonctionnement des 13 contrôles métier.
- Préparer le code pour que les écritures futures ciblent exclusivement les tables `AUDIT_*_V2`.
- Ne pas modifier les tables Hive V1 existantes.

**Limite de cette première étape :** ne pas encore créer les tables V2, activer les écritures Hive V2 ou lancer un traitement qui nécessite leur existence.

Il est acceptable que cette étape ne constitue pas encore une version exécutable de bout en bout contre Hive : l'activation du stockage sera effectuée à l'étape 2.

## 5. Préparation du partitionnement

Préparer uniquement les champs et la propagation de `run_date`.

Les futures règles sont :

- `AUDIT_TEST_V2` : non partitionnée.
- `AUDIT_RUN_V2` : partitionnée par `run_date`.
- `AUDIT_RESULT_V2` : partitionnée par `run_date`.
- `AUDIT_DETAIL_V2` : partitionnée par `run_date`.

Ne pas encore modifier les écritures Spark pour activer le partitionnement.

Attention : le moteur utilise actuellement `insertInto()`, qui dépend de l'ordre des colonnes. Préparer des schémas cohérents avec la future organisation des colonnes de partition, sans introduire d'écriture incompatible.

Ne pas introduire d'optimisation Spark ou d'évolution des règles métier dans ce chantier.

## 6. Tests et vérifications

Adapter ou ajouter les tests unitaires pertinents pour couvrir :

1. Les nouveaux noms de colonnes dans les structures V2.
2. La transmission de `source_table_name` pour les deux sources de `instrument`.
3. La propagation des métadonnées pour les autres familles Source → Vault.
4. Les BK composites, notamment `position_mens`.
5. La valeur NULL pour les contrôles sans source externe.
6. La distinction de deux sources différentes dans la déduplication d'AUDIT_TEST.
7. Le rattachement cohérent entre les structures TEST, RESULT et DETAIL.
8. Le calcul de `run_date` en UTC et le cas d'un run passant minuit.
9. La non-régression des 13 contrôles métier.

Exécuter les suites locales de tests counting et analyzer.

Si certains tests d'intégration Hive ne peuvent pas passer avant la création des tables V2, les identifier clairement : ne pas introduire de mécanisme V1 temporaire uniquement pour les faire réussir.

## 7. Bilan attendu

À la fin, présente un retour concis et factuel comprenant :

- Les fichiers et fonctions modifiés.
- Les nouveaux schémas des quatre tables V2.
- Le chemin de propagation de `source_table_name`.
- La logique retenue pour la déduplication d'AUDIT_TEST et le rattachement des détails.
- Le calcul de `run_date`.
- Les tests exécutés avec leurs résultats.
- Les éventuelles limites et le travail précisément restant pour l'étape 2.

**Contraintes finales :**

- Aucun changement des règles métier des 13 contrôles.
- Aucune modification des tables Hive existantes.
- Aucune création physique de table V2 dans cette étape.
- Aucun lancement KSH/YARN, packaging ou déploiement.
- Aucun changement du dashboard pour le moment.
- Ne pas traiter d'autres sujets que ceux de cette première étape.

Le but est d'obtenir un modèle V2 cohérent et testé avant de passer à la création et aux écritures des tables Hive partitionnées.
