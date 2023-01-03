# aula13
Nesta aula introduzimos as vantagens e utilização de PL/SQL. Falamos sobre estruturas básicas Stored Procedures, Stored Functions e Triggers (mais detalhes na [aula seguinte](https://github.com/ULHT-BD/aula14/)) e apresentamos clausulas para controlo de fluxo (if, case, while e loop).
Bom trabalho!

[0. Requisitos](#0-requisitos)

[1. Stored Procedures](#1-stored-procedures)

[2. Functions](#2-functions)

[3. Condições IF e CASE](#3-condições-if-e-case)

[4. Ciclos WHILE, REPEAT e LOOP](#4-ciclos-while-repeat-e-loop)

[5. Trabalho de Casa](#5-trabalho-de-casa)

[6. Resoluções](#6-resoluções)

[Bibliografia e Referências](#bibliografia-e-referências)

[Outros](#outros)

## 0. Requisitos
Requisitos: Para esta aula, precisa de ter o ambiente de trabalho configurado ([Docker](https://www.docker.com/products/docker-desktop/) com [base de dados HR](https://github.com/ULHT-BD/aula03/blob/main/docker_db_aula03.zip) e [DBeaver](https://dbeaver.io/download/)). Caso ainda não o tenha feito, veja como fazer seguindo o passo 1 da [aula03](https://github.com/ULHT-BD/aula03/edit/main/README.md#1-prepare-o-seu-ambiente-de-trabalho).

Caso já tenha o docker pode iniciá-lo usando o comando ```docker start mysgbd``` no terminal, ou através do interface gráfico do docker-desktop:
<img width="1305" alt="image" src="https://user-images.githubusercontent.com/32137262/194916340-13af4c85-c282-4d98-a571-9c4f7b468bbb.png">

Deve também ter o cliente DBeaver.

## 1. Stored Procedures
Stored Procedure são procedimentos em SQL armazenados como objetos de código pré-compilado na base de dados e que podem ser executados diretamente no SGBD. Permitem assegurar eficiência e segurança.

A sintaxe para criação de um SP é:

``` sql
CREATE PROCEDURE <proc-name> 
		(param_spec1, param_spec2, …, param_specn ) 
BEGIN
	-- codigo a executar	
END;
```

onde os parâmetros podem ser definidos como de entrada ```IN```, saída ```OUT``` ou entrada e saída ```INOUT```.

Por exemplo
``` sql
DELIMITER $$

CREATE PROCEDURE sp_exemplo (IN param1 INT, OUT param2 INT)
BEGIN
	SELECT COUNT(*) INTO param2 
	FROM tabela
	WHERE num=param1;
END$$

DELIMITER ;
```

Neste exemplo a cláusula ```DELIMITER``` permite alterar o caracter delimitador de instruções para que este possa ser usado no corpo do bloco de código. No final é importante repôr o caracter ```;```.

Chamamos o procedimento usando a cláusula ```CALL```, por exemplo:
``` sql
CALL sp_exemplo(1, @a);
```

Podemos destruir um stored procedure usando a cláusula ```DROP```:
``` sql
DROP PROCEDURE sp_exemplo;
```

## 2. Functions
À semelhança dos Stored Procedures, as Stored Functions são um tipo de objeto constituído por um bloco de código précompilado armazenado na base de dados e que pode ser executado em qualquer momento. Uma função pode ser criada através da sintaxe:

``` sql
CREATE FUNCTION <nome> (<arg1> <type1>, ) [<arg2> <type2> [,…]) RETURNS <return type> [DETERMINISTIC]
BEGIN
	-- codigo a executar
END
```

por exemplo:

``` sql
DELIMITER $$

CREATE FUNCTION ufn_darBonus (notaantiga DOUBLE, bonus DOUBLE)
DETERMINISTIC
BEGIN
	DECLARE notabonificada DOUBLE;
	
	SET notabonificada = notaantiga + bonus;
	RETURN notabonificada;
END$$

DELIMITER ;
```

As functions são funções definidas pelo utilizador que, uma vez criadas, têm um comportamento semelhante a funções já existentes no SGBD e podem ser usadas nas queries, por exemplo:

``` sql
SELECT nome, nota, ufn_darBonus(nota, 2) FROM Aluno;
```

Podemos destruir uma função usando a cláusula ```DROP```:
``` sql
DROP FUNCTION ufn_darBonus;
```

## 3. Condições IF e CASE
Podemos em SQL usar condições para definir o controlo de fluxo de execução dos blocos de código. Tal como em muitas outras linguagens, a cláusula ```IF``` permite definir o comportamento de execução de acordo com o resultado de teste de uma dada condição. Podemos testar várias condições através do encadeamento de várias cláusulas ```IF-ELSIF```.
A sintaxe é:

``` sql
IF <condition> THEN
<statements>
ELSEIF <condition> THEN
<statements>
ELSE
<statements>
END IF
```

por exemplo:
``` sql
IF salario > 760 THEN
  SET res = 'acima do salário mínimo';
ELSE
  SET res = 'abaixo do salário mínimo';
END IF
```

Conjunto de várias condições podem também ser testadas usando a cláusula ```CASE```, a sintaxe é:

``` sql
CASE <expression>
WHEN <value> THEN
<statements>
WHEN <value> THEN
<statements>
…
ELSE
<statements>
END CASE;
```

ou

``` sql
CASE
WHEN <condition> THEN
<statements>
WHEN <condition> THEN
<statements>
…
ELSE
<statements>
END CASE;
```

por exemplo:
``` sql
CASE JOB_ID
  WHEN 'IT_ADMIN' THEN SET res = 'Administrador de Sistemas';
  WHEN 'HR_MGR' THEN SET res = 'Gestor de Recurso Humanos';
ELSE
  SET res = 'Não definido';
END CASE;
```

## 4. Ciclos WHILE, REPEAT e LOOP
Podemos em SQL usar ciclos para definir o controlo de fluxo de execução e repetir blocos de código de acordo com o teste de uma condição. Tal como em outras linguagens podemos usar a cláusula ```LOOP```, ```REPEAT``` e ```WHILE```.

A sintaxe para ```LOOP``` é:

``` sql
[begin_label:] LOOP
<statement list>
END LOOP [end_label]
```

A sintaxe para ```REPEAT/UNTIL``` é:

``` sql[begin_label:] REPEAT
<statement list>
UNTIL <search_condition>
END REPEAT [end_label]
```

A sintaxe para ```WHILE``` é:

``` sql
[begin_label:] WHILE <search_condition> DO
<statement list>
END WHILE [end_label]
```

Por exemplo:
``` sql
DECLARE sal FLOAT DEFAULT 0;
DECLARE id_iter INT DEFAULT 1;

WHILE sal < 1000 DO
  SELECT salary INTO sal WHERE id = id_iter;
  SET id_iter = id_iter + 1;
END WHILE
```

podemos controlar o fluxo de execução de loops recorrendo às cláusulas ```ITERATE <label>``` para recomeçar loop de novo (equivalente a continue) ou ```LEAVE <label>``` para sair do ciclo (equivalente a break).


### Exercícios
1. Escreva o código PL/SQL que permite criar o procedimento que recebe como argumento dois inteiros e retorna soma, subtração e multiplicação. Deverá ser executado usando
``` sql
CALL sp_operacoes(10, 15, @soma, @diferenca, @multiplicacao);
```

2. Escreva o código PL/SQL que permite criar o procedimento que recebe como argumento um JOB_ID e devolve o salário médio para essa posição
``` sql
CALL sp_avgsalary('IT_ADMIN', @salmedio);
```

3. Escreva o código PL/SQL que permite criar uma função que recebe cinco valores inteiros e devolve o valor máximo
``` sql
SELECT ufn_max3(3, 8, 2, 100, 6);
```

4. Escreva o código PL/SQL que permite criar uma função que recebe um código de departamento e retorna o valor médio de salários desse departamento. Caso seja NULL, retorna o salário de todos os empregados da empresa.
``` sql
SELECT ufn_avgsalary(NULL);

SELECT ufn_avgsalary(90);
```

## 5. Trabalho de Casa
(não há trabalho de casa devido a entrega de trabalho de grupo até dia 8)


## 6. Resoluções
[Resolução dos exercícios em aula](https://github.com/ULHT-BD/aula13/blob/main/aula13_resolucao.sql)


## Bibliografia e Referências
* [Slides aula](https://github.com/ULHT-BD/aula13/blob/main/Aula13.pdf) 
* [Documentação PLSQL - tutorialspoint](https://www.tutorialspoint.com/plsql/index.htm)


## Outros
Para dúvidas e discussões pode juntar-se ao grupo slack da turma através do [link](
https://join.slack.com/t/ulht-bd/shared_invite/zt-1iyiki38n-ObLCdokAGUG5uLQAaJ1~fA) (atualizado 2022-11-03)
