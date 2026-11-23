Avant de poursuivre, je souhaite une vérification ciblée de deux points dans le contrôle `SOURCE_BK_ABSENT_HUB` :

1. **Déduplication des références** : dans `config_loader.py`, vérifie si `reference_key` peut fusionner à tort deux références ayant les mêmes tables et mappings, mais des valeurs différentes de `filtre` ou `bk_default_value`. Vérifie si ce cas existe dans les configurations réelles.

2. **Concaténation des BK composites** : dans `control_logic.py`, vérifie si l'utilisation de `SOURCE_KEY_DELIMITER` avec `F.concat_ws()` peut provoquer des collisions entre deux combinaisons de BK différentes. Examine notamment le séparateur, les NULL et les valeurs métier possibles.

Pour chaque point, indique :
- Le comportement exact du code actuel.
- Si le risque est réel, théorique ou déjà couvert.
- Les configurations concernées, le cas échéant.
- Si une correction est nécessaire, avec une justification courte.

**Analyse uniquement ces deux points. Aucune modification de code, aucun nouveau test, aucune exécution Spark et aucun commit. Fournis simplement un bilan synthétique et factuel.**
