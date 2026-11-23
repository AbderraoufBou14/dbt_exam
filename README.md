Nous poursuivons la mise en conformité des contrôles fonctionnels Source → Raw Vault.

Les contrôles SOURCE → HUB et SOURCE → SAT ont déjà été améliorés. Le refactoring modulaire est terminé. Nous souhaitons maintenant **implémenter les quatre contrôles restants dans un même lot**, sans nouvelle phase d'analyse générale.

### Périmètre fonctionnel — spécification Excel

1. **SOURCE → MAS** : comparer (BK du ou des HUB parents + discriminant + HDIFF MAS) avec (BK source + discriminant + HDIFF recalculé depuis le payload source).

2. **SOURCE → LINK** : reconstruire les BK métiers à travers les HUB associés au LINK, puis comparer les relations présentes dans la source à celles du Raw Vault, conformément aux jointures et anti-jointures prévues par l'Excel.

3. **SOURCE → LINK/SAT** : reconstruire les BK via HUB + LINK, puis comparer les couples (BK, HDIFF satellite) avec ceux de la source, HDIFF recalculé depuis le payload.

4. **SOURCE → LINK/MAS** : reconstruire les BK via HUB + LINK, puis comparer (BK, discriminant, HDIFF MAS) avec les données sources correspondantes.

### Principes d'implémentation

- Utilise le code actuel, les JSON réels et les patterns de chargement comme références techniques définitives.
- Réutilise les mécanismes existants pour les BK composites, HK, CDK, discriminants, filtres, valeurs par défaut et HDIFF.
- Respecte les règles de construction des identités et les mappings source/vault, y compris pour les relations multi-HUB.
- Préserve les conventions `AUDIT_TEST`, `AUDIT_RUN`, `AUDIT_RESULT`, `AUDIT_DETAIL` et les statuts OK/KO/ERROR.
- Ne retourne jamais un faux OK lorsqu'une configuration ne permet pas une comparaison fiable : restitue explicitement l'impossibilité de contrôle.
- Préserve les interfaces existantes, `controls.py`, le lanceur KSH et le packaging.
- Ne modifie pas les contrôles déjà finalisés, sauf correction indispensable et justifiée.

### Exécution et validation

Développe les quatre fonctionnalités successivement, en réutilisant les composants communs lorsque c'est pertinent.

Pour chaque contrôle :
- Ajouter des tests représentatifs OK/KO/ERROR, notamment sur les différences de contenu, les clés composites, les discriminants et les relations multi-HUB.
- Vérifier la non-régression des contrôles existants.
- Éviter les lectures et calculs Spark inutiles sans introduire d'optimisation générale hors périmètre.

L'objectif est une implémentation fonctionnelle cohérente avec la spécification Excel, pas une simple vérification de présence des HK.

### Contraintes

Pas de nouveau refactoring architectural, pas de partitionnement des audits, pas de changement global des paramètres Spark.

Pas de commit, push, packaging ou exécution distante.

Travaille directement sur les fonctionnalités. Si une ambiguïté de configuration bloque un cas particulier, documente précisément cette limite et poursuis les autres cas fiables, sans inventer de règle métier.

### Bilan attendu

À la fin, fournis un rapport synthétique :
- État des quatre contrôles : implémenté, partiel ou bloqué.
- Logique de comparaison effectivement réalisée.
- Fichiers modifiés et tests réussis.
- Écarts éventuels avec l'Excel.
- Points nécessitant une validation d'intégration DEV.

Ne considère pas les tests PySpark locaux comme une validation d'intégration.
