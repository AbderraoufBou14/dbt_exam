Nous reprenons le chantier `SOURCE_BK_ABSENT_HUB`, après une revue fonctionnelle avec l'équipe.

**Décisions déjà validées :**
- Un contrôle par source, sans UNION.
- Anti-jointure Source → HUB, sens inchangé.
- Calcul des BK composites conservé.
- Nous souhaitons que `AUDIT_RESULT.uniqueness_columns` contienne les colonnes BK source (`source_key_columns`) et non les colonnes BK du HUB (`hub_bk_columns`).

Exemple `instrument` :
- Source `lac_pvr_instrument_delta` : `["id"]`
- Source `lac_pvo_inventaires_quot_dern_vers_instrument` : `["codevaleur"]`
- Actuellement, les deux résultats présentent `["bk_id_instrument"]`.

**Travail demandé : analyse et vérification uniquement, sans correction pour le moment.**

### 1. Analyse d'impact de `uniqueness_columns`

En t'appuyant sur le code actuel, vérifie les conséquences d'un remplacement de :

`source_hub_reference.hub_bk_columns`

par :

`source_hub_reference.source_key_columns`

dans le contrôle `SOURCE_BK_ABSENT_HUB`.

Examine uniquement les composants concernés :
- `control_logic.py` et `config_loader.py`
- Les écritures dans `AUDIT_TEST`, `AUDIT_RESULT` et `AUDIT_DETAIL`
- Les tests unitaires existants
- Le dashboard `analyzer.py`

Vérifie notamment si ce changement affecterait les calculs, les comparaisons, les détails d'anomalies, les regroupements ou l'affichage.

### 2. Vérification du comptage des BK normalisées

Effectue un test unitaire PySpark ciblé pour vérifier ce cas limite :

- `bk_default_value = "-1"`
- Source contenant `NULL` et `"-1"`
- HUB sans la BK correspondante

Détermine si les deux valeurs produisent la même `composite_bk` et si le moteur compte une ou deux anomalies.

Vérifie également la cohérence entre `AUDIT_RESULT.value` et les enregistrements générés dans `AUDIT_DETAIL`.

Signale si le même problème peut concerner les BK composites.

### 3. Retour attendu

Fournis un rapport court et factuel avec :

1. Les impacts réels du changement de `uniqueness_columns`.
2. Les éventuelles incompatibilités identifiées.
3. Le résultat du test PySpark : valeurs normalisées, nombre d'anomalies et détails.
4. Ta recommandation pour une correction minimale et sécurisée.

**Contraintes :**
- Ne modifie pas le code métier ni les schémas Hive.
- Tu peux exécuter un test unitaire isolé, sans conserver de fichier temporaire dans Git.
- Ne traite pas encore l'ajout de `source_table_name` ni le partitionnement.
- Aucun commit, push, packaging ou déploiement.

Nous déciderons de l'implémentation après ton retour.
