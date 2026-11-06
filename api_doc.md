# Documentação da API NHTSA vPIC --- GetAllMakes

> **Projeto:** Automotive Data Lakehouse\
> **Fonte:** NHTSA --- National Highway Traffic Safety Administration
> (Estados Unidos)\
> **Endpoint:**
> `GET https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes?format=json`\
> **Data da documentação:** 06/11/2026

## 1. Objetivo

O endpoint `GetAllMakes` retorna a lista de marcas (*makes*) disponíveis
no conjunto de dados vPIC --- *Product Information Catalog and Vehicle
Listing*, mantido pela NHTSA.

A API foi criada para permitir que desenvolvedores e pesquisadores
consultem informações padronizadas sobre veículos, fabricantes, marcas,
modelos e dados relacionados à identificação veicular.

**Importante:** neste endpoint, *Make* significa a marca/nome atribuído
a um grupo de veículos. Não se deve interpretar automaticamente cada
registro como uma empresa fabricante distinta.

## 2. Identificação e acesso

  -------------------------------------------------------------------------------------------------------
  Propriedade                         Descrição
  ----------------------------------- -------------------------------------------------------------------
  Provedor                            NHTSA --- National Highway Traffic Safety Administration

  País / jurisdição                   Estados Unidos

  Serviço                             vPIC --- Product Information Catalog and Vehicle Listing

  Recurso                             `GetAllMakes`

  Método HTTP                         `GET`

  Formato solicitado                  JSON

  Autenticação                        A documentação pública consultada não indica chave ou autenticação
                                      para este endpoint

  URL                                 `https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes?format=json`
  -------------------------------------------------------------------------------------------------------

### Parâmetro

  ------------------------------------------------------------------------
  Parâmetro        Tipo                       Obrigatório Descrição
  ---------------- ---------------- --------------------- ----------------
  `format`         string            Não, mas recomendado Define o formato
                                                          da resposta.
                                                          Neste projeto
                                                          usamos `json`.

  ------------------------------------------------------------------------

A documentação vPIC também apresenta formatos como XML e CSV para
métodos compatíveis.

## 3. Estrutura da resposta

A resposta JSON documentada para `GetAllMakes` contém propriedades de
resumo e uma lista `Results`.

Estrutura conceitual:

``` json
{
  "Count": 0,
  "Message": "string",
  "SearchCriteria": "string",
  "Results": [
    {
      "Make_ID": 0,
      "Make_Name": "string"
    }
  ]
}
```

Os valores acima são **ilustrativos do formato e dos tipos**, não
representam uma captura ao vivo nem uma contagem atual. A quantidade e
os registros devem ser obtidos executando a requisição.

### 3.1 Propriedades da resposta

  -----------------------------------------------------------------------
  Campo                   Tipo esperado           Descrição
  ----------------------- ----------------------- -----------------------
  `Count`                 integer                 Quantidade informada
                                                  pela API para o
                                                  resultado.

  `Message`               string                  Mensagem de retorno da
                                                  API.

  `SearchCriteria`        string ou null          Critério de pesquisa
                                                  associado à consulta.

  `Results`               array de objetos        Lista de marcas
                                                  retornadas.
  -----------------------------------------------------------------------

### 3.2 Campos de cada item de `Results`

  -----------------------------------------------------------------------
  Campo                   Tipo esperado           Descrição
  ----------------------- ----------------------- -----------------------
  `Make_ID`               integer                 Identificador da marca
                                                  no conjunto de dados
                                                  vPIC.

  `Make_Name`             string                  Nome da marca conforme
                                                  registrado no vPIC.
  -----------------------------------------------------------------------

O identificador `Make_ID` deve ser preservado como chave de referência.
Não substitua esse identificador por um código criado localmente.

## 4. Exemplo de requisição com Python

Instale a dependência:

``` bash
python -m pip install requests
```

Exemplo de consulta com tratamento básico de erros:

``` python
import requests

url = "https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes"
params = {"format": "json"}

try:
    response = requests.get(url, params=params, timeout=30)
    response.raise_for_status()

    payload = response.json()
    makes = payload.get("Results", [])

    print("Status HTTP:", response.status_code)
    print("Count informado pela API:", payload.get("Count"))
    print("Registros recebidos:", len(makes))

    for item in makes[:10]:
        print(item.get("Make_ID"), "-", item.get("Make_Name"))

except requests.exceptions.Timeout:
    print("A API excedeu o tempo limite da requisição.")

except requests.exceptions.HTTPError as error:
    print("A API retornou um erro HTTP:", error)

except requests.exceptions.RequestException as error:
    print("Falha na comunicação com a API:", error)

except ValueError:
    print("A resposta recebida não pôde ser interpretada como JSON.")
```

## 5. Exemplo de inspeção da resposta real

Para registrar a resposta efetivamente recebida, execute:

``` python
import json
import requests

url = "https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes"
response = requests.get(url, params={"format": "json"}, timeout=30)
response.raise_for_status()

payload = response.json()

print(json.dumps(payload, indent=2, ensure_ascii=False))
```

