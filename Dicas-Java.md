# Dicas Java

## Links

* <a href="https://docs.oracle.com/javase/tutorial/" target="_blank" rel="nofollow noopener noreferrer">https://docs.oracle.com/javase/tutorial/</a>
    * <a href="https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html" target="_blank" rel="nofollow noopener noreferrer">https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html</a> (básico/inicial)
* <a href="https://www.javatpoint.com/java-tutorial" target="_blank" rel="nofollow noopener noreferrer">https://www.javatpoint.com/java-tutorial</a>
* <a href="https://www.tutorialspoint.com/java/index.htm" target="_blank" rel="nofollow noopener noreferrer">https://www.tutorialspoint.com/java/index.htm</a>

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

* Entender uso do bloco try-catch-finally para desenvolvimento de código defensivo
    * try-with-resources (Java 7+)
    * Diferenças Checked exceptions X Unchecked exceptions (RuntimeException)
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
* Não confundir tipos de dados primitivos com as classes Wrapper (notar que o Java converte automaticamente os valores durante o runtime - conceito do unboxing/autoboxing)
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
* Para Java 8+, evitar usar `java.util.Date` ou `java.util.Calendar` para representar datas e horários. Usar classes do pacote `java.time.*`. Por este mesmo motivo, evitar usar o `joda.time`. Mais detalhes em https://stackoverflow.com/a/32443004
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

* <a href="http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings" target="_blank" rel="nofollow noopener noreferrer">http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings</a>
* <a href="https://medium.com/@yetanothersoftwareengineer/maven-lifecycle-phases-plugins-and-goals-25d8e33fa22" target="_blank" rel="nofollow noopener noreferrer">https://medium.com/@yetanothersoftwareengineer/maven-lifecycle-phases-plugins-and-goals-25d8e33fa22</a>

Estrutura mínima de um projeto Maven (gerado a partir do comando `tree /F /A` no PowerShell do Windows 11)

```
projeto-maven-java
|   pom.xml
|
+---src
|   +---main
|   |   \---java
|   |       \---<ESTRUTURA DE DIRETÓRIOS DO PACKAGE CRIADO>
|   |
|   \---test
|       \---java
|           \---<ESTRUTURA DE DIRETÓRIOS DO PACKAGE CRIADO>
\---target
    +---classes
    |   \---<ARQUIVOS .class COMPILADOS A PARTIR DO .java>
    |   <OUTROS ARQUIVOS RESULTANTES DA COMPILAÇÃO>
```

## Java moderno (8+)

Alguns links para ver:

* <a href="https://twitter.com/Abh1navv/status/1512047289388347395" target="_blank" rel="nofollow noopener noreferrer">https://twitter.com/Abh1navv/status/1512047289388347395</a>

## Exemplos úteis Java
* Calcular idade Java 8+: https://gist.github.com/ftsuda-senac/14279241af919ecb847b24f5fc8352be
* Singleton: https://gist.github.com/ftsuda-senac/e9f1e7b89dfd43d01d0df4bebc2e4072
* Classe utilitária (métodos static): TODO

## Alguns problemas pegos nas correções do PI1 - 2022-1

Considerando somente o uso de recursos básicos da linguagem e evitando uso de componentes externos.

* Estrutura de um arquivo/classe Java - O nome do arquivo (sem extensão .java) e o nome da classe devem ser **exatamente iguais** (considerando letras maiúsculas e minúsculas)

    Exemplo **ERRADO**:
    ```java
    // Arquivo classeerradaV3.java
    
    public class ClasseErrada { ... } // Notar que o C e o E são maiúsculas no códgo e minúsculas no nome do arquivo e tem o "v3" no nome do arquivo
    ```
    
    Exemplos corretos:
    ```java
    // Arquivo ClasseCorreta.java
    
    public class ClasseCorreta { ... }
    
    
    // Arquivo ClasseCorretaV3.java
    
    public class ClasseCorretaV3 { ... }
    ```
    
