# 🤖 GitHub Actions: Introdução e Estrutura (CI)

Se a imagem é a "receita" (o Dockerfile),\
Se o contêiner é o "prato montado", ou seja: a "aplicação em execução",\
E o registros de contêineres (Container Registry) é livro completo de receitas,

**Então a estrutura de CI é o próprio cozinheiro 👨🏽‍🍳**

O que isso quer dizer? 🤔

A partir de agora, vamos automatizar a execução de todos os passos que fazíamos manualmente (instalar dependências, rodar testes, buildar a Imagem, fazer o push). Para isso, usaremos o **GitHub Actions**, que é o sistema de CI/CD nativo da plataforma GitHub.

## 1. O que é o GitHub Actions?

É uma ferramenta de automação que permite definir fluxos de trabalho (**workflows**) para serem executados em resposta a eventos no seu repositório (como `push`, `pull_request`, criação de `release`, etc.).

> 💡 Conceito: O GitHub Actions é o "motor" que orquestra os processos de CI, garantindo que a qualidade e a entrega do seu código sejam consistentes e automáticas. Ele faz por você o que seria feito na mão.

## 2. Sintaxe Geral de uma Pipeline (Workflow)

Os fluxos de trabalho são definidos em arquivos `YAML` e devem ser colocados obrigatoriamente no diretório `.github/workflows/` do seu repositório.

Exemplo: `.github/workflows/main.yml`

A estrutura de um arquivo de workflow segue esta hierarquia:

### A. TOPO DO ARQUIVO: Workflow (`name` e `on`)

O topo do arquivo define o nome do fluxo de trabalho (apenas para organização) e quando ele deve ser executado.

* `name`: Nome amigável que aparecerá no painel do Actions.

* `on`: Define os gatilhos (eventos) que iniciarão o workflow.

Exemplo:

```yaml
name: widget-server pipe ECR

on:
  pull_request: # Gatilho: Qualquer Pull Request que atinja os branches abaixo
    branches: 
      - main
    types: 
      - opened
      - reopened
      - synchronize # Rodar quando novos commits são adicionados ao PR
```

### B. ESCOPO MAIOR DO ARQUIVO: `Jobs` (Tarefas)

Um workflow é composto por um ou mais `jobs` (tarefas). Os jobs são executados em paralelo por padrão, mas podem ser configurados para rodar em sequência.

* `jobs`: Bloco que contém todas as tarefas.
  * `runs-on`: Define o ambiente (servidor virtual) onde o job será executado. Geralmente usamos `ubuntu-latest`.

```yaml
name: widget-server pipe ECR

on:
  pull_request: # Gatilho: Qualquer Pull Request que atinja os branches abaixo
    branches: 
      - main
    types: 
      - opened
      - reopened
      - synchronize # Rodar quando novos commits são adicionados ao PR

jobs:
  run-ci-aws-ecr: # Nome do Job (Ex: rodar a CI)
    name: widget-server run-ci ECR
    runs-on: ubuntu-latest # O sistema operacional do "Runner"
```

## C. `Steps` (Passos)

Cada `job` é uma sequência de `steps` (passos) que executam comandos ou usam Actions pré-existentes.

* `name`: Nome descritivo do passo.
* `uses`: Indica que você usará uma Action do GitHub Marketplace.
* `run`: Executa comandos de linha de comando (Shell, Bash).

```yaml
name: widget-server pipe ECR

on:
  pull_request:
    branches:
      - main
    types: 
      - opened
      - reopened
      - labeled
      - unlabeled
      - synchronize

jobs:
  run-ci-aws-ecr:
    name: widget-server run-ci ECR
    runs-on: ubuntu-latest

    steps:
      - name: Checkout 
        uses: actions/checkout@v4  # Action para baixar o código do repo (obrigatório)
        id: checkout

      - name: Configure Node
        id: configure-node
        uses: actions/setup-node@v4
        with:
          node-version: 22.15
```

## 3. GitHub Marketplace e o Conceito de Actions

O **GitHub Marketplace** é um repositório onde a comunidade (e o próprio **GitHub/AWS**) publica `Actions` (blocos de código reutilizáveis para pipelines de CI/CD).

Vantagem: Em vez de escrever comandos complexos, como instalar e configurar a `AWS CLI`, você simplesmente usa uma `Action` pronta (ex: `aws-actions/configure-aws-credentials@v4`). Isso acelera o desenvolvimento da sua pipeline.

Exemplo: A Action `aws-actions/amazon-ecr-login@v2` simplifica todo o processo de `aws ecr get-login-password | docker login...`, encapsulando a complexidade para você.