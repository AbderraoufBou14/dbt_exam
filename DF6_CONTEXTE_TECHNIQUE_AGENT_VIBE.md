# DF6 — Contexte technique de référence pour un agent de développement

> **Nature** : complément autonome aux passations issues d'autres sessions. Ce document privilégie les invariants applicatifs, les mécanismes observés, les décisions techniques déjà prises et les points de vigilance. Il ne remplace **ni le code courant**, **ni les JSON actifs**, **ni les spécifications fonctionnelles**.
>
> **Périmètre** : architecture Data Vault sur Spark/Hadoop, outillage de contrôles automatisés, tests, exploitation technique, GitLab CI/CD et packaging. Aucun contexte personnel, nom de personne, numéro de ticket, nom de client, URL privée, nom d'hôte ou calendrier n'est nécessaire.
>
> **Niveau de preuve** : **[Confirmé]** = observé dans le code, les logs, les captures ou des documents du projet ; **[Évolution observée]** = annoncé par l'agent précédent et documenté par ses sorties de tests, à revalider dans le nouveau workspace ; **[À vérifier]** = ne pas inférer sans examen du code et des schémas physiques. Les chemins et conventions sont ceux des sources consultées ; leur existence dans un nouveau clone doit être vérifiée.
>
> **Confidentialité** : les conventions applicatives et noms de bases peuvent demeurer internes. Utiliser ce document uniquement dans les outils et environnements autorisés.

## 1. Vue fonctionnelle et architecture data

```text
Applications et tables sources, notamment SI3 / LAC
                  |
                  | configurations d'alimentation et de transformation
                  v
            PySpark / Spark SQL
           + bibliothèque de patterns Vault
                  |
                  v
      Raw Vault DF6 dans Hive/HDFS
     HUB hrv_*    LINK lrv_*
     SAT srv_*    MAS  mrv_*
                  |
                  v
       Business Vault / exposition
                  |
                  v
       Analytique et usages aval

       En parallèle, moteur de contrôles :
      JSON + tables source/Vault
                  |
                  v
     raw_vault_tests via KSH / Spark
                  |
                  v
AUDIT_TEST / AUDIT_RUN / AUDIT_RESULT / AUDIT_DETAIL
                  |
                  v
          Hue / Impala / dashboard
```

**[Confirmé]** Plateforme principalement on-premise Cloudera : serveur Edge Linux pour les scripts/packaging, HDFS pour les fichiers, métastore Hive pour les tables, Impala/Hive pour l'interrogation depuis Hue, Spark/PySpark sur YARN pour les traitements, KSH pour les lanceurs, GitLab pour le code et la CI/CD.

**Ne pas confondre :**
- Un **répertoire du clone Git** et un **répertoire applicatif installé** : des fichiers de noms identiques peuvent contenir des versions différentes.
- Un **fichier HDFS**, une **table/catalogue Hive** et le **moteur SQL sélectionné dans Hue**.
- Une **branche Git** `feature/...` et l'argument KSH `-b <objet_metier>`, qui sélectionne un répertoire fonctionnel de configurations.
- Le Raw Vault dans une base nommée `df6_brute` et une éventuelle zone amont d'ingestion « brute ».
- La **date métier**, la **date fonctionnelle** du JSON, la **date de chargement** et la **date de lancement**.

Bases rencontrées en DEV : `db_dev_si3` (sources amont), `db_dev_df6_brute` (Raw Vault), `db_dev_df6` (couches de transformation selon objets), `db_dev_df6_travail` (audit et travail). Les noms de bases d'autres environnements ne doivent pas être extrapolés sans vérification.

## 2. Modèle Data Vault : sens des clés et granularités

| Composant | Convention | Fonction | Précautions de contrôle |
|---|---|---|---|
| HUB | `hrv_*` | Identité d'une entité métier, BK et HK | Une BK peut être composite ; comparer des ensembles d'identités, pas forcément les volumes bruts de lignes |
| LINK | `lrv_*` | Relation entre plusieurs identités HUB, HK de relation et HK participantes | La HK LINK ne remplace pas les HK parentes ; CDK éventuelle |
| SAT | `srv_*` | Valeurs descriptives historisées d'un parent HUB ou LINK | Une HK répétée peut être légitime à travers plusieurs versions ; grain d'unicité à définir |
| MAS | `mrv_*` | Plusieurs occurrences simultanément actives, historisées par groupe | Discriminants, CDK et clés parentes doivent être distingués ; ne jamais comparer deux HK uniquement sur leur nom |