* Uso redundante do `if/else` e `switch` - Ambos tem o mesmo objetivo

    Exemplo **INADEQUADO**:
    ```java
    Scanner entrada = new Scanner(System.in);
    int opcao = entrada.nextInt();
    
    switch (opcao) {
        case 1:
            if (opcao == 1) { /* Faz algo */ }
            break;
        case 2:
            if (opcao == 2) { /* Faz outra coisa  */ }
            break;
    }
    ```
    
    Exemplos mais adequados:
    ```java
    Scanner entrada = new Scanner(System.in);
    int opcao = entrada.nextInt();
    
    switch (opcao) {
        case 1:
            // Faz algo
            break;
        case 2:
            // Faz outra coisa
            break;
    }
    ```
    
    **OU**
    
    ```java
    Scanner entrada = new Scanner(System.in);
    int opcao = entrada.nextInt();
    
    if (opcao == 1) { /* Faz algo */ }
    else if (opcao == 2) { /* Faz outra coisa */ }
    ```
 
* Possibilidade de criação de variáveis _"globais"_ - Pode-se declarar uma variável dentro do bloco de código do `class` para que ele seja visível por todas as funções da classe (OBS: passa a ser má prática quando usar conceitos de orientação a objetos)

   ```java
   // Arquivo Jogo.java
   
   public class Jogo {

       static int vidasJogador = 5;

       static void batalha() {
           vidasJogador--;
       }

       public static void main(String[] args) {
           batalha();
           System.out.println("Vidas: " + vidasJogador); // 4
       }
   }
   ```
* Analisar o código para identificar repetições de código e transformar (refatorar) para deixar na forma de funções.

    Exemplo **INADEQUADO**
    ```java
    import java.util.concurrent.TimeUnit;
    
    public class Jogo {
    
        public static void main(String[] args) {
            System.out.println("Mensagem");
            TimeUnit.SECONDS.sleep(1);
            
            System.out.println("Outra mensagem");
            TimeUnit.SECONDS.sleep(2);
        }
    }
    ```
    
    Exemplo mais adequado
    ```java
    import java.util.concurrent.TimeUnit;
    
    public class Jogo {
    
        // Notar que a "complexidade" de fazer o System.out.println e esperar fica
        // isolada dentro da função, deixando o código principal mais legível
        static mostrarMensagemComEspera(String msg, long tempoEspera) {
            System.out.println(msg);
            TimeUnit.SECONDS.sleep(tempoEspera);
        }
    
        public static void main(String[] args) {
            mostrarMensagemComEspera("Mensagem", 1);
            mostrarMensagemComEspera("Outra mensagem", 2);
        }
    }
    ```
* Evitar declarar muitas variáveis e atribuir valores para eles em uma mesma linha, pois dificulta leitura. Se possível colocar somente UMA declaração por linha

    Exemplo **INADEQUADO 1**
    ```java
    int x, y, z, cachorro, gato, i, j, xpto = 0;
    x = y = z = 2;
    cachorro = gato = 20;
    xpto = 100;
    ```
    

    Exemplo **INADEQUADO 2**
    ```java
    int x = 2, y = 2, z = 2, cachorro = 20, gato = 20, i = 0, j = 0, xpto = 100;
    ```
    
    Exemplo mais adequado
    ```java
    int x = 2;
    int y = 2;
    int z = 2;
    int i, j = 0; // Se fizerem parte de um mesmo "contexto" de informação, é aceitável dessa forma.
    int cachorro, gato = 20;
    int xpto = 100;
    ```

## Quotes

> "Programs must be written for people to read, and only incidentally for machines to execute."
> 
> &mdash; <cite>Unknown</cite> <a href="https://twitter.com/MakadiaHarsh/status/1512821254318870537" target="_blank" rel="nofollow noopener noreferrer">tweet</a>

> "Programming is not about typing, it's about thinking."
> 
>  &mdash; <cite>Rich Hickey</cite> <a href="https://twitter.com/CodeWisdom/status/1511684402039959555" target="_blank" rel="nofollow noopener noreferrer">tweet</a>
