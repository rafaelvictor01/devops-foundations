# Comandos Essenciais do Ciclo de Vida do Docker

Estes são os comandos que transformam o seu Dockerfile em uma Imagem e o colocam para rodar como um Contêiner.

## 🔨 Build e Imagens

| Comando | Função | Exemplo |
| :--- | :--- | :--- |
| **`docker build`** | Constrói a Imagem a partir do `Dockerfile`. | `docker build -t widget-server:v1 .` |
| **`-t` (tag)** | Atribui um nome (`widget-server`) e uma tag (`v1`) à Imagem. | |
| **`docker image ls`** | Lista todas as imagens construídas localmente. | `docker image ls` |

---

## 🚀 Contêineres em Execução

| Comando | Função | Exemplo |
| :--- | :--- | :--- |
| **`docker run`** | Coloca a Imagem para rodar como um Contêiner. | `docker run -p 3000:3333 -d widget-server:v1` |
| **`-p` (port)** | Mapeamento de portas. Expõe a porta **3333** do Contêiner na porta **3000** da Máquina Local (`3000:3333`). | |
| **`-d` (detach)** | Executa o Contêiner em *background*, liberando o terminal. | |
| **`docker ps`** | Lista todos os contêineres **ativos** (em execução). | `docker ps` |
| **`docker logs`** | Exibe os logs de saída do contêiner. | `docker logs <ID do container>` |
| **`docker stop`** | Para a execução de um contêiner ativo. | `docker stop <ID do container>` |
| **`docker start`** | Inicia um contêiner que foi parado. | `docker start <ID do container>` |