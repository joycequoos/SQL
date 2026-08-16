# SQL

[← Voltar a Engenharia de Dados](https://github.com/joycequoos/Data_Enginer/blob/main/README.md)

Coleção de scripts SQL Server prontos para uso no dia a dia: criação e alteração de tabelas, views, procedures, functions, controle de permissões (grants), leitura de JSON, conversão de datas/formatos e monitoramento de processos ativos no banco.

## Índice

- [Tabelas](#tabelas)
- [Views](#views)
- [Procedures](#procedures)
- [Functions](#functions)
- [Grants e Permissões](#grants-e-permissões)
- [JSON no SQL Server](#json-no-sql-server)
- [Conversão de datas e formatos](#conversão-de-datas-e-formatos)
- [Monitoramento](#monitoramento)
- [Próximos passos](#próximos-passos)

---

## Tabelas

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Create_Tabel.sql](https://github.com/joycequoos/SQL/blob/main/Tables/01_Create_Tabel.sql) | `CREATE TABLE` condicional | Cria tabelas (temporária e definitiva) apenas se elas ainda não existirem, verificando `INFORMATION_SCHEMA.TABLES`, e define valores padrão (`DEFAULT`) para colunas como data de cadastro |
| [02_Alter_Table_addfield.sql](https://github.com/joycequoos/SQL/blob/main/Tables/02_Alter_Table_addfield.sql) | `ALTER TABLE ... ADD` condicional | Adiciona uma nova coluna a uma tabela existente somente se ela ainda não existir, consultando `sys.sysobjects` e `sys.all_columns` |
| [03_Drop_Table.sql](https://github.com/joycequoos/SQL/blob/main/Tables/03_Drop_Table.sql) | `DROP TABLE` condicional | Remove uma tabela (ou view) apenas se ela existir na base, evitando erro de "objeto inexistente" |
| [04_Drop_Field.sql](https://github.com/joycequoos/SQL/blob/main/Tables/04_Drop_Field.sql) | `ALTER TABLE ... DROP COLUMN` | Remove uma coluna específica de uma tabela |

> **Conceito-chave:** verificar a existência do objeto (`IF (NOT) EXISTS ...`) antes de criar, alterar ou remover é uma boa prática para tornar os scripts **idempotentes** — ou seja, podem ser executados mais de uma vez sem gerar erro.

## Views

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Create_View.sql](https://github.com/joycequoos/SQL/blob/main/02_Views/01_Create_View.sql) | `CREATE VIEW` condicional | Recria uma view (`VW_TPINVESTIDOR`), primeiro removendo (`DROP VIEW`) caso já exista, e depois a criando com colunas renomeadas a partir de uma tabela base |

> **Conceito-chave:** uma view é uma consulta salva que se comporta como uma tabela virtual — útil para simplificar consultas repetidas ou padronizar nomes de colunas para outros sistemas consumirem.

## Procedures

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Consulta_Conteudos_Procedures.sql](https://github.com/joycequoos/SQL/blob/main/Procedures/03_Consulta_Conteudos_Procedures.sql) | Metadados do banco | Consulta `sys.sql_modules` para encontrar o código-fonte de procedures que contenham um determinado trecho de texto (ex.: `qtd_dias`) — útil para localizar onde uma regra de negócio está implementada. Também mostra `sp_helptext`, outra forma de ver o código de uma procedure |
| [02_Create_Procedure.sql](https://github.com/joycequoos/SQL/blob/main/Procedures/05_Create_Procedure.sql) | Procedure com `TRY/CATCH` | Cria a procedure `spcb_valores_elevados`, que recebe dezenas de parâmetros (incluindo tabelas `readonly`) e gera alertas quando a movimentação financeira de um cliente ultrapassa determinados parâmetros, com regras diferentes para pessoa física e jurídica |
| [03_Procedure_Cursor.txt](https://github.com/joycequoos/SQL/blob/main/Procedures/07_Procedure_Cursor.txt) | `CURSOR` aninhado | Exemplo mais avançado: a procedure `spgr_varredura` usa múltiplos cursores aninhados (produtos → clientes → enquadramentos) para percorrer registros um a um e executar dinamicamente outras procedures (`sp_sqlexec`) — típico de rotinas de varredura/auditoria que processam grandes volumes de forma sequencial |
| [04_Procedure_Average_Calculation.sql](https://github.com/joycequoos/SQL/blob/main/Procedures/10_Procedure_Average_Calculation.sql) | Cálculo de média e geração de alerta | A procedure `spcc_aumento_vol` calcula a média mensal de créditos de um cliente em um período e gera um alerta quando a movimentação atual ultrapassa essa média em um percentual definido por parâmetro |

> **Conceito-chave:** todas as procedures usam o padrão `BEGIN TRY ... END TRY / BEGIN CATCH ... END CATCH`, chamando uma procedure central de tratamento de erro (`spgr_tratar_erro`) — uma boa prática para centralizar o log de falhas em vez de duplicar essa lógica em cada rotina.

## Functions

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Functions_Replace.sql](https://github.com/joycequoos/SQL/blob/main/Functions/06_Functions_Replace.sql) | `REPLACE()` encadeado | Remove pontos, hífens e barras de uma coluna de documentos, encadeando três `REPLACE()` — útil para padronizar CPF/CNPJ armazenados com máscara |
| [02_Function_DiaUtil.sql](https://github.com/joycequoos/SQL/blob/main/Functions/09_Function_DiaUtil.sql) | `FUNCTION` com laço `WHILE` | A function `fncDia_Util_Anterior` recebe uma data e uma quantidade de dias úteis, e retorna a data resultante subtraindo dias, pulando finais de semana e feriados cadastrados em `dbo.tgr_feriados` |

> **Conceito-chave:** diferente de uma procedure, uma `FUNCTION` sempre retorna um valor e pode ser usada diretamente dentro de outras consultas (ex.: `SELECT dbo.fncDia_Util_Anterior(...)`).

## Grants e Permissões

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Grants_Users.sql](https://github.com/joycequoos/SQL/blob/main/Grants/01_Grants_Users.sql) | `CREATE LOGIN` / `CREATE USER` / `GRANT` | Exemplo completo de controle de acesso: cria um banco de testes, dois logins com permissões diferentes (um com a role `bulkadmin`, outro apenas com `INSERT`), associa cada login a um usuário do banco e testa na prática a diferença — um consegue rodar `BULK INSERT`, o outro não |

> **Conceito-chave:** o princípio do **menor privilégio** — cada usuário deve ter apenas as permissões estritamente necessárias para sua função, reduzindo o risco de alterações indevidas.

## JSON no SQL Server

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Importar_Arquivo_JsonSQL.sql](https://github.com/joycequoos/SQL/blob/main/SQL_Json/01_Importar_Arquivo_JsonSQL.sql) | `OPENROWSET` + `OPENJSON` | Lê um arquivo `.json` do disco para uma variável (`OPENROWSET ... BULK`), converte esse JSON em linhas de tabela com `OPENJSON ... WITH (...)`, e no sentido inverso, transforma uma tabela em JSON usando `FOR JSON AUTO` |
| [02_Importar_Json_Git.sql](https://github.com/joycequoos/SQL/blob/main/SQL_Json/03_Importar_Json_Git.sql) | `OPENJSON` simples | Mesmo padrão de leitura de arquivo JSON, aplicado a um JSON mais simples (lista de logins), extraindo apenas a coluna `login` |

> **Conceito-chave:** o SQL Server nativo (desde a versão 2016) permite ler e gerar JSON sem precisar de uma ferramenta externa — útil para integrações onde o dado de origem ou destino é um arquivo JSON.

## Conversão de datas e formatos

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Convert_Substring.sql](https://github.com/joycequoos/SQL/blob/main/01_Convert_Substring.sql) | `SUBSTRING` + `OPENQUERY` | Procedure completa de carga (`spcr_operacao_corretora`): busca dados de um servidor vinculado (`OPENQUERY`) para tabelas temporárias, converte datas em formato texto para `smalldatetime` usando `SUBSTRING`, trata datas fora do intervalo suportado, e depois insere/atualiza as tabelas definitivas |
| [02_Convert_Datas.sql](https://github.com/joycequoos/SQL/blob/main/02_Convert_Datas.sql) | `CONVERT()` com diferentes estilos | Compara os principais estilos de formatação de data do `CONVERT()` (103, 113, 22, 111, 130, 109, 120) e mostra como converter datas em texto (`DD/MM/AAAA`, `DDMMAAAA`, `AAAAMMDD`) para `smalldatetime` usando `SUBSTRING` |
| [03_Tempo_Entre_DatasHoras.sql](https://github.com/joycequoos/SQL/blob/main/04_Tempo_Entre_DatasHoras.sql) | `DATEDIFF()` | Calcula a diferença entre duas datas em dias, horas, minutos e segundos, usando `DATEDIFF` com diferentes unidades de tempo |

> **Conceito-chave:** quando uma data vem como texto num formato não padrão (ex.: `14042021`), `SUBSTRING` é usado para "fatiar" o ano, mês e dia nas posições certas antes de converter para um tipo de data real.

## Monitoramento

| Script | Conceito | O que faz |
| --- | --- | --- |
| [01_Processos_Ativos.sql](https://github.com/joycequoos/SQL/blob/main/03_Processos_Ativos.sql) | `sys.sysprocesses` | Lista os processos ativos no SQL Server (`runnable` ou `suspended`), mostrando computador, usuário, status, se está bloqueado por outro processo e o aplicativo de origem — útil para identificar travamentos e consultas presas |

## Próximos passos

- Padronizar a nomenclatura dos scripts (hoje mistura numeração por pasta e por repositório inteiro).
- Adicionar exemplos de dados de entrada/saída para os scripts mais complexos (ex.: `07_Procedure_Cursor.txt`), facilitando o entendimento sem precisar rodar no banco.
- Documentar as tabelas e o modelo de dados (`ttp_`, `tgr_`, `tcl_`, etc.) referenciadas pelos scripts, já que os nomes seguem um padrão de prefixos.
- Revisar o uso de cursores (ex.: `07_Procedure_Cursor.txt`) para avaliar se alguma dessas rotinas poderia ser reescrita como uma operação baseada em conjunto (`set-based`), geralmente mais performática no SQL Server.
