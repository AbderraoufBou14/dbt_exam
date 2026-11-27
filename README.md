Nous poursuivons l'analyse des divergences HDIFF sur les quatre contrôles :

- SOURCE_HK_ABSENT_SAT
- SOURCE_HK_ABSENT_MAS
- SOURCE_HK_ABSENT_LINK_SAT
- SOURCE_HK_ABSENT_LINK_MAS

Ton précédent rapport a présenté six solutions (A à F). Je veux maintenant une **investigation complémentaire, approfondie mais ciblée**, pour déterminer quelle solution est réellement applicable au moteur DF6 existant.

Notre objectif est de permettre aux utilisateurs métier d'identifier les colonnes du payload responsables d'un HDIFF divergent, directement ou facilement depuis les résultats d'audit.

Le problème des discriminants NULL est déjà corrigé en V2. Ne reviens pas sur ce sujet. Le dashboard est hors périmètre.

**Aucune modification de code ou de tables : analyse uniquement.**

### 1. Déterminer comment identifier la bonne ligne Vault

C'est la question prioritaire.

Examine les implémentations réelles des loaders SAT/MAS et leurs configurations JSON, ainsi que les quatre contrôles d'audit.

Nous savons qu'une même HK peut correspondre à plusieurs versions historisées.

Je veux comprendre précisément :

- Comment le loader détermine qu'un nouvel état descriptif doit être inséré.
- Quels rôles jouent `ldts`, `dt_fonc`, `dt_deb_val` et les éventuelles autres dates présentes dans les objets concernés.
- Si le loader compare avec le dernier état, un état à date fonctionnelle donnée, ou selon une autre règle.
- Pour les MAS, si le pattern utilisé fonctionne par occurrence ou par état d'ensemble, notamment en version MAS v2.
- Comment sélectionner sans ambiguïté la ligne Vault comparable à une ligne source dont le couple HK/HDIFF est absent.
- Que faire lorsqu'il existe zéro, une ou plusieurs versions candidates, et comment éviter une sélection arbitraire.

Étudie au moins un SAT et un MAS réellement configurés dans DF6, ainsi que la particularité des satellites rattachés aux LINK.

**Question centrale : sommes-nous techniquement capables d'identifier une version Vault pertinente pour comparer les colonnes du payload, ou seulement de constater l'absence du couple HK/HDIFF dans l'historique ?**

Si le code des loaders est dans un package externe, inspecte la version accessible ; sinon, distingue clairement ce qui est démontré de ce qui reste inconnu.

### 2. Vérifier la fidélité des calculs de HDIFF

Compare précisément le calcul de `source_hdiff_expression()` dans notre moteur d'audit avec celui du pattern de chargement réellement utilisé.

Analyse :
- L'ordre et la sélection des champs `payload` du JSON.
- Les transformations et casts selon `col_type`.
- Les valeurs NULL, chaînes vides, dates et timestamps.
- Les décimaux et la représentation des nombres.
- Les filtres, expressions ou transformations appliqués avant le calcul.
- Les différences potentielles entre une valeur source brute, une valeur transformée et une valeur stockée dans le Vault.

Identifie les divergences réelles possibles entre le loader et l'audit, avec les fichiers/fonctions concernés.

Je veux surtout savoir **si un HDIFF absent peut être provoqué par notre propre reconstruction du hash alors que les données métier ont été correctement chargées**.

Propose les tests ciblés permettant de vérifier cette cohérence, sans les implémenter pour le moment.

### 3. Examiner la faisabilité d'un diagnostic automatique

À partir du code actuel de `control_logic.py`, `models.py` et `audit_store.py`, explique comment nous pourrions :

1. Conserver le calcul principal et les statuts actuels.
2. Récupérer uniquement les anomalies pertinentes après l'anti-jointure.
3. Retrouver les lignes sources originales et les versions Vault comparables.
4. Comparer les colonnes du payload selon les mêmes règles que le loader.
5. Extraire uniquement les noms des colonnes divergentes.
6. Éviter les diagnostics erronés lorsqu'aucune version comparable n'est identifiable.

Vérifie particulièrement ce qui reste disponible après les `groupBy`, les transformations et la limite actuelle de 1 000 détails.

Précise les risques de jointures multiples, les besoins en colonnes intermédiaires et les conséquences Spark sur la volumétrie.

### 4. Déterminer le meilleur format de restitution

Compare les solutions suivantes dans notre architecture V2 réelle :

**A.** Enrichir le JSON `AUDIT_DETAIL_V2.key_values` avec un diagnostic.

**B.** Ajouter une colonne dédiée, par exemple `mismatch_columns`, à `AUDIT_DETAIL_V2`.

**C.** Créer une table complémentaire de diagnostic contenant le contexte du contrôle, les clés de rapprochement, le type d'anomalie et les colonnes divergentes.

**D.** Produire un diagnostic à la demande plutôt que lors de chaque run.

**E.** Séparer les types d'anomalies en plusieurs `test_id`, comme dans ton rapport précédent, ou conserver les `test_id` existants avec une qualification de l'anomalie.

Pour chaque solution, précise :
- Sa faisabilité réelle avec le code existant.
- Le coût Spark prévisible et les données supplémentaires à relire.
- L'impact sur les tables V2, les schémas et les écritures.
- La facilité d'exploitation directement dans Hue.
- Le comportement en cas de plusieurs versions historisées.
- Les risques de mauvaise interprétation métier.
- La complexité de développement et de maintenance.

Nous voulons éviter une V3 ou de nouvelles tables si elles n'apportent pas une valeur suffisante, mais nous sommes ouverts à une table complémentaire si c'est réellement la solution la plus propre.

### 5. Présentation métier souhaitée

Analyse la possibilité de produire un résultat comme :

- Source : table source réellement contrôlée.
- Cible : satellite ou MAS concerné.
- HK et discriminants éventuels.
- HDIFF attendu.
- Type d'anomalie : identité absente / état descriptif non retrouvé / comparaison impossible.
- Colonnes divergentes : liste des noms de colonnes, lorsque la comparaison est fiable.

En première intention, nous privilégions **les noms des colonnes divergentes**, sans stocker systématiquement les valeurs métier en clair.

Détermine si les valeurs source et Vault peuvent être conservées comme diagnostic optionnel, sans introduire inutilement de données sensibles dans les audits.

Montre deux ou trois exemples illustratifs de résultats attendus, notamment un KO sans identité Vault et un KO avec HDIFF divergent.

### 6. Recommandation finale attendue

Je veux un arbitrage technique argumenté répondant à quatre questions :

1. Quel mécanisme serait le plus fiable fonctionnellement pour nos SAT, MAS, LINK-SAT et LINK-MAS ?
2. Quel mécanisme serait le plus simple à maintenir et le moins coûteux en Spark ?
3. Comment intégrer cette fonctionnalité sans remettre en cause les 13 contrôles et les tables d'audit V2 déjà opérationnelles ?
4. Quelles vérifications sont indispensables avant toute implémentation ?

Présente un tableau comparatif, une recommandation priorisée et un plan de mise en œuvre en petites étapes testables.

Appuie les conclusions sur des fonctions, schémas et JSON réels. Distingue clairement les faits confirmés, les risques théoriques et les informations non vérifiables.

**Ne réexplique pas longuement la logique générale des HK/HDIFF déjà connue. Concentre-toi sur les problèmes concrets de correspondance des versions, de fidélité du hash, de disponibilité du payload et de coût du diagnostic.**

Aucun changement des règles métier, aucune modification du moteur, aucune exécution réelle, aucune opération Hive, aucun déploiement.
