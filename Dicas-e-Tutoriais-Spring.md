# Spring

## Arquitetura
* 3-tier (Domain, Services, Presentation) -> Ver DDD ou Onion Architecture
    * https://blog.avenuecode.com/domain-driven-design-and-onion-architecture#:~:text=Onion%20Architecture%20is%20based%20on,but%20rather%20on%20domain%20models.
    * https://www.infoq.com/br/articles/onion-architecture/
    * https://www.javaguides.net/2020/07/three-tier-three-layer-architecture-in-spring-mvc-web-application.html
* Anotações `@Component`, `@Controller`, `@Service`, `@Repository`
    * Premissa de um `@Service` para quem desenvolve web: a funcionalidade será executada corretamente se for feita através de outra interface de operação, como por exemplo linha de comando (considerar entradas primárias tipo String, int, validações, mensagens de sucesso, erros, etc)?
* Configurações
    * application.properties
        * Acesso a variáveis de ambiente
        * SpEL com valor padrão no arquivo de propriedades
    * Uso de profiles

## Core
* Injeção de dependências
    * Processo de component scanning
    * Criar beans programaticamente através de classes anotadas com `@Configuration`e métodos anotados com `@Bean`
    * Usar valores padrões nas anotações `@Value`
    ```java
    @Value("${valor-booolean-properties:true}")
    boolean valorBooleanoInjetado;
    ```
    
* Spring Boot/Spring Initialzr
    * Cuidado ao configurar Beans que sobrescrevem o comportamento padrão
    * Dependências úteis: devtools, actuator, configuration-processor
* EventListeners
* SpEL

## MVC
* `@Controller` e `@RestController`
* Tratamento de erros com `@ControllerAdvice`
* Escopo dos beans
    * `@RequestScope`
    * `@SessionScope` (cuidado ao usar em aplicações REST)
    * `@ApplicationScope`
    * https://www.baeldung.com/spring-bean-scopes
* i18n
* OpenAPI/Swagger2
* Upload e mapeamento para acesso via HTTP
    * Integração com serviços externos (ex: AWS S3)
* Utilitários 
* HATEOAS

## Data JPA
* Entender a hierarquia das interfaces `Repository` <- `CrudRepository` <- `PagingAndSortingRepository` <- `JpaRepository` e verificar as funcionalidades já fornecidas por padrão https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.core-concepts

```mermaid
classDiagram
class Repository {
    <<interface>>
}
class CrudRepository {
    <<interface>>
}
class PagingAndSortingRepository {
    <<interface>>
}
class JpaRepository {
    <<interface>>
}
Repository <|-- CrudRepository
CrudRepository <|-- PagingAndSortingRepository
PagingAndSortingRepository <|-- JpaRepository
```

* Formas de desenvolver lógica do repositório
    * Query methods - uso de convenções de nomenclatura dos métodos da interface. Dessa forma, o Spring Data irá criar automaticamente as queries de consulta ao repositório
        * https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.details
        * https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods
        * https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords
    * Uso do `@Query` com JPQL OU SQL nativo (dependente do Banco de dados usado)
        * Envolve definir um método qualquer no repositório e incluir a anotação `@Query` (importado de `org.springframework.data.jpa.repository.Query`) com a JPQL de consulta e sempre que possivel usando parametros nomeados com @Param (importado de `org.springframework.data.repository.query.Param`) ou query nativa
         ```java
         // JPQL
         @Query("SELECT p FROM Person p WHERE p.firstName = :nameParam")
         Optional<Person> findSomethingJpql(@Param("nameParam") String nameParam);

         // SQL nativo - sintaxe pode variar dependendo do banco de dados e comandos usados
         // Lembrar de usar nomes de tabelas e colunas de acordo com o banco de dados e não à classe
         @Query(nativeQuery = true, value = "SELECT * FROM person WHERE first_name = :nameParam")
         Optional<Person> findSomethingNative(@Param("nameParam") String nameParam);
         ```
        
        * Exemplo para concatenar parâmetros às funções e wildcards
        ```java
        // JPQL
        @Query("SELECT p FROM Person p WHERE upper(p.fullName) LIKE UPPER('%'||:searchTerm||'%')")
        List<Person> searchByNameJpql(@Param("searchTerm") String searchTerm);
        
        // SQL nativo
        @Query(nativeQuery = true, value = "SELECT * FROM person WHERE upper(full_name) LIKE upper('%'||:searchTerm||'%')")
        List<Person> searchByNameNative(@Param("searchTerm") String searchTerm);
        ```

    * Specification ​+ Criteria API
        * Geração dos metamodels usando Hibernate JPA Modelgen
            * Fazer via console usando `mvn clean generate-sources`, senão ao abrir projeto no Eclipse gera erros
            * Não salvar arquivos gerados no repositório de código-fonte.
            * Tutorial de Criteria/JPQL: https://www.objectdb.com/java/jpa/query/jpql/structure
    * Uso do JPA "puro"
    * Uso do JDBC
        * Lembrar de usar **try-with-resources** ao usar recursos como `Connection`, `PreparedStatement` e `ResultSet`

