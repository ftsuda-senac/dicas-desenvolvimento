# GraalVM Native Image com Spring Boot

Este documento reúne conceitos, configurações e exemplos práticos sobre a compilação de projetos Spring Boot com GraalVM Native Image, com ênfase nas implementações que frequentemente causam conflitos nesse processo:

- **Visão geral do GraalVM Native Image**: como o compilador analisa o código e quais são seus limites;
- **Configuração de build**: Maven e Gradle para geração do binário nativo;
- **Processamento AOT**: como o Spring Boot prepara o contexto para compilação antecipada;
- **Reflexão e proxies dinâmicos**: os principais vetores de conflito e como registrá-los corretamente;
- **JPA/Hibernate**: restrições de inicialização e geração de bytecode em tempo de execução;
- **Jackson e serialização**: mapeamento dinâmico de tipos e problemas comuns;
- **Spring AOP e CGLIB**: limitações com subclasses e proxies JDK;
- **Recursos e arquivos de configuração**: acesso a recursos no classpath em imagens nativas;
- **Bibliotecas de terceiros**: compatibilidade e estratégias de contorno;
- **Diagnóstico e depuração**: ferramentas para identificar e corrigir falhas na compilação.

Os exemplos seguem Spring Boot 3.x, GraalVM 22+ e Jakarta EE.

---

## 1. Como o GraalVM Native Image funciona

### 1.1. Análise estática de alcançabilidade

O compilador GraalVM Native Image realiza uma análise estática do grafo de chamadas a partir do ponto de entrada da aplicação (`main`). Somente o código **alcançável estaticamente** é incluído no binário final. Qualquer uso de:

- **Reflexão** (`Class.forName`, `Method.invoke`, `Field.get`);
- **Proxies dinâmicos** (`java.lang.reflect.Proxy`, CGLIB);
- **Carregamento dinâmico de classes** (`ClassLoader.loadClass`);
- **Serialização Java** (`ObjectInputStream`, `ObjectOutputStream`);
- **JNI** (`System.loadLibrary`, métodos nativos);
- **Acesso a recursos** (`ClassLoader.getResourceAsStream`);

...precisa ser **declarado explicitamente** em arquivos de metadados ou via hints de configuração, pois o compilador não consegue inferir esses usos por análise estática.

### 1.2. Closed-world assumption

O Native Image opera sob a premissa de "mundo fechado" (*closed-world assumption*): todo o código que será executado já é conhecido no momento da compilação. Isso é fundamentalmente incompatível com o modelo tradicional do Spring, que carrega e configura beans dinamicamente em tempo de execução.

O Spring Boot 3.x resolve esse conflito por meio do **processamento AOT** (Ahead-of-Time), descrito na seção 3.

### 1.3. Diferença entre execução na JVM e execução nativa

| Característica            | JVM (JIT)                    | Native Image (AOT)              |
|---------------------------|------------------------------|---------------------------------|
| Inicialização             | Lenta (segundos)             | Rápida (milissegundos)          |
| Uso de memória            | Alto                         | Baixo                           |
| Reflexão                  | Livre em tempo de execução   | Requer registro explícito       |
| Proxies dinâmicos         | Gerados em tempo de execução | Precisam ser declarados ou substituídos |
| Geração de bytecode       | Permitida (ASM, CGLIB, etc.) | Não permitida em tempo de execução |
| Carregamento de classes   | Dinâmico                     | Apenas classes incluídas no build |
| Perfil de desempenho      | Melhora com o tempo (JIT)    | Constante desde o início        |

---

## 2. Configuração do projeto

### 2.1. Dependências Maven

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Plugin responsável pela compilação nativa -->
        <plugin>
            <groupId>org.graalvm.buildtools</groupId>
            <artifactId>native-maven-plugin</artifactId>
        </plugin>

        <!-- Spring Boot Maven Plugin com suporte a AOT -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!-- Exclui Lombok do binário final (apenas compile-time) -->
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 2.2. Perfis Maven para build nativo e JVM

```xml
<profiles>
    <!-- Perfil padrão: execução na JVM -->
    <profile>
        <id>jvm</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>

    <!-- Perfil para geração do Native Image -->
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>build-native</id>
                            <goals>
                                <goal>compile-no-fork</goal>
                            </goals>
                            <phase>package</phase>
                        </execution>
                        <execution>
                            <id>test-native</id>
                            <goals>
                                <goal>test</goal>
                            </goals>
                            <phase>test</phase>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

### 2.3. Comandos de build

```bash
# Processar fontes AOT (gera código em target/spring-aot/main)
./mvnw spring-boot:process-aot

# Compilar o Native Image (requer GraalVM instalado)
./mvnw -Pnative native:compile

# Compilar Native Image usando Docker (sem precisar do GraalVM local)
./mvnw -Pnative spring-boot:build-image

# Executar testes no modo nativo
./mvnw -Pnative test
```

### 2.4. Configuração Gradle equivalente

```groovy
plugins {
    id 'org.springframework.boot' version '3.4.0'
    id 'io.spring.dependency-management' version '1.1.6'
    id 'org.graalvm.buildtools.native' version '0.10.3'
    id 'java'
}

graalvmNative {
    binaries {
        main {
            // Opções extras para o compilador nativo
            buildArgs.add('--initialize-at-build-time=org.slf4j.LoggerFactory')
            buildArgs.add('-H:+ReportExceptionStackTraces')
        }
    }
}
```

---

## 3. Processamento AOT no Spring Boot 3.x

### 3.1. O que o AOT faz

O processamento AOT do Spring Boot analisa o contexto de aplicação e gera:

- **Código Java** com definições de beans sem uso de reflexão;
- **Arquivos de metadados** (`reflect-config.json`, `proxy-config.json`, `resource-config.json`) para o GraalVM;
- **Fábricas de beans** que substituem o carregamento dinâmico do `ApplicationContext`.

```
target/spring-aot/main/
├── sources/          ← código Java gerado (BeanDefinition, proxies, etc.)
├── resources/
│   └── META-INF/
│       └── native-image/
│           ├── reflect-config.json
│           ├── proxy-config.json
│           ├── resource-config.json
│           └── serialization-config.json
└── classes/          ← classes compiladas do código gerado
```

### 3.2. Como o contexto AOT é iniciado

```java
// Spring Boot detecta automaticamente o modo AOT via variável de sistema
// -Dspring.aot.enabled=true (ativado automaticamente no Native Image)

