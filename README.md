Nous allons maintenant préparer le développement du ticket DF6DFM-75 à partir de ton rapport d'audit technique.

Ton rapport constitue notre base de travail. Avant de développer, je souhaite approfondir plusieurs points critiques et recueillir tes recommandations techniques.

Tu disposes d'un accès direct au code et aux dépendances du projet : utilise-le pour vérifier tes hypothèses et proposer les solutions les plus adaptées, sans te limiter aux pistes que je mentionne.

### 1. Construction des clés et contrôle MAS → HUB

Explore les traitements Raw Vault réels pour comprendre la construction des clés SAT et MAS, notamment dans la bibliothèque `icdcpatternvault`, les fonctions techniques partagées et les configurations JSON.

Identifie :
- Les HK propres aux MAS et SAT.
- Les clés représentant leurs HUB parents.
- Les règles de calcul et de rattachement réellement utilisées.
- La manière correcte d'implémenter `MAS_HK_ABSENT_HUB`.

Appuie tes conclusions sur le code et les configurations, avec les chemins et fonctions concernés.

Utilise notamment l'objet métier `couru` comme exemple concret.

### 2. Partitionnement des tables d'audit

Nous devons conserver l'historique actuel de `AUDIT_RUN`, `AUDIT_RESULT` et `AUDIT_DETAIL`, qui contient des exécutions réelles des audits de comptage et de qualité.

Étudie les différentes stratégies possibles pour passer à des tables partitionnées par date d'exécution.

Évalue en particulier une migration non destructive avec tables intermédiaires, reprise des données existantes, validation et bascule contrôlée.

Vérifie la cohérence de la date de partition entre les trois tables pour un même `run_id`, la compatibilité Spark/Hive/Impala et les impacts sur le dashboard.

Recommande la solution la plus sûre et la plus simple à maintenir dans notre environnement.

### 3. Compatibilité des points d'entrée

L'actuel `controls.py` est utilisé comme point d'entrée Spark par le KSH.

Réévalue ta proposition de modularisation pour minimiser les changements d'interfaces et préserver le lancement, le packaging et les imports existants.

Je privilégie une architecture claire mais sans sur-modularisation.

### 4. Stratégie de tests

Propose une stratégie de validation comprenant :
- Tests unitaires des nouvelles fonctions.
- Tests des cas OK, KO et erreurs.
- Tests de non-régression après refactoring.
- Tests d'intégration avec les tables d'audit.
- Validation sur les objets métier en DEV puis en RECETTE.
- Mesures avant/après pour les optimisations Spark pertinentes.

Réutilise autant que possible les tests existants.

### 5. Revalidation du rapport

Vérifie la branche Git active et explique pourquoi le rapport mentionne `feature/DF6DFM-547_Hub_Flux_Financiers`, alors que l'analyse était destinée au ticket DF6DFM-75.

Revérifie également la matrice des contrôles : elle annonce cinq contrôles qualité manquants, alors que six lignes paraissent signalées comme manquantes.

### Résultat attendu

Je souhaite un complément d'analyse concis, avec :
- Tes conclusions vérifiées dans le code.
- Les risques et points d'incertitude.
- Tes recommandations argumentées.
- Les éventuelles décisions nécessitant validation humaine.
- Une proposition actualisée des premières étapes de développement.

Notre objectif est de livrer un travail fiable, propre et maintenable en RECETTE d'ici le 20 novembre 2026, sans refactoring disproportionné ni optimisation non mesurée.

Pour cette étape, reste en lecture seule : aucune modification de code, aucune création/suppression de table et aucune exécution avec effet sur les données.
