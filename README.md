# Mission — Générer un cahier de validation fonctionnelle SQL pour le Raw Vault DF6

## 1. Objectif

Tu es un Data Engineer senior spécialisé en Apache Spark, Hive, Impala, Data Vault et validation fonctionnelle des données.

Je travaille sur l'implémentation et la validation de contrôles automatisés de qualité des données du Raw Vault DF6.

Je souhaite que tu produises **un document SQL unique, structuré, commenté et directement exploitable dans Hue (moteur Impala)**, permettant de vérifier indépendamment les résultats générés par notre moteur d'audit PySpark.

**Le document doit couvrir les contrôles fonctionnels de qualité des données, et non les contrôles de comptage déjà traités dans un autre périmètre.**

## 2. Contexte technique

Le moteur d'audit se trouve dans :

`src/Tools/raw_vault_tests/engine.py`

Le point d'entrée se trouve dans :

`src/Tools/raw_vault_tests/controls.py`

Les configurations des objets métier sont dans :

`parm/raw/init/<objet_metier>/`

Les composants Raw Vault utilisent notamment les préfixes :

- HUB : `hrv_`
- SAT : `srv_`
- MAS : `mrv_`
- LINK : `lrv_`

Bases DEV :

- `db_dev_si3` : sources.
- `db_dev_df6_brute` : Raw Vault.
- `db_dev_df6_travail` : tables d'audit.

Tables d'audit :

- `audit_run`
- `audit_test`
- `audit_result`
- `audit_detail`

Les résultats des contrôles sont reliés à une exécution par `run_id`.

Un audit a déjà été exécuté avec succès sur l'objet métier `instrument` le 09/11/2026, avec 52 résultats enregistrés.

Run ID observé :

`5a942b04-96e9-41ad-8614-767a4345a4fe`

L'objet métier `instrument` permet de vérifier plusieurs contrôles, mais ne comporte pas nécessairement toutes les relations LINK/MAS nécessaires.

## 3. Contrôles à couvrir

Le document doit couvrir les six familles fonctionnelles suivantes :

| N° | Famille | Contrôle |
|---|---|---|
| 1 | Source / HUB | `SOURCE_BK_ABSENT_HUB` |
| 2 | SAT / HUB | `SAT_HK_ABSENT_HUB` |
| 3 | MAS / HUB | Identifier le contrôle réel dans le moteur |
| 4 | LINK / HUB | `LINK_FK_ABSENT_HUB` |
| 5 | LINK / SAT | Contrôles bidirectionnels existants |
| 6 | LINK / MAS | Contrôles bidirectionnels existants |

Vérifier les noms exacts des `test_id` dans le code avant de construire les requêtes.

Ne pas inventer de contrôle ou de relation absente du dépôt.

## 4. Exploration préalable obligatoire

Avant de rédiger le document SQL :

1. Examiner `engine.py` pour identifier toutes les fonctions de qualité réellement implémentées.
2. Identifier leurs règles de comparaison, clés de jointure, traitements des NULL et méthodes de comptage.
3. Examiner les configurations JSON pour identifier les relations entre objets.
4. Identifier les tables physiques et colonnes réellement disponibles.
5. Examiner la structure des résultats d'audit et les champs disponibles.
6. Exploiter les spécifications Excel si elles sont accessibles dans le workspace.

**Ne pas déduire les colonnes de jointure uniquement à partir des préfixes ou noms de tables.**

Ne pas modifier le code Python ni les configurations.

## 5. Structure attendue du document

Créer un fichier :

`validation_fonctionnelle_raw_vault.sql`

Il doit contenir les sections suivantes.

### Section 0 — Contexte d'exécution

Commentaires indiquant :

- Environnement DEV.
- Base Raw Vault.
- Base des audits.
- Objet métier testé.
- Run ID.
- Date de validation.
- Version du package si connue.
- Limites de couverture.

Ajouter une requête permettant de consulter les résultats de l'exécution concernée dans `audit_result`.

### Section 1 — Contrôle BK source / HUB

Inclure :

- Vérification d'un cas OK.
- Vérification d'un cas KO si disponible.
- Comparaison indépendante via `LEFT ANTI JOIN`.
- Contrôle des différentes tables sources.
- Cas de sources multiples.
- Point de vigilance concernant l'union des BK sources.
- Vérification du résultat par rapport à `SOURCE_BK_ABSENT_HUB`.

Pour `instrument`, les deux sources documentées sont :

- `db_dev_si3.lac_pvr_instrument_delta_histo`, BK `id`.
- `db_dev_si3.lac_pvo_inventaires_quot_dern_vers_instrument`, BK `codevaleur`.

HUB :

`db_dev_df6_brute.hrv_instrument`

BK cible :

`id_instrument`

Résultat déjà observé : 0 BK absente sur chacune des deux sources.

Prendre en compte les particularités de normalisation ou de transformation des BK si le moteur les applique.

### Section 2 — Contrôle SAT / HUB

Inclure au minimum deux scénarios déjà identifiés :

**Cas OK :**

- SAT : `srv_inst_future`
- Résultat d'audit : OK / 0
- Requête SQL indépendante : 0 anomalie observée

**Cas KO :**