// Classe gerada pelo AOT — não editar manualmente
public class MinhaAplicacaoApplicationAotConfiguration
        implements BeanDefinitionRegistrar {

    @Override
    public void register(BeanDefinitionRegistry registry,
                         ResourceLoader resourceLoader) {
        // registros de beans sem reflexão em tempo de execução
    }
}
```

### 3.3. Beans condicionais e AOT

Beans com `@ConditionalOnProperty`, `@ConditionalOnClass` ou similares são avaliados **em tempo de build** pelo processador AOT. Isso significa que a presença ou ausência de um bean no Native Image é determinada no momento da compilação, não da execução.

```yaml
# application.properties lido durante o build AOT
# Condições baseadas em propriedades são fixadas no binário nativo
feature.pagamento.enabled=true
```

**Conflito comum**: usar `@ConditionalOnProperty` para ativar ou desativar funcionalidades em tempo de execução não funciona como esperado em imagens nativas. O bean estará presente ou ausente definitivamente conforme o valor da propriedade no momento do build.

---

## 4. Reflexão: o principal vetor de conflito

### 4.1. Por que a reflexão causa problemas

O GraalVM não inclui automaticamente classes acessadas apenas por reflexão. O código abaixo funciona perfeitamente na JVM, mas falha silenciosamente ou lança exceção em um Native Image sem configuração adicional:

```java
// Padrão problemático: carregamento dinâmico de classe por nome
String nomeClasse = config.getProperty("estrategia.classe");
Class<?> clazz = Class.forName(nomeClasse);           // falha no Native Image
Object instancia = clazz.getDeclaredConstructor().newInstance();
```

### 4.2. Registrando tipos para reflexão com @RegisterReflectionForBinding

A anotação `@RegisterReflectionForBinding` do Spring instrui o processador AOT a incluir os tipos informados nos metadados de reflexão:

```java
package br.com.exemplo.config;

import org.springframework.aot.hint.annotation.RegisterReflectionForBinding;
import org.springframework.context.annotation.Configuration;

import br.com.exemplo.dto.PedidoDTO;
import br.com.exemplo.dto.ClienteDTO;

@Configuration
@RegisterReflectionForBinding({ PedidoDTO.class, ClienteDTO.class })
public class ReflexaoConfig {
    // Registra PedidoDTO e ClienteDTO para reflexão completa (campos, métodos, construtores)
}
```

### 4.3. Registrando reflexão via RuntimeHintsRegistrar

Para cenários mais complexos, implementar `RuntimeHintsRegistrar` oferece controle granular:

```java
package br.com.exemplo.config;

import org.springframework.aot.hint.MemberCategory;
import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportRuntimeHints;

import br.com.exemplo.dominio.Produto;
import br.com.exemplo.dominio.Categoria;

@Configuration
@ImportRuntimeHints(NativeHintsConfig.ReflexaoHints.class)
public class NativeHintsConfig {

    static class ReflexaoHints implements RuntimeHintsRegistrar {

        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            // Reflexão em campos e métodos públicos
            hints.reflection()
                .registerType(Produto.class,
                    MemberCategory.INVOKE_PUBLIC_METHODS,
                    MemberCategory.PUBLIC_FIELDS)
                .registerType(Categoria.class,
                    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
                    MemberCategory.INVOKE_PUBLIC_METHODS);

            // Recurso estático no classpath
            hints.resources().registerPattern("templates/*.html");
            hints.resources().registerPattern("i18n/messages*.properties");

            // Proxy JDK para interfaces de repositório
            hints.proxies().registerJdkProxy(
                br.com.exemplo.repositorio.ProdutoRepository.class);
        }
    }
}
```

### 4.4. Arquivo manual reflect-config.json

Quando bibliotecas de terceiros não fornecem metadados GraalVM, é possível criar o arquivo manualmente em `src/main/resources/META-INF/native-image/<group-id>/<artifact-id>/reflect-config.json`:

```json
[
  {
    "name": "br.com.exemplo.dominio.Produto",
    "allDeclaredConstructors": true,
    "allPublicConstructors": true,
    "allDeclaredMethods": true,
    "allPublicMethods": true,
    "allDeclaredFields": true,
    "allPublicFields": true
  },
  {
    "name": "br.com.exemplo.dominio.Categoria",
    "allDeclaredConstructors": true,
    "allPublicMethods": true
  }
]
```

---

## 5. Proxies dinâmicos

### 5.1. Proxies JDK (interfaces)

O Spring usa extensivamente proxies JDK para implementar AOP, transações e segurança sobre interfaces. Em imagens nativas, cada combinação de interfaces usada como proxy precisa ser declarada:

```java
// Conflito: o Spring cria um proxy JDK dinamicamente em tempo de execução
@Service
public class PedidoService implements IPedidoService, Auditable {
    // O proxy JDK para {IPedidoService, Auditable} precisa ser registrado
}
```

Registro via `RuntimeHints`:

```java
hints.proxies().registerJdkProxy(
    IPedidoService.class,
    Auditable.class,
    org.springframework.aop.SpringProxy.class,
    org.springframework.aop.framework.Advised.class,
    org.springframework.core.DecoratingProxy.class
);
```

O Spring Boot AOT registra automaticamente proxies de beans gerenciados pelo contexto. O problema ocorre principalmente em proxies criados manualmente ou em bibliotecas de terceiros.

### 5.2. Proxies CGLIB (subclasses)

O CGLIB gera subclasses em tempo de execução estendendo a classe alvo. Esse mecanismo **não é suportado** em Native Image.

```java
// Conflito clássico: classe concreta anotada com @Transactional sem interface
@Service
public class EstoqueService {          // sem interface!

    @Transactional
    public void atualizar(Long id, int quantidade) {
        // O Spring precisa de um proxy CGLIB aqui
        // isso funciona na JVM, mas requer cuidado no Native Image
    }
}
```

O Spring Boot 3.x resolve esse conflito com o modo `proxyTargetClass=false` e geração de subclasses em tempo de build via AOT. Contudo, **classes `final` nunca podem ser subclassificadas**:

```java
// NUNCA faça isso em beans Spring usados com AOP/transações no Native Image
@Service
public final class EstoqueService {   // final impede qualquer proxy
    @Transactional
    public void atualizar(...) { }
}
```

**Solução recomendada**: sempre extrair uma interface para serviços que usam `@Transactional`, `@Cacheable`, `@Async` ou qualquer aspecto AOP.

---

## 6. JPA e Hibernate

### 6.1. Geração de bytecode em tempo de execução

O Hibernate gera proxies e classes de enhancement em tempo de execução por padrão. O Spring Boot 3.x configura automaticamente o Hibernate para usar um modo compatível com Native Image, mas algumas configurações manuais podem reverter esse comportamento.

```yaml
# application.yml — configuração automática do Spring Boot 3 para Native Image
spring:
  jpa:
    properties:
      hibernate:
        # Evita geração de bytecode em tempo de execução
        bytecode:
          provider: none
        # Usa enhance estático (aplicado durante o build)
        enhance:
          enable-lazy-initialization: true
          enable-dirty-tracking: true
          enable-association-management: true
