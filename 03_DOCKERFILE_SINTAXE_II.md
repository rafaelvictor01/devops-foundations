# Dockerfile: Finalizando a Receita (Sintaxe II)

O nosso `Dockerfile` não está muito polido, mas já está funcional apenas com os comandos anteriores!

Mas... Para refinar a Imagem e permitir que ela funcione como um Contêiner evitando uma porção de problemas ainda precisamos de mais alguns comandos, vamos passar rápido por alguns deles:

### 1. EXPOSE: Abertura da Porta

* **`EXPOSE <porta>`**: Informa ao Docker qual porta o contêiner **pretende usar**. É mais uma documentação do que uma abertura de porta em si. A abertura real é feita no comando `docker run`.

### 2. CMD: O Comando de Inicialização

* **`CMD`** (Command): É o comando principal que será executado **após o contêiner ser inicializado**. Ele não é executado durante o build da Imagem (diferente do `RUN`). É útil para rodar a aplicação lá no fim do processo (ex: `pnpm start`).

* **Sintaxe Recomendada:** Use o formato de array (executável e argumentos) para evitar problemas de sinalização de processo: `CMD ["executável", "arg1", "arg2"]`.

## Primeira Versão Verdadeiramente Funcional do Dockerfile

Assumindo que sua aplicação seja uma API Node.js que rode na porta `3333`:

```dockerfile
FROM node:22.15

RUN npm i -g pnpm

WORKDIR /app

COPY package.json pnpm-lock.yaml ./

RUN pnpm i

# Cópia de todo o restante do projeto (código-fonte)
COPY . .

EXPOSE 3333

RUN pnpm build
CMD ["node", "app/build/server.js"]
```