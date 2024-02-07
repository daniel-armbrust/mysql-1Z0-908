# Instalação e configuração do servidor

## 5. Configurando o MySQL usando opções e arquivos de opções

Tanto o serviço do MySQL ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)) quanto seus clientes de linha de comando ([mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html), [mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html), etc), podem ser configurados/parametrizados basicamente de três maneiras diferentes, nessa ordem:

1. Variáveis de ambiente
2. Arquivo de configurações
3. Linha de comando

Lembrando que há uma ordem de precedência envolvida ao usar as opções de configuração. Ou seja, se uma determinada opção foi especificada em uma _variável de ambiente_ e também pela _linha de comando_, o valor da _linha de comando_ será usado pois é a última forma a ser inspecionada.

>_**__NOTA:__** Há uma exceção nessa ordem de precedência. O servidor MySQL ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)), irá processar por último o arquivo **mysqld-auto.cnf**, se este existir dentro do [diretório de dados (datadir)](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html). Ele tem precedência até mesmo sobre as opções de linha de comando._

### Variáveis de ambiente

Neste _[link](https://dev.mysql.com/doc/refman/8.0/en/environment-variables.html)_ é possível encontrar a lista das _[variáveis de ambiente](https://dev.mysql.com/doc/refman/8.0/en/environment-variables.html)_ disponíveis para utilização.

```
[opc@orl8 mysql]$ export MYSQL_PS1='servidor-1 >> '

[opc@orl8 mysql]$ bin/mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.36 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

servidor-1 >>
```

### Arquivo de opções/configurações

Por padrão, as opções de configuração são lidas através dos seguintes arquivos nesta ordem:

1. /etc/my.cnf
2. /etc/mysql/my.cnf
3. /usr/local/mysql/etc/my.cnf
4. ~/.my.cnf

Eu particularmente prefiro utilizar o diretório _"/etc/mysql"_ para incluír todos os arquivos de configuração do MySQL. Neste caso, o MySQL irá procurar pelo arquivo _"my.cnf"_ dentro do diretório _"/etc/mysql"_:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
socket  = /tmp/mysql.sock

[mysql]
default-character-set = latin1
```

O arquivo de configurações é divido em diferentes _seções_ que também são chamados de _grupos_ ou _cabeçalhos_. Cada seção representa diretamente um programa do MySQL que possui um conjunto específico de configurações.

![alt_text](/imgs/mysql-configfile-1.png "Arquivo de configuração - 1")

### \[mysqld\] e \[mysqld_safe\]

Em _"[Iniciar e parar o serviço do MySQL](/start-and-stop-mysql.md)"_, vimos que é possível iniciar o serviço do MySQL diretamente pelo binário _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ ou pelo script _[mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)_. Cada modo de inicialização lê as suas configuraçõesas de diferentes _seções_:

- Para o binário _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_, as suas configurações são lidas de qualquer uma das seções: _\[mysqld\]_, _\[mysql\_cluster\]_, _\[server\]_ ou _\[mysqld-8.0\]_

- Já para o script _[mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)_, as suas configurações são lidas de qualquer uma das seções: _\[mysqld\]_, _\[server\]_, _\[mysqld\_safe\]_ ou _\[safe\_mysqld\]_

Independente da forma que se inicializa, há diversos parâmetros que configuram o modo de operação do serviço MySQL. Para se obter a lista das opções disponíveis e seus valores, usa-se o comando abaixo:

```
[root@orl8 mysql]# bin/mysqld --verbose --help
...
tls-ciphersuites                                             (No default value)
tls-version                                                  TLSv1.2,TLSv1.3
tmp-table-size                                               16777216
tmpdir                                                       /tmp
transaction-alloc-block-size                                 8192
transaction-isolation                                        REPEATABLE-READ
transaction-prealloc-size                                    4096
transaction-read-only                                        FALSE
...
```

Estas opções podem ser transportadas para dentro do arquivo de configurações com seu valor modificado, se for o caso:

![alt_text](/imgs/mysql-configfile-2.png "Arquivo de configuração - 2")

Após reiniciar o serviço, é possível verificar o novo valor definido para o parâmetro _"tmpdir"_:

```
[root@orl8 mysql]# bin/mysqld --verbose --help
...
tls-ciphersuites                                             (No default value)
tls-version                                                  TLSv1.2,TLSv1.3
tmp-table-size                                               16777216
tmpdir                                                       /var/tmp
transaction-alloc-block-size                                 8192
transaction-isolation                                        REPEATABLE-READ
transaction-prealloc-size                                    4096
transaction-read-only                                        FALSE
...
```

Uma outra forma para se verificar esses valores é a partir do cliente de linha de comando _[mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)_, através da instrução abaixo:

```
mysql> SHOW VARIABLES \G
```

Ou, pelo _[mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)_:

```
[root@orl8 mysql]# bin/mysqladmin -p variables
```

Após qualquer alteração no arquivo de configurações, é importante validar seu conteúdo com o comando abaixo:

```
[root@orl8 mysql]# bin/mysqld --validate-config
```

Será exibido um erro caso o arquivo tenha qualquer opção ou valor inválido:

```
[root@orl8 mysql]# bin/mysqld --validate-config
2024-02-05T12:26:16.910193Z 0 [ERROR] [MY-000067] [Server] unknown variable 'diretorio_tmp=/var/tmp'.
2024-02-05T12:26:16.910225Z 0 [ERROR] [MY-010119] [Server] Aborting
```

É possível validar um arquivo de configuração que encontra-se em um local fora do padrão:

```
[root@orl8 mysql]# bin/mysqld --defaults-file=/opt/mysql/my.cnf --validate-config
```

>_**__NOTA:__** A opção --validade-config é particularmente útil quando se atualiza a versão do MySQL pois ela dirá, caso uma opção tenha sido removida na versão atualizada._