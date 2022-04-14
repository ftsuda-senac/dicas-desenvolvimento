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
    * Criar beans programaticamente via `@Bean`
    * Usar valores padrões nas anotações @Value (ex: `@Value("${algo-boolean:true}`)
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
    * [https://www.baeldung.com/spring-bean-scopes](https://www.baeldung.com/spring-bean-scopes)
* i18n
* OpenAPI/Swagger2
* Upload e mapeamento para acesso via HTTP
    * Integração com serviços externos (ex: AWS S3)
* HATEOAS

## Data JPA
* Formas de desenvolver lógica do repositório
    * Query methods
    * Uso do `@Query` com JPQL
    * Specification ​+ Criteria API
        * Geração dos metamodels usando Hibernate JPA Modelgen
            * Fazer via console usando `mvn clean generate-sources`, senão ao abrir projeto no Eclipse gera erros
            * Não salvar arquivos gerados no repositório de código-fonte
    * Uso do JPA "puro"
    * Uso do JDBC

* Outras dicas e pontos de atenção
    * Uo da configuração `open-in-view=false`
    * Uso do `@Transactional` (importado de `org.springframework.transaction.annotation.Transactional`) na camada `@Service`
    * Uso do Hibernate Envers para auditoria
        * Criar schema a parte no Banco de dados
    * Mapeamento das entidades com annotations do JPA
        * Fetchs EAGER, LAZY e como configurar fetch nas consultas usando Entity Graph
        * Uso do Cascade e orphanRemoval
        * IDs compostos para relacionamentos many-to-many "manuais" + campos extras na relação
        * Uso dos Listeners de eventos (ex: `@PostLoad`, `@PrePersist`, `@PostPersist`, etc)
            * Referência: https://www.baeldung.com/jpa-entity-lifecycle-events
        * Nomear FKs
        * Criação de índices
        * Usar mesma PK em relações do tipo one-to-one
        * Identificadores internos e externos (UUID)

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

## "Receitas de bolo" para requisitos de alguns microservices
* Autenticação/Autorização de acesso
    * Cadastro de novo usuário
    * Controle de tentativa máximas de erros
    * Troca de credenciais (alterar senha)
    * Reinicializar senha
        * Self-service
        * Via help-desk
        * Mecanismos para gerar códigos de confirmação (TODO)
    * Auditoria
    * n-factor authentication
        * Referência: https://blog.nec.com.br/autenticacao-de-usuario-um-mundo-alem-de-senhas
* Disparo de tarefas automatizadas com Quartz
    * Cadastrar tarefas
    * Ativar/Desativar dinamicamente
    * Registrar data/horario e resultados de execução
* Sistema de comentários
    * Apresentação dos comentários
    * Entrada de comentário
    * Moderação, Aprovação/Reprovação
* Sistema de notas/ranking
* Sistema newsletter
    * Inscrição
    * Cancelamento de inscrição
* CRUD genérico
    * Listagem das informações
        * Buscas textuais
        * Filtros
            * Inclusão de Filtro
            * Remoção de Filtro
            * Resultados facetados (refinamento dos filtros - Conceito do Solr)
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

## Outros
* Lombok
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
        * Arquivo properties externo (fora do diretório de deploy) X configurações BD X environment variables -> Confirmar hot-reload
        * Em containers, prever uso de volumes para manter estes arquivos
    * Configuração externa de logs (SLF4J, logback)
