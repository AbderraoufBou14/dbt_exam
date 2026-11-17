# DF6 — Dossier Control-M
## Complément du HUB Flux Financiers — DF6DFM-547

**Version :** Brouillon V1 — Revue technique à effectuer  
**Périmètre :** 5 MAS Raw Vault — 6 jobs Control-M proposés  
**Famille :** `XDDF6AJR25`

---

## Fiche 1 — XDDF6AJR25_RESULTAT_CESSION_QUOT

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Résultat de cession quotidien. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eod_delta` |
| | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eod_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_resultat_cession_quot` |
| **Périodicité** | Quotidien — EOD (calendrier à confirmer) |
| **Condition de déclenchement** | FIN du job `XDSI3AJL21_RESCESEOD` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Exécution exclusive des autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b resultat_cession_quot false` |

---

## Fiche 2 — XDDF6AJR25_RESULTAT_CESSION_INTRA

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation intraday de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Résultat de cession quotidien. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eod_delta` |
| | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eod_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_resultat_cession_quot` |
| **Périodicité** | Intraday — calendrier à confirmer |
| **Condition de déclenchement** | FIN du job `XDSI3AJL21RESCES_INTRA` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Ne pas exécuter simultanément avec le job EOD ni les autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b resultat_cession_quot false` |

**Remarque :** les fiches 1 et 2 utilisent le même groupe `-b` et alimentent le même MAS, comme les jobs EOD et INTRA du premier lot. L'orthographe du nom SI3 INTRA est à revérifier dans Control-M.

---

## Fiche 3 — XDDF6AJR25_RESULTAT_CESSION_MENS

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Résultat de cession mensuel. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eom_delta` |
| | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_eom_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_resultat_cession_mens` |
| **Périodicité** | Mensuel — calendrier à confirmer |
| **Condition de déclenchement** | FIN du job `XDSI3AJL22_RESCESEOM` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Exécution exclusive des autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b resultat_cession_mens false` |

---

## Fiche 4 — XDDF6AJR25_RESULTAT_CESSION_FIFO_INTRA

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Résultat de cession FIFO quotidien. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_fifo_eod_delta` |
| | `db_prod_si3.lac_pvo_pdt_fi_resultat_cession_fifo_eod_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_resultat_cession_fifo_quot` |
| **Périodicité** | Intraday — calendrier à confirmer |
| **Condition de déclenchement** | FIN du job `XDSI3AJL23_RESCESFIFO_INTRA` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Exécution exclusive des autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b resultat_cession_fifo_quot false` |

**Remarque :** la fiche SI3 disponible concerne `RE​SCESFIFO_INTRA`, bien que la source porte le suffixe EOD et que la périodicité inscrite dans le document soit Mensuel. Un éventuel job FIFO EOD distinct n'a pas encore été identifié.

---

## Fiche 5 — XDDF6AJR25_PRODUITS_FI_YTD_DOPE_QUOT

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Produits FI YTD DOPE quotidien. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_produits_fi_ytd_dope_quot_delta` |
| | `db_prod_si3.lac_pvo_produits_fi_ytd_dope_quot_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_produits_fi_ytd_dope_quot` |
| **Périodicité** | Quotidien — calendrier à confirmer |
| **Condition de déclenchement** | FIN du job `XDSI3AJL24_PRDFINYTDQUOT` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Exécution exclusive des autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b produits_fi_ytd_dope_quot false` |

**Remarque :** la fiche SI3 du job `PRDFINYTDQUOT` indique Mensuel, malgré son suffixe QUOT. La périodicité proposée doit être confirmée.

---

## Fiche 6 — XDDF6AJR25_PRODUITS_FI_YTD_DOPE_MENS

| Rubrique | Description |
|---|---|
| **Script** | `XDDF6AJR0_RAW_VAULT.KSH` |
| **Description** | Alimentation de la zone RAW de DF6, avec le chargement du HUB Flux Financiers et du MAS Produits FI YTD DOPE mensuel. |
| **Tables sources (Hive)** | `db_prod_si3.lac_pvo_produits_fi_ytd_dope_mens_delta` |
| | `db_prod_si3.lac_pvo_produits_fi_ytd_dope_mens_delta_histo` |
| **Tables alimentées** | `db_prod_df6_brute.hrv_flux_financiers` |
| | `db_prod_df6_brute.mrv_produits_fi_ytd_dope_mens` |
| **Périodicité** | Mensuel — calendrier à confirmer |
| **Condition de déclenchement** | FIN du job `XDSI3AJL25_PRDFINYTDMENS` |
| **Condition Control-M de sortie** | `TOPJOB_XDDF6FIN` |
| **Accès concurrent** | Non |
| **Consignes de reprise** | Voir Liste des consignes (chapitre 5) |
| **Autres informations** | Exécution exclusive des autres alimentations du HUB Flux Financiers |
| **Paramètres** | `-d date_ctrlm -t load -c raw -b produits_fi_ytd_dope_mens false` |

---

## Notes communes pour la revue du brouillon

1. **Numérotation :** `JR25` est retenu comme disponible. Les noms fonctionnels complets des six nouveaux jobs restent des propositions.
2. **Condition de sortie :** la valeur `TOPJOB_XDDF6FIN` reprend la précision communiquée. Dans les fiches Word du lot 1, le libellé affiché est « TOP JOB XDDF6FIN » ; vérifier si la représentation doit rester identique.
3. **Accès concurrent :** les six jobs doivent respecter l'exclusion avec les alimentations existantes `JR21`, `JR22`, `JR23`, ainsi qu'entre eux, puisqu'ils écrivent dans le HUB commun.
4. **Périodicités :** les termes QUOT, MENS et INTRA constituent la base du brouillon. Les calendriers effectifs seront précisés par l'exploitation.
5. **Correspondances SI3 :** elles sont reconstituées à partir des tables produites et de la logique observée dans le lot 1.
6. **Fichier de configuration :** les noms des tables `_delta` et `_delta_histo` sont repris dans la description comme dans les anciennes fiches ; cela ne signifie pas que les deux sont systématiquement lues à chaque `LOAD` récurrent.
7. **Cas FIFO :** une seule fiche est proposée à ce stade. La présence éventuelle d'un job SI3 FIFO EOD supplémentaire devra être vérifiée avant validation définitive.

**Bilan : six fiches Control-M pour cinq MAS, dont deux jobs EOD et INTRA alimentant `mrv_resultat_cession_quot`.**
