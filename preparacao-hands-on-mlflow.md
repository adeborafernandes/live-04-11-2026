# Preparação do hands on de MLflow (Hands on 2)

**Live hoje, 04/11, 19h30.** O hands on 2 começa às 20h25 e dura 25 minutos.
Notebook: [`notebook/live_dvc_mlflow.ipynb`](notebook/live_dvc_mlflow.ipynb) · Passo a passo das células: [`hands-on.md`](hands-on.md)

> **O que já foi testado:** o notebook inteiro, localmente, sem erros (incluindo o Registry, o challenger e o rollback), e o servidor da UI do MLflow respondeu.
> **O que NÃO foi testado:** nada disso no Colab de verdade. Por isso o passo 2 abaixo é o mais importante.

---

## 1. Colocar o notebook no Colab (10 min)

- [ ] Abrir [colab.research.google.com](https://colab.research.google.com) → **Arquivo → Fazer upload de notebook** → escolher `live_dvc_mlflow.ipynb`.
- [ ] Salvar no seu Drive com um nome claro, por exemplo `Live 04-11 - DVC e MLflow (ALUNOS)`.
- [ ] Deixar uma **segunda cópia** só sua, `... (PROFESSORA)`, para ensaiar e para a demo. A cópia dos alunos fica sempre limpa, sem saídas.

## 2. Ensaio completo, do zero (15 a 20 min), faça isso primeiro

Na cópia da professora:

- [ ] **Ambiente → Desconectar e excluir ambiente de execução** (para começar limpa, como o aluno vai começar).
- [ ] Rodar **todas as células, em ordem**, cronometrando. Anote os tempos reais:

| Etapa | Estimativa | Seu tempo |
|---|---|---|
| Célula 1 (`pip install`) | ~1 min | |
| Parte A inteira (DVC) | ~3 min de execução | |
| B3 e B4 (três experimentos) | ~1 min | |
| B8 (treino do challenger) | ~30 s | |

- [ ] Se o Colab mostrar **"Reiniciar sessão"** depois da célula 1 (acontece quando o `pip` atualiza bibliotecas já carregadas): clique, e **rode de novo da célula 2 em diante**. Se isso acontecer, avise na live: *"se aparecer o botão de reiniciar, clique e rode a partir da célula 2"*.
- [ ] Conferir que não há **nenhuma célula vermelha** e que os números batem com os checkpoints:

| Onde | Deve aparecer |
|---|---|
| Depois do `dvc add` | `churn.csv.dvc` e `.gitignore` |
| `wc -l` na v1 / v2 | 5.001 / 7.044 |
| B1 | um texto de 32 caracteres (o md5) |
| B5 | tabela ordenada por `roc_auc`, com `data_md5` em todas as linhas |
| B6 | `versão registrada: 1` |
| B8 | v1 = `champion`, v2 = `challenger` |
| B10 / B11 | `champion agora é a versão 2` / `champion voltou para a versão 1` |

- [ ] Anotar o **melhor modelo e o AUC real** que apareceram. No teste local, a regressão logística ganhou (AUC 0,843). Se for assim no Colab, o slide "Os três experimentos" (que pergunta "qual vai ganhar?") tem resposta surpreendente: vale comentar.

## 3. A interface do MLflow (a parte mais incerta, 15 min)

Rode a célula opcional da B5 na cópia da professora e veja qual opção funciona:

- [ ] **Opção 1, iframe do Colab** (`serve_kernel_port_as_iframe`): a interface deve aparecer dentro do notebook. Se carregar, **use esta**.
- [ ] **Opção 2, `cloudflared`** (célula seguinte): só se a 1 falhar. Ela imprime um link `https://....trycloudflare.com`; abra em outra aba.
- [ ] Se **nenhuma** funcionar, siga o **plano B** (seção 6). Não gaste a live tentando consertar.

**Escolha agora** qual será a sua demo e escreva no topo da sua cópia:

> Demo da UI: ☐ iframe ☐ cloudflared ☐ prints

**O que mostrar na UI** (30 a 60 s cada):
1. Experimento **churn** → lista de runs, ordenar por `roc_auc`.
2. Selecionar 2 ou 3 runs → **Compare**.
3. Abrir uma run → params, metrics, artifacts e a tag `data_md5`.
4. Menu **Models** → `churn-model` → versões, aliases (`champion`, `challenger`) e tags. Este é o ponto alto, porque conecta com os slides do Registry.

## 4. Backups para ter prontos (15 min)

- [ ] **Cópia com saídas salvas:** na cópia da professora, depois de rodar tudo, **Arquivo → Salvar uma cópia no Drive** (as saídas ficam junto). Se o Colab cair ao vivo, você abre esta e mostra os resultados.
- [ ] **Prints da UI** (4 imagens: lista de runs, comparação, run aberta, aba Models com aliases). Salve numa pasta fácil de abrir.
- [ ] **Gravação de tela curta** (1 a 2 min) da UI funcionando, opcional.
- [ ] **O `.ipynb` local** guardado (`aulas/live-04-11-dvc-mlflow/notebook/`), caso o Drive falhe.

## 5. Compartilhar com os alunos (5 min)

- [ ] Na cópia **dos alunos**: **Compartilhar → Qualquer pessoa com o link → Leitor**. (Eles abrem e usam *Salvar uma cópia no Drive*; é o que o slide "Abra o notebook agora" diz.)
- [ ] Testar o link numa **janela anônima**: abre sem pedir permissão?
- [ ] Deixar o link **pronto para colar no chat** (e anotado num bloco de notas). O slide "Abra o notebook agora" não tem link, então você cola no chat quando chegar nele (slide 5).
- [ ] Se quiser um QR code no slide, me envie o link que eu acrescento.

## 6. Plano B, se algo quebrar ao vivo

| Problema | O que fazer |
|---|---|
| UI do MLflow não abre | Mostrar os **prints** e seguir. Os alunos veem os mesmos dados na tabela do `search_runs`. Dizer: *"a interface mostra o que a tabela mostra, de forma visual"*. |
| Colab cai ou desconecta | Abrir a **cópia com saídas salvas** e narrar. Os alunos refazem depois pela gravação. |
| Célula 1 pede reiniciar | Reiniciar e rodar da célula 2 (já combinado acima). |
| Aluno com erro individual | *"Anota a célula e a mensagem no chat; vemos no grupo de estudos de 18/11."* Não pare a turma. |
| Estou atrasada | Cortar nesta ordem: **UI (B5)**, depois **B7** (tags e descrição), depois **B9** (junta com B10). Nunca cortar B8, B10 e B11 (challenger, promoção e rollback): são o ponto da aula. |

## 7. Roteiro cronometrado do hands on 2 (25 min)

| Min | Célula | O que falar |
|---|---|---|
| 2 | **B1** preparar dados | "Os dados vêm do DVC. Esse hash é a impressão digital da versão." |
| 1 | **B2** configurar | "SQLite local: é o que habilita o Registry." |
| 4 | **B3** função de treino | Ler `log_params`, `log_metrics`, `set_tag`, `log_model`: *"o que escolhi, o que obtive, contexto, o artefato"*. |
| 3 | **B4** três experimentos | Pedir palpites no chat (sem esperar) enquanto roda. |
| 4 | **B5** comparar + UI | Tabela primeiro, depois a UI (seção 3). |
| 2 | **B6** registrar | "Versão 1, alias `champion`. Quem consome só conhece o alias." |
| 2 | **B7** versões e tags | Mostrar a tabela de versões e a aba **Models**. |
| 2 | **B8** challenger | "Versão 2, alias `challenger`. Nada mudou em produção." |
| 1 | **B9** comparar | "A decisão vem de comparação, não de palpite." |
| 2 | **B10** promover | "Mover o alias. Mesmo endereço, outro modelo." Compare com o `production.pkl` da Aula 07. |
| 1 | **B11** rollback | "Nenhum arquivo copiado." |
| **24** | | 1 min de folga. A **Parte C** (rastreabilidade) fica no bloco de fechamento. |

## 8. Hoje, antes da live (linha do tempo sugerida)

- [ ] **Agora:** passos 1 e 2 (upload e ensaio completo).
- [ ] **Em seguida:** passo 3 (UI) e passo 4 (backups).
- [ ] **1 hora antes:** passo 5 (compartilhar e testar o link) e abrir as abas: Colab dos alunos (limpo), Colab da professora (com saídas), UI do MLflow, slides.
- [ ] **15 min antes:** fechar o que não precisa, silenciar notificações, deixar o link do notebook pronto para colar no chat.
- [ ] **Na live, slide 5 (Abra o notebook agora):** colar o link no chat e pedir para rodarem as células 1 e 2.

## 9. Se der erro no ensaio

Anote a **célula** e a **mensagem de erro** e me envie. Os candidatos mais prováveis, por ordem:
1. Versões de bibliotecas do Colab diferentes das do meu teste (o `pip install` instala as mais novas).
2. A interface do MLflow no iframe (nunca testei no Colab).
3. Download do dataset (usa o GitHub da IBM; se cair, o notebook precisa de um arquivo local como alternativa).
