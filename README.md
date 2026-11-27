Nous poursuivons l'étude du moteur d'audit Raw Vault DF6 avant de décider du mécanisme de diagnostic des divergences HDIFF.

Le contexte fonctionnel est important : DF6 fonctionne avec des alimentations INIT et récurrentes, des sources `_delta` / `_delta_histo`, des traitements quotidiens, intraday et mensuels, orchestrés via Control-M.

Notre audit compte actuellement 13 contrôles. Les tables AUDIT_*_V2 sont opérationnelles. Le problème des discriminants est corrigé.

Nous avons déjà analysé les règles de hash, les versions SAT/MAS et la faisabilité de comparer les payloads. **Ne refais pas cette analyse générale.**

Je veux maintenant comprendre comment les cycles de chargement influencent la pertinence des contrôles Source → SAT/MAS/LINK-SAT/LINK-MAS.

Analyse le code DF6, les configurations JSON, les loaders accessibles et les KSH.

### 1. Périmètre réellement contrôlé

- Comment les sources `_delta` et `_delta_histo` sont-elles utilisées par les loaders et l'audit ?
- Quel rôle jouent INIT, récurrent, les éventuels suffixes `_histo` et les filtres JSON ?
- La date `-d` du KSH influence-t-elle réellement le périmètre de données contrôlé ou sert-elle uniquement au contexte d'exécution ?
- L'audit compare-t-il toute la source disponible, un snapshot, une période ou les données d'un chargement précis ?
- Dans quelles conditions source et Vault peuvent-ils représenter des périmètres temporels différents ?

### 2. Scénarios fonctionnels concrets

Étudie trois objets réels et représentatifs dans les configurations accessibles, idéalement :
- `instrument`, avec ses différentes sources ;
- un objet `position`, potentiellement concerné par plusieurs périodicités ;
- un objet avec MAS ou LINK-MAS et historisation significative.

Pour chacun, explique ce que vérifierait notre audit après un INIT, un chargement récurrent ou une reprise, selon ce que permet réellement la configuration.

Considère aussi les risques liés aux chargements intraday, quotidiens et mensuels. Ne présume pas connaître les dépendances Control-M exactes si elles ne sont pas accessibles.

### 3. Impact sur les KO et les HDIFF

Analyse particulièrement ces situations :

- Source mise à jour mais chargement Vault pas encore terminé.
- Ligne source historique dont le HDIFF correspond à une ancienne version du satellite.
- Nouvelle correction source produisant un HDIFF déjà historisé.
- Chargement partiel ou reprise d'un traitement.
- Source `_delta_histo` contenant plusieurs états successifs d'une même HK.

Pour chaque situation, indique si notre contrôle actuel retourne OK ou KO, si le résultat est fonctionnellement pertinent et quels risques de faux OK ou faux KO existent.

Distingue bien un écart de périmètre temporel d'une véritable divergence des attributs métier.

### 4. Conséquences pour notre futur diagnostic

Réponds précisément :

- La comparaison avec le dernier `ldts` est-elle suffisante pour identifier les colonnes divergentes ?
- Dans quels cas cette comparaison peut-elle être trompeuse ?
- Peut-on identifier un état Vault de référence plus pertinent avec les informations déjà disponibles ?
- Quelles informations manquent éventuellement à notre moteur pour produire un diagnostic fiable ?
- Faut-il enrichir seulement le diagnostic ou envisager une évolution de certaines règles de contrôle ? Ne propose de modifier une règle que si un problème concret est démontré.

### 5. Bilan attendu

Je veux une analyse factuelle, appuyée sur des fonctions et des JSON réels, comprenant :

1. Le périmètre effectivement contrôlé aujourd'hui.
2. Un tableau des scénarios à risque, avec le résultat attendu.
3. Les limites fonctionnelles démontrées du moteur actuel.
4. Les implications pour la comparaison des payloads et le diagnostic HDIFF.
5. Ta recommandation pour obtenir un moteur fiable en recette et en exploitation.

Distingue systématiquement ce qui est confirmé par le code de ce qui reste hypothétique.

**Analyse uniquement. Aucune modification, aucun test réel Hive, aucun commit. Pas de dashboard ni de refonte du moteur.**
