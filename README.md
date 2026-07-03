# Pipeline-anac-databricks
<img width="1253" height="577" alt="data_gold com" src="https://github.com/user-attachments/assets/a03df09f-f37a-47c2-864d-80365631f5cd" />
<img width="1253" height="577" alt="data_gold com" src="https://github.com/user-attachments/assets/49585071-04ee-47e2-980a-82037115d614" />

# ✈️ Pipeline de Dados ANAC - Arquitetura Medalhão no Databricks

Este repositório contém um projeto prático de Engenharia de Dados focado no processamento e higienização de dados abertos da ANAC (Agência Nacional de Aviação Civil), utilizando a **Arquitetura Medalhão** dentro do ecossistema **Databricks**.

O objetivo principal deste projeto é demonstrar a extração de dados brutos, a aplicação de regras de qualidade e governança (limpeza e tipagem), e a disponibilização final dos dados modelados para consumo de negócios.

## 🛠️ Tecnologias Utilizadas
* **Databricks:** Ambiente de processamento e orquestração.
* **PySpark:** Framework principal para manipulação distribuída e transformação dos DataFrames.
* **Unity Catalog:** Gerenciamento de metadados, schemas e separação lógica entre Volumes (armazenamento físico) e Tabelas.
* **Formatos Otimizados:** Leitura e escrita utilizando formato `Parquet` com compressão `Snappy`.

---

## 🏛️ Arquitetura do Pipeline

O pipeline foi estruturado em três notebooks distintos, refletindo a jornada do dado pelas camadas da Arquitetura Medalhão:

### 🥉 Camada Bronze (Ingestão)
O objetivo da camada Bronze é manter o dado o mais fiel possível à sua origem, alterando apenas o formato de armazenamento para otimizar a leitura futura.
* Leitura dos arquivos originais disponibilizados pela ANAC.
* Escrita dos dados brutos em formato Parquet no Volume `bronze` do Unity Catalog, garantindo a preservação do histórico.

### 🥈 Camada Silver (Higienização e Governança)
É o coração técnico do pipeline, onde as regras de qualidade de dados (Data Quality) são aplicadas.
* **Padronização de Nomes:** Renomeação de colunas essenciais para o padrão minúsculo e sem espaços (ex: `Operador_Padronizado` ➔ `operador_padronizado`).
* **Tratamento de Tipos e Modo ANSI:** Devido ao modo restrito (ANSI Mode) habilitado por padrão no Databricks, o uso do método comum `.cast("int")` pode gerar o erro fatal `[CAST_INVALID_INPUT]` ao encontrar strings malformadas ou lixo nos dados. Para garantir a resiliência do pipeline, foi utilizada a função `expr("try_cast(...)")`, que converte falhas em valores `NULL` verdadeiros sem interromper a esteira.
* **Limpeza de Nulos:** Tratamento de regras de negócio específicas utilizando `when().otherwise()` (ex: substituindo valores "null" em texto por "NAO_IDENTIFICADO") e preenchimento de nulos numéricos com `0` utilizando `.na.fill()`.
* **Desduplicação:** Remoção de registros duplicados com base na chave `Numero_da_Ocorrencia`.

### 🥇 Camada Gold (Consumo de Negócios)
A última etapa transforma os dados limpos em tabelas estruturadas, prontas para ferramentas de visualização (Power BI, Tableau) ou consultas SQL analíticas.
* Aplicação de agregações e regras de negócio focadas no usuário final.
* Criação e salvamento de Tabelas Gerenciadas (`table_gold`) no schema do projeto dentro do Unity Catalog.

---

## 🚀 Como Executar
Os scripts foram desenvolvidos para rodar na versão Community/Free do Databricks.
1. Clone este repositório para o seu Workspace utilizando a integração via "Git Folders" (Repos).
2. Certifique-se de configurar os Volumes (Bronze e Silver) e o Schema no Unity Catalog antes de executar.
3. Rode os notebooks na ordem cronológica: `01_Bronze` ➔ `02_Silver` ➔ `03_Gold`.