```

### 6.2. Lazy loading e proxies de entidades

O carregamento lazy de associações JPA depende de proxies gerados pelo Hibernate. Em Native Image, o Hibernate usa **enhance estático** (instrumentação de bytecode no momento do build) para substituir os proxies dinâmicos:

```java
package br.com.exemplo.dominio;

import jakarta.persistence.*;
import java.util.List;

@Entity
@Table(name = "pedidos")
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Lazy loading funciona via enhance estático — sem proxy dinâmico
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;

    // Coleções lazy também funcionam com enhance estático
    @OneToMany(mappedBy = "pedido", fetch = FetchType.LAZY)
    private List<ItemPedido> itens;
}
```

**Conflito**: entidades sem enhance estático configurado terão carregamento lazy desativado silenciosamente ou lançarão exceção ao tentar acessar associações não inicializadas.

### 6.3. DDL e validação de schema

A geração de DDL em tempo de execução (`spring.jpa.hibernate.ddl-auto=create`) envolve inspeção de metadados que pode falhar em Native Image:

```yaml
# Conflito potencial no Native Image
spring:
  jpa:
    hibernate:
      ddl-auto: create       # problemático em produção nativa

# Configuração recomendada para Native Image em produção
spring:
  jpa:
    hibernate:
      ddl-auto: validate     # apenas valida, não gera DDL
  sql:
    init:
      mode: never
```

Use **Flyway** ou **Liquibase** para gerenciamento de schema em ambientes de produção com Native Image:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
```

### 6.4. Tipos customizados e conversores JPA

Conversores JPA e tipos customizados do Hibernate precisam de registro explícito para reflexão:

```java
package br.com.exemplo.config;

import br.com.exemplo.dominio.conversor.StatusPedidoConverter;
import br.com.exemplo.dominio.tipo.TipoMoedaCustom;
import org.springframework.aot.hint.MemberCategory;
import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;

public class HibernateHints implements RuntimeHintsRegistrar {

    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.reflection()
            .registerType(StatusPedidoConverter.class,
                MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
                MemberCategory.INVOKE_PUBLIC_METHODS)
            .registerType(TipoMoedaCustom.class,
                MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
                MemberCategory.INVOKE_PUBLIC_METHODS);
    }
}
```

### 6.5. Herança de entidades

As três estratégias de herança do JPA têm comportamentos distintos em Native Image:

**`SINGLE_TABLE`** — a mais segura para Native Image: todas as subclasses compartilham a mesma tabela e não há necessidade de joins ou queries polimórficas que exijam resolução dinâmica de tipo.

**`JOINED`** e **`TABLE_PER_CLASS`** — o Hibernate precisa resolver o tipo concreto de cada linha em tempo de execução. As subclasses precisam estar registradas para reflexão:

```java
package br.com.exemplo.dominio;

import jakarta.persistence.*;

@Entity
@Table(name = "pagamentos")
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Pagamento {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}

@Entity
@Table(name = "pagamentos_cartao")
public class PagamentoCartao extends Pagamento {
    private String bandeira;
    private int parcelas;
}

@Entity
@Table(name = "pagamentos_pix")
public class PagamentoPix extends Pagamento {
    private String chavePix;
}
```

Registro necessário para hierarquias com `JOINED` ou `TABLE_PER_CLASS`:

```java
public class HibernateHerancaHints implements RuntimeHintsRegistrar {

    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        // Todas as subclasses concretas precisam de reflexão completa
        Stream.of(Pagamento.class, PagamentoCartao.class, PagamentoPix.class)
            .forEach(tipo -> hints.reflection().registerType(tipo,
                MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
                MemberCategory.INVOKE_PUBLIC_METHODS,
                MemberCategory.PUBLIC_FIELDS,
                MemberCategory.DECLARED_FIELDS));
    }
}
```

**`@DiscriminatorValue`** com estratégia `SINGLE_TABLE` também pode falhar se o Hibernate tentar instanciar subclasses por reflexão para resolver o discriminador em alguns provedores. O registro preventivo das subclasses resolve o problema.

### 6.6. Cache de segundo nível (L2 Cache)

O cache de segundo nível do Hibernate com provedores como **Ehcache** ou **Caffeine via JCache** requer atenção especial, pois envolve serialização de entidades fora do contexto do Jackson.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jcache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          use_query_cache: true
          region.factory_class: org.hibernate.cache.jcache.JCacheCacheRegionFactory
        javax.cache.provider: com.github.benmanes.caffeine.jcache.spi.CaffeineCachingProvider
```

**Conflito**: o Hibernate serializa entidades cacheadas usando reflexão para acessar campos. Registrar as entidades anotadas com `@Cache`:

```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Produto {
    // campos...
}

// Na configuração de hints:
hints.reflection().registerType(Produto.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
    MemberCategory.DECLARED_FIELDS,        // necessário para serialização do cache
    MemberCategory.INVOKE_PUBLIC_METHODS);
hints.serialization().registerType(Produto.class);  // se usar serialização Java no cache
```

**Recomendação**: preferir Caffeine como cache L2 com `CacheConcurrencyStrategy.READ_WRITE`. Evitar `NONSTRICT_READ_WRITE` com Native Image, pois utiliza caminhos de código menos testados para atualização assíncrona.

### 6.7. Criteria API e metamodelo estático

A Criteria API do JPA usa o metamodelo estático gerado em tempo de compilação (`Produto_`, `Pedido_`, etc.). Esse metamodelo é compatível com Native Image, mas as classes geradas precisam ser detectadas pelo processador AOT:

```java
// Uso correto com metamodelo estático — compatível com Native Image
public List<Produto> buscarPorPreco(BigDecimal precoMinimo) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Produto> cq = cb.createQuery(Produto.class);
    Root<Produto> root = cq.from(Produto.class);

    // Produto_.preco é a classe de metamodelo gerada pelo processador de anotações
    cq.where(cb.greaterThanOrEqualTo(root.get(Produto_.preco), precoMinimo));
    return entityManager.createQuery(cq).getResultList();
}
```

**Conflito com metamodelo dinâmico**: acessar atributos por `String` em vez do metamodelo estático pode falhar se o Hibernate tentar resolver o atributo via reflexão:

```java
// Evitar: resolução dinâmica de atributo por String
root.get("preco")   // pode falhar em Native Image sem hints

