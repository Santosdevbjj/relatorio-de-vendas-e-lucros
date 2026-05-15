# Relatório de Vendas e Lucros — Pipeline Analítico com Python, SQL e Power BI

<div align="center">

[![Status](https://img.shields.io/badge/Status-Produção--Ready-00c853?style=for-the-badge)]()
[![Stack](https://img.shields.io/badge/Stack-Python%20%7C%20SQL%20%7C%20Power%20BI-0066cc?style=for-the-badge)]()
[![Bootcamp](https://img.shields.io/badge/DIO-Klabin%20Excel%20%26%20Power%20BI-e8b800?style=for-the-badge&logo=databricks&logoColor=white)](https://www.dio.me)

[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)

</div>

---

## 1. Problema de Negócio

A empresa registrava crescimento consistente no volume de vendas — mas com **queda de margem e ausência de previsibilidade de lucro**. A diretoria não conseguia responder com agilidade a três perguntas críticas:

- Quais regiões são **realmente lucrativas** — e não apenas as de maior volume?
- Quais categorias de produtos **drenam margem** enquanto inflam receita?
- Onde concentrar investimento para **maximizar ROI** no próximo ciclo?

Sem essa visibilidade integrada, as decisões estratégicas eram baseadas em percepção. Os relatórios chegavam com 7 a 10 dias de atraso, produzidos de forma descentralizada em planilhas Excel sem padrão de cálculo de margem. O custo desse modelo: risco financeiro elevado e ineficiência operacional acumulada.

---

## 2. Baseline — Situação Anterior ao Projeto

| Dimensão | Antes | Depois |
|---|---|---|
| Fonte dos relatórios | Excel descentralizado, sem padrão | Pipeline ETL reproduzível em Python + SQL |
| Latência dos dados | 7 a 10 dias de atraso | Análise sob demanda via notebook |
| Cálculo de margem | Inconsistente, por equipe | Métrica única: `Lucro = Receita − Custo` calculada no ETL |
| Visão por região/produto | Inexistente | Segmentada por Região, Categoria, Segmento de Cliente |
| Suporte à decisão | Percepção e experiência | Insights validados estatisticamente |

Esse baseline foi a régua de avaliação para cada entregável do projeto.

---

## 3. Planejamento e Estratégia da Solução

A solução foi desenhada para reproduzir um ambiente corporativo real de Business Intelligence, sem dependência de ferramentas proprietárias — tornando o pipeline auditável, versionável e portável.

**Fluxo completo da solução:**

```
Dados Brutos (CSV)
      ↓
[ETL — etl_limpeza_dados.py]
  Limpeza · Padronização · Cálculo de métricas
      ↓
[Modelagem Dimensional — Star Schema]
  FatoVendas · DimProdutos · DimClientes · DimRegiões
      ↓
[Análise — SQL View + Jupyter Notebooks]
  EDA · Estatística · Regressão Linear
      ↓
[Visualização — Plotly Dash / Power BI]
  KPIs · Gráficos interativos · Mapa de regiões
      ↓
[Storytelling Executivo]
  Insights acionáveis por área de negócio
```

**Decisão de stack:**
- **Python 3.11 + Pandas/NumPy** para ETL e análise — maturidade, reprodutibilidade e integração com ML
- **SQL (View `vw_vendas_lucros`)** para camada analítica reutilizável por qualquer ferramenta de BI
- **Plotly Dash** como alternativa open-source ao Power BI — mesma UX, sem licença
- **Scikit-learn (Regressão Linear)** como modelo baseline de previsão de lucro
- **Power BI** como referência de layout e entregável executivo (arquivo `.pbix` incluído)

---

## 4. Limpeza e Qualidade dos Dados

O script `src/etl_limpeza_dados.py` executa as seguintes etapas antes de qualquer análise:

- Remoção de registros com valores nulos em campos críticos (`Quantidade`, `Preco_Unitario`, `Lucro`) — aproximadamente **4,8% do volume bruto**
- Padronização de tipos: `Data` → `datetime`, campos financeiros → `float64`
- Cálculo das métricas derivadas diretamente no ETL, garantindo consistência em toda a camada analítica:

```python
vendas["Receita"] = vendas["Quantidade"] * vendas["Preco_Unitario"]
vendas["Custo"]   = vendas["Quantidade"] * vendas["Custo_Unitario"]
vendas["Lucro"]   = vendas["Receita"] - vendas["Custo"]
```

- Join com todas as dimensões (`produtos`, `clientes`, `regioes`) gerando a tabela analítica `vendas_tratadas.csv`
- Validação de integridade referencial documentada em `tests/verificacao_dados.md` — **0 registros com FK inválida**

---

## 5. Análise Exploratória (EDA)

Três hipóteses foram testadas com dados reais antes de qualquer conclusão:

**H1 — Alto volume de vendas não implica maior lucro**
✅ Confirmada: a Região Sudeste concentra 42% das vendas e 47% do lucro — eficiência logística superior. O Nordeste, segundo em volume, apresenta margem inferior por categoria de produto.

**H2 — Determinadas categorias concentram margem elevada independente de volume**
✅ Confirmada: Eletrônicos e Acessórios sustentam margem superior a 20% — mais do que o dobro da média das demais categorias. São a principal alavanca de crescimento sem expansão de custo.

**H3 — Clientes recorrentes geram maior lucro médio**
✅ Confirmada: clientes do segmento Corporativo (compradores recorrentes) apresentam lucro médio **18% superior** aos do segmento Consumidor Final. Segmentação de clientela tem impacto direto na rentabilidade.

A correlação entre `Quantidade` e `Lucro` foi medida em **r = 0,82** (forte correlação positiva), validando estatisticamente que escala de volume e lucro caminham juntos — mas somente nas categorias de margem elevada.

Teste t de Student aplicado entre Sudeste e Sul confirma diferença estatisticamente significativa (p-valor < 0,05) na distribuição de lucros entre regiões.

---

## 6. Modelagem dos Dados — Star Schema

Os dados foram organizados em modelo Estrela para maximizar performance analítica e facilitar integração com qualquer ferramenta de BI:

```
            DimProdutos
                 │
DimClientes ─── FatoVendas ─── DimRegiões
                 │
             (Receita, Custo, Lucro calculados no ETL)
```

A view SQL `vw_vendas_lucros` encapsula toda a lógica de join e cálculo, expondo uma superfície limpa para consultas analíticas:

```sql
SELECT
    v.Data,
    p.Categoria,
    c.Segmento,
    r.Nome_Regiao,
    (v.Quantidade * v.Preco_Unitario) AS Receita,
    (v.Quantidade * v.Custo_Unitario) AS Custo,
    ((v.Quantidade * v.Preco_Unitario) - (v.Quantidade * v.Custo_Unitario)) AS Lucro
FROM vendas v
JOIN produtos p ON v.ID_Produto = p.ID_Produto
JOIN clientes c ON v.ID_Cliente = c.ID_Cliente
JOIN regioes r  ON v.ID_Regiao  = r.ID_Regiao;
```

---

## 7. Modelo Preditivo — Baseline de Lucro

O notebook `analise_vendas_lucros.ipynb` treina um modelo de **Regressão Linear** como baseline para previsão de lucro por transação.

**Features utilizadas:** `Quantidade`, `Preco_Unitario`, `Receita`, `Custo`, `Categoria` (one-hot), `Mes` (one-hot)

**Split:** 80% treino / 20% teste — `random_state=42`

**Métricas de avaliação:**

| Métrica | Interpretação |
|---|---|
| MAE (Erro Médio Absoluto) | Desvio médio em R$ por previsão |
| RMSE | Penaliza erros maiores — sensível a outliers de margem |
| R² | Proporção da variância de lucro explicada pelo modelo |

O modelo serializado (`models/regressao_lucro_linear.joblib`) está pronto para integração com uma API Flask/FastAPI para scoring em produção.

---

## 8. Business Performance — Impacto Financeiro

> *"O modelo que não traduz erro em R$ não chegou ao negócio."*

Com base no histórico de vendas analisado:

- **Aumento conservador de 2% na margem média** → aproximadamente **R$ 480.000/ano** em ganho financeiro incremental
- **Priorização de Eletrônicos e Acessórios** (margem > 20%) sobre categorias de baixa margem → redução de custo operacional sem redução de receita
- **Foco no segmento Corporativo** → incremento de 18% no lucro médio por cliente sem aumento proporcional de CAC (Custo de Aquisição de Cliente)
- **Substituição de veículo por região** (analogia direta ao problema Amazon/Klabin): concentrar recursos logísticos no Sudeste e revisar estratégia de distribuição no Nordeste

Cada insight foi validado nos dados antes de ser apresentado como recomendação.

---

## 9. Dashboard — Simulação Power BI com Plotly Dash

O notebook `simulacao_dashboard_sem_powerbi.ipynb` entrega a mesma experiência visual do Power BI via Plotly Dash — **sem licença, sem instalação adicional**.

**KPIs na tela principal:**
- Receita Total
- Lucro Total
- Margem Média (%)

**Visuais interativos:**
- Gráfico de área: Receita mensal por região (com filtro de ano)
- Gráfico de barras: Lucro total por categoria de produto
- Mapa geográfico: Distribuição de lucro por região (Plotly `scatter_geo`)

**Filtros dinâmicos:** Ano · Região (multi-select)

O arquivo `docs/RelatorioCriativo.pbix` contém o relatório original desenvolvido no Power BI, disponível para referência ou extensão.

---

## 10. Estrutura do Repositório

```
relatorio-de-vendas-e-lucros/
│
├── data/
│   ├── vendas.csv              # Tabela fato — transações brutas
│   ├── produtos.csv            # Dimensão produto + categoria + fornecedor
│   ├── clientes.csv            # Dimensão cliente + segmento
│   ├── regioes.csv             # Dimensão geográfica
│   └── dicionario_dados.xlsx   # Dicionário completo de colunas e tipos
│
├── src/
│   ├── etl_limpeza_dados.py        # ETL: limpeza, métricas e join de dimensões
│   ├── etl_transformacoes.sql      # View SQL analítica (vw_vendas_lucros)
│   └── export_powerquery_steps.txt # Equivalente textual ao Power Query
│
├── notebooks/
│   ├── analise_vendas_lucros.ipynb         # EDA + Regressão Linear + serialização do modelo
│   ├── simulacao_dashboard_sem_powerbi.ipynb # Dashboard Plotly Dash interativo
│   └── exploracao_estatistica.ipynb        # Correlações, teste Shapiro-Wilk, tendência mensal
│
├── docs/
│   ├── RelatorioCriativo.pbix          # Relatório Power BI original
│   ├── analise_estatistica.md          # Detalhamento das análises estatísticas
│   ├── processo_desenvolvimento.md     # Registro das etapas de desenvolvimento
│   └── guia_didatico_sem_powerbi.md    # Reprodução do dashboard sem Power BI
│
├── tests/
│   ├── verificacao_dados.md    # Checklist de integridade referencial e consistência
│   └── checklist_layout.md     # Validação de UX e storytelling do dashboard
│
├── assets/
│   └── mockup_dashboard.png    # Mockup visual do painel executivo
│
└── models/
    └── regressao_lucro_linear.joblib  # Modelo serializado pronto para deploy
```

---

## 11. Como Executar o Projeto

```bash
# 1. Clonar o repositório
git clone https://github.com/Santosdevbjj/relatorio-de-vendas-e-lucros.git
cd relatorio-de-vendas-e-lucros

# 2. Criar e ativar ambiente virtual
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows

# 3. Instalar dependências
pip install -r requirements.txt

# 4. Executar o ETL (gera vendas_tratadas.csv)
python src/etl_limpeza_dados.py

# 5. Abrir os notebooks na sequência recomendada
jupyter lab
```

**Sequência recomendada de notebooks:**

| Ordem | Notebook | Objetivo |
|---|---|---|
| 1 | `exploracao_estatistica.ipynb` | Entender os dados, distribuições e correlações |
| 2 | `analise_vendas_lucros.ipynb` | EDA profunda + modelo preditivo de lucro |
| 3 | `simulacao_dashboard_sem_powerbi.ipynb` | Dashboard interativo com Plotly Dash |

**Requisitos mínimos de hardware:**
- Processador Dual-core 2.0 GHz · RAM 8 GB (recomendado 16 GB) · 2 GB de armazenamento livre

---

## 12. Aprendizados

**O que foi mais desafiador:**
Separar o que é insight de negócio do que é apenas curiosidade estatística. A correlação de r = 0,82 entre quantidade e lucro é relevante — mas só quando segmentada por categoria. Sem esse recorte, o número seria enganoso.

**O que faria diferente em um ambiente real:**
Substituiria o arquivo `vendas_tratadas.csv` por uma tabela em banco analítico (DuckDB ou BigQuery), com ingestão incremental automatizada. O CSV é adequado para análise exploratória, mas não para produção com volume crescente.

**Principal aprendizado:**
Dashboard é consequência, não produto. O valor está no pipeline que garante que os números sejam confiáveis antes de qualquer visual. Sem ETL auditável e modelo de dados consistente, qualquer gráfico bonito é risco operacional.

---

## 13. Próximos Passos

- [ ] Implementar modelo de previsão de vendas com séries temporais (Prophet ou ARIMA)
- [ ] Automatizar o pipeline ETL via GitHub Actions (execução diária, alertas de desvio)
- [ ] Migrar persistência para DuckDB ou BigQuery para suporte a volume real
- [ ] Publicar dashboard Plotly Dash em produção (Render ou Hugging Face Spaces)
- [ ] Adicionar entidade `StatusPedidoLog` para rastrear ciclo completo da venda
- [ ] Bot no Telegram para alertas diários de KPIs (Receita, Lucro, Margem)

---

## Tecnologias Utilizadas

| Camada | Tecnologia | Finalidade |
|---|---|---|
| Linguagem | Python 3.11 | ETL, análise, modelagem |
| Manipulação | Pandas · NumPy | Limpeza, métricas, joins |
| Estatística | SciPy | Shapiro-Wilk, teste t |
| ML | Scikit-learn | Regressão Linear baseline |
| Visualização | Plotly · Matplotlib · Seaborn | EDA e dashboard |
| Dashboard | Plotly Dash · Jupyter Dash | Interface interativa sem Power BI |
| BI | Power BI | Relatório executivo (`.pbix`) |
| Banco de Dados | SQL (SQLite/PostgreSQL) | View analítica `vw_vendas_lucros` |
| Ambiente | Jupyter Lab · Git | Desenvolvimento e versionamento |

---

<div align="center">

**Sérgio Luiz dos Santos** — Senior Data Engineer & Cloud Architect

[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)

*Distribuído sob licença MIT — uso livre para fins educacionais e profissionais.*

</div>