- **BK** : Business Key ; identité métier construite depuis des colonnes source, parfois avec valeurs par défaut, concaténation, ordre et normalisation.
- **HK** : Hash Key calculée sur les éléments d'identité définis par le pattern exécuté ; elle est comparable à une autre HK seulement si sa construction et son sens sont identiques.
- **HDIFF** / `hd_*` : hash de contenu descriptif servant à détecter les changements, **pas** une clé de rattachement.
- **CDK** (Child Dependent Key) : composant de clé dépendant du parent pouvant intervenir dans la construction de clés de relation ou de MAS ; **ne pas assimiler automatiquement une CDK au discriminant du MAS**.
- **Discriminant MAS** (`champ_discriminant`) : distingue des occurrences simultanément actives au sein d'un groupe parent.
- **`ldts`**, **`rsrc`**, **`dt_fonc`** : chargement technique, provenance et temps fonctionnel ; sens et disponibilité précis à confirmer par table.

**MAS v2** : les documents décrivent une historisation d'un **état complet d'occurrences pour une HK parent**. Si le nombre d'occurrences, un couple HK/discriminant ou un HDIFF change, le nouvel état du groupe peut être inséré en entier. Par conséquent, l'unicité et les réconciliations doivent respecter le **grain temporel et le groupe**, non la simple unicité d'une HK dans toute la table.

## 3. Source de vérité des transformations : JSON + patterns + schémas

Emplacements documentés :

```text
parm/raw/init/<objet_metier>/df6_raw_vault_hub.json
parm/raw/init/<objet_metier>/df6_raw_vault_link.json
parm/raw/init/<objet_metier>/df6_raw_vault_sat.json
parm/raw/init/<objet_metier>/df6_raw_vault_mas.json

parm/raw/<objet_metier>/df6_raw_vault_{hub,link,sat,mas}.json
```

Un objet métier n'a pas nécessairement les quatre familles. Un fichier contient souvent une propriété `tables` avec **plusieurs définitions de destination**. Les structures varient selon le composant et la version du pattern : ne pas supposer que `cles_src` a la même forme à chaque profondeur du JSON.

Champs rencontrés et rôle :
- `appli_source`, `suffixe_base_source`, `src_table_name` : résolution de l'amont ; des sources non-SI3 peuvent exister.
- `dest_table_name`, `tables` : cibles et regroupement des configurations.
- `cles_src`, `src_bk_name`, `target_bk_name`, `bk_default_value` : BK et transformations de clés, selon la famille.
- `hubs` : références aux identités parentes, potentiellement plusieurs pour un MAS ou LINK.
- `cdk`, `champ_discriminant` : composants de relation et granularité multi-active.
- `payload`, `filtre`, `dt_fonc`, `schema_partition` : contenu, sous-population et temporalité.

**Chaîne d'interprétation obligatoire** :

```text
JSON effectivement sélectionné
  -> code de résolution de la source / du parent
  -> pattern de création/chargement réellement appelé
  -> fonctions partagées de normalisation et hashing
  -> schéma physique Hive/HDFS
  -> grain et algorithme de comparaison du contrôle
```

Points d'entrée de chargement/création dans les documents : `src/raw/create/main.py` et `src/raw/load/main.py`. La dépendance Vault a été explorée sous `site-packages/icdcpatternvault`, notamment `load/load_multi_active_satellites.py`, `load/load_satellite.py`, et les fonctions communes sous `configuration/commons/`. **Vérifier la version réellement installée** : la présence d'une implémentation v2 dans la documentation n'implique pas que tous les objets utilisent cette version.

**INIT vs récurrent** : le lanceur sélectionne des configurations `parm/raw/init/` ou `parm/raw/` selon le mode. Les suffixes `_delta`, `_delta_histo` ou autres ne suffisent pas à déduire une équivalence de populations, une sémantique « FULL » ou la fréquence effective du flux. Ne jamais comparer naïvement un chargement historique complet avec une source incrémentale sans reconstruire le périmètre attendu.

## 4. Cartographie utile du code de contrôle