// Preferir: referência ao metamodelo estático
root.get(Produto_.preco)   // seguro — referência direta ao campo gerado
```

Configuração do processador de anotações do metamodelo no Maven:

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <scope>provided</scope>
</dependency>
```

**Spring Data Specifications** — ao usar `Specification<T>` do Spring Data JPA, preferir referências ao metamodelo estático pelos mesmos motivos:

```java
public static Specification<Produto> comPrecoAcimaDe(BigDecimal valor) {
    return (root, query, cb) ->
        cb.greaterThanOrEqualTo(root.get(Produto_.preco), valor);
}
```

### 6.8. Projeções e resultados customizados

O Spring Data JPA oferece três formas de projeção, com compatibilidades diferentes em Native Image:

**Projeções baseadas em interface** — compatíveis, pois o Spring Data gera os proxies em tempo de build via AOT:

```java
// Seguro: proxy gerado pelo AOT do Spring Data
public interface ProdutoResumo {
    String getNome();
    BigDecimal getPreco();
}

List<ProdutoResumo> findAllProjectedBy();
```

**Projeções baseadas em classe (DTO constructor expression)** — requerem que o construtor esteja registrado para reflexão:

```java
// O Hibernate instancia ProdutoDTO via reflexão
public record ProdutoDTO(String nome, BigDecimal preco) {}

@Query("SELECT new br.com.exemplo.dto.ProdutoDTO(p.nome, p.preco) FROM Produto p")
List<ProdutoDTO> buscarResumidos();

// Hint necessário:
hints.reflection().registerType(ProdutoDTO.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS);
```

**Projeções com `@SqlResultSetMapping`** — o mapeamento de resultado nativo depende de reflexão para instanciar as classes mapeadas:

```java
@SqlResultSetMapping(
    name = "ProdutoComEstoque",
    classes = @ConstructorResult(
        targetClass = ProdutoEstoqueDTO.class,
        columns = {
            @ColumnResult(name = "nome", type = String.class),
            @ColumnResult(name = "quantidade", type = Integer.class)
        }
    )
)
@Entity
public class Produto { /* ... */ }
```

Hints necessários para `@SqlResultSetMapping`:

```java
hints.reflection().registerType(ProdutoEstoqueDTO.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
    MemberCategory.INVOKE_PUBLIC_METHODS);
```

### 6.9. Callbacks de entidades e listeners

Callbacks JPA (`@PrePersist`, `@PostLoad`, `@PreUpdate`, etc.) definidos tanto na própria entidade quanto em classes externas via `@EntityListeners` são invocados por reflexão:

```java
// Callback na própria entidade — AOT geralmente detecta
@Entity
public class Pedido {

    @PrePersist
    void prePersist() {
        this.criadoEm = LocalDateTime.now();
    }

    @PostLoad
    void postLoad() {
        // lógica pós-carregamento
    }
}

// Listener externo — requer hint explícito
@EntityListeners(AuditoriaListener.class)
@Entity
public class Produto { /* ... */ }

public class AuditoriaListener {

    @PrePersist
    @PreUpdate
    public void aoSalvar(Object entidade) {
        // lógica de auditoria
    }
}
```

Hint para listeners externos:

```java
hints.reflection()
    .registerType(AuditoriaListener.class,
        MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
        MemberCategory.INVOKE_PUBLIC_METHODS,
        MemberCategory.DECLARED_METHODS);   // inclui métodos anotados com @Pre/@Post
```

**Conflito frequente**: listeners que injetam `ApplicationContext` ou usam `@Autowired`. Em Native Image, listeners JPA são instanciados pelo Hibernate, não pelo Spring. A solução é usar o padrão `ApplicationContextAware` com um bean estático:

```java
@Component
public class SpringContextHolder implements ApplicationContextAware {

    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        context = ctx;
    }

    public static <T> T getBean(Class<T> tipo) {
        return context.getBean(tipo);
    }
}

public class AuditoriaListener {

    @PrePersist
    public void aoSalvar(Object entidade) {
        // Obtém o bean via holder estático em vez de @Autowired
        AuditoriaService service = SpringContextHolder.getBean(AuditoriaService.class);
        service.registrar(entidade);
    }
}
```

### 6.10. IDs compostos e tipos embutidos

**`@EmbeddedId`** e **`@IdClass`** — as classes de chave composta precisam de reflexão para instanciação e comparação:

```java
// Chave composta como @Embeddable
@Embeddable
public class PedidoItemId implements Serializable {
    private Long pedidoId;
    private Long produtoId;
    // equals e hashCode obrigatórios
}

@Entity
public class PedidoItem {
    @EmbeddedId
    private PedidoItemId id;
    private int quantidade;
}
```

Hints para chaves compostas:

```java
hints.reflection().registerType(PedidoItemId.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
    MemberCategory.INVOKE_PUBLIC_METHODS,
    MemberCategory.DECLARED_FIELDS);
hints.serialization().registerType(PedidoItemId.class);  // Serializable é obrigatório
```

**`@Embeddable` em campos comuns** — tipos embutidos usados como atributos de entidades:

```java
@Embeddable
public class Endereco {
    private String logradouro;
    private String cidade;
    private String cep;
}

@Entity
public class Cliente {
    @Embedded
    private Endereco endereco;
}
```

O Spring Boot AOT geralmente detecta `@Embeddable` referenciados em entidades gerenciadas. Contudo, se o tipo embutido for compartilhado entre módulos ou adicionado via biblioteca, o hint manual é necessário:

```java
hints.reflection().registerType(Endereco.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
    MemberCategory.DECLARED_FIELDS);
```

### 6.11. Repositórios com fragmentos customizados

O Spring Data JPA permite compor repositórios com fragmentos de implementação customizada. O mecanismo de descoberta de fragmentos usa reflexão:

```java
// Interface do fragmento
public interface ProdutoRepositoryCustom {
    List<Produto> buscarComFiltros(FiltroProduto filtro);
}

// Implementação do fragmento — o Spring Data descobre pelo nome da convenção
public class ProdutoRepositoryImpl implements ProdutoRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<Produto> buscarComFiltros(FiltroProduto filtro) {
        // implementação com Criteria API ou JPQL dinâmico
    }
}

// Repositório final compondo os fragmentos
public interface ProdutoRepository
        extends JpaRepository<Produto, Long>, ProdutoRepositoryCustom {
}
```

