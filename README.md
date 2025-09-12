# Projeto Lighthouse – Ciência de Dados no Setor Cinematográfico


<img src="img/capa_projeto_lighthouse.png" alt="Capa do Projeto" width="1070" height="500">


# Project Lighthouse — Movie Analytics
> Apoiar a decisão de um estúdio (PProductions) sobre **qual tipo de filme produzir a seguir**, por meio de EDA, respostas a perguntas de negócio e modelos preditivos (incluindo previsão da **nota IMDb**).

---
## 1) Problema de Negócio
A diretoria precisa de **evidências de dados** para priorizar tipos de filmes com melhor potencial. Devemos:
* Explorar e **gerar hipóteses** (EDA).
* Responder:
  a) **Recomendação de filme** para “público desconhecido”;
  b) **Fatores relacionados a alta receita**;
  c) **O que extrair do *overview*** (sinopse) e se dá para **inferir gênero**.
* **Prever a nota IMDb** e **salvar o modelo**.

---
## 2) Estratégia de Solução
1. **03** — Matching das bases **Lighthouse × Kaggle** (82,2% de cobertura) e *enrichment*.
2. **K01** — Opção pela base mais robusta (Kaggle), consolidar e otimizar (metadata, credits, links, ratings).
3. **K02** — EDA dirigida a hipóteses (H1–H8) + quadro-resumo.
4. **K03** — Modelagem:
   * **H9**: texto → gêneros (multilabel).
   * **H10**: fatores de faturamento.
   * **H11**: regressão para **nota IMDb** (pipeline `prep+model` salvo em `.pkl`).
5. **Relatórios**: técnico e de negócios consolidados.
6. **Notebook final**: narrativa única para a banca (EDA → modelos → respostas → conclusões).

---
## 3) Dados & Fontes
* **Lighthouse/Indicium** (999 filmes).
* **Kaggle** (`movies_metadata`, `credits`, `links`, `ratings`, `keywords`). (+40k filmes).
* **Enriquecimento & chaves**: `title_norm + Year` e `original_title_norm + Year` (fallback).
* Saídas principais:
  * `data/processed/kaggle_otimizado.csv`
  * `data/processed/lighthohouse_enriched.csv`
  * `data/processed/matching_diagnostics.csv`, `data/processed/unmatched_lighthouse.csv`

---
## 4) Estrutura do Repositório
```
Project_lighthouse/
├─ data/
│  ├─ raw/               # bases originais (Lighthouse + Kaggle)
│  ├─ intermediary/      # limpezas e chaves de match
│  └─ processed/         # bases finais para análise/modelagem
├─ docs/                 # relatórios (técnico, negócios) e anexos
├─ img/                  # figuras (EDA, correlações, capa)
├─ models/               # artefatos (.joblib / .pkl)
├─ notebooks/
│  ├─ 00_setup_e_context
│  ├─ 01_data_overview_lighthouse
│  ├─ 02_data_overview_kaggle_metabase
│  ├─ 03_matching_merge_lh_kaggle
│  ├─ K01_merge_limpeza_otimizacao_kaggle
│  ├─ K02_eda_kaggle
│  └─ K03_modelagem_treinamento
├─ reports/              # tabelas de métricas exportadas
├─ src/, utils/          # (reservado p/ funções reutilizáveis)
├─ README.md
├─ requirements.txt
└─ 00_final_movie_analytics.ipynb   # notebook final (apresentação)
```

---
## 5) Como Reproduzir
### 5.1 Ambiente
```bash
python -m venv .venv && source .venv/bin/activate  # (Windows: .venv\Scripts\activate)
pip install -r requirements.txt
```

### 5.2 Ordem de execução (sugestão)
1. `notebooks/00_setup_e_context`
2. `01_data_overview_lighthouse` → salva `lighthouse_clean.csv`
3. `02_data_overview_kaggle_metabase` → salva chaves `kaggle_movies_key_*`
4. `03_matching_merge_lh_kaggle` → salva `lighthohouse_enriched.csv`
5. `K01_merge_limpeza_otimizacao_kaggle` → `kaggle_otimizado.csv`
6. `K02_eda_kaggle` → **quadro H1–H8** + figuras em `img/`
7. `K03_modelagem_treinamento` → **salva modelo** `models/h11_imdb_rating_model.pkl` e  `h9_multilabel_pipeline.joblib`
8. `00_final_movie_analytics.ipynb` → narrativa final

---
## 6) Principais Achados (EDA → Negócios)
* **H1** Orçamento ↑ → Receita ↑ (⚠️ parcial): cresce a receita bruta, **não** garante ROI.
* **H2** Popularidade antecipa bilheteria (✅): bom *leading indicator* de \$.
* **H3** Nota IMDb ↑ → Receita ↑ (❌.): melhora longevidade, não garante bilheteria.
* **H4** Runtime alto (>150m) tende a reduzir nota (❌ sem correlação).
* **H5** Gênero importa (medianas diferentes), mas **dispersão** é alta (sem “gênero seguro”).
* **H6** Diretor/Elenco ajudam receita; **ROI** depende de custo.
* **H7** Recomendação p/ desconhecido: **alta nota + muitos votos** (consenso). Exemplos: **Dilwale Dulhania Le Jayenge (1995)** e **The Shawshank Redemption (1994)**
* **H8** Após 2010: **engajamento ↑**, **mediana de receita ↓** (fragmentação/streaming).

