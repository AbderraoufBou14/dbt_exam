Nous avons confirmé que les contrôles Source → SAT/MAS/LINK-SAT/LINK-MAS recherchent le tuple (HK, discriminants éventuels, HDIFF) dans tout l'historique Vault.

Je souhaite maintenant trancher un seul point : **comment choisir la bonne version Vault pour expliquer les colonnes payload divergentes lorsqu'un tuple source est KO ?**

Analyse uniquement les éléments nécessaires dans les loaders réels, les JSON et le moteur d'audit.

1. Examine précisément `dt_fonc`, `dt_deb_val`, `ldts`, les colonnes temporelles des sources `_delta_histo` et le rôle de `is_fonc`. Peut-on réellement rapprocher une version source d'une version Vault de façon fiable ?
2. Étudie les cas de plusieurs versions pour un même HK (+ discriminants), corrections rétroactives, dates fonctionnelles identiques, HDIFF différents et absence de date exploitable.
3. Compare trois stratégies : dernier `ldts`, `dt_deb_val` le plus proche de la date source, ou autre règle plus fidèle au loader. Indique les cas où chacune donne un diagnostic trompeur.
4. Précise si une comparaison des payloads peut être considérée comme certaine ou seulement indicative, et comment détecter les cas où aucune référence fiable n'existe.
5. Propose une règle de décision simple et déterministe applicable aux quatre familles, avec des exemples concrets et les informations à restituer dans `AUDIT_DETAIL_V2.key_values`.

Ne refais pas l'analyse générale des 13 contrôles, des schémas V2 ni du partitionnement.

**Aucune modification de code, aucun test Hive/YARN, aucun commit.** Je veux un bilan factuel, avec les preuves dans le code, les limites et une recommandation argumentée.