**Conflito**: a descoberta automática do fragmento (`ProdutoRepositoryImpl`) depende de convenção de nomenclatura resolvida via reflexão. O Spring Boot AOT 3.x detecta esse padrão para beans no contexto Spring, mas falha se a implementação não seguir a convenção ou estiver em um jar separado.

**Solução**: registrar explicitamente a implementação do fragmento:

```java
hints.reflection().registerType(ProdutoRepositoryImpl.class,
    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
    MemberCategory.INVOKE_PUBLIC_METHODS);
```

### 6.12. Auditoria com Spring Data JPA

A auditoria automática via `@EnableJpaAuditing` usa `AuditorAware` e listeners internos que dependem de reflexão:

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(
            SecurityContextHolder.getContext()
                .getAuthentication()
                .getName()
        );
    }
}

@Entity
@EntityListeners(AuditingEntityListener.class)
public class Produto {

    @CreatedDate
    private LocalDateTime criadoEm;

    @LastModifiedDate
    private LocalDateTime atualizadoEm;

    @CreatedBy
    private String criadoPor;

    @LastModifiedBy
    private String atualizadoPor;
}
```

O `AuditingEntityListener` do Spring Data é registrado automaticamente pelo processador AOT. Contudo, o `AuditorAware` customizado precisa ser um bean Spring gerenciado para ser detectado corretamente — não instanciá-lo manualmente.

**Conflito com `@Version` (controle de concorrência otimista)**: o Hibernate acessa o campo de versão via reflexão para comparar e incrementar. Certificar que o campo está acessível:

```java
@Entity
public class Produto {

    @Version
    private Long versao;   // o Hibernate usa reflexão para ler e gravar este campo
}

// Se a entidade não estiver registrada com DECLARED_FIELDS, o versionamento pode falhar
hints.reflection().registerType(Produto.class,
    MemberCategory.DECLARED_FIELDS,
    MemberCategory.INVOKE_PUBLIC_METHODS);
```

---

## 7. Jackson e serialização/desserialização

### 7.1. Mapeamento polimórfico com @JsonTypeInfo

O Jackson usa reflexão para instanciar subtipos em mapeamentos polimórficos. Todos os tipos concretos precisam ser registrados:

```java
package br.com.exemplo.dto;

import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

// Conflito: Jackson instancia subtipos via reflexão em tempo de execução
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, property = "tipo")
@JsonSubTypes({
    @JsonSubTypes.Type(value = PagamentoCartaoDTO.class, name = "cartao"),
    @JsonSubTypes.Type(value = PagamentoBoletoDTO.class, name = "boleto"),
    @JsonSubTypes.Type(value = PagamentoPixDTO.class,    name = "pix")
})
public abstract class PagamentoDTO { }
```

Registro via hints:

```java
@Configuration
@RegisterReflectionForBinding({
    PagamentoDTO.class,
    PagamentoCartaoDTO.class,
    PagamentoBoletoDTO.class,
    PagamentoPixDTO.class
})
public class JacksonNativeConfig { }
```

### 7.2. Mixins e módulos Jackson

Módulos Jackson registrados dinamicamente (via `ObjectMapper.registerModule`) precisam que seus tipos sejam declarados:

```java
package br.com.exemplo.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        // JavaTimeModule tem suporte nativo via spring-boot-starter
        // mas módulos customizados precisam de hints adicionais
        mapper.registerModule(new JavaTimeModule());
        return mapper;
    }
}
```

### 7.3. Desserialização de tipos genéricos

```java
// Conflito: TypeReference usa reflexão para capturar o tipo genérico em tempo de execução
List<ProdutoDTO> produtos = objectMapper.readValue(
    json,
    new TypeReference<List<ProdutoDTO>>() {}   // pode falhar em Native Image
);

// Solução: usar JavaType explicitamente
JavaType tipo = objectMapper.getTypeFactory()
    .constructCollectionType(List.class, ProdutoDTO.class);
List<ProdutoDTO> produtos = objectMapper.readValue(json, tipo);
```

---

## 8. Spring AOP e aspectos

### 8.1. Aspectos com @Aspect e @Around

O Spring AOP baseado em CGLIB funciona em Native Image quando as classes alvo têm enhance estático. Contudo, **pointcuts dinâmicos** que dependem de inspeção de anotações ou nomes de métodos em tempo de execução podem falhar:

```java
package br.com.exemplo.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class AuditoriaAspecto {

    // Pointcut baseado em anotação — funciona bem com Native Image
    @Around("@annotation(br.com.exemplo.anotacao.Auditavel)")
    public Object auditar(ProceedingJoinPoint joinPoint) throws Throwable {
        // ...
        return joinPoint.proceed();
    }

    // Pointcut baseado em nome de pacote — funciona bem
    @Around("execution(* br.com.exemplo.servico.*.*(..))")
    public Object monitorar(ProceedingJoinPoint joinPoint) throws Throwable {
        // ...
        return joinPoint.proceed();
    }
}
```

### 8.2. Programmatic AOP com ProxyFactory

Proxies criados programaticamente via `ProxyFactory` não são detectados pelo processador AOT e precisam de registro manual:

```java
// Conflito: proxy criado em tempo de execução fora do contexto Spring
ProxyFactory factory = new ProxyFactory(meuServico);
factory.addAdvice(new MeuInterceptor());
IMeuServico proxy = (IMeuServico) factory.getProxy();   // falha no Native Image

// Solução: usar beans Spring normais com @Transactional/@Aspect
// e deixar o AOT gerar os proxies em tempo de build
```

### 8.3. @Async e proxies assíncronos

```java
// A interface é necessária para que o proxy JDK seja gerado em tempo de build
public interface NotificacaoService {
    void enviarAsync(String mensagem);
}

@Service
public class NotificacaoServiceImpl implements NotificacaoService {

    @Async
    @Override
    public void enviarAsync(String mensagem) {
        // executado em thread separada
    }
}

// Configuração necessária
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

---

## 9. Spring Security

### 9.1. Filtros e AuthenticationProvider dinâmicos

O Spring Security registra filtros e providers em tempo de inicialização por meio de reflexão. O processador AOT do Spring Boot 3 lida com a maior parte dos casos padrão, mas implementações customizadas precisam de atenção:

