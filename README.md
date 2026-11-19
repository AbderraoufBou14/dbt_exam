Avant de poursuivre le développement des contrôles manquants, nous souhaitons refactorer le moteur d'audit Raw Vault pour améliorer sa modularité, sa lisibilité et sa maintenabilité.

Tu disposes du code complet et du contexte technique. Analyse la structure actuelle et **propose un plan de refactoring adapté à l'existant**, en privilégiant la simplicité, la séparation des responsabilités et la facilité d'ajout de nouveaux contrôles.

Je te laisse identifier les problèmes structurels, choisir l'architecture cible et définir les étapes de refactoring les plus pertinentes.

Présente un plan synthétique comprenant :
- Les problèmes identifiés dans l'architecture actuelle.
- Le découpage proposé et les responsabilités des modules.
- Les étapes d'implémentation et leur ordre.
- Les impacts potentiels sur les interfaces, le packaging et les performances.
- La stratégie de tests et de non-régression.

**Contraintes :** préserver le comportement fonctionnel, les contrôles existants, la compatibilité KSH/Spark et la persistance des audits. Éviter toute sur-ingénierie.

**Pour le moment, aucune modification de code, aucun test ni commit.** Nous allons d'abord examiner ton plan et l'ajuster si nécessaire.

Réponse concise, basée sur le code réel, sans nouvelle exploration exhaustive.
