# 2. Arquitetura

## 2.1 - Visão geral da Arquitetura do MySQL

A arquitetura básica do MySQL é apresentada na figura abaixo:

![alt_text](/imgs/mysql-arch-1.png "Arquitetura - 1")

De um modo geral, é possível dividir o funcionamento do _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ em três diferentes camadas:

- Connection Layer
- SQL Layer
- Storage Layer

![alt_text](/imgs/mysql-arch-2.png "Arquitetura - 2")

### Connection Layer

Programas clientes instalados localmente no servidor ou, em máquinas remotas da rede, se conectam ao processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ passando pela **_Connection Layer_**.

>_**__NOTA:__** Aqui, um "programa cliente" pode ser um cliente mysql especializado em manipular e recuperar dados, quanto um programa administrativo para realizar qualquer tipo de manutenção no servidor MySQL._

### SQL Layer

### Storage Layer

O **_Storage Layer_** suporta diferentes _storage engines_ como o _[InnoDB](https://pt.wikipedia.org/wiki/InnoDB)_ ou _[MyISAM](https://pt.wikipedia.org/wiki/MyISAM)_, sendo estes os mais utilizados.

### [mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)

Independente do sistema operacional onde o MySQL foi instalado, o processo principal é o _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_. O _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ é o servidor MySQL.

_[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ é executado como um único processo _([daemon](https://pt.wikipedia.org/wiki/Daemon_(computa%C3%A7%C3%A3o)))_ que consegue lidar com múltiplas conexões _([multithreaded](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)))_, além de ser capaz de gerenciar a sua memória e o acesso aos bancos de dados armazenados em disco. Ou seja, ele é iniciado como um único processo que cria diversas outras _[threads](https://pt.wikipedia.org/wiki/Thread_(computa%C3%A7%C3%A3o))_ para servir as necessidades dos usuários.

O _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ possui um conjunto de _[variáveis de sistema (Server System Variables)](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)_ que podem ser alteradas para modificar a sua operação. Algumas variáveis devem ser definidas antes do processo iniciar e outras podem ser alteradas em _tempo de execução (runtime)_. Além disso, há outras _[variáveis de status (Server Status Variables)](https://dev.mysql.com/doc/refman/8.0/en/server-status-variable-reference.html)_ que fornecem informações sobre o seu funcionamento. 

Para se obter o valor padrão das _[variáveis de sistema](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)_, execute o comando abaixo:

```
[root@orl8 mysql]# bin/mysqld --verbose --help
```

Para se obter as _[variáveis de sistema](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)_ e seus valores que foram definidos para o servidor em execução, execute o comando abaixo:

```
[root@orl8 mysql]# bin/mysqladmin variables -u root -p
[root@orl8 mysql]# bin/mysqladmin extended-status -u root -p
```

_[Variáveis de sistema](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)_ possuem dois escopos diferentes:

- GLOBAL
    - Definem um valor que irá afetar a operação de todo o servidor MySQL.

```
mysql> SHOW GLOBAL STATUS;
```

- SESSION
    - Definem um valor que irá afetar somente uma sessão específica (uma sessão é uma conexão ativa no servidor). 

```
mysql> SHOW SESSION STATUS;
```

No caso do modificador não ser informado, _SESSION_ será usado por padrão. Observe o comando abaixo para visualizar dados estatísticos e _status_ da sessão ativa do servidor em execução:

```
mysql> SHOW STATUS;
```