# 🐳 Docker: A Plataforma de Conteinerização

Os desenvolvedores até podem criar contêineres sem o Docker trabalhando diretamente com recursos integrados ao Linux e outros sistemas operacionais, mas o Docker torna a conteinerização mais rápida e fácil. 😅

[O Docker é uma plataforma de código aberto que permite aos desenvolvedores construir, implementar, executar, atualizar e gerenciar containers.](https://www.ibm.com/br-pt/think/topics/docker).

> Docker é a ferramenta de conteinerização mais utilizada, com 82,84% de participação de mercado. Por isso, "Docker" e "contêineres" são usados muitas vezes de forma intercambiável.

---

## Dockerfile: A Receita do Container

Vimos que os containers vão conter uma "receita" de como a aplicação deve estar sendo executada. Okay! Então para escrever essa receita, vamos usar o **Dockerfile**.

Um `Dockerfile` é um script que contém instruções sequenciais para construir uma Imagem Docker. Nele você "ensinará" para o computador os passos que ele deve seguir para rodar o seu projeto. Vamos pegar por exemplo um projeto Node.js: você ensinará para o computador a localizar o seu arquivo `package.json`, ensinará ao computador a rodar por conta própria o `npm i`, ensinará ao computador como criar o build da aplicação e a como rodar ela, tudo isso por meio do `Dockerfile`.

### Sintaxe Básica do Dockerfile

O `Dockerfile` nada mais é do que um arquivo com este nome (sem extensão, com 'D' maiúsculo) que fica na raiz do seu projeto.

E para escreve-lo, sempre começamos com o comando `FROM`.
O comando `FROM` vai especificar para o docker o nosso ponto de partida.

Por exemplo:

```bash
FROM ubuntu:20.04
```

Este comando está definindo que o seu container vai partir de uma imagem pré-existente do ubuntu na sua versão 20.04.

Sim! a imagem que você está criando agora para a sua aplicação vai se basear em uma outra imagem já pré-disponibilizada do ubuntu! Neste contexto, a imagem do ubuntu servirá para nós como **imagem base**.

Existe uma grande variedade de imagens-base na internet já prontas para uso. Veja em: https://hub.docker.com/. Além disso, se um dia você quiser disponibilizar a sua própria imagem para outros DEVs usarem, seja de modo publico ou privado, é por lá que você vai fazer isso! (Vamos explorar isso daqui a pouco 👀).

Outro caminho (talvez até mais prático) do que usar o comando `FROM` diretamente com um sistema operacional, seria usar este comando apontando para uma imagem mais especifica para os seus propósitos.

Imagine que você precisa colocar uma aplicação NodeJs em um container.
Claro que você pode começar usando o ubuntu e instalar nele tudo que você precisa.
Mas na internet já existem imagens pré-prontas e otimizadas para você que deseja apenas rodar uma aplicação NodeJs.

Algumas opções:

1 - Você pode usar a imagem oficial do node: https://hub.docker.com/_/node \

2 - Você pode usar uma imagem do ubuntu já com o node: https://hub.docker.com/r/ubuntu/node \

3 - Ou, você pode pegar imagens customizadas por outros DEVs que tentaram criar um ambiente otimizado para execução do node: https://hub.docker.com/r/joxit/node

Uma vez definida a nossa imagem base, poderíamos prosseguir com a montagem da nossa receita.