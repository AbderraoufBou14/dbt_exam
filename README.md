Ton analyse montre des écarts fonctionnels dans `SOURCE_BK_ABSENT_HUB`, notamment sur les clés composites, la normalisation, les valeurs par défaut et les filtres.

Je souhaite maintenant corriger ce contrôle en priorité.

Appuie-toi sur le code actuel, les configurations réelles et les patterns de chargement pour choisir l'implémentation adaptée.

Objectifs :
- Comparer les BK composites comme des clés complètes, et non champ par champ.
- Respecter les mappings, transformations, valeurs par défaut et filtres réellement utilisés lors du chargement.
- Préserver le fonctionnement des BK simples et des sources multiples.
- Ajouter des tests ciblés couvrant notamment les faux OK des clés composites, les NULL, les filtres et la non-régression.
- Ne pas modifier la granularité des résultats par source ni le périmètre inter-répertoires à cette étape.

Évite tout refactoring supplémentaire et toute modification des autres contrôles sans nécessité démontrée.

Implémente les corrections, lance les tests locaux et fournis un bilan synthétique des changements et des limites restantes.

Pas de commit, de push ni d'exécution distante.
