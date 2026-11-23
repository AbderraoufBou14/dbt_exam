Les contrôles Source → Raw Vault étant maintenant implémentés, je souhaite une revue ciblée de la qualité du code avant de poursuivre.

Analyse les modules actuels de `raw_vault_tests`, principalement `control_logic.py`, pour identifier les redondances significatives, fonctions inutilement longues et simplifications réellement utiles.

Privilégie la lisibilité, la maintenabilité et la réduction du code dupliqué, sans complexifier l'architecture ni multiplier les modules.

Pour chaque amélioration, indique brièvement :
- Le problème constaté et les fonctions concernées.
- La simplification envisageable.
- Le gain attendu et le risque de régression.
- La priorité : utile maintenant ou reportable.

Ne remets pas en cause les règles fonctionnelles des contrôles ni les interfaces KSH/Spark.

Fournis un plan de simplification court, limité aux changements pertinents.

**Analyse uniquement : aucune modification de fichier, aucun nouveau test, aucun commit ni push.**
