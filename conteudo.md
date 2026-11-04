# Live 04/11/2026 — Controle de dados e modelo: DVC e MLflow

**Pós Tech ML Engineering · Fase 2 (MLOps) · 19h30 às 21h00 (90 min)**
**Formato:** misto teoria + hands on, ao vivo, com gravação. Tom educativo.
**Público:** misto (quem já é de tecnologia + quem migrou). Nada de pré-requisito além de ter visto Git e Python básico.

---

## 1. Objetivo da live

Ao final, o aluno consegue:

1. Explicar **por que** código, dados e modelo precisam ser versionados juntos (reprodutibilidade).
2. Usar **DVC** para versionar um dataset, guardar num remote e voltar no tempo.
3. Usar **MLflow** para registrar experimentos, comparar execuções e registrar um modelo.
4. **Ligar os dois**: dado qual versão de dados treinou qual modelo.

**Frase-guia da aula:** *"Se eu não consigo reproduzir o resultado de ontem, eu não tenho um modelo, tenho uma sorte."*

## 2. Onde esta live se encaixa no curso

Os alunos já passaram (ou estão passando) por: *MLOps · Aula 3 (Controle de versão para ML)*, *Aula 4 (Empacotamento)* e *Aula 5 (Containers para ML)*. A live **aprofunda e põe a mão na massa** em cima de Aula 3, e prepara o terreno para CI/CD (Aula 8) e Monitoramento (Aula 9). Vale citar isso na abertura para ancorar.

> Conferir com a coordenação o que cada aula já cobriu de DVC/MLflow para não repetir slide por slide.

---

## 3. Cronograma (90 min)

| Horário | Min | Bloco | Tipo |
|---|---|---|---|
| 19:30 | 8 | Abertura: boas-vindas, objetivos, mapa, cenário e gancho + link do Colab | Teoria |
| 19:38 | 10 | Bloco 1 — O problema e o DVC (conceitos) | Teoria |
| 19:48 | 22 | Hands on 1 — DVC | Prática |
| 20:10 | 15 | Bloco 2 — MLflow e Model Registry (conceitos) | Teoria |
| 20:25 | 25 | Hands on 2 — MLflow | Prática |
| 20:50 | 10 | Bloco 3 — Juntando tudo + próximos passos + encerramento | Misto |

Sem pausa na live (grade corrida de 90 min). Folga: se algo atrasar, o que corta primeiro é o **pipeline `dvc repro`** (já é opcional) e depois o **Model Registry**, que vira demo só da professora.

---

## 4. Roteiro detalhado

### Abertura (8 min)
- Boas-vindas e onde estamos na Fase 2 (esta live aprofunda a Aula 3 de MLOps).
- O que o aluno vai saber fazer ao final (4 objetivos) e o mapa da aula.
- Cenário: pessoa de dados numa operadora de telecom, modelo de churn, e o que é AUC em uma frase (nota de 0,5 a 1).
- Gancho, contado como história curta (cenário hipotético): *"Segunda-feira o modelo tinha 82% de AUC. Sexta ninguém sabe dizer com qual dado, qual código, qual parâmetro. O que aconteceu?"*
- Pedir para abrirem o Colab (link no chat) **já agora**, e rodarem a primeira célula de instalação enquanto eu falo.
- Combinar: perguntas pelo chat, eu pego as que forem pontuais; o restante, no fórum/grupo de estudos de 18/11.

### Bloco 1 — O problema e o DVC (10 min, teoria)

> **Conceitos no início:** logo depois de "Três perguntas, três eixos" entram os slides **O que é DVC?** e **O que é MLflow?** (1 minuto cada), para o aluno ter a definição das duas ferramentas antes de qualquer detalhe.

1. **Os três eixos da reprodutibilidade em ML** (3 min): código, dados, modelo/experimento. Git resolve o primeiro. Os outros dois?
2. **Por que Git não serve para dados** (2 min): tamanho, diff inútil em binário/CSV grande, repositório inchado. Analogia: *Git guarda a receita; dado é o ingrediente, que não cabe na gaveta da receita.*
3. **Como o DVC resolve** (5 min):
   - Ideia central: o Git versiona um **arquivo pequeno de ponteiro** (`.dvc`, com o hash MD5); o dado de verdade vai para o **remote** (S3, Drive, pasta...).
   - Conceitos: `dvc add`, cache local, remote, `dvc push` / `dvc pull`, `dvc checkout`.
   - Diagrama: *workspace → cache → remote* com o Git ao lado guardando os ponteiros.
4. **Onde cada coisa mora** (2 min): tabela "vai pro Git / vai pro DVC". Ponte com a fase 2: o remote do DVC pode ser S3, o mesmo bucket de Data Lake que eles já viram.

**Slides sugeridos (5-6):** cenário-gancho · os 3 eixos · Git vs dado · ponteiro `.dvc` (diagrama) · comandos essenciais · "vai pro Git / vai pro DVC".

