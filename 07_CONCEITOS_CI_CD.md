# 🔄 Integração e Entrega Contínuas (CI/CD)

Entramos agora na fase de **automação** do nosso fluxo de trabalho. A ideia central aqui é eliminar passos manuais, garantir a qualidade do código desde o início e agilizar a entrega de novas features ou correções.

O fluxo de **CI/CD** é composto por duas fases que se complementam: Continuous Integration (CI): Integração Contínua e Continuous Delivery (CD): Entrega Contínua.

## 1. ⚙️ Continuous Integration (CI): Integração Contínua

**CI** é a prática de integrar as alterações de código dos desenvolvedores em um repositório central várias vezes ao dia. O foco principal da **CI** é criar uma esteira automatizada que garanta qualidade e a preparação da nossa aplicação para que posteriormente ela possa ser publicada sem dores de cabeça!

> 💡 Conceito: A **CI** garante que seu código funcione e esteja pronto para ir adiante. Ela prepara a "caixa" (a Imagem) que será entregue.

### O que a CI faz na prática?

Quando você envia um novo código (um `git push`), uma Pipeline de CI é automaticamente acionada e executa uma série de passos padronizados em sequência. Um exemplo poderia ser:

1. **Verificação de Lint / Formatação**: Garante que o código segue os padrões definidos pelo time.

2. **Execução de Testes Unitários e de Integração**: Verifica se novas mudanças quebraram funcionalidades existentes (o "não quebrou o que já funcionava").

3. **Build da Aplicação**: Gera os artefatos finais (ex: arquivos JavaScript compilados, bundles de frontend, etc.).

4. **Build da Imagem Docker**: Cria a Imagem Docker a partir do seu Dockerfile (a "receita").

5. **Análise de Segurança (Vulnerabilidades)**: Verifica a Imagem recém-construída em busca de CVEs (como fazemos com o Trivy).

6. **Push da Imagem**: Envia a Imagem final e tagueada para um Container Registry (ex: ECR ou Docker Hub), tornando-a disponível para uso.

## 2. 🚚 Continuous Delivery (CD): Entrega Contínua

A **Entrega Contínua (CD)** é a próxima fase, que só pode acontecer se a CI for bem-sucedida.

CD é a prática onde as alterações de código que passaram pela **CI** são automaticamente entregues a um repositório (como o Registry) e estão prontas para serem implantadas em um ambiente (como staging ou produção) a qualquer momento.

>💡 Diferença Chave: A CI prepara o pacote (a Imagem). O CD leva o pacote para um ambiente real (o deploy).

**CD (Entrega)**: O deploy é feito via um botão ou aprovação manual (humana). A aplicação está pronta para ir a produção, mas espera uma ação de quem é responsável.

**Continuous Deployment (Implantação)**: É uma etapa além, onde o deploy em produção ocorre automaticamente assim que a CI/CD termina com sucesso.