| Chemin | Rôle connu | Sensibilité |
|---|---|---|
| `src/Tools/raw_vault_tests/engine.py` | Lecture des JSON, modèles internes, exécution des contrôles PySpark, persistance des audits ; historiquement volumineux | Forte : modifications localisées, revue des jointures et actions Spark |
| `src/Tools/raw_vault_tests/controls.py` | Point d'entrée CLI Python invoqué par le KSH/Spark | **Contrat de compatibilité à préserver** ; éviter changement de signature/import |
| `src/Tools/raw_vault_tests/__init__.py` | Exports du package | Garder la compatibilité avec les imports existants |
| `src/Tools/raw_vault_tests/analyzer.py` | Analyse des spécifications de tests et identifiants, pas le moteur Spark | Ne pas confondre avec `engine.py` |
| `src/Tools/raw_vault_tests/README.md` | CLI, exécution, conventions de statuts, modes de sortie | Confronter au code effectif |
| `src/Tools/dashboard/app.py` | Consultation/affichage des audits et éventuellement lancement selon habilitations | Contrat de lecture `AUDIT_*` à préserver |
| `test/test_json_generation_tool/test_raw_vault_counting_controls.py` | Tests unitaires et locaux du moteur | Réutiliser fixtures et helpers réels, couvrir les régressions |
| `test/test_json_generation_tool/test_rawVaultTestsAnalyzer.py` | Tests de l'analyseur | Conserver en non-régression |
| `shell/*RAW_VAULT_AUDIT*.KSH` | Lanceur batch de l'audit | Vérifier le nom réellement présent et la sélection de version |
| `parm/df6.env`, `pyproject.toml` | Variables/version de package | Risque de divergence entre code Git et package exécuté |

**Contrat logique** : détection des composants configurés → chargement des tables/parquet via Spark → calcul de métriques et anti-jointures → production `OK`/`KO`/`ERROR` → écritures Hive éventuelles → consultation via Hue/Impala/dashboard.

## 5. Contrôles préexistants et logique de validation

Contrôles rencontrés :

| Catégorie | Identifiants connus |
|---|---|
| Comptage / doublons | `<FAMILLE>_COUNT_GLOBAL`, `<FAMILLE>_COUNT_DISTINCT`, `<FAMILLE>_DUPLICATES` |
| Source → HUB | `SOURCE_BK_ABSENT_HUB` |
| SAT → HUB | `SAT_HK_ABSENT_HUB` |
| LINK → HUB | `LINK_FK_ABSENT_HUB` |
| LINK ↔ SAT | `LINK_HK_ABSENT_SAT`, `SAT_HK_ABSENT_LINK` |
| LINK ↔ MAS | `LINK_HK_ABSENT_MAS`, `MAS_HK_ABSENT_LINK` |
| MAS → HUB | `MAS_HK_ABSENT_HUB` — évolution récente, voir section suivante |

`left_anti` est une technique fréquemment employée pour identifier les clés présentes dans un enfant et absentes de son parent. La **direction** de l'anti-jointure fait partie de la règle métier. Dans les tests de comptage/distinct/doublon, préciser explicitement le **grain** (lignes physiques, HK distinctes, HK + HDIFF, HK + discriminants + HDIFF, etc.).

Dans le contrôle de BK source/HUB, plusieurs sources peuvent produire des BK de formes différentes mappées sur une même cible. Exemple observé pour `instrument` : sources `lac_pvr_instrument_delta_histo.id` et `lac_pvo_inventaires_quot_dern_vers_instrument.codevaleur` vers `hrv_instrument.id_instrument`. Des tests source/HUB séparés ont été observés. Une spécification de réconciliation mentionne par ailleurs une **union des BK de plusieurs sources** : ne pas remplacer une implémentation par l'autre sans examiner la sémantique et les attendus réels. 

**États** :
- `OK` : aucune anomalie trouvée dans le périmètre réellement contrôlé.
- `KO` : une anomalie de donnée a été trouvée pour la règle ; ne signifie pas à lui seul que le programme a échoué.
- `ERROR` : exécution/contrôle techniquement impossible ou invalide ; ne doit pas devenir un faux `OK` pour faire passer la suite.

Un résultat `OK` n'atteste pas qu'une configuration a été détectée ; un « skip » silencieux peut masquer des relations non couvertes. Les `NULL` filtrés d'un contrôle doivent être explicitement pris en charge ailleurs ou documentés.

