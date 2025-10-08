# 📊 Análise de Dados da PNAD COVID com AWS Data Lake

## 🧩 1. Visão Geral do Projeto
Este projeto implementa um **pipeline de dados completo** para processar, analisar e disponibilizar os microdados da pesquisa **PNAD COVID-19 do IBGE**.  
O objetivo é transformar os dados brutos em um conjunto de indicadores de saúde e socioeconômicos, prontos para serem consumidos por ferramentas de **Business Intelligence (BI)**, como o **Power BI**.

A solução é construída inteiramente em **nuvem**, utilizando serviços da **Amazon Web Services (AWS)** para criar uma arquitetura de **Data Lake robusta, escalável e otimizada para custos**.

---

## 🏗️ 2. Arquitetura da Solução
O projeto segue a metodologia de **Data Lake** com camadas de maturidade de dados (**Bronze, Silver e Gold**), garantindo **governança e qualidade** em cada etapa do processo.

- **Camada Bronze:** Armazena os dados brutos da PNAD COVID-19 no formato CSV, sem nenhuma modificação.  
  → Funciona como uma cópia fiel da fonte original.

- **Camada Silver:** Contém os dados limpos, padronizados e enriquecidos.  
  Os dados são armazenados em formato **Parquet** e **particionados por ano e mês** para otimizar a performance e os custos das consultas.  
  Esta camada é a **fonte de verdade** para análises e machine learning.

- **Camada Gold:** Armazena dados agregados e **KPIs (Key Performance Indicators)** específicos para as necessidades de negócio.  
  São tabelas menores e pré-calculadas que aceleram a geração de dashboards e relatórios.

- **Camada de Exportação:** Materializa a base de dados final consumida pelo Power BI, unindo KPIs e contagens de entrevistados.

---

## ⚙️ 3. Pipeline de Dados (ETL)
O pipeline é orquestrado pelo notebook **`tc-pnad-covid-final.ipynb`** e executa as seguintes etapas:

### 🧱 Setup do Ambiente
- Instalação das bibliotecas Python necessárias (`pandas`, `awswrangler`, `boto3`).
- Configuração da sessão AWS.

### 📥 Ingestão de Dados Brutos
- Leitura dos arquivos CSV da PNAD COVID (Agosto a Novembro de 2020) diretamente da **camada Bronze (S3)**.

### 🧹 Transformação e Limpeza (Pandas)
- Padronização dos nomes das colunas (minúsculas).
- Conversão de respostas de sintomas e testes para formato binário (0/1).
- Criação de variáveis derivadas (`qualquer_sintoma`, `teste_positivo_qualquer`).
- Cálculo de variáveis socioeconômicas (`ocupado`, `desempregado`, `forca_trabalho`) com base na coluna `C007B`.
- Cálculo de `renda_total` a partir da soma de todas as variáveis de rendimento.
- Enriquecimento dos dados com rótulos (`uf_nome`, `sexo_nome`).

### 🎯 Seleção de Variáveis
- Limitação da camada Silver para **20 colunas principais**, conforme análise de negócio.

### 💾 Carregamento na Camada Silver
- Salvamento do DataFrame em formato **Parquet (compressão Snappy)** no S3, particionado por **ano e mês**.

### 🗂️ Catalogação com AWS Glue
- Criação ou atualização de tabela no **AWS Glue Data Catalog**, permitindo consultas via **Amazon Athena**.

### 📊 Análise e Criação de Views (Athena)
- Execução de queries SQL para criar views agregadas, como:
  - `vw_kpis_econ`
  - `vw_kpis_full`

### 🪙 Materialização na Camada Gold
- Armazenamento das agregações em formato Parquet, otimizando o acesso do BI.

### 🚀 Exportação Final para BI
- Query final no Athena para unir KPIs, contagens de entrevistados, testados e positivos.
- Resultado salvo em um único arquivo **Parquet** no S3, pronto para uso no **Power BI**.

---

## ☁️ 4. Tecnologias Utilizadas

### Linguagem
- **Python 3**

### Bibliotecas Principais
- **Pandas:** Manipulação e transformação de dados  
- **AWS Data Wrangler:** Integração simplificada com serviços AWS  
- **Boto3:** SDK oficial da AWS para Python  
- **PyArrow:** Manipulação do formato Parquet

### Serviços AWS
- **Amazon S3:** Armazenamento das camadas do Data Lake  
- **AWS Glue Data Catalog:** Catálogo de metadados persistente  
- **Amazon Athena:** Consultas SQL interativas sobre dados do S3

---

## 🧬 5. Schema da Camada Silver

A camada Silver contém as **20 variáveis principais** abaixo:

| **Categoria** | **Variáveis** |
|----------------|---------------|
| **Demografia** | uf_codigo, idade, sexo, escolaridade |
| **Saúde e Testes** | plano_saude, fez_teste_covid, qualquer_sintoma, teste_positivo_qualquer |
| **Sintomas** | sint_febre_semana, sint_tosse_semana, sint_dor_garganta_semana, sint_falta_ar_semana, sint_perda_olfato_paladar_semana |
| **Economia** | ocupado, desempregado, forca_trabalho, renda_total |
| **Rótulos** | uf_nome, sexo_nome, mes_label |

> As colunas **ano** e **mes** são usadas como partições e não entram na contagem das 20 variáveis.

---

## 🧠 6. Como Executar o Projeto

### 🔧 Pré-requisitos
- Conta AWS com permissões de acesso a **S3**, **Glue** e **Athena**  
- **AWS CLI** configurada (`aws configure`)  
- Bucket S3 criado para o Data Lake

### 🧭 Configuração do Ambiente
1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/pnad-covid-aws-datalake.git
   cd pnad-covid-aws-datalake

### No notebook, altere a variável:
S3_BUCKET = "nome-do-seu-bucket"

### 📤 Carga dos Dados na Camada Bronze
1. Baixe os arquivos CSV da PNAD-COVID (Ago–Nov 2020).
2. Faça upload para a pasta: s3://<seu-bucket>/bronze/

### ▶️ Execução do Pipeline
1. Abra o Jupyter Notebook tc-pnad-covid-final.ipynb.
2. Execute todas as células em ordem.
3. O notebook cuidará de todo o processo ETL → Glue → Exportação.

