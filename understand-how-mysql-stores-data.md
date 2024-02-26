# 2. Arquitetura

## 2.3 - Entender como o MySQL armazena os dados

Aqui será apresentado uma visão geral de como o MySQL executa as instruções SQL na **_SQL Layer_** e em como os dados são armazenados e recuperados na **_Storage Layer_**.

### SQL Layer

1. [Analisador (Parser)](https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html)
2. [Autorização](https://dev.mysql.com/doc/refman/8.0/en/account-management-statements.html)
3. [Otimização](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
4. [Query Execution](https://dev.mysql.com/doc/refman/8.0/en/sql-data-manipulation-statements.html)
5. [Query Logging](https://dev.mysql.com/doc/refman/8.0/en/query-log.html)

Depois de estabelecer a conexão e se autenticar com sucesso, a primeira etapa dentro do _SQL Layer_ consiste em analisar a sintaxe da _[instrução SQL](https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html)_ recebida. Se a instrução estiver correta, o servidor verifica se o usuário conectado possui a devida _autorização_ para recuperar e/ou manipular os objetos no qual a instrução faz referência.

Logo após, o servidor aplica diversas _[otimizações](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)_ antes da sua execução. A função do _otimizador_ é criar um _[plano de execução](https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html)_ o mais eficiente possível, que envolve a escolha de quais _[índices](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)_ utilizar e, qual a ordem de processamento das tabelas. É possível especificar _"[hints](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html)"_ através de palavras-chaves especiais como forma de alterar o seu processo de decisão.

Depois de concluír as otimizações, a _instrução SQL_ é enfim executada. Opcionalmente é possível registrar em log o que o servidor MySQL recebeu ou executou dos seus clientes.

![alt_text](/imgs/mysql-arch-3.png "Arquitetura - 3")

>_**__NOTA:__** Em versões antigas do MySQL, um cache interno era usado com o propósito de servir os resultados a partir dele. Porém, este cache foi desabilitado na versão 5.7.20 e removido por completo na versão 8 pelo fato dele prejudicar o desempenho do servidor. Mesmo um "query cache" não estar ativado por padrão, fazer cache dos resultados que são solicitados com frequência, ainda é uma boa prática. Soluções como [memcache](https://pt.wikipedia.org/wiki/Memcached) e [Redis](https://en.wikipedia.org/wiki/Redis), são bastante utilizadas._

### Storage Layer




