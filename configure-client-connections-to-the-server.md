# 2. Arquitetura

## 2.1 - Configurar conexões cliente ao servidor

A arquitetura básica do MySQL é apresentada na figura abaixo:

![alt_text](/imgs/mysql-arch-1.png "Arquitetura - 1")

Independente do sistema operacional onde o MySQL foi instalado, o principal é a execução do processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_.

[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) é executado como um único processo _([daemon](https://pt.wikipedia.org/wiki/Daemon_(computa%C3%A7%C3%A3o)))_ que consegue lidar com múltiplas conexões _([multithreaded](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)))_, além de ser capaz de gerenciar a sua memória e o acesso aos bancos de dados armazenados em disco.

Programas clientes instalados localmente no servidor ou, em máquinas remotas da rede, se conectam ao processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ passando pela **_Connection Layer_** que manipulam os dados.

>_**__NOTA:__** Aqui, um "programa cliente" pode ser um cliente mysql especializado em manipular ou recuperar dados, quanto um programa administrativo para realizar qualquer tipo de manutenção no servidor MySQL._

O **_Storage Layer_** suporta diferentes _storage engines_ como o _[InnoDB](https://pt.wikipedia.org/wiki/InnoDB)_ ou _[MyISAM](https://pt.wikipedia.org/wiki/MyISAM)_, sendo estes os mais utilizados.

Múltiplas instâncias do processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_, podem ser executadas dentro de um único sistema operacional sendo que cada instância irá gerenciar as suas conexões e o seu próprio _[diretório de dados (datadir)](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html)_.

![alt_text](/imgs/mysql-arch-2.png "Arquitetura - 2")