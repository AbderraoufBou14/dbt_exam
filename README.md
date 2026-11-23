Nous allons effectuer la validation d'intégration DEV du moteur de contrôles Raw Vault sur l'objet `instrument`, après exécution du KSH.

Prépare un **script SQL ad hoc unique**, directement copiable et exécutable dans Hue/Impala, pour vérifier indépendamment les résultats du moteur.

Appuie-toi sur le code actuel, les JSON de `parm/raw/instrument`, les schémas physiques et les règles des patterns de chargement. Ne suppose aucun nom de colonne ni aucune formule de calcul non vérifiée.

Le script doit couvrir uniquement les vérifications pertinentes pour cet objet :

1. **Exécution de l'audit** : retrouver le dernier `run_id` de `instrument`, vérifier les contrôles effectivement exécutés, les statuts OK/KO/ERROR et les valeurs enregistrées dans `AUDIT_RESULT`.
2. **Détails des anomalies** : consulter `AUDIT_DETAIL` pour les contrôles en KO et vérifier la cohérence avec les résultats agrégés.
3. **Contrôles de comptage** : recalculer les comptages et doublons pertinents directement sur les tables HUB/SAT/MAS.
4. **Intégrité SAT → HUB et MAS → HUB** : recalculer indépendamment les clés orphelines dans les tables réelles et comparer les volumes avec l'audit.
5. **SOURCE → HUB** : vérifier les BK provenant des différentes tables sources configurées, en respectant les mappings `cles_src`, les clés composites, les filtres et les valeurs par défaut.
6. **SOURCE → SAT et SOURCE → MAS** : vérifier la présence des enregistrements par identité et contenu, avec les BK/HK, discriminants et HDIFF reconstruits selon les règles effectives des loaders.

N'inclus les contrôles LINK ou LINK/SAT/MAS que si l'objet `instrument` possède effectivement les configurations et relations nécessaires dans son périmètre d'exécution.

### Consignes pour le script

- Produis **un seul bloc SQL**, organisé dans l'ordre logique de validation.
- Utilise des commentaires courts et simples avant chaque requête : objectif du contrôle et résultat attendu.
- Évite les requêtes redondantes et celles qui n'apportent pas de vérification indépendante.
- Privilégie les comparaisons de volumes et les anti-jointures réellement utiles ; ajoute une consultation détaillée seulement si nécessaire pour diagnostiquer les écarts.
- Les requêtes doivent être en lecture seule (`SELECT`, `WITH`, `UNION ALL`).
- Utilise les noms exacts des bases, tables et colonnes DEV.
- Pour les contrôles de contenu, respecte précisément les règles de calcul utilisées par les loaders ; ne remplace pas une comparaison métier par une simple comparaison de HK.
- Distingue clairement un KO métier attendu d'une erreur technique.
- Si une vérification n'est pas fiable ou pas réalisable en SQL Impala, ne l'invente pas : indique brièvement la limite en commentaire et poursuis les autres vérifications.
- Ne crée aucune table ni vue et ne modifie aucune donnée.

**Important :** je veux uniquement le SQL final dans un seul bloc copiable, sans rapport d'analyse, sans explications longues et sans multiplication de requêtes inutiles.

Ne modifie aucun fichier du projet, ne lance aucun traitement et n'effectue aucun commit ou push.
