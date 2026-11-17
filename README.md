## DF6DFM-75 — Développement Lot 1 : contrôle MAS → HUB

Nous passons maintenant à l'implémentation du premier contrôle fonctionnel manquant : `MAS_HK_ABSENT_HUB`.

**État Git :** nous sommes sur la branche du ticket 75 et le working tree est propre. Les deux documents de contexte sont déjà disponibles dans cette session.

### Objectif fonctionnel

Détecter les enregistrements MAS dont le rattachement au HUB parent est absent ou incohérent, selon la structure Data Vault et les configurations réellement utilisées dans le projet.

Nous avons décidé de prévoir une logique couvrant les différents cas : MAS mono-HUB, multi-HUB et avec CDK.

Attention : ne suppose pas que la HK du MAS est systématiquement identique à celle du HUB parent. Vérifie comment les clés sont réellement construites dans les patterns existants.

### Travail demandé

1. Examine uniquement les fonctions du moteur d'audit, les configurations JSON et les portions de patterns nécessaires à ce contrôle. Ne recommence pas une cartographie générale.
2. Identifie la stratégie de rattachement MAS → HUB applicable aux différentes configurations : clé parent explicite, reconstruction fiable ou autre mécanisme déjà présent dans le framework.
3. Implémente le contrôle en réutilisant autant que possible l'architecture et les fonctions existantes.
4. Prévois des tests unitaires ciblés couvrant les cas OK, KO, données vides, clés nulles et configurations non prises en charge.
5. Vérifie que les résultats respectent les conventions existantes : statuts, compteurs, détails d'anomalies et traçabilité.
6. Exécute les tests pertinents dans l'environnement disponible. Si certains tests nécessitent Spark/YARN ou un déploiement du package, indique-le sans lancer de traitement distant.

### Contraintes

- Ne modifie pas le point d'entrée `controls.py` ni ses interfaces publiques, sauf nécessité démontrée et préalablement signalée.
- Ne lance pas de refactoring général.
- Ne modifie aucune table Hive/HDFS ni donnée d'audit existante.
- Pas de nouvelles dépendances sans justification.
- Aucun commit, push, MR ou déploiement automatique.
- Si une catégorie de MAS ne permet pas un contrôle fiable avec les métadonnées disponibles, ne produis pas de faux résultat OK : identifie précisément le cas et signale-le.

### Résultat attendu

Effectue le développement et les tests du lot, puis présente brièvement :

- Les fichiers modifiés et les fonctions ajoutées.
- Les différentes catégories de MAS prises en charge et les limites éventuelles.
- Les tests exécutés et leurs résultats réels.
- Les points qui nécessitent encore une validation fonctionnelle.
- Le diff synthétique et une proposition de message de commit.

**Travaille de manière ciblée et économe en contexte. Ne produis pas de nouveau rapport exhaustif. Arrête-toi après ce premier lot pour que nous puissions le valider avant le commit.**
