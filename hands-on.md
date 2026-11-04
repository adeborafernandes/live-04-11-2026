# Hands on — DVC + MLflow no Google Colab

Cada bloco abaixo é **uma célula** do notebook. Linhas com `!` rodam no terminal; `%cd` muda de pasta de verdade no Colab (`!cd` não funciona).

> **Notebook pronto:** [`notebook/live_dvc_mlflow.ipynb`](notebook/live_dvc_mlflow.ipynb) reúne todas as células abaixo. Foi executado de ponta a ponta localmente (célula por célula, 0 erros; dvc 3.67, mlflow 3.1, sklearn 1.6) e o servidor da UI do MLflow respondeu 200. **Ainda não foi testado no Colab**: o iframe da UI e o túnel `cloudflared` dependem do Colab. Testar antes da live.

**Dataset:** Telco Customer Churn (IBM) — 7.043 clientes, coluna-alvo `Churn` (Yes/No).

---

## Parte 0 — Preparação (rodar na abertura)

```python
# Célula 1 — instalar
!pip install -q dvc mlflow scikit-learn pandas
```

```python
# Célula 2 — identidade do Git e pasta do projeto
!git config --global user.email "aluno@exemplo.com"
!git config --global user.name "Aluno"
!git config --global init.defaultBranch main

!mkdir -p /content/projeto-churn
%cd /content/projeto-churn
```

---

## Parte A — DVC (25 min)

### A1. Iniciar Git e DVC
```python
!git init
!dvc init
!git commit -m "inicia projeto com dvc"
```
**Falar:** o `dvc init` criou a pasta `.dvc/` (configuração) e o `.dvcignore`. Mostrar com `!ls -a`.

### A2. Dados v1 (5.000 linhas)
```python
import pandas as pd, os
os.makedirs("data/raw", exist_ok=True)

URL = "https://raw.githubusercontent.com/IBM/telco-customer-churn-on-icp4d/master/data/Telco-Customer-Churn.csv"
df = pd.read_csv(URL)
df.iloc[:5000].to_csv("data/raw/churn.csv", index=False)   # v1: só as 5.000 primeiras
print(df.shape)
```

```python
!dvc add data/raw/churn.csv
!cat data/raw/churn.csv.dvc
!cat data/raw/.gitignore
```
**Falar:** o CSV foi para o *cache* do DVC, o Git **ignora** o CSV e guarda só o ponteiro `churn.csv.dvc` (com o hash MD5). É esse arquivinho que vai pro Git.

```python
!git add data/raw/churn.csv.dvc data/raw/.gitignore
!git commit -m "dados v1: 5000 linhas"
!git tag data-v1
```

### A3. Remote (pasta local) e push
```python
!dvc remote add -d storage /content/dvc-remote
!git add .dvc/config
!git commit -m "configura remote do dvc"
!dvc push
```
```python
!find /content/dvc-remote -type f
```
**Falar:** é aqui que o dado de verdade mora. Num projeto real esse remote seria um bucket S3/GCS/Azure; só muda a URL (`s3://meu-bucket/dvc`). **Aviso:** `/content` some quando o Colab desconecta.

### A4. Chegaram dados novos (v2, 7.043 linhas)
```python
df.to_csv("data/raw/churn.csv", index=False)   # v2: base completa
```
```python
!dvc add data/raw/churn.csv
!git add data/raw/churn.csv.dvc
!git commit -m "dados v2: base completa"
!git tag data-v2
!dvc push
```
**Falar:** o mesmo caminho de arquivo, outro hash. O diff do Git mostra **só o hash mudando**, não 7 mil linhas.
```python
!git diff data-v1 data-v2 -- data/raw/churn.csv.dvc
```

### A5. Viagem no tempo
```python
!wc -l data/raw/churn.csv          # v2
!git checkout data-v1
!dvc checkout
!wc -l data/raw/churn.csv          # voltou para v1
```
```python
!git checkout main
!dvc checkout
!wc -l data/raw/churn.csv          # v2 de novo
```
> `wc -l` conta uma linha a mais por causa do cabeçalho: v1 = 5.001, v2 = 7.044.

### A6. Desastre e recuperação
```python
!rm -rf data/raw/churn.csv .dvc/cache
!ls data/raw
!dvc pull
!wc -l data/raw/churn.csv
```
**Falar:** apaguei o dado **e** o cache local. O `dvc pull` buscou do remote. Este é o motivo de ter remote.

