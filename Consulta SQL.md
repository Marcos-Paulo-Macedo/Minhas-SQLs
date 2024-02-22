
SELECT SEL.*
       REVERSE(SUBSTRING(REVERSE('0000000000'|| SEL.PEDVENDA_ID ) FROM 1 FOR 10)) AS BARRAS
FROM 
( 
    SELECT 1 AS RELATORIO,
           SEL.PEDVENDA_ID,
           SEL.CLIFOR_ID AS CODCLIENTE, 
           COALESCE(C.NOME,'') || ' - ' || COALESCE(C.NOMEFANTASIA,'') AS NOMECLIENTE, 
           SEL.CARGA_ID,
           SEL.QUANTIDADE,
           SEL.DATA,
           SUM(COALESCE(SEL.MASSA,0)) AS MASSA
    FROM 
    ( 
        SELECT SEL.PEDVENDA_ID,
               SEL.CLIFOR_ID,
               SEL.CARGA_ID,
               SEL.QUANTIDADE,
               SEL.DATA,
               COUNT(VOL.ALMOXVOLUME_ID) AS VOLUMESTOTAL,
               SUM(CASE
                    WHEN (VOL.ALMOXVOLUMETIPO_ID = 6) THEN 1 ELSE 0
                   END ) AS MASSA
        FROM
        (
                SELECT  SEL.PEDVENDA_ID,SEL.CLIFOR_ID,SEL.CLIFOR_ID,
                                         SEL.QUANTIDADE,
                        SEL.DATA
                FROM
                (
                        SELECT  SEL.*,
                                PI.QUANTIDADE,
                                (PI.QUANTIDADE * P.NOTAPESOLIQUIDO) AS PESOTOTALITEM
                        FROM
                        (
                                    SELECT PV.PEDVENDA_ID, PV.CLIFOR_ID,
                                           PV.VALORTOTALPEDIDO AS TOTALPEDIDO,
                                           PV.OBSERVACAO, PV.OBSERVACAO2,
                                           PV.CARGA_ID, PV.NNOTA,
                                           PV.INDICE_CLI, PV.INDICE_CID,
                                           PV.DATA
                                    FROM PEDVENDAS PV
                                    WHERE (PV.CARGA_ID IN (:NUMCARGA))
                        ) SEL
                        INNER JOIN PEDVENDAITENS PI
                        ON (PI.PEDVENDA_ID = SEL.PEDVENDA_ID)
                        INNER JOIN PRODS P
                        ON (P.PROD_ID = PI.PROD_ID)
                        WHERE PI.PROD_ID = 5370
                        ORDER BY 1,2
                ) SEL
                GROUP BY 1,2
        ) SEL
        LEFT JOIN PEDVENDA_VOLUMES_AREA(SEL.PEDVENDA_ID) V
        ON  (0=0) 
        LEFT JOIN ALMOXVOLUMES VOL
        ON  (V.OUT_ALMOXPEDVENDA_ID = VOL.ALMOXPEDVENDA_ID)
        GROUP BY 1,2,3,4,5,6
    ) SEL 
    INNER JOIN CLIFORS C 
    ON  (SEL.CLIFOR_ID = C.CLIFOR_ID) 
    INNER JOIN CIDADES CI 
    ON  (C.CIDADE_ID = CI.CIDADE_ID) 
 
    GROUP BY 1,2,3,4,5,6
) SEL

ORDER BY SEL.CARGA_ID, SEL.INDICE_CID NULLS LAST, CIDADECLIENTE,
         SEL.INDICE_CLI NULLS LAST, NOMECLIENTE, PEDVENDA_ID DESC
