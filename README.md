# Automotive Data Lakehouse

Pipeline de dados local que consome a API NHTSA (National Highway Traffic Safety Administration) para construir um data lakehouse em camadas com PySpark + Delta Lake, expondo os dados via dashboard Streamlit.

## Arquitetura

```
API NHTSA → Landing → Bronze → Silver → Gold → Streamlit Dashboard
```

- **Landing:** JSON bruto da API com metadados de ingestão
- **Bronze:** Tabela Delta com dados extraídos de `Results[]`
- **Silver:** Dados validados, padronizados e deduplicados
- **Gold:** Agregações analíticas (totais, evolução, novas marcas)

## Pré-requisitos

- **Python 3.10+**
- **Java 8 ou 11** (necessário para PySpark)
  - Verifique: `java -version`
  - Configure `JAVA_HOME` se necessário

## Setup

### 1. Criar ambiente virtual

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2. Instalar dependências

```powershell
pip install -r requirements.txt
```

### 3. Verificar instalação

```powershell
python -c "import pyspark; import delta; import streamlit; print('OK')"
```

## Execução

### Pipeline completo

```powershell
python run_pipeline.py
```

### Executar etapas individualmente

```powershell
# 1. Ingerir dados da API
python ingestion/ingest_makes.py

# 2. Processar Landing → Bronze
python processing/bronze.py

# 3. Processar Bronze → Silver
python processing/silver.py

# 4. Processar Silver → Gold
python processing/gold.py
```

### Dashboard

```powershell
streamlit run app/dashboard.py
```

## Testes

```powershell
pytest tests/ -v
```

## Estrutura de Dados

```
data/
├── landing/nhtsa/makes/ingest_date=YYYY-MM-DD/
│   ├── makes_YYYYMMDD_HHMMSS.json
│   └── metadata_YYYYMMDD_HHMMSS.json
├── bronze/makes/                    # Delta Table
├── silver/makes/                    # Delta Table
└── gold/makes_summary/              # Delta Table
```

## Fonte de Dados

- **API:** NHTSA vPIC (Product Information Catalog and Vehicle Listing)
- **Endpoint:** `https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes?format=json`
- **Documentação:** Ver `api_doc.md`

## Referências

- [NHTSA vPIC API](https://vpic.nhtsa.dot.gov/api/)
- [Delta Lake Documentation](https://docs.delta.io/)
- [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)
