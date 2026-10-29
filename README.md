# TP01 — Integração e Análise de Dados de Videojogos

**Autor:** Vitor Rezende — **N.º 31521**  
**Curso:** Licenciatura em Engenharia de Sistemas Informáticos  
**Unidade curricular:** Integração de Sistemas de Informação (2026/2027)  
**Ferramenta ETL:** Pentaho Data Integration (Kettle / Spoon)

**Repositório:** https://github.com/Shuchuga85/Trabalho1ETL_31521  
**Vídeo de demonstração:** https://vimeo.com/1238015643

## Objetivo

Este trabalho implementa um processo **ETL (Extract, Transform, Load)** para recolher e centralizar informações sobre videojogos provenientes de fontes diferentes. Os metadados dos jogos são obtidos através da **RAWG API**, enquanto os dados históricos de vendas são importados de um ficheiro **CSV**. Um ficheiro **XML** auxiliar é usado para uniformizar os géneros.

O **Pentaho Data Integration** limpa e transforma os dados, relaciona títulos entre as duas origens, guarda os resultados numa base de dados **Microsoft SQL Server** e gera relatórios CSV. Um **Job** coordena a execução das quatro transformações.

## Estrutura dos ficheiros

A estrutura abaixo mantém `jobs`, `transformations`, `logs` e `data`, para preservar os caminhos relativos usados no projeto.

```text
tp01-31521/
├── README.md
├── doc/
│   └── 31521_doc.pdf
├── jobs/
│   └── 00_main_job.kjb
├── transformations/
│   ├── 01_import_sales.ktr
│   ├── 02_import_rawg_paginated.ktr
│   ├── 03_integrate_games.ktr
│   └── 04_generate_statistics.ktr
├── logs/
│   ├── 01_import_sales.log
│   ├── 02_import_rawg_paginated.log
│   ├── 03_integrate_games.log
│   └── 04_generate_statistics.log
└── data/
    ├── input/
    │   └── vgsales.csv
    ├── reference/
    │   └── genre_mapping.xml
    └── output/
        ├── valid_sales.csv
        ├── invalid_sales.csv
        ├── genre_statistics.csv
        ├── integration_statistics.csv
        └── normalized_genre_statistics.csv
```

> Os ficheiros da pasta `data/output` são gerados ao executar as transformações. Podem ser incluídos no ZIP como exemplos de resultados. O SQL Server é externo aos ficheiros `.ktr` e `.kjb`, pelo que **a base de dados/tabelas também têm de existir** no ambiente onde o projeto for executado.

### Transformações e Job

| Ficheiro | Função |
|---|---|
| `01_import_sales.ktr` | Importa `vgsales.csv`, trata campos e valores inválidos, calcula métricas de vendas, exporta CSV e carrega `dbo.Staging_Sales`. |
| `02_import_rawg_paginated.ktr` | Consulta a RAWG API por páginas, extrai o JSON e carrega `dbo.Staging_RawgGames`. |
| `03_integrate_games.ktr` | Normaliza nomes com expressões regulares, cruza RAWG e vendas com `Merge Join` e carrega `dbo.Integrated_Games`. |
| `04_generate_statistics.ktr` | Calcula estatísticas por género e por estado de integração, usa o XML num `Stream Lookup` e exporta relatórios CSV. |
| `00_main_job.kjb` | Executa as quatro transformações pela ordem correta, com caminhos de sucesso e erro. |

## Fontes de dados

| Fonte | Formato | Ligação | Utilização |
|---|---|---|---|
| RAWG Video Games Database | REST / JSON | https://rawg.io/apidocs | Nome, data de lançamento, classificações e outros metadados dos jogos. Requer uma chave de API gratuita. |
| Video Game Sales (`vgsales.csv`) | CSV | https://zenodo.org/records/5898311/files/vgsales.csv?download=1 | Vendas históricas por jogo, plataforma e região. |
| `genre_mapping.xml` | XML local | Incluído em `data/reference` | Associa géneros originais a categorias normalizadas para o relatório final. |