### (Opcional) A7. Pipeline com `dvc repro` — só se sobrar tempo
Fica como material complementar: `dvc.yaml` com estágio `train` (deps, params, outs, metrics) + `dvc repro`, `dvc metrics diff`. Não está no roteiro principal.

---

## Parte B — MLflow (25 min)

### B1. Preparar os dados (já versionados pelo DVC)
```python
import warnings; warnings.filterwarnings("ignore")
import pandas as pd, yaml
from sklearn.model_selection import train_test_split

df = pd.read_csv("data/raw/churn.csv")
df["TotalCharges"] = pd.to_numeric(df["TotalCharges"], errors="coerce").fillna(0)

y = (df["Churn"] == "Yes").astype(int)
X = df.drop(columns=["Churn", "customerID"])
cat_cols = X.select_dtypes("object").columns.tolist()

Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

# A "impressão digital" da versão de dados que estamos usando (vem do DVC)
data_md5 = yaml.safe_load(open("data/raw/churn.csv.dvc"))["outs"][0]["md5"]
print(data_md5)
```

### B2. Configurar o MLflow
```python
import mlflow, mlflow.sklearn
from mlflow import MlflowClient
from mlflow.models import infer_signature

mlflow.set_tracking_uri("sqlite:///mlflow.db")   # banco local (necessário para o Registry)
mlflow.set_experiment("churn")
```

### B3. Função de treino com tracking
```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score

def treinar(nome, modelo, params):
    pre = ColumnTransformer([("cat", OneHotEncoder(handle_unknown="ignore"), cat_cols)],
                            remainder="passthrough")
    pipe = Pipeline([("pre", pre), ("clf", modelo)])

    with mlflow.start_run(run_name=nome) as run:
        mlflow.log_params(params)
        mlflow.set_tag("data_md5", data_md5)        # <- ponte com o DVC

        pipe.fit(Xtr, ytr)
        pred = pipe.predict(Xte)
        proba = pipe.predict_proba(Xte)[:, 1]

        mlflow.log_metrics({
            "accuracy": accuracy_score(yte, pred),
            "f1": f1_score(yte, pred),
            "roc_auc": roc_auc_score(yte, proba),
        })
        mlflow.sklearn.log_model(pipe, name="model",
                                 signature=infer_signature(Xte, pred),
                                 input_example=Xte.head(3))
        return run.info.run_id
```
**Falar, linha a linha:** params = "o que eu escolhi", metrics = "o que eu obtive", tag = "etiqueta de contexto", model = "o artefato".

### B4. Três experimentos
```python
treinar("logreg",  LogisticRegression(max_iter=1000),
        {"modelo": "logreg", "max_iter": 1000})
treinar("rf-prof5", RandomForestClassifier(max_depth=5, random_state=42),
        {"modelo": "rf", "max_depth": 5})
treinar("rf-prof15", RandomForestClassifier(max_depth=15, random_state=42),
        {"modelo": "rf", "max_depth": 15})
```

### B5. Comparar
```python
runs = mlflow.search_runs(order_by=["metrics.roc_auc DESC"])
runs[["tags.mlflow.runName", "params.modelo", "params.max_depth",
      "metrics.accuracy", "metrics.f1", "metrics.roc_auc", "tags.data_md5"]]
```

**UI do MLflow (demo da professora).** Opções, em ordem de preferência — *nenhuma foi testada no Colab ainda*:
1. Iframe do próprio Colab: `google.colab.output.serve_kernel_port_as_iframe(5000)` depois de subir `mlflow ui` em segundo plano (está no notebook, sem conta e sem download).
2. Túnel `cloudflared` (baixa um binário oficial da Cloudflare; no notebook, como plano B).
3. Plano C: rodar o mesmo notebook localmente com `mlflow ui` (no Mac, a porta 5000 é ocupada pelo AirPlay Receiver: use `--port 5055`) e compartilhar a tela, ou usar prints.

Os alunos acompanham pela tabela do `search_runs`. Na UI, mostrar: lista de runs, ordenar por `roc_auc`, comparar 2 runs, abrir uma run e ver params/metrics/artifacts/tag `data_md5`.

### B6. Registry e alias
```python
melhor = runs.iloc[0].run_id
mv = mlflow.register_model(f"runs:/{melhor}/model", "churn-model")
MlflowClient().set_registered_model_alias("churn-model", "champion", mv.version)
print("versão registrada:", mv.version)
```

