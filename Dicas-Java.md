# Dicas Java

## Links

* https://docs.oracle.com/javase/tutorial/
    * https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html (básico/inicial)
* https://www.javatpoint.com/java-tutorial
* https://www.tutorialspoint.com/java/index.htm

## Dicas diversas

* Ao comparar String, colocar o literal como responsável pela chamada à comparação, para evitar NullPointerException.
* Não comparar Strings usando `==`. Em vez disso, deve-usar os métodos `equals()` ou `equalsIgnoreCase()`
```
String str = "Abc";
"Abc" == str; // resultado FALSE - Compara local de alocação em memória
"Abc".equals(str); // resultado TRUE
"abc".equals(str); // resultado FALSE - A maiúsculo diferente de minúsculo
"abc".equalsIgnoreCase(str); // resultado TRUE
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

## Maven

* http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings
* https://medium.com/@yetanothersoftwareengineer/maven-lifecycle-phases-plugins-and-goals-25d8e33fa22
