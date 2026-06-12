# Tech Challenge — Fase 1 · Case NPS Preditivo

Análise de dados de um e-commerce com foco em entender **quais fatores operacionais influenciam o Net Promoter Score (NPS)** dos clientes e como a empresa pode agir de forma proativa para melhorar a experiência antes mesmo da aplicação da pesquisa.

> Projeto desenvolvido para o Tech Challenge da Fase 1 (Pós-Tech). A pontuação desta atividade corresponde a 90% da nota final da fase.

---

## 👥 Participantes

- Andre Aparecido Torres dos Santos
- Fernanda Carmona de Carvalho
- Nicolly Costa Santos do Vale
- Thyago de Oliveira Pereira
- Isabella Pereira Lima

---

## 🎯 Objetivo do projeto

Com o crescimento acelerado do e-commerce, a empresa passou a lidar com um volume cada vez maior de pedidos, entregas e interações com clientes. Esse crescimento revelou uma **alta variabilidade no NPS** entre diferentes perfis de consumidores: mesmo com indicadores operacionais semelhantes, alguns clientes se tornam promotores e outros, detratores.

Hoje o NPS é coletado **apenas após o encerramento da jornada de compra**, o que limita a capacidade da empresa de antecipar problemas e atuar de forma preventiva. O objetivo deste trabalho é **transformar dados operacionais (pedido, logística e atendimento) em insights acionáveis**, traduzindo análises técnicas em recomendações claras para áreas como logística, atendimento, produto e estratégia.

A pergunta central que orienta a análise é:

> **Quais fatores operacionais realmente influenciam a satisfação do cliente e como a empresa pode agir proativamente para melhorar a experiência antes da pesquisa de NPS?**

---

## 🗂️ Estrutura do repositório

```
TECH-CHALLENGE-FASE-1/
├── Dados/
│   └── cliente_nps.csv          # Base de dados (2.500 registros)
├── Notebook/
│   ├── analise_nps.ipynb        # Entendimento do negócio, target e EDA
│   └── Desafio 4.ipynb          # Modelo preditivo de NPS (classificação)
├── requirements.txt             # Dependências do projeto
└── README.md
```

---

## 📊 Descrição da base de dados

Arquivo: `Dados/cliente_nps.csv` — **2.500 registros** e **19 colunas** com dados históricos de pedidos, entregas e interações com o atendimento.

### Dicionário de dados

| Coluna | Descrição |
|--------|-----------|
| `customer_id` | Identificador único do cliente |
| `order_id` | Identificador único do pedido |
| `customer_age` | Idade do cliente |
| `customer_region` | Região geográfica do cliente |
| `customer_tenure_months` | Tempo de relacionamento do cliente com a empresa (em meses) |
| `order_value` | Valor total do pedido |
| `items_quantity` | Quantidade de itens no pedido |
| `discount_value` | Valor de desconto aplicado ao pedido |
| `payment_installments` | Número de parcelas do pagamento |
| `delivery_time_days` | Tempo total de entrega (em dias) |
| `delivery_delay_days` | Quantidade de dias de atraso na entrega |
| `freight_value` | Valor do frete |
| `delivery_attempts` | Número de tentativas de entrega |
| `customer_service_contacts` | Número de contatos do cliente com o atendimento |
| `resolution_time_days` | Tempo para resolução de problemas (em dias) |
| `complaints_count` | Número de reclamações registradas pelo cliente |
| `repeat_purchase_30d` | Indica se houve recompra em até 30 dias após o pedido (0 = não, 1 = sim) |
| `csat_internal_score` | Score interno de satisfação do cliente |
| `nps_score` | **Variável alvo.** Nota de satisfação do cliente (NPS), de 0 a 10, coletada após a experiência de compra |

---

## 🔬 Metodologia

O trabalho está estruturado no notebook `Notebook/analise_nps.ipynb`, seguindo as etapas do desafio:

1. **Entendimento do negócio** — reflexão analítica sobre o problema, a importância do NPS para um e-commerce, as áreas beneficiadas pelos insights e o impacto do NPS em recompra, boca a boca e market share.
2. **Definição da target** — escolha e justificativa do `nps_score` como variável que representa a satisfação do cliente, momento de coleta na jornada e riscos de uso inadequado.
3. **Análise Exploratória dos Dados (EDA)** — análise com **foco em negócio**, escrita em linguagem acessível (como se explicada a um gestor de operações que não entende de estatística), respondendo:
   - Quais fatores parecem mais críticos para a satisfação?
   - O que mais gera detratores?
   - Existe algum "ponto de ruptura" na experiência do cliente?
   - Que tipo de cliente tende a ter NPS mais alto ou mais baixo?