**Documentação das ferramentas:** [Pentaho Kettle](https://github.com/pentaho/pentaho-kettle) · [Microsoft SQL Server](https://learn.microsoft.com/sql/).

## Requisitos para executar

- **Pentaho Data Integration (Kettle)**, através da aplicação **Spoon**.
- **Microsoft SQL Server**, com uma base de dados chamada `VideoGamesETL`.
- Um programa de administração SQL, por exemplo **SQL Server Management Studio (SSMS)**.
- **Driver JDBC do SQL Server** compatível com a instalação do Pentaho, caso não esteja incluído.
- Ligação à Internet e uma **chave de API RAWG** (obtida gratuitamente no site indicado acima).
- O ficheiro `vgsales.csv` e o ficheiro `genre_mapping.xml` nas pastas indicadas.

### Preparar a base de dados

A base de dados deve conter as tabelas `dbo.Staging_Sales`, `dbo.Staging_RawgGames` e `dbo.Integrated_Games`, com os campos esperados pelas transformações. A respetiva estrutura deve ser criada no SQL Server **antes** de executar o Job (por exemplo, a partir de um script SQL exportado do SSMS e incluído na entrega).

No Spoon, confirme ou adapte a ligação de base de dados denominada **`VideoGamesDB`** para o servidor SQL, porta, utilizador e palavra-passe do ambiente local. Não coloque credenciais pessoais no repositório.

## Como executar

1. Extraia o projeto, **sem alterar a organização relativa** das pastas `jobs/`, `transformations/` e `data/`.
2. Abra o Pentaho Spoon (`Spoon.bat`, em Windows, na pasta de instalação do Pentaho).
3. Abra `dataintegration/jobs/00_main_job.kjb` em **File → Open**.
4. Confirme que a ligação **`VideoGamesDB`** funciona e que a base de dados e as três tabelas estão criadas.
5. Confirme que o parâmetro **`RAWG_API_KEY`** é passado do Job para a transformação RAWG. Forneça o valor na janela de execução; não o guarde como valor por defeito nos ficheiros partilhados.
6. Clique em **Run** para executar o Job. A sequência esperada é: **Sales → RAWG → Integração → Estatísticas**.
7. Consulte os logs do Spoon. Se todas as etapas tiverem sucesso, verifique os dados no SQL Server e os ficheiros gerados na pasta `data/output`.
8. Confirme que a palavra passe e utilizador do envio de emails estão preenchidos.

**Nota sobre os caminhos:** os ficheiros originais utilizam caminhos relativos ao contexto de execução do Pentaho. A estrutura acima mantém `jobs`, `transformations` e `data` como pastas irmãs. Se mudar a estrutura, terá de ajustar os caminhos nos steps de entrada/saída e no Job. É recomendável testar o **Job completo a partir da pasta final de entrega**, antes de criar o ZIP.

### Verificação rápida no SQL Server

```sql
USE VideoGamesETL;

SELECT COUNT(*) AS TotalSalesRecords FROM dbo.Staging_Sales;
SELECT COUNT(*) AS TotalRawgGames FROM dbo.Staging_RawgGames;
SELECT
    COUNT(*) AS TotalGames,
    SUM(CASE WHEN has_sales_data = 1 THEN 1 ELSE 0 END) AS MatchedGames,
    SUM(CASE WHEN has_sales_data = 0 THEN 1 ELSE 0 END) AS UnmatchedGames
FROM dbo.Integrated_Games;
```

## Resultados do teste documentado

Com **5 páginas da RAWG**, de até 40 jogos por página, foram importados **200 jogos**. O cruzamento automático por nome normalizado associou **87 jogos** a dados históricos de vendas; **113** ficaram sem correspondência (**43,5%** de correspondências). São resultados deste ensaio, não um indicador de cobertura de toda a RAWG.

A pasta `data/output` contém os registos válidos e rejeitados e os relatórios de vendas por género, estado de integração e géneros normalizados via XML.

## Limitações

A associação entre as fontes usa principalmente o **nome normalizado** do videojogo. Diferenças entre edições, plataformas ou títulos podem criar correspondências incompletas ou incorretas. As vendas do CSV são históricas e **não representam vendas atuais**. O projeto é um protótipo académico e não pretende fornecer um catálogo completo de videojogos.

## Relatório e demonstração

- **Relatório:** [`docs/31521_doc.pdf`](docs/31521_doc.pdf)
- **Vídeo:** https://vimeo.com/1238015643
- **Código e ficheiros do trabalho:** https://github.com/Shuchuga85/Trabalho1ETL_31521

