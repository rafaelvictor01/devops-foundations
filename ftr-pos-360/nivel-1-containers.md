# Containers e Imagens

> Isolamento, Portabilidade, Normalização

Quando estamos trabalhando / desenvolvendo uma aplicação NodeJs (ou de qualquer outra tecnologia), é comum que, ao trabalhar sem containers, precisamos instalar o node na nossa própria máquina para o desenvolvimento local e depois instalar o node no servidor onde a aplicação vai rodar. O mesmo vale para as demais dependencies que o node precisaria para 
funcionar.

Um grande ponto que devemos observar é que: as aplicações que rodam de modo local sem containers nas nossas máquinas estão profundamente sujeitas as interferências de todo o ecossistema das nossas máquinas. Por exemplo: se o windows começar um update, isto pode ou não influenciar na sua aplicação. O mesmo vale para a aplicação que roda um servidor sem o uso de containers.

Observe ainda que: caso você deseje rodar a sua aplicação em um servidor extra, ainda seria necessário configurar manualmente todo o novo servidor e não teríamos certeza se ao seguir os mesmos passos de configuração do servidor anterior, a aplicação rodaria do mesmo modo no novo servidor.

*~ Nada garante que se funcionar na máquina, vá funcionar na sua. Porque os ambientes não são normalizados.*

