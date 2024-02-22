SELECT SEL.CLIFOR_ID,  --CLIENTE
       SEL.FAVORECIDO, --CLIENTE
       SEL.OPI_ID,
       SEL.PROD_ID,                --PRODUTO
       SEL.CODIGOFABRICANTE,       --PRODUTO
       SEL.PRODUTO AS NOMEPRODUTO, --PRODUTO
       SEL.QUANTIDADE,             --PRODUTO
       SEL.CUSTOTOTAL,             --PRODUTO
       SEL.PRODUZIDO AS SITUCAO, --SITUACAO DA ORDEM
       SEL.DATAPRODUCAO, --PRODUÇÃO
       SEL.HORAPRODUCAO, --PRODUÇÃO
       SEL.DATAFECHAMENTO,                     --FECHAMENTO
       SEL.HORAFECHAMENTO,                     --FECHAMENTO
       SEL.USER_ID_FECHAMENTO,                 --FECHAMENTO
       SEL.USUARIO_ID AS USUARIO_IDFECHAMENTO, --FECHAMENTO
       SEL.DATADIVISAO,     --DIVISÃO
       SEL.HORADIVISAO,     --DIVISÃO
       SEL.USER_ID_DIVISAO, --DIVISÃO
       SEL.CUSTOMEDIO,          --ORDDEMP FICHA
       SEL.UNIDADE,             --ORDDEMP FICHA
       SEL.OPFICHA_ID,          --ORDDEMP FICHA
       SEL.APONTADO,            --ORDDEMP FICHA
       SEL.DATAAPONTAMENTO,     --ORDDEMP FICHA
       SEL.HORAAPONTAMENTO,     --ORDDEMP FICHA
       SEL.USER_ID_APONTAMENTO, --ORDDEMP FICHA
       U.USUARIO_ID,   --USUARIO
       U.NOME          --USUARIO
FROM(
     ---------------------------------------------------------------------------
        SELECT 
            SEL.CLIFOR_ID,
            SEL.FAVORECIDO,
            SEL.OPI_ID,
            SEL.PROD_ID,
            SEL.CODIGOFABRICANTE,
            SEL.NOME AS PRODUTO,
            SUM(CAST(OPFICHA.QUANTIDADE AS NUMERIC(15,4))) AS QUANTIDADE,
            SUM(CAST((OPFICHA.QUANTIDADE)*(P.VALORCUSTOMEDIO) AS NUMERIC(15,4))) AS CUSTOTOTAL,
            OPFICHA.VALOR AS CUSTOMEDIO,
            OPFICHA.UNIDADE,
            SEL.PRODUZIDO,
            SEL.DATAPRODUCAO,
            SEL.HORAPRODUCAO,
            SEL.DATAFECHAMENTO,
            SEL.HORAFECHAMENTO,
            SEL.USUARIO_ID,
            SEL.USER_ID_FECHAMENTO,
            SEL.DATADIVISAO,
            SEL.HORADIVISAO,
            SEL.USER_ID_DIVISAO,
            OPFICHA.OPFICHA_ID,
            OPFICHA.APONTADO,
            OPFICHA.DATAAPONTAMENTO,
            OPFICHA.HORAAPONTAMENTO,
            OPFICHA.USER_ID_APONTAMENTO

        FROM (
              ------------------------------------------------------------------
                    SELECT  
                        OP.CLIFOR_ID,
                        CLI.NOME AS FAVORECIDO,
                        OPI.OPI_ID,
                        PI.NOME,
                        OPI.PROD_ID,
                        PI.CODIGOFABRICANTE,
                        OPI.PRODUZIDO,
                        OPI.DATAPRODUCAO,
                        OPI.HORAPRODUCAO,
                        OPI.DATAFECHAMENTO,
                        OPI.HORAFECHAMENTO,
                        OPI.USER_ID_PRODUCAO AS USUARIO_ID,
                        OPI.USER_ID_FECHAMENTO,
                        OPI.DATADIVISAO,
                        OPI.HORADIVISAO,
                        OPI.USER_ID_DIVISAO
        
                    FROM OP
                    INNER JOIN OPI
                     ON (OP.OP_ID = OPI.OP_ID)
                    INNER JOIN PRODS PI
                     ON (PI.PROD_ID = OPI.PROD_ID)
                    INNER JOIN OPFICHA OPF
                     ON (OPF.OPI_ID = OPI.OPI_ID)
                    LEFT JOIN CLIFORS CLI
                     ON (CLI.CLIFOR_ID = OP.CLIFOR_ID)
        
                    WHERE (OPI.DATAPRODUCAO BETWEEN :DTINICIAL AND :DTFINAL)
                    AND   ((:CLIFOR = 0) OR (OP.CLIFOR_ID = :CLIFOR))
                    AND   ((:PRODBASE_ID = 0) OR (PI.PROD_ID = :PRODBASE_ID))
                    AND   ((:OP_ID = 0) OR (OPI.OP_ID = :OP_ID))
                    AND   (PI.PRODSUBGRUPO_ID IN (149,140,159,99,148,163,97,161,151,106,105,150,98,160,152))
                    AND   (
                          ((:NSITUACAO0 = 0) AND (OPI.PRODUZIDO = 0)) OR
                          ((:NSITUACAO1 = 1) AND (OPI.PRODUZIDO = 1)) OR
                          ((:NSITUACAO2 = 2) AND (OPI.PRODUZIDO = 2))
                          )
        
                    GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16
                    ORDER BY 1,2,3
        
              ) AS SEL
              ------------------------------------------------------------------
        INNER JOIN OPFICHA
         ON (SEL.OPI_ID = OPFICHA.OPI_ID)
        INNER JOIN PRODS P
         ON (P.PROD_ID = OPFICHA.PROD_ID)
        
        GROUP BY 1,2,3,4,5,6,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25
        ORDER BY SEL.CLIFOR_ID, SEL.FAVORECIDO, SEL.OPI_ID

     )AS SEL
     ---------------------------------------------------------------------------
LEFT JOIN USUARIOS U
 ON U.USUARIO_ID = SEL.USUARIO_ID
WHERE U.USUARIO_ID = :USUARIO
