# Instalação e configuração do servidor

## 5. Configurando o MySQL usando opções e arquivos de opções

Tando o serviço do MySQL ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)) quanto seus clientes de linha de comando ([mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html), [mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html), etc), podem ser configurados basicamente de duas maneiras diferentes:

1. Através de um arquivo de opções/configurações
2. Por argumentos/opções informados pela linha de comando

### Arquivo de opções/configurações

Por padrão, as opções de configuração são lidas através dos seguintes arquivos em ordem:

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

O arquivo de configurações é divido em _seções_, que também são chamados de _grupos_ ou _cabeçalhos_. Cada seção representa diretamente um binário do MySQL e este possui um conjunto específico de configurações.

![alt_text](/imgs/mysql-configfile-1.png "Arquivo de configuração - 1")

### \[mysqld\]

O serviço MySQL ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)) pode ter as suas configurações lidas de qualquer uma das seções nomeadas: _\[mysqld\]_, _\[mysql\_cluster\]_, _\[server\]_ ou _\[mysqld-8.0\]_

Ele possui diversas opções que podem ser definidas na sua inicialização que configuram o seu modo de operação. Para se obter a lista de opções e seus valores padrão, usa-se o comando abaixo:

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

Tais opções podem ser transportadas para dentro do arquivo de configurações com seu valor modificado, se for o caso:

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