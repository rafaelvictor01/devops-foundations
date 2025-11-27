# 🚀 Workflow Completo: Testes, Segurança (Trivy) e Push

A `CI` de alto nível não apenas faz o `push` da imagem, mas também garante a qualidade (testes/lint) e a segurança (scanner) antes de publicá-la.

## 1. Qualidade: Testes e Lint

Os passos de qualidade devem ser executados antes de qualquer `docker build`. Se o código falhar nos testes, não devemos gastar tempo buildando ou scaneando a Imagem.

> Falhar rápido para corrigir rápido. ⚡

Pré-requisitos: Instalar Node.js e o gerenciador de pacotes (pnpm). Sem estes recursos, seria mais complicado executar os testes. Então nosso comando fica após eles.

```yaml
  - name: Configure Node
    uses: actions/setup-node@v4
    with:
      node-version: 22.15

  - name: Install pnpm
    uses: pnpm/action-setup@v4

  - name: Install dependencies
    run: pnpm install --frozen-lockfile

  - name: Run Tests
    run: pnpm run test # Você já tem o step para rodar os testes!
```

## 2. Geração de Tag Única (e o Problema da Imutabilidade)

Antes de fazer o build da nossa imagem, precisamos refletir sobre as `tags`. 🤔

Enquanto estávamos fazendo tudo localmente, era simples usarmos tags como `v1`, `v2`, etc. Mas na nossa pipeline isto deve ser dinâmico. Devemos gerar tags únicas e imutáveis.

Uma estratégia bastante comum é reaproveitar parte dos caracteres do ID do commit do código para gerar a tag. O ID do commit também é conhecido como `SHA` ou `hash` do commit. Veja como ficaria um step que gera nossa tag dinâmica baseada no `hash` do commit:

```yaml
  - name: Generate tag
    id: generate-tag
    run: |
      SHA=$(echo $GITHUB_SHA | head -c7) 
      echo "sha=$SHA" >> $GITHUB_OUTPUT

      # Pega os 7 primeiros caracteres do hash do commit
      # Salva o valor em uma variável chamada `sha` para uso em outros steps
```

## 3. Build da Imagem e Segurança (Trivy)

O passo de `docker build` pode ser seguido imediatamente pelo scanner de vulnerabilidades `Trivy`, caso seja do seu interesse *(deveria ser 🤨)*. A execução do `Trivy` aqui implementa a filosofia `"Shift Left"`—encontrar falhas de segurança o mais cedo possível na pipeline.

### A. Build da Imagem

Usamos o docker build com as variáveis de ambiente que definem o registro, repositório e tag.

```yaml
  - name: Build Image
    id: build-image
    env:
      ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}
      ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
      IMAGE_TAG: ${{ steps.generate-tag.outputs.sha }}
    run: |
      docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
```

### B. Scanner de Vulnerabilidades (Trivy)

Utilizamos a Action oficial da `Aqua Security` para rodar o scanner na Imagem recém-construída.

```yaml
  - name: Run Vulnerability Scanner
    id: run-trivy-scanner
    uses: aquasecurity/trivy-action@0.28.0
    with:
      # Referencia a imagem que acabamos de construir no step anterior
      image-ref: '${{steps.login-ecr.outputs.registry}}/${{ vars.ECR_REPOSITORY }}:${{ steps.generate-tag.outputs.sha }}'
      format: 'table'
      ignore-unfixed: true
      vuln-type: 'os,library'
      severity: 'CRITICAL,HIGH,MEDIUM,LOW' # Define quais criticidades serão reportadas
```

> 💡 Nota: Se o Trivy encontrar vulnerabilidades no nível de severity definido, este step pode ser configurado para falhar a pipeline, impedindo que a imagem seja publicada (um Quality Gate).

## 4. Push da Imagem

O step final, que só é alcançado se todos os steps anteriores (Testes, Build e Trivy) forem bem-sucedidos.

```yaml
  - name: Push the Image to AWS ECR
    id: push-image
    env:
      ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}
      ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
      IMAGE_TAG: ${{ steps.generate-tag.outputs.sha }}
    run: |
      docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

### 🎯 Alternativa ao Push: `docker/build-push-action`

Em vez de usar o `docker build` e `docker push` separadamente e de forma explicita como nos exemplos anteriores, uma alternativa popular e otimizada é a Action `docker/build-push-action`. Ela faz o `build`, o `push` e lida com otimizações de cache em um único step, simplificando o código e melhorando a performance.

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
        # with:
          # version: 8
  
      - name: Install dependencies
        id: install-dependencies
        run: pnpm install --frozen-lockfile
  
      # - name: Run Tests
      #   run: pnpm run test

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

      - name: Generate tag
        id: generate-tag
        run: |
          SHA=$(echo $GITHUB_SHA | head -c7)
          echo "sha=$SHA" >> $GITHUB_OUTPUT

      - name: Build Image
        id: build-image
        env:
          ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}
          ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
          IMAGE_TAG: ${{ steps.generate-tag.outputs.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .

      - name: Run Vulnerability Scanner
        id: run-trivy-scanner
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: '${{steps.login-ecr.outputs.registry}}/${{ vars.ECR_REPOSITORY }}:${{ steps.generate-tag.outputs.sha }}'
          format: 'table'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH,MEDIUM,LOW'

      - name: Push the Image to AWS ECR
        id: push-image
        env:
          ECR_REGISTRY: ${{steps.login-ecr.outputs.registry}}
          ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
          IMAGE_TAG: ${{ steps.generate-tag.outputs.sha }}
        run: |
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```
