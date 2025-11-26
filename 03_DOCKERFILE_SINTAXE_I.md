# Dockerfile: Construção Passo a Passo (Node.js)

Vamos construir nossa receita, pensando no que uma aplicação Node.js precisa: instalar dependências (`pnpm i`), fazer o build (`pnpm run build`) e, por fim, rodar a aplicação.

Lembre-se, o primeiro passo é definir a nossa *imagem base* como o comando `FROM`. Em seguida, outros comandos que sempre vão ser do seu interesse e sempre vão estar nos seus `Dockerfile`s são:

### 1. RUN: Executando Comandos no Build

O comando **`RUN`** permite executar linhas de comando *dentro do container* **enquanto a Imagem está sendo gerada**.

Para nossa aplicação Node.js com pnpm, ficaria algo como:

```dockerfile
# Definindo a minha imagem base
FROM node:22.15 

# Instalando o gerenciador de pacotes pnpm globalmente, pois não é nativo do Node
RUN npm i -g pnpm 

# Depois, podemos rodar os comandos do projeto
RUN pnpm i
RUN pnpm run build
RUN pnpm run start
```

Mas epa! 😱 Se você tentar criar o seu container desta forma, provavelmente vai dar erro ‼️

Pensa so: o que o comando `pnpm i` tenta fazer? Ele tenta instalar as dependências do projeto que estão descritas nos arquivos `package.json` e `pnpm-lock.yaml`, certo?

Dai eu lhe pergunto: Estes arquivos existem no nosso container? 🤔\
Certamente não 🤨

O nosso container atualmente é apenas uma imagem limpa do node na versão 22.15

**Precisamos levar estes arquivos para dentro do container** para que, quando o comando de instalação for executado, ele encontre os arquivos dos quais ele depende.

### 2. COPY: Levando o Código para o Container

O comando COPY define qual arquivo da máquina local queremos copiar e para onde queremos levá-lo dentro do nosso container.

```dockerfile
FROM node:22.15

# Copiando os arquivos de dependência
COPY package.json pnpm-lock.yaml ./

RUN npm i -g pnpm
RUN pnpm i
RUN pnpm run build
```

Neste caso, estaríamos copiando os arquivos `package.json` e `pnpm-lock.yaml` para a raiz do nosso container. Perfeito né? 🌈

Mas aqui cabe um ⚠️ ALERTA ⚠️:

> ⚠️ Má Prática: Copiar arquivos para a raiz (./) do container, pois ela contém os recursos da Imagem Base (node). Isso polui o ambiente.

Por definição a raiz do nosso container contem os recursos da nossa imagem base: o node. Não é legal que os nossos arquivos `package.json` e `pnpm-lock.yaml` fiquem perdidos no meio dos demais arquivos de configuração do node. Então vamos copia-los para um local mais adequado, um **diretório de trabalho**.

### WORKDIR: Definindo o Local de Trabalho

O comando WORKDIR define o diretório de trabalho padrão para todas as instruções subsequentes (como COPY e RUN).

```dockerfile
FROM node:22.15

RUN npm i -g pnpm

# 1. Define /app como o diretório principal
WORKDIR /app 

# 2. Copia os arquivos DE/PARA /app
COPY package.json pnpm-lock.yaml ./ 

RUN pnpm i
RUN pnpm run build
```

Observe que como o diretório de trabalho foi definido antes do comando `COPY`, ao realizarmos a cópia os arquivos vão para /app 