Não fixe no código a quantidade de marcas. A base pode ser atualizada
pelo provedor. Registre em cada execução a data/hora da coleta, o status
HTTP, o valor de `Count` e a quantidade efetiva de itens em `Results`.

## 6. Uso no Automotive Data Lakehouse

O endpoint será a primeira fonte de referência do projeto. O fluxo
proposto é:

### Landing --- dado recebido da fonte

-   Fazer a chamada HTTP e guardar a resposta JSON original, sem
    transformar os campos.
-   Registrar metadados de ingestão, como data/hora UTC da coleta, URL
    consultada e status HTTP.
-   Manter uma cópia por execução, evitando sobrescrever a evidência
    anterior.

Exemplo de organização:

``` text
landing/
└── nhtsa/
    └── makes/
        └── ingest_date=AAAA-MM-DD/
            └── makes_YYYYMMDD_HHMMSS.json
```

### Bronze --- ingestão técnica

-   Ler o JSON armazenado na Landing.
-   Extrair os itens de `Results`.
-   Persistir os registros em tabela Delta.
-   Preservar `Make_ID` e `Make_Name`, além dos metadados de ingestão.

### Silver --- padronização e qualidade

-   Validar se `Make_ID` está preenchido e pode ser convertido para
    inteiro.
-   Validar se `Make_Name` está preenchido.
-   Verificar duplicidades de `Make_ID`.
-   Remover espaços indevidos sem alterar o valor original armazenado na
    Landing.
-   Manter rastreabilidade até a coleta de origem.

### Gold --- consumo analítico

Possíveis produtos de dados: - total de marcas disponíveis na fonte; -
marcas por critérios de classificação que venham a ser obtidos em outras
fontes; - evolução da quantidade de registros entre coletas; - relatório
de qualidade e variações do conjunto.

**Limite analítico:** este endpoint, isoladamente, não fornece modelos,
anos, peças, aplicações ou compatibilidade entre peças e veículos. Essas
análises exigirão outros endpoints ou fontes autorizadas.

## 7. Qualidade, cobertura e limitações

1.  **Cobertura:** a vPIC é uma base da NHTSA construída a partir de
    informações submetidas por fabricantes de veículos no contexto
    regulatório dos Estados Unidos. Não deve ser descrita como um
    catálogo mundial completo nem como uma referência completa do
    mercado brasileiro.
2.  **Marca não é necessariamente fabricante:** o campo `Make_Name`
    identifica uma *make*. Para obter entidades e detalhes de
    fabricantes, devem ser avaliados os endpoints específicos de
    fabricantes.
3.  **Atualização:** os dados podem mudar quando novas informações ou
    correções são incorporadas à base. Por isso, a quantidade de
    registros não deve ser tratada como constante.
4.  **Nulos e valores inesperados:** validar os campos em cada execução;
    não presumir que todos os registros futuros terão sempre os mesmos
    valores.
5.  **Duplicidade:** verificar duplicidade de `Make_ID`. Investigar
    antes de remover registros com nomes iguais, pois nomes iguais não
    provam que os identificadores sejam equivalentes.
6.  **Limites de uso:** a NHTSA informa que aplica controle automatizado
    de tráfego às APIs para proteger a disponibilidade do serviço.
    Evitar chamadas excessivas e implementar timeout e tratamento de
    falhas.
7.  **Contrato da API:** acompanhar a documentação oficial e validar
    estrutura, tipos e campos sempre que a ingestão for executada.

## 8. Referências oficiais

-   [API vPIC --- página principal e métodos
    disponíveis](https://vpic.nhtsa.dot.gov/api/)
-   [Endpoint GetAllMakes em
    JSON](https://vpic.nhtsa.dot.gov/api/vehicles/GetAllMakes?format=json)
-   [NHTSA vPIC --- informações sobre o serviço e
    downloads](https://vpic.nhtsa.dot.gov/About)

## 9. Checklist para a implementação

-   [ ] Consultar o endpoint e confirmar HTTP 200.
-   [ ] Guardar a resposta JSON original na Landing.
-   [ ] Registrar data/hora, URL, status HTTP e quantidade recebida.
-   [ ] Validar `Count`, `Results`, `Make_ID` e `Make_Name`.
-   [ ] Criar a tabela Bronze em Delta Lake.
-   [ ] Criar as regras de qualidade da Silver.
-   [ ] Disponibilizar os dados consolidados para a camada Gold e para o
    Streamlit.

------------------------------------------------------------------------

**Nota de reprodutibilidade:** esta documentação descreve o contrato do
endpoint com base na documentação pública da NHTSA. O exemplo de
estrutura é ilustrativo. Para registrar amostras e contagens atuais,
execute o código de inspeção e armazene a resposta obtida na camada
Landing.

A arquitetura ficará assim: 

Fontes de dados 

API pública NHTSA — fabricantes e veículos 

Python + PySpark + Delta Lake 

Processamento local no Windows 

OCI Object Storage 

Landing · Bronze · Silver · Gold 
