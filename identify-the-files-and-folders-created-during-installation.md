# Instalação e configuração do servidor

## 2. Identifique os arquivos e pastas criados durante a instalação

De acordo com a instalação feita na seção _["Instalar e utilizar o servidor MySQL e programas cliente"](/install-and-use-the-mysql-server-and-client-programs.md)_, todos os arquivos do MySQL estão localizados no diretório _"/usr/local/mysql"_.

Começarei descrevendo o conteúdo do diretório _"/usr/local/mysql/bin"_:

```
[root@orl8 mysql]# ls bin/
ibd2sdi         my_print_defaults    mysqld         mysql_migrate_keyring      mysql_upgrade
innochecksum    mysql                mysqld-debug   mysqlpump                  perror
lz4_decompress  mysqladmin           mysqld_multi   mysql_secure_installation  zlib_decompress
myisamchk       mysqlbinlog          mysqld_safe    mysqlshow
myisam_ftdump   mysqlcheck           mysqldump      mysqlslap
myisamlog       mysql_config         mysqldumpslow  mysql_ssl_rsa_setup
myisampack      mysql_config_editor  mysqlimport    mysql_tzinfo_to_sql
``` 

>_**__NOTA:__** A documentação oficial sobre o conteúdo dessa seção pode ser acessada por este [link aqui](https://dev.mysql.com/doc/refman/8.0/en/programs-overview.html)._

### Programa principal

- **[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)**
    - Servidor MySQL (ou SQL daemon) é o programa principal que executa a maior parte do trabalho.
    - Ele gerencia o acesso ao diretório de dados (datadir) no qual contém banco de dados e tabelas.

- **[mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)**
    - Shell script usado para executar o _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_.
    - Script usado como meio recomendado para iniciar o MySQL em ambientes Unix/Linux pois adiciona alguns recursos de segurança, como o de reiniciar serviço em caso de erro.

### Programas usados na pós-instalação

- **[mysql_secure_installation](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html)**
    - Programa que habilita algumas questões de segurança após a instalação do MySQL (remove o banco de dados de testes, define uma senha para o usuário root, etc.)

- **[mysql_tzinfo_to_sql](https://dev.mysql.com/doc/refman/8.0/en/mysql-tzinfo-to-sql.html)**
    - Utilitário para popular as tabelas de timezone.

- **[mysql_ssl_rsa_setup](https://dev.mysql.com/doc/refman/8.0/en/mysql-ssl-rsa-setup.html)**
    - Utilitário para criar um par de chaves RSA usados em conexão segura ao serviço do MySQL.
    - Este utilitário foi descontinuado na versão 8.0.34. Deve-se utilizar o próprio servidor MySQL para gerar os arquivos SSL/RSA.

- **[mysql_upgrade](https://dev.mysql.com/doc/refman/8.0/en/mysql-upgrade.html)**
    - Programa que verifica o conteúdo do banco de dados e garante que eles são compatíveis com a versão do MySQL.
    - A partir da versão 8.0.16, o MySQL executa essas verificações de forma automática na sua inicialização.

### Programas cliente de linha de comando (command-line client programs)

Ao usar um programa de linha de comando, é necessário se autenticar através das opções _--user (-u)_ e _--password (-p)_. Além dessas informações em alguns casos, é necessário especificar também a forma de conexão ao banco de dados através das opções _--host (-h)_ ou _--socket (-s).

- **[mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)**
    - Cliente de linha de comando usado para se conectar no serviço Mysql (SQL shell).

- **[mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)**
    - Utilitário para tarefas administrativas (monitorar e administrar um serviço MySQL).

- **[mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)** 
    - Utilitário que executa _[backups lógicos](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_logical_backup)_ (gera como saída sentenças SQL como CREATE TABLE e INSERT).
    - Gera como saída um DUMP do banco de dados em formato SQL, texto ou XML.

- **[mysqlpump](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html)**
    - Utilitário similar ao [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) porém descontinuado a partir da versão 8.0.34.

- **[mysqlimport](https://dev.mysql.com/doc/refman/8.0/en/mysqlimport.html)**
    - Utilitário usado para importar um arquivo de dados delimitados em sua respectiva tabela no banco de dados (interface a sentença SQL LOAD DATA).    

- **[mysqlslap](https://dev.mysql.com/doc/refman/8.0/en/mysqlslap.html)**
    - Programa para diagnóstico e projetado para emular testes de carga/desempenho ao servidor MySQL (benchmark).
    - Funciona como se vários clientes estivessem acessando o servidor. 

- **[mysqlshow](https://dev.mysql.com/doc/refman/8.0/en/mysqlshow.html)**
    - Utilitário usado para exibir metadados de objetos do banco de dados (tabelas, colunas, indíces e banco de dados).
    - Funciona como uma interface para o comando SQL SHOW.

- **[mysqlcheck](https://dev.mysql.com/doc/refman/8.0/en/mysqlcheck.html)**
    - Utilitário para realizar checagem e otimização das tabelas (verifica, repara, otimiza ou analisa tabelas).

- **[mysql_config_editor](https://dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html)**
    - Utilitário para facilitar o login ao MySQL. Este irá criar um arquivo seguro no HOMEDIR do usuário que contém as suas credênciais para autenticação.

### Demais utilitários e programas administrativos

- **[mysqlbinlog](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html)**
    - Utilitário que exibe o conteúdo dos logs binários. Logs binários são eventos que descrevem as modificações no banco de dados e possibilitam executar uma recuperação total do mesmo.

- **[mysqldumpslow](https://dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html)**
    - Utilitário que exibe informações sobre as consultas que demoram muito para processar e retornar resultado (slow query log).

- **[ibd2sdi](https://dev.mysql.com/doc/refman/8.0/en/ibd2sdi.html)**
    - Utilitário usado para extraír informações serializadas do dicionário contído em arquivos tablespace InnoDB (SDI - serialized dictionary information).

### MySQL Shell

- **[mysqlsh](https://dev.mysql.com/doc/mysql-shell/8.0/en/)**
    - MySQL Shell é um cliente de linha de comando avançado e editor de código para o servidor MySQL.

### Diretórios

O processo de instalação do MySQL através de pacotes _[RPM](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-rpm.html)_ cria e utiliza alguns diretórios do Sistema Operacional. São eles:

- **[/var/lib/mysql](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html)**
    - Diretório de Dados ou datadir (MySQL Data Directory).
    - InnoDB log files, undo tablespaces, system tablespace.
    - mysql.idb (tablespace do dicionário de dados).
    - Certificados SSL.

- **/usr/sbin**
    - Diretório onde reside o binário [mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) (servidor MySQL).

- **/usr/bin**
    - Diretório onde residem os diversos programas clientes e scripts do MySQL ([mysql](https://dev.mysql.com/doc/refman/8.0/en/mysql.html), [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html), etc).

- **/etc/my.cnf**
    - Arquivo de configuração do MySQL

- **/var/log/mysql/mysqld.log**
    - Arquivo que contém os logs de erro do MySQL.

- **/usr/lib/systemd/system/mysqld.service**
    - Script Systemd de inicialização single instance

- **/usr/lib/systemd/system/mysqld@.service**
    - Script Systemd de inicialização multi instances

- **/var/lib/mysql-keyring**

- **/var/lib/mysql-files**