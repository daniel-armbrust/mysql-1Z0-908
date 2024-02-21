# 2. Arquitetura

## 2.2 - Configurar conexões cliente ao servidor

Aqui será apresentado uma visão geral de como o MySQL gerencia as conexões dos clientes na **_Connection Layer_**. 

### Connection Layer

Os clientes podem se conectar ao servidor MySQL através da rede, utilizando o protocolo _TCP/IP_ ou, via _[Unix Domain Socket](https://pt.wikipedia.org/wiki/Soquete_de_dom%C3%ADnio_Unix)_ que é um arquivo especial no sistema operacional Linux que possibilita a comunicação de dois processos dentro do mesmo host (cliente e servidor MySQL).

As conexões dos clientes ao servidor são gerenciadas pela _Connection Layer_. Basicamente, o _Connection Manager_ associa a cada cliente uma _[thread](https://pt.wikipedia.org/wiki/Thread_(computa%C3%A7%C3%A3o))_ de conexão dedicada que irá cuidar da autenticação e processar as suas requisições.

A quantidade máxima de conexões clientes simultâneas permitidas pelo servidor, é controlada pela variável de sistema _[max\_connections](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connections)_. Se o servidor recusar uma conexão porque o limite _max\_connections_ foi atingido, a variável de status _[Connection\_errors\_max\_connections](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html#statvar_Connection_errors_max_connections)_ será incrementada para cada conexão recusada.

```
mysql> SHOW STATUS WHERE variable_name = 'Connection_errors_max_connections';
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| Connection_errors_max_connections | 0     |
+-----------------------------------+-------+
1 row in set (0.00 sec)
```

Independente da forma de conexão do cliente ao servidor, via protocolo _TCP/IP_ ou _Unix Domain Socket_, a _Connection Layer_ sempre irá disponibilizar uma _thread_ para tratar a conexão do cliente. Lembrando que, a atividade de criar novas _threads_ é uma operação custosa em termos computacionais.

Para aumentar o desempenho, o MySQL trabalha internamente com uma memória _[cache](https://pt.wikipedia.org/wiki/Cache)_ com o propósito de armazenar algumas _threads_ de conexão.

Quando um cliente encerra a sua conexão, o próprio MySQL guarda aquela _thread_ no **_thread cache_** para que essa seja reutilizada por um novo cliente no futuro. O papel do _thread cache_ é armazenar _threads de conexão_ para que essas sejam reutilizadas por outros novos clientes, evitando quando possível a criação de novas _threads_.

A variável de sistema _[thread_cache_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_cache_size)_ controla a quantidade de _threads_ que ficam armazenadas em _cache_ para reutilização futura. Por padrão, o seu valor é automaticamente calculado na inicialização do MySQL. Um valor igual a 0, desabilita completamente o _cache_ e faz com que a _Connection Layer_ sempre crie uma nova _thread_ para atender um cliente.

>_**__NOTA:__** Lembre-se de que novas threads de conexão são sempre criadas até o limite especificado por "max\_connections". A ideia do "thread cache" é reutilizar threads, evitando quando possível a criação de novas threads de conexão._

![alt_text](/imgs/mysql-connmgr-1.png "Connection Manager - 1")

Toda _thread_ consome memória do servidor. Por isso, ter muitas _threads_ em _cache_ sem utilização irá desperdiçar memória. 

A variável de ambiente _[thread_stack](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_stack)_ controla o _[tamanho da pilha em memória (stack size)](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)_ usada por uma _thread_. Ajustar essa variável para um valor muito pequeno, irá limitar a complexidade das instruções SQL que o servidor pode tratar. O valor padrão é adequado na maioria dos casos.

O comando abaixo monitoram as _threads_ gerenciadas pelo servidor MySQL:

```
mysql> SHOW STATUS WHERE variable_name LIKE 'Threads%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 0     |
| Threads_connected | 1     |
| Threads_created   | 1     |
| Threads_running   | 2     |
+-------------------+-------+
4 rows in set (0.00 sec)
```

### Enterprise Thread Pool plugin

O _[MySQL Enterprise Edition](https://www.mysql.com/products/enterprise/)_ vem equipado com um plugin chamado _[MySQL Enterprise Thread Pool](https://dev.mysql.com/doc/refman/8.0/en/thread-pool.html)_ que melhora o gerenciamento das _threads_ pelo servidor MySQL. 

De um modo geral, esse plugin irá gerenciar as conexões clientes em _thread groups_ e reduzir o número de _threads_ criadas pelo servidor. Como resultado, há um aumento no desempenho do servidor.

### Protocolos de Conexão

#### TCP/IP

O protocolo TCP/IP é o mais usado para se comunicar com o servidor MySQL. A partir da versão 5.7.12, há dois protocolos disponíveis para conexões clientes:

- [MySQL Classic Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_PROTOCOL.html) (ou legacy protocol):
    - Utiliza por padrão a porta 3306/TCP.

- [MySQL X Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/page_mysqlx_protocol.html):
    - Introduzido na versão 5.7.12 e é habilitado por padrão na versão 8.
    - É um novo protocolo que permite comunicação assíncrona e de alto desempenho. Uma das suas vantagens é poder tratar de grandes conjuntos de dados de uma maneira mais eficiente.
    - Utiliza por padrão a porta 33060/TCP.

Cada protocolo tem o seu próprio conjunto de opções:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
bind_address = 10.0.0.72
port = 3306

mysqlx_bind_address = 127.0.0.1
mysqlx_port = 33060
```

Para desabilitar o _[X Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/page_mysqlx_protocol.html)_, insira _"mysqlx = off"_ no arquivo de configurações do MySQL:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
bind_address = 10.0.0.72
port = 3306

mysqlx = off
```

O cliente de linha de comando _[mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)_ suporta diferentes opções para conexão com o servidor:

```
[root@orl8 mysql]# bin/mysql -h 127.0.0.1 --protocol=TCP -P 3306 -u root -p
```

Para interção com o _[X Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/page_mysqlx_protocol.html)_, utilize o _[MySQL Shell 8.0](https://dev.mysql.com/doc/mysql-shell/8.0/en/)_ documentado posteriormente.

#### Unix Domain Socket

Conforme já explicado, o _[Unix Domain Socket](https://pt.wikipedia.org/wiki/Soquete_de_dom%C3%ADnio_Unix)_ é um arquivo especial criado pelo MySQL no sistema operacional Linux que possibilita a comunicação de dois processos dentro do mesmo host (cliente e servidor MySQL).

Uma comunicação que utiliza _[Unix Domain Socket](https://pt.wikipedia.org/wiki/Soquete_de_dom%C3%ADnio_Unix)_ é mais performática em comparação a uma comunicação através do protocolo _TCP/IP_. Porém, essa comunicação só é possível entre cliente e servidor dentro do mesmo sistema operacional.

>_**__NOTA:__** Consulte este [link](https://lists.freebsd.org/pipermail/freebsd-performance/2005-February/001143.html) para uma comparação entre ["unix domain sockets vs.internet sockets"](https://lists.freebsd.org/pipermail/freebsd-performance/2005-February/001143.html)._

As opções _[socket](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_socket)_ e _[mysqlx_socket](https://dev.mysql.com/doc/refman/8.0/en/x-plugin-options-system-variables.html#sysvar_mysqlx_socket)_, controlam a criação do arquivo no sistema operacional Linux para o _[Classic Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_PROTOCOL.html)_ e _[X Protocol](https://dev.mysql.com/doc/dev/mysql-server/latest/page_mysqlx_protocol.html)_ respectivamente:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
socket = /tmp/mysql.sock
mysqlx_socket = /tmp/mysqlx.sock
```

A conexão ao servidor através do _[Unix Domain Socket](https://pt.wikipedia.org/wiki/Soquete_de_dom%C3%ADnio_Unix)_, pode ser feita pelo comando abaixo:

```
[root@orl8 mysql]# bin/mysql -S /tmp/mysql.sock -u root -p
```

### Conexões Criptografadas via TLS

Para conexões através do protocolo TCP/IP, a _Connection Layer_ pode ser configurada para aceitar criptografia usando o protocolo _[TLS (Transport Layer Security)](https://pt.wikipedia.org/wiki/Transport_Layer_Security)_. 

O _TLS_ usa algoritmos de criptografia para garantir que os dados recebidos por uma rede pública sejam confiáveis. Ele possui mecanismos para detectar alteração, perda, reprodução falsa dos dados _[(replay attack)](https://en.wikipedia.org/wiki/Replay_attack)_ e também, incorpora algoritmos que fornecem verificação de identidade usando o padrão _[X.509](https://en.wikipedia.org/wiki/X.509)_.

>_**__NOTA:__** O protocolo TLS (Transport Layer Security) é o sucessor do antigo protocolo SSL (Secure Sockets Layer). O MySQL não utiliza o protocolo SSL pois a sua criptografia é fraca._

A funcionalidade de criptografia é fornecido pela biblioteca _[OpenSSL](https://www.openssl.org/)_ e o comando abaixo, verifica se a instalação do MySQL possui tal suporte (disponível a partir do MySQL 8.0.30):

```
mysql> SHOW STATUS WHERE variable_name = 'Tls_library_version';
+---------------------+----------------------------+
| Variable_name       | Value                      |
+---------------------+----------------------------+
| Tls_library_version | OpenSSL 3.0.12 24 Oct 2023 |
+---------------------+----------------------------+
1 row in set (0.00 sec)
```

Um servidor MySQL com suporte ao _OpenSSL_ irá automaticamente gerar um conjunto de chaves e certificados no _[diretório de dados (datadir)](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html)_ na inicialização do _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_:

```
[root@orl8 mysql]# ls -1 data/*.pem
data/ca-key.pem
data/ca.pem
data/client-cert.pem
data/client-key.pem
data/private_key.pem
data/public_key.pem
data/server-cert.pem
data/server-key.pem
```

A variável de sistema _[auto_generate_certs](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_auto_generate_certs)_ controla a geração automática dessas chaves e certificados.

Os certificados e as chaves geradas automaticamente são:

- ca.pem
    - Certificado da CA auto-assinado.

- ca-key.pem
    - Chave privada da CA.

- server-cert.pem
    - Certificado do servidor MySQL.

- server-key.pem
    - Chave privada do servidor MySQL.

- client-cert.pem 
    - Certificado dos clientes MySQL.

- client-key.pem 
    - Chave privada dos clientes MySQL.

- private_key.pem 
    - Membro privado do par de chaves privada/pública.

- public_key
    - Membro público do par de chaves privada/pública.

Todas esses arquivos são gerados no formato _[PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail)_ (formato obrigatório) e as informações de um certificado, podem ser visualizadas através do próprio _OpenSSL_:

```
[root@orl8 mysql]# openssl x509 -text -in data/server-cert.pem
```

Ao utilizar certificados digitais, é importante checar a sua data de expiração:

```
mysql> SHOW STATUS LIKE 'Ssl_server_not%';
+-----------------------+--------------------------+
| Variable_name         | Value                    |
+-----------------------+--------------------------+
| Ssl_server_not_after  | Feb 17 12:26:31 2034 GMT |
| Ssl_server_not_before | Feb 20 12:26:31 2024 GMT |
+-----------------------+--------------------------+
2 rows in set (0.03 sec)
```

O utilitário de linha de comando _[mysql_ssl_rsa_setup](https://dev.mysql.com/doc/refman/8.0/en/mysql-ssl-rsa-setup.html)_ utiliza o _OpenSSL_ para criar novas chaves e certificados caso os antigos tenham sido expirados:

```
[root@orl8 mysql]# bin/mysql_ssl_rsa_setup --datadir=data/
WARNING: mysql_ssl_rsa_setup is deprecated and will be removed in a future version. Use the mysqld server instead.
Ignoring -days; not generating a certificate
Generating a RSA private key
......+++++
...................................+++++
writing new private key to 'ca-key.pem'
-----
Ignoring -days; not generating a certificate
Generating a RSA private key
..................+++++
...+++++
writing new private key to 'server-key.pem'
-----
Ignoring -days; not generating a certificate
Generating a RSA private key
.............+++++
..........................+++++
writing new private key to 'client-key.pem'
-----
```

Através da variável de sistema _[require_secure_transport](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_require_secure_transport)_, é possível exigir somente conexões criptografadas ao servidor MySQL:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
require_secure_transport = on
ssl_ca = /usr/local/mysql/data/ca.pem
ssl_cert = /usr/local/mysql/data/server-cert.pem
ssl_key = /usr/local/mysql/data/server-key.pem

[mysql]
ssl_ca = /usr/local/mysql/data/ca.pem
ssl_cert = /usr/local/mysql/data/client-cert.pem
ssl_key = /usr/local/mysql/data/client-key.pem
```

Ou, diretamente através do _[mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)_:

```
mysql> SET PERSIST require_secure_transport = on;
Query OK, 0 rows affected (0.00 sec)
```

Ao tentar se conectar sem usar criptografia, através da opção _[ssl-mode](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-mode)_, irá resultar em um erro:

```
[root@orl8 mysql]# bin/mysql --ssl-mode=DISABLED
WARNING: no verification of server certificate will be done. Use --ssl-mode=VERIFY_CA or VERIFY_IDENTITY.
ERROR 3159 (HY000): Connections using insecure transport are prohibited while --require_secure_transport=ON.
```

Uma vez conectado, o uso ou não do _SSL_ pode ser verificado através do comando _SESSION_ ou _\s_:

```
mysql> \s
--------------
bin/mysql  Ver 8.0.36 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          11
Current database:
Current user:           root@localhost
SSL:                    Cipher in use is TLS_AES_256_GCM_SHA384
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.36 MySQL Community Server - GPL
Protocol version:       10
Connection:             127.0.0.1 via TCP/IP
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8mb4
Conn.  characterset:    utf8mb4
TCP port:               3306
Binary data as:         Hexadecimal
```

Conexões que utilizam criptografia com o servidor:

```
SSL: Cipher in use is TLS_AES_256_GCM_SHA384
```

Sem criptografia:

```
SSL: Not in use
```

Para configurar uma conta MySQL e forçar o uso da criptografia nas conexões, inclua uma cláusula _REQUIRE_ na instrução _[CREATE USER](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)_ conforme mostrado abaixo:

```
mysql> CREATE USER 'darmbrust'@'localhost' REQUIRE X509;
Query OK, 0 rows affected (0.01 sec)
```

### Acesso Administrativo

O servidor MySQL suporta uma única conexão administrativa além do limite definido pela opção _[max\_connections](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connections)_. 

A ideia é permitir que um usuário no qual tenha o privilégio _[CONNECTION_ADMIN](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_connection-admin)_, possa acessar o servidor em momentos de _alta utilização_ e realizar qualquer tarefa administrativa.

>_**__NOTA:__** Não há limite para o número de conexões administrativas, mas as conexões só são permitidas apenas para usuários que possuem o privilégio [SERVICE\_CONNECTION\_ADMIN](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_service-connection-admin)._

A partir da versão 8.0.14 do MySQL, é possível especificar uma _thread_ exclusiva _([create_admin_listener_thread](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_create_admin_listener_thread))_, um endereço IP _([admin_address](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_address))_ e porta _([admin_port](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_port))_ para o acesso administrativo:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
create_admin_listener_thread = on
admin_address = 127.0.0.1
admin_port = 33064
```

A conexão pode ser feita especificando a porta administrativa pelo cliente _[mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)_: 

```
[root@orl8 mysql]# bin/mysql --port=33064 --user=root -p
```

Para utilizar criptografia nas conexões administrativas, as opções _[admin-ssl](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_admin-ssl)_, [admin_ssl_ca](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_ssl_ca), [admin_ssl_cert](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_ssl_cert) e _[admin_ssl_key](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_admin_ssl_key)_ podem ser usadas:

```
[root@orl8 mysql]# cat /etc/mysql/my.cnf
[mysqld]
admin_address = 127.0.0.1
admin_port = 33064
admin-ssl = on
admin_ssl_ca = /usr/local/mysql/data/ca.pem
admin_ssl_cert = /usr/local/mysql/data/server-cert.pem
admin_ssl_key = /usr/local/mysql/data/server-key.pem
```

### IPv6