# DF6 — Contexte technique complémentaire : Raw Vault, audits, exploitation et diagnostics

> **Usage** : document de contexte pour un assistant de développement opérant sur un workspace autorisé.
> **Périmètre** : écosystème technique DF6 et tests fonctionnels automatisés, sans rattachement à un ticket unique.
> **Confidentialité** : volontairement sans nom de client, identité personnelle, nom d'hôte, URL privée, identifiant de compte ou secret. Les conventions DF6 demeurent potentiellement internes : diffusion exclusivement dans les outils approuvés.
> **Nature des informations** : **[OBSERVÉ]** capture/code/log/exécution vu ; **[DOCUMENTÉ]** README, Confluence ou référentiel fourni ; **[À CONFIRMER]** comportement non établi pour tous les environnements.

## 1. Carte générale du périmètre technique

```text
Sources SI3/LAC (tables Hive : lac_*, delta, *_histo selon les sources)
                  |
                  v
     Configurations JSON DF6 + traitements PySpark
                  |
                  v
     RAW VAULT (HUB / LINK / SAT / MAS)
       hrv_* / lrv_* / srv_* / mrv_*
                  |
                  v
     Business Vault / mise à plat / data marts
                  |
                  v
      Consommation analytique en aval

En parallèle : audit des objets Raw Vault
 JSON objet métier -> KSH ou spark-submit -> Spark/YARN
    -> AUDIT_TEST / AUDIT_RUN / AUDIT_RESULT / AUDIT_DETAIL
    -> consultation Hue/Impala et éventuellement Streamlit
```

- **[OBSERVÉ]** Plateforme : Cloudera CDP On-Premise, Edge Linux en SSH, HDFS, Hive, Impala, Hue, Spark/PySpark, YARN, Python, KSH, GitLab.
- **[DOCUMENTÉ]** Outillage transverse : packaging Python (.whl, environnements virtuels, payloads ZIP), Artifactory, livraison applicative via pipeline et dispositif de déploiement, ordonnancement Control-M, dashboard Streamlit.
- **[OBSERVÉ]** Bases en DEV : `db_dev_si3` (amont/LAC), `db_dev_df6_brute` (Raw Vault), `db_dev_df6` (Business Vault), `db_dev_df6_travail` (travail et audits). **[OBSERVÉ]** Une base `db_rec_df6_travail` existe en recette. Les autres conventions de noms doivent être vérifiées localement.
- **[DOCUMENTÉ]** Plusieurs espaces DEV, RECETTE, PREX, PROD ; PREX, PSI et PROD ne doivent pas être assimilés sans examiner les procédures de livraison.
- **[IMPORTANT]** Distinguer cinq couches : fichiers Linux sur Edge, fichiers HDFS, métadonnées/table Hive, moteur SQL Hue (Hive ou Impala), exécution distribuée Spark/YARN.

## 2. Objets métier et métadonnées

**[OBSERVÉ]** Le dépôt contient des configurations sous `parm/raw/init/<objet_metier>/` et d'autres chemins `parm/raw/` ; chaque répertoire d'objet métier peut contenir :

```text
parm/raw/init/<objet_metier>/
├── df6_raw_vault_hub.json
├── df6_raw_vault_sat.json
├── df6_raw_vault_mas.json
└── df6_raw_vault_link.json   # si applicable
```

- Les JSON définissent les tables à produire/contrôler : **un même fichier peut déclarer plusieurs tables**. La présence d'un composant détermine quels tests sont pertinents ; une relation doit être déclarée pour les contrôles relationnels correspondants.
- **[OBSERVÉ]** Exemples d'objets métier vus sous `init` : `instrument`, `echeanciers_quot`, `echeanciers_mens`, `dividendes_ytd_quot`, `dividendes_ytd_mens` et plusieurs objets `link_*`.
- **[OBSERVÉ]** Le code des tests lit notamment `tables`, `dest_table_name`, `hubs`, `cles_src`, `src_table_name`, `suffixe_base_source`/`appli_source`, `champ_discriminant`, `cdk`. Vérifier chaque propriété sur les JSON réels et ne pas généraliser leurs valeurs.
- **[IMPORTANT]** `init` signifie contexte de configuration/chargement initial ; ne pas supposer que tous les audits de production doivent toujours utiliser `init` ni qu'une exécution agrège automatiquement `init` et hors-`init`.
- **[IMPORTANT]** « branche fonctionnelle » passée au KSH avec `-b` = objet/répertoire métier ; **ce n'est pas une branche Git**.

