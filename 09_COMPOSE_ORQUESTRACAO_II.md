# 🐳 Docker Compose II: Adicionando Serviços (Banco de Dados)

O poder do **Docker Compose** reside na capacidade de definir todos os componentes da sua aplicação (os serviços) em um único arquivo `YAML`. O caso de uso mais clássico é a combinação de uma API/Frontend com um Banco de Dados.

## 1. Integrando o PostgreSQL como Serviço

Para adicionar o `PostgreSQL`, definimos um novo bloco dentro da seção `services`. O melhor de tudo é que podemos usar Imagens oficiais prontas do Docker Hub ou de Registros como o Bitnami. Vamos ver como isso ficaria:

```yaml
  db:
    image: bitnami/postgresql:latest
    container_name: postgresql
    ports:
      - $POSTGRES_PORT:$POSTGRES_PORT
    environment:
      - POSTGRES_USER=$POSTGRES_USER
      - POSTGRES_PASSWORD=$POSTGRES_PASSWORD
      - POSTGRES_DB=$POSTGRES_DB
    volumes:
      - db:/bitnami/postgresql # Onde o Bitnami persiste os dados
    restart: unless-stopped

```

Observe que a configuração deste serviço do `PostgreSQL` segue a mesma estrutura do serviço `widget-server` que vimos antes

* `image`: Imagem base a ser usada.
  * Não precisamos de build, pois a Imagem já existe no Docker Hub (Bitnami).
* `container_name`: Define um nome fixo para o contêiner. Essa é uma boa prática
  * Útil para referências externas, mas dentro da rede, usamos o nome do serviço (db).
* `ports`: Mapeamento de portas.
  * Opcional, mas útil para acessar o banco de dados diretamente da sua máquina (via psql ou GUI). Neste caso estamos definindo que a porta da nossa máquina que acessará o db é a mesma na qual o db está exposto dentro do container. Ambas as portas estão descritas no arquivo `.env`
* `environment`: Variáveis de ambiente.
  * CRUCIAL! A Imagem Bitnami usa estas variáveis para configurar o usuário, senha e banco de dados inicial na primeira execução. Estamos pegando os valores do arquivo `.env`
* `volumes`: Mecanismo de persistência.
  * Essencial para bancos de dados. Garantimos que os dados sobrevivam mesmo se o contêiner for removido. (Veremos a fundo na próxima seção).

## 2. Definindo o Volume para o Banco de Dados

Um banco de dados NUNCA deve rodar sem persistência. Se o contêiner for excluído, os dados são perdidos.

Para garantir que o PostgreSQL salve seus dados no seu disco local (Host), precisamos declarar e referenciar um `Volume`.

###  A. Declaração do Volume

O bloco `volumes` normalmente fica no final do arquivo e simplesmente declara que o nome dos volumes que precisamos:

Exemplo:

```yaml
volumes:
  db: # Volume nomeado para o serviço 'db'
```

### B. Uso no Serviço

APÓS DECLARAR O VOLUME DB, PODEMOS USA-LO.

No serviço db, mapeamos este volume nomeado para o diretório interno do contêiner onde o PostgreSQL armazena os dados (que, no caso da Imagem Bitnami, é /bitnami/postgresql).

```yaml
  volumes:
    - db:/bitnami/postgresql 
    # Sintaxe: <Nome do Volume Nomeado>:<Caminho Interno do Contêiner>
```

Resultado: Quando você rodar `docker compose up`, o Docker cria um Volume no seu sistema, e todos os dados do banco serão persistidos lá, garantindo que seu banco de dados esteja seguro entre ups e downs do Contêiner.
