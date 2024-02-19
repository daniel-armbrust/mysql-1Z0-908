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