## 6. Évolution récente : contrôle MAS → HUB, état et limites

**[Évolution observée — revalider sur les fichiers transférés]** Un développement ciblé a ajouté une logique de contrôle `MAS_HK_ABSENT_HUB` dans `engine.py` et des tests locaux, sans modifier `controls.py` ; des exports additionnels ont été faits dans `__init__.py`. Les éléments nommés dans les rapports de développement comprennent `MasHubReference`, `load_mas_hub_references`, une branche de traitement dans `execute_counting_controls`, et la présence de `MAS_HK_ABSENT_HUB` dans les identifiants de détail.

Constat sur un périmètre de **22 configurations MAS examinées**, à l'instant de l'étude :

1. **20 MAS mono-HUB à CDK vide** : la construction des patterns étudiés permettrait `hk_mrv == hk_hrv` dans cette configuration. Le contrôle peut rechercher directement les MAS sans HK HUB (`left_anti`), avec `OK`/`KO` et détails.
2. **2 MAS non directement comparables** : MAS multi-HUB identifiés dans les configurations de notation. Le code développé annoncerait une **vérification indirecte MAS → LINK → HUB**, à condition de trouver un LINK réellement compatible avec les mêmes HUB/CDK et une HK de relation correspondante. Cela doit valider à la fois la présence de la relation MAS dans le LINK **et** les HK parentes du LINK dans chaque HUB ; `OK`/`KO` avec détails si la relation est bien établie.
3. **MAS multi-HUB/CDK sans LINK exploitable** : pas de mécanisme de comparaison directe sûr si les entrées de hash ne sont pas présentes dans la table MAS ; la décision implémentée est un `ERROR` explicite, plutôt qu'un faux `OK`. Aucun cas réel de ce type n'a été trouvé dans les configurations examinées. **Une conception permettant de traiter réellement un futur cas CDK sans LINK reste une limite**, et n'est pas garantie par ce développement.
4. **HUB parent absent du répertoire JSON local** : une résolution de la table physique du HUB a été ajoutée pour éviter un « skip » silencieux, avec au moins une relation concernée dans les configurations de position. Les règles de choix du HUB doivent demeurer déterministes.

**Vérifications locales rapportées** : 17 tests MAS ciblés réussis, 35 tests réussis sur la suite moteur, 7 tests analyseur exécutés sous Python 3.9 dans un examen antérieur. Un des tests de l'analyseur a des annotations problématiques sous Python 3.8, alors que le projet vise Python ≥3.10 ; ne pas confondre incompatibilité de l'interpréteur local et défaut introduit par le développement. Les environnements de test rapportés utilisaient un PySpark local, **pas une campagne end-to-end Spark/YARN avec le nouveau package livré**.

**Réserves techniques explicites du retour de développement :**
- `_load_link_hub_table` conserve un fallback par préfixe : risque théorique de choisir une mauvaise table en présence de noms composés/préfixes similaires ; aucune collision observée dans les configurations examinées.
- Lorsqu'un MAS non comparable pourrait correspondre à **plusieurs LINK**, l'algorithme signalé prendrait le **premier dans l'ordre JSON** ; aucun cas rencontré alors, mais une ambiguïté future ne doit pas être résolue arbitrairement.
- Dans la boucle SAT, **deux relations** pour un objet de lien d'identité restaient ignorées silencieusement par le code préexistant ; problème indépendant à ne pas oublier lors du refactoring/contrôle de couverture.
- Un `ERROR` peut donner un **code retour non nul** au programme/KSH : c'est un signal d'échec technique, pas une anomalie métier détectée.
- Le fait qu'un test unitaire produise `OK` ne prouve pas que le join trouve la bonne population en Hive ; le contrôle doit être confronté à des SQL indépendants, des configurations réelles et la bonne version packagée.

**Important :** aucune affirmation ici ne garantit que ces changements sont déjà commités, poussés ou installés dans le nouvel environnement. Vérifier `git status`, `git log`, `git diff`, et les tests dans le **workspace présent** avant de décrire l'état comme livré.

## 7. Autres contrôles fonctionnels à produire ou vérifier

Au-delà de la relation MAS/HUB, cinq **familles de réconciliations source ↔ Raw Vault** ont été identifiées comme absentes ou incomplètes dans l'analyse précédente :

