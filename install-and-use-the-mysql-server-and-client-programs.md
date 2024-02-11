# Instalação e configuração do servidor

## 1.1 Instalar e utilizar o servidor MySQL e programas cliente

### Download 

A documentação oficial sobre a instalação do MySQL, pode ser encontrada no link: [https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)

![alt_text](/imgs/mysql-install-1.png "Instalação MySQL - 1")

![alt_text](/imgs/mysql-install-2.png "Instalação MySQL - 2")

Antes de seguir com a instalação em si, deve-se primeiramente determinar qual é o sistema operacional e a plataforma de instalação do MySQL.

- [Plataformas suportadas](https://www.mysql.com/support/supportedplatforms/database.html):
    - Oracle Linux / Red Hat / CentOS / Rocky Linux
    - Canonical Ubuntu (20, 22)
    - SUSE
    - Debian
    - Microsoft Windows
    - Oracle Solaris
    - Apple macOS

Há diferentes formas para se instalar o MySQL. É possível realizar a instalação através de binários prontos para download, pacotes RPM ou DEB e também é possível compilar os binários através do código-fonte.

1. Primeiro, deve-se realizar o download do MySQL acessando o link: [https://www.mysql.com/downloads/](https://www.mysql.com/downloads/)

2. A versão que utilizaremos para as demonstrações é a [MySQL Community (GPL)](https://dev.mysql.com/downloads/):

![alt_text](/imgs/mysql-install-3.png "Instalação MySQL - 3")

![alt_text](/imgs/mysql-install-4.png "Instalação MySQL - 4")

3. Deve-se selecioar a versão e o sistema operacional correspondente para o download:

![alt_text](/imgs/mysql-install-5.png "Instalação MySQL - 5")

4. Verificar a versão do _[GNU libc](https://en.wikipedia.org/wiki/Glibc)_ no sistema operacional Linux:

```
[darmbrust@orl8 ~]$ ldd --version
ldd (GNU libc) 2.28
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
```

5. A partir da versão retornada do _[GNU libc](https://en.wikipedia.org/wiki/Glibc)_ (2.28), selecionar o link correspondente para download:

![alt_text](/imgs/mysql-install-6.png "Instalação MySQL - 6")

![alt_text](/imgs/mysql-install-7.png "Instalação MySQL - 7")

6. Download concluído:

```
[darmbrust@orl8 ~]$ ls
mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
```

### Instalando o MySQL em Unix/Linux através de binários prontos para download

A documentação oficial deste processo de instalação, pode ser encontrado no link: _[https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)_

#### 1. Descompactar o arquivo baixado em /usr/local e criar um link simbólico:

```
[root@orl8 ~]# tar xf mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz -C /usr/local/

[root@orl8 ~]# ln -s /usr/local/mysql-8.0.36-linux-glibc2.28-x86_64/ /usr/local/mysql

[root@orl8 ~]# ls -ld /usr/local/mysql
lrwxrwxrwx. 1 root root 47 Jan 27 13:34 /usr/local/mysql -> /usr/local/mysql-8.0.36-linux-glibc2.28-x86_64/
```

#### 2. Criar um grupo e usuário que serão usados para executar o banco de dados:

```
[root@orl8 ~]# groupadd mysql
[root@orl8 ~]# useradd -r -g mysql -s /bin/false mysql
```

#### 3. Acessar o diretório do MySQL:

```
[root@orl8 ~]# cd /usr/local/mysql
```

#### 4. Criar o diretório _"mysql-files"_ e permissionar corretamente:

```
[root@orl8 mysql]# mkdir mysql-files
[root@orl8 mysql]# chown mysql:mysql mysql-files
[root@orl8 mysql]# chmod 750 mysql-files
```

#### 5. Inicializar o diretório de dados (datadir):

Após instalação dos arquivos, o _"diretório de dados (dadadir)"_ deve ser inicializado. Este procedimento só é necessário neste tipo de instalação, a partir de binários pré-compilados onde essa tarefa deve ser feita de forma manual.

```
[root@orl8 mysql]# bin/mysqld --initialize \
> --user=mysql \
> --basedir=/usr/local/mysql \
> --datadir=/usr/local/mysql/data
2024-01-27T14:12:13.683502Z 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.36-linux-glibc2.28-x86_64/bin/mysqld (mysqld 8.0.36) initializing of server in progress as process 7379
2024-01-27T14:12:13.695547Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-01-27T14:12:14.622429Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-01-27T14:12:16.916599Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: lixIJc(Zg0uo
```

Para este caso, foi especificado o diretório _"/usr/local/mysql/data"_ como sendo o local do _"diretório de dados"_ no qual o MySQL usou para criar e popular o _"[mysql System Schema](https://dev.mysql.com/doc/refman/8.0/en/system-schema.html)"_. Este incluí diversas tabelas importantes para o seu funcionamento (data dictionary tables, grant tables, time zone tables, e server-side help tables).

>_**__NOTA:__** Verifique que na última linha, após inicializar o "diretório de dados", uma senha temporária foi gerada para acessar o MySQL: lixIJc(Zg0uo_

#### 6. Iniciando o serviço do MySQL 

Utilizar o binário [mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html), é o meio recomendado para iniciar o servidor MySQL. Nele, há algumas funcionalidades como a de reiniciar o serviço automaticamente caso algum erro ocorra.

>_**__NOTA:__** Iniciar o MySQL através do [mysqld_safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html) em servidores Linux com suporte ao [systemd](https://docs.oracle.com/en/operating-systems/oracle-linux/9/osmanage/osmanage-WorkingWithSystemServices.html), não é necessário. Isto pelo fato do próprio [systemd](https://docs.oracle.com/en/operating-systems/oracle-linux/9/osmanage/osmanage-WorkingWithSystemServices.html) cuidar de reiniciar o serviço em caso de problemas._

```
[root@orl8 mysql]# bin/mysqld_safe --user=mysql &
[1] 8002
Logging to '/usr/local/mysql/data/orl8.err'.
2024-01-27T14:41:49.109542Z mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data
```

#### 7. Alterar a senha inicial e temporária do usuário root

Após iniciar o serviço, deve-se alterar a senha do usuário que foi gerada de forma temporária:

```
[root@orl8 mysql]# bin/mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.36

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER USER() IDENTIFIED BY 'Sup3rS3cr3t0!';
Query OK, 0 rows affected (0.01 sec)
```
>_**__NOTA:__** Não confundir a conta "root" do MySQL com a conta "root" do Sistema Operacional. Elas são contas diferentes._

É possível também, alterar essa senha através do comando _"mysqladmin"_:

```
[root@orl8 mysql]# bin/mysqladmin --user=root --password password
Enter password:
New password:
Confirm new password:
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
```

#### 8. (Opcional) Popular as tabelas de Time Zone

```
[root@orl8 mysql]# bin/mysql_tzinfo_to_sql /usr/share/zoneinfo | bin/mysql -u root mysql -p
```

#### 9. (Opcional) Verfificar a versão instalada

```
mysql> SELECT @@version;
+-----------+
| @@version |
+-----------+
| 8.0.36    |
+-----------+
1 row in set (0.00 sec)
```

### Executando o MySQL em um contêiner Docker

#### 1. Instalação do Docker

A documentação oficial sobre a instalação do _[Docker](https://developer.oracle.com/pt-BR/learn/technical-articles/what-is-docker)_ no _[Oracle Linux 8](https://docs.oracle.com/en/operating-systems/oracle-linux/8/)_ pode ser encontrada neste _[link aqui](https://oracle-base.com/articles/linux/docker-install-docker-on-oracle-linux-ol8)_.

```
[root@orl8 ~]# dnf install -y dnf-utils zip unzip

[root@orl8 ~]# dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

[root@orl8 ~]# dnf remove -y runc

[root@orl8 ~]# dnf install -y docker-ce --nobest
```

#### 2. Iniciando e habilitando o serviço do Docker

```
[root@orl8 ~]# systemctl enable docker.service
[root@orl8 ~]# systemctl start docker.service
```

#### 3. Download da imagem Docker

Há diversas imagens Docker do MySQL disponíveis para download neste _[link aqui](https://container-registry.oracle.com/ords/ocr/ba/mysql/community-server)_. O comando abaixo irá baixar versão _[8.0.36 Community](https://container-registry.oracle.com/ords/ocr/ba/mysql/community-server)_:

```
[root@orl8 ~]# docker pull container-registry.oracle.com/mysql/community-server:8.0.36
```

```
[root@orl8 ~]# docker images
REPOSITORY                                             TAG       IMAGE ID       CREATED       SIZE
container-registry.oracle.com/mysql/community-server   8.0.36    8a7d4390828d   12 days ago   661MB
```

#### 4. Iniciando o contêiner Docker do MySQL

Antes de iniciar o contêiner Docker do MySQL, certifique-se de que não exista outro serviço MySQL em execução. O comando abaixo executa um _"shutdown"_ no serviço MySQL,caso ele esteja em execução:

```
[root@orl8 mysql]# bin/mysqladmin -u root -p shutdown
Enter password:
```

Feito isso, basta iniciar o contêiner:

```
[root@orl8 ~]# docker run --net=host --name=mysql -d container-registry.oracle.com/mysql/community-server:8.0.36
```

Após iniciar o contêiner, deve-se alterar a senha do usuário root que foi gerada. Para obter a senha temporária gerada, utiliza-se o comando abaixo:

```
[root@orl8 ~]# docker logs mysql 2>&1 | grep "ROOT"
[Entrypoint] GENERATED ROOT PASSWORD: tQPB+4lm/n=Pi0^9^58j1bfS%8w0hv:,
```

Tendo a senha, é possível acessar o contêiner e realizar a alteração:

```
[root@orl8 mysql]# docker exec -it mysql /bin/bash
bash-4.4# bin/mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.36

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER USER() IDENTIFIED BY 'Sup3rS3cr3t0!';
Query OK, 0 rows affected (0.01 sec)

mysql>
```

