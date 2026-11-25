Nous poursuivons l'analyse du contrôle `SOURCE_BK_ABSENT_HUB`, particulièrement la cohérence du champ `uniqueness_columns` dans les tables d'audit.

Sur l'objet `instrument`, les sources utilisent `id` et `codevaleur` (`src_bk_name`), tandis que le HUB utilise `bk_id_instrument` (`target_bk_name`). Actuellement, `uniqueness_columns` présente les colonnes cibles, ce qui soulève une question sur sa signification et sa lisibilité.

**Décision déjà prise :** nous conservons un résultat de contrôle par table source, sans faire d'UNION des sources, afin de faciliter la traçabilité des anomalies. Ce point n'est plus à arbitrer.

### Analyse demandée

Examine de manière ciblée le code actuel (`models.py`, `config_loader.py`, `control_logic.py`, `audit_store.py` et les composants concernés) pour déterminer :

1. Quelle est la signification réelle de `uniqueness_columns` dans le moteur, depuis sa construction jusqu'à son écriture dans `audit_result` et `audit_detail` ?
2. Comment ce champ est-il renseigné pour `SOURCE_BK_ABSENT_HUB` et pour les autres familles de contrôles ? Sa signification est-elle uniforme ?
3. Pour `instrument`, est-il techniquement et fonctionnellement cohérent d'afficher `target_bk_name` plutôt que `src_bk_name` ? Quels avantages et limites présente chaque option ?
4. Comment distingue-t-on les résultats des différentes tables sources d'un même HUB dans `audit_result` et `audit_detail` ? La traçabilité est-elle suffisante actuellement ?
5. L'ajustement de présentation que tu proposais dans `audit_result` résout-il réellement le problème ? A-t-il des conséquences sur les autres contrôles, le schéma Hive ou le dashboard ?

### Retour attendu

Fournis un bilan factuel et synthétique comprenant :
- Le fonctionnement actuel, avec les fonctions et champs concernés.
- La distinction entre logique de calcul, identification des anomalies et métadonnées de restitution.
- Les éventuelles incohérences démontrées.
- Deux ou trois options de correction ou d'amélioration, avec leurs avantages, limites et impacts.
- Ta recommandation argumentée, en privilégiant la solution la plus simple et la moins risquée.

**Important : aucune modification de code, aucun test, aucune opération Hive, aucun commit.**

Ne relance pas une analyse générale du dépôt. Nous voulons d'abord comprendre le comportement et choisir une solution avant toute implémentation.