1. **Source ↔ SAT** : vérifier identités et/ou contenu descriptif chargé, y compris temporalité/HDIFF selon spécification.
2. **Source ↔ MAS** : contrôler les groupes multi-actifs, discriminants, contenu et périmètre historique.
3. **Source ↔ LINK** : reconstruire les identités de relation et vérifier les clés/parents correspondants.
4. **Source ↔ SAT rattaché à LINK** : contrôler le satellite de relation et ses composantes sources.
5. **Source ↔ MAS rattaché à LINK** : contrôler la structure multi-active portée par une relation et son grain.

**Ces cinq étiquettes sont des familles de besoins, pas des identifiants `test_id` imposés ni une preuve que cinq fonctions Python nouvelles suffisent.** Le document de spécification et les JSON/patterns doivent fixer pour chaque cas : la population source, le mapping BK/HK, le contenu HDIFF, les filtres, le mode INIT/récurrent, le sens des écarts, la granularité, la temporalité et la conduite à tenir si la configuration manque.

Pour chaque nouveau contrôle, établir une mini-fiche :

```text
Règle fonctionnelle source -> cible
Sources et cible(s) réelles
Transformations à reproduire (mapping, filtre, valeur BK par défaut, concat/hashing)
Clés et grain de comparaison (y compris temporel)
Sens du test : éléments absents source->cible, cible->source, ou les deux
Résultat : métrique / OK / KO / ERROR / détails
Scénarios : vide, NULL, doublons, multi-source, transformations, cas non supporté
Contre-vérification SQL indépendante
```

La couverture source/HUB existante peut inspirer la structure logicielle, mais elle n'est **pas** forcément une réconciliation suffisante pour les SAT/MAS/LINK, dont le contenu et l'historisation diffèrent.

## 8. Persistance des audits et stratégie de partitionnement

Quatre tables d'audit existent dans la zone de travail (à vérifier par environnement) :

| Table | Finalité |
|---|---|
| `audit_test` | Catalogue des contrôles connus/configurés |
| `audit_run` | Exécution (identité `run_id`, objet, horodatages, état global) |
| `audit_result` | Résultat par contrôle/table, état `OK`/`KO`/`ERROR`, valeur/message |
| `audit_detail` | Anomalies concrètes : identifiants, clés, compte de duplication, détails |

Des exécutions précédentes ont alimenté de l'historique partagé : **ne pas supprimer, recréer ou vider les tables actuelles**. Ne pas supposer que relancer l'audit permettrait de reproduire l'historique original, les `run_id`, les données sources et les résultats à une date passée.

**Orientation technique retenue** : partitionner quotidiennement **`audit_run`, `audit_result`, `audit_detail`**, pas `audit_test`, via une colonne **`partition_date STRING` au format `yyyy-MM-dd`**, sous réserve de compatibilité avec les DDL/Hive/Impala du workspace.

**Mécanisme de migration non destructif recommandé :**

1. Sauvegarder les schémas, formats, propriétés, `LOCATION` HDFS, comptages et `run_id` des tables d'origine ; identifier les éventuels écrivains concurrents.
2. Créer des **tables V2** à noms distincts, avec partition de date ; aucune modification destructrice de l'existant.
3. Pour chaque ancien `run_id`, déterminer une date de partition de référence **depuis `audit_run`** (idéalement `started_at` si correctement renseigné et timezone/documentation cohérente). Propager la même date aux lignes `audit_result` et `audit_detail` par `run_id`.
4. Effectuer un **backfill** par SQL/Spark avec insertion explicite des colonnes et de la partition cible ; ne pas utiliser de `SELECT *` aveugle, vérifier les éventuels doublons de jointure, valeurs orphelines et `NULL` de dates.
5. Comparer comptes globaux et par `run_id`, ensembles de clés, statuts, compteurs, messages, détails ; vérifier l'absence de lignes dupliquées/perdues et les requêtes Hue/Impala.
6. Tester un nouveau `run_id` écrit dans les V2, avec cohérence intertables, lecture du dashboard et compatibilité Impala.
7. **Seulement après accord technique** : préparer bascule de noms/configuration des écrivains et lecteurs, absence d'écritures concurrentes, sauvegarde et retour arrière. `CREATE TABLE IF NOT EXISTS` ne change pas une table existante en table partitionnée.

