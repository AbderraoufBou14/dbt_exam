Ton analyse d'impact est validée.

Implémente maintenant uniquement l'ajustement de `uniqueness_columns` pour le contrôle `SOURCE_BK_ABSENT_HUB`.

Objectif : remplacer `hub_bk_columns` par `source_key_columns` dans la métadonnée `uniqueness_columns`, afin d'afficher les colonnes BK source.

Exemples attendus :
- Instrument, source PVR : `["id"]`
- Instrument, source inventaire : `["codevaleur"]`
- Position avec BK composite : liste complète des colonnes BK source, dans l'ordre des mappings.

Consignes :
- Ne modifier ni les calculs des BK composites, ni l'anti-jointure, ni les regroupements, ni le comptage des anomalies.
- Adapter les assertions des tests existants et ajouter un test ciblé si nécessaire.
- Exécuter les suites de tests de non-régression (counting et analyzer).
- Vérifier que `AUDIT_RESULT` contient la nouvelle métadonnée et que `AUDIT_DETAIL` conserve son comportement.
- Ne pas modifier le schéma Hive.
- Signaler clairement la limitation actuelle d'`AUDIT_TEST` pour les HUB multisources, sans chercher à la corriger dans ce lot.
- Ne pas traiter `source_table_name`, le partitionnement ni le cas limite de comptage NULL/default dans cette modification.

Fournis un bilan concis des fichiers modifiés et des tests exécutés.

Aucun commit, push ni déploiement.
