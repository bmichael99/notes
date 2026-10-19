to create a gold view (for user consumption)

CREATE OR REPLACE VIEW ECOMM.GOLD.amz_fct_promotion_performance AS
SELECT
    marketplace_id,
    promotion_id,
    select_from,
    select_to,
    bronze_loaded_at,
    row_data
FROM ECOMM.SILVER.amz_promotion_performance


