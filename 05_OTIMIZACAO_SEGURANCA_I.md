# Otimização e Segurança I: Boas Práticas

Um `Dockerfile` funcional não é necessariamente um `Dockerfile` otimizado. Estes passos melhoram o tamanho da Imagem e a sua segurança.

---

## 1. O Arquivo `.dockerignore`

* **Problema:** O comando `COPY . .` copia *todos* os arquivos do diretório atual, incluindo pastas pesadas ou sensíveis (`node_modules`, `.git`, `.env`, `dist`).

* **Solução:** Crie um arquivo chamado **`.dockerignore`** na raiz do projeto. Tudo o que estiver listado nele será **ignorado** durante o processo de build.

* **Exemplo:**
    ```dockerignore
    node_modules
    .env
    dist
    .git
    npm-debug.log
    ```

## 2. Removendo Dependências de Desenvolvimento

Muitos pacotes (ex: ESLint, Jest, TypeScript) são necessários apenas para o desenvolvimento e build, mas não para a execução em produção.

* **Comando:** Inclua o comando **`RUN pnpm prune --prod`** após a instalação das dependências. Isso descarta as dependências de desenvolvimento, deixando apenas as de produção.

* ⚠️ Atenção ⚠️: O comando acima é bem legal porque: mesmo se as suas dependências estiverem listadas como dev-dependencies, ao executar `pnpm i` as dev-dependencies são carregadas na pasta `node_modules`. O comando `RUN pnpm prune --prod` remove os resquícios de tudo que não for necessário para seu ambiente de produção.

---

## ⚠️ Regra de Ouro da Segurança

**Quanto menor a sua Imagem e menos recursos desnecessários ela carrega dentro dela, menor a sua superfície de ataque.** ☠️

O objetivo é sempre minimizar o tamanho e o número de pacotes instalados.