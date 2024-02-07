# Instalação e configuração do servidor

## 3. Iniciar e parar o serviço do MySQL

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