```python
modelo = mlflow.pyfunc.load_model("models:/churn-model@champion")
modelo.predict(Xte.head(5))
```
**Falar:** quem consome o modelo não sabe o número da versão, só o alias. Para trocar o modelo em produção, **move-se o alias** e o código de quem consome não muda.

### B7. Explorando o Registry: versões, tags e descrição
```python
client = MlflowClient()
client.set_model_version_tag("churn-model", mv.version, "data_md5", data_md5)
client.set_model_version_tag("churn-model", mv.version, "validado", "sim")
client.update_model_version("churn-model", mv.version, description=f"Melhor run por roc_auc: {runs.iloc[0]['tags.mlflow.runName']}")
mostrar_versoes()   # função definida no notebook: versão, aliases, run, validado
```
**Falar:** cada versão guarda de onde veio (a run), tags e descrição. Na UI do MLflow, o menu **Models** mostra o mesmo.

### B8. Um challenger entra em cena
Treina `gb-challenger` (GradientBoosting), registra como **versão 2** e dá o alias `challenger`. O `champion` segue na versão 1.

### B9. Champion x challenger
Compara o `roc_auc` das duas versões pelo alias (`get_model_version_by_alias`). **Falar:** a promoção vem de uma comparação, não de um palpite. No teste local a diferença foi pequena (0,8430 contra 0,8442).

### B10. Promover: mover o alias
```python
anterior = vc
client.set_registered_model_alias("churn-model", "champion", vh)
```
Carregar `models:/churn-model@champion` de novo: mesmo endereço, outro modelo. **Falar:** aqui promovemos para ver o mecanismo; na vida real só com a comparação aprovando (por exemplo, uma margem mínima de ganho, como na aula07).

### B11. Rollback: mover o alias de volta
```python
client.set_registered_model_alias("churn-model", "champion", anterior)
```
**Falar:** nenhum arquivo foi copiado. Compare com o `rollback()` da aula07, que copiava o `.pkl` arquivado.

---

## Parte C — Juntando tudo (demo, 4 min)

```python
champ = MlflowClient().get_model_version_by_alias("churn-model", "champion")
run = mlflow.get_run(champ.run_id)
print("Modelo champion -> run:", champ.run_id)
print("Treinado com dados (md5):", run.data.tags["data_md5"])
```
```python
# E esse hash é o mesmo que o DVC tem para a versão atual dos dados
!cat data/raw/churn.csv.dvc | grep md5
# Para recuperar exatamente esses dados em qualquer máquina:
#   git checkout data-v2 && dvc checkout   (ou dvc pull)
```
**Fecho:** modelo → run → hash dos dados → versão no Git → dado no remote. Reprodutibilidade fechada.

---

## Problemas comuns (para ter na manga)

| Sintoma | Causa | Solução |
|---|---|---|
| `dvc: command not found` | instalação não terminou | rodar a célula 1 de novo |
| `git commit` pede identidade | célula 2 não rodou | rodar célula 2 |
| `dvc push` diz "everything is up to date" | já foi enviado | normal |
| `dvc add` falha dizendo que o arquivo está sob o Git | CSV foi commitado antes | `git rm --cached data/raw/churn.csv` e repetir |
| `ModuleNotFoundError` depois de reconectar | Colab reiniciou | rodar de novo as células 1 e 2 e refazer o projeto |
| Registry reclama de store | tracking URI sem banco | usar `sqlite:///mlflow.db` (B2) |
| Resultados diferentes entre alunos | splits/seeds diferentes | manter `random_state=42` |

---

## Checkpoints: o que o aluno deve ver

**Parte A**
- Depois do `dvc add`: aparecem `churn.csv.dvc` e `.gitignore` em `data/raw`.
- Depois do `dvc push`: o arquivo aparece dentro de `/content/dvc-remote`.
- Viagem para `data-v1`: `wc -l` mostra 5.001. De volta para `data-v2`: 7.044.
- Depois do `dvc pull`: o arquivo volta com 7.044 linhas.

**Parte B**
- B1: um texto de 32 caracteres (o md5 do `.dvc`).
- B4: cada célula roda sem erro e o experimento `churn` passa a ter 3 runs.
- B5: tabela ordenada por `roc_auc`, com `data_md5` em todas as linhas.
- B6: `versão registrada: 1` e, no `predict`, um array de 0 e 1 com 5 valores.
- B8: tabela com v1 = `champion` e v2 = `challenger`.
- B10: `champion agora é a versão 2`. B11: `champion voltou para a versão 1`.

> As métricas podem variar um pouco entre ambientes; o que importa é a estrutura da saída.