La présence d'une colonne de partition ne garantit pas un gain : les requêtes de consommation doivent filtrer dessus pour activer le **partition pruning**. Déterminer si le format `STRING` est celui utilisé par les outils existants avant de figer les DDL.

## 9. Mesures et optimisation PySpark

Plusieurs pistes ont été repérées dans l'ancien moteur `engine.py` : répétition de `.count()`, relectures de tables, actions déclenchées en série, joins et matérialisation de détails. Ce sont **des hypothèses d'optimisation**, pas une preuve de lenteur ni un bénéfice quantifié.

Méthode convenue :

1. Mesurer une **baseline** sur les mêmes objets, mêmes données, même configuration Spark/YARN et version de package connue ; relever durée, stages/jobs, volumes et coût des actions principales.
2. Examiner le plan logique/physique (`explain`) des opérations coûteuses. Identifier clairement les `.count()`, réutilisations de DataFrames et échanges shuffle.
3. N'introduire `persist()`/`unpersist()` que sur des DataFrames réellement réutilisés, en tenant compte de la mémoire cluster et des coûts de matérialisation.
4. Ne pas changer aveuglément le broadcast. Un KSH observé désactive le broadcast automatique (`spark.sql.autoBroadcastJoinThreshold=-1`) ; tout changement nécessite une mesure, une justification et le respect du contexte mémoire/exploitation.
5. Comparer avant/après la **durée** ET les **résultats fonctionnels** (nombre de contrôles, valeurs, détails, codes retour). Un job plus rapide qui oublie des contrôles est une régression.

Pas d'optimisation massive ni de refonte des appels Spark tant que les nouveaux contrôles et la non-régression ne sont pas stabilisés.

## 10. Tests et validations : quatre niveaux distincts

1. **Tests unitaires / PySpark local** : données minimales, `OK/KO/ERROR`, `NULL`, source vide, doublons, clés composites, CDK, multi-HUB, tables manquantes, ambiguïtés et écritures simulées. Éviter des fixtures énormes ou des tests locaux interprétés comme preuve d'exécution YARN.
2. **Non-régression** : les contrôles historiques doivent émettre les mêmes tests/valeurs/statuts sur les mêmes entrées, hors changements explicitement voulus. Préserver le CLI, `controls.py`, les imports et le format des audits.
3. **Intégration DEV** : source validée → wheel/package réellement construit → installation de la bonne version → KSH Spark/YARN → `run_id` + résultats/détails dans Hive → contre-vérification SQL Hue/Impala.
4. **Validation sur un environnement aval** : version effectivement promue, même code et configs visés, nouveaux audits enregistrés et résultats interprétés indépendamment du succès de la pipeline.

Quelques observations de tests sur de **vraies tables DEV** avant les nouvelles modifications : un audit sur l'objet `instrument` a généré **52 lignes** dans `audit_result` ; les deux vérifications BK source/HUB renvoyaient `OK/0`, avec contre-requêtes SQL concordantes. Un contrôle SAT/HUB sur un satellite d'instrument renvoyait `KO/7`, confirmé par `left_anti` indépendant ; un autre SAT/HUB renvoyait `OK/0`. Des contrôles LINK/MAS bidirectionnels étudiés ont aussi été confirmés à `OK/0`. Ces exemples prouvent que le moteur et les contrôles antérieurs ont fonctionné sur des données réelles, **pas** que le nouveau contrôle MAS/HUB ou les contrôles source sont déjà validés en DEV.

**Conseil de lecture Hue** : sélectionner le bon moteur, la bonne base, le bon environnement ; filtrer `audit_run` et les deux tables filles par **le même `run_id`** ; ne pas confondre `KO` de contrôle avec un échec de lancement, ni `finished` avec couverture exhaustive.

## 11. Chaîne d'exécution KSH / version effectivement utilisée

Les traces/documentations décrivent une chaîne du type :

```text
Lanceur KSH de la famille Raw Vault Audit
  -> sélection de l'environnement / des options et JSON
  -> génération ou lecture d'un fichier .conf
  -> résolution de PACKAGE_VERSION et PYTHON_VERSION
  -> préparation venv, payloads et fichiers Spark
  -> spark3submit / spark3-submit sur YARN (mode cluster)
  -> module controls.py installé dans bin/<package>_venv
  -> contrôles PySpark -> Hive/HDFS
  -> logs KSH, application, YARN et AUDIT_*
```

