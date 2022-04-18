# Dicas Java

## Links

* https://docs.oracle.com/javase/tutorial/
    * https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html (básico/inicial)
* https://www.javatpoint.com/java-tutorial
* https://www.tutorialspoint.com/java/index.htm

## Convenções para escrita do código

### Objetivos

* Padronização de práticas já estabelecidas por desenvolvedores mais experientes dentro da linguagem escolhida
* Facilidade em manter um código-fonte legível e organizado
* Facilita o entendimento do código por todos os membros da equipe (atuais e futuros)

### Estilos de sintaxe para palavras compostas

* Compor termos compostos removendo os espaços em branco.
* https://medium.com/better-programming/string-case-styles-camel-pascal-snake-and-kebab-case-981407998841
* Estilos:
    * `sintaxeCamelCase`
    * `SintaxePascalCase`
    * `sintaxe-kebab-case`
    * `sintaxe_snake_case`

### Convenções em Java

* Packages – nomes com todas as letras em minúsculas.
    * `br.com.empresa.projeto`
    * `br.com.equipex.nomecomposto`
* Classes – primeira letra maiúscula e demais minúsculas. Se for nome composto, o segundo nome começa com letra maiúscula e as demais são minúsculas (Pascal case).
    * `public class Classe`
    * `public class ClasseNomeComposto`
* Atributos constantes (final static) – todas as letras em maiúsculas. Se for nome composto, separar usando o caractere "`_`" (underline) (SNAKE_CASE).
    * `final static double PI = 3.1415;`
    * `final static int TAMANHO_MINIMO = 100;`
* Atributos/variáveis – todas as letras minúsculas. Se for nome composto, o segundo nome começa com letra maiúscula e as demais são minúsculas (camelCase). Usar nomes que remetam à utilidade da variável
    * `String nome;`
    * `int codigoProduto`
* Métodos – igual aos atributos/variáveis (camelCase)
    * `void calcular() { ... }`
    * `void efetuarPedido() { ... }`
    * `int obterValor() { ... }`
    * `String getNome() { ... }`
* Recomendação: Instalar extensão SonarLint para ajudar na verificação dos itens acima - https://www.sonarlint.org/
    * Para programadores iniciantes, **NÃO** usar esta extensão no inicio, pois a forma como os alertas são apresentados podem ser confundidos como erros e eventualmente travar o desenvolvimento de aplicações iniciais. Instalar somente após o domínio básico do Java e com o objetivo de melhorar a qualidade dos códigos.
* Referências:
    * http://www.oracle.com/technetwork/java/javase/documentation/codeconvtoc-136057.html
    * https://www.geeksforgeeks.org/java-naming-conventions/
    * https://google.github.io/styleguide/javaguide.html

## Dicas diversas

* Evitar criar classes dentro do pacote "default"
* Ao comparar String, colocar o literal como responsável pela chamada à comparação, para evitar NullPointerException.
* Não comparar Strings usando `==`. Em vez disso, deve-usar os métodos `equals()` ou `equalsIgnoreCase()`
```
String str = "Abc";
"Abc" == str; // resultado FALSE - Compara local de alocação em memória
"Abc".equals(str); // resultado TRUE
"abc".equals(str); // resultado FALSE - A maiúsculo diferente de minúsculo
"abc".equalsIgnoreCase(str); // resultado TRUE
```
* Clareza do código - comparar exemplos abaixo
```java
function boolean compararInadequado(int valor1, int valor2) {
    if (valor1 > valor2) {
        return true;
    } else {
        return false;
    }
}
function boolean compararAdequado(int valor1, int valor2) {
    return (valor1 > valor2);
}
```
* Não usar float ou double para representar valores com casas decimais bem definidas (ex: Preços) - Neste caso, usar `java.math.BigDecimal`
```java
BigDecimal valor1 = new BigDecimal("2.50");
BigDecimal valor2 = new BigDecimal("1.50");
BigDecimal soma = valor1.add(valor2); // Valor de soma deve ser 4.00
```
* Não confundir tipos de dados primitivos com as classes Wrapper (notar que o Java converte automaticamente os valores durante o runtime - conceito do autoboxing/unboxing)
    * Exemplos de classes Wrapper usuais:
        * `char` -> `Character`
        * `int` -> `Integer`
        * `long` -> `Long`
        * `float` -> `Float`
        * `double` -> `Double`
    * Classes Wrapper aceitam o `null` como valor
* Regras de precedência de operadores - Ver https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html
* Cuidado ao importar classes que possuem o mesmo nome, mas que vem de pacotes diferentes.
```
import java.util.Date;
import java.sql.Date;
```
* Para Java 8+, evitar usar `java.util.Date` ou `java.util.Calendar` para representar datas e horários. Usar classes do pacote `java.time.*`. Por este mesmo motivo, evitar usar o `joda.time`.
* Em qualquer código-fonte, evitar usar acentuação e caracteres especiais, tanto dentro do código quanto nos nomes dos arquivos/diretórios.
* Componentizar a aplicação
    * Evitar cópia de trecho de códigos iguais entre as diferentes classes do projeto
    * Nestes casos, manter o trecho em comum em uma função ou classe separada
 * Para facilitar entendimento, usar nomes de atributos/variáveis/métodos de acordo com o seu uso/ação.
     * Exemplos:
         * `int qtde = 100;`
         * `public int calcularTotal(int qtde, BigDecimal precoUnitario) { ... }`
     * Contra exemplos:
         * `int q = 100;`
         * `public int calc(int q, BigDecimal x) { ... }`

## Maven

* http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings
* https://medium.com/@yetanothersoftwareengineer/maven-lifecycle-phases-plugins-and-goals-25d8e33fa22

## Java moderno (8+)

Alguns links para ver:

* https://twitter.com/Abh1navv/status/1512047289388347395


## Quotes

> "Programs must be written for people to read, and only incidentally for machines to execute."
> 
> &mdash; <cite>Unknown</cite> [tweet](https://twitter.com/MakadiaHarsh/status/1512821254318870537)

> "Programming is not about typing, it's about thinking."
> 
>  &mdash; <cite>Rich Hickey</cite> [tweet](https://twitter.com/CodeWisdom/status/1511684402039959555)