## 3. Anatomie de l'outillage d'audit

**[OBSERVÉ + DOCUMENTÉ]** Racine du module : `src/Tools/raw_vault_tests/`.

| Élément | Responsabilité technique |
|---|---|
| `engine.py` | Parsing des JSON Raw Vault, résolution tables/colonnes, contrôles PySpark, construction et écriture des audits Hive. |
| `controls.py` | Wrapper/point d'entrée CLI compatible avec jobs KSH et soumissions Spark ; importe le moteur en résolvant les chemins du projet. |
| `analyzer.py` | Analyse les spécifications de tests (CSV issu du tableau métier), construit des IDs, classe les contrôles et recherche des références dans le code. **Ce n'est pas le moteur d'exécution PySpark.** |
| `README.md` | Règles, paramètres CLI, exemples Spark, statuts et diagnostic YARN. |
| `shell/*RAW_VAULT_AUDIT*.KSH` | Prépare environnement, configuration et lancement Spark via le cadre applicatif. Nom exact variable selon installation. |
| `src/Tools/dashboard/app.py` | Dashboard Streamlit pour lancer/consulter les audits, selon accès et configuration. |
| `test/test_json_generation_tool/` | Tests pytest/PySpark documentés, notamment sur les contrôles de comptage et cas OK/KO. |

### Mécanique connue du moteur

1. Détection des JSON HUB/SAT/MAS/LINK présents pour un objet métier.
2. Résolution des tables physiques (`hrv_`, `srv_`, `mrv_`, `lrv_`) et des colonnes HK/Hashdiff suivant les conventions et métadonnées.
3. Lecture des tables Hive source et Raw Vault avec Spark.
4. Calcul de comptages, distincts, doublons et comparaisons de clés via des opérations de type **left anti join**.
5. Production de résultats `OK`, `KO` ou `ERROR` et, si activée, écriture des tables d'audit.
6. Réconciliation possible par `run_id` des événements, résultats et détails.

**[OBSERVÉ]** Un contrôle source↔HUB est identifié `SOURCE_BK_ABSENT_HUB` ; le code exclut les clés nulles dans l'une des fonctions observées et compare les ensembles de clés source/HUB avec une jointure `left_anti`. Le compte obtenu représente les clés/groupes absents selon l'implémentation, pas nécessairement le nombre de lignes métier perdues.

### Identifiants de contrôles connus

| Famille | IDs observés/documentés |
|---|---|
| Comptages HUB/SAT/MAS/LINK | `<COMPOSANT>_COUNT_GLOBAL`, `<COMPOSANT>_COUNT_DISTINCT`, `<COMPOSANT>_DUPLICATES` |
| BK source/HUB | `SOURCE_BK_ABSENT_HUB` |
| SAT/HUB | `SAT_HK_ABSENT_HUB` |
| LINK/HUB | `LINK_FK_ABSENT_HUB` |
| LINK/SAT | `LINK_HK_ABSENT_SAT`, `SAT_HK_ABSENT_LINK` |
| LINK/MAS | `LINK_HK_ABSENT_MAS`, `MAS_HK_ABSENT_LINK` |

**[DOCUMENTÉ]** Clés d'unicité des tests : HUB = HK ; LINK = HK LINK ; SAT = HK + Hashdiff ; MAS = HK + discriminants JSON + Hashdiff. **[VIGILANCE]** Une duplication de HK seule dans SAT/MAS peut être de l'historisation légitime ; valider le grain et les règles de versions avant de conclure à une anomalie.