Agora, quando trabalhamos com containers, temos a solução para os problemas mencionados. Por meio dos containers, conseguimos isolar a execução das aplicações do ecossistema da máquina que ela esta rodando, conseguimos também normalizar a execução destas aplicações, garantindo que a forma como a aplicação está rodando na máquina dos Dev`s vai ser idêntica a forma como a aplicação vai rodar em qualquer servidor e ganhamos ainda o beneficio da portabilidade, pois todas as instruções de como a aplicação deve rodar ficam agrupadas em uma única "receita". Caso seja necessário criar um novo servidor, basta "ler a receita".

Todos estes benefícios dos containers vem por meio das **Imagens**

**📄 A imagem é a "receita" que determina tudo que a sua aplicação precisa conter;\
O contêiner é a imagem em execução; 🧁**

*💭 Pensamento: Conceito semelhante ao de classes e objetos: Classes determinam como os objetos são, Objetos são as instâncias das classes.*

*💭 Pensamento: As imagens são imutáveis (uma vez construídas, não mudam), enquanto o Container é a instância mutável (pode ter estado de execução, arquivos de log, etc.).*

Veja que há uma semelhança entre o que está sendo definido como container e o que já conhecemos como Máquina-Virtual (VM). De fato, são ideias semelhantes, mas os containers são mais simples de serem configurados e mais isolados, não sendo VMs propriamente ditas, mas sim uma simples camada isolada para execução das aplicações.

**Responsabilidade do container: Rodar a sua aplicação**\
**Responsabilidade de uma VM: Ser uma máquina completa, com seu próprio SO, recursos, etc**

> Para NERDS 🤓: A diferença principal é que containers usam recursos do kernel (como Namespaces e cgroups) para isolamento, enquanto VMs usam um Hypervisor para virtualizar o hardware. Mencionar Namespaces e cgroups (mesmo que brevemente) eleva o nível técnico da explicação sobre isolamento.

# 🐳 Docker

Os desenvolvedores até podem criar contêineres sem o Docker trabalhando diretamente com recursos integrados ao Linux e outros sistemas operacionais, mas o Docker torna a conteinerização mais rápida e fácil. 😅

[O Docker é uma plataforma de código aberto que permite aos desenvolvedores construir, implementar, executar, atualizar e gerenciar containers.](https://www.ibm.com/br-pt/think/topics/docker). Existem outras ferramentas que cumprem o mesmo proposito, mas o docker é com certeza a mais popular delas.

Docker é a ferramenta de conteinerização mais utilizada, com 82,84% de participação de mercado. O Docker é tão popular hoje que "Docker" e "contêineres" são usados de forma intercambiável. No entanto, as primeiras tecnologias relacionadas a contêineres estavam disponíveis há anos — até mesmo décadas — antes do Docker ser lançado publicamente como código aberto em 2013.

## Dockerfile

Vimos então que os containers vão conter uma "receita" de como a aplicação deve estar sendo executada.\
Para escrever essa receita, vamos usar o `dockerfile`.

Um Dockerfile é um script que contém instruções para construir uma imagem Docker. Ele define a imagem base, configura variáveis de ambiente, roda comandos e configura o contêiner para uma aplicação ou serviço específico.

### Sintaxe do Dockerfile

O `Dockerfile` nada mais é do que um arquivo com este nome (sem extensão, com 'D' maiúsculo) que fica na raiz do seu projeto.

E para escreve-lo, sempre começamos com o comando `FROM`.
O comando `FROM` vai especificar para o docker o nosso ponto de partida.

Por exemplo: `FROM ubuntu:20.04`

Este comando está definindo que o seu container vai partir de uma imagem limpa do ubuntu na sua versão 20.04.

Sim! a imagem que você está criando agora para a sua aplicação vai se basear em uma outra imagem já pré-disponibilizada do ubuntu! Neste contexto, a imagem do ubuntu servirá para nós como **imagem base**.

Existe uma grande variedade de imagens-base na internet já prontas para uso. Veja em: https://hub.docker.com/. Além disso, se um dia você quiser disponibilizar a sua própria imagem para outros DEVs usarem, seja de modo publico ou privado, é por lá que você vai fazer isso!

Outro caminho (talvez até mais prático) do que usar o comando `FROM` diretamente com um sistema operacional, seria usar este comando apontando para uma imagem mais especifica para os seus propósitos.

Imagine que você precisa colocar uma aplicação NodeJs em um container.
Claro que você pode começar usando o ubuntu e instalar nele tudo que você precisa.
Mas na internet já existem imagens pré-prontas e otimizadas para você que deseja apenas rodar uma aplicação NodeJs.

Algumas opções:

1 - Você pode usar a imagem oficial do node: https://hub.docker.com/_/node \
2 - Você pode usar uma imagem do ubuntu já com o node: https://hub.docker.com/r/ubuntu/node \
3 - Ou, você pode pegar imagens customizadas por outros DEVs que tentaram criar um ambiente otimizado para execução do node: https://hub.docker.com/r/joxit/node

Uma vez definida a nossa imagem base, poderíamos prosseguir com a execução da nossa aplicação dentro do container.

Aqui cabe uma reflexão: 🤔 O que uma aplicação node precisa para ser executada? 🤔

Bom, são poucas coisas, na verdade.\
Num primeiro momento, precisamos apenas rodar comandos como `pnpm i` para instalar as dependências do projeto e `pnpm run build` para rodar o script de build da aplicação. Por fim, `pnpm run start` colocaria a aplicação para rodar e ficar disponível em alguma porta, correto?

🤔 Mas, como rodamos o nosso comando `pnpm i` dentro do container?\
🤔 Será que o container reconhecer o comando `pnpm`?

Para tratar destes assunto, usamos no dokerfile o comando `RUN`

o comando `RUN` vai permitir que executemos linhas de comando dentro do container enquanto ele está sendo criado. Então bastaria que o nosso `dockerfile` estivesse como: 

```dockerfile
FROM node:22.15

RUN pnpm i
RUN pnpm run build
```

Mas, de fato, é provável que o gerenciador de pacotes pnpm não exista no nosso container, porque o pnpm não é o gerenciador de pacotes nativo do node e, portanto, a imagem base que estamos usando não trouxe este recurso pré-instalado. Então podemos rodar: `RUN npm i -g pnpm` para instalar este recurso globalmente no nosso container.

```dockerfile
FROM node:22.15

