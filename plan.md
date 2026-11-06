# Plano de Implementação — Automotive Data Lakehouse (NHTSA Makes)

## Problem Statement

Construir um pipeline de dados local que consome o endpoint `GetAllMakes` da API NHTSA,
armazena e processa os dados em camadas Landing → Bronze → Silver → Gold usando
PySpark + Delta Lake, e disponibiliza os resultados em um dashboard Streamlit.

## Requirements

- Scripts Python puros para ingestão e processamento (sem notebooks)
- Armazenamento local (OCI fica como etapa futura)
- Estrutura flat de pastas: `ingestion/`, `processing/`, `app/`, `tests/`
- Camadas: Landing (JSON bruto), Bronze (Delta), Silver (Delta validado), Gold (Delta agregado)
- Dashboard Streamlit como produto final de consumo
- Ambiente Python gerenciado com `venv` + `pip`
- Rastreabilidade completa: metadados de ingestão em cada execução

## Background

- API NHTSA `GetAllMakes` não exige autenticação
- Retorna JSON com `Count`, `Message`, `SearchCriteria` e `Results[]` contendo `Make_ID` (int) e `Make_Name` (string)
- Delta Lake suporta execução local via `delta-spark` — requer Java 8/11
- PySpark funciona bem no Windows com `venv` desde que `JAVA_HOME` esteja configurado
- Quantidade de registros varia — nunca fixar contagens no código

## Proposed Solution

Pipeline batch composto por 4 scripts independentes + 1 app Streamlit, com dados particionados por data de ingestão.

```
API NHTSA → ingestion/ingest_makes.py → data/landing/
          → processing/bronze.py      → data/bronze/
          → processing/silver.py      → data/silver/
          → processing/gold.py        → data/gold/
          → app/dashboard.py          → Streamlit UI
```

## Estrutura de Pastas

```
dados-automotivos/
├── ingestion/
│   └── ingest_makes.py
├── processing/
│   ├── bronze.py
│   ├── silver.py
│   └── gold.py
├── app/
│   └── dashboard.py
├── tests/
│   ├── test_ingestion.py
│   ├── test_bronze.py
│   ├── test_silver.py
│   └── test_gold.py
├── data/                  # gerado em runtime, ignorado pelo git
│   ├── landing/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── requirements.txt
├── .gitignore
├── plan.md
└── README.md
```

## Task Breakdown

### Task 1: Setup do projeto e ambiente
**Objetivo:** Criar estrutura de pastas, requirements.txt, .gitignore, README.md
**Teste:** `python -c "import pyspark; import delta; import streamlit; print('OK')"`
**Demo:** ambiente instalado e importações funcionando

### Task 2: Script de ingestão Landing
**Objetivo:** `ingestion/ingest_makes.py` — chama API, salva JSON bruto + metadados
**Teste:** `tests/test_ingestion.py` — mock HTTP, valida arquivos gerados
**Demo:** arquivos em `data/landing/` após execução

### Task 3: Camada Bronze
**Objetivo:** `processing/bronze.py` — lê Landing, escreve Delta com metadados
**Teste:** `tests/test_bronze.py` — fixture local, valida schema Delta
**Demo:** tabela Delta em `data/bronze/makes/`

### Task 4: Camada Silver
**Objetivo:** `processing/silver.py` — valida, deduplica, padroniza
**Teste:** `tests/test_silver.py` — dados sujos, valida rejeição e log
**Demo:** tabela limpa em `data/silver/makes/` + quality_log

### Task 5: Camada Gold
**Objetivo:** `processing/gold.py` — totais, evolução, novas/removidas
**Teste:** `tests/test_gold.py` — dois snapshots simulados
**Demo:** tabela de evolução em `data/gold/makes_summary/`

### Task 6: Dashboard Streamlit
**Objetivo:** `app/dashboard.py` — resumo, evolução, novas/removidas, explorador
**Demo:** `streamlit run app/dashboard.py` com dados reais

### Task 7: Orquestrador e validação end-to-end
**Objetivo:** `run_pipeline.py` — executa ingest → bronze → silver → gold
**Teste:** `pytest tests/` passando 100%
**Demo:** pipeline completo do zero até o dashboard