### Hands on 1 — DVC (22 min)
Passo a passo completo em [`hands-on.md`](hands-on.md), Parte A. Resumo:

| Min | O que acontece | Conceito que fica |
|---|---|---|
| 3 | Instalar, `git init`, `dvc init` | O que o `dvc init` cria |
| 4 | Baixar dataset v1 (5.000 linhas) e `dvc add` | Ponteiro `.dvc` + `.gitignore` |
| 3 | Configurar remote (pasta) e `dvc push` | Onde o dado realmente fica |
| 5 | "Chegaram dados novos": v2 (7.043 linhas), `dvc add`, commit, tag | Versão de dado = commit do Git |
| 4 | **Viagem no tempo**: `git checkout data-v1` + `dvc checkout` | Momento "uau" da parte 1 |
| 4 | **Desastre**: apaga dado e cache, `dvc pull` recupera | Remote como seguro de vida |

*Opcional se sobrar tempo (3 min):* `dvc.yaml` com um estágio de treino e `dvc repro`.

**Interação pontual (sem esperar resposta):** antes da viagem no tempo, perguntar no chat *"quantas linhas vocês acham que o arquivo vai ter?"* e seguir; mostrar o resultado logo depois.

### Bloco 2 — MLflow e Model Registry (15 min, teoria)

1. **O problema do "caderno de laboratório"** (1 min): rodou 30 experimentos no notebook, qual foi o melhor? Qual parâmetro? Planilha manual não escala.
2. **Soluções do MLflow** (2 min): **Tracking**, **Models** e **Model Registry** (vamos usar); **Serving**, **Evaluate** e **Tracing/GenAI** (só menção).
3. **Anatomia de uma run** (2 min): experimento → run → params / metrics / artifacts / tags. Diagrama.
4. **Model Registry em 4 slides** (10 min), o ponto forte da live:
   - *Por que existe:* o champion/challenger "na mão" (Aula 07 do curso: `production.pkl`, pasta `archive`, rollback copiando arquivo) versus versões e aliases.
   - *Versões e aliases:* `churn-model`, versões 1, 2, 3, alias `champion`, carga por alias (`@champion`) ou por versão.
   - *Ciclo de vida de uma versão:* treinar, registrar, comparar, promover (mover alias), rollback (mover de volta).
   - *Dos stages para os aliases:* o cheatsheet da Aula 07 usa `transition_model_version_stage` e `models:/nome/Production`; hoje o caminho recomendado é alias. Tabela Staging → `challenger`, Production → `champion`, Archived → versão antiga guardada.

**Slides:** caderno de laboratório · soluções do MLflow · anatomia da run · Registry (por que existe, versões e aliases, ciclo de vida, stages x aliases).

### Hands on 2 — MLflow (25 min)
Passo a passo em [`hands-on.md`](hands-on.md), Parte B. Resumo:

| Min | O que acontece | Conceito que fica |
|---|---|---|
| 4 | Preparar dados (puxados pelo DVC) e `set_experiment` | Experimento organiza as runs |
| 8 | Função `treinar()` + 3 runs (LogReg, RF depth 5, RF depth 15) | `log_params`, `log_metrics`, `log_model` |
| 3 | Comparar: `search_runs` + **UI do MLflow** (demo da professora) | Escolher o melhor com evidência |
| 4 | Registrar o melhor, alias `champion`, carregar via `models:/churn-model@champion` e prever (B6) | Modelo versionado e consumível |
| 2 | Tags e descrição na versão, tabela de versões (B7) | O Registry guarda metadados e linhagem |
| 6 | Challenger (v2), comparar, promover e rollback movendo o alias (B8 a B11) | Promover e voltar = mover alias |

### Bloco 3 — Juntando tudo + encerramento (10 min)

1. **Rastreabilidade ponta a ponta** (4 min, parte demo): a run do MLflow guarda a tag `data_md5`, que é o hash do `.dvc`. Mostrar: do modelo `champion` → run → `data_md5` → `git checkout data-v1` / `dvc checkout`. *"De um modelo em produção eu chego no dado e no código que o geraram."*
2. **Mapa do que vem** (3 min): onde isso entra em **CI/CD** (Aula 8: testes + `dvc pull` no pipeline), **Containers** (Aula 5) e **Monitoramento** (Aula 9: dado em produção vs dado de treino). Ponte para a **live de 09/12 (Docker)**.
3. **Boas práticas + erros comuns** (2 min): nunca `git add` do dado; sempre commitar o `.dvc`; fixar `random_state`; logar o que importa e não tudo; nomear experimentos.
4. **Fechamento** (1 min): recapitulação em 3 linhas, repositório e notebook disponíveis, convite para o grupo de estudos de **18/11**.

---

## 4b. O que os repositórios do curso oferecem

