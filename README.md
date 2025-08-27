# X-Health — Inferência de Default (B2B)

Estimativa da probabilidade de **default** em pedidos B2B…

## 🎯 Objetivo
- pipeline completo…
- threshold operacional…
- função de predição…





---

## 🗃️ Dados

- Arquivo: `_data/dataset_2021-5-26-10-14.csv`  
- Leitura: `sep="\t"`, `encoding="utf-8"`, faltantes como `"missing"`.
- 1 linha = 1 pedido; variáveis internas (comportamento) e externas (bureaus).
- **Taxa de default** aproximada: **~16,7%**.

> Caso o CSV completo não possa ser publicado, incluo um **sample** e instruções para posicionar o arquivo real no mesmo caminho.

---

## ⚙️ Como rodar

### Opção A) `venv` (Python puro)

```bash
# Windows
python -m venv .venv
. .venv/Scripts/activate
pip install -r requirements.txt

# Linux/Mac
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

## ⚙️ Como rodar

### Opção A) `venv` (Python puro)

```bash
# Windows
python -m venv .venv
. .venv/Scripts/activate
pip install -r requirements.txt

# Linux/Mac
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt



Executando

Abra os notebooks em notebooks/ e rode em ordem:

01_EDA.ipynb

02_Modelagem.ipynb → ao final, serão gerados artifacts/model.pkl e artifacts/threshold.pkl

03_Predicao.ipynb → contém a função de predição pedida

Garanta que o CSV esteja em _data/ (ou ajuste o caminho no notebook)


# 🧪 Metodologia (resumo)

Split estratificado (80/20), mantendo a proporção da classe.

Pré-processamento com ColumnTransformer:

Numéricas: SimpleImputer(median) + StandardScaler

Categóricas: SimpleImputer(most_frequent) + OneHotEncoder(handle_unknown="ignore")

Modelagem: comparação de Logistic Regression, Random Forest e XGBoost.
Selecionado: RandomForest (class_weight="balanced") dentro de Pipeline.

Validação cruzada (5 folds estratificados) no treino, métricas: ROC-AUC, PR-AUC (classe rara) e Brier (qualidade da probabilidade).

Avaliação no teste com as mesmas métricas.

Escolha do threshold por F1 (equilíbrio precision × recall).
Cutoff encontrado: 0.30.

Exportação: model.pkl e threshold.pkl e função predict_default(d).


📊 Resultados (teste)

ROC-AUC: ~0.931

PR-AUC: ~0.804

Brier: ~0.072

Threshold (F1): 0.30

Matriz de confusão (t = 0.30): TN=18491, FP=1055, FN=1102, TP=2807
Classe 1 (default): Precision ≈ 0.727 | Recall ≈ 0.718 | F1 ≈ 0.722

Com t = 0.30, capturamos ~72% dos inadimplentes com ~73% de precisão.
Aumentar o cutoff ↑precision e ↓recall; diminuir o cutoff faz o oposto.


🧩 Predição (item 3 do enunciado)

A função final recebe um dicionário com novos dados e retorna apenas a predição binária:

# 03_Predicao.ipynb
predict_default({"ioi_3months": 3, "valor_vencido": 152000, "valor_total_pedido": 35000})
# -> {"default": 1}


Opcionalmente, há uma versão que também retorna a probabilidade (p_default) para diagnóstico.



🧠 Variáveis importantes (insights)

Variáveis financeiras e de eventos negativos (ex.: valor_vencido, valor_quitado, valor_por_vencer, valor_total_pedido, protestos, dividas_vencidas_qtd) dominam as importâncias.

Sinais cadastrais/temporais (ex.: tipo_sociedade, opcao_tributaria, month/year) têm contribuição menor/moderada.

Importâncias calculadas agregando dummies por variável original. Para o sentido do efeito, recomenda-se SHAP (opcional).


🧭 Política sugerida (exemplo)

Com p_default e o cutoff:

p_default < 0.15 → Aprovar

0.15 ≤ p_default < 0.30 → Revisar / ajustar limite / garantia

p_default ≥ 0.30 → Negar ou exigir garantia

Ajuste as faixas conforme o custo de erro (FN vs. FP) e o apetite de risco.

🔁 Reprodutibilidade

Semente única: RANDOM_STATE = 42

Versões: ver requirements.txt (ou environment.yml)

Pré-processamento e modelo no mesmo Pipeline (evita vazamento)

Artefatos versionados em artifacts/


🚑 Solução de problemas

Caminho do CSV: confirme _data/dataset_...csv e sep="\t" / encoding="utf-8".

Categorias novas na predição: OneHotEncoder(handle_unknown="ignore") já previne erro.

Tipos na predição: a função força numéricos/objetos antes de aplicar o pipeline.
