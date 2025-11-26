# Containers e Imagens

> Isolamento, Portabilidade e Normalização

Quando estamos desenvolvendo uma aplicação de qualquer tecnologia, é comum que, ao trabalhar sem containers, a gente instale uma porção de recursos nas nossas próprias máquinas para o desenvolvimento local e depois a gente reinstale estes mesmos recursos no servidor onde a aplicação vai rodar.

Um grande ponto que devemos observar é que: as aplicações que rodam de modo local sem containers nas nossas máquinas estão profundamente sujeitas as interferências de todo o ecossistema das nossas máquinas. Por exemplo: se o windows começar um update, isto vai influenciar na sua aplicação? 🤔.

Como todo bom sênior diria: Depende! (e depende de muitos fatores!)

Mas o ponto se torna mais critico quando percebemos que nossas aplicações sem containers vão rodar em servidores que também podem passar por updates imprevistos que podem influenciar nossas aplicações. 😰

Observe ainda que: caso você deseje rodar a sua aplicação em um servidor extra, ainda seria necessário configurar manualmente todo o novo servidor e não teríamos certeza nenhuma que ao seguir os mesmos passos de configuração do servidor anterior, a aplicação rodaria do mesmo modo no novo servidor. 😞

*~ Nada garante que se funciona na minha máquina, vá funcionar na sua. Porque os ambientes não são normalizados.*

Agora, quando trabalhamos com **containers**, temos a solução para os problemas mencionados. Por meio dos containers, conseguimos isolar a execução das aplicações do ecossistema da máquina que ela esta rodando, conseguimos também normalizar a execução destas aplicações, garantindo que a forma como a aplicação está rodando na máquina dos Dev`s vai ser idêntica a forma como a aplicação vai rodar em qualquer servidor e ganhamos ainda o beneficio da portabilidade, pois todas as instruções de como a aplicação deve rodar ficam agrupadas em uma única "receita". Caso seja necessário criar um novo servidor, basta "ler a receita".

## Imagens

Todos estes benefícios dos containers vem por meio das **Imagens**

**📄 A imagem é a "receita" que determina tudo que a sua aplicação precisa conter;\
O contêiner é a imagem em execução; 🧁**

*💭 Pensamento: Conceito semelhante ao de classes e objetos: Classes determinam como os objetos são, Objetos são as instâncias das classes.*

*💭 Pensamento: As imagens são imutáveis (uma vez construídas, não mudam), enquanto o Container é a instância mutável (pode ter estado de execução, arquivos de log, etc.).*

## Container vs. Máquina Virtual (VM)

Veja que há uma certa semelhança entre o que está sendo definido como *container* e o que já conhecemos como *Máquina-Virtual (VM)*.

De fato, são ideias semelhantes, mas os containers são mais simples de serem configurados e mais isolados, **não sendo VMs propriamente ditas**, mas sim uma simples camada isolada para execução das aplicações.

**Responsabilidade do container: Rodar a sua aplicação**\
**Responsabilidade de uma VM: Ser uma máquina completa, com seu próprio SO, recursos, etc**

> Para NERDS 🤓: A diferença principal é que containers usam recursos do kernel (como Namespaces e cgroups) para isolamento, enquanto VMs usam um Hypervisor para virtualizar o hardware. Mencionar Namespaces e cgroups (mesmo que brevemente) eleva o nível técnico da explicação sobre isolamento.