Exemple d'appel observé dans un environnement de développement ; **ne pas l'exécuter sans autorisation et vérification du chemin réel** :

```bash
cd shell
./DDDF6AJR1_RAW_VAULT_AUDIT.KSH -d 20261109 -t audit -c raw -b instrument
```

Le KSH peut rechercher un `controls.py` dans un environnement versionné, avec une forme analogue à :

```text
bin/df6-0.0.N_venv/lib/pythonX.Y/site-packages/Tools/raw_vault_tests/controls.py
bin/df6-0.0.N_pyspark_payload.zip
```

**Piège majeur** : le clone Git contient ton nouveau `engine.py`, mais le KSH lance une **copie packagée plus ancienne** si sa résolution de version pointe sur un autre artefact. `pyproject.toml`, `parm/df6.env`, les répertoires `bin`, l'archive payload et les logs du KSH doivent correspondre. Changer seulement une version déclarée ne construit pas un package et ne met pas à jour un runtime.

Diagnostic rapide, sans effet sur les données :

```bash
git status -sb
git branch --show-current
git log -3 --oneline
rg -n 'PACKAGE_VERSION|PYTHON_VERSION' parm/df6.env pyproject.toml
find bin -maxdepth 1 -type d -name '*_venv' 2>/dev/null
find bin -maxdepth 1 -type f -name '*payload.zip' 2>/dev/null
```

**[Confirmé]** Des échecs antérieurs ont été causés ou accompagnés par des problèmes de génération `.conf`, ressources de packaging introuvables sur Spark/YARN, interpréteur `python` absent d'un contexte de lancement direct, ou espace disque saturé lors d'installation. Ne jamais attribuer une erreur YARN à la logique fonctionnelle avant d'avoir isolé la première exception pertinente.

## 12. GitLab / CI-CD / livraison : ne pas confondre les états

Chaîne observée sur le projet :

```text
Clone/branche Git + commit
        |
       push
        v
GitLab pipeline (jobs conditionnels)
   setup -> pytest/coverage/sécurité -> qualité/lint/Sonar
              -> wheel -> packaging .tar.gz
        |
        v
  livraison DEV via job Fusion (artefact -> SAS -> Fusion)
        |
        v
  installation applicative sur Edge DEV
        |
        v
  lancement KSH puis Spark/YARN
        |
        v
  validation des données Hive/Hue
```

Jobs effectivement vus : `dx_py_ensure_venv`, `dx_py_generate_variables_env`, `dx_py_pytest_coverage`, `dx_py_lint`, `dx_py_sonar`, `dx_py_build_wheel_artifact`, `dx_py_build_icdc_packaging`, `dx_fusion_delivery` ; peuvent exister également `secret_detection`, audits de dépendances et autres jobs conditionnels. **La liste et les règles réelles sont celles du `.gitlab-ci.yml` et des includes en vigueur.**

Le build produit au moins deux artefacts distincts : wheel `df6-0.0.N-py3-none-any.whl` et package applicatif `df6-0.0.N.tar.gz`. Une livraison réussie peut transférer ce dernier par SAS puis Fusion, et installer `bin/`, `parm/`, `shell/` dans l'environnement visé. Il a été observé des promotions séparées vers d'autres environnements, parfois manuelles ; ne pas les déclencher implicitement.

**Rappels Git pertinents pour une session de travail :**
- `git fetch origin` actualise les références distantes ; ne merge pas, ne déploie pas et ne modifie pas les fichiers du working tree.
- `git merge origin/release`, sur une branche feature, **intègre localement** la référence de release fetchée. Cela peut modifier d'autres fichiers que `parm/raw/` (ex. `pyproject.toml`, code commun) ; vérifier l'état Git et les éventuels conflits avant/après.
- Un `git commit` est local. Un **push** rend les commits visibles sur GitLab et peut déclencher une pipeline selon ses règles. Un merge purement local n'ouvre aucune MR et n'appelle aucun runner.
- Une branche à jour avec `origin/feature/...` n'est pas forcément à jour avec `origin/release` ; distinguer tracking/upstream et base commune.
- Une livraison applicative réussie **n'implique pas** qu'un KSH ait été exécuté, qu'un audit ait été écrit ni qu'un résultat métier soit `OK`.

