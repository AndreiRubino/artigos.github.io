# RESOLVENDO PROBLEMAS SIMPLES E COMPLEXOS COM SUBQUERY

Este artigo tem como objetivo descrever as diversas formas de como podemos utilizar subquerys para resolver problemas simples e complexos. As formas abaixo são uteis no dia a dia de cada desenvolvedor/dba.


Uma subquery é uma instrução SELECT que está condicionado à outra instrução SQL.
A subquery é uma instrução muito versátil a qual pode ser utilizada em diversos cenários e servem geralmente para resolver problemas que teriam que ser feitas com 2 ou mais consultas.

Podemos utilizar subquerys em instruções select, insert, update e delete. E nessas instruções podemos fazer o uso de subquery em diversas clausulas como a clausula into, values, set, where, having, o Oracle também permite utilizarmos a subquery tanto  ao lado direito quanto esquerdo do operador =. Show, não é?

Agora vamos esclarecer alguns conceitos e criar um cenário para aplicarmos nossas subquerys.

Outer Query 
É sempre a query externa, ou seja, a query da esquerda.

Inner Query 
É sempre a query interna, ou seja, a query da direita.

Subquerys Escalares (single-row)
São subquerys que retornam apenas uma linha e uma coluna.

Subquerys Correlacionadas (mutilple rows)
São subquerys que se relacionam com a query externa, e são processados linha a linha.

Para realizar nossos testes iremos utilizar o schema HR do banco XE.

Iremos resolver 6 problemas, sendo eles:
1)	Buscar os funcionários que pertencem ao departamento Sales, no entanto não sabemos o ID do departamento e/ou o mesmo pode ser alterado um dia.
Para resolver esse problema iremos criar uma inner query single-row(retorna uma única linha e coluna), que será utilizada na clausula where pela outer query.
```
SELECT * 
FROM   HR.EMPLOYEES E 
WHERE  E.DEPARTMENT_ID = (SELECT DEPARTMENT_ID  FROM   HR.DEPARTMENTS D 
                             WHERE UPPER(D.DEPARTMENT_NAME) = 'SALES') 
ORDER  BY E.EMPLOYEE_ID;
```

2. Buscar o nome do funcionário e do seu Manager desde que eles tenham o mesmo sobrenome.
Nesse caso iremos criar uma inner query mutilple-row(podem retornar mais de uma linha e coluna), que será utilizada na clausula where pela outer query e irá garantir nossa regra definida, também iremos utilizar outra inner query no formato de coluna na instrução select.
Com os funcionários já filtrados pela inner query da clausula where a subquery realiza a consulta com esses dados(funcionários) e retorna o nome e sobrenome de seu Manager.

```
SELECT 
        E.EMPLOYEE_ID,
        E.FIRST_NAME || ' ' || E.LAST_NAME AS NOME_FUNCIONARIO
        E.MANAGER_ID,
        (SELECT M.FIRST_NAME || ' ' || M.LAST_NAME 
         FROM HR.EMPLOYEES M 
         WHERE  E.MANAGER_ID = M.EMPLOYEE_ID) AS NOME_MANAGER
FROM   HR.EMPLOYEES E 
WHERE  ( E.DEPARTMENT_ID, E.LAST_NAME ) IN 
       (SELECT M2.DEPARTMENT_ID, 
               M2.LAST_NAME 
        FROM   HR.EMPLOYEES M2 
        WHERE  E.MANAGER_ID = M2.EMPLOYEE_ID) 
ORDER  BY E.EMPLOYEE_ID;
```

3) Precisamos inserir em uma tabela nova chamada Promotion Employees os funcionários a quais trocaram de setor nos últimos 12 meses e que irão receber uma bonificação de 5% em seu salário. Aqui iremos utilizar a subquery em conjunto com a instrução Insert.

