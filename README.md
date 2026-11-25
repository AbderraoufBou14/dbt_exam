Lors de la dernière vérification de `SOURCE_BK_ABSENT_HUB`, tu as indiqué que le script temporaire `/tmp` de test bout en bout échouait à cause d'une assertion trop stricte, et non du moteur.

Je veux vérifier précisément ce point avant de clôturer le chantier.

1. Retrouve le script temporaire concerné et identifie l'assertion qui échoue.
2. Explique brièvement pourquoi elle échoue : erreur dans l'assertion, résultat inattendu ou problème du moteur.
3. Corrige uniquement l'assertion si elle est effectivement incorrecte.
4. Relance le test complet avec Spark en mémoire et `write_audit_tables`.
5. Vérifie que les deux sources d'`instrument` produisent bien `["id"]` et `["codevaleur"]` dans `AUDIT_RESULT`, avec des valeurs et détails cohérents.

**Retour attendu :** assertion initiale, cause exacte, correction effectuée et résultat final PASS/FAIL.

Ne modifie aucun fichier métier ni test versionné. Aucun commit, push ou déploiement. Si le script temporaire n'existe plus, indique-le plutôt que de prétendre l'avoir relancé.
