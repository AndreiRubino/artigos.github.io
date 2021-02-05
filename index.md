# RESOLVENDO PROBLEMAS SIMPLES E COMPLEXOS COM SUBQUERY
Este artigo tem como objetivo descrever as diversas formas de como podemos utilizar subquerys para resolver problemas simples e complexos. As formas abaixo são uteis no dia a dia de cada desenvolvedor/dba.



Uma subquery é uma instrução SELECT que está condicionado à outra instrução SQL.
A subquery é uma instrução muito versátil a qual pode ser utilizada em diversos cenários e servem geralmente para resolver problemas que teriam que ser feitas com 2 ou mais consultas.

Podemos utilizar subquerys em instruções select, insert, update e delete. E nessas instruções podemos fazer o uso de subquery em diversas clausulas como a clausula into, values, set, where, having, o Oracle também permite utilizarmos a subquery tanto  ao lado direito quanto esquerdo do operador =. Show, não é?

Agora vamos esclarecer alguns conceitos e criar um cenário para aplicarmos nossas subquerys.

**Outer Query**

É sempre a query externa, ou seja, a query da esquerda.

**Inner Query**

É sempre a query interna, ou seja, a query da direita.

**Subquerys Escalares (single-row)**

São subquerys que retornam apenas uma linha e uma coluna.
**Subquerys Correlacionadas (mutilple rows)**

São subquerys que se relacionam com a query externa, e são processados linha a linha.


Para realizar nossos testes iremos utilizar o schema HR do banco XE.

**Iremos resolver 6 problemas, sendo eles:**
Buscar os funcionários que pertencem ao departamento Sales, no entanto não sabemos o ID do departamento e/ou o mesmo pode ser alterado um dia.1
Para resolver esse problema iremos criar uma inner query single-row(retorna uma única linha e coluna), que será utilizada na clausula where pela outer query.
```
SELECT * 
FROM   HR.EMPLOYEES E 
WHERE  E.DEPARTMENT_ID = (SELECT DEPARTMENT_ID  FROM   HR.DEPARTMENTS D 
                             WHERE UPPER(D.DEPARTMENT_NAME) = 'SALES') 
ORDER  BY E.EMPLOYEE_ID;
```

