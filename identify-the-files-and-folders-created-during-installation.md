# Instalação e configuração do servidor

## 2. Identifique os arquivos e pastas criados durante a instalação

### Diretórios e arquivos instalados

- **/var/lib/mysql**
    - InnoDB log files, undo tablespaces, system tablespace
    - mysql.idb (tablespace do dicionário de dados)
    - Certificados SSL

- **/usr/sbin**
    - mysqld (servidor MySQL)

- **/usr/bin**
    - mysql, mysqldump (diversos programas clientes e scripts)

- **/var/lib/mysql-files**

- **/etc/my.cnf**
    - Arquivo de configuração do MySQL

- **/var/log/mysql/mysqld.log**
    - Error log

- **/usr/lib/systemd/system/mysqld.service**
    - Script Systemd de inicialização single instance

- **/usr/lib/systemd/system/mysqld@.service**
    - Script Systemd de inicialização multi instances

- **/var/lib/mysql-keyring**

- **mysql_secure_installation**
    - Programa que habilita algumas questões de segurança após a instalação do MySQL (remove o banco de dados de testes, define uma senha para o usuário root, etc.)

- **mysql_tzinfo_to_sql**
    - Utilitário para popular as tabelas de timezone.

- **mysql_upgrade**
    - Programa que verifica o conteúdo do banco de dados e garante que eles são compatíveis com a versão do MySQL.
    - A partir da versão 8.0.16, o MySQL executa essas verificações de forma automática na sua inicialização.

- **mysql_config_editor**
    - Utilitário para facilitar o login ao MySQL. Este irá criar um arquivo seguro no HOMEDIR do usuário que contém as suas credênciais para autenticação.

```
$ mysql_config_editor set \
    --login-path=login-path \
    --user=user \
    --password \
    --host=hostname

$ mysql_config_editor print --login-path=login-path

$ mysql_config_editor print --all

$ mysql_config_editor remove --login-path=login-path
```

- **mysqlbinlog**
    - Utilitário que exibe o conteúdo dos logs binários. Logs binários possibilitam executar uma recuperação total do banco de dados.

- **mysqldumpslow**
    - Utilitário que possibilita ler e sumarizar o conteúdo do "slow query log" (quais consultas estão levando mais tempo para trazer resultados ou quais consultas não estão utilizando índices, etc).

- **mysql_ssl_rsa_setup**
    - Utilitário usado para criar chaves e certificados TLS.

- **ibd2sdi**
    - Utilitário usado para extraír informações serializadas do dicionário contído em arquivos tablespace InnoDB (SDI - serialized dictionary information).

### Programas cliente de linha de comando (command-line client programs)

- **mysql**
    - Cliente MySQL de linha de comando.

- **mysqladmin**
    - Utilitário para monitorar, administrar e executar um _"shutdown"_ no MySQL.

- **mysqldump** e **mysqlpump**
    - Utilitários de backup para criar scripts SQL usados para restaurar a estrutura e conteúdo do banco de dados.

- **mysqlimport**
    - Utilitário usado para importar arquivo de dados delimitados.

- **mysqlslap**
    - Load emulation client.
    - Utilizado para testes de desempenho (benchmark).

- **mysqlshow**
    - Utilitário usado para exibir metadados de objetos do banco de dados.

- **mysqlcheck**
    - Utilitário para realizar checagem e otimização das tabelas.

- **mysqlsh**
    - MySQL Shell é um cliente de linha de comando avançado e editor de código para o servidor MySQL.

Ao usar um programa de linha de comando, é necessário se autenticar através das opções _--user (-u)_ e _--password (-p)_. Além dessas informações em alguns casos, é necessário especificar também a forma de conexão ao banco de dados através das opções _--host (-h)_ ou _--socket (-s)_.

## 3. Iniciar e parar o MySQL

### Iniciando o servidor

```
$ mysqld --user=mysql --datadir=/var/lib/mysql --socket=/tmp/mysql.sock
```

- Métodos que podem ser usados para iniciar o servidor MySQL:
    - Executar o binário _"mysqld"_ diretamente.
    - Executar o binário _"mysqld_safe"_.
    - Linux: _"systemctl start mysql"_ ou _"service mysqld start"_

- Métodos que podem ser usados para parar o servidor MySQL:
    - Linux: _"systemctl stop mysql"_ ou _"service mysqld stop"_
    - Executar o comando: _"mysqladmin shutdown"_
    - Executar a sentença SQL: _"SHUTDOWN"_

```
$ mysqladmin --login-path=admin shutdown
```

## 4. Atualizar para o MySQL 8.0

- Consultar sempre a documentação sobre Upgrade de versão:
    - [Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)

- MySQL Shell Upgrade Chcker Utility
    - Existe a função _"util.checkForServerUpgrade()"_ dentro do MySQL Shell que pode ser utilizada para verificar se é possível o upgrade de versão.
    - Verifica questões de compatibilidade ao atualizar a versão.
    - Suporta MySQL 5.7 e 8.0 (GA).

```
$ mysqlsh -- util checkForServerUpgrade  user@localhost:3306 \
    --target-version=8.0.18 --config-path=/etc/mysql/my.cnf
```

>_**__NOTA:__** A opção --target-version se refere a versão que este servidor MySQL pretende ser atualizado._

### Processo de atualização _"In-Place"_

- Método recomendado quando se deseja atualizar da versão 5.7 para  8.0:

1. Parar o servidor MySQL.
2. Substituír o binário _mysqld_ pelo binário de versão maior.
3. Iniciar o servidor MySQL da versão maior.
4. Para as versões anteriores a 8.0.16, é necessário executar o utilitário *mysql_upgrade*. Para a versão 8.0.16 e posteriores, a reinicialização do serviço irá realizar as tarefas de atualização de forma manual.

### Processo de atualização lógico

- Realizar um backup/restore:

1. Utilizar o utilitário _mysqldump_ para realizar um backup do banco de dados.
2. Instalar, inicializar e iniciar a nova versão do MySQL.
3. Realizar um _"restore"_ a partir do arquivo de _dump_ gerado.
4. Para as versões anteriores a versão 8.0.16, é necessário executar o utilitário *mysql_upgrade*. Para a versão 8.0.16 e posteriores, primeiramente deve-se executar um _shutdown_ no MySQL e iniciá-lo novamente usando a opção _--upgrade=FORCE_ para realizar as demais tarefas de atualização.

### mysql_upgrade

- Verifica se existe incompatibilidade em todas as tabelas do banco de dados.
- Repara qualquer problema encontrado nas tabelas (se existir problema de incompatibilidade).
- Atualiza as tabelas de sistema para adicionar novos privilégios ou novas funcionalidades disponíveis na nova versão.
- Este utilitário não é necessário em versões do MySQL >= 8.0.16. A nova opção _--upgrade_ foi introduzida na versão 8.0.16 para controlar o processo de atualização.

## 5. Configurar o MySQL através de opções e arquivo de opções

## 6. Configurar as variáveis do MySQL

## 7. Iniciar múltiplos servidores MySQL no mesmo host