## 4. Persistance et interprétation des audits

| Table | Sens |
|---|---|
| `AUDIT_TEST` | Référentiel/catalogue des contrôles détectés |
| `AUDIT_RUN` | Une trace par exécution et objet audité, avec `run_id`, objet, horodatages, état |
| `AUDIT_RESULT` | Un résultat par contrôle/table exécuté, avec `status`, `value`, éventuellement `error_message` |
| `AUDIT_DETAIL` | Clés en anomalie, groupes de doublons et détails selon contrôle |

- **`OK`** : règle respectée pendant cette exécution ; **`KO`** : anomalie détectée ; **`ERROR`** : contrôle techniquement impossible/échoué. Ne pas transformer `ERROR` en `KO`.
- Les résultats d'exécutions successives peuvent coexister ; filtrer **toujours** par `run_id` pour ne pas confondre plusieurs campagnes. Ne pas supposer que toute table est automatiquement purgée ou que la rétention est infinie.
- **[OBSERVÉ]** Le schéma des résultats inclut notamment `component`, `test_id`, `physical_table_name`, `status`, `value`, `error_message` ; les détails incluent `duplicate_count` et `key_values`.
- `--skip-audit-write` exécute les contrôles sans alimenter les tables d'audit Hive.
- **[DOCUMENTÉ]** Une anomalie KO ne bloque pas en elle-même l'exécution technique ; la correction/acceptation dépend de l'analyse fonctionnelle et de la gouvernance. Vérifier le code retour et les conventions spécifiques des jobs.

## 5. Premier audit DEV réellement observé — cas `instrument` (2026-11-09)

**[OBSERVÉ]** Une exécution a alimenté `db_dev_df6_travail.audit_run` et `audit_result` pour l'objet métier `instrument`, à partir d'environ **09:46** le **9 novembre 2026**. La requête de résultats liée à ce `run_id` a renvoyé **52 lignes de résultats**.

- Des comptages `COUNT_GLOBAL`, `COUNT_DISTINCT` et contrôles de doublons sont visibles sur des HUB, SAT et MAS.
- Deux lignes `SOURCE_BK_ABSENT_HUB` pour `hrv_instrument` sont visibles en `OK`, `value = 0` : cela prouve la réussite de ces comparaisons lors de cette exécution, pas une couverture exhaustive de tous les objets/sources.
- Plusieurs résultats `SAT_HK_ABSENT_HUB` sont **KO** : `srv_inst_fonds_cote` (4), `srv_inst_action` (7), `srv_inst_obligation` (19), `srv_inst_tcn` (49), `srv_inst_fcc` (1), `srv_inst_garantid` (1). **Somme affichée : 81 unités de contrôle**, pas nécessairement 81 HK globalement distinctes.
- Les contrôles de doublons visibles sont `OK`, mais cela ne remplace pas une vérification sur l'ensemble des lignes/règles attendues.
- Les 52 résultats ne démontrent **ni** que tous les contrôles du projet ont été testés, **ni** que les résultats KO proviennent d'un bug de l'outil.
- Le statut global de l'exécution doit être lu séparément dans `AUDIT_RUN` ; ne pas assimiler la présence de résultats à une validation fonctionnelle totale.

### Requêtes Hue/Impala de base (lecture seule)

```sql
SELECT run_id, object_name, started_at, finished_at, status
FROM db_dev_df6_travail.audit_run
WHERE object_name = 'instrument'
ORDER BY started_at DESC
LIMIT 5;

SELECT test_id, component, physical_table_name, status, value, error_message
FROM db_dev_df6_travail.audit_result
WHERE run_id = '<RUN_ID>'
ORDER BY component, test_id, physical_table_name;

SELECT test_id, physical_table_name, duplicate_count, key_values
FROM db_dev_df6_travail.audit_detail
WHERE run_id = '<RUN_ID>'
LIMIT 100;
```

