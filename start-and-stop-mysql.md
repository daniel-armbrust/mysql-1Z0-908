# 1. Instalação e configuração do servidor

## 1.3 - Iniciar e parar o serviço do MySQL

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

### Iniciando e parando o serviço do MySQL

Há duas formas usadas para iniciar o serviço do MySQL que irá depender da instalação feita. 

1. Através do script _[mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)_:

```
[root@orl8 mysql]# bin/mysqld_safe --user=mysql \
    --basedir=/usr/local/mysql \
    --datadir=/usr/local/mysql/data &
```

2. Diretamente através do binário _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_:

```
[root@orl8 mysql]# bin/mysqld --user=mysql \
> --basedir=/usr/local/mysql \
> --datadir=/usr/local/mysql/data &
```

>_**__NOTA:__** O método recomendado é iniciar o MySQL através do script [mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html). Instalações feitas por pacotes RPM/DEB, configuram a inicialização do serviço através do [systemd](https://dev.mysql.com/doc/refman/8.0/en/using-systemd.html) onde a inicialização pelo [mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html) não é necessário._

Para parar o serviço do MySQL:

```
[root@orl8 mysql]# bin/mysqladmin -u root -p shutdown
Enter password:
```

Ou:

```
mysql> SHUTDOWN;
Query OK, 0 rows affected (0.01 sec)
```

Em instalações feitas através de pacotes _RPM/DEB_, é possível iniciar o serviço pelo comando abaixo:

```
[root@orl8 ~]# systemctl start mysqld
```

Ou:

```
[root@orl6 ~]# service mysqld start
``` 

Para parar o serviço do MySQL:

```
[root@orl8 ~]# systemctl stop mysqld
```

Ou:

```
[root@orl6 ~]# service mysqld stop
```

>_**__NOTA:__** A documentação oficial sobre o conteúdo dessa seção pode ser acessada por este [link aqui](https://dev.mysql.com/doc/refman/8.0/en/starting-server.html)._


### Testando o serviço do MySQL

Os comandos abaixo executam testes simples e podem ser utilizados para verificar a execução correta do MySQL:

```
[root@orl8 mysql]# bin/mysqladmin -u root -p version

[root@orl8 mysql]# bin/mysqladmin -u root -p variables
```

```
[root@orl8 mysql]# bin/mysqlshow -u root -p
Enter password:
+--------------------+
|     Databases      |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

```
[root@orl8 mysql]# bin/mysqlshow -u root -p mysql
Enter password:
Database: mysql
+------------------------------------------------------+
|                        Tables                        |
+------------------------------------------------------+
| columns_priv                                         |
| component                                            |
| db                                                   |
| default_roles                                        |
| engine_cost                                          |
| func                                                 |
...
``` 

>_**__NOTA:__** A documentação oficial sobre o conteúdo dessa seção pode ser acessada por este [link aqui](https://dev.mysql.com/doc/refman/8.0/en/testing-server.html)._