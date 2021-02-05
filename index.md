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


SELECT * 
FROM   HR.EMPLOYEES E 
WHERE  E.DEPARTMENT_ID = (SELECT DEPARTMENT_ID 
                          FROM   HR.DEPARTMENTS D 
                          WHERE  UPPER(D.DEPARTMENT_NAME) = 'SALES') 
ORDER  BY E.EMPLOYEE_ID;

 
 
  
  
  
  
  
  
  
  
  
  
  
  
 
 
 

 We are a family owned and operated business.
We are a family owned and operated business.