Primeiro iremos criar a nova tabela visto que o schema HR não a possui.
```
CREATE TABLE HR.PROMOTION_EMPLOYEES (
	PROMOTION_EMPLOYEE_ID NUMBER(6,0)
	, EMPLOYEE_ID NUMBER(6,0) NOT NULL
	, DT_PROMOTION DATE NOT NULL
	, CONSTRAINT PRM_EMP_ID_PK PRIMARY KEY(PROMOTION_EMPLOYEE_ID)
	, CONSTRAINT PRM_EMP_EMP_FK FOREIGN KEY(EMPLOYEE_ID) REFERENCES HR.EMPLOYEES(EMPLOYEE_ID)
);
```

Agora iremos fazer o insert com subselect obedecendo às regras definidas, que são funcionários a qual trocaram de setor nos últimos 12 meses.
```
INSERT INTO HR.PROMOTION_EMPLOYEES (EMPLOYEED_ID, DT_PROMOTION)
  SELECT
    E.EMPLOYEE_ID,
    SYSDATE
  FROM HR.EMPLOYEES E
  JOIN HR.JOB_HISTORY J ON E.EMPLOYEE_ID = J.EMPLOYEE_ID
  WHERE E.JOB_ID = J.JOB_ID
  AND MONTHS_BETWEEN(SYSDATE, J.END_DATE) < 12
  ORDER BY E.EMPLOYEE_ID;
```

Agora vamos verificar os registros inseridos.
Ótimo, podemos ver que havia 9 funcionários que estavam dentro de nossas regras definidas


4)	Precisamos listar o Nome, Cargo e Salário dos funcionários que possuem o salário maior do que a média dos salários de todos os funcionários, além disso, queremos também na mesma consulta a média de salário por cargos.

Nesse Caso a inner query serve de fonte de dados para a outer query. A inner query é executada primeiro e a outer query realiza o select em cima dos dados retornados pela inner query. Ou seja, primeiramente é realizado o select que carrega o Nome, Cargo, Salário de todos os funcionários e a Média Salarial por cargo, e então nesse momento a outer query é executada e realiza a validação se o salário do funcionários é maior do que a média salarial de seu cargo, retornando apenas os funcionários a quais estão dentro desta regra.
 
```
SELECT SUBQUERY.FIRST_NAME, 
       SUBQUERY.JOB_ID, 
       SUBQUERY.SALARY, 
       SUBQUERY.MEDIA_SALARIO_CARGO
FROM   (SELECT E.FIRST_NAME,
               E.JOB_ID,
               E.SALARY,
               (SELECT AVG(E2.SALARY) 
                FROM   HR.EMPLOYEES E2 
                WHERE  E.JOB_ID = E2.JOB_ID 
                GROUP  BY JOB_ID) MEDIA_SALARIO_CARGO 
        FROM   HR.EMPLOYEES E) SUBQUERY 
WHERE  SUBQUERY.SALARY > SUBQUERY.MEDIA_SALARIO_CARGO; 
```

5)	Queremos descobrir os cargos que possuem a média salarial maior do que a média salarial do cargo Finance Manager(FI_MGR). Nesse caso utilizamos a subquery para comparação de valores com a clausula Having.
```
SELECT J.JOB_ID, 
       J.JOB_TITLE,
       AVG(SALARY) 
FROM   HR.EMPLOYEES E 
JOIN   HR.JOBS J ON J.JOB_ID = E.JOB_ID
GROUP  BY J.JOB_ID 
 , J.JOB_TITLE
HAVING AVG(SALARY) > (SELECT AVG(SALARY) 
                      FROM   HR.EMPLOYEES E2
                      WHERE JOB_ID = 'FI_MGR');
```

6)	Precisamos atualizar o salário dos funcionários para o mesmo valor do funcionário que possui o maior valor do seu cargo, Para resolver esse problema iremos criar uma inner query, que será utilizada na clausula SET pela outer query.
```
UPDATE HR.EMPLOYEES E 
SET    SALARY = (SELECT MAX(SALARY) 
                 FROM   HR.EMPLOYEES E2 
                 WHERE  E2.JOB_ID = E.JOB_ID);
```                 
