Ton analyse des redondances est pertinente. Nous validons le principe d'une simplification ciblée du moteur.

Tu peux maintenant implémenter les améliorations réellement utiles de ton plan, en privilégiant la factorisation des traitements communs et la lisibilité de `execute_counting_controls`.

Quelques orientations :
- Conserver l'architecture actuelle et ne créer aucun nouveau module.
- Mutualiser les traitements identiques sans masquer les particularités fonctionnelles des contrôles.
- Éviter les abstractions trop génériques ou les fonctions avec trop de paramètres.
- Conserver strictement les résultats, statuts, messages d'erreur, détails d'audit, test_id et interfaces KSH/Spark.
- Procéder progressivement avec tests de non-régression après chaque étape.
- Ne pas intégrer d'optimisation Spark ni de partitionnement dans ce lot.

Tu peux adapter ton plan si tu identifies une solution plus simple dans le code.

À la fin, fournis un bilan concis : fonctions simplifiées, lignes économisées, fichiers modifiés, résultats des tests et risques résiduels.

Pas de commit, push, packaging ou exécution distante.
