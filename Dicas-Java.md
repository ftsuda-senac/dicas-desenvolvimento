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

1. Estrutura de um arquivo/classe Java - O nome do arquivo (sem extensão .java) e o nome da classe devem ser **exatamente iguais** (considerando letras maiúsculas e minúsculas)

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
    
1. Uso redundante do `if/else` e `switch` - Ambos tem o mesmo objetivo

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
1. Se o switch tiver que verificar strings que dependem de letras maiúsculas e minúsculas, fazer primeiro uma conversão para diminuir a quantidade de cases a serem tratados. É recomendável sanitizar o valor, para remover espaços em branco e acentuações também.

    Exemplo **INADEQUADO**
    ```java
    String valor = "LuA";
    
    switch (valor) {
        case "lua":
        case "LUA":
        case "Lua":
        case "lUa":
        case "luA":
        case "LUa":
        // case "LuA": // Notar que propositalmente a "LuA" igual ao valor acima foi comentado para não ser encontrado
        case "lUA":
            System.out.println("Digitou \"lua\"");
            break;
        default:
            System.out.println("Valor inválido");
    }
    ```
    
    Exemplo mais adequado
    ```java
    String valor = "Lua";
    valor = valor.toLowerCase(); // <-- Converte as letras para minúsculas
    
    switch (valor) {
        case "lua": // Aqui sabemos que o valor vai estar com todas letras minúsculas
            System.out.println("Digitou \"lua\"");
            break;
        default:
            System.out.println("Valor inválido");
    }
    ```
    
    Exemplo com sanitização (precisa importar `java.text.Normalizer`)
    ```java
    import java.text.Normalizer;

    public class ExemploSanitizacao {

        static String sanitizarString(String original) {
            // 1) Remove espaços em branco no começo e final da string
            original = original.trim(); 

            // 2) Converte as letras para minúsculas
            original = original.toLowerCase(); 

            // 3) Normaliza os caracteres acentuados e converte para versão sem acento
            original = Normalizer
                    .normalize(original, Normalizer.Form.NFD)
                    .replaceAll("[\\p{InCombiningDiacriticalMarks}]", ""); // Ou "[^\\p{ASCII}]" 
            return original;
        }

        public static void main(String[] args) {
            String valor = sanitizarString("  ÁgÜã   ");
            switch (valor) {
                case "agua": // Aqui sabemos que o valor vai estar com todas letras minúsculas, sem espaços em branco no começo e final e sem acentos
                    System.out.println("Digitou \"água\"");
                    break;
                default:
                    System.out.println("Valor inválido");
            }
        }

    }
    ```