RUN npm i -g pnpm
RUN pnpm i
RUN pnpm run build
```

Mas epa! 😱 Se você tentar criar o seu container desta forma, provavelmente vai dar erro, porque pense so: o que o comando `pnpm i` tenta fazer? Ele tenta instalar as dependências do projeto que estão descritas nos arquivos `package.json` e `pnpm-lock.yaml`. Dai eu lhe pergunto: Estes arquivos existem no nosso container? 🤔

Certamente não 🤨.\
O nosso container atualmente é apenas uma imagem limpa do node na versão 22.15
Precisamos levar estes arquivos para dentro do container para que, quando o comando de instalação for executado, ele encontre os arquivos dos quais ele depende.

Para isso, usamos o comando `COPY` do `Dockerfile`
com o comando `COPY`, definimos qual arquivo queremos copiar e para onde queremos leva-lo dentro do nosso container.

```dockerfile
FROM node:22.15

COPY package.json pnpm-lock.yaml ./

RUN npm i -g pnpm
RUN pnpm i
RUN pnpm run build
```

Neste caso, estaríamos copiando os arquivos `package.json` e `pnpm-lock.yaml` para a raiz do nosso container, ‼️**mas isto não é uma boa prática** ‼️. Por definição a raiz do nosso container contem os recursos da nossa imagem base: o node. Não é legal que os nossos arquivos `package.json` e `pnpm-lock.yaml` fiquem perdidos no meio dos demais arquivos de configuração do node. Então vamos copia-los para um local mais adequado, um diretório de trabalho.

Para isso, podemos usar o comando `WORKDIR`. Este comando define o diretório de trabalho para instruções subsequentes.

```dockerfile
FROM node:22.15

RUN npm i -g pnpm

WORKDIR /app
COPY package.json pnpm-lock.yaml ./

RUN pnpm i
RUN pnpm run build
```

Observe que como o diretório de trabalho foi definido antes do comando `COPY`, ao realizarmos a cópia os arquivos vão para /app 

Outros comandos uteis: 

`EXPOSE`: Informa ao Docker que o contêiner está trabalhando na porta exposta.

`CDM`: Atua de modo semelhante ao comando `RUN`, executando comandos dentro do container. Mas o comando `RUN` é executado enquanto a imagem está sendo gerada e a aplicação não está rodando. O comando `CDM` será aplicado após o container ser inicializado. Isso é util para quando queremos por exemplo: rodar a aplicação em modo de desenvolvimento após o container ser inicializado

Então, uma primeira versão do nosso dockerfile hipotético poderia ser:

```dockerfile
FROM node:22.15

RUN npm i -g pnpm

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN pnpm i

COPY . . # Cópia de todo o restante do projeto

