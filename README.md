# aula13
Nesta aula introduzimos as vantagens e utilização de PL/SQL. Falamos sobre estruturas básicas Stored Procedures, Stored Functions e Triggers (mais detalhes na [aula seguinte](https://github.com/ULHT-BD/aula14/)) e apresentamos clausulas para controlo de fluxo (if, case, while e loop).
Bom trabalho!

[0. Requisitos](#0-requisitos)

[1. Stored Procedures](#1-stored-procedures)

[2. Functions](#2-functions)

[3. Condições IF e CASE](#3-condições-if-e-case)

[4. Ciclos WHILE e LOOP](#4-ciclos-while-e-loop)

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

A sintaxe para criação de um SP é

``` sql
CREATE PROCEDURE <proc-name> 
		(param_spec1, param_spec2, …, param_specn ) 
BEGIN
	-- codigo a executar	
END;
```

por exemplo
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

Chamamos o procedimento usando a cláusula ```CALL```, por exemplo:
``` sql
CALL sp_exemplo(1, @a);
```


## 2. Functions
A execução concorrente de várias transações pode conduzir a várias anomalias bem conhecidas Dirty Read, Non-Repeatable Read e Phantom Read.

Consoante a aplicação a desenvolver, podemos querer tolerar/permitir a ocorrência de algumas destas anomalias ou assegurar um maior nível de isolamento.

A tabela resume as anomalias e níveis de isolamento:

<img width="552" alt="image" src="https://user-images.githubusercontent.com/32137262/207202337-6c54effa-a76e-4200-9b10-6af2669d2048.png">

Em SQL podemos alterar o nível de isolamento de forma global, durante a sessão ou apenas para a transação, usando:

``` sql
SET [GLOBAL | SESSION] TRANSACTION ISOLATION LEVEL level;
```

Exemplo, para alterar nivel de isolamento da sessão para repeatable read
``` sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```


## 3. Condições IF e CASE
No MySQL o autocommit está ligado por default, o que significa que todas as instruções são executadas como uma transação quando é encontrado o caracter ```;```. Podemos desligar este mecanismo usando:

``` sql
SET autocommit=0;
```

Em SQL podemos as cláusulas ```START TRANSACTION``` inicia uma transação e a cláusula ```COMMIT``` termina e executa a transação.

``` sql
START TRANSACTION;
(sequência de comandos corpo da transação)
COMMIT;
```

Exemplo:

<img width="272" alt="image" src="https://user-images.githubusercontent.com/32137262/207200753-1bfd6f09-55b2-4e78-a012-932c35ed8516.png">

Durante a execução de uma transação, podemos anular a sua execução voltando ao estado anterior usando a cláusula ```ROLLBACK```:

``` sql
START TRANSACTION;
(sequência de comandos corpo da transação)
ROLLBACK;
```

O estado da base de dados é reposto para o ponto anterior à transação.
Podemos inclusivamente definir pontos chave (savepoints) até onde queremos reverter usando a cláusula ```SAVEPOINT nome-savepoint```. Podemos escolher até onde deve ir o rollback usando ```ROLLBACK TO SAVEPOINT nome-savepoint```.

### Exercícios
Abra duas sessões/tabs com scripts diferentes no DBeaver. Sugestão no topo adicione um comentário sessão1 ou sessão2
1. Verifique o funcionamento do autocommit. Execute os comandos pela ordem indicada:
  * Na sessão 1, desligue o autocommit
  * Na sessão 1, verifique o nome do empregado cujo id é 100
  * Na sessão 2, verifique o nome do empregado cujo id é 100
  * Na sessão 1, altere o nome do empregado para Cristiano Ronaldo e verifique que os dados foram atualizados
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?)
  * Na sessão 1, execute a transação e verifique o resultado
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?)
2. Volte a executar a sequência de passos descrita no exercício anterior mas no penúltimo passo reverta a transação em vez de executa-la.
3. Testar funcionamento de uma transação
  * Na sessão 1, volte a ligar o autocommit
  * Na sessão 1, inicie uma transação   
  * Na sessão 1, verifique o nome do empregado cujo id é 100
  * Na sessão 2, verifique o nome do empregado cujo id é 100
  * Na sessão 1, altere o nome do empregado para Fernando Santos e verifique que os dados foram atualizados
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?)
  * Na sessão 1, execute a transação e verifique o resultado
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?)
4. Volte a executar a sequência de passos descrita no exercício anterior mas no penúltimo passo reverta a transação em vez de executa-la.
5. Testar funcionamento do nível de isolamento
  * Na sessão 1, altere o nível de isolamento global para read uncommitted
  * Na sessão 1, inicie uma transação   
  * Na sessão 1, verifique o nome do empregado cujo id é 100
  * Na sessão 2, verifique o nome do empregado cujo id é 100
  * Na sessão 1, altere o nome do empregado para Diogo Costa e verifique que os dados foram atualizados
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?)
  * Na sessão 1, reverta a transação e verifique o resultado
  * Na sessão 2, verifique o nome do empregado cujo id é 100 (qual a conclusão?) 
6. Volte a executar a sequência de passos descrita no exercício anterior para verificar o resultado quando o nível de isolamento é read commited e repeatable read.


## 4. Ciclos WHILE e LOOP
(não há trabalho de casa devido a entega de trabalho de grupo)

### Exercícios
Abra duas sessões/tabs com scripts diferentes no DBeaver. Sugestão no topo adicione um comentário sessão1 ou sessão2


## 5. Trabalho de Casa
(não há trabalho de casa devido a entega de trabalho de grupo)


## 6. Resoluções
[Resolução dos exercícios em aula](https://github.com/ULHT-BD/aula13/blob/main/aula13_resolucao.sql)


## Bibliografia e Referências
* [Slides aula](https://github.com/ULHT-BD/aula13/blob/main/Aula13.pdf) 
* [Documentação PLSQL - tutorialspoint](https://www.tutorialspoint.com/plsql/index.htm)


## Outros
Para dúvidas e discussões pode juntar-se ao grupo slack da turma através do [link](
https://join.slack.com/t/ulht-bd/shared_invite/zt-1iyiki38n-ObLCdokAGUG5uLQAaJ1~fA) (atualizado 2022-11-03)