1. Possibilidade de criação de variáveis _"globais"_ - Pode-se declarar uma variável dentro do bloco de código do `class` para que ele seja visível por todas as funções da classe (OBS: passa a ser má prática quando usar conceitos de orientação a objetos)

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
1. Analisar o código para identificar repetições de código e transformar (refatorar) para deixar na forma de funções.

    Exemplo **INADEQUADO**
    ```java
    import java.util.concurrent.TimeUnit;
    
    public class Jogo {
    
        public static void main(String[] args) {
            System.out.println("Mensagem");
            TimeUnit.SECONDS.sleep(1); // OU Thread.sleep(1000);
            
            System.out.println("Outra mensagem");
            TimeUnit.SECONDS.sleep(2); // OU Thread.sleep(2000);
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
            TimeUnit.SECONDS.sleep(tempoEspera); // OU Thread.sleep(1000 * tempoEspera);
        }
    
        public static void main(String[] args) {
            mostrarMensagemComEspera("Mensagem", 1);
            mostrarMensagemComEspera("Outra mensagem", 2);
        }
    }
    ```
    
    Outro exemplo **INADEQUADO**
    ```java
    public class Jogo {

        static void mostrarStatusFulanoECachorro(int hpFulano, int hpCachorro) {
            System.out.println("HP Fulano: " + hpFulano);
            System.out.println("HP Cachorro: " + hpCachorro);
        }
        
        static void mostrarStatusFulanoEGato(int hpFulano, int hpGato) {
            System.out.println("HP Fulano: " + hpFulano);
            System.out.println("HP Gato: " + hpGato);
        }
        
        static void mostrarStatusCiclanoECachorro(int hpCiclano, int hpCachorro) {
            System.out.println("HP Ciclano: " + hpCiclano);
            System.out.println("HP Cachorro: " + hpCachorro);
        }
        
        static void mostrarStatusCiclanoEGato(int hpCiclano, int hpGato) {
            System.out.println("HP Ciclano: " + hpCiclano);
            System.out.println("HP Gato: " + hpGato);
        }

        // Notar que as 4 funções acima fazem a mesma lógica: mostrar o HP de uma pessoa e o HP de um animal
        // Ideal é deixar mais genérico/abstrato

        public static void main(String[] args) {
            int hpFulano, hpCiclano = 500;
            int hpCachorro, hpGato = 100;
            mostrarStatusFulanoECachorro(hpFulano, hpCachorro);
            mostrarStatusFulanoEGato(hpFulano, hpGato);
            mostrarStatusCiclanoECachorro(hpCiclano, hpCachorro);
            mostrarStatusCiclanoEGato(hpCiclano, hpGato);
        }
    }
    ```
    
    Exemplo mais adequado
    ```java
    public class Jogo {

        // Notar que a função foi alterada para receber de qual pessoa e qual animal os valores de HP se referem
        // Dessa forma, reaproveita a mesma lógica independente de "Fulano/Ciclano" ou "cachorro/gato"
        static void mostrarStatusPessoaEAnimal(String pessoa, int hpPessoa, String animal, int hpAnimal) {
            System.out.println("HP " + pessoa + ": " + hpPessoa);
            System.out.println("HP " + animal + ": " + hpAnimal);
        }

        public static void main(String[] args) {
            int hpFulano, hpCiclano = 500;
            int hpCachorro, hpGato = 100;
            mostrarStatusPessoaEAnimal("Fulano", hpFulano, "Cachorro", hpCachorro);
            mostrarStatusPessoaEAnimal("Fulano", hpFulano, "Gato", hpGato);
            mostrarStatusPessoaEAnimal("Ciclano", hpCiclano, "Cachorro",  hpCachorro);
            mostrarStatusPessoaEAnimal("Ciclano", hpCiclano, "Gato", hpGato);
        }
    }
    ```
    
1. Evitar declarar muitas variáveis e atribuir valores para eles em uma mesma linha, pois dificulta leitura. Se possível colocar somente UMA declaração por linha
      
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
    