EXPOSE 3333
CMD ["pnpm", "dev"]
```

Para testarmos o dockerfile anterior, podemos usar no terminal o comando: `docker build -t widget-server:v1 .` (Supondo que widget-server seja o nome da imagem e v1 seja a sua tag)

Este comando: 

- Inicia o processo de build usando o Dockerfile localizado no diretório atual (por causa do . no final).
- A flag -t significa tag, que permite dar nome à imagem. No caso:
    * Nome da imagem: widget-server
    * Tag da imagem: v1

Em seguida, veja se a imagem foi criada com o comando `docker image ls`

Se sim, podemos colocar a imagem para rodar em um container!
execute o comando: `docker run -p 3000:3333 -d widget-server:v1`

Este comando: 

- Está colocando para rodar a imagem que demos o nome de widget-server com a tag v1.
- Está falando que vamos expor o que o container mostra na porta 3333 dele na porta 3000 da nossa máquina local:
- Está executando o comando com a flag -d que não irá travar o nosso terminal

Para ver se funcionou, avalie se o docker criou algum processo com o seu container: `docker ps`\
para ver os logs do seu container: `docker logs <Id do container>`\
para parar a execução do container: `docker stop <Id do container>`\
para iniciar a execução do container: `docker start <Id do container>`

Legal!
Este `dockerfile` não está perfeito, mas já é bastante funcional!

Alguns itens que podemos nos atentar para melhorar a execução do docker:

1 - Criar o arquivo `.dockerignore` para que não seja levado para o container itens indesejados. Por exemplo: pasta `node_modules`, arquivos `env`, pasta `dist`, etc. Isso ajuda a deixar sua imagem e seu container mais leves

2 - Inclua o comando `RUN pnpm prune --prod` no seu dockerfile para descartar as dependências de desenvolvimento. Isso ajuda a deixar sua imagem e seu container mais leves.

Lembre-se sempre que: **quanto maior a sua imagem e mais recursos desnecessários ela carrega dentro dela, mais oportunidades de ataques os hackers tem**. ☠️

#### Multistage Build

'Multistage Build' é o nome que damos a uma abordagem para otimizar o processo de criação da imagem docker. Do mesmo modo como incluir um `.dockerignore` pode ajudar a reduzir o tamanho final das nossas imagens e dos nossos containers, aplicar corretamente a técnica de `Multistage Build` fará o mesmo.

Como o próprio nome sugere, pretendemos com a aplicação da técnica segmentar a execução do dockerfile em múltiplas etapas de build. Para isso, podemos repetir o comando `FROM` dentro do Dockerfile.

Antes do recurso multi-stage era permitido apenas uma instrução `FROM` que era carregada inicialmente e estabelecia qual imagem seria utilizada, afetando todos os comandos subsequentes.  Com o novo recurso podemos utilizar quantos comandos `FROM` precisarmos.

Cada `FROM` é um novo estágio que substitui o anterior, é como uma nova imagem, totalmente independente e isolada.

Vamos ver um exemplo:

```dockerfile
# === Estágio 1: Build ===

# Usamos 'AS builder' para dar um nome a este estágio
FROM node:22.15 AS builder 

RUN npm i -g pnpm 

WORKDIR /app
COPY package.json pnpm-lock.yaml ./ 

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm run build

# === Estágio 2: Produção (Final) ===

# Escolhemos uma imagem mais leve para o ambiente de execução final
FROM node:22.15-slim

WORKDIR /app

# Copia APENAS os artefatos necessários (código compilado e dependências de produção)

# --from=builder indica a origem: Estágio 1
COPY --from=builder /app/package.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist # Copia o resultado do build (artefatos)

RUN pnpm prune --prod

EXPOSE 3333

CMD ["node", "dist/main.js"]
```

- Estágio de Build: Onde todas as ferramentas pesadas (como compiladores, linter, dependências de desenvolvimento) são instaladas e o código é compilado (Ex: TypeScript para JS, React para bundle estático, etc.).

- Estágio de Produção (Runtime): Uma imagem muito mais leve (Ex: node:22.15-slim ou alpine) onde apenas os artefatos finais (o código compilado) são copiados do estágio de build.

Isso garante que ferramentas de build e dependências de desenvolvimento não entrem na imagem final de produção, que é o que você usa no servidor.

#### DistroLess

Observe que alguns dos nossos últimos esforços até o momento foram voltados para a tentativa de deixar as imagens cada vez menores e com menos recursos desnecessários. Isto é natural e intuitivo, pois quanto menor as nossas imagens menores são as nossas superfícies de ataques e mais leves ficam nossas aplicações.

A ideia de **DistroLess Containers** vem deste objetivo de criar containers cada vez menores, com menos pacotes desnecessários (visto que o container precisa apenas dos pacotes necessários para executar a aplicação).

Na prática, imagens DistroLess são uma classe especial de imagens Docker que se destacam por sua abordagem minimalista. 

Ao contrário das imagens tradicionais, que incluem um sistema operacional completo, as imagens DistroLess contêm apenas o necessário para executar uma aplicação. Isso significa que não há pacotes ou utilitários desnecessários, o que reduz significativamente a superfície de ataque do containers.

A própria Google fabrica imagens DistroLess. [@SEE](https://github.com/GoogleContainerTools/distroless)