* Outras dicas e pontos de atenção
    * Uo da configuração `open-in-view=false` (considerada má-prática, pois mantém uma conexão aberta com o banco de dados para cada acesso realizado à aplicação)
    * Na criação das Entities, sempre que possível usar as anotações padrão do JPA puro (pacote `jakartar.persistence`/`javax.persistence`) e evitar usar anotações específicas do Hibernate.
    * Uso do `@Transactional` (importado de `org.springframework.transaction.annotation.Transactional`) na camada `@Service`
        * TODO: Ver diferenças de comportamento com `javax.transaction.Transactional` do JEE
            * https://www.baeldung.com/spring-vs-jta-transactional
            * https://stackoverflow.com/a/62702146
    * Uso do Hibernate Envers para auditoria
        * Criar schema a parte no Banco de dados
    * Mapeamento das entidades com annotations do JPA
        * Fetchs EAGER, LAZY e como configurar fetch nas consultas usando Entity Graph
            * Atentar ao uso do "fetch EAGER" nas anotações de relacionamento (@OneToOne, @OneToMany/@ManyToOne e @ManyToMany). Se usado incorretamente, pode ocasionar problema sérios de desempenho. Sempre dar preferência ao uso do "fetch LAZY" e quando necessário fazer o fetch manualmente usando JOIN FETCH do JPQL ou usar `@EntityGraph` ou `@NamedEntityGraph`.
        * Uso do Cascade e orphanRemoval
        * IDs compostos para relacionamentos many-to-many "manuais" + campos extras na relação
        * Uso dos Listeners de eventos (ex: `@PostLoad`, `@PrePersist`, `@PostPersist`, etc)
            * Referência: https://www.baeldung.com/jpa-entity-lifecycle-events
        * Nomear FKs
        * Criação de índices.
        * Usar mesma PK em relações do tipo one-to-one
            * Sempre que possível, associações @OneToOne devem ser feitas usando o ID da entidade principal como PK da entidade secundária (usar @JoinColumn + @MapsId OU @PrimaryKeyJoinColumn).
        * Identificadores internos e externos (UUID).
    * Integração com Project Lombok
        * Evitar usar `@Data`, pois ele transforma a classe em `Serializable` e reimplementa o `equals`, podendo ocasionar incompatibilidades com o modelo de funcionamento do JPA
        * Cuidado ao usar `@ToString`, pois se estiver mal configurado pode gerar loops infinitos.
    * Records (Java 14+)
        * Records **NÃO** podem ser usados para representar uma Entidade JPA, pois são imutáveis.
        * https://thorben-janssen.com/java-records-hibernate-jpa/

* Controle de versões do banco de dados
    * Liquidbase
    * Flyway
    * Referências:
        * https://dzone.com/articles/flyway-vs-liquibase
        * https://medium.com/@ruxijitianu/database-version-control-liquibase-versus-flyway-9872d43ee5a4

## Security
* Hash de senhas (ex: bcrypt)
* OpenID
* OAuth2
* JWT
* Keycloak como servidor de autenticação/autorização https://www.keycloak.org/

## Properties úteis

Ver https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties

