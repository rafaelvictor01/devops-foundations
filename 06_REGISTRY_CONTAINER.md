## 📚 Registros de Contêineres (Container Registry)

Se a Imagem é a "receita" (o Dockerfile)\
E o Contêiner é a "aplicação em execução",\
o então o **Registros de Contêineres (Container Registry)** é livro completo de receitas!

Chamamos de **Container Registry** o depósito centralizado e seguro onde todas as Imagens são guardadas e de onde todos os ambientes (desenvolvimento, testes, produção) retiram suas cópias.

O principal propósito de um Registro é ser o elo que concretiza a Portabilidade e a Normalização. Ao invés de copiarmos manualmente os artefatos de build para o servidor, nós simplesmente enviamos a Imagem para o Registro, e o servidor baixa (pull) exatamente a mesma Imagem.

> Definição: Um Container Registry é um sistema de armazenamento e distribuição de imagens de contêineres que organiza as imagens em repositórios.

Os **Container Registry**s são como "GitHubs" para imagens.

‼️IMPORTANTE: Observe que enviar suas imagens para algum Container Registry é fundamental para fazer o deploy das duas aplicações em produção. A sua aplicação será publicada em produção a partir do momento em que uma nova imagem for enviada para o seu Container Registry.

### 🐳 Docker Hub: O Catálogo Global

O Docker Hub é, de longe, o Registro de Imagens de Contêineres mais conhecido. *Bem parecido com o GitHub mesmo.*

Inclusive, nós já usamos ele nos exemplos anteriores. Foi de lá que buscamos as *image bases* para montar nossos exemplos anteriores! O Docker Hub é o repositório padrão do Docker. Então, quando você usou o comando `FROM node:22.15` no seu `Dockerfile`, o Docker foi, de forma implícita, ao Docker Hub buscar essa Imagem Base oficial do Node.js.

### ☁️ AWS ECR: O Registro Empresarial na Nuvem

Embora o Docker Hub seja ótimo, muitas empresas começaram a aderir o `AWS Elastic Container Registry (ECR)` como repositório privado na nuvem para as imagens de suas aplicações. Ele funcionará de modo semelhante ao Docker Hub, mas algumas vantagens que podemos destacar são:

* Segurança: Controle de acesso refinado via IAM (Identity and Access Management) da AWS.
* Latência Reduzida: A imagem fica próxima aos seus servidores de produção na nuvem, acelerando o tempo de implantação (pull).
* Gerenciamento Simplificado: Não há necessidade de gerenciar a infraestrutura do Registro; a AWS cuida disso.

#### 🚀 Publicando sua Imagem (Push)

O processo de envio da sua imagem para um Registro é chamado de Push (Tanto para AWS quanto para o Docker Hub).

Lembre-se da nossa imagem que criamos de modo exemplificativo nos passos anteriores: `widget-server:v1`

Para que o comando docker push saiba para onde enviar a imagem, precisamos antes tagueá-la (marcar) com o endereço completo do Registro de destino.

> Processo semelhante a criar um commit. Primeiro criamos um commit e depois damos um Push, Correto?

##### Exemplo (Assumindo Docker Hub):

```bash

docker tag widget-server:v1 meu-usuario/widget-server:v1

# Com este comando, estamos falando que a nossa imagem local chamada 'widget-server:v1' deve ser enviada para o repositório do meu usuário no Docker Hub que contem o mesmo nome widget-server:v1

docker push meu-usuario/widget-server:v1

# Se o meu usuário no Docker Hub já possuir este repositório criado, maravilha! se não, ele será criado automaticamente após o push

```

##### Exemplo (Assumindo AWS ECR):

Se o endereço do seu Registro ECR for 123456789012.dkr.ecr.us-east-1.amazonaws.com, a tag seria:

```bash

docker tag widget-server:v1 123456789012.dkr.ecr.us-east-1.amazonaws.com/widget-server:v1

# Com este comando, estamos falando que a nossa imagem local chamada 'widget-server:v1' deve ser enviada para o repositório do meu usuário no ECR que contem o mesmo nome widget-server:v1

docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/widget-server:v1
```

~ 💭 Nota: Enquanto o Docker Hub cria o repositório automaticamente no primeiro push, a maioria dos Registros de Nuvem, como o ECR, exige que o repositório seja provisionado (criado) antes do primeiro push ser realizado.

#### ⬇️ Pull: Baixando em Produção

Depois do push, seu servidor de produção ou qualquer pessoa autorizada pode garantir a Normalização rodando exatamente a mesma imagem:

```bash

# O servidor de produção baixa a versão específica do repositório
docker pull meu-usuario/widget-server:v1

# E a coloca para rodar, garantindo que o ambiente é idêntico ao do Dev.
docker run -p 3000:3333 -d meu-usuario/widget-server:v1

```