**Ne pas présumer qu'Artifactory est obligatoirement dans le chemin de livraison observé** : les logs consultés montrent des artefacts de jobs GitLab, un `.tar.gz`, un SAS puis Fusion ; d'autres moyens de distribution peuvent exister.

## 13. Modifications structurelles et refactoring

Le moteur historique concentrait de nombreuses responsabilités dans `engine.py`. Une cible de modularisation raisonnable a été envisagée : `models.py`, `config_loader.py`, `control_logic.py`, `audit_store.py`, avec `engine.py` allégé et `controls.py` **inchangé en tant qu'interface externe**. Ce sont des **pistes**, pas des noms de modules obligatoires.

La stratégie retenue est **incrémentale** : extraire seulement les responsabilités utiles, réduire les duplications avérées, garder les imports/CLI/contrats d'audit, exécuter les tests après chaque étape, puis vérifier le packaging. Éviter de déplacer des centaines de lignes pour une simple question d'esthétique ou d'introduire des abstractions qui complexifient la validation.

Les évolutions de contrôle fonctionnel, le refactoring, le partitionnement et les optimisations de performance doivent rester **logiquement séparables**, avec tests et commits cohérents.

## 14. Comportement attendu du nouvel agent

**Ordre de confiance :** (1) code et configuration présents dans le workspace ; (2) package réellement exécuté et schémas/data en environnement ; (3) spécifications/README/documents ; (4) rapports de passation ; (5) hypothèses. En cas de divergence, la signaler, ne pas réécrire silencieusement le besoin.

**Au démarrage :** lire les documents de passation déjà fournis, puis vérifier rapidement la branche, les fichiers effectivement présents et les tests pertinents. Éviter de recommencer une cartographie exhaustive, de parcourir intégralement `bin/` ou `site-packages`, ou de réécrire les rapports.

**Pendant chaque lot :** examiner uniquement les fichiers nécessaires, comprendre les transformations, annoncer en peu de lignes la stratégie, développer avec modifications ciblées, lancer les tests disponibles, présenter le diff et les limites. Préserver les comportements documentés et ne pas inventer les noms des colonnes HK ou des JSON.

**Actions avec effet de bord :** ne pas exécuter sans validation explicite : `git commit`/`push`, MR, pipeline de livraison, installation, KSH distant, création ou mutation Hive/HDFS, purge d'audit, migration ou bascule de tables partagées.

**Exigence de qualité :** distinguer un test qui n'a pas été exécuté, un test au statut `KO`, et un test `ERROR`. Les résultats « passés » de pytest prouvent les scénarios locaux, pas la conformité des données réelles. Toujours conserver la possibilité de reconstituer la validation par `run_id`, version de package, objet, environnement et schéma.

**Format de retour souhaité** : bref tableau « fonction/fichier modifié — justification — tests réellement exécutés — risques restants — prochaine vérification ». Pour les écarts, fournir le chemin et la fonction précis. Ne pas générer de longs rapports ni refaire des investigations générales sans demande.

---

## 15. Repères de vérification dans les autres documents

- `CONTEXTE_APPLICATIF_DF6.md` : architecture SI3/LAC → DF6, structures JSON et relations Data Vault.
- `02_ARCHITECTURE_DATA.md` et `03_DATA_VAULT_MODELISATION.md` : circulation des données et granularité/temporalité Data Vault.
- `06_DEPOT_GITLAB_DF6.md` et `07_CONFIGURATIONS_PATTERNS.md` : dépôt/runtime, configuration et patterns.
- `08_EXECUTION_SPARK_KSH.md` : lanceurs, packaging/version, logs et séparation Edge/YARN.
- `09_GITFLOW_CICD_DEPLOIEMENT.md` : règles GitFlow documentées, pipeline, artefacts et Fusion.
- `10_TESTS_VALIDATION_DATA.md` : niveaux de validation, contrôles SQL et limites d'interprétation.

Ces références sont informatives ; le nouvel agent n'a pas besoin de les charger intégralement s'il dispose déjà des deux passations et du présent complément. Les éventuelles questions non résolues doivent être tranchées par les fichiers et les preuves du workspace actuel.
