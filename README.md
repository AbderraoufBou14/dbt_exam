
WITH delta AS (
    SELECT
        m_reference,
        portefeuille,
        numerodoperation,
        dateope,
        COUNT(*) AS nb_delta
    FROM db_prex_si3.lac_pvo_pdt_fi_resultat_cession_fifo_eod_delta
    GROUP BY
        m_reference, portefeuille, numerodoperation, dateope
),
mas AS (
    SELECT
        m_reference,
        portefeuille,
        numerodoperation,
        dateope,
        SUM(CASE
            WHEN ldts = CAST('2026-11-18 17:52:29' AS TIMESTAMP)
            THEN 1 ELSE 0 END) AS nb_init,
        SUM(CASE
            WHEN ldts = CAST('2026-11-18 18:19:06' AS TIMESTAMP)
            THEN 1 ELSE 0 END) AS nb_load
    FROM db_prex_df6_brute.mrv_resultat_cession_fifo_quot
    GROUP BY
        m_reference, portefeuille, numerodoperation, dateope
)
SELECT
    d.*,
    COALESCE(m.nb_load, 0) AS nb_load,
    COALESCE(m.nb_init, 0) AS nb_deja_init,
    d.nb_delta - COALESCE(m.nb_load, 0) AS ecart
FROM delta d
LEFT JOIN mas m
    ON d.m_reference <=> m.m_reference
   AND d.portefeuille <=> m.portefeuille
   AND d.numerodoperation <=> m.numerodoperation
   AND d.dateope <=> m.dateope
WHERE d.nb_delta > COALESCE(m.nb_load, 0)
ORDER BY ecart DESC;




----------------



SELECT *
FROM db_prex_si3.lac_pvo_pdt_fi_resultat_cession_fifo_eod_delta
WHERE m_reference = 'VALEUR_IDENTIFIEE';