**[OBSERVÉ]** Hue utilisait ici l'éditeur **Impala**. Toujours vérifier le moteur et la base active. Les métadonnées peuvent différer temporairement entre Hive/Impala ; rechercher la cause avant d'inférer une absence de données.

## 6. Commandes de lancement connues et statut de fiabilité

### KSH d'audit

**[OBSERVÉ]** Un lancement manuel depuis le clone DEV a été réalisé avec la forme :

```bash
cd ~/df6/shell
./<SCRIPT_AUDIT_PRESENT>.KSH -d <AAAAMMJJ> -t audit -c raw -b instrument
```

Le lancement préparait l'environnement TOOLS, signalait un mode **hors Control-M**, créait un log sous `logctm/`, puis exécutait une soumission Spark/YARN. Le nom local effectivement présent doit être vérifié avec `find shell -iname '*AUDIT*.KSH'` ; ne pas déduire le nom d'un script DEV à partir du nom donné pour PROD dans une documentation.

### Lancement direct Spark du README

**[DOCUMENTÉ]** Exemple de commande (à adapter après validation des chemins/base source et des paramètres du cluster) :

```bash
spark-submit src/Tools/raw_vault_tests/controls.py \
  --config-path parm/raw/init/instrument \
  --object-name instrument \
  --hive-database db_dev_df6_brute \
  --audit-database db_dev_df6_travail \
  --format text \
  --skip-audit-write
```

Le README documente également `--format json`, `--output`, des paramètres source, et l'écriture Hive par défaut en l'absence de `--skip-audit-write`. L'existence de la commande ne prouve pas que `spark-submit` direct charge les mêmes configurations cluster/Python que le KSH.

### Dashboard Streamlit

**[DOCUMENTÉ]** Pour le hors-production :

```bash
cd ~/df6
streamlit run src/Tools/dashboard/app.py --server.address 0.0.0.0 --server.port 8501
```

Pré-requis : dépendances installées, accès aux données d'audit, authentification Kerberos valide, accès réseau **autorisé**. `0.0.0.0` est une adresse d'écoute et non l'URL du navigateur ; préférer les mécanismes réseau/tunnel approuvés. La présence du dashboard ne garantit pas que les déclenchements soient disponibles dans chaque environnement.

## 7. Incidents réellement rencontrés : symptômes, causes probables et conduite à tenir

| Situation | Preuve observée | Diagnostic prudent | Vérification / action sûre |
|---|---|---|---|
| Nom de KSH recopié depuis la documentation | `Aucun fichier ou dossier de ce type` | Nom de fichier inadéquat pour le répertoire/environnement courant | `pwd; find shell -iname '*AUDIT*.KSH'`; utiliser le script réellement présent |
| Lancement KSH crée un log mais l'audit échoue | Préparation YARN puis `java.io.FileNotFoundException` sur un chemin `...site-packages/Tools/...` | Ressource locale attendue par Spark absente/inaccessible ou packaging/chemin incohérent ; nom final du fichier tronqué dans capture | Lire **la ligne complète** de l'exception et le `spark3-submit` effectif ; contrôler chemin et artefact ; pas de correction aveugle |
| Dans le même KSH, erreur préalable de génération CONF | `fg_gen_properties : erreur lors de la génération du fichier ...df6_audit_raw_instrument.conf` | Problème potentiellement distinct ; relation causale avec l'échec Spark **non démontrée** | Inspecter génération `.conf`, entrée, permissions, log complet, code retour |
| `spark-submit` direct | `Cannot run program "python": error=2` | L'interpréteur `python` est absent du PATH visible par le processus Spark | Vérifier `command -v python python3`, environnement virtuel, `PYSPARK_PYTHON`, `PYSPARK_DRIVER_PYTHON`, mode YARN ; éviter d'inventer un chemin |
| Tentative `pip3 install dataclasses` | `[Errno 28] Aucun espace disponible sur le périphérique` | Espace disque ou quota utilisateur saturé ; installation interrompue | `df -h`, `df -i`, `du -sh ~/.cache/pip` et règles de quota ; ne pas supprimer des fichiers partagés ni relancer les installations en boucle |
| Dépôt Git différent du ticket actif | `git switch` vers une autre branche feature | Risque de tester/modifier le mauvais état de code | `git status -sb`, `git branch --show-current`, revenir sur la bonne branche **après** sauvegarde des modifications locales |
| Version Python de runtime inattendue | KSH utilise package `0.0.21` alors que `0.0.22` et `0.0.24` semblent présents | La version est résolue par variables de lancement et `df6.env`, pas choisie automatiquement comme « dernière version » | Lire `parm/df6.env`, `pyproject.toml`, contenu de `bin/` et logs de lancement |