- SAT : `srv_inst_action`
- Résultat d'audit : KO / 7
- Requête SQL indépendante : 7 anomalies observées

Les clés de jointure exactes doivent être vérifiées dans les JSON et le code.

Inclure une requête de comptage des anomalies et une requête permettant de consulter les Hash Keys concernées, avec une limite raisonnable de résultats.

### Section 3 — Contrôle MAS / HUB

Identifier un MAS réellement configuré et son HUB parent.

Produire :

- Une requête de comparaison des Hash Keys.
- Une requête de comptage des anomalies.
- Une requête de consultation des détails.
- Une requête de comparaison avec les résultats de l'audit.

Si l'objet `instrument` ne possède pas la relation nécessaire pour couvrir la règle, identifier un autre objet configuré.

Ne pas inventer de cas KO s'il n'en existe pas dans les données observées.

### Section 4 — Contrôle LINK / HUB

Pour chaque relation LINK/HUB pertinente :

- Identifier la table LINK.
- Identifier le HUB référencé.
- Identifier la Foreign Key présente dans le LINK.
- Vérifier les références manquantes par `LEFT ANTI JOIN`.
- Comparer les résultats avec `LINK_FK_ABSENT_HUB`.

Inclure un cas OK et un cas KO lorsque disponibles.

### Section 5 — Contrôle LINK / SAT

Couvrir les directions réellement implémentées :

- LINK → SAT.
- SAT → LINK.

Pour chaque direction :

- Identifier les colonnes de comparaison.
- Construire la requête SQL.
- Comparer au résultat d'audit.
- Documenter les différences éventuelles.

Les deux directions ne sont pas nécessairement équivalentes.

### Section 6 — Contrôle LINK / MAS

Couvrir les directions réellement implémentées :

- LINK → MAS.
- MAS → LINK.

**Ce contrôle est actuellement en attente d'identification d'un objet métier possédant une relation LINK/MAS exploitable.**

Explorer les configurations JSON pour proposer un objet candidat.

Si les tables ou les résultats nécessaires ne sont pas disponibles, conserver la section comme **À EXÉCUTER ULTÉRIEUREMENT**, avec des requêtes préparées seulement si leurs noms et clés ont été confirmés.

Ne jamais annoncer un scénario comme validé sans résultat réellement observé.

### Section 7 — Synthèse de validation

Préparer des requêtes pour récupérer les résultats par famille de contrôles et par statut :

- OK.
- KO.
- ERROR.

Ajouter un tableau de suivi en commentaires SQL :

| Contrôle | Objet | Cas OK | Cas KO | Résultat comparé | Statut |
|---|---|---|---|---|---|

Distinguer :

- Exécuté.
- Vérifié dans Hue.
- Validé sur les cas testés.
- Partiellement validé.
- Non couvert.
- À investiguer.

## 6. Règles indispensables pour les requêtes

Chaque requête SQL doit être précédée de commentaires précisant :

1. Son identifiant et le contrôle concerné.
2. Son objectif fonctionnel.
3. Les tables interrogées.
4. Les colonnes utilisées.
5. Le scénario : OK, KO ou investigation.
6. Le résultat attendu, uniquement lorsqu'il est connu.
7. La façon d'interpréter le résultat.
8. Les éventuelles limites de la vérification.

Privilégier des requêtes **SELECT uniquement**.

Aucune commande `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `DROP`, `ALTER` ou `CREATE` sur les données partagées.

Éviter les requêtes inutilement coûteuses.

Les requêtes doivent être compatibles avec Impala et exécutables individuellement depuis Hue.

Ne pas considérer `LIMIT` comme une garantie de faible coût d'exécution pour les jointures et agrégations.

Ne pas confondre :

- Nombre de lignes manquantes.
- Nombre de clés distinctes manquantes.
- Nombre de groupes de doublons.
- Nombre de résultats d'audit.

Reproduire exactement la métrique du moteur PySpark avant de comparer les chiffres.

Ne pas fabriquer un scénario KO en modifiant les tables Hive. Si aucun KO réel n'est disponible, le signaler et proposer un test unitaire isolé pour compléter la couverture.

## 7. Livrable demandé

Produis directement le fichier SQL complet et enregistré dans le workspace, idéalement dans un emplacement de travail non versionné tant que je n'ai pas validé son intégration au dépôt.

Je veux un **cahier de validation fonctionnelle opérationnel**, utilisable pour :

- Exécuter les contrôles manuellement depuis Hue.
- Comparer les résultats avec le moteur automatisé PySpark.
- Conserver les preuves de validation DEV.
- Identifier les contrôles non couverts.
- Préparer la recette fonctionnelle.
- Alimenter les commentaires du ticket de suivi.

Termine avec une synthèse courte indiquant :

- Les contrôles pour lesquels des requêtes ont été préparées.
- Les cas OK/KO déjà documentés.
- Les contrôles encore sans objet représentatif.
- Les informations impossibles à déterminer à partir du code et des configurations.

**Ne modifie pas `engine.py`, ne lance aucun Spark-submit, ne modifie aucune table Hive et ne réalise aucun déploiement.**

Le livrable attendu est uniquement le fichier SQL, ses commentaires et la synthèse de couverture.
