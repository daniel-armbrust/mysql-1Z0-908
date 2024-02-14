# 1. Instalação e configuração do servidor

## 1.7 - Inicie vários servidores MySQL no mesmo host

Múltiplas instâncias do processo _[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)_ podem ser executadas dentro de um único sistema operacional sendo que cada uma irá cuidar das suas conexões de rede e, gerenciar o seu próprio _[diretório de dados (datadir)](https://dev.mysql.com/doc/refman/8.0/en/data-directory.html)_.

![alt_text](/imgs/multi-mysqld-1.png "Múltiplos mysqld - 1")