**[OBSERVÉ]** Un premier audit ultérieur a effectivement enregistré 52 résultats : cela prouve qu'au moins une exécution a réussi à atteindre Hive après les difficultés précédentes. **La correction exacte ayant résolu les erreurs précédentes n'est pas documentée** ; ne pas prétendre qu'un simple changement de version en constitue la preuve.

### Résolution des versions — attention majeure

**[OBSERVÉ]** Le KSH construit un point d'entrée sous une forme équivalente à :

```bash
${_YCDBIN}/${_YCICODE}-${PACKAGE_VERSION}_venv/lib/python${PYTHON_VERSION}/site-packages/Tools/raw_vault_tests/controls.py
```

**[OBSERVÉ]** Des affectations `PACKAGE_VERSION` figurent dans `parm/df6.env` : lecture de `pyproject.toml` dans un contexte de développement HOME, et logique de recherche d'artefacts installés dans un autre contexte. Les affectations ultérieures peuvent remplacer les premières selon les branches du script.

**Conséquence** : modifier `version` dans `pyproject.toml` ne construit ni n'installe automatiquement le package correspondant. Contrôler l'accord entre **branche Git**, **version TOML**, **venv**, **ZIP payload PySpark**, **répertoire Python**, **variables Spark** et **version effectivement exécutée**.

Diagnostics de lecture seule utiles :

```bash
pwd
 git status -sb
 grep -n 'version' pyproject.toml
 grep -n 'PACKAGE_VERSION' parm/df6.env
 ls -ld bin/*_venv 2>/dev/null
 ls -l bin/*pyspark_payload.zip 2>/dev/null
 env | grep -E 'PYSPARK|SPARK_HOME|PYTHONPATH'
```

## 8. Développement Git, CI/CD et livraison : séparation des étapes

- **[OBSERVÉ]** Dépôt GitLab `df6-data`, branches `feature/...`, `develop`, `release` et branches de production selon le workflow local. Une branche dédiée aux contrôles de qualité a été créée **à partir de `release`**, puis utilisée sur l'Edge DEV.
- **[DOCUMENTÉ/OBSERVÉ]** Une branche feature modifie le code versionné ; un package installé dans `bin/` peut rester ancien. `git fetch`/`git pull` n'équivalent ni à un build, ni à une livraison, ni à un lancement Spark.
- **[OBSERVÉ]** Pipeline : étapes de préparation, tests/coverage, audit de dépendances et secret detection, lint/format/Sonar, build wheel/package et livraison. La livraison DEV a été observée via un mécanisme de distribution dédié ; RECETTE suit un palier distinct.
- **[DOCUMENTÉ]** L'exploitation après livraison peut passer par Control-M et des scripts KSH. **[À CONFIRMER]** La fréquence effective des audits (quotidienne, mensuelle ou à la demande) et leurs dépendances exactes Control-M ; ne pas les déduire de la fréquence de chargement des objets Raw Vault.
- **[IMPORTANT]** La version applicative peut rester identique pour de nombreuses exécutions quotidiennes/mensuelles. Un nouvel audit produit un nouveau `run_id` ; un changement de code validé produit une nouvelle version/livraison selon les procédures locales.
- **[IMPORTANT]** La livraison d'un package, l'installation sur Edge, le succès d'un job Spark, les écritures Hive et la validation métier constituent **cinq preuves distinctes**.

## 9. Procédure de diagnostic recommandée pour l'assistant

