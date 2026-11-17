Le développement du contrôle MAS_HK_ABSENT_HUB me paraît prêt pour une validation finale.

Avant le premier commit, effectue une revue rapide et ciblée des trois points suivants :

1. Vérifie que le LINK utilisé pour les MAS multi-HUB correspond exactement aux clés et relations métier déclarées dans les configurations, sans correspondance ambiguë.
2. Vérifie que la résolution des HUB hors répertoire de configuration sélectionne de façon déterministe la bonne table physique.
3. Vérifie qu'aucun cas non contrôlé n'est silencieusement considéré comme OK.

Relance uniquement les tests pertinents et présente le résultat, les fichiers modifiés et les éventuelles réserves.

Ne modifie le code que si une anomalie concrète est identifiée. Pas de refactoring supplémentaire ni de nouvelle exploration générale.

Ne crée pas encore de commit : je le ferai après validation du diff.
