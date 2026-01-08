# 🎵 Orquestração e Docker Compose

Show! 😎

Se você chegou até aqui, é provável que você já saiba como fazer a gestão de contêineres isolados e incorpora-los em um fluxo de `CI` automatizado.

Entretanto, dificilmente as nossas aplicações reais se resumem a um único container. 😞

É provável que frequentemente você precise ficar instanciando diversos contêineres na sua máquina apenas para fazer o desenvolvimento dos seus códigos... (ex: sua API Node.js + um Banco de Dados PostgreSQL).

Então vamos discutir agora sobre um conjunto de técnicas que visa resolver o problema de rodar o ambiente completo da nossa aplicação nas nossas máquinas de modo simplificado. O `Docker Compose`

## 1. Conhecendo Orquestração

Orquestração é o gerenciamento automatizado do ciclo de vida de aplicações em Contêineres, incluindo implantação, dimensionamento (scaling), redes e armazenamento.

* `Docker Compose` - Orquestração Local: Usamos o `Docker Compose` para orquestrar serviços em uma única máquina (seu ambiente de desenvolvimento, por exemplo). É simples e ideal para testar a aplicação completa.

* `Kubernetes, ECS` - Orquestração em Cluster: Usamos ferramentas mais robustas para gerenciar aplicações em múltiplas máquinas/servidores (produção, alta disponibilidade).

> 💡 Foco do Docker Compose: Gerenciar a Aplicação Multi-Contêiner em um ambiente de desenvolvimento ou testes simples. Ele mapeia como os serviços se conectam e se comunicam.

## 2. Configurando o Primeiro Docker Compose

O `Docker Compose` usa um único arquivo de configuração, geralmente chamado `docker-compose.yml` (ou `.yaml`), que define o estado desejado dos seus serviços.

### A. Estrutura Básica do `docker-compose.yml`

O arquivo YAML possui três chaves principais:

1. `version`: Define a versão do formato do arquivo Compose (geralmente a mais recente, como '3.8' ou superior).

2. `services`: O coração do arquivo. Aqui definimos cada Contêiner que compõe a nossa aplicação. Cada serviço será mapeado para um Contêiner.

3. `networks` e `volumes`: Seções opcionais para configurar redes e persistência de dados (abordaremos em detalhes mais adiante).

### B. Mapeando um Serviço Existente

Podemos definir o nosso widget-server (assumindo que ele tem uma Imagem já pronta) para ser rodado pelo Compose:

```yaml
version: '3.8'

services:
  # Nome do serviço (que também será o hostname dentro da rede)
  widget-server: 
    # Usa uma imagem existente (ex: a que fizemos push para o ECR)
    image: [710585551315.dkr.ecr.us-east-1.amazonaws.com/rocketseat/widget-server:v2](https://710585551315.dkr.ecr.us-east-1.amazonaws.com/rocketseat/widget-server:v2) 
    # Mapeamento de portas: <Porta no Host>:<Porta no Contêiner>
    ports: 
      - "3333:3333" 
    # Reinicia o contêiner se ele falhar
    restart: always 
```

### C. Mapeando um Serviço a ser Construído (Build)

Se o nosso serviço ainda precisa ser construído a partir do Dockerfile local (ideal para desenvolvimento), trocamos a chave image por build:

```yaml
version: '3.8'

services:
  widget-server:
    # Indica que o Docker deve construir a imagem a partir do diretório atual
    build: . 
    ports: 
      - "3333:3333"
    restart: always 
```

### D. Comandos Essenciais

Após criar o arquivo `docker-compose.yml` alguns comandos uteis são:

`docker compose up`
Cria e inicia todos os serviços definidos.

`docker compose up -d`
Cria e inicia todos os serviços em segundo plano (detached mode - sem travar o terminal).

`docker compose up --build -d`
Cria e builda e inicia todos os serviços em segundo plano (detached mode - sem travar o terminal).

`docker compose down`
Para e remove os Contêineres, Networks e Volumes (por padrão).

`docker compose ps`
Lista o status de todos os Contêineres do projeto Compose.