Repositórios analisados: [aula02](https://github.com/josenetoo/fiap-mlet-mlops-aula02), [aula05](https://github.com/josenetoo/fiap-mlet-mlops-aula05) e [aula07](https://github.com/josenetoo/fiap-mlet-mlops-aula07).

| Repositório | O que tem | Como usar na live |
|---|---|---|
| **aula02** (ingestão de dados e feature engineering) | Introdução ao DVC: `dvc init`, `dvc add`, ponteiro `.dvc`, **sem remote** (o próprio texto diz que o remote vem depois). Cheatsheet com `remote add`, `push`, `pull`, `checkout`. Dataset Iris. `dvc==3.67.1`. | Os alunos **já viram** `dvc add`. A live revisita em 1 slide e aprofunda: remote, push/pull, viagem no tempo, recuperação, e a ligação com o MLflow. |
| **aula05** (Airflow) | DAG `ml_pipeline` (ingest, validate, train, evaluate, deploy copiando `.pkl`), retry, timeout, schedule. **Sem MLflow nem DVC.** | Só como ponte no fechamento: o pipeline do Airflow poderia fazer `dvc pull` e registrar o modelo no Registry. |
| **aula07** (re-treino automático) | Champion/challenger e rollback **implementados com arquivos** (`models/production.pkl`, `archive/`, `production_meta.json`). `requirements` com `mlflow==2.8.1`. Cheatsheet com Registry usando **stages**. | É a base do slide "Registry: o champion sem arquivos soltos" e do "Dos stages para os aliases". O hands on 2 refaz o champion/challenger e o rollback com aliases. |

Pontos de atenção:
- **Versões:** a aula07 fixa `mlflow==2.8.1`. O notebook da live instala `mlflow>=3.1` no Colab. Aliases existem nas duas, mas o argumento `name=` do `log_model` é do MLflow 3. Alunos que reaproveitarem o ambiente da aula07 devem usar `artifact_path=`.
- **Dataset:** os repositórios usam Iris. A live usa Telco Churn (mais realista, e a AUC fica mais simples de explicar em duas classes). Dá para trocar se preferir continuidade.
- **Numeração:** as aulas dos repositórios (02 ingestão, 05 Airflow, 07 re-treino) **não batem** com a lista de aulas de MLOps que consta no briefing (Aula 3 controle de versão, Aula 5 containers, Aula 8 CI/CD). Confirmar qual é a grade da turma antes de citar números de aula nos slides.

## 5. Decisões já tomadas

| Decisão | Escolha |
|---|---|
| Ambiente do hands on | Google Colab |
| Remote do DVC | Pasta local (`/content/dvc-remote`); S3 só como slide/menção |
| Dataset | Telco Customer Churn (IBM), 7.043 linhas, tabular, relatável para todos. Foco na ferramenta, não no modelo |
| Backend do MLflow | SQLite local (`sqlite:///mlflow.db`) — necessário para o Registry |
| Material | Markdown no repo (este arquivo + `hands-on.md`); slides depois |

## 6. Riscos e plano B

| Risco | Impacto | Plano B |
|---|---|---|
| **Colab apaga tudo** ao desconectar (inclusive o "remote" em `/content`) | Aluno perde o estado ao pausar/voltar | Avisar logo no início; notebook roda do zero em ~5 min. Remote local é didático, **não** é para produção |
| **UI do MLflow no Colab** não abre direto | Aluno não vê a interface | Alunos usam `mlflow.search_runs()` (tabela). Eu mostro a UI no meu ambiente/tunnel e gravo prints. **Testar antes** |
| `pip install` demora ou muda versão | Atraso na abertura | Versões fixadas na célula de instalação; pedir para rodarem na abertura |
| Download do CSV fora do ar | Hands on trava | Deixar cópia do CSV no repositório/Drive como fallback |
| Atraso geral | Estoura 21h00 | Corto `dvc repro` e deixo o Registry como demo |

## 7. Pontos de atenção para o público misto

- **Vocabulário:** explicar *hash*, *commit*, *remote*, *artefato* na primeira vez que aparecem. Evitar jargão sem analogia.
- **Quem migrou para tecnologia:** terminal pode assustar. No Colab tudo vira célula com `!`; falar *"o ponto de exclamação significa: roda isso no terminal"*.
- **Quem já é da área:** dar uma "camada extra" curta (ex.: DVC com S3, `dvc.yaml`, assinatura do modelo) sem desacelerar quem está começando.
- **Gravação:** falar o que estou digitando antes de rodar; deixar cada saída na tela 3 segundos; evitar "como vocês podem ver aqui".

## 8. Próximos passos

1. Você revisa este roteiro (ajustes de tempo, ordem, ênfase).
2. Eu testo o notebook no Colab de verdade (especialmente a UI do MLflow) e gero o `.ipynb`.
3. Montamos os **slides da apresentação** (≈ 12-14 slides) e do **hands on** (≈ 6-8 slides de apoio).
