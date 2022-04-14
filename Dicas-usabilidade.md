# Dicas Usabilidade e projetos de interface

* Operações críticas como Delete devem sempre ter uma caixa de confirmação da operação
* Sempre mostrar alguma mensagem de feedback após operações que não envolvem troca de tela
* Usar tags semânticas corretas do HTML5 (SEO), principalmente em páginas públicas indexadas pelos buscadores
* URLs - Usar sintaxe kebak-case para nomear caminhos com nomes compostos (SEO)
* Para formulários - Criar uma página genérica contendo estilos para os input/select/textarea e considerando locais para mensagens de feedback, erros nos campos, identificação dos campos obrigatórios, textos explicativos complementares, indicação de campos obrigatórios.
* CSS - Se for usado algum framework ou biblioteca existente, evitar sobrescrever estilos padrão, sempre criar classes CSS personalizadas e com escopo reduzido (ex: alterar todos elementos `<div>` - Criar `<div class="escopo"`> e CSS para `.escopo { ... }`)
* Se possível, prever mecanismo de debounce em botões para evitar cliques repetidos, juntamente com alguma indicação de carregando (principalmente para operações mais lentas)
* Cuidado ao usar bibliotecas Javascript, pois muitas podem estar em desuso e os recursos são suportados pelo Javascript padrão (ex: momment.js para data/hora)
