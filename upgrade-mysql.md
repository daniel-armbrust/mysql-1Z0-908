# 1. Instalação e configuração do servidor

## 1.4 - Atualizar para o MySQL 8.0

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