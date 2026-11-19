## Refactoring du moteur de contrôles Raw Vault

Nous avons terminé une première implémentation du contrôle `MAS_HK_ABSENT_HUB`. Avant de développer les cinq contrôles fonctionnels restants, nous souhaitons améliorer la structure du moteur existant.

Tu disposes du code complet et des documents de contexte. **Je te laisse analyser et choisir l'approche de refactoring la plus pertinente**, en fonction des responsabilités réelles, des dépendances et des conventions du projet.

### Objectifs

- Améliorer la modularité, la lisibilité et la maintenabilité.
- Clarifier la séparation des responsabilités et réduire les duplications.
- Faciliter l'ajout des futurs contrôles sans complexifier davantage le moteur.
- Préserver les performances et éviter la sur-ingénierie.

### Contraintes

- Refactoring à comportement fonctionnel constant : conserver les résultats, statuts, compteurs et détails d'audit.
- Préserver les interfaces existantes, notamment le point d'entrée `controls.py`, les appels KSH, le packaging et la persistance Hive.
- Conserver le fonctionnement du contrôle MAS/HUB récemment ajouté.
- Privilégier des modifications progressives, cohérentes et testables.
- Ne pas développer les cinq contrôles restants à cette étape.
- Ne pas modifier les tables Hive ni lancer de traitements distants.
- Ne pas effectuer de commit ou push automatiquement.

### Méthode

Commence par identifier brièvement les principaux problèmes structurels et propose un découpage raisonnable. **Ne considère pas les propositions des handovers comme une architecture imposée.**

Ensuite, réalise le refactoring par lots cohérents, avec des tests de non-régression après chaque lot.

À la fin, présente :
1. Les choix d'architecture et leur justification.
2. Les fichiers créés ou modifiés et leurs responsabilités.
3. Les tests réellement exécutés et leurs résultats.
4. Les éventuels risques ou points restant à valider.
5. Une proposition de découpage en commits.

Évite de produire une nouvelle analyse exhaustive. L'objectif est d'obtenir un moteur plus clair, stable et facilement extensible, **sans changement de comportement métier**.