4. **Modelo preditivo de NPS (Desafio 4 — opcional)** — no notebook `Notebook/Desafio 4.ipynb`, uma prova de conceito que prevê, **antes da pesquisa de NPS**, se o cliente se tornará detrator, usando apenas dados operacionais da jornada de compra.

A análise da EDA utiliza estatística descritiva, **matriz de correlação** com o `nps_score` e **agrupamentos** (NPS médio por dias de atraso e por recompra), apoiada por visualizações em `matplotlib`/`seaborn`.

### Modelo preditivo (Desafio 4)

| Etapa | Decisão |
|-------|---------|
| **Abordagem** | Classificação binária — Detrator vs. Não-Detrator (preferida à regressão por ser mais acionável e estável) |
| **Variável alvo** | `target_detrator = 1` quando `nps_score ≤ 6` (padrão NPS da Bain & Co.) |
| **Features** | 17 variáveis operacionais + *feature engineering* (`delivery_issue`, `service_issue`, `high_delay`) |
| **Prevenção de *data leakage*** | Exclusão de `nps_score`, `csat_internal_score`, `repeat_purchase_30d` e identificadores |
| **Separação** | 80% treino / 20% teste, estratificado (classes desbalanceadas: ~74% detratores) |
| **Modelos comparados** | Baseline (Dummy), Regressão Logística, Random Forest, XGBoost — validação cruzada 5-fold |
| **Modelo escolhido** | **Random Forest** (melhor F1, estável e interpretável) — AUC-ROC ~0,87 e F1 ~0,88 no teste |
| **Threshold** | Calibrado em **0,40** para priorizar *recall* (capturar o máximo de detratores reais) |

O notebook ainda inclui análise de importância das features, calibração de threshold, simulação de *scoring* de novos pedidos e uma proposta de **pipeline de uso em produção** (dado operacional → previsão → alerta → ação preventiva).

---

## 💡 Principais insights

- **A experiência logística é o fator mais crítico.** O atraso na entrega (`delivery_delay_days`) tem a correlação negativa mais forte com o NPS (**-0,597**).
- **Ponto de ruptura em ~2 dias de atraso.** O NPS médio cai de **6,86** (sem atraso) para **5,55** (1 dia) e **4,58** (2 dias) — a partir daí os detratores disparam.
- **Atendimento também pesa.** Reclamações (**-0,497**), contatos com o suporte (**-0,351**) e tempo de resolução (**-0,191**) reduzem a satisfação.
- **Cliente satisfeito recompra.** Quem recomprou em 30 dias tem NPS médio de **9,01**; quem não recomprou, apenas **3,94** (correlação de **+0,570**).

**Recomendação prática:** priorizar o cumprimento dos prazos de entrega e a resolução rápida de problemas no atendimento são as ações com maior potencial de elevar o NPS.

> O desafio opcional (item 4 — modelo preditivo) foi implementado no notebook `Notebook/Desafio 4.ipynb`: um classificador Random Forest que prevê detratores a partir de dados operacionais, com AUC-ROC ~0,87 e F1 ~0,88 no conjunto de teste.

---

## ▶️ Como reproduzir os resultados

### Pré-requisitos
- Python 3.10+
- `pip`

### Passo a passo

```bash
# 1. Clone o repositório
git clone <url-do-repositorio>
cd TECH-CHALLENGE-FASE-1

# 2. (Recomendado) Crie e ative um ambiente virtual
python -m venv .venv
source .venv/bin/activate      # Linux/macOS
# .venv\Scripts\activate       # Windows

# 3. Instale as dependências
pip install -r requirements.txt

# 4. Abra os notebooks
jupyter notebook Notebook/analise_nps.ipynb   # Etapas 1 a 3 (negócio, target, EDA)
jupyter notebook "Notebook/Desafio 4.ipynb"   # Etapa 4 (modelo preditivo — opcional)
```

No Jupyter, execute as células na ordem (menu **Run → Run All Cells**).

- **`analise_nps.ipynb`** lê os dados de `../Dados/cliente_nps.csv` (caminho relativo já configurado), portanto deve ser executado a partir da pasta `Notebook/`.
- **`Desafio 4.ipynb`** espera o arquivo `desafio_nps_fase_1.csv` na pasta de execução. É a mesma base de `Dados/cliente_nps.csv` — copie/renomeie o arquivo para a pasta `Notebook/` (ou ajuste o caminho na célula de carregamento) antes de rodar.

### Dependências

`pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`, `notebook`, `ipykernel`, `scikit-learn`, `xgboost` (ver `requirements.txt`).
