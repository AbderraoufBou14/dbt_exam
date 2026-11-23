Nous poursuivons la mise en conformité des contrôles source avec la spécification fonctionnelle Excel.

Commence par le contrôle SOURCE ↔ SAT uniquement.

Exigence Excel : comparer les couples (BK HUB, HDIFF SAT) avec (BK source, HDIFF source recalculé à partir du payload), afin d'identifier les enregistrements présents dans la source mais absents du satellite.

Appuie-toi sur le code actuel, les JSON et les patterns de chargement pour déterminer les bons mappings, transformations et calculs HDIFF.

Adapte le contrôle existant en réutilisant au maximum le code déjà développé. Respecte les conventions d'audit, les interfaces KSH/Spark et l'architecture modulaire.

Ajoute les tests ciblés OK/KO/ERROR et vérifie la non-régression des contrôles existants.

Ne modifie pas les quatre autres contrôles source pour l'instant. Pas de refactoring supplémentaire, pas de traitement distant, pas de commit ni de push.

À la fin, fournis un bilan court : logique implémentée, fichiers modifiés, tests réussis et éventuelles limites fonctionnelles.