```java
package br.com.exemplo.seguranca;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Component;

// Classe concreta sem interface extra — o AOT precisa detectá-la como bean
@Component
public class JwtAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) {
        // lógica de autenticação
        return null;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return true;
    }
}
```

### 9.2. SpEL em @PreAuthorize

Expressões SpEL usadas em `@PreAuthorize` e `@PostAuthorize` são avaliadas em tempo de execução e dependem de reflexão para acessar métodos e propriedades:

```java
// Conflito potencial: SpEL acessa métodos por nome via reflexão
@PreAuthorize("@permissaoService.podeEditar(authentication, #id)")
public ProdutoDTO atualizar(@PathVariable Long id, @RequestBody ProdutoDTO dto) {
    // ...
}
```

Para que isso funcione, o tipo do objeto (`permissaoService`) e os métodos invocados via SpEL precisam estar registrados para reflexão. O Spring Boot AOT faz esse registro automaticamente para beans do contexto Spring, mas métodos invocados apenas por SpEL podem ser removidos pelo compilador.

**Solução**: anotar os métodos invocados por SpEL com `@RegisterReflectionForBinding` ou adicionar hints manuais.

---

## 10. Spring Cache

### 10.1. Cache com Caffeine

O Caffeine é totalmente compatível com Native Image:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=500,expireAfterWrite=10m
```

### 10.2. Cache com Redis

O Spring Data Redis tem suporte a Native Image via metadados incluídos na biblioteca desde a versão 3.x:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**Conflito comum**: usar `GenericJackson2JsonRedisSerializer` com tipos polimórficos. O serializador precisa do tipo completo no JSON e usa reflexão para instanciar as classes corretas:

```java
// Conflito: GenericJackson2JsonRedisSerializer usa reflexão para tipos concretos
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(
            new GenericJackson2JsonRedisSerializer()   // requer hints de reflexão para todos os tipos cacheados
        ));
}

// Solução mais segura: serializar apenas tipos conhecidos
@Bean
public RedisCacheConfiguration cacheConfiguration(ObjectMapper objectMapper) {
    Jackson2JsonRedisSerializer<Object> serializer =
        new Jackson2JsonRedisSerializer<>(objectMapper, Object.class);
    return RedisCacheConfiguration.defaultCacheConfig()
        .serializeValuesWith(
            RedisSerializationContext.SerializationPair.fromSerializer(serializer));
}
```

---

## 11. Carregamento de recursos

### 11.1. Recursos no classpath

O acesso a arquivos no classpath via `ClassPathResource`, `ResourceLoader` ou `ClassLoader.getResourceAsStream` funciona em Native Image somente para recursos declarados nos metadados:

```java
// Conflito: recurso não declarado nos metadados será "invisível" no Native Image
Resource template = new ClassPathResource("templates/email-boas-vindas.html");
String conteudo = template.getContentAsString(StandardCharsets.UTF_8);
```

Declaração via `RuntimeHints`:

```java
hints.resources()
    .registerPattern("templates/*.html")
    .registerPattern("i18n/messages*.properties")
    .registerPattern("config/regras-negocio.json")
    .registerPattern("META-INF/services/*");
```

### 11.2. Arquivo resource-config.json manual

```json
{
  "resources": {
    "includes": [
      { "pattern": "\\Qtemplates/email-boas-vindas.html\\E" },
      { "pattern": "i18n/messages.*\\.properties" },
      { "pattern": "config/regras-negocio\\.json" }
    ]
  },
  "bundles": [
    { "name": "i18n.messages" }
  ]
}
```

### 11.3. ResourceBundle e internacionalização

Bundles de mensagens acessados via `MessageSource` precisam ser registrados:

```java
hints.resources().registerResourceBundle("i18n.messages");
hints.resources().registerResourceBundle("i18n.validacao");
```

---

## 12. Serialização Java

### 12.1. ObjectInputStream e ObjectOutputStream

A serialização Java nativa (`Serializable`) requer que todas as classes envolvidas sejam declaradas em `serialization-config.json`:

```json
[
  {
    "name": "br.com.exemplo.dominio.Sessao",
    "allDeclaredFields": true,
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true
  }
]
```

Via `RuntimeHints`:

```java
hints.serialization()
    .registerType(Sessao.class)
    .registerType(DadosCache.class);
```

**Recomendação**: evitar a serialização Java nativa em novos projetos. Preferir JSON (Jackson), Protocol Buffers ou outros formatos com suporte explícito a Native Image.

---

## 13. Lombok

O Lombok atua exclusivamente no processamento de anotações durante a compilação Java. Ele não tem dependências em tempo de execução e é totalmente compatível com Native Image. Contudo, existem alguns pontos de atenção:

### 13.1. @SneakyThrows

```java
// Lombok gera código de supressão de exceções verificadas
// Funciona corretamente no Native Image
@SneakyThrows
public String lerArquivo(String caminho) {
    return Files.readString(Path.of(caminho));
}
```

### 13.2. @Builder com herança

```java
// Conflito com Lombok + herança em Native Image (raro, mas possível)
@Data
@SuperBuilder          // requer @SuperBuilder nas subclasses também
public class EntidadeBase {
    private Long id;
    private LocalDateTime criadoEm;
}

@Data
@SuperBuilder
@EqualsAndHashCode(callSuper = true)
public class Produto extends EntidadeBase {
    private String nome;
    private BigDecimal preco;
}
```

### 13.3. Lombok e registros de reflexão

DTOs e entidades gerados com `@Data` ou `@Value` têm seus construtores, getters e setters acessíveis apenas via métodos gerados, que já são incluídos no bytecode. Não é necessário nenhum hint especial para Lombok.

---

## 14. MapStruct

O MapStruct também opera em tempo de compilação e gera implementações concretas de interfaces de mapeamento. É totalmente compatível com Native Image:

```java
package br.com.exemplo.mapeamento;

import org.mapstruct.Mapper;
import org.mapstruct.Mapping;

// MapStruct gera ProdutoMapperImpl em tempo de compilação — sem reflexão
@Mapper(componentModel = "spring")
public interface ProdutoMapper {

    @Mapping(source = "categoriaId", target = "categoria.id")
    Produto dtoParaEntidade(ProdutoDTO dto);

