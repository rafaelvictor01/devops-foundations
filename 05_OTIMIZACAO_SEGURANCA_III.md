# Otimização e Segurança III: DistroLess Containers

Observe que alguns dos nossos últimos esforços até o momento foram voltados para a tentativa de deixar as imagens cada vez menores e com menos recursos desnecessários.

## A Ideia DistroLess

A ideia de **DistroLess Containers** vem deste objetivo de criar containers cada vez menores, com menos pacotes desnecessários.

* **Definição:** Imagens **DistroLess** (do inglês, *sem distribuição*) são uma classe especial de imagens Docker que se destacam por sua abordagem minimalista.
* **Composição:** Ao contrário das imagens tradicionais (que incluem um sistema operacional completo, como o Ubuntu ou Debian), as imagens DistroLess contêm **apenas o necessário para executar a aplicação** (ex: o runtime do Node.js).
* **Vantagem:** Isso significa que **não há pacotes ou utilitários desnecessários**, o que reduz significativamente a **superfície de ataque** do container.
* **Exemplo:** A própria Google fabrica imagens DistroLess. [@SEE](https://github.com/GoogleContainerTools/distroless)
