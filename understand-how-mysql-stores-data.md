# 2. Arquitetura

## 2.3 - Entender como o MySQL armazena os dados

Aqui será apresentado uma visão geral de como o MySQL executa as instruções SQL na **_SQL Layer_** e em como os dados são armazenados e recuperados na **_Storage Layer_**.

### SQL Layer

Depois de estabelecer a conexão e se autenticar com sucesso, uma _[instrução SQL](https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html)_ quando submetida passa pelas seguintes etapas dentro da _SQL Layer_:

1. [Analisador (Parser)](https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html)
2. [Autorização](https://dev.mysql.com/doc/refman/8.0/en/account-management-statements.html)
3. [Otimização](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
4. [Query Execution](https://dev.mysql.com/doc/refman/8.0/en/sql-data-manipulation-statements.html)
5. [Query Logging](https://dev.mysql.com/doc/refman/8.0/en/query-log.html)

A primeira etapa consiste em analisar a sintaxe da _[instrução SQL](https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html)_ recebida. Se a instrução estiver correta, o servidor verifica se o usuário conectado possui a devida _autorização_ para recuperar e/ou manipular os objetos no qual a instrução faz referência.

Logo após, o servidor aplica diversas _[otimizações](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)_ antes da sua execução. A função do _otimizador_ é criar um _[plano de execução](https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html)_ o mais eficiente possível, que basicamente envolve a escolha de quais _[índices](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)_ utilizar e, qual a ordem de processamento das tabelas. É possível especificar _"[hints](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html)"_ através de palavras-chaves especiais como forma de alterar o seu processo de decisão.

Depois de concluír as otimizações, a _instrução SQL_ é enfim executada. Opcionalmente é possível _[registrar em log](https://dev.mysql.com/doc/refman/8.0/en/server-logs.html)_ o que o servidor MySQL recebeu ou executou dos seus clientes.

![alt_text](/imgs/mysql-arch-3.png "Arquitetura - 3")

>_**__NOTA:__** Em versões antigas do MySQL, um cache interno era usado com o propósito de servir os resultados a partir dele. Porém, este cache foi desabilitado na versão 5.7.20 e removido por completo na versão 8 pelo fato dele prejudicar o desempenho do servidor. Mesmo um "query cache" não estar ativado por padrão, fazer cache dos resultados que são solicitados com frequência, ainda é uma boa prática. Soluções como [memcache](https://pt.wikipedia.org/wiki/Memcached) e [Redis](https://en.wikipedia.org/wiki/Redis), são bastante utilizadas._

### Storage Layer

Depois do processamento feito pela _SQL Layer_, o próximo passo é acessar os dados que estão no disco. Esse processo é feito pela _Storage Layer_ e seu componente principal chamado _[Storage Engine](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)_.

_Storage Engine_ é o componente que trata das operações SQL para diferentes tipos de tabelas. Ele é o software que processa como os dados são armazenados e recuperados da camada de storage.

Há diferentes tipos de _Storage Engine_ disponíveis para utilização sendo o _[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)_ o padrão e o recomendado a ser utilizado na maioria dos casos.

![alt_text](/imgs/mysql-arch-4.png "Arquitetura - 4")

>_**__NOTA:__** A Oracle recomenda utilizar o InnoDB na maioria dos casos. A instrução [CREATE TABLE](https://dev.mysql.com/doc/refman/8.0/en/create-table.html) usa o InnoDB por padrão._

O comando _[SHOW ENGINES](https://dev.mysql.com/doc/refman/8.0/en/show-engines.html)_ exibe as informações dos diferentes _Storage Engines_ suportados nesta instalação do MySQL:

```
mysql> SHOW ENGINES;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| ndbcluster         | NO      | Clustered, fault-tolerant tables                               | NULL         | NULL | NULL       |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| ndbinfo            | NO      | MySQL Cluster system information storage engine                | NULL         | NULL | NULL       |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
11 rows in set (0.00 sec)
```

Alguns dos _storage engines_ mais utilizados desde o surgimento do MySQL, são:

- **[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)**
    - É o _storage engine_ padrão na versão 8 do MySQL e o recomendado a ser utilizado na maioria dos casos.

- **[MyISAM](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html)**
    - Utilizado em antigas aplicações (legado).
    - Otimizado para aplicações que realizam muitas leituras dos dados (principalmente leitura).
    - Não possui suporte a _[chaves estrangeiras](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)_ e também não suporta _[transações](https://dev.mysql.com/doc/refman/8.0/en/commit.html)_.

- **[Memory](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html)**
    - Aqui os dados são armazenados na memória RAM (antigamente conhecido como _"HEAP Engine"_).
    - É rápido porém seu uso deve ser restrito para áreas de trabalho temporárias ou caches somente leitura.
    - Pelo fato dos dados estarem em memória RAM, caso o servidor MySQL reiniciar ou parar, tais dados serão perdidos.

- **[Archive](https://dev.mysql.com/doc/refman/8.0/en/archive-storage-engine.html)**
    - Usado em tabelas que armazenam grandes quantidades de dados.
    - As linhas são compactadas e descompactadas à medida que são inseridas e recuperadas nas tabelas.

- **[Blackhole](https://dev.mysql.com/doc/refman/8.0/en/blackhole-storage-engine.html)**
    - Age como um _"buraco negro"_ no qual aceita dados porém não os armazena.    
    - Toda recuperação de dados sempre retornam um resultado vazio.

- **[CSV](https://dev.mysql.com/doc/refman/8.0/en/csv-storage-engine.html)**
    - Armazena dados no formato CSV (valores separados por vírgula).

- **[NDBCLUSTER](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html)**
    - Banco de dados clusterizado é especialmente adequado para aplicativos que exigem o maior grau possível de disponibilidade.
    - Disponível nas distribuições _[MySQL NDB Cluster 8](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html)_.

É possível utilizar diferentes _storage engines_ em diferentes tabelas dentro de um mesmo _[schema](https://pt.wikipedia.org/wiki/Esquema_de_banco_de_dados)_. A palavra-chave _ENGINE_ da instrução _[CREATE TABLE](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)_, permite especificar o tipo de _storage engine_ que será usado por uma tabela:

```
mysql> CREATE TABLE usuarios (
    -> id INTEGER NOT NULL,
    -> nome VARCHAR(100) NOT NULL,
    -> cidade VARCHAR(50) NOT NULL
    -> ) ENGINE=CSV;
Query OK, 0 rows affected (0.01 sec)
```