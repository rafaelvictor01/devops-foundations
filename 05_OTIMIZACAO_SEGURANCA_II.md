# Otimização e Segurança II: Multistage Build

O Multistage Build é uma técnica avançada que otimiza o tamanho final e a segurança da Imagem, garantindo que as ferramentas de *build* pesadas não cheguem no ambiente de produção.

## O Conceito

A técnica segmenta a execução do `Dockerfile` em múltiplas etapas ou estágios (usando múltiplos comandos **`FROM`**).

* Cada **`FROM`** inicia um novo estágio, isolado do anterior.
* Usamos o comando **`COPY --from=<nome-do-estagio>`** para transferir *apenas* os artefatos finais (o código compilado) de um estágio para o outro.

### Exemplo Multistage (Node.js)

```dockerfile
# === Estágio 1: Build ===
# 1. Imagem completa para o build (com ferramentas de dev/compilador)
FROM node:22.15 AS builder 

RUN npm i -g pnpm 

WORKDIR /app
COPY package.json pnpm-lock.yaml ./ 
RUN pnpm install --frozen-lockfile # Instala tudo (dev + prod)

COPY . .
RUN pnpm run build # Executa o build da aplicação

# === Estágio 2: Produção (Final) ===
# 2. Imagem mais leve para o ambiente de execução final
FROM node:22.15-slim

WORKDIR /app

# Copia APENAS o código compilado e as dependências de produção do Estágio 1
COPY --from=builder /app/package.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist # Artefatos finais

RUN pnpm prune --prod # Descarta o que não for de produção do que foi copiado

EXPOSE 3333
CMD ["node", "dist/main.js"] # O comando de produção
```

- Estágio de Build: Onde todas as ferramentas pesadas (como compiladores, linter, dependências de desenvolvimento) são instaladas e o código é compilado (Ex: TypeScript para JS, React para bundle estático, etc.).

- Estágio de Produção (Runtime): Uma imagem muito mais leve (Ex: node:22.15-slim ou alpine) onde apenas os artefatos finais (o código compilado) são copiados do estágio de build.

Isso garante que ferramentas de build e dependências de desenvolvimento não entrem na imagem final de produção, que é o que você usa no servidor.