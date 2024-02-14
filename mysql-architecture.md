# 2. Arquitetura

## 2.1 - Visão geral da Arquitetura do MySQL

A arquitetura básica do MySQL é apresentada na figura abaixo:

![alt_text](/imgs/mysql-arch-1.png "Arquitetura - 1")

### [mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)

Independente do sistema operacional onde o MySQL foi instalado, o processo principal é o _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_. O _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ é o servidor MySQL.

_[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ é executado como um único processo _([daemon](https://pt.wikipedia.org/wiki/Daemon_(computa%C3%A7%C3%A3o)))_ que consegue lidar com múltiplas conexões _([multithreaded](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)))_, além de ser capaz de gerenciar a sua memória e o acesso aos bancos de dados armazenados em disco. 

Ele possui um conjunto de _[variáveis de sistema (Server System Variables)](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)_ que podem ser alteradas para modificar a sua operação. Algumas variáveis devem ser definidas antes do processo iniciar e outras podem ser alteradas em _tempo de execução (runtime)_. Além disso, há outras _[variáveis de status (Server Status Variables)](https://dev.mysql.com/doc/refman/8.0/en/server-status-variable-reference.html)_ que fornecem informações sobre o seu funcionamento. 

Para se obter o valor padrão das variáveis de sistema utilizados pelo servidor, execute o comando abaixo:

```
[root@orl8 mysql]# bin/mysqld --verbose --help
```

ou

```
mysql> SHOW VARIABLES;

[root@orl8 mysql]# bin/mysqladmin variables -u root -p
```

Para visualizar dados estatísticos e status do servidor em execução:

```
mysql> SHOW STATUS;

[root@orl8 mysql]# bin/mysqladmin extended-status -u root -p
```

Informações de _status_ para todas as conexões ativas ao MySQL, podem ser obtidas através da palavra-chave _GLOBAL_:

```
mysql> SHOW GLOBAL STATUS;
```

Para se obter tais informações somente da sessão corrente, utilize a palavra-chave _SESSION_:

```
mysql> SHOW SESSION STATUS;
```

Múltiplas instâncias do processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ podem ser executadas dentro de um único sistema operacional sendo que cada uma irá cuidar das suas conexões de rede e, gerenciar o seu próprio _[diretório de dados (datadir)](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html)_.

![alt_text](/imgs/mysql-arch-2.png "Arquitetura - 2")

De um modo geral, é possível dividir o funcionamento do _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ em três diferentes camadas:

- Connection Layer
- SQL Layer
- Storage Layer

![alt_text](/imgs/mysql-arch-3.png "Arquitetura - 3")

### Connection Layer

Programas clientes instalados localmente no servidor ou, em máquinas remotas da rede, se conectam ao processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ passando pela **_Connection Layer_**.

>_**__NOTA:__** Aqui, um "programa cliente" pode ser um cliente mysql especializado em manipular e recuperar dados, quanto um programa administrativo para realizar qualquer tipo de manutenção no servidor MySQL._

### SQL Layer

### Storage Layer

O **_Storage Layer_** suporta diferentes _storage engines_ como o _[InnoDB](https://pt.wikipedia.org/wiki/InnoDB)_ ou _[MyISAM](https://pt.wikipedia.org/wiki/MyISAM)_, sendo estes os mais utilizados.

