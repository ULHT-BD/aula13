# aula13
Nesta aula introduzimos as vantagens e utilização de PL/SQL. Falamos sobre estruturas básicas Stored Procedures, Stored Functions e Triggers e apresentamos clausulas para controlo de fluxo (if, case, while e loop).
Bom trabalho!

[0. Requisitos](#0-requisitos)

[1. Stored Procedures](#1-stored-procedures)

[2. Functions](#2-functions)

[3. Condições IF e CASE](#3-condições-if-e-case)

[4. Ciclos WHILE e LOOP](#4-ciclos-while-e-loop)

[5. Resoluções](#5-resoluções)

[Bibliografia e Referências](#bibliografia-e-referências)

[Outros](#outros)

## 0. Requisitos
Requisitos: Para esta aula, precisa de ter o ambiente de trabalho configurado ([Docker](https://www.docker.com/products/docker-desktop/) com [base de dados HR](https://github.com/ULHT-BD/aula03/blob/main/docker_db_aula03.zip) e [DBeaver](https://dbeaver.io/download/)). Caso ainda não o tenha feito, veja como fazer seguindo o passo 1 da [aula03](https://github.com/ULHT-BD/aula03/edit/main/README.md#1-prepare-o-seu-ambiente-de-trabalho).

Caso já tenha o docker pode iniciá-lo usando o comando ```docker start mysgbd``` no terminal, ou através do interface gráfico do docker-desktop:
<img width="1305" alt="image" src="https://user-images.githubusercontent.com/32137262/194916340-13af4c85-c282-4d98-a571-9c4f7b468bbb.png">

Deve também ter o cliente DBeaver.

## 1. Transação e Propriedades Acid
Uma transação consiste na execução de um conjunto de instruções que acede e possivelmente altera dados e que deve ser executada de forma atómica. Um exemplo de transação é a transferência de 50€ entre uma conta bancária A e outra conta bancária B uma vez que envolve uma sequência de instruções de leitura e escrita do saldo.

Exemplo:
```
Read(A)
A := A – 50
Write(A)
Read(B)
B := B + 50
Write(B)
```

O fluxo de execução de uma transação pode ser descrito por uma máquina de estados, composta por 5 estados, bem definidos:
<img width="577" alt="image" src="https://user-images.githubusercontent.com/32137262/207198610-ea64864c-bd67-4ccc-93fb-fa6589c16701.png">


Para assegurar a integridade dos dados, a BD deve garantir as propriedades ACID:

* Atomicidade – conjunto é realizado ou nada é realizado

* Consistência – execução de transações em isolamento preserva consistência de dados

* Isolamento –transações devem ignorar existência de transações concorrentes 

* Durabilidade – conclusão da transação implica dados persistentes


## 2. Anomalias e Níveis de Isolamento
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

## 3. Transações em SQL
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


## 4. Trabalho de Casa
(a publicar)

## 5. Resoluções
[Resolução dos exercícios em aula](https://github.com/ULHT-BD/aula11/blob/main/aula11_resolucao.sql)

## Bibliografia e Referências
* [Slides aula](https://github.com/ULHT-BD/aula12/blob/main/Aula12.pdf) 
* [Isolation Levels - Medium]((https://medium.com/analytics-vidhya/understanding-mysql-transaction-isolation-levels-by-example-1d56fce66b3d))


## Outros
Para dúvidas e discussões pode juntar-se ao grupo slack da turma através do [link](
https://join.slack.com/t/ulht-bd/shared_invite/zt-1iyiki38n-ObLCdokAGUG5uLQAaJ1~fA) (atualizado 2022-11-03)
