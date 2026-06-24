# Exportações Brasileiras — Do Data Warehouse ao Machine Learning

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DW-4169E1?logo=postgresql&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?logo=scikitlearn&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-EDA-150458?logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebooks-F37626?logo=jupyter&logoColor=white)

Solução de dados ponta a ponta sobre as exportações brasileiras: da coleta de dados públicos à modelagem dimensional, carga em PostgreSQL, análise exploratória e modelos preditivos de Machine Learning, culminando em insights de negócio.

> Projeto final da disciplina **Desenvolvimento para Ciência de Dados II** — construção de uma solução completa de dados, da modelagem ao Machine Learning.

---

## Índice

- [Sobre o projeto](#sobre-o-projeto)
- [Fonte de dados](#fonte-de-dados)
- [Arquitetura do Data Warehouse](#arquitetura-do-data-warehouse)
- [Tecnologias](#tecnologias)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como executar](#como-executar)
- [Pipeline de execução](#pipeline-de-execução)
- [Resultados](#resultados)
- [Insights de negócio](#insights-de-negócio)
- [Requisitos atendidos](#requisitos-atendidos)
- [Autores](#autores)
- [Licença](#licença)

---

## Sobre o projeto

O objetivo é demonstrar, sobre dados reais, todas as etapas de uma esteira analítica profissional:

1. **Coleta** de um dataset público com mais de 10 mil registros, contendo variáveis numéricas e categóricas.
2. **Modelagem dimensional** em *Star Schema*, com tabela fato e dimensões.
3. **Implementação** do Data Warehouse no PostgreSQL, com carga via ETL e views analíticas.
4. **Análise exploratória (EDA)** com tratamento da base e visualizações.
5. **Machine Learning**, com algoritmos de classificação e regressão devidamente avaliados.
6. **Insights de negócio** acionáveis a partir dos resultados.

---

## Fonte de dados

Dados oficiais de exportação do Brasil, publicados pelo **Comex Stat (SECEX / MDIC)**.

| Característica | Valor |
|---|---|
| Registros (base bruta) | ~5,56 milhões |
| Registros após tratamento | ~5,13 milhões |
| Período | 2023 a 2026 *(2026 parcial: jan–mai)* |
| Países de destino | 248 |
| Produtos (códigos NCM) | 8.811 |
| Variáveis numéricas | `vl_fob`, `kg_liquido`, `valor_medio_kg`, `qt_estat` |
| Variáveis categóricas | país, estado/região, produto (NCM), modal de transporte |

> Os arquivos de dados (`.parquet` e `.csv`) **não são versionados** neste repositório por excederem o limite de tamanho do GitHub. Eles são reproduzidos pela execução dos notebooks (ver [Pipeline de execução](#pipeline-de-execução)).

---

## Arquitetura do Data Warehouse

Modelo **Star Schema**, com uma tabela fato central conectada diretamente a quatro dimensões desnormalizadas.

```
                         ┌──────────────┐
                         │  dim_tempo   │
                         │ ano, mês,    │
                         │ trimestre    │
                         └──────┬───────┘
                                │
        ┌──────────────┐  ┌─────┴────────────┐  ┌──────────────┐
        │   dim_ncm    │──│ fact_exportacao  │──│   dim_pais   │
        │ produto,     │  │ vl_fob,          │  │ nome do país │
        │ capítulo     │  │ kg_liquido,      │  └──────────────┘
        └──────────────┘  │ valor_medio_kg   │
                          └─────┬────────────┘
                                │
                         ┌──────┴───────┐
                         │  dim_estado  │
                         │ estado,      │
                         │ região       │
                         └──────────────┘
```

**Técnicas aplicadas:**

- **Star Schema** — fato ligado diretamente a cada dimensão, sem normalização em floco.
- **Dimensões desnormalizadas** — ex.: `dim_estado` traz estado e região na mesma tabela.
- **Dimensão conformada** — `dim_tempo` é reutilizável por qualquer fato e por todas as views.
- **Chaves substitutas (*surrogate keys*)** — `id_tempo`, `id_pais`, `id_estado`, `id_ncm` como inteiros sequenciais.

**Views analíticas:**

- `vw_exportacao_detalhada` — junta o fato a todas as dimensões (granularidade de transação).
- `vw_exportacao_por_pais` — agrega valor total, peso e preço médio por kg por país.

---

## Tecnologias

| Camada | Ferramentas |
|---|---|
| Linguagem | Python 3.12 |
| Manipulação de dados | pandas, pyarrow |
| Banco de dados | PostgreSQL |
| Conexão / ETL | SQLAlchemy, psycopg, python-dotenv |
| Visualização | matplotlib |
| Machine Learning | scikit-learn |
| Ambiente | Jupyter Notebook |

---

## Estrutura do repositório

```
.
├── notebooks/
│   ├── extracao_csv.ipynb            # 1. Coleta e consolidação dos dados brutos
│   ├── transformacao_dw.ipynb        # 2. Construção das dimensões e da tabela fato
│   ├── carga_postgres.ipynb          # 3. Carga no PostgreSQL e criação das views
│   ├── 04_analise_exploratoria.ipynb # 4. Tratamento, EDA e insights
│   └── 05_machine_learning.ipynb     # 5. Modelos de classificação e regressão
├── data/
│   ├── auxiliar/                     # tabelas de apoio (PAIS, NCM, UF) — não versionado
│   └── processed/                    # parquets gerados pelo pipeline — não versionado
├── Apresentacao_Exportacoes_DW_ML.pptx
├── .env.example                      # modelo das variáveis de ambiente
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Como executar

### Pré-requisitos

- Python 3.12+
- PostgreSQL instalado e em execução
- Git

### 1. Clonar e instalar dependências

```bash
git clone https://github.com/lannacsoares/comex-analytics-br.git
cd <repositorio>

python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

pip install -r requirements.txt
```

### 2. Configurar o banco de dados

Crie o banco no PostgreSQL:

```sql
CREATE DATABASE comex_dw;
```

### 3. Configurar as variáveis de ambiente

Copie o modelo e preencha com suas credenciais:

```bash
cp .env.example .env
```

Conteúdo do `.env`:

```env
DB_USER=postgres
DB_PASSWORD=sua_senha
DB_HOST=localhost
DB_PORT=5432
DB_NAME=comex_dw
```

> O `.env` está no `.gitignore` e **nunca** deve ser enviado ao repositório.

---

## Pipeline de execução

Execute os notebooks **na ordem abaixo** (cada etapa gera os artefatos consumidos pela seguinte):

| Ordem | Notebook | O que faz |
|---|---|---|
| 1 | `extracao_csv.ipynb` | Lê e consolida os dados brutos do Comex Stat em `exportacoes.parquet`. |
| 2 | `transformacao_dw.ipynb` | Cria as dimensões e a tabela fato (Star Schema) em `data/processed/`. |
| 3 | `carga_postgres.ipynb` | Carrega as tabelas no PostgreSQL e cria as views analíticas. |
| 4 | `04_analise_exploratoria.ipynb` | Trata a base, gera a EDA e exporta `base_analitica.parquet`. |
| 5 | `05_machine_learning.ipynb` | Treina e avalia os modelos de classificação e regressão. |

---

## Resultados

### Classificação — *alto valor agregado* (US$/kg acima da mediana)

Alvo binário balanceado (corte na mediana de ~US$ 9/kg). Divisão treino/teste de 70/30 estratificada, com padronização para os modelos baseados em distância.

| Modelo | Acurácia | F1-score |
|---|---|---|
| Regressão Logística | **0,833** | **0,832** |
| KNN | 0,827 | 0,829 |
| Random Forest | 0,823 | 0,821 |
| Árvore de Decisão | 0,809 | 0,810 |

> **Sem vazamento de dados:** `vl_fob` e `valor_medio_kg` foram propositalmente excluídos das *features*, pois o alvo deriva deles. As variáveis preditoras são apenas estruturais (região, destino, produto, modal, volume e período).

### Regressão Linear — previsão do valor FOB

Previsão de `log(vl_fob)` a partir de fatores estruturais (a transformação log corrige a forte assimetria da variável).

| Métrica | Valor |
|---|---|
| R² | 0,857 |
| MAE (escala log) | 0,85 |
| RMSE (escala log) | 1,17 |

**Comparação:** os quatro classificadores ficaram próximos (81–83%). A Regressão Logística liderou porque o sinal que separa alto/baixo valor agregado é **estrutural e quase linear** (capítulo do produto e modal de transporte quase determinam a faixa de valor), reduzindo a vantagem de modelos não-lineares.

---

## Insights de negócio

1. **Dependência de destino** — a China absorve **29,4%** de todo o valor exportado; os 5 maiores destinos somam **51%**. Alta exposição a poucos compradores.
2. **Concentração regional** — a região **Sudeste** responde por **50%** das exportações em valor, evidenciando desigualdade territorial.
3. **Perfil de commodity** — combustíveis, soja, minérios e carnes lideram a pauta: produtos de baixo valor agregado escoados majoritariamente por via marítima.
4. **Valor agregado x modal** — o modal aéreo tem valor mediano de **~US$ 54/kg**, cerca de **11x** o marítimo (~US$ 4,7/kg). "Commodity navega, produto nobre voa."
5. **Estabilidade sem crescimento** — 2023–2025 estáveis em ~US$ 340 bi/ano, sinal de pauta madura e exposta ao preço de commodities.

**Recomendações:** diversificar destinos, agregar valor à pauta (industrialização) e investir em logística no Norte, Nordeste e Centro-Oeste para reduzir a concentração no Sudeste.

---

## Requisitos atendidos

- [x] Dataset real com +10.000 registros, variáveis numéricas e categóricas
- [x] Modelagem dimensional (Star Schema): 1 fato + 4 dimensões
- [x] Dimensões desnormalizadas e dimensão conformada
- [x] Implementação no PostgreSQL com ETL e 2 views analíticas
- [x] EDA com gráficos de distribuição, comparação entre categorias e evolução temporal
- [x] Classificação: KNN, Árvore de Decisão, Random Forest e Regressão Logística
- [x] Regressão: Regressão Linear
- [x] Separação treino/teste, padronização e avaliação dos modelos
- [x] Métricas: acurácia, precision, recall, F1-score e matriz de confusão
- [x] Comparação de modelos e insights de negócio
- [x] Apresentação final

---

## Licença

Projeto de caráter acadêmico, distribuído sob a licença MIT. Consulte o arquivo `LICENSE` para mais detalhes.