![Image](https://bn1301files.storage.live.com/y4p-Svy8hgENZ0qB5HyWeRUtupZDA_swBuV38-Y7hPSYrMM_5nK0Hq9cwQLD1BTfw8DD5ALst-jmILSzVEUOvBYk-p0HvVLjWAKlv0XxVf0cMZH-8fj_QqQjl9U15XsoroejkiDUwrbMQz97cEHJO-2sCNyo6Z1_47uRtiUvCrhhI8nG2mfkQszgO3JLspQ-Nw9/1.png?psid=1&width=1176&height=613)


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

![Image](https://bn1301files.storage.live.com/y4pKzwikNPJvoCYGiCOdtj_bL1KZM9hZt6aP7HyFXM_Ua06zeqgAaQE7qQETdPyZVHDBYVfeDSlqgzVTeM66LJb1JuQfh5QOBrn1m6-fNj7CcntQjHviCpp7mBBlfnXQTvGpBP5E8H_LavHOPvdJ_QDCDfeMilpN6iR7KsaIHDrMhl3AnuXf1sziFeA7g61LjshtXrjOcKQtDnZ7wj5p72lxv1txWFKc3QrC9BjsBEretg/2.png?psid=1&width=1231&height=613)


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

![Image](https://bn1301files.storage.live.com/y4pbJnnvipKhrVI0DoFwwaa9OgN0tHGXIaRDQXK18kc_JwfsJ4BJI5leeAnb3Eco2YlgIYz9qGJHqRX05CVUODYK7Xcyh05mioXC585RfIAwd9UX5Yi9dmA2bqGFtCHeku3lk-NlDQeZZr_ZAI1Bmwfr9PLGExCm3tshu-QqVrJQqWo3w0mobeQyTlm1kvB6YW3w023BENrVXinJgjwml-I-jpDJKL2XvFLU716VxnCaS0/3.png?psid=1&width=1183&height=613)


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
![Image](https://bn1301files.storage.live.com/y4peMM08kak6VkoSQ8EuavTGhlPfbFkR_JhjjLl1BKsg18JPE4X4aj1Tad2CVwGINUofSuWUwwquIWnWpJJdmF-vkiRqJDfwm1ffV5LATSxpX234BqqZ-HcTr251Hec_TlbhMTSpJjpdxw1b7oT08k1MsUkKRHWiEkDDuFUJxUJ__73MO6RNzZn4Q-LLqDX1IGT2_MFS71aAyTPlFlQX14R65X4oHOO-XjzmD1JjR1yWyk/3-3.png?psid=1&width=1185&height=613)

Agora vamos verificar os registros inseridos.

![Image](https://bn1301files.storage.live.com/y4p8xAYOGCRAWBcthUkX7Ry3sA2c6sxI9uHFeusgXwJMd6Rr6Ssk-k1gDMxPKWgQgpd1mEV9qniZ7BwNf9GXEk1ySA3RIa3llbiUv6CaBx128-WcrkEdYJRgKvIx4ZBdzNFLp4gngW8aQ4QQsenupl6XTNDeDCNqcKrxS4WvR9LNLkrGQsllc3M5wXNeTtSbDY88fDWOjd6kFiQjbri0Hy8TR5N1cC7lRLXqA1kCXkUgso/3-4.png?psid=1&width=1187&height=613)


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

![Image](https://bn1301files.storage.live.com/y4p06CXHQPGZyZyoQNKLJ3XpiVsfFxKvu0UkaaAatq6_TXFyOp6gcpxUK3X4hBfTo4P9b2YwajxzlWXseh5sw_sGLs163ZIrjBby8dzXbYDJjhD96cw1QSR_iR1DkXNB9HuW7m_TzqUFe6adQ5fB4T1_MpX50df48umaLnFykrnjNEMiYnCnZm63RMGUb-bm_rBJ1poVfLHfZF09Daeo0O39RbqbtClbqgaFe9iDboqafk/4.png?psid=1&width=1190&height=613)


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

![Image](https://bn1301files.storage.live.com/y4p7dHC7PGpvgErIYL48Jc00hejBxCCgCqIPzJvDRXTKe_9EFf3YhWGY02yomUeXDu4YmCTQwxfcEfRidQePNbn7ctfhURhFTfTziuSxF4TFOYV6zP93IpAueo4LJJxMlfxeO7gTek5ooCJl4R4E8q2gUB5rTf4iJq7-R4Bj041M1Bu0lcfHFWG2TicWdgoBdUMeg84IvH-hv8eiAZb_X7DANVRkZqPidrxtwRCQSrP4vs/5.png?psid=1&width=1229&height=613)


6)	Precisamos atualizar o salário dos funcionários para o mesmo valor do funcionário que possui o maior valor do seu cargo, Para resolver esse problema iremos criar uma inner query, que será utilizada na clausula SET pela outer query.
```
UPDATE HR.EMPLOYEES E 
SET    SALARY = (SELECT MAX(SALARY) 
                 FROM   HR.EMPLOYEES E2 
                 WHERE  E2.JOB_ID = E.JOB_ID);
```                 

![Image](https://bn1301files.storage.live.com/y4pzjEUD-Hblc7BJQy0QXy2UTwElKIspMrS3lxnlDSBXlnZSYh2wpKYiJsmBTbLiAldZCmo7ekP3OrpyFaReWmkWS6NdySN7PMhBq7obhgRy_09DPDWxWEWjk2bzZX_q9LI08Y2Mgapozeppm03oCrqWHlC_pQNgZh7B9glDLSkYh7LWHyyZ8zfG9pjcBuCs_GirP_4lca8Bg464SqW2IYg_Hi6XIHSvJFE_oRDoSA9ZLE/6.png?psid=1&width=1227&height=613)


Vamos verificar os funcionários e seus salários.
![Image](https://bn1301files.storage.live.com/y4peTfcMV7vjv5KiicTMCCHUVQxwdWdMYqaZq_vhlG-5B_JjqaZKYLfRk-rcQNfiBE_zVHO9rPoyNH99Wntx9FVgCmfxZOOwQCHAAVJjbGgfHboK7R16lybqXBuROJJztHC5aaJdJNyCVaE3Fzgd47sKiFDz_gfubxwJca3COfGRXuU4QUT_C-hWgRWjPtHfn4djmuK0txYSNrYudkdTXJ9UIF8LLgyfEduQMLwmZ8Vbeg/6-2.png?psid=1&width=1235&height=613)


Podemos verificar que todos os funcionários com o mesmo cargo (JOB_ID), possuem o mesmo salário.