> O quadro completo H1–H8 é exibido no final do `K02_eda_kaggle`.

---
## 7) Modelagem (H11 — Previsão da Nota IMDb)
* **Problema:** regressão supervisionada da `vote_average` (0–10).
* **Features**:
  * Num: `log_budget`, `log_revenue`, `roi`, `runtime`, `popularity`, `vote_count`, `mean_rating`, `num_ratings`, `overview_len`, `year`.
  * Cat (top-K + “Other”): `original_language`, `release_season`, `genre_primary`, `director`, `star1..4`.
  * Texto: `overview` (TF-IDF 1–2gram, vocabulário enxuto).
* **Modelos testados:** Linear / Ridge / ElasticNet / **RandomForest** / **HistGradientBoosting (HGB)**.
* **Melhores resultados (teste):**
  * **HGB** → **RMSE ≈ 1.13**, **R² ≈ 0.66**
  * **RF**  → RMSE ≈ 1.14, R² ≈ 0.65
  * Lineares ficaram bem abaixo (underfitting).
* **Modelo entregue:** `models/h11_imdb_rating_model.pkl` (**pipeline completo** `prep+model`).
* **Case do enunciado (Shawshank):** previsto **6,34** vs. real **8,50** → erro ≈ **2× RMSE** (fora da média; provável efeito de prestígio/época e redução de cardinalidade em diretor/atores).


7.1) Classificação de Gêneros (H9) — curto

Artefato: models/h9_multilabel_pipeline.joblib
Tarefa: prever gêneros a partir do overview (multirrótulo).
Como foi treinado (resumo): TF-IDF (1–2-gram) → One-vs-Rest com classificador linear.
Entrada/Saída: texto da sinopse → vetor binário (e, se disponível, probabilidades por gênero).


---
## 8) Respostas ao Desafio
* **(1) Filme recomendado p/ desconhecido:** top **nota** com **muitos votos** (consenso social).
* **(2) Fatores ligados a alta receita:** orçamento (para receita bruta), **popularidade/engajamento** (para ROI), janela/sazonalidade, gênero e talentos —  **ROI** exige equilíbrio de custo.
* **(3) Overview/gênero:** o texto traz sinal semântico; **dá para inferir gênero** (H9 multilabel).
  * **- Modelo salvo:** `h9_multilabel_pipeline.joblib`.
* **(4) Previsão da nota IMDb:** regressão; `TF-IDF + One-Hot + num (log/escala)`; **HGB/RF** melhores; métricas **RMSE/MAE/R²**.
  * **- Nota do exemplo:** **6,34** (vs. 8,50).
  * **- Modelo salvo:** `h11_imdb_rating_model.pkl`.
* **(5) Entrega**: README, `requirements.txt`, relatórios (PDF/Notebook), código de modelagem, `.pkl` e `.joblib`

---

## 9) Próximos Passos
* Fazer o modelo de previsão sobre a base enriquecida da lighthouse e comparar os resultados.
* Reorganizar e refatorar os códigos.
* H9 (texto → gêneros): Vamos aumentar F1-micro em ≥3–5 p.p. (e F1-macro em ≥2–3 p.p.) via (i) vocabulário  `TF-IDF max_features` / embeddings, (ii) calibração de threshold/Top-K, (iii) balanceamento por classe.
* H11 (regressão): Vamos reduzir o RMSE em ≥20% e elevar o R² em +0.03–0.05 com (i) **top-K** de `director`/`stars`; (ii) criar `decade`, (iii) TF-IDF mais rico e/ou blending.”
* Avaliar **ROI** por **mix** (gênero × orçamento × janela × elenco) com *uplift* simples.
* Exportar **figuras de importância/SHAP** para o relatório técnico.

---
## 10) Como usar o modelo salvo
1. **Carregue o pipeline salvo:**
```bash
import pickle
with open("models/h11_imdb_rating_model.pkl", "rb") as f:
    obj = pickle.load(f)
pipe = obj["pipe"] if isinstance(obj, dict) else obj
```

2. **Prepare os dados de entrada:**
Use a função ``build_row_from_lighthouse(movie_dict, req_cols)`` para montar uma linha com as mesmas features do treino.
/
3. **Faça a Predição:**
```bash
x_new = build_row_from_lighthouse(movie, req_cols).reindex(columns=req_cols)
pred = pipe.predict(x_new)[0]
print(round(pred, 2))
```

**Dica:** Veja exemplos completos no notebook K03_modelagem_treinamento.ipynb.

---
## 11) Requisitos
Instale via:
```bash
pip install -r requirements.txt
```
> Mínimo esperado: `numpy`, `pandas`, `scikit-learn`, `matplotlib`, `seaborn`, `joblib`, `pyarrow` (e demais usados no projeto).

---
## 12) Contato
* **Autor:** Emerson Carlos de Oliveira
* **Email:** `emerson_uo@hotmail.com`
* **Linkedin:** https://www.linkedin.com/in/emerson-carlos-oliveira/
* **Página de Portfólio:** https://sites.google.com/view/emerson-oliveira-portfolio

