# Resolvendo Problemas Simples e Complexos com Subquery
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


1)	 Buscar os funcionários que pertencem ao departamento Sales, no entanto não sabemos o ID do departamento e/ou o mesmo pode ser alterado um dia. 	
Para resolver esse problema iremos criar uma inner query single-row(retorna uma única linha e coluna), que será utilizada na clausula where pela outer query.
```
SELECT * 
FROM   HR.EMPLOYEES E 
WHERE  E.DEPARTMENT_ID = (SELECT DEPARTMENT_ID  FROM   HR.DEPARTMENTS D 
                             WHERE UPPER(D.DEPARTMENT_NAME) = 'SALES') 
ORDER  BY E.EMPLOYEE_ID;
```

![Image](https://bn1301files.storage.live.com/y4mZhu5Nfg1XieOpN3OIKhiBEgntiOO9fFxIwWUrpRCNwlGHfviN68tSbRlllpTZlQo0O88DI3keqWKPkYey_C5nTmX9yhc8C83URY-4X9CHGMXSXGA4W-WKaCvruPVg1qChFoVig2hF1je4Ecaz2OrJuZLaH940DIE76Tpsyx70wKAKNW8niUE8TD5jZh87sVc?width=1279&height=667&cropmode=none)


2)	 Buscar o nome do funcionário e do seu Manager desde que eles tenham o mesmo sobrenome.
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

![Image](https://bn1301files.storage.live.com/y4m2ZMcAwF8fiXcfNYXMzubVWqMe7BJIWXPePq0ZCzRsZWysY53-kEt80jkO99TQm1KpX8ULssJ2k7cyGS1bB6WDj8k9Q93FTgnw1VDmB2Mt1S7toyyup4GaPtbdfa5CXTYVpiwjw8s2sr3Z-D65fCLl45K56Qj-pbpCfsOghR0Ix-YXC8dwkzeewDTIqULFjxU?width=1283&height=639&cropmode=none)


 3)	Precisamos inserir em uma tabela nova chamada Promotion Employees os funcionários a quais trocaram de setor nos últimos 12 meses e que irão receber uma bonificação de 5% em seu salário. Aqui iremos utilizar a subquery em conjunto com a instrução Insert.

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

![Image](https://bn1301files.storage.live.com/y4mnUex9C9CaxlNY6e7a0n133X8d4-7dwvBXes_9QhoZg2RjOmvwGwfNlpIScBOq12OyPoiNoR2SmYidRluibqxUJBYe70TOIHyNHmWJ21_c0scvy_pK45OdMihwFX98U0HKph712aF7gpSrKeC4bt-rAQiG4GmJDZKPVjkLiB0IpTTQ9UJU21HzzPqtaRDjyvi?width=1275&height=661&cropmode=none)


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
![Image](https://bn1301files.storage.live.com/y4mRvXGW9P5OtebB7JqBbK_Qr9uonha695kEyNjGadUzmCYmQKMgACnE4aGZ4XVaxrmzlzCTrJgffON-4QChqLldlWRsQHHO1VA1gRWmP0JdYNUjp9l7VIwG50EgL_ufd3aOrDnOZ8HgZ2nxfCIy3ApcmkNshAUCgVMvSuPRO6FeTFfrdWhXJX3Q_Si6NDps9w-?width=1277&height=661&cropmode=none)

Agora vamos verificar os registros inseridos.

![Image](https://bn1301files.storage.live.com/y4mxrGwlW1Fyg26dQTSdYtLFyYaNlP3l4WVRu_OYanSiG1A-DCXpjKMpS9JH1i0VEap18aiW6QIvIokCezafaTdMNncNezs6OU6-h8Ik72KqHfoUvxN-19I901WmdUBVl8h5dNfejzMJKyzzRrLUfJoMlbXzdETyazkHLw5M4GqRNGNEb4rsm3exuTSjn23hUos?width=1279&height=661&cropmode=none)


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

![Image](https://bn1301files.storage.live.com/y4mIV1hjHQEWF2gqaoMjfN7xVWSWxd7n9bGXF22DeJ1bv3fyuoxvCMSzzqrSzFuCwiS0kipFJJs9nd4ThF3eBxAcDIXoeGQVxYAREwQ_iNfiCSTMk13GYGmwUc98Aq6f0C25nVcITrDBY7IgAT0Y3BkZoUe4fnIIqdqyrvc8HYxSgQKjuGzk1IdtZujsdbcrkyS?width=1279&height=659&cropmode=none)


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

![Image](https://bn1301files.storage.live.com/y4mCJTg0_tXGsuL2AxbglCnrhsgPpjMBLJpljeiR8AkTMpyXvgX_yWyZN5etWecP38aKP9KgHUUNJLzO6nKPwAnIJcY9deaciJcK7fMk9lUvJwC5JBBtqII0DojpCHYHjMGJvDdOcmf-dbMK7YnrNGZlBueZs_8_Wadw62VA_w3f6rQNCX3Q4ebC4fCwPQW069O?width=1277&height=637&cropmode=none)


 6)	Precisamos atualizar o salário dos funcionários para o mesmo valor do funcionário que possui o maior valor do seu cargo, Para resolver esse problema iremos criar uma inner query, que será utilizada na clausula SET pela outer query.
```
UPDATE HR.EMPLOYEES E 
SET    SALARY = (SELECT MAX(SALARY) 
                 FROM   HR.EMPLOYEES E2 
                 WHERE  E2.JOB_ID = E.JOB_ID);
```                 

![Image](https://bn1301files.storage.live.com/y4mZn8DIG0S935sg1iv4gYtjPZhSnH1Wr-C3BoJAq4Dx9EB2_HdHRRd-y2sODDLJnuJfcC2ot_48ilSG9AGMjY0RHeVCpy6LwCXzt5oUbxBB7QIA-1cfvs_f1ODqRUFcV06fxn8yJ_451nMip0evli6fQ9-luSPlLpjscFuHkS-BSfAHmXg0OWrrq8TXfl62rUU?width=1279&height=639&cropmode=none)


Vamos verificar os funcionários e seus salários.
![Image](https://bn1301files.storage.live.com/y4mQpLBnay5zZOeLLemNAsA-3gAYe0r0AuTaRfgISX01sJ4lPYYerE9ULE4IvIjCigTv-XJieDgfbQ71MCT6OhqU1kWiiWDYERc3E-SF_Shw6dbt3esMD-4iDKf8ax3WtocZWT8b8SjWCgRZJwJFKdElbNrn3uQKXyZm2Dpwb8-aJA_wM_Xgaf71jAHjiJ6SlPO?width=1275&height=633&cropmode=none)


Podemos verificar que todos os funcionários com o mesmo cargo (JOB_ID), possuem o mesmo salário.
