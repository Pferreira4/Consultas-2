---AUC_Medio
(
SELECT
    DATE_TRUNC('month', dam.dat_reference) AS mes_ref,
    '%{semana}' AS semana,
    CASE
        WHEN idt_financial_service IN (21053) THEN 'CDB'
        WHEN idt_financial_service IN (21063) THEN 'Tesouro Direto'
        WHEN idt_financial_service IN (21083) THEN 'Fundos de Investimentos'
        WHEN idt_financial_service IN (21073) THEN 'Renda Variável'
        WHEN idt_financial_service IN (21093) THEN 'RF3 - Emissão Bancária de Terceiros'
        WHEN idt_financial_service IN (21096) THEN 'Compromissada'
        ELSE 'outros'
    END AS Produto,
    'AUC_Medio' AS Indicador,
    ROUND(
        SUM(num_transaction_value) / COUNT(DISTINCT DATE_PART('day', dam.dat_reference)),
        2
    ) AS Valor
FROM dax.analytical_customer_transactions dam
JOIN investimentos.dim_periodo_investimentos dp
    ON dp.dat_dia = dam.dat_reference
   AND dp.flg_util = 1
WHERE DATE_PART('day', dam.dat_reference) <= '${dia}'
  AND mes_ref IN ('2024-10-01','2025-08-01','2025-09-01','2025-10-01')
  AND Produto NOT IN ('outros')
GROUP BY 1, 2, 3, 4
ORDER BY 1, 3
)


--------

  -- 3. AuC_Medio (agora média diária no próprio dia, então basicamente igual ao valor)
  select
    DATE_TRUNC('month', dat_reference) AS mes_ref,
    '${semana}' as semana,
  CASE
        WHEN idt_financial_service IN (21053) THEN 'CDB'
        WHEN idt_financial_service IN (21063) THEN 'Tesouro Direto'
        WHEN idt_financial_service IN (21083) THEN 'Fundos de Investimentos'
        WHEN idt_financial_service IN (21073) THEN 'Renda Variável'
        WHEN idt_financial_service IN (21093) THEN 'RF3 - Emissão Bancária de Terceiros'
        WHEN idt_financial_service IN (21096) THEN 'Compromissada'
        ELSE 'outros'
    end as Produto,
    'AuC_Medio' as Indicador,
    round(sum(num_transaction_value), 2) as Valor
  from dax.analytical_customer_transactions 
  where dat_reference::date >= '2024-01-01'
    and idt_financial_service in (21053,21063,21083,21073,21093,21096)
  group by 1, 2, 3, 4
