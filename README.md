Je souhaite vérifier la conformité de `SOURCE_BK_ABSENT_HUB` concernant les mappings `cles_src` et les HUB multi-sources.

Examine le code actuel de `config_loader.py` et `control_logic.py`, puis confronte-le à ces configurations réelles :

- `instrument` : plusieurs sources avec `id` ou `codevaleur` → `id_instrument`.
- `portefeuille` : sources DF5/SI3 avec noms de BK différents.
- `position_mens`, `position_quot`, `position_quot_caceis` : même HUB `position`, BK composite de 7 champs et mappings différents.
- `flux_financiers` : plusieurs répertoires sources avec `xx_date_param` → `dtvalorisation`.

Vérifie précisément :
1. La gestion des mappings `src_bk_name` → `target_bk_name`, y compris les BK composites et leur ordre.
2. La normalisation des valeurs, `bk_default_value` et les filtres.
3. Le traitement de plusieurs sources d'un même HUB, y compris lorsqu'elles sont déclarées dans différents répertoires.
4. La conformité avec la spécification Excel demandant l'union des BK des sources d'un HUB.
5. Le comportement des champs particuliers comme `xx_date_param`.

Pour chaque point, indique : déjà implémenté, partiel ou absent, avec la fonction concernée et la conséquence fonctionnelle.

Pas de modification de code ni d'exécution distante. Réponse synthétique, sans exploration générale du dépôt.