    @Mapping(source = "categoria.id", target = "categoriaId")
    ProdutoDTO entidadeParaDto(Produto produto);
}
```

**Conflito potencial**: quando o MapStruct usa `ReflectionUtils` internamente para mapeamentos complexos com `qualifiedByName` e tipos genéricos. Nesses casos, adicionar hints de reflexão para os tipos envolvidos resolve o problema.

---

## 15. Bibliotecas de terceiros e compatibilidade

### 15.1. Verificando suporte nativo

O repositório [GraalVM Reachability Metadata](https://github.com/oracle/graalvm-reachability-metadata) contém metadados contribuídos pela comunidade para bibliotecas populares. O `native-maven-plugin` os inclui automaticamente:

```xml
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <configuration>
        <!-- Ativa metadados de alcançabilidade da comunidade -->
        <metadataRepository>
            <enabled>true</enabled>
        </metadataRepository>
    </configuration>
</plugin>
```

### 15.2. Bibliotecas com suporte confirmado (Spring Boot 3.x)

| Biblioteca                  | Suporte nativo | Observações                                      |
|-----------------------------|----------------|--------------------------------------------------|
| Spring Web MVC              | Completo       | Configuração automática pelo AOT                 |
| Spring WebFlux              | Completo       | Preferido para alta concorrência nativa          |
| Spring Data JPA + Hibernate | Parcial        | Requer enhance estático; sem DDL dinâmico        |
| Spring Data MongoDB         | Completo       | Metadados incluídos na biblioteca                |
| Spring Data Redis           | Completo       | Metadados incluídos na biblioteca                |
| Spring Security             | Completo       | Configurações AOT automáticas                    |
| Flyway                      | Completo       | —                                                |
| Liquibase                   | Completo       | —                                                |
| Caffeine Cache              | Completo       | —                                                |
| Micrometer / Prometheus     | Completo       | —                                                |
| Resilience4j                | Completo       | —                                                |
| Lombok                      | Completo       | Somente compile-time; sem impacto em runtime     |
| MapStruct                   | Completo       | Somente compile-time                             |
| Apache POI                  | Parcial        | Requer hints extensos; avaliar alternativas      |
| Quartz Scheduler            | Parcial        | Jobs dinâmicos e persistência requerem hints     |
| Jasper Reports              | Não suportado  | Usa compilação Groovy em tempo de execução       |
| Drools / jBPM               | Não suportado  | Geração de bytecode em tempo de execução         |

### 15.3. Estratégias para bibliotecas sem suporte

Quando uma biblioteca não suporta Native Image, há três alternativas principais:

**a) Isolar em um microsserviço JVM separado**

```
┌─────────────────────┐    HTTP/gRPC    ┌──────────────────────────┐
│  API Principal      │ ─────────────── │  Serviço de Relatórios   │
│  (Native Image)     │                 │  (JVM — Jasper Reports)  │
└─────────────────────┘                 └──────────────────────────┘
```

**b) Contribuir metadados ao repositório de comunidade**

Criar `reflect-config.json`, `resource-config.json` e `proxy-config.json` usando o **GraalVM Tracing Agent** e submeter uma contribuição ao repositório oficial.

**c) Substituir a biblioteca por uma compatível**

Exemplo: substituir Apache POI por [FastExcel](https://github.com/dhatim/fastexcel), que foi projetado com Native Image em mente.

---

## 16. GraalVM Tracing Agent

### 16.1. O que é e quando usar

O Tracing Agent é uma ferramenta do GraalVM que monitora a aplicação em execução normal (na JVM) e registra todos os usos de reflexão, proxies e recursos, gerando automaticamente os arquivos de metadados necessários.

```bash
# Executar a aplicação com o agente de rastreamento ativo
java -agentlib:native-image-agent=config-output-dir=src/main/resources/META-INF/native-image \
     -jar target/minha-aplicacao.jar
```

Depois de executar os casos de uso relevantes (navegação pela API, testes de integração, cenários de erro), encerrar a aplicação. Os arquivos de configuração são gravados automaticamente.

### 16.2. Cobertura dos cenários de rastreamento

O Tracing Agent só gera metadados para os caminhos de código **efetivamente executados**. É fundamental cobrir todos os fluxos relevantes durante o rastreamento:

```bash
# Script de exercício da API após iniciar com o agente ativo
curl -X POST http://localhost:8080/api/pedidos \
     -H "Content-Type: application/json" \
     -d '{"clienteId": 1, "itens": [{"produtoId": 10, "quantidade": 2}]}'

curl http://localhost:8080/api/pedidos/1

curl -X DELETE http://localhost:8080/api/pedidos/1

# Testar também fluxos de erro e exceção
curl http://localhost:8080/api/pedidos/9999
```

### 16.3. Integrando o agente com testes

```xml
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <configuration>
        <agent>
            <!-- Ativa o agente durante os testes de integração -->
            <enabled>true</enabled>
            <options>
                <option>experimental-class-loader-support</option>
            </options>
        </agent>
    </configuration>
</plugin>
```

```bash
# Executar testes com agente de rastreamento ativo
./mvnw -Pagent test

# Os metadados são gerados em target/native/agent-output/
```

---

## 17. Inicialização de classes em tempo de build vs. tempo de execução

### 17.1. Inicialização no build (build-time initialization)

Por padrão, o GraalVM inicializa as classes em tempo de execução. Contudo, algumas classes podem — e devem — ser inicializadas durante o build para evitar problemas:

```
# Adicionar ao native-maven-plugin buildArgs
--initialize-at-build-time=org.slf4j.LoggerFactory
--initialize-at-build-time=ch.qos.logback.classic.Logger
```

```java
@Bean
public NativeImageOptions nativeImageOptions() {
    NativeImageOptions options = new NativeImageOptions();
    options.getInitializeAtBuildTime().add("org.slf4j.LoggerFactory");
    return options;
}
```

### 17.2. Conflito com estado mutável em inicialização no build

Inicializar uma classe no build que mantém estado mutável (como conexões com banco de dados ou caches em memória inicializados no static block) é um erro grave:

```java
// Conflito crítico: estado inicializado no build é "fotografado" no binário
// e reutilizado em todas as execuções
public class ConexaoPool {
    private static final DataSource POOL = criarPool();   // NÃO deve ser inicializado no build!
    // o pool seria congelado no binário com configurações do ambiente de build
}

// Correto: inicialização lazy ou via Spring IoC
@Configuration
public class DataSourceConfig {
    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
        // inicializado em tempo de execução, com variáveis de ambiente corretas
    }
}
```

---

## 18. Variáveis de ambiente e configuração em tempo de execução

### 18.1. Propriedades dinâmicas vs. fixadas no build

Em Native Image, o arquivo `application.properties` ou `application.yml` incluído no classpath é parte do binário. Contudo, variáveis de ambiente e propriedades passadas via linha de comando ainda são aplicadas em tempo de execução:

```yaml
# Fixado no build (pode ser sobrescrito por variáveis de ambiente)
spring:
  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/dev}
    username: ${DATABASE_USER:dev}
    password: ${DATABASE_PASSWORD:dev}
