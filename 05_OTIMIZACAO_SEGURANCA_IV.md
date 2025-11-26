# Otimização e Segurança IV: Redução de Privilégios e Análise de Vulnerabilidades

Nesta etapa, focamos em dois pilares cruciais para a segurança das nossas Imagens: garantir que o Contêiner não execute com privilégios desnecessários e automatizar a verificação de vulnerabilidades.

## 1. O Risco do Usuário root e o Comando USER 🚨

O padrão do Docker é que os comandos dentro do Contêiner sejam executados como usuário `root` (o superusuário).

### 🚨 Por que evitar o root?

Se um atacante conseguir explorar uma vulnerabilidade na sua aplicação, rodar o Contêiner como root concede a ele privilégios máximos sobre o Contêiner e, em cenários críticos (ou se o kernel não estiver bem isolado), potencialmente sobre o próprio Sistema Operacional do Host. ☠️

A regra de ouro é: **princípio do menor privilégio**. Um Contêiner só deve ter as permissões estritamente necessárias para executar sua aplicação.

#### 🚨🔐 Usando o Comando `USER`

O comando `USER` no `Dockerfile` define qual usuário será usado para executar as instruções subsequentes e, mais importante, qual usuário será usado para o comando final (`CMD` ou `ENTRYPOINT`).

Para criar e usar um usuário sem privilégios:

Crie um grupo e um usuário (se a Imagem base não tiver um usuário não-root por padrão).

Use o comando USER para alternar.

Exemplo de Aplicação Segura (Node.js)\
Se a Imagem base (como node:22.15-slim) já contém utilitários para adicionar usuários, você pode fazer:

```dockerfile
# ... Estágios de BUILD anteriores ...

# === Estágio 2: Produção (Final) ===
FROM node:22.15-slim

# 1. Cria um grupo e um usuário não-root (ex: "appuser")
# A flag -r é para usuário do sistema, -u define o UID
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
# ... COPY de artefatos ...

# 2. Define o usuário que rodará a aplicação (CMD)
USER appuser 

EXPOSE 3333
CMD ["node", "dist/main.js"]
```

Resultado: Sua aplicação (`dist/main.js`) será executada com as permissões do usuário appuser, limitando drasticamente o estrago que um invasor pode causar.

# 2. Análise de Vulnerabilidades com Trivy

Mesmo usando Imagens base oficiais e técnicas de otimização (como Multistage e DistroLess), ainda existem vulnerabilidades de software nos pacotes instalados.

`Trivy` é uma ferramenta bem legal, de código aberto, rápida e abrangente, projetada para analisar Imagens de Contêineres em busca de vulnerabilidades de segurança `(Common Vulnerabilities and Exposures - CVEs)`.

## 🔎 Panorama Geral do Trivy

* O que ele analisa? O Trivy verifica o SO da Imagem (pacotes como glibc, openssl, etc.) e as dependências de linguagens de programação (pacotes npm, pacotes Python, etc.).

* Por que usá-lo? Ele fornece um relatório detalhado sobre as vulnerabilidades encontradas, incluindo o nível de criticidade (LOW, MEDIUM, HIGH, CRITICAL) e a versão do pacote necessária para corrigir o problema.

* Integração em DevOps: É uma ferramenta ideal para ser integrada nas Pipelines de CI/CD (Continuous Integration/Continuous Delivery), permitindo que você bloqueie o push de uma Imagem para o Registro se ela contiver vulnerabilidades críticas.

## ⚙️ Instalação e Uso Rápido

O Trivy pode ser instalado de várias maneiras (via Homebrew, apt-get, ou mesmo rodando-o como um Contêiner).

### Instalação (Exemplo Linux/Debian)

```bash
# Baixa o binário do Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

```

### Uso (Analisando sua Imagem Local)

Para analisar a Imagem que acabamos de criar, use o comando `image`:

```bash
# Analisa a imagem local 'widget-server:v1'
trivy image widget-server:v1
```

O Trivy irá retornar uma tabela de vulnerabilidades encontradas na Imagem, permitindo que você tome decisões informadas sobre a segurança da sua aplicação antes de enviá-la para produção.
