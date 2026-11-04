# Live 04/11/2026: Controle de dados e modelo com DVC e MLflow

Pós Tech · Machine Learning Engineering · Fase 2 (MLOps)

Notebook do hands on da live de 04/11/2026, das 19h30 às 21h00.

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/adeborafernandes/live-04-11-2026/blob/main/notebook/live_dvc_mlflow.ipynb)

## O que tem aqui

[`notebook/live_dvc_mlflow.ipynb`](notebook/live_dvc_mlflow.ipynb): um notebook para rodar no Google Colab, em três partes.

| Parte | O que você faz |
|---|---|
| **A. DVC** | Versiona um dataset, guarda no remote, volta no tempo entre duas versões e recupera um dado apagado |
| **B. MLflow e Registry** | Treina três modelos, compara as runs, registra o melhor, e promove um challenger com rollback usando aliases |
| **C. Juntando tudo** | Liga o modelo em produção à versão exata dos dados que o treinou |

## Como usar

1. Clique no botão **Abrir no Colab** acima (é preciso ter uma conta Google).
2. Salve uma cópia: *Arquivo → Salvar uma cópia no Drive*.
3. Rode as células **na ordem**, de cima para baixo.

## Dicas

- **Instalação:** a primeira célula instala as bibliotecas e leva cerca de 1 minuto. Se o Colab mostrar **"Reiniciar sessão"**, clique em reiniciar e rode **a partir da célula 2**.
- **Recomeçar do zero:** rode de novo a **célula 2**, que apaga o projeto anterior, e depois as células seguintes.
- **Pasta temporária:** os arquivos ficam em `/content`, que some quando o Colab desconecta. Se isso acontecer, rode tudo de novo.
- **Cada treino cria uma run nova:** se você rodar a célula de treino duas vezes, a tabela de comparação terá duas runs de cada modelo.
- **Interface do MLflow:** as duas células marcadas como opcionais abrem a interface web do MLflow. A segunda baixa um programa externo (`cloudflared`); use só se a primeira não funcionar.

## Dataset

[Telco Customer Churn](https://github.com/IBM/telco-customer-churn-on-icp4d), da IBM, baixado direto do GitHub pelo próprio notebook.