```

```bash
# Sobrescrita em tempo de execução — funciona normalmente
DATABASE_URL=jdbc:postgresql://prod.servidor.com:5432/producao \
DATABASE_USER=app_user \
DATABASE_PASSWORD=senha_segura \
./minha-aplicacao
```

### 18.2. Profiles Spring em Native Image

Perfis Spring são avaliados **em tempo de execução** em Native Image (desde Spring Boot 3.1). Isso significa que é possível ter um único binário nativo e ativá-lo com perfis diferentes:

```bash
# Ativar perfil de produção em tempo de execução
./minha-aplicacao --spring.profiles.active=producao
```

**Restrição**: beans condicionais (`@ConditionalOnProfile`) cujos efeitos afetam quais classes são incluídas no binário ainda são avaliados no build. Apenas a seleção de configurações de propriedades por perfil funciona totalmente em tempo de execução.

---

## 19. Diagnóstico e depuração de compilações nativas

### 19.1. Opções de diagnóstico no build

```xml
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <configuration>
        <buildArgs>
            <!-- Rastrear origens de classes inicializadas no build -->
            <buildArg>--trace-class-initialization=br.com.exemplo</buildArg>

            <!-- Relatório detalhado de exceções em tempo de execução -->
            <buildArg>-H:+ReportExceptionStackTraces</buildArg>

            <!-- Mostrar stack trace completo em erros de reflexão -->
            <buildArg>-H:+PrintAnalysisCallTree</buildArg>

            <!-- Diagnóstico de missing registrations (GraalVM 22.3+) -->
            <buildArg>--strict-image-heap</buildArg>
        </buildArgs>
    </configuration>
</plugin>
```

### 19.2. MissingReflectionRegistrationError

O erro mais comum em Native Image é tentar acessar via reflexão uma classe não registrada:

```
Exception in thread "main" com.oracle.svm.core.jdk.UnsupportedFeatureError:
  Reflection classes not registered for runtime access:
  class br.com.exemplo.dominio.Produto

For full details, run with -Dspring.native.verbose=true
```

Solução: adicionar `@RegisterReflectionForBinding(Produto.class)` ou configurar `RuntimeHintsRegistrar`.

### 19.3. MissingResourceRegistrationError

```
java.io.FileNotFoundException: Resource not found: templates/email.html
```

Solução: registrar o recurso via hints ou em `resource-config.json`.

### 19.4. Modo verbose do Spring Native

```bash
# Ativar logs detalhados do processador AOT
./mvnw spring-boot:process-aot -Dspring.native.verbose=true

# Executar o Native Image com diagnóstico de problemas de inicialização
./minha-aplicacao -Dspring.native.verbose=true
```

### 19.5. Usando o Native Build Tools Dashboard

```bash
# Gerar relatório HTML de análise do binário nativo
./mvnw -Pnative native:compile \
  -Dspring-boot.native.build-args="-H:+DashboardAll -H:DashboardDump=./native-dashboard"
```

---

## 20. Testes em modo nativo

### 20.1. @SpringBootTest com Native Image

```java
package br.com.exemplo;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.aot.DisabledInAotMode;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class MinhaAplicacaoNativeIT {

    @Autowired
    private ProdutoService produtoService;

    @Test
    void deveListarProdutos() {
        var produtos = produtoService.listarTodos();
        assertThat(produtos).isNotNull();
    }

    // Desabilitar testes que dependem de comportamento exclusivo da JVM
    @Test
    @DisabledInAotMode
    void testeDependenteDeReflexaoDinamica() {
        // este teste só roda na JVM, não no Native Image
    }
}
```

### 20.2. Executando testes nativos

```bash
# Compilar e executar testes no Native Image
./mvnw -Pnative test

# Apenas compilar os testes nativos sem executar
./mvnw -Pnative native:compile -DskipTests
```

---

## 21. Boas práticas consolidadas

### 21.1. Design para compatibilidade nativa

1. **Prefer interfaces a classes concretas** em pontos de injeção para facilitar proxies JDK em vez de CGLIB;
2. **Evitar `Class.forName` e `Method.invoke`** diretamente; usar a abstração do Spring quando possível;
3. **Nunca usar classes `final` em beans** que precisam de proxies AOP (transação, cache, segurança);
4. **Migrar de serialização Java** (`Serializable`) para JSON para dados distribuídos;
5. **Declarar recursos explicitamente** em vez de descobri-los dinamicamente;
6. **Usar Flyway ou Liquibase** para gerenciamento de schema; evitar `ddl-auto=create`;
7. **Testar com o Tracing Agent** no pipeline de CI para capturar regressões de metadados.

### 21.2. Checklist antes do build nativo

```
[ ] Todos os DTOs e entidades mapeados com Jackson estão anotados com @RegisterReflectionForBinding?
[ ] Tipos polimórficos Jackson têm todos os subtipos registrados?
[ ] Interfaces de repositório Spring Data têm proxies JDK registrados?
[ ] Recursos estáticos (templates, arquivos de configuração) estão em resource-config.json?
[ ] Bundles de mensagens (MessageSource) estão registrados?
[ ] Bibliotecas de terceiros têm suporte confirmado ou metadados customizados?
[ ] Nenhuma classe final com @Transactional/@Cacheable/@Async?
[ ] Spring JPA configurado com enhance estático e ddl-auto=validate?
[ ] Variáveis de ambiente usadas em vez de valores fixos para configurações de ambiente?
[ ] Testes de integração cobrem todos os fluxos para o Tracing Agent?
```

### 21.3. Quando não usar Native Image

O Native Image não é a escolha certa em todos os cenários. Considerar manter a JVM quando:

- A aplicação usa **bibliotecas sem suporte a Native Image** (Jasper Reports, Drools, etc.) de forma central;
- O **tempo de aquecimento da JVM não é crítico** e os benefícios de desempenho do JIT superam o consumo inicial de memória;
- A **equipe não tem familiaridade** com os conceitos de closed-world e os ciclos de depuração de metadados adicionariam risco ao projeto;
- A aplicação realiza **carregamento dinâmico extensivo** de plugins ou módulos em tempo de execução.