```
#========= GENERAL CONFIG ==========
debug=true
spring.main.banner-mode=off

# PEGANDO VALOR DO PROFILE CONFIGURADO NO pom.xml
spring.profiles.active=@build.profile.id@
spring.profiles.include=@build.profile.id@

#========= WEB ==========
server.port=8080
server.context-path=/
spring.servlet.multipart.max-file-size=5MB
spring.servlet.multipart.max-request-size=10MB
server.compression.enabled=true
server.compression.min-response-size=50KB

#========= DATABASE/JPA ==========
spring.jpa.open-in-view=false
spring.jpa.show-sql=true
# Pode ser none/validate/create/create-drop/update - https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#configurations-hbmddl
spring.jpa.properties.hibernate.ddl-auto=update

#========== JSON ==========
spring.jackson.serialization.INDENT_OUTPUT=true
spring.jackson.deserialization.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT=true
spring.jackson.deserialization.FAIL_ON_IGNORED_PROPERTIES=false
spring.jackson.deserialization.FAIL_ON_UNKNOWN_PROPERTIES=false

#========== OUTROS ==========
# USO DE VARIÁVEIS DE AMBIENTE
app.some-directory=${HOME}/directory

# PROPRIEDADES COM FALLBACK
app.some-text=${SOME_ENV_VAR:Texto fallback caso variável não exista}
```

```xml
<!-- trecho do pom.xml com declaração do profile -->
<profles>
   <profile>
      <id>dev</id>
      <activation>
         <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
				<build.profile.id>dev</build.profile.id>
      </properties>
      ...
</profiles>
```

## Links úteis

Versões das dependências usadas: https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions

## "Receitas de bolo" para requisitos de alguns microservices
* Autenticação/Autorização de acesso
    * Cadastro de novo usuário
    * Controle de tentativa máximas de erros de login
    * Troca de credenciais (alterar senha)
    * Reinicializar senha
        * Self-service
        * Via help-desk
        * Mecanismos para gerar códigos aleatórios de confirmação (TODO)
    * Auditoria
    * n-factor authentication
        * Referência: https://blog.nec.com.br/autenticacao-de-usuario-um-mundo-alem-de-senhas
    * Gerenciamento
        * Ativação/Inativação de conta
        * Permissões de acesso (Role/Authority)
    * Notificações de eventos (criação, alteração, troca de senha, etc)
* Disparo de tarefas automatizadas com Quartz
    * Cadastrar tarefas
    * Ativar/Desativar execução de dinamicamente
    * Registrar data/horario e resultados de execução
    * cron expressions
* Sistema de comentários
    * Apresentação dos comentários
    * Entrada de comentário
    * Moderação, Aprovação/Reprovação
    * Feedback em casos de reprovação
* Sistema de notas/ranking
* Sistema newsletter
    * Inscrição
    * Cancelamento de inscrição
    * Integração com sistema de notificações
* CRUD genérico
    * Listagem das informações
        * Buscas textuais (Solr, ElasticSearch/Opensearch, PostgreSQL full text search)
        * Filtros
            * Inclusão de Filtro
            * Remoção de Filtro
            * Resultados facetados (refinamento dos filtros - conceito do Solr)
        * Ordenação
        * Paginação/Quantidade de resultados limitados
    * Formulários de inclusão/alteração
        * Validações de campos
            * Client-side (usar recursos do HTML5 para validações simples)
            * Server-side (sempre mandatório)
        * Cadastro com upload de arquivos
* Notificações
    * Internas (dentro do próprio sistema - Webhook, Server-sent events)
    * Envio de e-mails
    * Outros (ex: SMS, WhatsApp, etc)
* LGPD
    * Apresentação dos termos de uso com informações de como os dados serão usados
    * Versionamento dos termos de uso
    * Aceite dos termos pelo usuário

## Outros
* Project Lombok
* Testes unitários e integração
    * JUnit 5
    * BDD + Cuccumber
    * Selenium Webdriver/Cypress
    * JMeter e teste de carga
* Containers
    * Docker
    * Kubernetes
* Webflux
* Websockets
* Spring Data REST
* Boas práticas
    * Configurações estáticas X configurações dinâmicas
        * Arquivo properties externo (fora do diretório de deploy) X configurações BD X environment variables -> Confirmar se "hot-reload" funciona nestes casos
        * Em containers, prever uso de volumes persistentes para manter estes arquivos
    * Configuração externa de logs (SLF4J, logback)