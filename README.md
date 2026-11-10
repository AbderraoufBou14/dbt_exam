
-- ============================================================
-- DF6DFM-75 | VALIDATION FONCTIONNELLE RAW VAULT | DEV
-- Objet testé : instrument
-- Audit : 09/11/2026
-- ============================================================


-- ============================================================
-- 0. AUDIT : CONSULTER LES RESULTATS
-- ============================================================

-- Historique des exécutions de l'objet métier
SELECT run_id, object_name, started_at, status
FROM db_dev_df6_travail.audit_run
WHERE object_name = 'instrument'
ORDER BY started_at DESC
LIMIT 5;


-- Résultats des contrôles qualité : OK / KO / ERROR
SELECT test_id, component, physical_table_name,
       status, value
FROM db_dev_df6_travail.audit_result
WHERE run_id = '5a942b04-96e9-41ad-8614-767a4345a4fe'
  AND test_id NOT LIKE '%COUNT%'
  AND test_id NOT LIKE '%DUPLICATES%'
ORDER BY test_id, physical_table_name;



-- ============================================================
-- 1. SOURCE BK -> HUB | SOURCE_BK_ABSENT_HUB
-- ============================================================

-- Vérifie les BK des deux sources de l'objet instrument
-- Attendu : source_1 = 0 et source_2 = 0 (OK)

SELECT 'source_1' AS source, COUNT(*) AS nb_absentes
FROM (
    SELECT s.id
    FROM db_dev_si3.lac_pvr_instrument_delta_histo s
    LEFT ANTI JOIN db_dev_df6_brute.hrv_instrument h
        ON s.id = h.id_instrument
    WHERE s.id IS NOT NULL
) t

UNION ALL

SELECT 'source_2' AS source, COUNT(*) AS nb_absentes
FROM (
    SELECT s.codevaleur
    FROM db_dev_si3.lac_pvo_inventaires_quot_dern_vers_instrument s
    LEFT ANTI JOIN db_dev_df6_brute.hrv_instrument h
        ON s.codevaleur = h.id_instrument
    WHERE s.codevaleur IS NOT NULL
) t;

-- NOTE : sources contrôlées séparément dans le moteur.
-- Union préalable des BK à confirmer avec la spécification.
-- NULL et transformations métier à vérifier séparément.



-- ============================================================
-- 2. SAT -> HUB | SAT_HK_ABSENT_HUB
-- ============================================================

-- CAS KO : srv_inst_action
-- Attendu : 7 anomalies

SELECT COUNT(*) AS nb_hk_absentes
FROM db_dev_df6_brute.srv_inst_action s
LEFT ANTI JOIN db_dev_df6_brute.hrv_instrument h
    ON s.hk_srv_inst_action = h.hk_hrv_instrument
WHERE s.hk_srv_inst_action IS NOT NULL;


-- CAS OK : srv_inst_future
-- Attendu : 0 anomalie

SELECT COUNT(*) AS nb_hk_absentes
FROM db_dev_df6_brute.srv_inst_future s
LEFT ANTI JOIN db_dev_df6_brute.hrv_instrument h
    ON s.hk_srv_inst_future = h.hk_hrv_instrument
WHERE s.hk_srv_inst_future IS NOT NULL;



-- ============================================================
-- 3. MAS -> HUB | CONTROLE A CONFIRMER
-- ============================================================

-- Vérifie que les HK du MAS existent dans son HUB parent
-- Remplacer les noms selon les JSON et le schéma Hive
-- Attendu : à comparer avec AUDIT_RESULT si contrôle présent

SELECT COUNT(*) AS nb_hk_absentes
FROM db_dev_df6_brute.<TABLE_MAS> m
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_HUB> h
    ON m.<HK_MAS_PARENT> = h.<HK_HUB>
WHERE m.<HK_MAS_PARENT> IS NOT NULL;



-- ============================================================
-- 4. LINK -> HUB | LINK_FK_ABSENT_HUB
-- ============================================================

-- Vérifie les références FK d'un LINK vers son HUB
-- Attendu : à comparer avec AUDIT_RESULT

SELECT COUNT(*) AS nb_fk_absentes
FROM db_dev_df6_brute.<TABLE_LINK> l
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_HUB> h
    ON l.<FK_LINK> = h.<HK_HUB>
WHERE l.<FK_LINK> IS NOT NULL;



-- ============================================================
-- 5. LINK <-> SAT | COHERENCE REFERENTIELLE
-- ============================================================

-- 5A. LINK -> SAT
-- HK du LINK absentes du SAT
-- Attendu : à comparer avec AUDIT_RESULT

SELECT COUNT(*) AS nb_absentes
FROM db_dev_df6_brute.<TABLE_LINK> l
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_SAT> s
    ON l.<HK_LINK> = s.<HK_SAT_PARENT>
WHERE l.<HK_LINK> IS NOT NULL;


-- 5B. SAT -> LINK
-- HK du SAT absentes du LINK

SELECT COUNT(*) AS nb_absentes
FROM db_dev_df6_brute.<TABLE_SAT> s
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_LINK> l
    ON s.<HK_SAT_PARENT> = l.<HK_LINK>
WHERE s.<HK_SAT_PARENT> IS NOT NULL;



-- ============================================================
-- 6. LINK <-> MAS | COHERENCE REFERENTIELLE
-- ============================================================

-- A EXECUTER SUR UN AUTRE OBJET METIER
-- Relation LINK/MAS non disponible dans le cas instrument étudié

-- 6A. LINK -> MAS

SELECT COUNT(*) AS nb_absentes
FROM db_dev_df6_brute.<TABLE_LINK> l
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_MAS> m
    ON l.<HK_LINK> = m.<HK_MAS_PARENT>
WHERE l.<HK_LINK> IS NOT NULL;


-- 6B. MAS -> LINK

SELECT COUNT(*) AS nb_absentes
FROM db_dev_df6_brute.<TABLE_MAS> m
LEFT ANTI JOIN db_dev_df6_brute.<TABLE_LINK> l
    ON m.<HK_MAS_PARENT> = l.<HK_LINK>
WHERE m.<HK_MAS_PARENT> IS NOT NULL;



-- ============================================================
-- 7. INVESTIGATION : DETAIL DES ANOMALIES
-- ============================================================

-- Détails historisés par le moteur d'audit

SELECT *
FROM db_dev_df6_travail.audit_detail
WHERE run_id = '5a942b04-96e9-41ad-8614-767a4345a4fe'
LIMIT 100;

-- FIN

