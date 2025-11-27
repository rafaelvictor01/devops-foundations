# 🔐 Configurando Credenciais e Autenticação (AWS ECR)

Um dos maiores desafios da `CI` é o gerenciamento seguro de credenciais. Nunca podemos expor chaves de acesso (como a `AWS_SECRET_ACCESS_KEY`) diretamente no arquivo `YAML` da pipeline, que é visível publicamente. 🤨

## 1. Gerenciando Segredos com GitHub Secrets

O GitHub oferece o recurso `Secrets` para armazenar informações sensíveis (como chaves de API ou tokens) de forma criptografada.

* **Como usar**: Dentro do seu workflow, você referencia o valor do segredo usando a sintaxe `${{ secrets.NOME_DO_SEGREDO }}`. O GitHub injeta o valor no momento da execução, mas ele nunca é logado no console.

## 2. Ação de Configuração da AWS

Em vez de usar comandos manuais, utilizamos a `Action` oficial da `AWS` para configurar as credenciais no ambiente do runner.

* Action: `aws-actions/configure-aws-credentials@v4`
* Parâmetros: Recebe as chaves e a região como inputs.

Exemplo:

```yaml
  - name: Configure AWS Credentials
    id: configure-aws-credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      # Segredos injetados de forma segura
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }} 
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      # Variáveis do GitHub (não sensíveis)
      aws-region: ${{ vars.AWS_REGION }} 
```

**OBS**: Tanto as `secrets` quando as `vars` voce configura rapidamente por dentro das configurações de segurança do seu repositório, pela tela mesmo.

## 3. Autenticando o Docker no ECR

Uma vez que as credenciais da `AWS` estejam configuradas, precisamos que o `Docker CLI` no runner saiba se comunicar com o `ECR (Elastic Container Registry)`.

* Action: `aws-actions/amazon-ecr-login@v2`
* Função: Esta Action executa automaticamente o comando de autenticação (similar a `aws ecr get-login-password | docker login...`) e salva as informações necessárias para que os comandos Docker subsequentes funcionem.

```yaml
      - name: Login to AWS ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
        # Resultado desta action pode ser recuperado em ${{steps.login-ecr.outputs.registry}}
```

Veja como ficaria uma pipeline mais completa que usa estes recursos:

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
        uses: actions/checkout@v4
        id: checkout

      - name: Configure Node
        id: configure-node
        uses: actions/setup-node@v4
        with:
          node-version: 22.15

      - name: Install pnpm
        id: install-pnpm
        uses: pnpm/action-setup@v4

      - name: Install dependencies
        id: install-dependencies
        run: pnpm install --frozen-lockfile
  
      - name: Configure AWS Credentials
        id: configure-aws-credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ vars.AWS_REGION }}
  
      - name: Login to AWS ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
```