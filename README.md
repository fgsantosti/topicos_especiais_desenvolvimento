### README.md

```markdown
# Análise de Dados — E-commerce no Brasil (EDA, Visualização e Estatística)

Este repositório traz uma base de dados sintética e um conjunto de questões orientadoras para praticar:
- Carregamento e inspeção de dados com Pandas
- Análise Exploratória de Dados (EDA)
- Visualizações com Matplotlib/Seaborn
- Estatística descritiva e storytelling com dados

O cenário simula um e-commerce no Brasil com foco em avaliação de satisfação de clientes, permitindo análises atuais como sazonalidade (Black Friday), impacto do canal (Web/App) e efeito de atrasos de entrega na avaliação.

## Base de dados proposta

Cenário: avaliação de satisfação de clientes de um e-commerce no Brasil, com informações de pedidos, categoria de produto, preço, desconto, data, avaliação (nota 1 a 5), canal (web/app), cidade/estado e tempo de entrega.

Arquivos CSV:
- customers.csv: `id_cliente`, `nome`, `cidade`, `estado`, `idade`, `genero`
- orders.csv: `id_pedido`, `id_cliente`, `data_pedido`, `canal`, `valor_bruto`, `desconto`, `categoria`, `subcategoria`, `prazo_estimado_dias`, `prazo_real_dias`, `avaliacao`
- products.csv: `id_produto`, `categoria`, `subcategoria`, `preco_lista`
- order_items.csv: `id_pedido`, `id_produto`, `quantidade`

Tamanho sugerido:
- customers.csv ~ 500 linhas
- products.csv ~ 300 linhas
- orders.csv ~ 2.000 linhas
- order_items.csv ~ 4.000 linhas

Observações:
- A coluna `valor_liquido` pode ser criada como `valor_bruto - desconto`.
- A coluna `atraso` pode ser criada como `prazo_real_dias - prazo_estimado_dias`.
- Recomenda-se criar `desconto_pct = desconto / valor_bruto` (clamp em [0,1]) para análises percentuais.

## Como começar

1) Requisitos
- Python 3.10+ e pip

2) Ambiente e dependências
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate
pip install pandas numpy matplotlib seaborn jupyter
```

3) Estrutura sugerida
```
data/
  customers.csv
  products.csv
  orders.csv
  order_items.csv
notebooks/
  01_intro_dados_visualizacao.ipynb
  02_analise_exploratoria.ipynb
  03_data_visualization.ipynb
  04_visualizacao_avancada.ipynb
  05_estatistica_descritiva.ipynb
src/
  carregar_dados.py
README.md
```

4) Carregamento rápido (exemplo)
```python
import pandas as pd

orders = pd.read_csv('data/orders.csv', parse_dates=['data_pedido'])
customers = pd.read_csv('data/customers.csv')

orders['valor_liquido'] = orders['valor_bruto'] - orders['desconto']
orders['atraso'] = orders['prazo_real_dias'] - orders['prazo_estimado_dias']
orders['desconto_pct'] = (orders['desconto'] / orders['valor_bruto']).clip(0,1)
```

## Questões analíticas e gráficos recomendados

A seguir, 10 perguntas para guiar a análise, com métricas e visualizações sugeridas. Use-as como roteiro de aulas/exercícios.

1) Qual é a evolução da receita ao longo do tempo? Há sazonalidade (ex.: Black Friday)?
- Métricas: receita mensal (`valor_liquido`), número de pedidos.
- Gráficos:
  - Linha temporal da receita mensal (line chart com marcadores)
  - Barras empilhadas por canal (Web/App) por mês
- Dicas: anote eventos como 11/11 e Black Friday (novembro/dezembro).
- Snippet:
  ```python
  orders['mes'] = orders['data_pedido'].dt.to_period('M').dt.to_timestamp()
  df = orders.groupby('mes')['valor_liquido'].sum().reset_index()
  ```

2) Como o canal (Web vs. App) impacta o ticket médio e a taxa de desconto?
- Métricas: ticket médio por pedido (`valor_liquido`), `desconto_pct`.
- Gráficos:
  - Boxplot do ticket por canal (distribuição e outliers)
  - Barras lado a lado do ticket médio por canal
  - Violin plot para detalhar a distribuição
- Snippet:
  ```python
  orders.groupby('canal')['valor_liquido'].mean()
  ```

3) Quais categorias e subcategorias mais contribuem para a receita e para o volume de pedidos?
- Métricas: soma de `valor_liquido`, contagem de pedidos.
- Gráficos:
  - Barras horizontais Top N categorias por receita (com rótulos)
  - Treemap por categoria/subcategoria (ou barras facetadas)