Lorsqu'une exécution échoue :

1. Vérifier **serveur/environnement** (sans exposer son hostname), `pwd`, branche Git, objet métier `-b`, date et commande lancée.
2. Lire le **nom exact du KSH** réellement exécuté et ses options (`--help` ou code source si disponible).
3. Contrôler la version déclarée, la version packagée, le venv/payload effectivement référencés et la configuration `.conf` générée.
4. Lire les logs **shell (`logctm`)**, **application (`logapp`)**, puis logs YARN/Spark ; distinguer la première erreur de la cascade d'erreurs.
5. Vérifier le code retour du job et l'existence des fichiers/colonnes nécessaires ; ne pas assimiler `submitted` à `completed`.
6. Si un audit a tourné, chercher `AUDIT_RUN` et les `AUDIT_RESULT` du même `run_id`, puis les détails `AUDIT_DETAIL`.
7. En cas de `KO`, établir le grain d'unicité et la relation réellement déclarée avant de décider d'un correctif.
8. Ne proposer une modification de code qu'après avoir isolé l'écart ; préparer un scénario reproductible et un test de non-régression.

## 10. Points ouverts à vérifier dans le workspace actuel

- Version exacte de l'outil exécutée lors du premier audit abouti et correspondance avec la branche Git.
- Commande KSH exacte et résolution complète de la version dans `df6.env` pour chaque contexte (HOME / installation partagée).
- Parcours de création, chargement et conservation des tables d'audit en DEV et RECETTE ; droits et rétention.
- Correspondance exacte entre les cas de tests du fichier de spécification Excel/CSV et les IDs PySpark générés par le moteur.
- Couverture réelle de chaque objet métier, selon ses JSON et ses relations ; jeux de tests OK/KO/ERROR et conformité métier.
- Causes précises des KO SAT→HUB constatés sur `instrument` ; vérifier la couverture temporelle, les relations et les données source.
- Ordonnancement Control-M du batch d'audit : existant ou à créer, dépendances avec les chargements, périodicités effectives.
- Versions et configurations Streamlit dans l'environnement hors-production ; raccordement aux données d'audit.

## 11. Contrat de comportement attendu de l'assistant

- Commencer par examiner les **fichiers réels du workspace**, ne jamais inventer des tests manquants.
- Fournir des commandes **en lecture seule par défaut** et spécifier explicitement lorsqu'une commande écrit dans Hive, HDFS ou le dépôt.
- Produire une matrice `règle fonctionnelle → contrôle attendu → code trouvé → couverture observée → écart prouvé → action → validation`.
- Différencier confirmation documentaire, observation d'exécution et hypothèse.
- Respecter séparation DEV/RECETTE/PROD, droits, quotas et règles d'approbation.
- Ne pas rendre des tests automatiquement « verts » en ignorant des anomalies sans validation fonctionnelle.
- Ne jamais lancer de déploiement, de purge, de transformation destructive ou de modification de configuration partagée sans accord explicite.

---

### Provenance des faits

Consolidation des référentiels fournis : `ICDC_CONTEXTE_GLOBAL.md`, `01_ECOSYSTEME_ICDC_DF6.md`, `02_ARCHITECTURE_DATA.md`, `03_DATA_VAULT_MODELISATION.md`, `04_ENVIRONNEMENTS_HDFS.md`, `05_HIVE_IMPALA_HUE.md`, `06_DEPOT_GITLAB_DF6.md`, `07_CONFIGURATIONS_PATTERNS.md`, `08_EXECUTION_SPARK_KSH.md`, `09_GITFLOW_CICD_DEPLOIEMENT.md`, `10_TESTS_VALIDATION_DATA.md`, `11_ORGANISATION_WORKFLOW.md`, `12_GLOSSAIRE_PROCEDURES.md`, ainsi que les captures/code/logs partagés pendant l'analyse de l'outil d'audit. Le document a été dépersonnalisé ; certaines valeurs d'exemple restent propres au périmètre technique DF6.