1. Uso de arrays (vetores e matrizes) para organizar as informações usadas no jogo SEM usar orientação a objetos.

    Fluxograma
    
    ```mermaid

    flowchart TD;
        A([Inicio]) --> B[Iniciar poderes jogador] --> C[Iniciar poderes inimigos] --> D[[Definir inimigo]] --> E{Escolher poder};
        E --> |poder0|F[[Ataque poder0]];
        E --> |poder1|G[[Ataque poder1]];
        E --> |poder2|H[[Ataque poder2]];
        E --> |poder3|I[[Ataque poder3]];
        F --> J{Inimigo escolhe ataque};
        G --> J;
        H --> J;
        I --> J;
        J --> |poder0|K[[Inimigo usa poder0]];
        J --> |poder1|L[[Inimigo usa poder1]];
        K --> M([Fim]);
        L --> M;
    ```

    <details>
        <summary>Exemplo inicial <strong>INADEQUADO V1</strong></summary>
    
    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Considerações importantes deste exemplo:
    - Contagem dos poderes e inimigos começa em 0 para facilitar entendimento
    - Jogador tem 4 poderes disponíveis (numerados de 0 a 3)
    - Existem 3 inimigos, com poderes diferentes (inimigos numerados de 0 a 2, poderes de 0 a 1)
    - O terceiro inimigo é o chefe.
    - Todos os poderes são fixos, então as variáveis correspondentes estão declaradas como globais
    - Cada variável representa uma combinação do jogador ou inimigo com os respectivos poderes
     */
    public class ExemploInicialInadequadoV1 {

        static int jogadorPoder0 = 100;
        static int jogadorPoder1 = 200;
        static int jogadorPoder2 = 300;
        static int jogadorPoder3 = 400;

        static int inimigo0Poder0 = 20;
        static int inimigo0Poder1 = 30;

        static int inimigo1Poder0 = 30;
        static int inimigo1Poder1 = 50;

        static int chefePoder0 = 100;
        static int chefePoder1 = 200;

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 0:
                    System.out.println("Você enfrentará inimigo0.");
                    break;
                case 1:
                    System.out.println("Você enfrentará inimigo1.");
                    break;
                case 2:
                    System.out.println("Você enfrentará chefe.");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.

            switch (poderEscolhido) {
                case 0:
                    System.out.println("Você causou " + jogadorPoder0 + " de dano usando poder 0.");
                    // Notar que o switch abaixo está se repetindo nos demais cases do poder do jogador
                    // Daqui podemos melhorar de duas maneiras:
                    // 1) Entender que, neste caso, o ataque do inimigo não depende do poder escolhido pelo jogador.
                    //    Dessa forma, podemos mover essa lógica para fora do switch do poder do jogador (ExemploInicialV2)
                    // 2) Converter em uma função (ExemploInicialV3)
                    // 3) Usar em conjunto os dois caminhos anteriores.
                    switch (qualInimigo) {
                        case 0:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 1:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 2:
                            if (poderInimigo == 0) {
                                System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                            }
                            break;
                    }
                    break;
                case 1:
                    System.out.println("Você causou " + jogadorPoder1 + " de dano usando poder 1.");
                    switch (qualInimigo) {
                        case 0:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 1:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 2:
                            if (poderInimigo == 0) {
                                System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                            }
                            break;
                    }
                    break;
                case 2:
                    System.out.println("Você causou " + jogadorPoder2 + " de dano usando poder 2.");
                    switch (qualInimigo) {
                        case 0:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 1:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 2:
                            if (poderInimigo == 0) {
                                System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                            }
                            break;
                    }
                    break;
                case 3:
                    System.out.println("Você causou " + jogadorPoder3 + " de dano usando poder 3.");
                    switch (qualInimigo) {
                        case 0:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 1:
                            if (poderInimigo == 0) {
                                System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                            }
                            break;
                        case 2:
                            if (poderInimigo == 0) {
                                System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                            } else if (poderInimigo == 1) {
                                System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                            }
                            break;
                    }
                    break;
            }
        }

    }
    ```
    </details>

    <details>
        <summary>Exemplo inicial V2</summary>

    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Notar que o código dentro do switch do ataque do jogador pode ser movido para fora
    e não precisa ser repetido dentro de cada case do ataque do jogador.
     */
    public class ExemploInicialV2 {

        static int jogadorPoder0 = 100;
        static int jogadorPoder1 = 200;
        static int jogadorPoder2 = 300;
        static int jogadorPoder3 = 400;

        static int inimigo0Poder0 = 20;
        static int inimigo0Poder1 = 30;

        static int inimigo1Poder0 = 30;
        static int inimigo1Poder1 = 50;

        static int chefePoder0 = 100;
        static int chefePoder1 = 200;

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 0:
                    System.out.println("Você enfrentará inimigo0.");
                    break;
                case 1:
                    System.out.println("Você enfrentará inimigo1.");
                    break;
                case 2:
                    System.out.println("Você enfrentará chefe.");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            switch (poderEscolhido) {
                case 0:
                    System.out.println("Você causou " + jogadorPoder0 + " de dano usando poder 0.");
                    break;
                case 1:
                    System.out.println("Você causou " + jogadorPoder1 + " de dano usando poder 1.");
                    break;
                case 2:
                    System.out.println("Você causou " + jogadorPoder2 + " de dano usando poder 2.");
                    break;
                case 3:
                    System.out.println("Você causou " + jogadorPoder3 + " de dano usando poder 3.");
                    break;
            }

            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.
            switch (qualInimigo) {
                case 0:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                    }
                    break;
                case 1:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                    }
                    break;
                case 2:
                    if (poderInimigo == 0) {
                        System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                    }
                    break;
            }
        }

    }
    ```
    </details>

    <details>
        <summary>Exemplo inicial V3</summary>

    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Conversão do switch do inimigo repetido em uma função
    */
    public class ExemploInicialV3 {

        static int jogadorPoder0 = 100;
        static int jogadorPoder1 = 200;
        static int jogadorPoder2 = 300;
        static int jogadorPoder3 = 400;

        static int inimigo0Poder0 = 20;
        static int inimigo0Poder1 = 30;

        static int inimigo1Poder0 = 40;
        static int inimigo1Poder1 = 50;

        static int chefePoder0 = 100;
        static int chefePoder1 = 200;

        static void mostrarAtaqueInimigo(int qualInimigo, int poderInimigo) {
            switch (qualInimigo) {
                case 0:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo0 causou " + inimigo0Poder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo0 causou " + inimigo0Poder1 + " de dano usando poder 1.");
                    }
                    break;
                case 1:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo1 causou " + inimigo1Poder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo1 causou " + inimigo1Poder1 + " de dano usando poder 1.");
                    }
                    break;
                case 2:
                    if (poderInimigo == 0) {
                        System.out.println("chefe causou " + chefePoder0 + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("chefe causou " + chefePoder1 + " de dano usando poder 1.");
                    }
                    break;
            }
        }

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 0:
                    System.out.println("Você enfrentará inimigo0.");
                    break;
                case 1:
                    System.out.println("Você enfrentará inimigo1.");
                    break;
                case 2:
                    System.out.println("Você enfrentará chefe.");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.

            switch (poderEscolhido) {
                case 0:
                    System.out.println("Você causou " + jogadorPoder0 + " de dano usando poder 0.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 1:
                    System.out.println("Você causou " + jogadorPoder1 + " de dano usando poder 1.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 2:
                    System.out.println("Você causou " + jogadorPoder2 + " de dano usando poder 2.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 3:
                    System.out.println("Você causou " + jogadorPoder3 + " de dano usando poder 3.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
            }
        }

    }
    ```
    </details>

    <details>
        <summary>Exemplo com vetor V1</summary>
    
    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Aqui será feita uma alteração para agrupar os poderes de cada personagem em um vetor
    Lembrar que nos vetores, as posições são contadas a partir do 0
    */
    public class ExemploVetorV1 {

        static int[] jogadorPoderes = new int[]{100, 200, 300, 400};

        static int[] inimigo0Poderes = new int[]{20, 30};

        static int[] inimigo1Poderes = new int[]{40, 50};

        static int[] chefePoderes = new int[]{100, 200};

        static void mostrarAtaqueInimigo(int qualInimigo, int poderInimigo) {
            switch (qualInimigo) {
                case 0:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo0 causou " + inimigo0Poderes[0] + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo0 causou " + inimigo0Poderes[1] + " de dano usando poder 1.");
                    }
                    break;
                case 1:
                    if (poderInimigo == 0) {
                        System.out.println("inimigo1 causou " + inimigo1Poderes[0] + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("inimigo1 causou " + inimigo1Poderes[1] + " de dano usando poder 1.");
                    }
                    break;
                case 2:
                    if (poderInimigo == 0) {
                        System.out.println("chefe causou " + chefePoderes[0] + " de dano usando poder 0.");
                    } else if (poderInimigo == 1) {
                        System.out.println("chefe causou " + chefePoderes[1] + " de dano usando poder 1.");
                    }
                    break;
            }
        }

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 0:
                    System.out.println("Você enfrentará inimigo0.");
                    break;
                case 1:
                    System.out.println("Você enfrentará inimigo1.");
                    break;
                case 2:
                    System.out.println("Você enfrentará chefe.");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.

            switch (poderEscolhido) {
                case 0:
                    System.out.println("Você causou " + jogadorPoderes[0] + " de dano usando poder 0.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 1:
                    System.out.println("Você causou " + jogadorPoderes[1] + " de dano usando poder 1.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 2:
                    System.out.println("Você causou " + jogadorPoderes[2] + " de dano usando poder 2.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
                case 3:
                    System.out.println("Você causou " + jogadorPoderes[3] + " de dano usando poder 3.");
                    mostrarAtaqueInimigo(qualInimigo, poderInimigo);
                    break;
            }
        }

    }
    ```
    </details>
    
    <details>
        <summary>Exemplo com vetor V2</summary>
    
    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Notar que no exemplo VetorV1, os valores de entrada e os gerados via random já correspondem aos valores das posições do vetor.
    Assim, dá para simplificar e usar diretamente estes valores para pegar os dados do vetor (tanto para jogador quanto para inimigos).
    */
    public class ExemploVetorV2 {

        static int[] jogadorPoderes = new int[]{100, 200, 300, 400};

        static int[] inimigo0Poderes = new int[]{20, 30};

        static int[] inimigo1Poderes = new int[]{40, 50};

        static int[] chefePoderes = new int[]{100, 200};

        static void mostrarAtaqueInimigo(int qualInimigo, int poderInimigo) {
            switch (qualInimigo) {
                case 0:
                    System.out.println("inimigo0 causou " + inimigo0Poderes[poderInimigo] + " de dano usando poder " + poderInimigo + ".");
                    break;
                case 1:
                    System.out.println("inimigo1 causou " + inimigo1Poderes[poderInimigo] + " de dano usando poder " + poderInimigo + ".");
                    break;
                case 2:
                    System.out.println("chefe causou " + chefePoderes[poderInimigo] + " de dano usando poder " + poderInimigo + ".");
                    break;
            }
        }

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 0:
                    System.out.println("Você enfrentará inimigo0.");
                    break;
                case 1:
                    System.out.println("Você enfrentará inimigo1.");
                    break;
                case 2:
                    System.out.println("Você enfrentará Chefe.");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            System.out.println("Você causou " + jogadorPoderes[poderEscolhido] + " de dano usando poder " + poderEscolhido + ".");

            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.
            mostrarAtaqueInimigo(qualInimigo, poderInimigo);
        }

    }
    ```
    </details>
    
    <details>
        <summary>Exemplo com matriz</summary>
    
    ```java
    import java.util.Random;
    import java.util.Scanner;

    /*
    Nesta última alteração, todas as informações dos inimigos serão mantidas em uma matriz, onde
    a primeira posição indica qual o inimigo e a segunda o poder dele.
    Notar que abstraimos que o chefe é o inimigo 2.
    */
    public class ExemploMatriz {

        static int[] jogadorPoderes = new int[]{100, 200, 300, 400};

        static int[][] inimigosPoderes = new int[][]{
            {20, 30}, // inimigo0
            {40, 50}, // inimigo1
            {100, 200} // chefe (no caso, o chefe passa a ser o inimigo2)
        };

        static void mostrarAtaqueInimigo(int qualInimigo, int poderInimigo) {
            switch (qualInimigo) {
                case 2:
                    System.out.println("chefe causou " + inimigosPoderes[qualInimigo][poderInimigo] + " de dano usando poder " + poderInimigo + ".");
                    break;
                default:
                    System.out.println("inimigo" + qualInimigo + " causou " + inimigosPoderes[qualInimigo][poderInimigo] + " de dano usando poder " + poderInimigo + ".");
                    break;
            }
        }

        public static void main(String[] args) {
            Scanner entrada = new Scanner(System.in);
            Random random = new Random();

            int qualInimigo = random.nextInt(3); // Random pode gerar 0, 1 ou 2 (chefe);
            switch (qualInimigo) {
                case 2:
                    System.out.println("Você enfrentará chefe.");
                    break;
                default:
                    System.out.println("Você enfrentará inimigo" + qualInimigo + ".");
                    break;
            }

            System.out.println("Escolha um poder de 0 a 3 para usar:");
            int poderEscolhido = entrada.nextInt(); // valores de 0 a 3 são válidos
            System.out.println("Você causou " + jogadorPoderes[poderEscolhido] + " de dano usando poder " + poderEscolhido + ".");

            int poderInimigo = random.nextInt(2); // Random pode gerar 0 ou 1.
            mostrarAtaqueInimigo(qualInimigo, poderInimigo);
        }

    }
    ```
    </details>

## Quotes

> "Programs must be written for people to read, and only incidentally for machines to execute."
> 
> &mdash; <cite>Unknown</cite> <a href="https://twitter.com/MakadiaHarsh/status/1512821254318870537" target="_blank" rel="nofollow noopener noreferrer">tweet</a>

> "Programming is not about typing, it's about thinking."
> 
>  &mdash; <cite>Rich Hickey</cite> <a href="https://twitter.com/CodeWisdom/status/1511684402039959555" target="_blank" rel="nofollow noopener noreferrer">tweet</a>