- Dicas: mostrar receita e volume para evitar viés.
- Snippet:
  ```python
  top_cat = orders.groupby('categoria')['valor_liquido'].sum().sort_values(ascending=False).head(10)
  ```

4) Descontos maiores realmente aumentam as avaliações dos clientes?
- Métricas: `desconto_pct` vs. `avaliacao`.
- Gráficos:
  - Scatterplot com linha de tendência (regplot)
  - Heatmap de correlação incluindo `desconto_pct` e `avaliacao`
- Dicas: segmentar por `atraso` (cores) para controlar confusão.
- Snippet:
  ```python
  orders[['desconto_pct','avaliacao']].corr()
  ```

5) Atrasos de entrega afetam a satisfação (avaliação) e variam por categoria?
- Métricas: `atraso` médio e `avaliacao` média por categoria.
- Gráficos:
  - Barras agrupadas (dois gráficos) para atraso e avaliação por categoria
  - Scatterplot (x=atraso médio, y=avaliação média) com rótulos de categoria
- Dicas: linhas de referência (atraso=0, avaliação=4).
- Snippet:
  ```python
  cat = orders.groupby('categoria')[['atraso','avaliacao']].mean()
  ```

6) Existe diferença geográfica (SP, RJ, MG etc.) em receita e avaliação?
- Métricas: receita por estado, avaliação média por estado.
- Gráficos:
  - Barras horizontais por estado (receita e avaliação)
  - Mapa coroplético (opcional)
- Dicas: ponderar avaliação por volume de pedidos.
- Snippet:
  ```python
  geo = orders.merge(customers[['id_cliente','estado']], on='id_cliente')
  geo.groupby('estado').agg(receita=('valor_liquido','sum'), avaliacao=('avaliacao','mean'))
  ```

7) Qual é a distribuição do ticket e onde estão os outliers?
- Métricas: `valor_bruto`/`valor_liquido`.
- Gráficos:
  - Histograma com KDE; usar eixo log se necessário
  - Boxplot por categoria/canal
- Dicas: traçar linhas verticais em P50, P90, P95.
- Snippet:
  ```python
  s = orders['valor_liquido'].dropna(); s.quantile([.5,.9,.95])
  ```

8) Quais subcategorias têm maior taxa de desconto e isso afeta a margem estimada?
- Métricas: `desconto_pct` médio por subcategoria, participação de receita.
- Gráficos:
  - Barras Top N por `desconto_pct`
  - Scatterplot (x=desconto_pct médio, y=receita total), tamanho=volume
- Dicas: destacar trade-offs (alto desconto x alta receita).
- Snippet:
  ```python
  sub = orders.groupby('subcategoria').agg(desconto_pct=('desconto_pct','mean'),
                                           receita=('valor_liquido','sum'),
                                           pedidos=('id_pedido','count'))
  ```

9) Qual é o tempo de entrega (estimado vs. real) e como ele evolui ao longo do tempo?
- Métricas: `prazo_estimado_dias`, `prazo_real_dias`, `atraso` médio por mês.
- Gráficos:
  - Linhas (duas séries) para estimado vs. real por mês
  - Linha do atraso médio mensal (com banda de desvio-padrão)
- Dicas: anotar picos sazonais e mudanças operacionais.
- Snippet:
  ```python
  monthly = orders.resample('M', on='data_pedido')[['prazo_estimado_dias','prazo_real_dias','atraso']].mean()
  ```

10) Quais combinações de categoria e canal maximizam receita e satisfação?
- Métricas: ticket médio por pedido (`valor_liquido` médio), `avaliacao` média por par (categoria, canal).
- Gráficos:
  - Heatmap (categoria x canal) colorindo por ticket médio
  - Segundo heatmap para avaliação média (ou dois facetados lado a lado)
- Dicas: buscar quadrantes “alta receita + alta avaliação” para priorizar.
- Snippet:
  ```python
  pivot_ticket = orders.groupby(['categoria','canal'])['valor_liquido'].mean().unstack('canal').fillna(0)
  pivot_rate   = orders.groupby(['categoria','canal'])['avaliacao'].mean().unstack('canal').fillna(0)
  ```

## Boas práticas de visualização
- Títulos informativos que expressem a mensagem-chave.
- Eixos e unidades claros (R$, dias, %).
- Anotações em pontos de interesse (picos, eventos).
- Paletas consistentes; rotação de rótulos para legibilidade.
- Escala log quando houver assimetria intensa.
- Sempre verificar se há outliers e amostras pequenas por grupo.

## Avisos e limitações
- Correlação não implica causalidade. Use EDA para gerar hipóteses.
- Avaliações podem ser afetadas por múltiplos fatores (atraso, canal, desconto, categoria).
- Em análises percentuais, preferir `desconto_pct` ao valor absoluto de `desconto`.
