# Instalação e configuração do servidor

## 3. Iniciar e parar o serviço do MySQL

### Iniciando e parando o serviço do MySQL

Para iniciar o serviço do MySQL:

```
[root@orl8 mysql]# bin/mysqld_safe --user=mysql \
    --basedir=/usr/local/mysql \
    --datadir=/usr/local/mysql/data &
```

Para parar o serviço do MySQL:

```
[root@orl8 mysql]# bin/mysqladmin -u root -p shutdown
Enter password:
```

ou

```
mysql> SHUTDOWN;
Query OK, 0 rows affected (0.01 sec)
```

- Outros meios que podem ser usados para se iniciar o serviço do MySQL:
    - Executar o binário _"mysqld"_ diretamente.    
    - Em instalações através do RPM/DEB: _"systemctl start mysql"_ ou _"service mysqld start"_

- Outros meios que podem ser usados para parar o serviço do MySQL:    
    - Executar o comando: _"mysqladmin shutdown"_
    - Executar a sentença SQL: _"SHUTDOWN"_
    - Em instalações através do RPM/DEB: _"systemctl stop mysql"_ ou _"service mysqld stop"_

>_**__NOTA:__** A documentação oficial sobre o conteúdo dessa seção pode ser acessada por este [link aqui](https://dev.mysql.com/doc/refman/8.0/en/starting-server.html)._


### Testando o serviço do MySQL

Os comandos abaixo executam testes simples e podem ser utilizados para verificar o serviço do MySQL:

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