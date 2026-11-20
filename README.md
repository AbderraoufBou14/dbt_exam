Merci pour ton plan. Nous souhaitons privilégier une architecture plus simple, adaptée à la taille du moteur et aux futurs contrôles.

Une alternative envisagée précédemment était de conserver seulement quatre modules spécialisés : `models.py`, `config_loader.py`, `control_logic.py` et `audit_store.py`, avec `engine.py` comme façade et `controls.py` comme point d'entrée inchangé.

Compare brièvement cette approche avec ton plan actuel à cinq modules. Détermine si `spark_ops.py` est réellement nécessaire ou si ses fonctions peuvent être intégrées proprement ailleurs, sans surcharger les modules.

Tu disposes du code complet : choisis le découpage le plus pertinent, sans chercher à multiplier les fichiers.

**Après cette courte évaluation, passe directement à l'implémentation du refactoring**, progressivement, en respectant ces principes :

- Préserver les comportements et les interfaces existantes, notamment KSH/Spark, packaging, imports et résultats d'audit.
- Commencer par déplacer les responsabilités sans modifier la logique fonctionnelle.
- Exécuter les tests de non-régression après chaque étape.
- Éviter les changements d'optimisation ou de logique métier pendant ce refactoring.
- Ne pas développer les nouveaux contrôles à cette étape.
- Ne pas effectuer de commit ni de push automatiquement.

Tu peux réaliser l'ensemble du refactoring en plusieurs étapes internes, sans demander une validation après chaque extraction.

À la fin, présente un bilan synthétique : architecture retenue, fichiers modifiés, tests exécutés, éventuelles réserves et propositions de commits.

Si tu rencontres une décision présentant un risque réel de rupture de compatibilité ou de changement fonctionnel, arrête-toi sur ce point et signale-le.

Évite les nouvelles investigations exhaustives et les longs rapports intermédiaires. Notre objectif est maintenant d'obtenir un refactoring fonctionnel et testé.
