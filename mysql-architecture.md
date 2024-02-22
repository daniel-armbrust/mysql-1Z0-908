# 2. Arquitetura

## 2.1 - Visão geral da Arquitetura do MySQL

A arquitetura básica do MySQL é apresentada na figura abaixo:

![alt_text](/imgs/mysql-arch-1.png "Arquitetura - 1")

De um modo geral, é possível dividir o funcionamento do _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ em três diferentes camadas:

- Connection Layer
    - Camada responsável pela autenticação e em gerenciar as conexões dos clientes ao servidor MySQL.

- SQL Layer
    - Camada que trata as instruções SQL (sintaxe, análise, otimização e execução).

- Storage Layer
    - Camada responsável pelo armazenamento e recuperação dos dados.

![alt_text](/imgs/mysql-arch-2.png "Arquitetura - 2")

### Connection Layer

Programas clientes instalados localmente no servidor ou, em máquinas remotas da rede, se conectam ao processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ passando pela **_Connection Layer_**.

>_**__NOTA:__** Aqui, um "programa cliente" pode ser um cliente mysql especializado em manipular e recuperar dados, quanto um programa administrativo para realizar qualquer tipo de manutenção no servidor._

### SQL Layer

A camada _SQL Layer_ é a camada que irá verificar se determinado usuário possui a devida autorização para recuperar ou manipular os dados, além de analisar e otimizar as instruções SQL. É nessa camada que também residem as _[stored procedures](https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html)_, _[triggers](https://dev.mysql.com/doc/refman/8.0/en/triggers.html)_ e _[views](https://dev.mysql.com/doc/refman/8.0/en/views.html)_.

### Storage Layer

O **_Storage Layer_** suporta diferentes _storage engines_ como o _[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html)_ ou _[MyISAM](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html)_, sendo estes os mais utilizados.

A comunicação do _SQL Layer_ para o _Storage Layer_ é através de API's bem definidas que escondem a implementação do _[storage engine](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)_ usado.