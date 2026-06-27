# Spring Boot Avançado

Este documento reúne informações e exemplos práticos sobre recursos avançados usados com frequência em aplicações Spring Boot:

- envio de e-mails com configuração padrão, configuração dinâmica de servidores SMTP e templates Thymeleaf;
- execução de jobs com Quartz, incluindo persistencia em banco, controle dinâmico e disparo manual;
- uso de JMS com Artemis, cobrindo filas, pub-sub, filas duráveis, uso de `pooled-jms` e mensagens com resposta;
- mensageria distribuída com Kafka e RabbitMQ, incluindo produtores, consumidores, DLQ e boas práticas;
- uso de Spring Events para comunicação interna desacoplada, com listeners síncronos, assíncronos e transacionais;
- GraphQL com Spring for GraphQL, incluindo schema, controllers, `@BatchMapping` e resolução do problema N+1;
- gRPC com Spring Boot, incluindo Protocol Buffers, implementação de servidor e cliente, e tipos de streaming;
- documentação de APIs com SpringDoc OpenAPI, incluindo Swagger UI, anotações e agrupamento de APIs;
- mecanismos de resiliência, incluindo timeout, retry, circuit breaker, bulkhead, rate limiter e fallback;
- rate limiting com Bucket4j para controle de acesso por IP, usuário, tenant ou chave de negócio;
- processamento em lote com Spring Batch, incluindo `Job`, `Step`, chunk processing, restart, skip e retry;
- cache e Spring Session, incluindo `@Cacheable`, `@CachePut`, `@CacheEvict`, Caffeine, Redis e sessão distribuída;
- upload e download de arquivos, incluindo `MultipartFile`, streaming, armazenamento local, validações de segurança e detecção de tipo real com Apache Tika;
- leitura e exportação de dados em CSV e Excel com Apache Commons CSV, Apache POI e `excel-streaming-reader`;
- concorrência com `java.util.concurrent`, incluindo locks (`ReentrantLock`, `ReentrantReadWriteLock`, `StampedLock`), classes atômicas (`AtomicInteger`, `LongAdder`, `AtomicReference`), coleções concorrentes (`ConcurrentHashMap`, `CopyOnWriteArrayList`, filas bloqueantes) e sincronizadores (`CountDownLatch`, `Semaphore`, `CyclicBarrier`, `Phaser`);
- Actuator e métricas customizadas, incluindo endpoints de saúde, health indicators, métricas técnicas e de negócio com Micrometer (`Counter`, `Timer`, `Gauge`, `DistributionSummary`), e integração com OpenTelemetry para traces, métricas e logs distribuídos;
- feature flags e estratégias de deploy, incluindo Blue-Green, Canary, Rolling Update e implementação de toggles com Spring.

Os exemplos abaixo seguem um estilo compatível com Spring Boot 3.x e Jakarta EE.

## 1. Envio de e-mails

### 1.1. Dependências

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

### 1.2. Configuração padrão

Quando a aplicação usa um único servidor SMTP, a forma padrão e configurar tudo em `application.yml`.

```yaml
spring:
  mail:
    host: smtp.office365.com
    port: 587
    username: no-reply@empresa.com.br
    password: ${MAIL_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
          connectiontimeout: 5000
          timeout: 5000
          writetimeout: 5000
```

Com isso, o Spring Boot cria automaticamente um `JavaMailSender`.

### 1.3. Exemplo de envio simples

```java
package br.com.exemplo.mail;

import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

@Service
public class EmailService {

    private final JavaMailSender mailSender;

    public EmailService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void enviarTexto(String destinatario, String assunto, String conteudo) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(destinatario);
        message.setSubject(assunto);
        message.setText(conteudo);
        mailSender.send(message);
    }
}
```

### 1.4. Exemplo de envio HTML com Thymeleaf

Template estático:

```text
src/main/resources/templates/mail/boas-vindas.html
```

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <h1>Bem-vindo(a), <span th:text="${nome}">Nome</span>!</h1>
    <p>Seu cadastro foi realizado com sucesso.</p>
    <p>Código de ativacao: <strong th:text="${codigoAtivacao}">0000</strong></p>
</body>
</html>
```

Configuração do Template Engine do Thymeleaf:

```java
@Configuration
public class ThymeleafEmailConfig {

    // PRECISA ADICIONAR O ACESSO AO MESSAGE SOURCE PARA SUPORTE A i18n
	@Bean(name = "emailMessageSource")
	MessageSource emailMessageSource() {
		ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
		messageSource.setBasename("classpath:i18n/email-messages"); // Arquivos em /resources/i18n/email-messages_[LOCALE].properties
		messageSource.setDefaultEncoding(StandardCharsets.UTF_8.toString());
		messageSource.setFallbackToSystemLocale(true);
		messageSource.setCacheSeconds(10); //reload messages every 10 seconds
		return messageSource;
	}

	@Bean(name = "emailTemplateEngine")
	TemplateEngine emailTemplateEngine(@Qualifier("emailMessageSource") MessageSource emailMessageSource) {
		ClassLoaderTemplateResolver configurationTemplateResolver = new ClassLoaderTemplateResolver();
		configurationTemplateResolver.setTemplateMode(TemplateMode.HTML);
		configurationTemplateResolver.setPrefix("templates/email/");
		configurationTemplateResolver.setSuffix(".html");
		configurationTemplateResolver.setCharacterEncoding(StandardCharsets.UTF_8.toString());
		configurationTemplateResolver.setOrder(1);
		configurationTemplateResolver.setCheckExistence(true);

		SpringTemplateEngine templateEngine = new SpringTemplateEngine();
		templateEngine.setTemplateResolver(configurationTemplateResolver);
		templateEngine.setMessageSource(emailMessageSource);
		templateEngine.setTemplateEngineMessageSource(emailMessageSource);
		templateEngine.setEnableSpringELCompiler(true);
		templateEngine.addDialect(new LayoutDialect()); // REQUER dependency do layout-dialect no pom.xml
		return templateEngine;
	}
}
```

Serviço:

```java
package br.com.exemplo.mail;

import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import java.nio.charset.StandardCharsets;
import java.util.Map;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class EmailTemplateService {

    private final JavaMailSender mailSender;
    private final TemplateEngine templateEngine;

    public EmailTemplateService(JavaMailSender mailSender,
            @Qualifier("emailTemplateEngine") TemplateEngine templateEngine) {
        this.mailSender = mailSender;
        this.templateEngine = templateEngine;
    }

    public void enviarBoasVindas(String destinatario, Map<String, Object> variaveis)
            throws MessagingException {
        Context context = new Context();
        context.setVariables(variaveis);

        String html = templateEngine.process("boas-vindas", context); // Arquvo  src/main/templates/email/boas-vindas.html
                                                                      // pasta "email" já foi configurada no emailTemplateEngine

        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(
                mimeMessage,
                MimeMessageHelper.MULTIPART_MODE_MIXED_RELATED,
                StandardCharsets.UTF_8.name()
        );

        helper.setTo(destinatario);
        helper.setSubject("Bem-vindo ao sistema");
        helper.setText(html, true);

        mailSender.send(mimeMessage);
    }
}
```

### 1.5. Cadastro e obtenção dinamica dos servidores de e-mail

Quando a aplicação precisa suportar vários clientes, tenants ou unidades, é comum armazenar as configurações SMTP no banco.

Entidade:

```java
package br.com.exemplo.mail.config;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "mail_server_config")
public class MailServerConfig {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String codigo;

    @Column(nullable = false)
    private String host;

    @Column(nullable = false)
    private Integer port;

    @Column(nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private boolean smtpAuth;

    @Column(nullable = false)
    private boolean startTls;

    @Column(nullable = false)
    private boolean ativo = true;

    @Column(nullable = false)
    private String remetentePadrao;

    // Getters e setters omitidos por brevidade.
}
```

Repositório:

```java
package br.com.exemplo.mail.config;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MailServerConfigRepository extends JpaRepository<MailServerConfig, Long> {

    Optional<MailServerConfig> findByCodigoAndAtivoTrue(String codigo);
}
```

Fábrica dinâmica de `JavaMailSender`:

```java
package br.com.exemplo.mail.config;

import java.util.Properties;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.stereotype.Component;

@Component
public class DynamicMailSenderFactory {

    public JavaMailSender criar(MailServerConfig config) {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        sender.setHost(config.getHost());
        sender.setPort(config.getPort());
        sender.setUsername(config.getUsername());
        sender.setPassword(config.getPassword());

        Properties props = sender.getJavaMailProperties();
        props.put("mail.smtp.auth", config.isSmtpAuth());
        props.put("mail.smtp.starttls.enable", config.isStartTls());
        props.put("mail.smtp.connectiontimeout", 5000);
        props.put("mail.smtp.timeout", 5000);
        props.put("mail.smtp.writetimeout", 5000);

        return sender;
    }
}
```

Serviço de envio usando configuração dinâmica:

```java
package br.com.exemplo.mail;

import br.com.exemplo.mail.config.DynamicMailSenderFactory;
import br.com.exemplo.mail.config.MailServerConfig;
import br.com.exemplo.mail.config.MailServerConfigRepository;
import jakarta.mail.internet.MimeMessage;
import java.nio.charset.StandardCharsets;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;

@Service
public class DynamicMailService {

    private final MailServerConfigRepository repository;
    private final DynamicMailSenderFactory mailSenderFactory;

    public DynamicMailService(
            MailServerConfigRepository repository,
            DynamicMailSenderFactory mailSenderFactory
    ) {
        this.repository = repository;
        this.mailSenderFactory = mailSenderFactory;
    }

    public void enviar(String codigoServidor, String destinatario, String assunto, String html)
            throws Exception {
        MailServerConfig config = repository.findByCodigoAndAtivoTrue(codigoServidor)
                .orElseThrow(() -> new IllegalArgumentException("Servidor SMTP não encontrado"));

        JavaMailSender sender = mailSenderFactory.criar(config);

        MimeMessage mimeMessage = sender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(
                mimeMessage,
                StandardCharsets.UTF_8.name()
        );

        helper.setFrom(config.getRemetentePadrao());
        helper.setTo(destinatario);
        helper.setSubject(assunto);
        helper.setText(html, true);

        sender.send(mimeMessage);
    }
}
```

API básica para cadastro e consulta:

```java
package br.com.exemplo.mail.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

public record MailServerConfigRequest(
        @NotBlank String codigo,
        @NotBlank String host,
        @NotNull Integer port,
        @NotBlank String username,
        @NotBlank String password,
        boolean smtpAuth,
        boolean startTls,
        @NotBlank String remetentePadrao
) {
}
```

```java
package br.com.exemplo.mail.api;

import br.com.exemplo.mail.config.MailServerConfig;
import br.com.exemplo.mail.config.MailServerConfigRepository;
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/mail-servers")
public class MailServerConfigController {

    private final MailServerConfigRepository repository;

    public MailServerConfigController(MailServerConfigRepository repository) {
        this.repository = repository;
    }

    @PostMapping
    public MailServerConfig criar(@RequestBody @Valid MailServerConfigRequest request) {
        MailServerConfig config = new MailServerConfig();
        config.setCodigo(request.codigo());
        config.setHost(request.host());
        config.setPort(request.port());
        config.setUsername(request.username());
        config.setPassword(request.password());
        config.setSmtpAuth(request.smtpAuth());
        config.setStartTls(request.startTls());
        config.setRemetentePadrao(request.remetentePadrao());
        config.setAtivo(true);
        return repository.save(config);
    }

    @GetMapping("/{codigo}")
    public MailServerConfig buscar(@PathVariable String codigo) {
        return repository.findByCodigoAndAtivoTrue(codigo)
                .orElseThrow(() -> new IllegalArgumentException("Configuração não encontrada"));
    }
}
```

Boas práticas:

- proteger senha com criptografia ou secret manager;
- validar conectividade antes de ativar a configuração;
- manter cache com invalidação se houver muitas leituras;
- nunca expor credenciais em logs ou APIs administrativas.

### 1.6. Gerenciamento de templates com Thymeleaf

#### Templates estáticos

Os templates estáticos ficam versionados com a aplicação:

```text
src/main/resources/templates/email/
  notificacao.html
  redefinicao-senha.html
  cobranca.html
```

Eles são ideais quando:

- o layout muda pouco;
- a equipe quer revisar alterações pelo Git;
- a publicação do template faz parte do deploy.

#### Templates dinâmicos

Quando o conteúdo precisa ser alterado sem novo deploy, pode-se salvar o HTML no banco e processa-lo com `StringTemplateResolver`.

Entidade:

```java
package br.com.exemplo.mail.template;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "mail_template")
public class MailTemplate {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String codigo;

    @Column(nullable = false)
    private String assunto;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String conteudoHtml;

    @Column(nullable = false)
    private boolean ativo = true;

    // Getters e setters omitidos por brevidade.
}
```

Repositorio:

```java
package br.com.exemplo.mail.template;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MailTemplateRepository extends JpaRepository<MailTemplate, Long> {

    Optional<MailTemplate> findByCodigoAndAtivoTrue(String codigo);
}
```

Configuração do renderizador (ou usar configuração mostrada anteriormente)

```java
package br.com.exemplo.mail.template;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.templatemode.TemplateMode;
import org.thymeleaf.templateresolver.StringTemplateResolver;

@Configuration
public class DynamicTemplateConfig {

    @Bean
    public TemplateEngine dynamicTemplateEngine() {
        StringTemplateResolver resolver = new StringTemplateResolver();
        resolver.setTemplateMode(TemplateMode.HTML);
        resolver.setCacheable(false);

        TemplateEngine templateEngine = new TemplateEngine();
        templateEngine.setTemplateResolver(resolver);
        return templateEngine;
    }
}
```

Servico:

```java
package br.com.exemplo.mail.template;

import java.util.Map;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class DynamicTemplateRenderer {

    private final TemplateEngine dynamicTemplateEngine;
    private final MailTemplateRepository repository;

    public DynamicTemplateRenderer(
            TemplateEngine dynamicTemplateEngine,
            MailTemplateRepository repository
    ) {
        this.dynamicTemplateEngine = dynamicTemplateEngine;
        this.repository = repository;
    }

    public RenderedMail render(String codigoTemplate, Map<String, Object> variaveis) {
        MailTemplate template = repository.findByCodigoAndAtivoTrue(codigoTemplate)
                .orElseThrow(() -> new IllegalArgumentException("Template não encontrado"));

        Context context = new Context();
        context.setVariables(variaveis);

        String html = dynamicTemplateEngine.process(template.getConteudoHtml(), context);
        String assunto = dynamicTemplateEngine.process(template.getAssunto(), context);

        return new RenderedMail(assunto, html);
    }
}
```

```java
package br.com.exemplo.mail.template;

public record RenderedMail(String assunto, String html) {
}
```

Exemplo de template dinâmico salvo em banco:

```html
<html>
<body>
    <h2>Ola, <span th:text="${nomeCliente}">Cliente</span></h2>
    <p>Seu pedido <strong th:text="${numeroPedido}">123</strong> foi faturado.</p>
    <p>Valor total: <strong th:text="${valorTotal}">0,00</strong></p>
</body>
</html>
```

Cuidados com templates dinâmicos:

- validar a sintaxe antes de publicar;
- manter histórico e versão do template;
- limitar quem pode editar;
- higienizar HTML quando houver edição rica;
- considerar fallback para template estatico.

Uma estratégia muito pratica é manter templates técnicos como estáticos e comunicações mais voláteis como dinâmicas.

## 2. Jobs com Quartz

### 2.1. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

Quando houver persistência funcional das definições:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

### 2.2. Configuração padrão

```yaml
spring:
  quartz:
    job-store-type: jdbc
    jdbc:
      initialize-schema: never
    properties:
      org:
        quartz:
          scheduler:
            instanceName: AppScheduler
            instanceId: AUTO
          jobStore:
            isClustered: true
            misfireThreshold: 60000
          threadPool:
            threadCount: 10
```

Notas importantes:

- `job-store-type: jdbc` persiste jobs, triggers e `JobDataMap` no banco;
- o ideal em produção é criar as tabelas do Quartz por migration;
- `isClustered: true` é importante quando há mais de uma instância da aplicação.

### 2.3. Exemplo de job

```java
package br.com.exemplo.quartz;

import java.time.LocalDateTime;
import org.quartz.DisallowConcurrentExecution;
import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@DisallowConcurrentExecution
public class EnvioRelatorioJob implements Job {

    private static final Logger log = LoggerFactory.getLogger(EnvioRelatorioJob.class);

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        JobDataMap dataMap = context.getMergedJobDataMap();
        String codigoEmpresa = dataMap.getString("codigoEmpresa");
        String emailDestino = dataMap.getString("emailDestino");

        log.info("Executando job de relatorio. empresa={} destino={} dataHora={}",
                codigoEmpresa, emailDestino, LocalDateTime.now());

        // Chamada real do servico de negocio.
    }
}
```

O uso de `@DisallowConcurrentExecution` evita que duas execuções do mesmo job rodem ao mesmo tempo.

### 2.4. Registro padrão de `JobDetail` e `Trigger`

```java
package br.com.exemplo.quartz;

import org.quartz.CronScheduleBuilder;
import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class QuartzConfig {

    @Bean
    public JobDetail envioRelatorioJobDetail() {
        return JobBuilder.newJob(EnvioRelatorioJob.class)
                .withIdentity("envioRelatorioJob", "relatorios")
                .storeDurably()
                .usingJobData("codigoEmpresa", "MATRIZ")
                .usingJobData("emailDestino", "relatorios@empresa.com.br")
                .build();
    }

    @Bean
    public Trigger envioRelatorioTrigger(JobDetail envioRelatorioJobDetail) {
        return TriggerBuilder.newTrigger()
                .forJob(envioRelatorioJobDetail)
                .withIdentity("envioRelatorioTrigger", "relatorios")
                .withSchedule(CronScheduleBuilder.cronSchedule("0 0/10 * * * ?"))
                .build();
    }
}
```

Essa é a forma padrão quando a agenda faz parte fixa da aplicação.

### 2.5. Gerenciamento dinâmico de jobs e triggers com dados em banco

Quando horários, parâmetros e ativação precisam mudar sem deploy, uma estratégia muito comum é manter uma tabela funcional da aplicação e sincronizar isso com o Quartz.

Entidade funcional:

```java
package br.com.exemplo.quartz.admin;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "scheduled_job_definition")
public class ScheduledJobDefinition {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String codigo;

    @Column(nullable = false)
    private String jobClass;

    @Column(nullable = false)
    private String jobGroup;

    @Column(nullable = false)
    private String triggerGroup;

    @Column(nullable = false)
    private String cronExpression;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String payloadJson;

    @Column(nullable = false)
    private boolean ativo = true;

    // Getters e setters omitidos por brevidade.
}
```

Serviço administrativo:

```java
package br.com.exemplo.quartz.admin;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.Map;
import org.quartz.CronScheduleBuilder;
import org.quartz.JobBuilder;
import org.quartz.JobDataMap;
import org.quartz.JobDetail;
import org.quartz.JobKey;
import org.quartz.Scheduler;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.quartz.TriggerKey;
import org.springframework.stereotype.Service;

@Service
public class QuartzAdminService {

    private final Scheduler scheduler;
    private final ObjectMapper objectMapper;

    public QuartzAdminService(Scheduler scheduler, ObjectMapper objectMapper) {
        this.scheduler = scheduler;
        this.objectMapper = objectMapper;
    }

    public void registrar(ScheduledJobDefinition definition) throws Exception {
        Class<?> rawType = Class.forName(definition.getJobClass());

        @SuppressWarnings("unchecked")
        Class<? extends org.quartz.Job> jobType = (Class<? extends org.quartz.Job>) rawType;

        Map<String, Object> payload = objectMapper.readValue(
                definition.getPayloadJson(),
                new TypeReference<>() {}
        );

        JobDataMap dataMap = new JobDataMap(payload);

        JobDetail jobDetail = JobBuilder.newJob(jobType)
                .withIdentity(definition.getCodigo(), definition.getJobGroup())
                .usingJobData(dataMap)
                .storeDurably()
                .build();

        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity(definition.getCodigo(), definition.getTriggerGroup())
                .forJob(jobDetail)
                .usingJobData(dataMap)
                .withSchedule(CronScheduleBuilder.cronSchedule(definition.getCronExpression()))
                .build();

        scheduler.scheduleJob(jobDetail, trigger);
    }

    public void atualizarAgenda(ScheduledJobDefinition definition) throws Exception {
        TriggerKey triggerKey = TriggerKey.triggerKey(
                definition.getCodigo(),
                definition.getTriggerGroup()
        );

        Trigger novoTrigger = TriggerBuilder.newTrigger()
                .withIdentity(triggerKey)
                .forJob(definition.getCodigo(), definition.getJobGroup())
                .withSchedule(CronScheduleBuilder.cronSchedule(definition.getCronExpression()))
                .build();

        scheduler.rescheduleJob(triggerKey, novoTrigger);
    }

    public void remover(String codigo, String jobGroup) throws Exception {
        scheduler.deleteJob(JobKey.jobKey(codigo, jobGroup));
    }
}
```

Na prática, os dados dinâmicos podem ficar:

- no `JobDataMap`, quando forem pequenos e de uso direto pelo Quartz;
- em uma tabela funcional da aplicação, quando exigirem auditoria, filtros e histórico;
- nas duas camadas, usando a tabela funcional como fonte principal e o Quartz como runtime.

Campos comuns em base funcional:

- expressão cron;
- identificador do tenant;
- parâmetros de negocio;
- usuário que criou ou alterou;
- flag de ativo/inativo;
- última e próxima execução;
- resultado da última tentativa.

### 2.6. Ativar e desativar jobs

Pausa e retomada diretamente no Quartz:

```java
package br.com.exemplo.quartz.admin;

import org.quartz.JobKey;
import org.quartz.Scheduler;
import org.springframework.stereotype.Service;

@Service
public class QuartzToggleService {

    private final Scheduler scheduler;

    public QuartzToggleService(Scheduler scheduler) {
        this.scheduler = scheduler;
    }

    public void desativar(String nome, String grupo) throws Exception {
        scheduler.pauseJob(JobKey.jobKey(nome, grupo));
    }

    public void ativar(String nome, String grupo) throws Exception {
        scheduler.resumeJob(JobKey.jobKey(nome, grupo));
    }
}
```

Também é comum manter `ativo = false` na tabela funcional e sincronizar isso com o scheduler ao salvar ou inicializar a aplicação.

### 2.7. Execução manual de jobs

```java
package br.com.exemplo.quartz.admin;

import org.quartz.JobDataMap;
import org.quartz.JobKey;
import org.quartz.Scheduler;
import org.springframework.stereotype.Service;

@Service
public class QuartzManualExecutionService {

    private final Scheduler scheduler;

    public QuartzManualExecutionService(Scheduler scheduler) {
        this.scheduler = scheduler;
    }

    public void executarAgora(String nome, String grupo) throws Exception {
        scheduler.triggerJob(JobKey.jobKey(nome, grupo));
    }

    public void executarAgoraComParametros(String nome, String grupo, JobDataMap parametros)
            throws Exception {
        scheduler.triggerJob(JobKey.jobKey(nome, grupo), parametros);
    }
}
```

Casos de uso:

- reprocessamento;
- execução sob demanda via painel administrativo;
- teste de agenda em homologação;
- reenvio de relatórios e integrações.

### 2.8. Endpoint administrativo de exemplo

```java
package br.com.exemplo.quartz.api;

import br.com.exemplo.quartz.admin.QuartzManualExecutionService;
import br.com.exemplo.quartz.admin.QuartzToggleService;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/jobs")
public class QuartzAdminController {

    private final QuartzToggleService toggleService;
    private final QuartzManualExecutionService manualExecutionService;

    public QuartzAdminController(
            QuartzToggleService toggleService,
            QuartzManualExecutionService manualExecutionService
    ) {
        this.toggleService = toggleService;
        this.manualExecutionService = manualExecutionService;
    }

    @PostMapping("/{grupo}/{nome}/pause")
    public void pausar(@PathVariable String grupo, @PathVariable String nome) throws Exception {
        toggleService.desativar(nome, grupo);
    }

    @PostMapping("/{grupo}/{nome}/resume")
    public void retomar(@PathVariable String grupo, @PathVariable String nome) throws Exception {
        toggleService.ativar(nome, grupo);
    }

    @PostMapping("/{grupo}/{nome}/run")
    public void executar(@PathVariable String grupo, @PathVariable String nome) throws Exception {
        manualExecutionService.executarAgora(nome, grupo);
    }
}
```

Boas práticas com Quartz:

- use `@DisallowConcurrentExecution` quando houver risco de concorrência;
- mantenha o job fino e delegue regra de negocio para services;
- trate `misfire` explicitamente em agendas criticas;
- monitore falhas, tempo médio e quantidade de reprocessamentos;
- evite objetos grandes no `JobDataMap`.

## 3. JMS com Artemis

### 3.1. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jms</artifactId>
</dependency>
```

### 3.2. Configuração básica

Exemplo com broker externo:

```yaml
spring:
  artemis:
    mode: native
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
  jms:
    listener:
      acknowledge-mode: auto
      concurrency: 2-10
```

Para desenvolvimento, também e possível usar broker embarcado se o projeto incluir as dependências necessárias.

#### Exemplo para Artemis embarcado em ambiente dev

Segundo a documentação oficial do Spring Boot, para usar Artemis embarcado a aplicação precisa ter o starter JMS do Artemis e também a dependência do broker embarcado no classpath.

Dependência adicional:

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-jakarta-server</artifactId>
</dependency>
```

Configuração recomendada para desenvolvimento:

```yaml
spring:
  artemis:
    mode: embedded
    embedded:
      enabled: true
      persistent: false
      queues: queue.pedidos,queue.respostas,queue.consulta.credito
      topics: topic.notificacoes
  jms:
    cache:
      session-cache-size: 5
```

O que essa configuração faz:

- `mode: embedded` exige broker embarcado;
- `persistent: false` evita gravação em disco, o que costuma ser adequado para dev;
- `queues` e `topics` criam destinos básicos com opções padrão;
- `session-cache-size` mantem o uso simples e eficiente durante desenvolvimento.

Se houver necessidade de configuração mais avançada das filas e tópicos, o próprio Spring Boot recomenda declarar beans do tipo `JMSQueueConfiguration` e `TopicConfiguration`.

Exemplo com fila durável e tópico predefinido:

```java
package br.com.exemplo.jms.config;

import org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration;
import org.apache.activemq.artemis.jms.server.config.TopicConfiguration;
import org.apache.activemq.artemis.jms.server.config.impl.JMSQueueConfigurationImpl;
import org.apache.activemq.artemis.jms.server.config.impl.TopicConfigurationImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ArtemisEmbeddedDevConfig {

    @Bean
    public JMSQueueConfiguration pedidosQueueConfiguration() {
        return new JMSQueueConfigurationImpl()
                .setName("queue.pedidos")
                .setBindings("queue/pedidos")
                .setDurable(true);
    }

    @Bean
    public TopicConfiguration notificacoesTopicConfiguration() {
        return new TopicConfigurationImpl()
                .setName("topic.notificacoes")
                .setBindings("topic/notificacoes");
    }
}
```

Para ambiente dev, esse modelo embarcado é útil quando:

- a equipe quer subir tudo com a própria aplicação;
- não compensa manter um broker externo local;
- os testes manuais precisam de filas e tópicos já disponíveis;
- o objetivo é reduzir dependência de infraestrutura no setup local.

Para homologação ou produção, normalmente vale migrar para um broker externo dedicado.

### 3.3. Filas duráveis

Em ActiveMQ Artemis, uma fila durável é uma fila que sobrevive a restart ou falha do broker. Entretanto, para que a mensagem em si sobreviva, ela também precisa ser enviada como mensagem durável ou persistente.

Na prática:

- fila durável protege a existência da fila;
- mensagem durável protege o conteúdo da mensagem;
- para persistência completa, os dois pontos precisam estar alinhados.

#### O que considerar em Artemis

- filas auto-criadas pelo broker são duráveis, não temporárias e não transientes;
- mesmo assim, elas podem ser removidas automaticamente se as políticas de `auto-delete-queues` estiverem habilitadas;
- para filas de negócio, normalmente vale predefinir a fila ou desabilitar auto-delete para aquele conjunto de endereços.

#### Exemplo para broker embarcado no Spring Boot

Quando a aplicação usa Artemis embarcado, o Spring Boot permite declarar beans do tipo `JMSQueueConfiguration` para configurações avançadas de fila.

```java
package br.com.exemplo.jms.config;

import org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration;
import org.apache.activemq.artemis.jms.server.config.impl.JMSQueueConfigurationImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ArtemisEmbeddedQueuesConfig {

    @Bean
    public JMSQueueConfiguration pedidosQueueConfiguration() {
        return new JMSQueueConfigurationImpl()
                .setName("queue.pedidos")
                .setBindings("queue/pedidos")
                .setDurable(true);
    }
}
```

Esse modelo é útil quando a própria aplicação sobe o broker e precisa garantir que determinadas filas de negócio existam desde a inicialização.

#### Exemplo de configuração no `broker.xml`

Para broker externo ou administrado separadamente, uma configuração típica é:

- em uma instalação standalone do Artemis, o arquivo fica em `<broker-instance>/etc/broker.xml`;
- ou seja, ele não deve ser colocado em `${ARTEMIS_HOME}`, mas no diretório da instancia criada do broker;
- em Docker, o caminho padrão da imagem oficial e `/var/lib/artemis-instance/etc/broker.xml`;
- quando for usado mecanismo de sobrescrita da imagem oficial, os arquivos customizados podem ser colocados em `/var/lib/artemis-instance/etc-override`, de onde serão copiados para `etc`.

Exemplo:

```xml
<addresses>
   <address name="queue.pedidos">
      <anycast>
         <queue name="queue.pedidos">
            <durable>true</durable>
         </queue>
      </anycast>
   </address>
</addresses>
```

Se a fila for de negócio e não puder desaparecer automaticamente, uma configuração complementar muito comum é:

```xml
<address-settings>
   <address-setting match="queue.#">
      <auto-create-queues>true</auto-create-queues>
      <auto-delete-queues>false</auto-delete-queues>
   </address-setting>
</address-settings>
```

#### Garantindo envio persistente da mensagem

Se a fila é durável, mas a mensagem for enviada como não persistente, ela não estará protegida em caso de reinicio ou falha do broker.

Exemplo com `JmsClient`:

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class PedidoDuravelProducer {

    private final JmsClient jmsClient;

    public PedidoDuravelProducer(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public void enviar(PedidoMessage pedido) {
        jmsClient.destination(Destinations.FILA_PEDIDOS)
                .withDeliveryPersistent(true)
                .send(pedido);
    }
}
```

Resumo prático:

- fila durável + mensagem persistente: indicado para pedidos, pagamentos, integrações críticas;
- fila durável + mensagem não persistente: a fila continua existindo, mas a mensagem pode se perder;
- fila temporária ou não durável: mais adequada para cenários efêmeros, testes ou respostas temporárias.

### 3.4. Uso de `pooled-jms`

Por padrão, o Spring Boot envolve a `ConnectionFactory` nativa com uma `CachingConnectionFactory`, configuravel por `spring.jms.*`. Se for preferível usar pooling nativo de JMS, o Spring Boot suporta isso com a dependência `org.messaginghub:pooled-jms` e propriedades `spring.artemis.pool.*`.

Segundo a documentação do Spring Boot, o caminho padrão para Artemis e:

- cache simples via `spring.jms.cache.session-cache-size`;
- pooling nativo via `spring.artemis.pool.enabled=true`.

Segundo a documentação do `pooled-jms`, o `JmsPoolConnectionFactory` faz pooling de `Connection`, `Session` e `MessageProducer`, reduzindo o custo de criação repetida desses recursos.

#### Dependência

```xml
<dependency>
    <groupId>org.messaginghub</groupId>
    <artifactId>pooled-jms</artifactId>
</dependency>
```

#### Configuração com Spring Boot

```yaml
spring:
  artemis:
    mode: native
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
    pool:
      enabled: true
      max-connections: 20
      max-sessions-per-connection: 50
      idle-timeout: 30s
```

Em geral:

- `max-connections` controla quantas conexões físicas podem ser mantidas;
- `max-sessions-per-connection` ajuda em cenários com muitas sessões simultâneas;
- `idle-timeout` evita manter recursos ociosos por tempo excessivo.

#### Quando `pooled-jms` e útil

Ele costuma ser especialmente útil quando há:

- muitos envios JMS de curta duração;
- alto volume de produtores concorrentes;
- request-reply com criação frequente de sessões;
- uso intensivo de `JmsTemplate` ou `JmsClient` em chamadas síncronas ou bursts.

Exemplos típicos:

- API HTTP que publica muitas mensagens por requisição;
- serviço integrador que envia mensagens para varias filas;
- aplicação com alto throughput de produção e baixa necessidade de listeners customizados.

#### Quando avaliar com mais cuidado

Para listeners de longa duração, a vantagem pode ser menor. A própria documentação do Spring Boot destaca que, na maioria dos cenários, os message listener containers devem trabalhar contra a `ConnectionFactory` nativa, pois assim cada listener container mantem sua própria conexão e responsabilidade de recuperação local.

Em outras palavras, `pooled-jms` tende a ser mais interessante para o lado produtor do que como principal preocupação para consumidores de longa vida.

#### Exemplo de uso com `JmsClient`

Não há mudança no código de envio. O ganho fica na infraestrutura da `ConnectionFactory`.

```java
package br.com.exemplo.jms.notificacao;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class NotificacaoPublisherPooled {

    private final JmsClient jmsClient;

    public NotificacaoPublisherPooled(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public void publicar(NotificacaoEvent event) {
        jmsClient.destination(Destinations.TOPICO_NOTIFICACOES)
                .send(event);
    }
}
```

Ou seja, a troca está na configuração do pool, não no estilo do producer.

### 3.5. Padronizando serialização JSON

Em projetos reais, é recomendável padronizar a troca de mensagens em JSON. Assim, os exemplos com `JmsTemplate`, `JmsClient` e `@JmsListener` passam a trabalhar com DTOs de forma consistente, sem depender de `ObjectMessage`.

Uma configuração comum é registrar um `MappingJackson2MessageConverter`:

```java
package br.com.exemplo.jms.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.support.converter.MappingJackson2MessageConverter;
import org.springframework.jms.support.converter.MessageType;

@Configuration
public class JmsMessageConverterConfig {

    @Bean
    public MappingJackson2MessageConverter jacksonJmsMessageConverter() {
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }
}
```

Com essa configuração:

- os DTOs podem ser enviados como JSON em `TextMessage`;
- o header `_type` ajuda na deserialização do tipo;
- o `JmsTemplate` auto-configurado passa a usar esse converter;
- o `JmsClient` auto-configurado também acompanha esse comportamento, pois reutiliza a infraestrutura JMS configurada pela aplicação.

### 3.6. Exemplos com `JmsClient` no Spring Boot 4

No Spring Boot 4, o `JmsClient` e autoconfigurado e pode ser injetado diretamente nos beans da aplicação. Ele oferece uma API fluente para envio, recebimento síncrono e request-reply, funcionando sobre a infraestrutura tradicional do Spring JMS.

Pontos importantes:

- o `JmsClient` é muito pratico para envio e recebimento síncrono;
- para consumo assíncrono, `@JmsListener` continua sendo a abordagem mais comum;
- o cliente autoconfigurado reutiliza a configuração do `JmsTemplate`;
- quando a aplicação mistura fila e tópico, pode ser útil ter clientes diferentes para cada domínio.

#### Exemplo simples de envio

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class PedidoProducerComJmsClient {

    private final JmsClient jmsClient;

    public PedidoProducerComJmsClient(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public void enviar(PedidoMessage pedido) {
        jmsClient.destination(Destinations.FILA_PEDIDOS)
                .send(pedido);
    }
}
```

#### Exemplo com headers e QoS

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import java.util.Map;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class PedidoProducerComQoS {

    private final JmsClient jmsClient;

    public PedidoProducerComQoS(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public void enviarPrioritario(PedidoMessage pedido) {
        jmsClient.destination(Destinations.FILA_PEDIDOS)
                .withDeliveryPersistent(true)
                .withPriority(9)
                .withTimeToLive(30000)
                .send(pedido, Map.of(
                        "tipo", "PEDIDO_CRIADO",
                        "prioridade", "ALTA"
                ));
    }
}
```

#### Recebimento síncrono com timeout

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import java.util.Optional;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class PedidoPollingService {

    private final JmsClient jmsClient;

    public PedidoPollingService(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public Optional<PedidoMessage> receberProximo() {
        return jmsClient.destination(Destinations.FILA_PEDIDOS)
                .withReceiveTimeout(2000)
                .receive(PedidoMessage.class);
    }

    public Optional<PedidoMessage> receberSomenteAltaPrioridade() {
        return jmsClient.destination(Destinations.FILA_PEDIDOS)
                .withReceiveTimeout(2000)
                .receive("prioridade = 'ALTA'", PedidoMessage.class);
    }
}
```

Esse tipo de consumo síncrono costuma ser útil em rotinas administrativas, testes de integração ou fluxos request-reply.

#### Request-reply com `sendAndReceive`

```java
package br.com.exemplo.jms.requestreply;

import java.util.Map;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class CreditoClientComJmsClient {

    private final JmsClient jmsClient;

    public CreditoClientComJmsClient(JmsClient jmsClient) {
        this.jmsClient = jmsClient;
    }

    public ConsultaCreditoResponse consultar(String documento) {
        return jmsClient.destination("queue.consulta.credito")
                .withReceiveTimeout(5000)
                .sendAndReceive(
                        new ConsultaCreditoRequest(documento),
                        Map.of("origem", "cadastro-cliente"),
                        ConsultaCreditoResponse.class
                )
                .orElseThrow(() -> new IllegalStateException("Nenhuma resposta recebida"));
    }
}
```

Esse formato simplifica bastante o fluxo de ida e volta, preservando o estilo do Spring Messaging.

#### Criando um `JmsClient` dedicado para tópicos

Quando o projeto trabalha com fila e pub-sub na mesma aplicação, o cliente autoconfigurado costuma ser suficiente para filas. Para tópicos, uma abordagem clara e criar um `JmsClient` baseado em um `JmsTemplate` com `pubSubDomain = true`.

Se existirem dois beans do tipo `JmsClient`, use `@Qualifier` na injeção para deixar explicito qual cliente deve ser usado em cada fluxo.

```java
package br.com.exemplo.jms.config;

import jakarta.jms.ConnectionFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.core.JmsClient;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class JmsClientConfig {

    @Bean
    public JmsClient topicJmsClient(ConnectionFactory connectionFactory) {
        JmsTemplate topicTemplate = new JmsTemplate(connectionFactory);
        topicTemplate.setPubSubDomain(true);
        return JmsClient.create(topicTemplate);
    }
}
```

Uso:

```java
package br.com.exemplo.jms.notificacao;

import br.com.exemplo.jms.Destinations;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jms.core.JmsClient;
import org.springframework.stereotype.Service;

@Service
public class NotificacaoPublisherComJmsClient {

    private final JmsClient topicJmsClient;

    public NotificacaoPublisherComJmsClient(@Qualifier("topicJmsClient") JmsClient topicJmsClient) {
        this.topicJmsClient = topicJmsClient;
    }

    public void publicar(NotificacaoEvent event) {
        topicJmsClient.destination(Destinations.TOPICO_NOTIFICACOES)
                .send(event);
    }
}
```

### 3.7. Suportando fila e pub-sub na mesma aplicação

A mesma aplicação pode trabalhar com:

- filas para processamento ponto a ponto;
- tópicos para eventos e notificações para vários consumidores.

Uma configuração clara e separar `JmsTemplate` e `JmsListenerContainerFactory` para cada modelo.

Destinations centralizadas:

```java
package br.com.exemplo.jms;

public final class Destinations {

    public static final String FILA_PEDIDOS = "queue.pedidos";
    public static final String FILA_RESPOSTAS = "queue.respostas";
    public static final String TOPICO_NOTIFICACOES = "topic.notificacoes";

    private Destinations() {
    }
}
```

Configuração:

```java
package br.com.exemplo.jms.config;

import jakarta.jms.ConnectionFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class JmsConfig {

    @Bean
    public DefaultJmsListenerContainerFactory queueListenerFactory(ConnectionFactory connectionFactory) {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setPubSubDomain(false);
        factory.setConcurrency("2-10");
        return factory;
    }

    @Bean
    public DefaultJmsListenerContainerFactory topicListenerFactory(ConnectionFactory connectionFactory) {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setPubSubDomain(true);
        factory.setSubscriptionDurable(true);
        factory.setClientId("app-notificacoes");
        return factory;
    }

    @Bean
    public JmsTemplate queueJmsTemplate(ConnectionFactory connectionFactory) {
        JmsTemplate template = new JmsTemplate(connectionFactory);
        template.setPubSubDomain(false);
        return template;
    }

    @Bean
    public JmsTemplate topicJmsTemplate(ConnectionFactory connectionFactory) {
        JmsTemplate template = new JmsTemplate(connectionFactory);
        template.setPubSubDomain(true);
        return template;
    }
}
```

### 3.8. Exemplo com filas

Mensagem:

```java
package br.com.exemplo.jms.pedido;

public record PedidoMessage(Long pedidoId, String cliente, Double valor) {
}
```

Producer:

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

@Service
public class PedidoProducer {

    private final JmsTemplate queueJmsTemplate;

    public PedidoProducer(JmsTemplate queueJmsTemplate) {
        this.queueJmsTemplate = queueJmsTemplate;
    }

    public void enviar(PedidoMessage pedido) {
        queueJmsTemplate.convertAndSend(Destinations.FILA_PEDIDOS, pedido);
    }
}
```

Consumer:

```java
package br.com.exemplo.jms.pedido;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoConsumer {

    @JmsListener(destination = Destinations.FILA_PEDIDOS, containerFactory = "queueListenerFactory")
    public void consumir(PedidoMessage pedido) {
        System.out.println("Processando pedido " + pedido.pedidoId());
    }
}
```

### 3.9. Exemplo com tópicos

Evento:

```java
package br.com.exemplo.jms.notificacao;

public record NotificacaoEvent(String tipo, String mensagem) {
}
```

Publisher:

```java
package br.com.exemplo.jms.notificacao;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

@Service
public class NotificacaoPublisher {

    private final JmsTemplate topicJmsTemplate;

    public NotificacaoPublisher(JmsTemplate topicJmsTemplate) {
        this.topicJmsTemplate = topicJmsTemplate;
    }

    public void publicar(NotificacaoEvent event) {
        topicJmsTemplate.convertAndSend(Destinations.TOPICO_NOTIFICACOES, event);
    }
}
```

Subscriber:

```java
package br.com.exemplo.jms.notificacao;

import br.com.exemplo.jms.Destinations;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class NotificacaoSubscriber {

    @JmsListener(destination = Destinations.TOPICO_NOTIFICACOES, containerFactory = "topicListenerFactory")
    public void receber(NotificacaoEvent event) {
        System.out.println("Evento recebido: " + event.tipo() + " - " + event.mensagem());
    }
}
```

### 3.10. Mensagens que necessitam de retorno

Quando o emissor envia uma mensagem e aguarda resposta, o modelo usado normalmente é request-reply.

Conceitos principais:

- `JMSReplyTo` indica o destino da resposta;
- `JMSCorrelationID` permite correlacionar requisicao e retorno;
- `JmsTemplate.convertSendAndReceive(...)` simplifica esse fluxo.

Requisicao:

```java
package br.com.exemplo.jms.requestreply;

public record ConsultaCreditoRequest(String documento) {
}
```

Resposta:

```java
package br.com.exemplo.jms.requestreply;

public record ConsultaCreditoResponse(String documento, String status, Double limite) {
}
```

Cliente:

```java
package br.com.exemplo.jms.requestreply;

import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

@Service
public class CreditoClient {

    private final JmsTemplate queueJmsTemplate;

    public CreditoClient(JmsTemplate queueJmsTemplate) {
        this.queueJmsTemplate = queueJmsTemplate;
        this.queueJmsTemplate.setReceiveTimeout(5000);
    }

    public ConsultaCreditoResponse consultar(String documento) {
        return (ConsultaCreditoResponse) queueJmsTemplate.convertSendAndReceive(
                "queue.consulta.credito",
                new ConsultaCreditoRequest(documento)
        );
    }
}
```

Consumidor com resposta:

```java
package br.com.exemplo.jms.requestreply;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Component;

@Component
public class CreditoRequestReplyConsumer {

    @SendTo
    @JmsListener(destination = "queue.consulta.credito", containerFactory = "queueListenerFactory")
    public ConsultaCreditoResponse processar(ConsultaCreditoRequest request) {
        return new ConsultaCreditoResponse(request.documento(), "APROVADO", 2500.0);
    }
}
```

Nesse modelo:

- o produtor envia a requisição;
- o Spring usa o destino de resposta adequado;
- a resposta volta ao emissor;
- o `JMSCorrelationID` ajuda a amarrar ida e volta.

### 3.11. Exemplo com correlação explicita

```java
package br.com.exemplo.jms.requestreply;

import java.util.UUID;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

@Service
public class CreditoClientComCorrelacao {

    private final JmsTemplate queueJmsTemplate;

    public CreditoClientComCorrelacao(JmsTemplate queueJmsTemplate) {
        this.queueJmsTemplate = queueJmsTemplate;
    }

    public void enviar(String documento) {
        String correlationId = UUID.randomUUID().toString();

        queueJmsTemplate.convertAndSend("queue.consulta.credito", new ConsultaCreditoRequest(documento), message -> {
            message.setJMSCorrelationID(correlationId);
            return message;
        });
    }
}
```

Esse formato e útil quando:

- há integração com sistemas legados;
- a resposta não será imediata;
- o identificador de correlação precisa ser persistido.

### 3.12. Boas práticas com Artemis e JMS

- usar nomes claros para destinations, como `queue.` e `topic.`;
- para filas criticas, alinhar fila durável com mensagens persistentes;
- evitar `auto-delete` em filas de negócio que precisam sobreviver ao ciclo de vida do consumidor;
- definir politica de redelivery e dead letter queue;
- padronizar a serialização em JSON com `MessageConverter`;
- evitar payloads muito grandes;
- monitorar filas, tempos de consumo e erros;
- usar correlation ID em fluxos críticos;
- avaliar `pooled-jms` principalmente quando houver muitos producers e criação frequente de sessões;
- documentar quais consumers são de fila e quais são de topico.

### 3.13. `@Transactional` e `@Async` em listeners JMS

#### O que já é implicitamente assíncrono

O `DefaultMessageListenerContainer` (DMLC) — configurado automaticamente pelo Spring Boot quando há um broker JMS — gerencia internamente seu próprio pool de threads. O `@JmsListener` roda em uma thread do DMLC, completamente separada da thread que publicou a mensagem:

```
Thread da requisição HTTP          Thread do DMLC (pool interno)
─────────────────────────          ──────────────────────────────
save() → COMMIT                    (aguarda mensagem no broker)
convertAndSend() ──────────────→   broker (Artemis)
retorna ao controller              ↓
                                   processar(event) ← @JmsListener
```

O `convertAndSend()` faz uma chamada síncrona ao broker e retorna. A partir daí, quem processa é o DMLC — em outra thread, em outro momento. Não é necessário `@Async` em lugar nenhum.

#### O problema de combinar `@Async` com `@JmsListener`

O `@Transactional` no consumer funciona porque o Spring cria um proxy que vincula a transação à thread corrente. Se `@Async` for adicionado, o método salta para outra thread — e o vínculo transacional se quebra silenciosamente:

```java
// ⚠️ Combinação problemática
@JmsListener(destination = "pedidos.criados")
@Transactional
@Async  // muda de thread → @Transactional perde o contexto
public void processar(PedidoCriadoEvent event) {
    // transação pode não estar ativa aqui
}
```

Além disso, se o método lançar exceção numa thread `@Async`, o DMLC não a captura — o ack/nack da mensagem fica em estado indefinido.

#### Quando `@Async` faz sentido no contexto JMS

O único cenário legítimo é desacoplar a chamada ao broker do fluxo principal da requisição — por exemplo, quando o `convertAndSend()` está num `@TransactionalEventListener` e não se quer que uma lentidão do broker atrase o response HTTP:

```java
// @Async aqui é no SENDER, não no listener JMS
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
@Async  // libera a thread HTTP imediatamente após o commit
public void onPedidoCriado(PedidoCriadoEvent event) {
    jmsClient.convertAndSend("pedidos.criados", event);
}
```

Esse padrão exige `@EnableAsync` na aplicação e um `TaskExecutor` configurado. O tradeoff é a perda de propagação de exceções: se o broker estiver fora do ar, a falha não aparece no fluxo normal da requisição.

#### Controlando o paralelismo no consumer via DMLC

Quando houver necessidade de mais paralelismo no consumer, o caminho correto é configurar o `DefaultJmsListenerContainerFactory` diretamente — é mais previsível e seguro do que `@Async`:

```java
@Bean
public DefaultJmsListenerContainerFactory jmsListenerContainerFactory(ConnectionFactory cf) {
    DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
    factory.setConnectionFactory(cf);
    factory.setConcurrency("3-10");  // mínimo 3, máximo 10 threads consumers
    factory.setSessionTransacted(true);
    return factory;
}
```

#### Resumo

| Contexto | `@Async` necessário? | Motivo |
|---|---|---|
| `@JmsListener` (consumer) | Não — e evite | DMLC já tem thread pool próprio; quebra `@Transactional` |
| `convertAndSend()` (sender) | Opcional | Só para desacoplar o envio da thread da requisição |
| Outbox relay (`@Scheduled`) | Não | `@Scheduled` já roda em thread separada |

## 4. Mensageria com Kafka e RabbitMQ

A seção anterior cobre mensageria embarcada com JMS e Artemis, adequada para cenários onde o broker faz parte da própria aplicação ou do mesmo cluster. Para cenários distribuídos com maior throughput, desacoplamento entre serviços independentes e retenção de eventos, as alternativas mais usadas no ecossistema Java são RabbitMQ e Apache Kafka.

### 4.1. Quando usar mensageria

| Cenário | Benefício |
|---------|-----------|
| Processamento assíncrono | Resposta imediata ao usuário; trabalho pesado em background |
| Desacoplamento de serviços | Produtor e consumidor evoluem independentemente |
| Picos de carga | Fila absorve a demanda e processa no ritmo do consumidor |
| Event-driven architecture | Serviços reagem a eventos de domínio |
| Garantia de entrega | Mensagens persistidas até serem consumidas com sucesso |

### 4.2. RabbitMQ vs Kafka

| Aspecto | RabbitMQ | Kafka |
|---------|----------|-------|
| **Modelo** | Message broker (fila) | Event streaming (log distribuído) |
| **Mensagem após consumo** | Removida da fila | Retida por período configurado |
| **Ordenação** | Por fila | Por partição |
| **Throughput** | Milhares/s | Milhões/s |
| **Replay** | Não | Sim (consumer offset) |
| **Caso de uso típico** | Tarefas assíncronas, RPC, notificações | Streaming, event sourcing, analytics |
| **Complexidade operacional** | Baixa | Alta |
| **Protocolo** | AMQP | Protocolo próprio sobre TCP |

### 4.3. RabbitMQ com Spring Boot

#### Dependência

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### Configuração

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

#### Configuração de fila, exchange e binding

```java
package br.com.exemplo.mensageria.rabbit;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.amqp.core.*;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    public static final String PEDIDO_QUEUE = "pedido.criado";
    public static final String PEDIDO_DLQ = "pedido.criado.dlq";
    public static final String PEDIDO_EXCHANGE = "pedido.exchange";

    @Bean
    public Queue pedidoQueue() {
        return QueueBuilder.durable(PEDIDO_QUEUE)
                .deadLetterExchange("")
                .deadLetterRoutingKey(PEDIDO_DLQ)
                .build();
    }

    @Bean
    public Queue pedidoDlq() {
        return QueueBuilder.durable(PEDIDO_DLQ).build();
    }

    @Bean
    public TopicExchange pedidoExchange() {
        return new TopicExchange(PEDIDO_EXCHANGE);
    }

    @Bean
    public Binding binding(Queue pedidoQueue, TopicExchange pedidoExchange) {
        return BindingBuilder.bind(pedidoQueue).to(pedidoExchange).with("pedido.#");
    }

    @Bean
    public Jackson2JsonMessageConverter messageConverter(ObjectMapper mapper) {
        return new Jackson2JsonMessageConverter(mapper);
    }
}
```

A Dead Letter Queue (DLQ) recebe mensagens que falharam após as tentativas de retry, evitando que mensagens com erro bloqueiem o processamento das demais.

#### Produtor

```java
package br.com.exemplo.mensageria.rabbit;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;

@Service
public class PedidoEventPublisher {

    private final RabbitTemplate rabbitTemplate;

    public PedidoEventPublisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void publicarPedidoCriado(PedidoCriadoEvent evento) {
        rabbitTemplate.convertAndSend(
                RabbitConfig.PEDIDO_EXCHANGE,
                "pedido.criado",
                evento
        );
    }
}
```

#### Consumidor

```java
package br.com.exemplo.mensageria.rabbit;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoEventConsumer {

    private static final Logger log = LoggerFactory.getLogger(PedidoEventConsumer.class);

    private final NotificacaoService notificacaoService;

    public PedidoEventConsumer(NotificacaoService notificacaoService) {
        this.notificacaoService = notificacaoService;
    }

    @RabbitListener(queues = RabbitConfig.PEDIDO_QUEUE)
    public void processarPedidoCriado(PedidoCriadoEvent evento) {
        log.info("Pedido recebido: {}", evento.pedidoId());
        notificacaoService.enviarConfirmacao(evento);
    }
}
```

### 4.4. Kafka com Spring Boot

#### Dependência

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

#### Configuração

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: pedido-service
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "br.com.exemplo.mensageria.eventos"
```

#### Produtor

```java
package br.com.exemplo.mensageria.kafka;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class PedidoKafkaProducer {

    private final KafkaTemplate<String, PedidoCriadoEvent> kafkaTemplate;

    public PedidoKafkaProducer(KafkaTemplate<String, PedidoCriadoEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publicar(PedidoCriadoEvent evento) {
        kafkaTemplate.send("pedido.criado", evento.pedidoId(), evento);
    }
}
```

A chave da mensagem (`evento.pedidoId()`) garante que todas as mensagens do mesmo pedido sejam enviadas para a mesma partição, preservando a ordem de processamento para aquele pedido.

#### Consumidor

```java
package br.com.exemplo.mensageria.kafka;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoKafkaConsumer {

    private static final Logger log = LoggerFactory.getLogger(PedidoKafkaConsumer.class);

    @KafkaListener(topics = "pedido.criado", groupId = "notificacao-service")
    public void processar(PedidoCriadoEvent evento) {
        log.info("Evento recebido: pedido={}", evento.pedidoId());
        // processar evento
    }
}
```

No Kafka, cada consumer group recebe todas as mensagens do tópico de forma independente. Isso permite que serviços diferentes (notificação, analytics, auditoria) consumam o mesmo evento sem interferência.

```java
// Outro consumer group processa o mesmo evento independentemente
@KafkaListener(topics = "pedido.criado", groupId = "analytics-service")
public void registrarAnalytics(PedidoCriadoEvent evento) {
    // registrar métricas e analytics
}
```

### 4.5. Boas práticas

#### Idempotência no consumidor

Mensagens podem ser entregues mais de uma vez (at-least-once delivery). O consumidor deve ser idempotente — processar a mesma mensagem duas vezes não pode causar efeitos duplicados.

```java
@RabbitListener(queues = RabbitConfig.PEDIDO_QUEUE)
public void processar(PedidoCriadoEvent evento) {
    if (registroProcessamento.jaProcessou(evento.pedidoId())) {
        log.info("Evento já processado, ignorando: {}", evento.pedidoId());
        return;
    }
    // processar evento
    registroProcessamento.marcarComoProcessado(evento.pedidoId());
}
```

#### Serialização

Prefira JSON com `Jackson2JsonMessageConverter` (RabbitMQ) ou `JsonSerializer` (Kafka) para facilitar debugging e compatibilidade entre versões. Avro e Protobuf são alternativas para cenários de alta performance com schema registry.

#### Docker Compose para ambiente local

```yaml
services:
  rabbitmq:
    image: rabbitmq:4-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"

  zookeeper:
    image: confluentinc/cp-zookeeper:7.7.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.7.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

#### Monitoramento

- RabbitMQ: painel de administração em `http://localhost:15672` (guest/guest) para acompanhar filas, mensagens pendentes e consumidores conectados.
- Kafka: ferramentas como Kafka UI, AKHQ ou Conduktor para visualizar tópicos, partições, consumer groups e lag.
- Em ambos os casos, exponha métricas via Micrometer/Actuator para integração com Prometheus e Grafana.

---

## 5. Events do Spring

Os Events do Spring são um mecanismo simples para comunicação interna entre componentes da mesma aplicação. Eles ajudam a reduzir acoplamento entre services e são muito uteis quando uma ação de negócio precisa disparar reações adicionais, como auditoria, notificação ou integrações internas.

Casos comuns:

- após concluir um cadastro;
- após confirmar um pagamento;
- para auditoria interna;
- para disparar tarefas complementares sem acoplar tudo no mesmo service.

### 5.1. Quando usar Spring Events

Spring Events funcionam muito bem quando:

- produtor e consumidor estão na mesma aplicação;
- não há necessidade de broker externo;
- o objetivo principal é desacoplamento interno;
- o fluxo pode ser tratado de forma simples no mesmo processo.

Não são a melhor escolha quando:

- a mensagem precisa sobreviver a queda da aplicação;
- vários sistemas externos precisam consumir o mesmo evento;
- há necessidade de garantia de entrega entre processos;
- o caso pede mensageria distribuída, como JMS, Kafka ou AMQP.

### 5.2. Eventos padrão do Spring mais usados

O Spring Framework publica alguns eventos padrão ligados ao ciclo de vida do `ApplicationContext` e, em aplicações web baseadas em `DispatcherServlet`, também eventos ligados ao processamento de requests.

| Evento | Quando ocorre | Uso comum | Observações |
| --- | --- | --- | --- |
| `ContextRefreshedEvent` | Quando o `ApplicationContext` é inicializado ou atualizado com `refresh()` | carregar caches, aquecer recursos, validar configurações após subida | pode ocorrer mais de uma vez em contextos que suportam refresh |
| `ContextStartedEvent` | Quando o contexto recebe `start()` | iniciar componentes que dependem do ciclo de vida do contexto | menos comum em apps Spring Boot do dia a dia |
| `ContextStoppedEvent` | Quando o contexto recebe `stop()` | parar componentes de forma controlada | também é menos comum em apps Spring Boot convencionais |
| `ContextClosedEvent` | Quando o contexto é fechado com `close()` | liberar recursos, encerrar agendadores, flush final | representa encerramento do ciclo de vida do contexto |
| `RequestHandledEvent` | Quando uma requisição HTTP foi concluída | auditoria, log técnico, métricas simples por request | evento específico de aplicações web usando `DispatcherServlet` |
| `ServletRequestHandledEvent` | Variante web com mais detalhes sobre a request servida | observabilidade e auditoria com método, URL, tempo e status | herda de `RequestHandledEvent` e costuma ser mais útil em apps MVC |

Na pratica:

- `ContextRefreshedEvent` é um dos mais usados para tarefas de inicialização;
- `ContextClosedEvent` é bastante útil para shutdown controlado;
- `RequestHandledEvent` e `ServletRequestHandledEvent` ajudam em cenários web;
- `ContextStartedEvent` e `ContextStoppedEvent` aparecem mais em cenários que usam explicitamente o ciclo de vida do contexto.

Exemplo ouvindo um evento padrão do contexto:

```java
package br.com.exemplo.events;

import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class ContextRefreshedListener {

    @EventListener
    public void onContextRefreshed(ContextRefreshedEvent event) {
        System.out.println("Contexto inicializado: " + event.getApplicationContext().getId());
    }
}
```

Exemplo ouvindo evento web:

```java
package br.com.exemplo.events;

import org.springframework.stereotype.Component;
import org.springframework.web.context.support.ServletRequestHandledEvent;
import org.springframework.context.event.EventListener;

@Component
public class RequestHandledListener {

    @EventListener
    public void onRequestHandled(ServletRequestHandledEvent event) {
        System.out.println("Request concluida: " + event.getRequestUrl());
    }
}
```

### 5.3. Publicando eventos com `ApplicationEventPublisher`

Evento:

```java
package br.com.exemplo.events;

import java.math.BigDecimal;

public record PedidoFaturadoEvent(Long pedidoId, String cliente, BigDecimal valorTotal) {
}
```

Publisher dentro de um service:

```java
package br.com.exemplo.events;

import java.math.BigDecimal;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
public class PedidoService {

    private final ApplicationEventPublisher eventPublisher;

    public PedidoService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void faturar(Long pedidoId) {
        // Regra de negocio e persistencia do faturamento.

        eventPublisher.publishEvent(new PedidoFaturadoEvent(
                pedidoId,
                "Cliente XPTO",
                new BigDecimal("150.00")
        ));
    }
}
```

O Spring permite publicar tanto objetos comuns quanto classes que estendem `ApplicationEvent`. Em projetos modernos, usar `record` ou POJO simples costuma ser mais limpo.

### 5.4. Consumindo eventos com `@EventListener`

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoFaturadoListener {

    @EventListener
    public void aoFaturarPedido(PedidoFaturadoEvent event) {
        System.out.println("Gerando auditoria para o pedido " + event.pedidoId());
    }
}
```

Outro listener para o mesmo evento:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class NotificacaoPedidoListener {

    @EventListener
    public void enviarNotificacao(PedidoFaturadoEvent event) {
        System.out.println("Enviando notificacao para o cliente " + event.cliente());
    }
}
```

Com isso, o `PedidoService` não precisa conhecer nenhum desses consumidores.

Exemplo de `@EventListener` com `condition` usando SpEL:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoAltoValorEventListener {

    @EventListener(condition = "#event.valorTotal != null and #event.valorTotal.doubleValue() >= 1000")
    public void processarSomentePedidosDeMaiorValor(PedidoFaturadoEvent event) {
        System.out.println("Pedido de alto valor recebido: " + event.pedidoId());
    }
}
```

Nesse caso, o listener só e executado quando a expressao SpEL retornar `true`.

### 5.5. Eventos síncronos

Por padrão, o processamento de eventos do Spring é síncrono. Isso significa:

- o listener roda na mesma thread do publisher;
- se um listener falhar, a exceção pode afetar o fluxo principal;
- o tempo do listener impacta diretamente o tempo da operação original.

Esse comportamento é adequado para:

- regras internas pequenas;
- auditoria simples;
- validações complementares;
- fluxos que precisam participar da mesma transação logica.

### 5.6. Eventos assíncronos com `@Async`

Quando o processamento não deve bloquear o fluxo principal, pode-se combinar eventos com execução assíncrona.

Configuração:

```java
package br.com.exemplo.events;

import java.util.concurrent.Executor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class AsyncEventsConfig {

    @Bean(name = "eventsExecutor")
    public Executor eventsExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4);
        executor.setMaxPoolSize(8);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("events-");
        executor.initialize();
        return executor;
    }
}
```

Listener assíncrono:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Component
public class EmailPedidoListener {

    @Async("eventsExecutor")
    @EventListener
    public void processar(PedidoFaturadoEvent event) {
        System.out.println("Enviando e-mail do pedido " + event.pedidoId());
    }
}
```

Cuidados:

- exceções assíncronas não sobem naturalmente para o publisher;
- é importante monitorar falhas e tempo de execução;
- para tarefas críticas, pode ser melhor usar fila ou broker.

### 5.7. Eventos transacionais com `@TransactionalEventListener`

Quando o evento depende do sucesso da transação, a melhor opção costuma ser `@TransactionalEventListener`.

Exemplo:

```java
package br.com.exemplo.events;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.context.ApplicationEventPublisher;

@Service
public class ClienteService {

    private final ClienteRepository clienteRepository;
    private final ApplicationEventPublisher eventPublisher;

    public ClienteService(
            ClienteRepository clienteRepository,
            ApplicationEventPublisher eventPublisher
    ) {
        this.clienteRepository = clienteRepository;
        this.eventPublisher = eventPublisher;
    }

    @Transactional
    public void cadastrar(Cliente cliente) {
        clienteRepository.save(cliente);
        eventPublisher.publishEvent(new ClienteCadastradoEvent(cliente.getId(), cliente.getEmail()));
    }
}
```

Evento:

```java
package br.com.exemplo.events;

public record ClienteCadastradoEvent(Long clienteId, String email) {
}
```

Listener transacional:

```java
package br.com.exemplo.events;

import org.springframework.stereotype.Component;
import org.springframework.transaction.event.TransactionPhase;
import org.springframework.transaction.event.TransactionalEventListener;

@Component
public class ClienteCadastradoListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void aposCommit(ClienteCadastradoEvent event) {
        System.out.println("Enviando boas-vindas para " + event.email());
    }
}
```

Fases mais comuns:

- `AFTER_COMMIT`: executa apenas se a transação confirmar;
- `AFTER_ROLLBACK`: executa se houver rollback;
- `AFTER_COMPLETION`: executa ao final, independentemente do resultado;
- `BEFORE_COMMIT`: executa antes do commit.

Na maioria dos casos de integração ou notificação, `AFTER_COMMIT` é a opção mais segura.

### 5.8. Ordenação e condições

Se houver vários listeners para o mesmo evento, é possível controlar ordem:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
public class AuditoriaPrioritariaListener {

    @Order(1)
    @EventListener
    public void processar(PedidoFaturadoEvent event) {
        System.out.println("Auditoria executada antes dos demais listeners");
    }
}
```

Também é possível usar condições:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoAltoValorListener {

    @EventListener(condition = "#event.valorTotal.doubleValue() >= 1000")
    public void processarSomenteGrandesPedidos(PedidoFaturadoEvent event) {
        System.out.println("Pedido de alto valor detectado: " + event.pedidoId());
    }
}
```

### 5.9. Event listeners retornando novos eventos

Um listener também pode retornar outro evento, permitindo encadear reações:

```java
package br.com.exemplo.events;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class GeracaoFinanceiraListener {

    @EventListener
    public NotaFiscalGeradaEvent processar(PedidoFaturadoEvent event) {
        return new NotaFiscalGeradaEvent(event.pedidoId(), "NF-" + event.pedidoId());
    }
}
```

```java
package br.com.exemplo.events;

public record NotaFiscalGeradaEvent(Long pedidoId, String numeroNota) {
}
```

Esse recurso é útil, mas deve ser usado com moderação para não tornar o fluxo difícil de rastrear.

### 5.10. Comparando Spring Events e JMS

Use Spring Events quando:

- a comunicação for interna;
- o processamento puder ocorrer no mesmo processo;
- o objetivo principal for desacoplamento entre beans.

Use JMS quando:

- a comunicação precisar ser distribuída;
- a entrega precisar ser desacoplada do ciclo de vida da aplicação;
- houver necessidade de broker, retry, fila, tópico ou persistência de mensagens.

Em muitos sistemas corporativos, os dois coexistem:

- Spring Events para comunicação interna entre services;
- JMS para integração externa ou comunicação entre sistemas.

### 5.11. Boas práticas com Spring Events

- mantenha os eventos pequenos e focados no fato de negócio;
- não coloque regra complexa diretamente no listener;
- prefira nomes orientados a acontecimentos, como `PedidoFaturadoEvent`;
- para integrações dependentes de commit, use `@TransactionalEventListener`;
- para carga alta ou necessidade de resiliência, considere mensageria externa;
- documente quais eventos são internos e quais disparam integrações.

### 5.12. Eventos padrão relacionados ao Spring Security

O Spring Security também publica eventos padrão para autenticação, autorização, logout e aspectos da sessão. Eles são muito uteis para:

- auditoria de login e logout;
- métricas de falha por credencial invalida, conta bloqueada ou usuário desabilitado;
- monitoramento de negações de acesso;
- rastreamento de eventos de segurança;
- reações técnicas como alertas e trilhas de auditoria.

Na pratica, vale separar mentalmente esses eventos em dois grupos:

- eventos de autenticação;
- eventos de autorização.

#### Configuração para publicação dos eventos

Para eventos de autenticação, a documentação oficial do Spring Security recomenda registrar um `AuthenticationEventPublisher`, normalmente usando `DefaultAuthenticationEventPublisher`.

```java
package br.com.exemplo.events.security;

import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationEventPublisher;
import org.springframework.security.authentication.DefaultAuthenticationEventPublisher;

@Configuration
public class SecurityAuthenticationEventsConfig {

    @Bean
    public AuthenticationEventPublisher authenticationEventPublisher(
            ApplicationEventPublisher applicationEventPublisher
    ) {
        return new DefaultAuthenticationEventPublisher(applicationEventPublisher);
    }
}
```

Para eventos de autorização, a documentação oficial recomenda registrar um `AuthorizationEventPublisher`, por exemplo com `SpringAuthorizationEventPublisher`.

```java
package br.com.exemplo.events.security;

import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authorization.AuthorizationEventPublisher;
import org.springframework.security.authorization.SpringAuthorizationEventPublisher;

@Configuration
public class SecurityAuthorizationEventsConfig {

    @Bean
    public AuthorizationEventPublisher authorizationEventPublisher(
            ApplicationEventPublisher applicationEventPublisher
    ) {
        return new SpringAuthorizationEventPublisher(applicationEventPublisher);
    }
}
```

#### Eventos padrão mais usados

| Evento | Quando ocorre | Uso comum | Observações |
| --- | --- | --- | --- |
| `AuthenticationSuccessEvent` | autenticação concluída com sucesso | auditoria de login, métricas de sucesso, registro de último acesso | representa sucesso de autenticação |
| `AbstractAuthenticationFailureEvent` | autenticação falhou | tratamento centralizado de falhas | classe base para eventos de falha |
| `AuthenticationFailureBadCredentialsEvent` | credenciais inválidas | contador de tentativas erradas, alertas, bloqueio progressivo | muito comum em login por usuário e senha |
| `AuthenticationFailureDisabledEvent` | usuário desabilitado tentou autenticar | auditoria e suporte operacional | bom para monitorar contas desativadas ainda em uso |
| `AuthenticationFailureLockedEvent` | autenticação falhou porque a conta está bloqueada | segurança e suporte | útil em fluxos de bloqueio por excesso de tentativas |
| `AuthenticationFailureCredentialsExpiredEvent` | autenticação falhou porque a credencial expirou | forcar redefinição de senha | comum em politicas corporativas de senha |
| `InteractiveAuthenticationSuccessEvent` | autenticação interativa foi bem-sucedida | auditoria de login web real | não estende `AuthenticationSuccessEvent` para evitar duplicidade |
| `LogoutSuccessEvent` | logout realizado com sucesso | trilha de auditoria, encerramento de sessão | disponível desde Spring Security 5.2 |
| `SessionFixationProtectionEvent` | o ID da sessão mudou por proteção contra session fixation | monitoramento de segurança de sessão | informa sessão antiga e nova |
| `AuthorizationDeniedEvent` | uma autorização foi negada | auditoria de acesso negado, alertas, métricas | recomendado para negações em modelos atuais de autorização |
| `AuthorizationGrantedEvent` | uma autorização foi concedida | trilha de acesso permitido quando necessário | pode ser habilitado conforme a estratégia de publicação |
| `AuthenticationSwitchUserEvent` | houve troca de usuário via switch user | auditoria administrativa | útil em ambientes com impersonation controlada |

Na maioria dos sistemas, os eventos mais valiosos no inicio são:

- `AuthenticationSuccessEvent`;
- `AuthenticationFailureBadCredentialsEvent`;
- `LogoutSuccessEvent`;
- `AuthorizationDeniedEvent`.

#### Exemplo de listener para autenticação

```java
package br.com.exemplo.events.security;

import org.springframework.context.event.EventListener;
import org.springframework.security.authentication.event.AuthenticationFailureBadCredentialsEvent;
import org.springframework.security.authentication.event.AuthenticationSuccessEvent;
import org.springframework.stereotype.Component;

@Component
public class SecurityAuthenticationEventsListener {

    @EventListener
    public void onSuccess(AuthenticationSuccessEvent event) {
        String username = event.getAuthentication().getName();
        System.out.println("Login realizado com sucesso por " + username);
    }

    @EventListener
    public void onBadCredentials(AuthenticationFailureBadCredentialsEvent event) {
        String username = event.getAuthentication().getName();
        System.out.println("Falha de autenticacao por credenciais invalidas para " + username);
    }
}
```

#### Exemplo de listener para autorização negada

```java
package br.com.exemplo.events.security;

import org.springframework.context.event.EventListener;
import org.springframework.security.authorization.event.AuthorizationDeniedEvent;
import org.springframework.stereotype.Component;

@Component
public class SecurityAuthorizationEventsListener {

    @EventListener
    public void onDenied(AuthorizationDeniedEvent<?> event) {
        System.out.println("Acesso negado para objeto protegido: " + event.getSource());
    }
}
```

#### Exemplo de listener para logout e proteção de sessão

```java
package br.com.exemplo.events.security;

import org.springframework.context.event.EventListener;
import org.springframework.security.authentication.event.LogoutSuccessEvent;
import org.springframework.security.web.authentication.session.SessionFixationProtectionEvent;
import org.springframework.stereotype.Component;

@Component
public class SecuritySessionEventsListener {

    @EventListener
    public void onLogout(LogoutSuccessEvent event) {
        System.out.println("Logout realizado por " + event.getAuthentication().getName());
    }

    @EventListener
    public void onSessionFixationProtection(SessionFixationProtectionEvent event) {
        System.out.println("Sessao trocada de " + event.getOldSessionId() + " para " + event.getNewSessionId());
    }
}
```

#### Eventos legados e eventos atuais de autorização

Nas versões atuais do Spring Security, os eventos legados do pacote `org.springframework.security.access.event` aparecem como descontinuados em vários casos. Para novos projetos, a recomendação e preferir os eventos modernos:

- `AuthorizationDeniedEvent`;
- `AuthorizationGrantedEvent`.

Isso é especialmente importante para evitar exemplos baseados em APIs antigas de autorização.

#### Boas práticas com eventos do Spring Security

- nunca registre senha ou token bruto em listeners;
- trate listeners de auditoria como componentes técnicos e enxutos;
- use esses eventos para métricas, alertas e trilha de segurança;
- em cenários críticos, persista a auditoria de forma assíncrona ou desacoplada;
- diferencie falha de autenticação de negação de autorização;
- ao contar tentativas de login, considere IP, usuário, horário e contexto da aplicação.

## 6. GraphQL com Spring Boot

GraphQL é uma linguagem de consulta para APIs criada pelo Facebook que permite ao cliente especificar exatamente quais dados precisa. Com o módulo Spring for GraphQL, o Spring Boot oferece suporte nativo para expor APIs GraphQL usando a mesma infraestrutura de controllers, validação e segurança.

### 6.1. GraphQL vs REST

| Aspecto | REST | GraphQL |
|---------|------|---------|
| **Endpoints** | Múltiplos (`/users`, `/orders`) | Único (`/graphql`) |
| **Dados retornados** | Fixos por endpoint | Cliente escolhe os campos |
| **Over-fetching** | Comum | Eliminado |
| **Under-fetching** | Requer múltiplas chamadas | Uma query resolve |
| **Versionamento** | URLs (`/v1/`, `/v2/`) | Schema evolui sem versão |
| **Caching** | HTTP nativo (GET + ETag) | Requer estratégia específica |
| **Curva de aprendizado** | Baixa | Moderada |

### 6.2. Dependência e configuração

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

```yaml
spring:
  graphql:
    graphiql:
      enabled: true
    schema:
      printer:
        enabled: true
```

Com essa configuração, o Spring Boot disponibiliza:
- Endpoint GraphQL: `POST /graphql`
- GraphiQL (interface interativa): `http://localhost:8080/graphiql`

### 6.3. Definição do schema

O schema define os tipos, queries e mutations disponíveis na API. Deve ser colocado em `src/main/resources/graphql/`.

```graphql
# src/main/resources/graphql/schema.graphqls
type Query {
    produtos(categoria: String, page: Int = 0, size: Int = 10): ProdutoPage!
    produto(id: ID!): Produto
}

type Mutation {
    criarProduto(input: ProdutoInput!): Produto!
    atualizarEstoque(id: ID!, quantidade: Int!): Produto!
}

type Produto {
    id: ID!
    nome: String!
    preco: BigDecimal!
    categoria: Categoria!
    avaliacoes: [Avaliacao!]!
}

type Categoria {
    id: ID!
    nome: String!
}

type Avaliacao {
    id: ID!
    nota: Int!
    comentario: String
    autor: String!
}

type ProdutoPage {
    content: [Produto!]!
    totalElements: Int!
    totalPages: Int!
}

input ProdutoInput {
    nome: String!
    preco: BigDecimal!
    categoriaId: ID!
}

scalar BigDecimal
```

### 6.4. Controllers GraphQL

Diferente de REST, o controller GraphQL usa `@Controller` (não `@RestController`) porque a serialização da resposta é gerenciada pelo framework GraphQL, não pelo Spring MVC. Todas as requisições passam por um único endpoint — `POST /graphql` — e o framework roteia para o método correto com base na query ou mutation recebida.

```java
package br.com.exemplo.graphql;

import org.springframework.data.domain.PageRequest;
import org.springframework.graphql.data.method.annotation.*;
import org.springframework.stereotype.Controller;

import java.util.List;
import java.util.Map;

@Controller
public class ProdutoGraphQLController {

    private final ProdutoService produtoService;
    private final AvaliacaoService avaliacaoService;

    public ProdutoGraphQLController(ProdutoService produtoService,
                                     AvaliacaoService avaliacaoService) {
        this.produtoService = produtoService;
        this.avaliacaoService = avaliacaoService;
    }

    @QueryMapping
    public ProdutoPage produtos(@Argument String categoria,
                                 @Argument int page,
                                 @Argument int size) {
        return produtoService.buscar(categoria, PageRequest.of(page, size));
    }

    @QueryMapping
    public Produto produto(@Argument Long id) {
        return produtoService.buscarPorId(id);
    }

    @MutationMapping
    public Produto criarProduto(@Argument ProdutoInput input) {
        return produtoService.criar(input);
    }

    @BatchMapping
    public Map<Produto, List<Avaliacao>> avaliacoes(List<Produto> produtos) {
        return avaliacaoService.buscarPorProdutos(produtos);
    }
}
```

#### Como o framework conecta schema e controller

O Spring for GraphQL usa convenção de nomes para vincular o schema ao controller. Cada método anotado com `@QueryMapping` ou `@MutationMapping` deve ter o **mesmo nome** da query ou mutation declarada no arquivo `.graphqls`:

```
Schema                                  Controller
──────────────────────                  ──────────────────────
type Query {
    produtos(...): ProdutoPage!    ──▶  @QueryMapping produtos(...)
    produto(id: ID!): Produto      ──▶  @QueryMapping produto(...)
}

type Mutation {
    criarProduto(...): Produto!    ──▶  @MutationMapping criarProduto(...)
}

type Produto {
    avaliacoes: [Avaliacao!]!      ──▶  @BatchMapping avaliacoes(...)
}
```

| Elemento no schema | Anotação no controller | Vinculação |
|---|---|---|
| `Query.produtos` | `@QueryMapping` no método `produtos()` | Pelo nome do método |
| `Query.produto` | `@QueryMapping` no método `produto()` | Pelo nome do método |
| `Mutation.criarProduto` | `@MutationMapping` no método `criarProduto()` | Pelo nome do método |
| `Produto.avaliacoes` | `@BatchMapping` no método `avaliacoes()` | Pelo nome do método + tipo de retorno |

O `@BatchMapping` no método `avaliacoes()` é acionado automaticamente sempre que uma query solicita o campo `avaliacoes` de um `Produto`. O Spring agrupa todos os produtos retornados e chama o método uma única vez com a lista completa, evitando o problema N+1.

Os parâmetros anotados com `@Argument` são mapeados a partir dos argumentos da query. O tipo Java deve ser compatível com o tipo GraphQL declarado no schema (`ID` → `Long`, `String` → `String`, `Int` → `int`, `BigDecimal` → `BigDecimal`, `input` → record/classe Java).

#### Anotações principais

| Anotação | Função |
|----------|--------|
| `@QueryMapping` | Mapeia um método para uma query do schema |
| `@MutationMapping` | Mapeia um método para uma mutation do schema |
| `@SchemaMapping` | Resolve um campo de um tipo (equivalente a field resolver) |
| `@BatchMapping` | Resolve um campo para uma lista de objetos de uma vez (resolve N+1) |
| `@Argument` | Injeta um argumento da query/mutation |

### 6.5. Resolvendo o problema N+1 com `@BatchMapping`

Sem `@BatchMapping`, ao buscar 10 produtos com suas avaliações, o framework faria 1 query para os produtos + 10 queries para as avaliações (uma por produto). Com `@BatchMapping`, o Spring agrupa todas as chamadas em uma única operação:

```java
@BatchMapping
public Map<Produto, List<Avaliacao>> avaliacoes(List<Produto> produtos) {
    List<Long> ids = produtos.stream().map(Produto::getId).toList();
    List<Avaliacao> todas = avaliacaoRepository.findByProdutoIdIn(ids);

    return produtos.stream().collect(Collectors.toMap(
            p -> p,
            p -> todas.stream()
                    .filter(a -> a.getProdutoId().equals(p.getId()))
                    .toList()
    ));
}
```

### 6.6. Exemplo de query e mutation via GraphiQL

O endpoint GraphQL fica disponível em:

```
POST http://localhost:8080/graphql
```

Para testar interativamente, acesse o GraphiQL no navegador:

```
http://localhost:8080/graphiql
```

#### Query — buscar produtos com avaliações

```graphql
query {
  produtos(categoria: "ELETRONICOS", page: 0, size: 5) {
    content {
      id
      nome
      preco
      avaliacoes {
        nota
        comentario
      }
    }
    totalElements
  }
}
```

Resposta:

```json
{
  "data": {
    "produtos": {
      "content": [
        {
          "id": "1",
          "nome": "Notebook Dell XPS 15",
          "preco": 4999.90,
          "avaliacoes": [
            { "nota": 5, "comentario": "Excelente!" },
            { "nota": 4, "comentario": "Bom, mas caro" }
          ]
        }
      ],
      "totalElements": 1
    }
  }
}
```

#### Query — buscar produto por ID

```graphql
query {
  produto(id: 1) {
    id
    nome
    preco
    categoria {
      nome
    }
  }
}
```

#### Mutation — criar produto

```graphql
mutation {
  criarProduto(input: { nome: "Monitor 4K", preco: 1899.90, categoriaId: 2 }) {
    id
    nome
    preco
  }
}
```

#### Mutation — atualizar estoque

```graphql
mutation {
  atualizarEstoque(id: 1, quantidade: 50) {
    id
    nome
    preco
  }
}
```

O cliente recebe exatamente os campos solicitados — sem over-fetching de dados desnecessários e sem under-fetching que exigiria chamadas adicionais.

### 6.7. Consumindo a API GraphQL com JavaScript/TypeScript

Toda comunicação com o endpoint GraphQL é feita via `POST /graphql` com um corpo JSON contendo a query (ou mutation) e opcionalmente as variáveis. Os exemplos abaixo mostram como consumir o `ProdutoGraphQLController` a partir de uma aplicação frontend.

#### Usando Fetch API (sem dependências)

```typescript
const GRAPHQL_URL = 'http://localhost:8080/graphql';

async function graphqlRequest<T>(query: string, variables?: Record<string, unknown>): Promise<T> {
  const response = await fetch(GRAPHQL_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query, variables }),
  });

  const json = await response.json();

  if (json.errors) {
    throw new Error(json.errors.map((e: { message: string }) => e.message).join(', '));
  }

  return json.data;
}
```

#### Query — buscar produtos

```typescript
interface Produto {
  id: string;
  nome: string;
  preco: number;
  avaliacoes: { nota: number; comentario: string | null }[];
}

interface ProdutoPage {
  produtos: {
    content: Produto[];
    totalElements: number;
    totalPages: number;
  };
}

async function buscarProdutos(categoria: string, page = 0, size = 10): Promise<ProdutoPage> {
  const query = `
    query BuscarProdutos($categoria: String, $page: Int, $size: Int) {
      produtos(categoria: $categoria, page: $page, size: $size) {
        content {
          id
          nome
          preco
          avaliacoes {
            nota
            comentario
          }
        }
        totalElements
        totalPages
      }
    }
  `;

  return graphqlRequest<ProdutoPage>(query, { categoria, page, size });
}

// uso
const resultado = await buscarProdutos('ELETRONICOS', 0, 5);
console.log(resultado.produtos.content);
```

#### Query — buscar produto por ID

```typescript
interface ProdutoDetalhe {
  produto: {
    id: string;
    nome: string;
    preco: number;
    categoria: { nome: string };
    avaliacoes: { nota: number; comentario: string | null; autor: string }[];
  } | null;
}

async function buscarProdutoPorId(id: number): Promise<ProdutoDetalhe> {
  const query = `
    query BuscarProduto($id: ID!) {
      produto(id: $id) {
        id
        nome
        preco
        categoria {
          nome
        }
        avaliacoes {
          nota
          comentario
          autor
        }
      }
    }
  `;

  return graphqlRequest<ProdutoDetalhe>(query, { id });
}
```

#### Mutation — criar produto

```typescript
interface CriarProdutoResult {
  criarProduto: {
    id: string;
    nome: string;
    preco: number;
  };
}

async function criarProduto(nome: string, preco: number, categoriaId: number): Promise<CriarProdutoResult> {
  const mutation = `
    mutation CriarProduto($input: ProdutoInput!) {
      criarProduto(input: $input) {
        id
        nome
        preco
      }
    }
  `;

  return graphqlRequest<CriarProdutoResult>(mutation, {
    input: { nome, preco, categoriaId },
  });
}

// uso
const novo = await criarProduto('Monitor 4K', 1899.90, 2);
console.log(`Produto criado com ID: ${novo.criarProduto.id}`);
```

#### Usando com React (exemplo com hook)

```tsx
import { useState, useEffect } from 'react';

function ProdutoList({ categoria }: { categoria: string }) {
  const [produtos, setProdutos] = useState<Produto[]>([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    buscarProdutos(categoria)
      .then((data) => {
        setProdutos(data.produtos.content);
        setTotal(data.produtos.totalElements);
      })
      .finally(() => setLoading(false));
  }, [categoria]);

  if (loading) return <p>Carregando...</p>;

  return (
    <div>
      <h2>Produtos ({total})</h2>
      <ul>
        {produtos.map((p) => (
          <li key={p.id}>
            {p.nome} — R$ {p.preco.toFixed(2)}
            <ul>
              {p.avaliacoes.map((a, i) => (
                <li key={i}>{'⭐'.repeat(a.nota)} {a.comentario}</li>
              ))}
            </ul>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### Diferenças em relação ao consumo de APIs REST

| Aspecto | REST (Fetch API) | GraphQL (Fetch API) |
|---|---|---|
| **URL** | Uma URL por recurso (`/api/produtos`, `/api/produtos/1`) | URL única (`/graphql`) |
| **Método HTTP** | GET, POST, PUT, DELETE | Sempre POST |
| **Corpo da requisição** | Dados do recurso (JSON) | Query/mutation + variáveis (JSON) |
| **Campos da resposta** | Todos os campos do recurso | Apenas os campos solicitados na query |
| **Múltiplos recursos** | Múltiplas requisições | Uma única query com campos aninhados |

### 6.8. Quando usar GraphQL vs REST

| Cenário | Recomendação |
|---------|-------------|
| API pública simples com recursos bem definidos | REST |
| Frontend precisa de dados de múltiplas entidades relacionadas | GraphQL |
| Mobile com banda limitada (evitar over-fetching) | GraphQL |
| API interna entre microsserviços | REST ou gRPC |
| CRUD simples com poucos campos | REST |
| Dashboard com consultas complexas e variáveis | GraphQL |
| Necessidade de cache HTTP nativo | REST |

Em muitos projetos, GraphQL e REST coexistem: REST para endpoints simples e operações CRUD, GraphQL para consultas complexas e agregação de dados.

---

## 7. gRPC com Spring Boot

O gRPC (Google Remote Procedure Call) é um framework de comunicação entre serviços de alto desempenho, amplamente utilizado em arquiteturas de microsserviços. Ele utiliza HTTP/2 como protocolo de transporte e Protocol Buffers (protobuf) como linguagem de definição de interface (IDL) e formato de serialização binária.

### 7.1. O que é gRPC

O gRPC foi desenvolvido pelo Google e posteriormente disponibilizado como projeto open source. Diferente do REST tradicional, onde a comunicação é baseada em JSON sobre HTTP/1.1, o gRPC utiliza serialização binária e aproveita os recursos avançados do HTTP/2.

Protocol Buffers (protobuf) é a linguagem utilizada para definir a estrutura das mensagens e os contratos dos serviços. A partir de um arquivo `.proto`, o compilador `protoc` gera automaticamente o código-fonte (stubs e skeletons) em diversas linguagens.

```
┌──────────────┐                                      ┌──────────────┐
│   Cliente     │                                      │   Servidor    │
│              │                                      │              │
│  Stub gerado │──── Chamada RPC (protobuf binário) ──▶│  Skeleton    │
│  (protoc)    │◀─── Resposta (protobuf binário) ──────│  (protoc)    │
│              │                                      │              │
│              │         Transporte: HTTP/2            │              │
└──────────────┘                                      └──────────────┘
```

| Característica | Descrição |
|---|---|
| **Serialização binária** | Protobuf é significativamente mais compacto e rápido que JSON |
| **HTTP/2 multiplexing** | Múltiplas requisições simultâneas em uma única conexão TCP |
| **Streaming** | Suporte nativo a streaming unidirecional e bidirecional |
| **Geração de código** | Stubs e skeletons gerados automaticamente a partir do `.proto` |
| **Contrato forte** | O arquivo `.proto` serve como contrato tipado entre cliente e servidor |
| **Multilinguagem** | Geração de código para Java, Go, Python, C#, Kotlin, entre outras |

### 7.2. gRPC vs REST vs GraphQL

| Aspecto | gRPC | REST | GraphQL |
|---|---|---|---|
| **Protocolo de transporte** | HTTP/2 | HTTP/1.1 ou HTTP/2 | HTTP/1.1 ou HTTP/2 |
| **Formato de dados** | Protobuf (binário) | JSON / XML (texto) | JSON (texto) |
| **Tipagem / Contrato** | Forte (arquivo `.proto`) | Fraca (OpenAPI opcional) | Forte (schema GraphQL) |
| **Streaming** | Nativo (4 tipos) | Não nativo (SSE, WebSocket) | Subscriptions (WebSocket) |
| **Performance** | Alta (binário + HTTP/2) | Moderada | Moderada |
| **Suporte a browser** | Limitado (requer grpc-web/proxy) | Nativo | Nativo |
| **Ferramentas de debug** | grpcurl, BloomRPC, Postman | curl, Postman, browser | GraphiQL, Playground |
| **Curva de aprendizado** | Alta (protobuf, geração de código) | Baixa | Média |
| **Ideal para** | Comunicação entre microsserviços | APIs públicas / CRUD | APIs com consultas flexíveis |

### 7.3. Tipos de comunicação gRPC

O gRPC suporta quatro padrões de comunicação:

| Tipo | Cliente | Servidor | Descrição |
|---|---|---|---|
| **Unary** | 1 mensagem | 1 mensagem | Request-response simples, similar a REST |
| **Server Streaming** | 1 mensagem | N mensagens | Cliente envia uma requisição, servidor responde com um fluxo |
| **Client Streaming** | N mensagens | 1 mensagem | Cliente envia um fluxo, servidor responde com uma única mensagem |
| **Bidirectional Streaming** | N mensagens | N mensagens | Ambos enviam e recebem fluxos simultaneamente |

```
Unary:              Client ──req──▶ Server ──res──▶ Client

Server Streaming:   Client ──req──▶ Server ══res══▶ Client
                                           ══res══▶
                                           ══res══▶

Client Streaming:   Client ══req══▶ Server
                           ══req══▶
                           ══req══▶        ──res──▶ Client

Bidirectional:      Client ══req══▶ Server
                           ══req══▶        ══res══▶
                           ══req══▶        ══res══▶ Client
```

Definição no arquivo `.proto`:

```protobuf
service ExemploService {
  rpc Buscar (Request) returns (Response);              // Unary
  rpc Listar (Request) returns (stream Response);       // Server Streaming
  rpc EnviarLote (stream Request) returns (Response);   // Client Streaming
  rpc Chat (stream Request) returns (stream Response);  // Bidirectional
}
```

### 7.4. Dependências e configuração

```xml
<properties>
    <grpc.version>1.68.0</grpc.version>
    <protobuf.version>4.28.3</protobuf.version>
    <grpc-spring-boot.version>3.1.0.RELEASE</grpc-spring-boot.version>
</properties>

<dependencies>
    <!-- gRPC Server -->
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-server-spring-boot-starter</artifactId>
        <version>${grpc-spring-boot.version}</version>
    </dependency>

    <!-- gRPC Client (em projetos que consomem serviços gRPC) -->
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-client-spring-boot-starter</artifactId>
        <version>${grpc-spring-boot.version}</version>
    </dependency>

    <!-- Protobuf -->
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>${protobuf.version}</version>
    </dependency>

    <!-- gRPC Stub e Protobuf -->
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>${grpc.version}</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>${grpc.version}</version>
    </dependency>

    <!-- Necessário para geração de código com Java 21+ -->
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>annotations-api</artifactId>
        <version>6.0.53</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.7.1</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>
                    com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}
                </protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>
                    io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}
                </pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

O plugin `protobuf-maven-plugin` compila automaticamente os arquivos `.proto` localizados em `src/main/proto/` durante o build, gerando as classes Java em `target/generated-sources/protobuf/`.

#### Configuração do servidor

```yaml
grpc:
  server:
    port: 9090

spring:
  application:
    name: produto-grpc-server
```

#### Configuração do cliente

```yaml
grpc:
  client:
    produto-service:
      address: static://localhost:9090
      negotiation-type: plaintext
      enable-keep-alive: true
      keep-alive-time: 30s
      keep-alive-timeout: 5s
```

Em produção, utilize `negotiation-type: tls` com certificados configurados para comunicação segura.

### 7.5. Definição do serviço (.proto)

O arquivo `.proto` define o contrato do serviço, incluindo as mensagens de entrada/saída e os métodos RPC disponíveis.

```protobuf
// src/main/proto/produto.proto
syntax = "proto3";

option java_multiple_files = true;
option java_package = "br.com.exemplo.grpc";
option java_outer_classname = "ProdutoProto";

package produto;

message ProdutoRequest {
  int64 id = 1;
}

message CriarProdutoRequest {
  string nome = 1;
  string descricao = 2;
  double preco = 3;
  int32 quantidade_estoque = 4;
}

message ProdutoResponse {
  int64 id = 1;
  string nome = 2;
  string descricao = 3;
  double preco = 4;
  int32 quantidade_estoque = 5;
  bool ativo = 6;
}

message ProdutoListResponse {
  repeated ProdutoResponse produtos = 1;
}

message Empty {}

service ProdutoService {
  rpc BuscarPorId (ProdutoRequest) returns (ProdutoResponse);
  rpc Listar (Empty) returns (stream ProdutoResponse);
  rpc Criar (CriarProdutoRequest) returns (ProdutoResponse);
}
```

Após executar `mvn compile`, as seguintes classes serão geradas automaticamente:

- `ProdutoRequest`, `ProdutoResponse`, `CriarProdutoRequest`, etc. — classes das mensagens
- `ProdutoServiceGrpc` — classe com os stubs e a base de implementação do servidor

### 7.6. Implementação do servidor

A implementação do servidor estende a classe base gerada pelo protobuf e utiliza a anotação `@GrpcService` do starter.

```java
package br.com.exemplo.grpc.server;

import br.com.exemplo.grpc.*;
import br.com.exemplo.service.ProdutoService;
import br.com.exemplo.model.Produto;
import io.grpc.Status;
import io.grpc.stub.StreamObserver;
import net.devh.boot.grpc.server.service.GrpcService;

import java.util.List;

@GrpcService
public class ProdutoGrpcServer extends ProdutoServiceGrpc.ProdutoServiceImplBase {

    private final ProdutoService produtoService;

    public ProdutoGrpcServer(ProdutoService produtoService) {
        this.produtoService = produtoService;
    }

    @Override
    public void buscarPorId(ProdutoRequest request,
                            StreamObserver<ProdutoResponse> responseObserver) {
        try {
            Produto produto = produtoService.buscarPorId(request.getId());

            ProdutoResponse response = ProdutoResponse.newBuilder()
                    .setId(produto.getId())
                    .setNome(produto.getNome())
                    .setDescricao(produto.getDescricao())
                    .setPreco(produto.getPreco())
                    .setQuantidadeEstoque(produto.getQuantidadeEstoque())
                    .setAtivo(produto.isAtivo())
                    .build();

            responseObserver.onNext(response);
            responseObserver.onCompleted();

        } catch (RuntimeException e) {
            responseObserver.onError(
                Status.NOT_FOUND
                    .withDescription("Produto não encontrado: " + request.getId())
                    .asRuntimeException()
            );
        }
    }

    @Override
    public void listar(Empty request,
                       StreamObserver<ProdutoResponse> responseObserver) {
        List<Produto> produtos = produtoService.listarTodos();

        for (Produto produto : produtos) {
            ProdutoResponse response = ProdutoResponse.newBuilder()
                    .setId(produto.getId())
                    .setNome(produto.getNome())
                    .setDescricao(produto.getDescricao())
                    .setPreco(produto.getPreco())
                    .setQuantidadeEstoque(produto.getQuantidadeEstoque())
                    .setAtivo(produto.isAtivo())
                    .build();

            responseObserver.onNext(response);
        }

        responseObserver.onCompleted();
    }

    @Override
    public void criar(CriarProdutoRequest request,
                      StreamObserver<ProdutoResponse> responseObserver) {
        Produto novoProduto = new Produto();
        novoProduto.setNome(request.getNome());
        novoProduto.setDescricao(request.getDescricao());
        novoProduto.setPreco(request.getPreco());
        novoProduto.setQuantidadeEstoque(request.getQuantidadeEstoque());

        Produto salvo = produtoService.salvar(novoProduto);

        ProdutoResponse response = ProdutoResponse.newBuilder()
                .setId(salvo.getId())
                .setNome(salvo.getNome())
                .setDescricao(salvo.getDescricao())
                .setPreco(salvo.getPreco())
                .setQuantidadeEstoque(salvo.getQuantidadeEstoque())
                .setAtivo(salvo.isAtivo())
                .build();

        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```

O gRPC utiliza códigos de status próprios, similares aos HTTP status codes:

| Código gRPC | Equivalente HTTP | Uso |
|---|---|---|
| `Status.OK` | 200 | Sucesso |
| `Status.NOT_FOUND` | 404 | Recurso não encontrado |
| `Status.INVALID_ARGUMENT` | 400 | Parâmetro inválido |
| `Status.ALREADY_EXISTS` | 409 | Recurso já existe |
| `Status.INTERNAL` | 500 | Erro interno do servidor |
| `Status.UNAUTHENTICATED` | 401 | Não autenticado |
| `Status.PERMISSION_DENIED` | 403 | Sem permissão |

### 7.7. Implementação do cliente

O starter do gRPC para Spring Boot facilita a criação de clientes com a anotação `@GrpcClient`.

#### Cliente síncrono (blocking stub)

```java
package br.com.exemplo.grpc.client;

import br.com.exemplo.grpc.*;
import net.devh.boot.grpc.client.inject.GrpcClient;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

@Service
public class ProdutoGrpcClient {

    @GrpcClient("produto-service")
    private ProdutoServiceGrpc.ProdutoServiceBlockingStub blockingStub;

    public ProdutoResponse buscarPorId(Long id) {
        ProdutoRequest request = ProdutoRequest.newBuilder()
                .setId(id)
                .build();

        return blockingStub.buscarPorId(request);
    }

    public List<ProdutoResponse> listarTodos() {
        Iterator<ProdutoResponse> iterator = blockingStub.listar(
                Empty.newBuilder().build()
        );

        List<ProdutoResponse> produtos = new ArrayList<>();
        iterator.forEachRemaining(produtos::add);
        return produtos;
    }

    public ProdutoResponse criar(String nome, String descricao,
                                  double preco, int quantidade) {
        CriarProdutoRequest request = CriarProdutoRequest.newBuilder()
                .setNome(nome)
                .setDescricao(descricao)
                .setPreco(preco)
                .setQuantidadeEstoque(quantidade)
                .build();

        return blockingStub.criar(request);
    }
}
```

#### Cliente assíncrono (async stub)

```java
package br.com.exemplo.grpc.client;

import br.com.exemplo.grpc.*;
import com.google.common.util.concurrent.ListenableFuture;
import io.grpc.stub.StreamObserver;
import net.devh.boot.grpc.client.inject.GrpcClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import java.util.concurrent.ExecutionException;

@Service
public class ProdutoGrpcAsyncClient {

    private static final Logger log = LoggerFactory.getLogger(ProdutoGrpcAsyncClient.class);

    @GrpcClient("produto-service")
    private ProdutoServiceGrpc.ProdutoServiceFutureStub futureStub;

    @GrpcClient("produto-service")
    private ProdutoServiceGrpc.ProdutoServiceStub asyncStub;

    public ProdutoResponse buscarPorIdAsync(Long id)
            throws ExecutionException, InterruptedException {
        ProdutoRequest request = ProdutoRequest.newBuilder()
                .setId(id)
                .build();

        ListenableFuture<ProdutoResponse> future = futureStub.buscarPorId(request);
        return future.get();
    }

    public void buscarPorIdComCallback(Long id) {
        ProdutoRequest request = ProdutoRequest.newBuilder()
                .setId(id)
                .build();

        asyncStub.buscarPorId(request, new StreamObserver<>() {
            @Override
            public void onNext(ProdutoResponse response) {
                log.info("Produto recebido: {} - {}", response.getId(), response.getNome());
            }

            @Override
            public void onError(Throwable t) {
                log.error("Erro ao buscar produto: {}", t.getMessage());
            }

            @Override
            public void onCompleted() {
                log.info("Chamada gRPC concluída");
            }
        });
    }
}
```

#### Expondo gRPC via REST

Em muitos cenários, o backend usa gRPC internamente entre microsserviços mas expõe REST para o frontend. Um controller intermediário faz a tradução:

```java
package br.com.exemplo.controller;

import br.com.exemplo.grpc.ProdutoResponse;
import br.com.exemplo.grpc.client.ProdutoGrpcClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/produtos")
public class ProdutoController {

    private final ProdutoGrpcClient grpcClient;

    public ProdutoController(ProdutoGrpcClient grpcClient) {
        this.grpcClient = grpcClient;
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProdutoResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(grpcClient.buscarPorId(id));
    }

    @GetMapping
    public ResponseEntity<List<ProdutoResponse>> listarTodos() {
        return ResponseEntity.ok(grpcClient.listarTodos());
    }
}
```

### 7.8. Quando usar gRPC

| Cenário | Justificativa |
|---|---|
| Comunicação entre microsserviços | Alta performance com serialização binária e HTTP/2 |
| Streaming de dados em tempo real | Suporte nativo a streaming bidirecional |
| Ambientes poliglotas | Geração de código para múltiplas linguagens a partir do mesmo `.proto` |
| Baixa latência / alto throughput | Protobuf é até 10x mais rápido que JSON para serialização |
| Contratos fortemente tipados | O arquivo `.proto` garante compatibilidade entre cliente e servidor |

**Quando NÃO usar gRPC:**

| Cenário | Motivo |
|---|---|
| APIs públicas acessadas por browsers | Browsers não suportam gRPC nativamente (requer proxy) |
| APIs REST simples (CRUD) | REST é mais simples, amplamente suportado e suficiente |
| Necessidade de debug fácil | JSON é legível por humanos; protobuf binário requer ferramentas |
| Comunicação com sistemas legados | Sistemas mais antigos geralmente suportam apenas REST/SOAP |

Para cenários onde o backend utiliza gRPC internamente mas precisa expor APIs para browsers ou clientes REST, as alternativas mais comuns são o Envoy Proxy (transcoding automático REST→gRPC), o grpc-gateway (gerador de reverse proxy REST a partir do `.proto`) ou um controller intermediário como mostrado acima.

```
┌──────────┐     REST/JSON      ┌──────────────────┐    gRPC/Protobuf    ┌──────────────┐
│  Browser  │──────────────────▶│  Envoy / Gateway  │───────────────────▶│  Serviço gRPC │
│  Mobile   │◀──────────────────│  REST Controller  │◀───────────────────│  Spring Boot  │
└──────────┘                    └──────────────────┘                     └──────────────┘
```

Em arquiteturas de microsserviços, uma abordagem comum é utilizar gRPC para comunicação interna entre serviços e REST para APIs externas consumidas por frontends e parceiros.

---

## 8. SpringDoc OpenAPI — Documentação de APIs

O SpringDoc gera automaticamente a documentação OpenAPI 3.0 a partir dos controllers REST do Spring, disponibilizando a especificação em JSON/YAML e uma interface interativa via Swagger UI. Isso elimina a necessidade de manter a documentação manualmente e garante que ela esteja sempre sincronizada com o código.

### 8.1. Dependência e configuração

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

```yaml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: method
```

Após subir a aplicação:
- Swagger UI interativo: `http://localhost:8080/swagger-ui.html`
- Especificação JSON: `http://localhost:8080/api-docs`
- Especificação YAML: `http://localhost:8080/api-docs.yaml`

### 8.2. Anotações para documentação

```java
package br.com.exemplo.api;

import io.swagger.v3.oas.annotations.*;
import io.swagger.v3.oas.annotations.responses.*;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springdoc.core.annotations.ParameterObject;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/produtos")
@Tag(name = "Produtos", description = "Gerenciamento de produtos do catálogo")
public class ProdutoController {

    private final ProdutoService produtoService;

    public ProdutoController(ProdutoService produtoService) {
        this.produtoService = produtoService;
    }

    @Operation(
        summary = "Buscar produtos",
        description = "Retorna lista paginada de produtos com filtro opcional por categoria"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista retornada com sucesso"),
        @ApiResponse(responseCode = "400", description = "Parâmetros inválidos")
    })
    @GetMapping
    public Page<ProdutoResponse> listar(
            @Parameter(description = "Nome da categoria para filtrar")
            @RequestParam(required = false) String categoria,
            @ParameterObject Pageable pageable) {
        return produtoService.buscar(categoria, pageable);
    }

    @Operation(summary = "Criar produto")
    @ApiResponse(responseCode = "201", description = "Produto criado com sucesso")
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ProdutoResponse criar(
            @Valid @RequestBody CriarProdutoRequest request) {
        return produtoService.criar(request);
    }
}
```

| Anotação | Onde usar | Função |
|----------|-----------|--------|
| `@Tag` | Classe do controller | Agrupa endpoints em seções na documentação |
| `@Operation` | Método do controller | Descreve o endpoint (summary e description) |
| `@ApiResponse` | Método do controller | Documenta códigos de resposta HTTP |
| `@Parameter` | Parâmetro do método | Descreve query params e path variables |
| `@ParameterObject` | Parâmetro composto (Pageable) | Expande os campos do objeto na documentação |

### 8.3. Separando documentação do controller com interface

Quando um controller tem muitos endpoints, as anotações do SpringDoc (`@Operation`, `@ApiResponse`, `@Parameter`, etc.) acabam poluindo a classe e dificultando a leitura da lógica real. Uma abordagem comum é extrair toda a documentação para uma interface que o controller implementa.

#### Interface de documentação

```java
package br.com.exemplo.api.doc;

import br.com.exemplo.api.CriarProdutoRequest;
import br.com.exemplo.api.ProdutoResponse;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import org.springdoc.core.annotations.ParameterObject;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.ResponseEntity;

@Tag(name = "Produtos", description = "Gerenciamento de produtos do catálogo")
public interface ProdutoControllerDoc {

    @Operation(
        summary = "Buscar produtos",
        description = "Retorna lista paginada de produtos com filtro opcional por categoria"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Lista retornada com sucesso"),
        @ApiResponse(responseCode = "400", description = "Parâmetros inválidos",
                content = @Content(schema = @Schema(hidden = true)))
    })
    Page<ProdutoResponse> listar(
            @Parameter(description = "Nome da categoria para filtrar")
            String categoria,
            @ParameterObject Pageable pageable);

    @Operation(summary = "Buscar produto por ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Produto encontrado"),
        @ApiResponse(responseCode = "404", description = "Produto não encontrado",
                content = @Content(schema = @Schema(hidden = true)))
    })
    ResponseEntity<ProdutoResponse> buscarPorId(
            @Parameter(description = "ID do produto", required = true)
            Long id);

    @Operation(summary = "Criar produto")
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Produto criado com sucesso"),
        @ApiResponse(responseCode = "400", description = "Dados inválidos",
                content = @Content(schema = @Schema(hidden = true)))
    })
    ResponseEntity<ProdutoResponse> criar(@Valid CriarProdutoRequest request);

    @Operation(summary = "Atualizar produto")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Produto atualizado"),
        @ApiResponse(responseCode = "404", description = "Produto não encontrado",
                content = @Content(schema = @Schema(hidden = true)))
    })
    ResponseEntity<ProdutoResponse> atualizar(Long id, @Valid CriarProdutoRequest request);

    @Operation(summary = "Remover produto")
    @ApiResponses({
        @ApiResponse(responseCode = "204", description = "Produto removido"),
        @ApiResponse(responseCode = "404", description = "Produto não encontrado",
                content = @Content(schema = @Schema(hidden = true)))
    })
    ResponseEntity<Void> remover(Long id);
}
```

#### Controller limpo

O controller implementa a interface e herda automaticamente todas as anotações de documentação. Ele fica focado apenas na lógica de entrada/saída:

```java
package br.com.exemplo.api;

import br.com.exemplo.api.doc.ProdutoControllerDoc;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/produtos")
public class ProdutoController implements ProdutoControllerDoc {

    private final ProdutoService produtoService;

    public ProdutoController(ProdutoService produtoService) {
        this.produtoService = produtoService;
    }

    @GetMapping
    @Override
    public Page<ProdutoResponse> listar(
            @RequestParam(required = false) String categoria,
            Pageable pageable) {
        return produtoService.buscar(categoria, pageable);
    }

    @GetMapping("/{id}")
    @Override
    public ResponseEntity<ProdutoResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(produtoService.buscarPorId(id));
    }

    @PostMapping
    @Override
    public ResponseEntity<ProdutoResponse> criar(@RequestBody CriarProdutoRequest request) {
        ProdutoResponse criado = produtoService.criar(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(criado);
    }

    @PutMapping("/{id}")
    @Override
    public ResponseEntity<ProdutoResponse> atualizar(
            @PathVariable Long id,
            @RequestBody CriarProdutoRequest request) {
        return ResponseEntity.ok(produtoService.atualizar(id, request));
    }

    @DeleteMapping("/{id}")
    @Override
    public ResponseEntity<Void> remover(@PathVariable Long id) {
        produtoService.remover(id);
        return ResponseEntity.noContent().build();
    }
}
```

Nesse modelo, as anotações do Spring Web (`@GetMapping`, `@RequestParam`, `@PathVariable`, `@RequestBody`) ficam no controller, enquanto as anotações do SpringDoc (`@Operation`, `@ApiResponse`, `@Parameter`, `@Tag`) ficam na interface. O SpringDoc reconhece as anotações herdadas da interface e gera a documentação normalmente.

#### Organização sugerida

```text
br.com.exemplo.api/
├── doc/
│   ├── ProdutoControllerDoc.java
│   ├── PedidoControllerDoc.java
│   └── CategoriaControllerDoc.java
├── ProdutoController.java
├── PedidoController.java
├── CategoriaController.java
├── CriarProdutoRequest.java
└── ProdutoResponse.java
```

Essa abordagem é especialmente útil quando os endpoints têm documentação extensa (múltiplos `@ApiResponse` com schemas de erro, `@Content`, exemplos) — situações em que as anotações inline tornam o controller difícil de ler.

### 8.4. Documentação de DTOs com `@Schema`

```java
package br.com.exemplo.api;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.*;
import java.math.BigDecimal;

@Schema(description = "Dados para criação de um produto")
public record CriarProdutoRequest(
    @Schema(description = "Nome do produto", example = "Notebook Dell XPS 15")
    @NotBlank
    String nome,

    @Schema(description = "Preço unitário", example = "4999.90")
    @NotNull @Positive
    BigDecimal preco,

    @Schema(description = "ID da categoria", example = "1")
    @NotNull
    Long categoriaId
) {}
```

O SpringDoc reconhece automaticamente as anotações do Bean Validation (`@NotBlank`, `@NotNull`, `@Positive`) e inclui as restrições na documentação gerada.

### 8.5. Configuração global e segurança

```java
package br.com.exemplo.api;

import io.swagger.v3.oas.models.*;
import io.swagger.v3.oas.models.info.*;
import io.swagger.v3.oas.models.security.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("API do E-Commerce")
                        .version("1.0")
                        .description("API REST para gerenciamento de produtos e pedidos")
                        .contact(new Contact()
                                .name("Equipe Backend")
                                .email("backend@exemplo.com")))
                .addSecurityItem(new SecurityRequirement().addList("Bearer"))
                .components(new Components()
                        .addSecuritySchemes("Bearer",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.HTTP)
                                        .scheme("bearer")
                                        .bearerFormat("JWT")));
    }
}
```

Com essa configuração, o Swagger UI exibe o botão "Authorize" que permite informar o token JWT manualmente para testar endpoints protegidos.

#### Integração com OAuth2 / OpenID Connect (Keycloak, Auth0, etc.)

Quando a aplicação utiliza um Identity Provider (IdP) externo com OAuth2 e OpenID Connect, o Swagger UI pode ser configurado para executar o fluxo de autenticação diretamente pelo navegador — sem necessidade de copiar tokens manualmente.

```java
package br.com.exemplo.api;

import io.swagger.v3.oas.models.*;
import io.swagger.v3.oas.models.info.*;
import io.swagger.v3.oas.models.security.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiOAuth2Config {

    @Value("${spring.security.oauth2.resourceserver.jwt.issuer-uri}")
    private String issuerUri;

    @Bean
    public OpenAPI customOpenAPI() {
        String authUrl = issuerUri + "/protocol/openid-connect/auth";
        String tokenUrl = issuerUri + "/protocol/openid-connect/token";

        return new OpenAPI()
                .info(new Info()
                        .title("API do E-Commerce")
                        .version("1.0")
                        .description("API REST protegida com OAuth2/OIDC"))
                .addSecurityItem(new SecurityRequirement().addList("oauth2"))
                .components(new Components()
                        .addSecuritySchemes("oauth2",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.OAUTH2)
                                        .flows(new OAuthFlows()
                                                .authorizationCode(new OAuthFlow()
                                                        .authorizationUrl(authUrl)
                                                        .tokenUrl(tokenUrl)
                                                        .scopes(new Scopes()
                                                                .addString("openid", "OpenID Connect")
                                                                .addString("profile", "Dados do perfil")
                                                                .addString("email", "Endereço de e-mail"))))));
    }
}
```

Configuração complementar no `application.yml`:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://idp.exemplo.com/realms/meu-realm

springdoc:
  swagger-ui:
    oauth:
      client-id: swagger-ui-client
      use-pkce-with-authorization-code-flow: true
```

| Configuração | Descrição |
|---|---|
| `SecurityScheme.Type.OAUTH2` | Indica que a autenticação usa o protocolo OAuth2 |
| `authorizationCode` | Fluxo recomendado para aplicações com interação de usuário (Authorization Code + PKCE) |
| `authorizationUrl` | URL do IdP para iniciar o login (redireciona o navegador) |
| `tokenUrl` | URL do IdP para trocar o authorization code por um access token |
| `scopes` | Escopos solicitados durante o login |
| `use-pkce-with-authorization-code-flow` | Habilita PKCE — obrigatório para clientes públicos como o Swagger UI |

Com essa configuração, o botão "Authorize" do Swagger UI abre a tela de login do IdP (Keycloak, Auth0, Entra ID, etc.) no navegador. Após o login, o token é obtido automaticamente e incluído nas requisições de teste.

As URLs do exemplo seguem o padrão do Keycloak. Para outros IdPs, ajuste os paths:

| IdP | Authorization URL | Token URL |
|-----|------------------|-----------|
| **Keycloak** | `{issuer}/protocol/openid-connect/auth` | `{issuer}/protocol/openid-connect/token` |
| **Auth0** | `https://{domain}/authorize` | `https://{domain}/oauth/token` |
| **Entra ID** | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize` | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token` |
| **Google** | `https://accounts.google.com/o/oauth2/v2/auth` | `https://oauth2.googleapis.com/token` |

#### Configuração via OpenID Connect Discovery (well-known)

O exemplo anterior exige montar manualmente as URLs de autorização e token. Com `SecurityScheme.Type.OPENIDCONNECT`, basta informar a URL do endpoint de discovery (`.well-known/openid-configuration`) e o Swagger UI resolve automaticamente todas as URLs do IdP:

```java
package br.com.exemplo.api;

import io.swagger.v3.oas.models.*;
import io.swagger.v3.oas.models.info.*;
import io.swagger.v3.oas.models.security.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiOidcConfig {

    @Value("${spring.security.oauth2.resourceserver.jwt.issuer-uri}")
    private String issuerUri;

    @Bean
    public OpenAPI customOpenAPI() {
        String discoveryUrl = issuerUri + "/.well-known/openid-configuration";

        return new OpenAPI()
                .info(new Info()
                        .title("API do E-Commerce")
                        .version("1.0")
                        .description("API REST protegida com OpenID Connect"))
                .addSecurityItem(new SecurityRequirement().addList("oidc"))
                .components(new Components()
                        .addSecuritySchemes("oidc",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.OPENIDCONNECT)
                                        .openIdConnectUrl(discoveryUrl)));
    }
}
```

A configuração YAML permanece a mesma:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://idp.exemplo.com/realms/meu-realm

springdoc:
  swagger-ui:
    oauth:
      client-id: swagger-ui-client
      use-pkce-with-authorization-code-flow: true
```

O endpoint `.well-known/openid-configuration` retorna um JSON com todos os metadados do IdP — authorization endpoint, token endpoint, escopos suportados, algoritmos de assinatura, etc. O Swagger UI lê esse JSON e configura o fluxo OAuth2 automaticamente.

Exemplo de resposta do well-known (resumido):

```json
{
  "issuer": "https://idp.exemplo.com/realms/meu-realm",
  "authorization_endpoint": "https://idp.exemplo.com/realms/meu-realm/protocol/openid-connect/auth",
  "token_endpoint": "https://idp.exemplo.com/realms/meu-realm/protocol/openid-connect/token",
  "userinfo_endpoint": "https://idp.exemplo.com/realms/meu-realm/protocol/openid-connect/userinfo",
  "jwks_uri": "https://idp.exemplo.com/realms/meu-realm/protocol/openid-connect/certs",
  "scopes_supported": ["openid", "profile", "email"],
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"]
}
```

#### Comparação das três abordagens

| Abordagem | Tipo do `SecurityScheme` | Quando usar |
|---|---|---|
| **JWT Bearer** | `HTTP` / `bearer` | Token já obtido externamente; cole no Swagger UI para testar |
| **OAuth2 (URLs explícitas)** | `OAUTH2` | Fluxo completo pelo Swagger UI; útil quando o IdP não expõe well-known ou quando se quer controle total sobre as URLs |
| **OpenID Connect Discovery** | `OPENIDCONNECT` | Fluxo completo pelo Swagger UI com configuração mínima; o IdP precisa expor `/.well-known/openid-configuration` (Keycloak, Auth0, Entra ID e Google expõem por padrão) |

### 8.6. Agrupamento de APIs com `GroupedOpenApi`

Em aplicações maiores, é útil separar a documentação em grupos para facilitar a navegação:

```java
package br.com.exemplo.api;

import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiGroupConfig {

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("publico")
                .pathsToMatch("/api/produtos/**", "/api/categorias/**")
                .build();
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
                .group("admin")
                .pathsToMatch("/admin/**")
                .build();
    }

    @Bean
    public GroupedOpenApi actuatorApi() {
        return GroupedOpenApi.builder()
                .group("actuator")
                .pathsToMatch("/actuator/**")
                .build();
    }
}
```

No Swagger UI, um seletor dropdown permite alternar entre os grupos de APIs.

### 8.7. Boas práticas

- em controllers com muitas anotações de documentação, extraia-as para uma interface `*ControllerDoc` — mantém o controller legível e a documentação organizada;
- use `@Tag` para agrupar endpoints por domínio de negócio, não por controller;
- documente `@Schema` nos records/DTOs de entrada e saída com `description` e `example`;
- configure `GroupedOpenApi` para separar APIs públicas de administrativas;
- em produção, considere desabilitar o Swagger UI por perfil (`springdoc.swagger-ui.enabled=false`);
- use `@Hidden` para ocultar endpoints internos que não devem aparecer na documentação pública.

---

## 9. Mecanismos de resiliência

Aplicações corporativas normalmente dependem de APIs, bancos, brokers e serviços externos. Por isso, é importante aplicar mecanismos de resiliência para evitar que falhas temporárias derrubem fluxos inteiros ou sobrecarreguem o sistema.

Os mecanismos mais comuns são:

- timeout;
- retry;
- circuit breaker;
- bulkhead;
- rate limiter;
- fallback.

Em Spring Boot, a combinação mais comum para esses casos é:

- `spring-retry` para repetição de tentativas em cenários simples;
- Resilience4j para circuit breaker, bulkhead, rate limiter, retry e timeout;
- observabilidade para acompanhar falhas, latência e degradação.

### 9.1. Quando aplicar resiliência

Esses mecanismos costumam ser aplicados em chamadas para:

- APIs REST externas;
- filas, brokers e integrações assíncronas;
- operações remotas via HTTP ou mensageria;
- consultas a recursos mais lentos ou instáveis.

Nem toda operação deve ter retry ou fallback. Em geral:

- use retry para falhas transientes;
- use timeout para evitar espera indefinida;
- use circuit breaker quando um serviço externo estiver instável;
- use bulkhead para isolar consumo de recursos;
- use rate limiter quando for necessário controlar volume de chamadas;
- use fallback apenas quando houver uma resposta degradada aceitável.

### 9.2. Dependências mais comuns

Exemplo com Resilience4j e Spring Retry:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot3</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.retry</groupId>
        <artifactId>spring-retry</artifactId>
    </dependency>
</dependencies>
```

O starter AOP e necessário porque boa parte dessas anotações funciona por proxy.

### 9.3. Timeout

O timeout impede que uma chamada fique presa por tempo indeterminado.

Em clientes HTTP, o ideal e combinar:

- timeout no cliente;
- timeout de resiliência;
- monitoramento de latência.

Exemplo de configuração com `RestClient`:

```java
package br.com.exemplo.resilience;

import java.time.Duration;
import org.springframework.boot.web.client.ClientHttpRequestFactories;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestClient;
import org.springframework.http.client.ClientHttpRequestFactory;

@Configuration
public class RestClientConfig {

    @Bean
    public RestClient restClient(RestClient.Builder builder) {
        ClientHttpRequestFactory requestFactory = ClientHttpRequestFactories.get(
                settings -> settings
                        .withConnectTimeout(Duration.ofSeconds(2))
                        .withReadTimeout(Duration.ofSeconds(3))
        );

        return builder
                .requestFactory(requestFactory)
                .baseUrl("https://api.externa.com")
                .build();
    }
}
```

Exemplo com `@TimeLimiter` do Resilience4j:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import java.util.concurrent.CompletableFuture;
import org.springframework.stereotype.Service;

@Service
public class ConsultaExternaAsyncService {

    @TimeLimiter(name = "consultaExterna", fallbackMethod = "fallbackConsulta")
    public CompletableFuture<String> consultar() {
        return CompletableFuture.supplyAsync(() -> {
            // Chamada remota demorada.
            return "resposta externa";
        });
    }

    public CompletableFuture<String> fallbackConsulta(Throwable ex) {
        return CompletableFuture.completedFuture("resposta degradada");
    }
}
```

Configuração:

```yaml
resilience4j:
  timelimiter:
    instances:
      consultaExterna:
        timeoutDuration: 3s
        cancelRunningFuture: true
```

### 9.4. Retry com Resilience4j

O retry deve ser usado quando a falha é temporária, por exemplo:

- timeout ocasional;
- erro 503 de serviço momentaneamente indisponível;
- falha de rede intermitente.

Exemplo:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.retry.annotation.Retry;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class ConsultaFornecedorService {

    private final RestClient restClient;

    public ConsultaFornecedorService(RestClient restClient) {
        this.restClient = restClient;
    }

    @Retry(name = "consultaFornecedor", fallbackMethod = "fallbackFornecedor")
    public String consultarProduto(String codigo) {
        return restClient.get()
                .uri("/produtos/{codigo}", codigo)
                .retrieve()
                .body(String.class);
    }

    public String fallbackFornecedor(String codigo, Throwable ex) {
        return "{\"status\":\"indisponivel\",\"codigo\":\"" + codigo + "\"}";
    }
}
```

Configuração:

```yaml
resilience4j:
  retry:
    instances:
      consultaFornecedor:
        maxAttempts: 3
        waitDuration: 500ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
```

Boas praticas:

- evitar retry em erro funcional, como validação;
- evitar retry infinito;
- combinar retry com timeout;
- observar o impacto de repetição em operações não idempotentes.

### 9.5. Retry com Spring Retry

Para casos mais simples, o Spring Retry ainda e uma opção bastante prática.

Habilitação:

```java
package br.com.exemplo.resilience;

import org.springframework.context.annotation.Configuration;
import org.springframework.retry.annotation.EnableRetry;

@Configuration
@EnableRetry
public class RetryConfig {
}
```

Uso:

```java
package br.com.exemplo.resilience;

import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

@Service
public class EstoqueIntegracaoService {

    @Retryable(
            retryFor = Exception.class,
            maxAttempts = 4,
            backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public String atualizarSaldo(String sku) {
        throw new IllegalStateException("Falha temporaria no serviço de estoque");
    }

    @Recover
    public String recuperar(Exception ex, String sku) {
        return "Não foi possível atualizar o saldo do SKU " + sku;
    }
}
```

Essa abordagem e boa quando o foco é apenas repetição de tentativa em um service local.

### 9.6. Circuit breaker

O circuit breaker evita que a aplicação continue insistindo em um serviço que está falhando de forma recorrente.

Estados principais:

- `CLOSED`: operação normal;
- `OPEN`: chamadas bloqueadas temporariamente;
- `HALF_OPEN`: algumas chamadas de teste são liberadas.

Exemplo:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class PagamentoGatewayService {

    private final RestClient restClient;

    public PagamentoGatewayService(RestClient restClient) {
        this.restClient = restClient;
    }

    @CircuitBreaker(name = "pagamentoGateway", fallbackMethod = "fallbackPagamento")
    public String autorizar(String payload) {
        return restClient.post()
                .uri("/pagamentos/autorizar")
                .body(payload)
                .retrieve()
                .body(String.class);
    }

    public String fallbackPagamento(String payload, Throwable ex) {
        return "{\"status\":\"processamento_pendente\"}";
    }
}
```

Configuração:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      pagamentoGateway:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 3
```

Esse mecanismo protege a aplicação e também reduz carga sobre o sistema externo problemático.

### 9.7. Bulkhead

O bulkhead isola recursos para que a falha ou lentidão de uma integração não consuma todas as threads da aplicação.

Exemplo com `SemaphoreBulkhead`:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.bulkhead.annotation.Bulkhead;
import org.springframework.stereotype.Service;

@Service
public class ConsultaDocumentoService {

    @Bulkhead(name = "consultaDocumento", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "fallback")
    public String consultar(String documento) {
        return "ok";
    }

    public String fallback(String documento, Throwable ex) {
        return "Serviço temporariamente sobrecarregado";
    }
}
```

Configuração:

```yaml
resilience4j:
  bulkhead:
    instances:
      consultaDocumento:
        maxConcurrentCalls: 10
        maxWaitDuration: 100ms
```

Quando usar:

- integrações lentas;
- operações que podem monopolizar threads;
- isolamento entre fluxos críticos e secundários.

### 9.8. Rate limiter

O rate limiter controla a quantidade de chamadas em determinado intervalo.

Exemplo:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import org.springframework.stereotype.Service;

@Service
public class ConsultaCepService {

    @RateLimiter(name = "consultaCep", fallbackMethod = "fallback")
    public String consultar(String cep) {
        return "resultado do CEP";
    }

    public String fallback(String cep, Throwable ex) {
        return "limite de consultas excedido";
    }
}
```

Configuração:

```yaml
resilience4j:
  ratelimiter:
    instances:
      consultaCep:
        limitForPeriod: 5
        limitRefreshPeriod: 1s
        timeoutDuration: 0
```

Esse mecanismo é útil para:

- proteger APIs com quota;
- evitar estouro de consumo em parceiros externos;
- reduzir picos causados por rajadas de requisições.

### 9.9. Rate limiter com Bucket4j

Outra abordagem bastante popular em Spring Boot é usar Bucket4j, uma biblioteca baseada no algoritmo de token bucket. Ela é especialmente útil quando o limite precisa ser aplicado diretamente sobre endpoints HTTP, usuários, IPs ou chaves de negócio.

Bucket4j funciona bem quando:

- o controle de taxa precisa ser muito explicito;
- o limite deve ser calculado por IP, usuário, token ou tenant;
- o projeto quer uma política de rate limit independente de um cliente HTTP específico;
- há necessidade de evoluir depois para buckets distribuídos.

Segundo a documentação oficial do Bucket4j, a dependência principal para Java 17+ é `com.bucket4j:bucket4j_jdk17-core`. O próprio projeto também destaca que, para casos genéricos com Spring Boot, existe um starter dedicado baseado em configuração, mas o uso direto da biblioteca e muito bom quando queremos controle fino em código.

#### Dependência

```xml
<dependency>
    <groupId>com.bucket4j</groupId>
    <artifactId>bucket4j_jdk17-core</artifactId>
    <version>8.15.0</version>
</dependency>
```

#### Exemplo simples em memória

Configuração de um bucket com capacidade de 20 requisições por minuto:

```java
package br.com.exemplo.resilience;

import io.github.bucket4j.Bandwidth;
import io.github.bucket4j.Bucket;
import java.time.Duration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Bucket4jConfig {

    @Bean
    public Bandwidth apiRateLimitBandwidth() {
        return Bandwidth.builder()
                .capacity(20)
                .refillGreedy(20, Duration.ofMinutes(1))
                .build();
    }
}
```

Serviço para obter buckets por chave, por exemplo IP ou usuário:

```java
package br.com.exemplo.resilience;

import io.github.bucket4j.Bucket;
import io.github.bucket4j.Bandwidth;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.stereotype.Service;

@Service
public class Bucket4jRateLimitService {

    private final Bandwidth apiRateLimitBandwidth;
    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    public Bucket4jRateLimitService(Bandwidth apiRateLimitBandwidth) {
        this.apiRateLimitBandwidth = apiRateLimitBandwidth;
    }

    public Bucket resolveBucket(String chave) {
        return buckets.computeIfAbsent(chave, key -> Bucket.builder()
                .addLimit(apiRateLimitBandwidth)
                .build());
    }
}
```

Filtro HTTP simples por IP:

```java
package br.com.exemplo.resilience;

import io.github.bucket4j.Bucket;
import io.github.bucket4j.ConsumptionProbe;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

@Component
public class Bucket4jRateLimitFilter extends OncePerRequestFilter {

    private final Bucket4jRateLimitService rateLimitService;

    public Bucket4jRateLimitFilter(Bucket4jRateLimitService rateLimitService) {
        this.rateLimitService = rateLimitService;
    }

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {
        String chave = request.getRemoteAddr();
        Bucket bucket = rateLimitService.resolveBucket(chave);
        ConsumptionProbe probe = bucket.tryConsumeAndReturnRemaining(1);

        if (probe.isConsumed()) {
            response.addHeader("X-Rate-Limit-Remaining", String.valueOf(probe.getRemainingTokens()));
            filterChain.doFilter(request, response);
            return;
        }

        response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
        response.setHeader("X-Rate-Limit-Retry-After-Seconds",
                String.valueOf(probe.getNanosToWaitForRefill() / 1_000_000_000L));
        response.getWriter().write("Limite de requisicoes excedido");
    }
}
```

Esse modelo e bom para:

- proteger endpoints públicos;
- limitar chamadas por IP;
- aplicar politicas simples sem depender de gateway externo.

#### Rate limit por usuário, token ou tenant

Em vez de usar IP, a chave pode ser:

- login do usuário autenticado;
- API key;
- identificador do tenant;
- combinação de usuario e endpoint.

Exemplo conceitual:

```java
String chave = tenantId + ":" + username + ":" + request.getRequestURI();
Bucket bucket = rateLimitService.resolveBucket(chave);
```

Isso permite políticas diferentes por perfil ou plano comercial.

#### Exemplo de resposta HTTP 429 enriquecida

Uma boa prática é devolver cabeçalhos que ajudem o cliente a entender o bloqueio:

- `X-Rate-Limit-Remaining`;
- `X-Rate-Limit-Retry-After-Seconds`;
- eventualmente um corpo JSON com detalhes.

Exemplo:

```json
{
  "status": 429,
  "erro": "TOO_MANY_REQUESTS",
  "mensagem": "Limite de requisições excedido para este cliente."
}
```

#### Buckets distribuídos

O exemplo anterior usa memória local e funciona bem em:

- aplicações simples;
- ambientes de desenvolvimento;
- uma única instancia.

Em produção com várias instancias, o ideal é compartilhar o estado do bucket usando tecnologia distribuída, como cache ou armazenamento compatível com os módulos distribuídos suportados pelo Bucket4j. A documentação oficial destaca suporte para cenários distribuídos com integrações como JCache, Hazelcast, Ignite, Infinispan e Coherence.

Nesses casos, o objetivo é fazer todas as instancias enxergarem o mesmo consumo de tokens.

#### Quando preferir Bucket4j em vez de `RateLimiter` do Resilience4j

Bucket4j costuma ser uma escolha melhor quando:

- o rate limit está ligado a requisições HTTP de entrada;
- a chave do limite depende do cliente chamador;
- o controle precisa ser por IP, usuário ou API key;
- o bloqueio deve ocorrer antes de chegar na regra de negocio.

O `RateLimiter` do Resilience4j costuma ser mais natural quando:

- queremos limitar chamadas de saída para um serviço externo;
- o controle está acoplado a um método específico;
- a política de resiliência faz parte de uma chamada remota do service.

### 9.10. Fallback

O fallback define o comportamento degradado quando a chamada principal falha.

Exemplos de fallback aceitável:

- retornar cache;
- retornar status simplificado;
- marcar processamento como pendente;
- enfileirar para tentativa posterior;
- responder com informação parcial.

Exemplo:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class FreteService {

    @CircuitBreaker(name = "freteApi", fallbackMethod = "fallbackFrete")
    public String calcular(String cep, Double peso) {
        throw new IllegalStateException("Falha na API de frete");
    }

    public String fallbackFrete(String cep, Double peso, Throwable ex) {
        return "{\"modalidade\":\"PADRAO\",\"prazo\":7}";
    }
}
```

Cuidados:

- fallback não deve mascarar erro grave sem observabilidade;
- a resposta degradada precisa ser coerente com o negócio;
- em fluxos financeiros ou críticos, talvez o correto seja falhar explicitamente.

### 9.11. Combinando mecanismos

Em muitos cenários, os mecanismos são usados juntos. Um arranjo comum para chamadas HTTP externas é:

1. timeout curto;
2. retry controlado;
3. circuit breaker;
4. fallback;
5. monitoramento por métricas e logs.

Exemplo conceitual:

```java
package br.com.exemplo.resilience;

import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;
import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import java.util.concurrent.CompletableFuture;
import org.springframework.stereotype.Service;

@Service
public class ClienteScoreService {

    @Retry(name = "scoreApi")
    @CircuitBreaker(name = "scoreApi", fallbackMethod = "fallbackScore")
    @TimeLimiter(name = "scoreApi")
    public CompletableFuture<String> consultar() {
        return CompletableFuture.supplyAsync(() -> "score=700");
    }

    public CompletableFuture<String> fallbackScore(Throwable ex) {
        return CompletableFuture.completedFuture("score=indisponivel");
    }
}
```

Ao combinar mecanismos, evite exagero. Muitas camadas de resiliência mal calibradas podem aumentar latência, mascarar problemas ou gerar tempestade de tentativas.

### 9.12. Resiliencia em consumidores JMS e jobs

Os mesmos conceitos valem para jobs e mensageria:

- listeners JMS podem usar retry controlado no processamento;
- jobs Quartz podem aplicar circuit breaker ao chamar serviços externos;
- falhas recorrentes podem redirecionar para dead letter queue ou reprocessamento posterior;
- operações demoradas devem ter timeout e isolamento de recursos.

Exemplo em listener JMS:

```java
package br.com.exemplo.resilience;

import br.com.exemplo.jms.Destinations;
import io.github.resilience4j.retry.annotation.Retry;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class PedidoResilienteConsumer {

    @Retry(name = "processamentoPedido")
    @JmsListener(destination = Destinations.FILA_PEDIDOS, containerFactory = "queueListenerFactory")
    public void consumir(String payload) {
        // Processamento sujeito a falhas transientes.
    }
}
```

### 9.13. Observabilidade e monitoramento

Não existe resiliencia boa sem visibilidade operacional. O ideal é acompanhar:

- quantidade de retries;
- circuit breakers abertos;
- chamadas lentas ou expiradas;
- taxa de fallback;
- volume rejeitado por bulkhead ou rate limiter.

Isso ajuda a responder perguntas como:

- o serviço externo está instável ou apenas lento?
- o retry está ajudando ou piorando?
- o fallback está sendo usado demais?
- o gargalo está no provedor externo ou na própria aplicação?

### 9.14. Boas práticas

- aplique resiliência apenas onde existe dependência instável ou remota;
- diferencie erro transiente de erro funcional;
- prefira operações idempotentes quando houver retry;
- use timeout curto o suficiente para proteger a aplicação;
- não use fallback enganoso em fluxos críticos;
- monitore tudo o que puder degradar silenciosamente;
- revise periodicamente os parâmetros de retry, timeout e circuit breaker.


## 10. Cache e Spring Session

Cache e gerenciamento de sessão são dois temas muito relevantes em aplicações Spring Boot tradicionais. Embora resolvam problemas diferentes, eles frequentemente usam tecnologias parecidas, como Redis:

- cache melhora desempenho e reduz chamadas repetidas;
- sessão distribuída permite que o estado do usuário sobreviva entre instancias da aplicação.

Em geral:

- use cache para dados repetidos e relativamente estáveis;
- use Spring Session para estado de usuário em aplicações web stateful;
- evite misturar os dois conceitos no desenho de chaves e TTLs.

### 10.1. Dependências

Para suporte geral a cache:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

Para Caffeine:

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

Para cache com Redis e sessão distribuída:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

### 10.2. Habilitando cache

```java
package br.com.exemplo.cache;

import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CacheConfig {
}
```

Com `@EnableCaching`, o Spring passa a interceptar os métodos anotados com `@Cacheable`, `@CachePut` e `@CacheEvict`.

### 10.3. Uso de `@Cacheable`

`@Cacheable` e usado quando queremos:

- consultar primeiro no cache;
- executar o método apenas se a chave ainda não existir;
- armazenar o retorno para reutilização posterior.

Exemplo:

```java
package br.com.exemplo.cache;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProdutoConsultaService {

    private final ProdutoRepository produtoRepository;

    public ProdutoConsultaService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    @Cacheable(cacheNames = "produtos", key = "#id")
    public ProdutoDto buscarPorId(Long id) {
        return produtoRepository.findDtoById(id)
                .orElseThrow(() -> new IllegalArgumentException("Produto não encontrado"));
    }
}
```

Observações importantes:

- a primeira chamada consulta a fonte original;
- chamadas seguintes com a mesma chave usam o cache;
- o método precisa ser chamado via proxy Spring para a anotação funcionar.

Também e possível usar `unless` e `condition`:

```java
@Cacheable(cacheNames = "produtos", key = "#id", unless = "#result == null")
public ProdutoDto buscarPorId(Long id) {
    // ...
}
```

### 10.4. Uso de `@CachePut`

`@CachePut` sempre executa o método e atualiza o cache com o retorno mais recente.

Ele é útil quando:

- o dado acabou de ser alterado;
- queremos manter o cache sincronizado sem depender de nova leitura;
- o retorno do método já representa o estado final salvo.

Exemplo:

```java
package br.com.exemplo.cache;

import org.springframework.cache.annotation.CachePut;
import org.springframework.stereotype.Service;

@Service
public class ProdutoCadastroService {

    private final ProdutoRepository produtoRepository;

    public ProdutoCadastroService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    @CachePut(cacheNames = "produtos", key = "#result.id")
    public ProdutoDto atualizar(Long id, AtualizarProdutoRequest request) {
        Produto produto = produtoRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Produto não encontrado"));

        produto.setNome(request.nome());
        produto.setPreco(request.preco());

        Produto salvo = produtoRepository.save(produto);
        return new ProdutoDto(salvo.getId(), salvo.getNome(), salvo.getPreco());
    }
}
```

### 10.5. Uso de `@CacheEvict`

`@CacheEvict` remove entradas do cache, o que é importante quando o dado original mudou ou foi excluído.

Exemplo removendo um item:

```java
package br.com.exemplo.cache;

import org.springframework.cache.annotation.CacheEvict;
import org.springframework.stereotype.Service;

@Service
public class ProdutoRemocaoService {

    private final ProdutoRepository produtoRepository;

    public ProdutoRemocaoService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    @CacheEvict(cacheNames = "produtos", key = "#id")
    public void excluir(Long id) {
        produtoRepository.deleteById(id);
    }
}
```

Exemplo limpando o cache inteiro:

```java
@CacheEvict(cacheNames = "produtos", allEntries = true)
public void limparCacheProdutos() {
}
```

### 10.6. Configuração com Caffeine

Caffeine é uma excelente opção para cache local em memória:

- muito rápido;
- simples de operar;
- ideal para uma única instancia ou para caches que podem divergir brevemente entre instancias;
- não serve como cache compartilhado entre nós.

Configuração simples:

```yaml
spring:
  cache:
    type: caffeine
    cache-names: produtos,categorias
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=10m,recordStats
```

Também e possivel customizar via `CaffeineCacheManager`:

```java
package br.com.exemplo.cache;

import com.github.benmanes.caffeine.cache.Caffeine;
import java.util.concurrent.TimeUnit;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CaffeineCacheConfiguration {

    @Bean
    public CacheManager caffeineCacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager("produtos", "categorias");
        manager.setCaffeine(Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .recordStats());
        return manager;
    }
}
```

Quando preferir Caffeine:

- aplicação monolítica em uma instancia;
- dados muito consultados e pouco alterados;
- necessidade de latência mínima;
- cache local como camada complementar.

### 10.7. Configuração com Redis para cache

Redis é uma boa opção quando:

- há várias instancias da aplicação;
- o cache precisa ser compartilhado;
- o TTL precisa ser centralizado;
- a invalidação precisa refletir entre vários nós.

Configuração base:

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
  cache:
    type: redis
    cache-names: produtos,categorias
    redis:
      time-to-live: 10m
      cache-null-values: false
      use-key-prefix: true
```

Customização mais detalhada:

```java
package br.com.exemplo.cache;

import java.time.Duration;
import org.springframework.boot.autoconfigure.cache.RedisCacheManagerBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;

@Configuration
public class RedisCacheConfigurationCustom {

    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return builder -> builder
                .withCacheConfiguration("produtos", RedisCacheConfiguration.defaultCacheConfig()
                        .entryTtl(Duration.ofMinutes(10))
                        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(
                                new GenericJackson2JsonRedisSerializer()
                        )))
                .withCacheConfiguration("categorias", RedisCacheConfiguration.defaultCacheConfig()
                        .entryTtl(Duration.ofMinutes(30)));
    }
}
```

Pontos importantes:

- manter `use-key-prefix` ajuda a evitar colisão entre caches;
- diferentes caches podem ter TTLs diferentes;
- serialização JSON costuma ser mais amigável para manutenção e diagnostico.

### 10.8. Estratégias de invalidação

Cache sem estratégia de invalidação costuma gerar inconsistências.

Estratégias comuns:

- TTL curto para dados mais voláteis;
- `@CacheEvict` ao atualizar ou excluir dados;
- `@CachePut` para refresh imediato;
- `allEntries = true` para invalidações amplas;
- invalidação por evento de negócio;
- aquecimento de cache apos subida ou refresh.

Exemplo com evento:

```java
package br.com.exemplo.cache;

public record ProdutoAtualizadoEvent(Long produtoId) {
}
```

```java
package br.com.exemplo.cache;

import org.springframework.cache.CacheManager;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class ProdutoCacheInvalidationListener {

    private final CacheManager cacheManager;

    public ProdutoCacheInvalidationListener(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }

    @EventListener
    public void onProdutoAtualizado(ProdutoAtualizadoEvent event) {
        var cache = cacheManager.getCache("produtos");
        if (cache != null) {
            cache.evict(event.produtoId());
        }
    }
}
```

Boas práticas:

- não cachear tudo indiscriminadamente;
- separar cache de leitura de cache derivado;
- documentar TTL e estratégia de remoção;
- medir hit ratio antes de expandir uso de cache.

### 10.9. Cache em aplicações Spring Boot tradicionais

Em aplicações web tradicionais, o cache costuma ser aplicado em:

- consultas de catálogo;
- configurações de negócio;
- parâmetros e tabelas auxiliares;
- resultados de chamadas externas;
- dados frequentemente acessados por controllers e services.

Uma combinação prática é:

- Caffeine para cache local de baixa latência;
- Redis para cache compartilhado entre instancias;
- invalidação orientada a evento ou alteração de dados.

### 10.10. Spring Session e sessão distribuída

Spring Session abstrai o armazenamento da sessão HTTP e permite mover esse estado para um backend compartilhado, como Redis ou JDBC.

Isso é especialmente útil quando:

- há mais de uma instancia da aplicação;
- o load balancer não usa sticky session;
- o usuário precisa manter a sessão ao trocar de instancia;
- o estado da autenticação precisa sobreviver em ambiente distribuído.

Segundo a documentação oficial do Spring Boot, em aplicações servlet o store de sessão pode ser autoconfigurado com Redis ou JDBC, sem necessidade de `@Enable*HttpSession` quando há um único modulo Spring Session no classpath.

### 10.11. Configurando Spring Session com Redis

Exemplo para aplicação web tradicional:

```yaml
server:
  servlet:
    session:
      timeout: 30m

spring:
  data:
    redis:
      host: localhost
      port: 6379
  session:
    store-type: redis
    redis:
      namespace: spring:session:app
      flush-mode: on_save
```

Em muitos cenários, apenas adicionar a dependência `spring-session-data-redis` já faz o Spring Boot configurar automaticamente a substituição do `HttpSession`.

Exemplo de uso em controller:

```java
package br.com.exemplo.session;

import jakarta.servlet.http.HttpSession;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/sessao")
public class SessaoController {

    @GetMapping("/incrementar")
    public String incrementar(HttpSession session) {
        Integer contador = (Integer) session.getAttribute("contador");
        if (contador == null) {
            contador = 0;
        }

        contador++;
        session.setAttribute("contador", contador);

        return "Contador atual: " + contador;
    }
}
```

Nesse modelo:

- o container continua oferecendo `HttpSession`;
- por baixo, os dados passam a ser persistidos no backend configurado;
- a aplicação segue com a mesma API de sessão.

### 10.12. Persistência em Redis e integração com Spring Security

Em aplicações web com login tradicional, o Spring Session integra-se muito bem com Spring Security. Quando o `SecurityContext` e salvo na sessão, ele também passa a ser persistido no Redis.

Isso permite:

- manter autenticação entre instancias;
- invalidar sessões de forma centralizada;
- acompanhar sessões ativas de forma mais controlada.

Segundo a documentação do Spring Session, um cookie `SESSION` e usado para apontar para a sessão persistida. O backend Redis armazena os atributos da sessão e cuida da expiração conforme a configuração.

### 10.13. Estratégias para aplicações web tradicionais

Em aplicações web stateful, algumas estratégias são bastante comuns:

- sessão local do servlet container para ambientes simples e uma única instancia;
- sticky session no balanceador para reduzir troca de sessão entre nós;
- Spring Session com Redis para alta disponibilidade e escalabilidade horizontal;
- redução do tamanho da sessão para evitar excesso de serialização.

Recomendações práticas:

- manter na sessão apenas o necessário;
- não guardar objetos grandes ou grafo pesado;
- preferir IDs e dados essenciais em vez de entidades completas;
- configurar timeout coerente com a experiencia esperada;
- invalidar sessão explicitamente no logout.

### 10.14. Encontrando e invalidando sessões por usuário

Quando a aplicação precisa listar sessões ativas de um usuário ou derrubar sessões remotamente, o Spring Session com repositório indexada ajuda bastante.

Exemplo conceitual:

```java
package br.com.exemplo.session;

import java.util.Collection;
import org.springframework.session.FindByIndexNameSessionRepository;
import org.springframework.session.Session;
import org.springframework.stereotype.Service;

@Service
public class SessaoAdministrativaService {

    private final FindByIndexNameSessionRepository<? extends Session> sessions;

    public SessaoAdministrativaService(
            FindByIndexNameSessionRepository<? extends Session> sessions
    ) {
        this.sessions = sessions;
    }

    public Collection<? extends Session> buscarPorUsuario(String username) {
        return sessions.findByPrincipalName(username).values();
    }

    public void invalidarSessao(String sessionId) {
        sessions.deleteById(sessionId);
    }
}
```

Esse tipo de recurso é especialmente útil em:

- painéis administrativos;
- forcar logout;
- controle de sessões simultâneas;
- auditoria de acesso.

### 10.15. Eventos de sessão

Quando o repositório indexado está configurado, o Spring Session também pode publicar eventos como:

- `SessionCreatedEvent`;
- `SessionDeletedEvent`;
- `SessionDestroyedEvent`;
- `SessionExpiredEvent`.

Exemplo:

```java
package br.com.exemplo.session;

import org.springframework.context.event.EventListener;
import org.springframework.session.events.SessionCreatedEvent;
import org.springframework.session.events.SessionExpiredEvent;
import org.springframework.stereotype.Component;

@Component
public class SessionEventsListener {

    @EventListener
    public void onSessionCreated(SessionCreatedEvent event) {
        System.out.println("Sessao criada: " + event.getSessionId());
    }

    @EventListener
    public void onSessionExpired(SessionExpiredEvent event) {
        System.out.println("Sessao expirada: " + event.getSessionId());
    }
}
```

### 10.16. Boas práticas com cache e Spring Session

- escolha Caffeine para cache local e Redis para cache ou sessão compartilhada;
- não use cache como substituto de consistência de negócio;
- defina TTL e estratégia de invalidação antes de ativar o cache;
- monitore hit ratio, latência e volume de chaves;
- mantenha sessões pequenas e controladas;
- em aplicações distribuídas, prefira Spring Session com backend compartilhado;
- teste invalidação de cache e expiração de sessão em cenários reais;
- evite salvar objetos não serializáveis ou muito grandes na sessão.

## 11. Spring Batch

Spring Batch e o modulo do ecossistema Spring voltado para processamento em lote. Ele e muito útil quando a aplicação precisa processar grande volume de dados de forma controlada, reexecutavel e auditavel.

Casos comuns:

- importacao de arquivos;
- conciliacao financeira;
- processamento noturno;
- geracao de relatorios pesados;
- carga ou migracao de dados;
- reprocessamento de registros com falha.

Segundo a documentação oficial, alguns dos componentes centrais são:

- `JobRepository`, responsavel por persistir metadados de execução;
- `JobLauncher`, usado para disparar jobs com `JobParameters`;
- `ItemReader`, para ler itens;
- `ItemProcessor`, para transformar ou validar itens;
- `ItemWriter`, para gravar itens.

### 11.1. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

Se o projeto vai usar persistencia real dos metadados do Batch, normalmente também havera um banco com `spring-boot-starter-data-jdbc` ou `spring-boot-starter-data-jpa`, dependendo da arquitetura adotada.

### 11.2. Conceitos principais

#### `Job`

Representa o processo batch completo, por exemplo:

- importar pedidos;
- recalcular indicadores;
- consolidar faturamento.

#### `Step`

Representa uma etapa do job. Um job pode ter um ou varios steps.

Exemplos:

- ler um CSV;
- validar dados;
- salvar no banco;
- mover arquivo processado.

#### Chunk-oriented processing

E o modelo mais comum no Spring Batch. Nele:

- o `ItemReader` le um item por vez;
- o `ItemProcessor` transforma ou valida;
- o `ItemWriter` grava um lote de itens por chunk;
- a transacao normalmente e fechada por chunk.

Esse modelo e muito bom para alto volume com commit controlado.

### 11.3. Configuração basica de Job e Step

```java
package br.com.exemplo.batch;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
public class BatchConfig {

    @Bean
    public Job importacaoPedidosJob(
            JobRepository jobRepository,
            Step importarPedidosStep
    ) {
        return new JobBuilder("importacaoPedidosJob", jobRepository)
                .start(importarPedidosStep)
                .build();
    }

    @Bean
    public Step importarPedidosStep(
            JobRepository jobRepository,
            PlatformTransactionManager transactionManager,
            PedidoItemReader reader,
            PedidoItemProcessor processor,
            PedidoItemWriter writer
    ) {
        return new StepBuilder("importarPedidosStep", jobRepository)
                .<PedidoEntrada, PedidoProcessado>chunk(100, transactionManager)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }
}
```

### 11.4. Exemplo completo de `ItemReader`, `ItemProcessor` e `ItemWriter`

Objetos de exemplo:

```java
package br.com.exemplo.batch;

import java.math.BigDecimal;

public record PedidoEntrada(Long id, String cliente, BigDecimal valor) {
}
```

```java
package br.com.exemplo.batch;

import java.math.BigDecimal;

public record PedidoProcessado(Long id, String cliente, BigDecimal valorComTaxa) {
}
```

Reader simples em memória:

```java
package br.com.exemplo.batch;

import java.math.BigDecimal;
import java.util.Iterator;
import java.util.List;
import org.springframework.batch.item.support.AbstractItemStreamItemReader;
import org.springframework.stereotype.Component;

@Component
public class PedidoItemReader extends AbstractItemStreamItemReader<PedidoEntrada> {

    private final Iterator<PedidoEntrada> iterator = List.of(
            new PedidoEntrada(1L, "Maria", new BigDecimal("100.00")),
            new PedidoEntrada(2L, "Joao", new BigDecimal("200.00"))
    ).iterator();

    @Override
    public PedidoEntrada read() {
        return iterator.hasNext() ? iterator.next() : null;
    }
}
```

Processor:

```java
package br.com.exemplo.batch;

import java.math.BigDecimal;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.stereotype.Component;

@Component
public class PedidoItemProcessor implements ItemProcessor<PedidoEntrada, PedidoProcessado> {

    @Override
    public PedidoProcessado process(PedidoEntrada item) {
        if (item.valor().compareTo(BigDecimal.ZERO) <= 0) {
            return null;
        }

        BigDecimal valorComTaxa = item.valor().multiply(new BigDecimal("1.10"));
        return new PedidoProcessado(item.id(), item.cliente(), valorComTaxa);
    }
}
```

Quando o `ItemProcessor` retorna `null`, o item é filtrado e não segue para o writer.

Writer:

```java
package br.com.exemplo.batch;

import java.util.List;
import org.springframework.batch.item.Chunk;
import org.springframework.batch.item.ItemWriter;
import org.springframework.stereotype.Component;

@Component
public class PedidoItemWriter implements ItemWriter<PedidoProcessado> {

    @Override
    public void write(Chunk<? extends PedidoProcessado> chunk) {
        List<? extends PedidoProcessado> items = chunk.getItems();
        items.forEach(item -> System.out.println("Gravando pedido: " + item));
    }
}
```

### 11.5. Leitura de arquivo CSV com Spring Batch

Um dos cenarios mais comuns é importar dados de CSV com `FlatFileItemReader`.

```java
package br.com.exemplo.batch;

import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.FileSystemResource;

@Configuration
public class BatchCsvReaderConfig {

    @Bean
    public FlatFileItemReader<PedidoEntrada> pedidoFileReader() {
        return new FlatFileItemReaderBuilder<PedidoEntrada>()
                .name("pedidoFileReader")
                .resource(new FileSystemResource("dados/pedidos.csv"))
                .delimited()
                .names("id", "cliente", "valor")
                .targetType(PedidoEntrada.class)
                .linesToSkip(1)
                .build();
    }
}
```

Esse tipo de reader e bastante usado em importacoes administrativas ou integrações legadas.

### 11.6. Writer para banco de dados

Exemplo com `JdbcBatchItemWriter`:

```java
package br.com.exemplo.batch;

import javax.sql.DataSource;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.database.builder.JdbcBatchItemWriterBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BatchWriterConfig {

    @Bean
    public JdbcBatchItemWriter<PedidoProcessado> pedidoDatabaseWriter(DataSource dataSource) {
        return new JdbcBatchItemWriterBuilder<PedidoProcessado>()
                .dataSource(dataSource)
                .sql("""
                        insert into pedido_processado (id, cliente, valor_com_taxa)
                        values (:id, :cliente, :valorComTaxa)
                        """)
                .beanMapped()
                .build();
    }
}
```

### 11.7. Restart e reprocessamento

Um dos grandes diferenciais do Spring Batch é a persistencia dos metadados de execução no `JobRepository`. Isso permite:

- saber o que ja rodou;
- identificar falhas por job e step;
- retomar execução quando o job e reiniciavel;
- evitar reprocessamento indevido do mesmo `JobInstance`.

Na pratica:

- um `JobInstance` e identificado pelo nome do job e seus `JobParameters`;
- mudar os parâmetros cria uma nova instancia;
- relancar com os mesmos parâmetros pode resultar em restart, se o job permitir.

Exemplo de disparo com parâmetro:

```java
package br.com.exemplo.batch;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.stereotype.Service;

@Service
public class BatchJobService {

    private final JobLauncher jobLauncher;
    private final Job importacaoPedidosJob;

    public BatchJobService(JobLauncher jobLauncher, Job importacaoPedidosJob) {
        this.jobLauncher = jobLauncher;
        this.importacaoPedidosJob = importacaoPedidosJob;
    }

    public void executarImportacao() throws Exception {
        JobParameters parameters = new JobParametersBuilder()
                .addLong("timestamp", System.currentTimeMillis())
                .toJobParameters();

        jobLauncher.run(importacaoPedidosJob, parameters);
    }
}
```

Se o objetivo for sempre criar uma nova execução, adicionar um `timestamp` ou identificador unico nos parâmetros e uma abordagem comum.

### 11.8. Skip e retry

Em importacoes reais, alguns registros podem vir invalidos e outros podem falhar temporariamente. O Spring Batch suporta ambos os cenarios.

Exemplo com `skip` e `retry`:

```java
package br.com.exemplo.batch;

import org.springframework.batch.core.Step;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.file.FlatFileParseException;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.dao.DeadlockLoserDataAccessException;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
public class BatchFaultToleranceConfig {

    @Bean
    public Step importarPedidosComToleranciaStep(
            JobRepository jobRepository,
            PlatformTransactionManager transactionManager,
            PedidoItemReader reader,
            PedidoItemProcessor processor,
            PedidoItemWriter writer
    ) {
        return new StepBuilder("importarPedidosComToleranciaStep", jobRepository)
                .<PedidoEntrada, PedidoProcessado>chunk(50, transactionManager)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .faultTolerant()
                .skipLimit(10)
                .skip(FlatFileParseException.class)
                .retryLimit(3)
                .retry(DeadlockLoserDataAccessException.class)
                .build();
    }
}
```

Interpretacao:

- erros de parse no arquivo podem ser ignorados ate o limite definido;
- falhas transientes de banco podem ser tentadas novamente;
- se o limite for ultrapassado, o step falha.

### 11.9. Tasklet step

Nem todo processamento batch precisa de chunk. Para tarefas simples e pontuais, pode-se usar `Tasklet`.

Exemplo:

```java
package br.com.exemplo.batch;

import org.springframework.batch.core.Step;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
public class BatchTaskletConfig {

    @Bean
    public Step moverArquivoProcessadoStep(
            JobRepository jobRepository,
            PlatformTransactionManager transactionManager
    ) {
        return new StepBuilder("moverArquivoProcessadoStep", jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("Movendo arquivo processado para pasta de historico");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }
}
```

Use `Tasklet` quando:

- a etapa for simples;
- não houver processamento item a item;
- a logica for unica, como mover arquivo, limpar diretorio ou gerar marcador de execução.

### 11.10. Boas praticas com Spring Batch

- persista metadados do Batch em banco relacional;
- mantenha jobs idempotentes sempre que possivel;
- use `JobParameters` conscientemente para distinguir instancias;
- prefira chunk para alto volume e `Tasklet` para tarefas pontuais;
- trate skip e retry com regras de negocio claras;
- registre metricas e auditoria das execucoes;
- para jobs pesados, considere disparo por scheduler ou operacao administrativa;
- para arquivos grandes, combine Spring Batch com leitores aprópriados, como CSV ou leitura streaming de XLSX.

### 11.11. Quando Spring Batch faz mais sentido

Spring Batch costuma ser uma excelente escolha quando:

- há volume relevante de dados;
- o processamento precisa de restart controlado;
- existe necessidade de trilha de execução;
- o fluxo precisa de chunk, skip, retry e gerenciamento de falha.

Em contrapartida, talvez ele seja excessivo quando:

- a tarefa é muito simples e eventual;
- um service comum ou job Quartz resolve com menos complexidade;
- não há estado de execução, checkpoint ou necessidade de restart.

## 12. Exportação de dados em CSV e Excel

Exportação de dados é uma necessidade muito comum em sistemas corporativos, especialmente para:

- relatorios administrativos;
- extracao de dados para analise;
- integracao manual com planilhas;
- conferencias operacionais;
- disponibilizacao de dados para usuarios de negocio.

Em Spring Boot, duas bibliotecas muito comuns para esse cenario são:

- Apache Commons CSV para geracao de arquivos CSV;
- Apache POI para geracao de arquivos Excel, principalmente `.xlsx`.

Conforme a documentação oficial do Apache Commons CSV, a biblioteca é voltada para leitura e escrita de variacoes do formato CSV. Ja o Apache POI informa oficialmente que `poi` cobre o ecossistema OLE2 e `poi-ooxml` cobre os formatos OOXML, incluindo Excel `.xlsx`.

### 12.1. Dependências

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-csv</artifactId>
        <version><!-- versao atual do projeto --></version>
    </dependency>

    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version><!-- versao atual do projeto --></version>
    </dependency>

    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version><!-- versao atual do projeto --></version>
    </dependency>
</dependencies>
```

Observacoes:

- para CSV, normalmente basta `commons-csv`;
- para Excel `.xlsx`, normalmente basta `poi-ooxml`, que ja puxa o necessario para OOXML;
- manter `poi` explicito pode ser útil para deixar claro o suporte ao ecossistema base do POI.

### 12.2. Quando usar CSV e quando usar Excel

Use CSV quando:

- o objetivo for simplicidade;
- o arquivo for consumido por varios sistemas;
- não houver necessidade de formatacao visual;
- o volume de dados for grande e o arquivo precisar ser leve.

Use Excel quando:

- houver necessidade de multiplas abas;
- for importante aplicar formatacao, estilos e larguras de coluna;
- usuarios de negocio esperarem uma planilha mais amigavel;
- houver necessidade de formulas, filtros ou organizacao visual.

### 12.3. Exemplo de DTO para exportação

```java
package br.com.exemplo.exportacao;

import java.math.BigDecimal;
import java.time.LocalDate;

public record PedidoExportacaoDto(
        Long id,
        String cliente,
        BigDecimal valorTotal,
        LocalDate dataCriacao,
        String status
) {
}
```

### 12.4. Exportando CSV com Apache Commons CSV

Servico:

```java
package br.com.exemplo.exportacao;

import java.io.ByteArrayOutputStream;
import java.io.OutputStreamWriter;
import java.nio.charset.StandardCharsets;
import java.util.List;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.springframework.stereotype.Service;

@Service
public class PedidoCsvExporter {

    public byte[] exportar(List<PedidoExportacaoDto> pedidos) throws Exception {
        try (
                ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
                OutputStreamWriter writer = new OutputStreamWriter(outputStream, StandardCharsets.UTF_8);
                CSVPrinter csvPrinter = new CSVPrinter(writer, CSVFormat.DEFAULT.builder()
                        .setHeader("ID", "CLIENTE", "VALOR_TOTAL", "DATA_CRIACAO", "STATUS")
                        .setDelimiter(';')
                        .build())
        ) {
            for (PedidoExportacaoDto pedido : pedidos) {
                csvPrinter.printRecord(
                        pedido.id(),
                        pedido.cliente(),
                        pedido.valorTotal(),
                        pedido.dataCriacao(),
                        pedido.status()
                );
            }

            csvPrinter.flush();
            return outputStream.toByteArray();
        }
    }
}
```

Pontos importantes:

- o delimitador pode ser `,` ou `;`, dependendo do padrão esperado;
- em cenarios brasileiros, `;` e comum por compatibilidade com Excel em algumas configuracoes regionais;
- usar UTF-8 evita problemas com acentos;
- `CSVPrinter` ajuda a lidar corretamente com escape e aspas.

### 12.5. Lendo arquivos CSV com Apache Commons CSV

Para importacao de dados, o Apache Commons CSV também oferece uma API simples e segura para parsing de arquivos CSV.

Exemplo de leitura a partir de upload:

```java
package br.com.exemplo.exportacao;

import java.io.InputStreamReader;
import java.math.BigDecimal;
import java.nio.charset.StandardCharsets;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVParser;
import org.apache.commons.csv.CSVRecord;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class PedidoCsvReader {

    public List<PedidoExportacaoDto> ler(MultipartFile file) throws Exception {
        try (
                InputStreamReader reader = new InputStreamReader(file.getInputStream(), StandardCharsets.UTF_8);
                CSVParser parser = CSVFormat.DEFAULT.builder()
                        .setHeader()
                        .setSkipHeaderRecord(true)
                        .setDelimiter(';')
                        .build()
                        .parse(reader)
        ) {
            List<PedidoExportacaoDto> pedidos = new ArrayList<>();

            for (CSVRecord record : parser) {
                pedidos.add(new PedidoExportacaoDto(
                        Long.valueOf(record.get("ID")),
                        record.get("CLIENTE"),
                        new BigDecimal(record.get("VALOR_TOTAL")),
                        LocalDate.parse(record.get("DATA_CRIACAO")),
                        record.get("STATUS")
                ));
            }

            return pedidos;
        }
    }
}
```

Boas praticas na leitura de CSV:

- validar encoding, normalmente `UTF-8`;
- deixar explicito o delimitador esperado;
- validar cabecalhos antes de processar;
- tratar linhas vazias, colunas faltantes e valores invalidos;
- registrar erros de importacao com numero da linha quando possivel.

### 12.6. Endpoint HTTP para download de CSV

```java
package br.com.exemplo.exportacao;

import java.util.List;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/exportacoes")
public class ExportacaoController {

    private final PedidoCsvExporter pedidoCsvExporter;
    private final PedidoExcelExporter pedidoExcelExporter;

    public ExportacaoController(
            PedidoCsvExporter pedidoCsvExporter,
            PedidoExcelExporter pedidoExcelExporter
    ) {
        this.pedidoCsvExporter = pedidoCsvExporter;
        this.pedidoExcelExporter = pedidoExcelExporter;
    }

    @GetMapping(value = "/pedidos.csv", produces = "text/csv")
    public ResponseEntity<byte[]> exportarCsv() throws Exception {
        List<PedidoExportacaoDto> pedidos = buscarPedidos();
        byte[] arquivo = pedidoCsvExporter.exportar(pedidos);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=pedidos.csv")
                .contentType(new MediaType("text", "csv"))
                .body(arquivo);
    }

    @GetMapping("/pedidos.xlsx")
    public ResponseEntity<byte[]> exportarExcel() throws Exception {
        List<PedidoExportacaoDto> pedidos = buscarPedidos();
        byte[] arquivo = pedidoExcelExporter.exportar(pedidos);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=pedidos.xlsx")
                .contentType(MediaType.parseMediaType(
                        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
                ))
                .body(arquivo);
    }

    private List<PedidoExportacaoDto> buscarPedidos() {
        return List.of(
                new PedidoExportacaoDto(1L, "Maria", java.math.BigDecimal.valueOf(120.50), java.time.LocalDate.now(), "FATURADO"),
                new PedidoExportacaoDto(2L, "Joao", java.math.BigDecimal.valueOf(89.90), java.time.LocalDate.now(), "PENDENTE")
        );
    }
}
```

Esse controller mostra os dois formatos no mesmo endpoint de exportação.

### 12.7. Exportando Excel com Apache POI

Exemplo usando `XSSFWorkbook` para gerar `.xlsx`:

```java
package br.com.exemplo.exportacao;

import java.io.ByteArrayOutputStream;
import java.util.List;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.CreationHelper;
import org.apache.poi.ss.usermodel.Font;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.stereotype.Service;

@Service
public class PedidoExcelExporter {

    public byte[] exportar(List<PedidoExportacaoDto> pedidos) throws Exception {
        try (
                XSSFWorkbook workbook = new XSSFWorkbook();
                ByteArrayOutputStream outputStream = new ByteArrayOutputStream()
        ) {
            Sheet sheet = workbook.createSheet("Pedidos");

            CellStyle headerStyle = workbook.createCellStyle();
            Font headerFont = workbook.createFont();
            headerFont.setBold(true);
            headerStyle.setFont(headerFont);

            CellStyle dateStyle = workbook.createCellStyle();
            CreationHelper creationHelper = workbook.getCreationHelper();
            dateStyle.setDataFormat(
                    creationHelper.createDataFormat().getFormat("dd/mm/yyyy")
            );

            Row header = sheet.createRow(0);
            header.createCell(0).setCellValue("ID");
            header.createCell(1).setCellValue("CLIENTE");
            header.createCell(2).setCellValue("VALOR_TOTAL");
            header.createCell(3).setCellValue("DATA_CRIACAO");
            header.createCell(4).setCellValue("STATUS");

            for (int i = 0; i < 5; i++) {
                header.getCell(i).setCellStyle(headerStyle);
            }

            int rowIndex = 1;
            for (PedidoExportacaoDto pedido : pedidos) {
                Row row = sheet.createRow(rowIndex++);
                row.createCell(0).setCellValue(pedido.id());
                row.createCell(1).setCellValue(pedido.cliente());
                row.createCell(2).setCellValue(pedido.valorTotal().doubleValue());

                var dataCell = row.createCell(3);
                dataCell.setCellValue(java.sql.Date.valueOf(pedido.dataCriacao()));
                dataCell.setCellStyle(dateStyle);

                row.createCell(4).setCellValue(pedido.status());
            }

            for (int i = 0; i < 5; i++) {
                sheet.autoSizeColumn(i);
            }

            workbook.write(outputStream);
            return outputStream.toByteArray();
        }
    }
}
```

Esse exemplo cobre:

- criacao de workbook e sheet;
- cabecalho formatado;
- celulas de data com estilo;
- ajuste automatico de largura das colunas.

### 12.8. Lendo arquivos XLSX com Apache POI

Quando o arquivo Excel não é muito grande, `XSSFWorkbook` ou `WorkbookFactory` costumam ser suficientes para leitura.

Exemplo com `WorkbookFactory`:

```java
package br.com.exemplo.exportacao;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.DataFormatter;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.usermodel.WorkbookFactory;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class PedidoExcelReader {

    public List<PedidoExportacaoDto> ler(MultipartFile file) throws Exception {
        try (Workbook workbook = WorkbookFactory.create(file.getInputStream())) {
            Sheet sheet = workbook.getSheetAt(0);
            List<PedidoExportacaoDto> pedidos = new ArrayList<>();
            DataFormatter formatter = new DataFormatter();

            boolean cabecalho = true;
            for (Row row : sheet) {
                if (cabecalho) {
                    cabecalho = false;
                    continue;
                }

                if (row == null) {
                    continue;
                }

                Cell idCell = row.getCell(0);
                if (idCell == null) {
                    continue;
                }

                Long id = (long) idCell.getNumericCellValue();
                String cliente = formatter.formatCellValue(row.getCell(1));
                BigDecimal valorTotal = BigDecimal.valueOf(row.getCell(2).getNumericCellValue());
                LocalDate dataCriacao = row.getCell(3).getLocalDateTimeCellValue().toLocalDate();
                String status = formatter.formatCellValue(row.getCell(4));

                pedidos.add(new PedidoExportacaoDto(id, cliente, valorTotal, dataCriacao, status));
            }

            return pedidos;
        }
    }
}
```

Esse modelo funciona bem quando:

- o arquivo tem tamanho pequeno ou medio;
- é necessario aproveitar melhor a API completa do POI;
- o processamento exige mais liberdade de navegacao pela planilha.

Para leitura, `DataFormatter` é útil porque ajuda a obter o valor textual exibido na celula sem precisar tratar todos os tipos manualmente em cada caso.

### 12.9. Diferencas entre `XSSFWorkbook` e `SXSSFWorkbook`

Na prática, os dois servem para gerar arquivos `.xlsx`, mas o comportamento é o custo de memória são diferentes.

#### Quando usar `XSSFWorkbook`

`XSSFWorkbook` e a implementacao tradicional do Apache POI para arquivos `.xlsx`. Ela carrega e manipula a planilha em memória, o que a torna mais simples e mais flexivel.

Use `XSSFWorkbook` quando:

- o volume de linhas for moderado;
- houver necessidade de leitura e escrita com acesso aleatorio;
- o relatorio usar muitos recursos do Excel;
- for importante aplicar estilos, formulas, comentarios, merges e outras operacoes mais ricas;
- a simplicidade do código for mais importante que a economia maxima de memória.

Vantagens:

- API mais direta;
- acesso completo a linhas e celulas ja criadas;
- melhor para planilhas pequenas e medias;
- mais confortavel para cenarios com bastante formatacao.

Limites práticos:

- consome mais memória;
- pode ficar pesado em exportacoes muito grandes;
- para arquivos extensos, o risco de lentidao e `OutOfMemoryError` aumenta.

#### Quando usar `SXSSFWorkbook`

`SXSSFWorkbook` é a versao streaming do `XSSFWorkbook`. Segundo a documentação oficial do Apache POI, ele permite gravar arquivos muito grandes sem esgotar memória, mantendo em memória apenas uma janela configuravel de linhas.

Use `SXSSFWorkbook` quando:

- a exportação tiver muitas linhas;
- o objetivo principal for escrita sequencial;
- a aplicação precisar reduzir consumo de memória;
- o relatorio for grande, mas relativamente simples do ponto de vista visual.

Vantagens:

- muito mais adequado para grandes volumes;
- mantém apenas parte das linhas em memória;
- reduz risco de estouro de memória em exportacoes extensas.

Cuidados importantes:

- depois que linhas antigas são descarregadas, elas não ficam mais disponiveis para acesso aleatorio;
- o POI usa arquivos temporarios em disco durante a geracao;
- recursos como comentarios e regiões mescladas ainda podem consumir memória significativa;
- ao final, é importante limpar os arquivos temporarios com `dispose()`.

#### Resumo pratico

`XSSFWorkbook`:

- melhor para arquivos pequenos e médios;
- melhor quando há muita formatacao e manipulacao posterior;
- mais simples para relatorios ricos.

`SXSSFWorkbook`:

- melhor para exportação massiva;
- melhor para escrita sequencial de grandes volumes;
- melhor quando memória e um fator critico.

Uma regra pratica bastante usada:

- ate algumas milhares de linhas, `XSSFWorkbook` costuma ser suficiente;
- para dezenas ou centenas de milhares de linhas, vale avaliar `SXSSFWorkbook`.

Essa regra e apenas uma referencia. O ponto real de troca depende da memória disponivel, da quantidade de colunas e da complexidade da planilha.

### 12.10. Exemplo de exportação com `SXSSFWorkbook`

```java
package br.com.exemplo.exportacao;

import java.io.ByteArrayOutputStream;
import java.util.List;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.springframework.stereotype.Service;

@Service
public class PedidoExcelStreamingExporter {

    public byte[] exportar(List<PedidoExportacaoDto> pedidos) throws Exception {
        SXSSFWorkbook workbook = new SXSSFWorkbook(100);

        try (
                workbook;
                ByteArrayOutputStream outputStream = new ByteArrayOutputStream()
        ) {
            workbook.setCompressTempFiles(true);

            Sheet sheet = workbook.createSheet("Pedidos");

            Row header = sheet.createRow(0);
            header.createCell(0).setCellValue("ID");
            header.createCell(1).setCellValue("CLIENTE");
            header.createCell(2).setCellValue("VALOR_TOTAL");
            header.createCell(3).setCellValue("DATA_CRIACAO");
            header.createCell(4).setCellValue("STATUS");

            int rowIndex = 1;
            for (PedidoExportacaoDto pedido : pedidos) {
                Row row = sheet.createRow(rowIndex++);
                row.createCell(0).setCellValue(pedido.id());
                row.createCell(1).setCellValue(pedido.cliente());
                row.createCell(2).setCellValue(pedido.valorTotal().doubleValue());
                row.createCell(3).setCellValue(String.valueOf(pedido.dataCriacao()));
                row.createCell(4).setCellValue(pedido.status());
            }

            workbook.write(outputStream);
            return outputStream.toByteArray();
        } finally {
            workbook.dispose();
        }
    }
}
```

Nesse exemplo:

- `new SXSSFWorkbook(100)` mantem apenas uma janela de 100 linhas em memória;
- `setCompressTempFiles(true)` reduz o impacto dos arquivos temporarios em disco;
- `dispose()` remove os arquivos temporarios criados pelo streaming workbook.

Para exportacoes muito grandes, esse modelo costuma ser mais seguro que `XSSFWorkbook`.

### 12.11. Leitura de arquivos grandes com `excel-streaming-reader`

Quando o problema não e exportar, mas sim importar ou ler planilhas `.xlsx` muito grandes, uma opcao bastante recomendada e o fork mantido por `pjfanning` da biblioteca `excel-streaming-reader`.

Esse fork e a evolucao mais atual da implementacao original e, conforme o README oficial, suporta Apache POI 5.x e Java 8+, alem de trazer correcoes e recursos adicionais.

Ela funciona como um wrapper sobre a leitura streaming do Apache POI e preserva uma API parecida com `Workbook`, `Sheet`, `Row` e `Cell`, mas com foco em baixo consumo de memória.

Ela costuma ser útil quando:

- o arquivo Excel recebido tem muitas linhas;
- a aplicação precisa importar planilhas grandes;
- não e viavel carregar o workbook inteiro em memória;
- o fluxo de leitura pode ser sequencial.

Segundo o README oficial:

- a biblioteca suporta apenas arquivos `.xlsx`;
- ela foi feita para leitura streaming, não para escrita;
- o acesso aleatorio a linhas não esta disponivel como em um workbook tradicional;
- nem todos os metodos do POI são suportados;
- o pacote Java mudou em relacao ao projeto original;
- o fork atual oferece opcoes extras para shared strings, comments e suporte melhorado a formatos OOXML Strict.

#### Dependência

```xml
<dependency>
    <groupId>com.github.pjfanning</groupId>
    <artifactId>excel-streaming-reader</artifactId>
    <version>5.2.0</version>
</dependency>
```

Se o projeto usar algumas implementacoes avancadas de shared strings, o README oficial informa que pode ser necessario adicionar também a dependência `poi-shared-strings`.

#### Exemplo de leitura streaming

```java
package br.com.exemplo.exportacao;

import com.github.pjfanning.xlsx.StreamingReader;
import java.io.InputStream;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

@Service
public class PedidoExcelStreamingReaderService {

    public List<PedidoExportacaoDto> ler(MultipartFile file) throws Exception {
        List<PedidoExportacaoDto> pedidos = new ArrayList<>();

        try (
                InputStream inputStream = file.getInputStream();
                Workbook workbook = StreamingReader.builder()
                        .rowCacheSize(100)
                        .bufferSize(4096)
                        .open(inputStream)
        ) {
            Sheet sheet = workbook.getSheetAt(0);
            Iterator<Row> rowIterator = sheet.iterator();

            if (rowIterator.hasNext()) {
                rowIterator.next();
            }

            while (rowIterator.hasNext()) {
                Row row = rowIterator.next();

                Long id = (long) row.getCell(0).getNumericCellValue();
                String cliente = row.getCell(1).getStringCellValue();
                BigDecimal valorTotal = BigDecimal.valueOf(row.getCell(2).getNumericCellValue());
                LocalDate dataCriacao = row.getCell(3).getLocalDateTimeCellValue().toLocalDate();
                String status = row.getCell(4).getStringCellValue();

                pedidos.add(new PedidoExportacaoDto(id, cliente, valorTotal, dataCriacao, status));
            }
        }

        return pedidos;
    }
}
```

Nesse exemplo:

- `rowCacheSize(100)` define quantas linhas ficam em memória;
- `bufferSize(4096)` define o buffer usado ao copiar o fluxo para arquivo temporario;
- o workbook deve ser fechado corretamente para liberar os recursos temporarios.

#### Exemplo com configuracoes extras para arquivos muito grandes

O fork `pjfanning` adiciona opcoes uteis para shared strings e comments, o que pode ajudar bastante em arquivos grandes.

```java
package br.com.exemplo.exportacao;

import com.github.pjfanning.poi.SharedStringsImplementationType;
import com.github.pjfanning.xlsx.StreamingReader;
import java.io.InputStream;
import org.apache.poi.ss.usermodel.Workbook;

public class ExcelStreamingAdvancedExample {

    public Workbook abrir(InputStream inputStream) throws Exception {
        return StreamingReader.builder()
                .rowCacheSize(100)
                .bufferSize(4096)
                .setSharedStringsImplementationType(SharedStringsImplementationType.TEMP_FILE_BACKED)
                .setEncryptSstTempFile(false)
                .open(inputStream);
    }
}
```

Esse modelo pode ser interessante quando:

- o arquivo possui muitas strings repetidas;
- o consumo de memória com shared strings comeca a crescer;
- a aplicação aceita usar arquivos temporarios para aliviar RAM.

#### Comparando com `XSSFWorkbook`

Use `excel-streaming-reader` quando:

- o objetivo principal for leitura de arquivos `.xlsx` grandes;
- a leitura puder ser sequencial;
- o baixo consumo de memória for prioridade.

Use `XSSFWorkbook` quando:

- o arquivo for pequeno ou medio;
- houver necessidade de recursos mais completos do POI;
- o código precisar de acesso mais livre ao workbook;
- o fluxo envolver leitura e manipulacao rica da planilha.

#### Limitacoes importantes

- funciona apenas com `.xlsx`;
- não serve para gerar planilhas;
- alguns recursos avancados do POI não estao disponiveis;
- como a leitura e streaming, linhas antigas não ficam livremente acessiveis;
- a biblioteca usa arquivo temporario ao trabalhar com `InputStream`;
- para arquivos muito grandes, o próprio README recomenda favorecer o uso de arquivos temporarios.

Na prática:

- `XSSFWorkbook` é melhor para leitura rica e planilhas menores;
- `SXSSFWorkbook` é melhor para escrita streaming de grandes arquivos;
- `excel-streaming-reader` e melhor para leitura streaming de `.xlsx` grandes.

### 12.12. Comparando abordagens de leitura para XLSX

Resumo prática:

- `WorkbookFactory` ou `XSSFWorkbook`: melhor para arquivos pequenos e medios, com API completa;
- `excel-streaming-reader`: melhor para leitura de arquivos grandes com menor consumo de memória;
- `SXSSFWorkbook`: não e biblioteca de leitura; ele e voltado para escrita streaming.

Escolha rápida:

- importar planilha pequena ou média: `XSSFWorkbook` ou `WorkbookFactory`;
- importar planilha muito grande: `excel-streaming-reader`;
- gerar planilha muito grande: `SXSSFWorkbook`.

### 12.13. Exportação com mais de uma aba

Quando o relatório é mais complexo, uma planilha Excel pode conter varias abas.

Exemplo conceitual:

```java
Sheet resumoSheet = workbook.createSheet("Resumo");
Sheet pedidosSheet = workbook.createSheet("Pedidos");
Sheet auditoriaSheet = workbook.createSheet("Auditoria");
```

Isso é útil quando:

- o arquivo precisa separar visões do mesmo relatorio;
- existem dados analiticos e resumo executivo;
- o usuário precisa navegar por categorias ou períodos.

### 12.14. Cuidados com memória e volume

Arquivos CSV geralmente são leves, mas planilhas Excel podem consumir bastante memória, principalmente com muitas linhas e estilos.

Boas praticas:

- para grandes volumes, preferir CSV quando possível;
- limitar quantidade de estilos diferentes no Excel;
- evitar manter muitos arquivos grandes em memória ao mesmo tempo;
- considerar exportação assíncrona para arquivos muito grandes;
- para escrita massiva em `.xlsx`, avaliar `SXSSFWorkbook`;
- lembrar que `SXSSFWorkbook` cria arquivos temporários e exige limpeza adequada.

### 12.15. Diferencas práticas entre CSV e Excel

CSV:

- mais simples;
- mais leve;
- mais fácil para integração entre sistemas;
- sem estilos, abas ou fórmulas.

Excel:

- mais amigável para usuários finais;
- suporta formatação, fórmulas e várias abas;
- normalmente mais pesado;
- mais apropriado para relatórios gerenciais.

### 12.16. Boas práticas para exportação

- usar nomes de arquivo claros, por exemplo `pedidos-2026-03-19.xlsx`;
- aplicar `Content-Disposition: attachment`;
- definir o `Content-Type` correto;
- evitar expor colunas sensíveis sem necessidade;
- considerar paginação, filtros e exportação assíncrona para grandes volumes;
- registrar auditoria quando a exportação envolver dados sensíveis.

## 13. Upload e download de arquivos

Upload e download de arquivos são requisitos muito comuns em aplicações Spring Boot tradicionais, especialmente em cenários como:

- envio de documentos;
- anexos de processos;
- importação manual de arquivos;
- download de comprovantes, relatórios e templates;
- armazenamento local ou em serviços externos.

Os cuidados principais normalmente envolvem:

- validação de tipo e tamanho;
- segurança do nome do arquivo;
- streaming para não sobrecarregar memória;
- estratégia de armazenamento;
- autorização de acesso ao arquivo.

### 13.1. Suporte básico a multipart no Spring Boot

Em aplicações servlet, o Spring Boot configura automaticamente o suporte a upload multipart quando o suporte multipart está habilitado e as classes necessárias estão no classpath.

Configuração comum:

```yaml
spring:
  servlet:
    multipart:
      enabled: true
      max-file-size: 10MB
      max-request-size: 20MB
      file-size-threshold: 2MB
      location: ${java.io.tmpdir}
```

Na pratica:

- `max-file-size` limita cada arquivo;
- `max-request-size` limita o total da requisição multipart;
- `file-size-threshold` define quando o upload vai para disco temporário;
- `location` define onde os temporários podem ser gravados.

### 13.2. Upload simples com `MultipartFile`

Exemplo de controller:

```java
package br.com.exemplo.arquivos;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/arquivos")
public class FileUploadController {

    private final FileStorageService fileStorageService;

    public FileUploadController(FileStorageService fileStorageService) {
        this.fileStorageService = fileStorageService;
    }

    @PostMapping("/upload")
    public UploadResponse upload(@RequestParam("file") MultipartFile file) throws Exception {
        ArquivoSalvo arquivo = fileStorageService.salvar(file);
        return new UploadResponse(arquivo.nomeOriginal(), arquivo.caminho());
    }
}
```

DTOs:

```java
package br.com.exemplo.arquivos;

public record UploadResponse(String nomeOriginal, String caminho) {
}
```

```java
package br.com.exemplo.arquivos;

public record ArquivoSalvo(String nomeOriginal, String caminho) {
}
```

### 13.3. Armazenamento local de arquivos

Serviço de armazenamento local:

```java
package br.com.exemplo.arquivos;

import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.UUID;
import org.springframework.stereotype.Service;
import org.springframework.útil.StringUtils;
import org.springframework.web.multipart.MultipartFile;

@Service
public class FileStorageService {

    private final Path diretorioBase = Paths.get("storage");

    public ArquivoSalvo salvar(MultipartFile file) throws Exception {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("Arquivo vazio");
        }

        Files.createDirectories(diretorioBase);

        String nomeOriginal = StringUtils.cleanPath(file.getOriginalFilename());
        String extensao = extrairExtensao(nomeOriginal);
        String nomeGerado = UUID.randomUUID() + extensao;
        Path destino = diretorioBase.resolve(nomeGerado).normalize();

        if (!destino.startsWith(diretorioBase)) {
            throw new IllegalArgumentException("Caminho de arquivo invalido");
        }

        try (InputStream inputStream = file.getInputStream()) {
            Files.copy(inputStream, destino, StandardCopyOption.REPLACE_EXISTING);
        }

        return new ArquivoSalvo(nomeOriginal, destino.toString());
    }

    public Path obter(String nomeArquivo) {
        Path arquivo = diretorioBase.resolve(nomeArquivo).normalize();
        if (!arquivo.startsWith(diretorioBase)) {
            throw new IllegalArgumentException("Caminho de arquivo invalido");
        }
        return arquivo;
    }

    private String extrairExtensao(String nomeOriginal) {
        int idx = nomeOriginal.lastIndexOf('.');
        return idx >= 0 ? nomeOriginal.substring(idx) : "";
    }
}
```

Esse modelo é útil quando:

- o volume de arquivos e moderado;
- a aplicação roda em um ambiente com disco acessível;
- não há necessidade imediata de armazenamento externo.

### 13.4. Validação de tipo e tamanho

Validações importantes no upload:

- tamanho máximo;
- content type esperado;
- extensão permitida;
- nome seguro;
- conteúdo realmente compatível com o tipo declarado.

Exemplo simples:

```java
package br.com.exemplo.arquivos;

import java.util.Set;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

@Component
public class FileValidationService {

    private static final Set<String> TIPOS_PERMITIDOS = Set.of(
            "application/pdf",
            "image/png",
            "image/jpeg"
    );

    public void validar(MultipartFile file) {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("Arquivo vazio");
        }

        if (file.getSize() > 10 * 1024 * 1024) {
            throw new IllegalArgumentException("Arquivo acima do tamanho permitido");
        }

        if (!TIPOS_PERMITIDOS.contains(file.getContentType())) {
            throw new IllegalArgumentException("Tipo de arquivo não permitido");
        }
    }
}
```

Em cenários sensíveis, o ideal é validar também a assinatura binária do arquivo e não apenas a extensão ou o content type informado pelo cliente. A biblioteca Apache Tika resolve exatamente isso.

### 13.5. Validação de conteúdo com Apache Tika

O `Content-Type` enviado pelo cliente em um upload é definido pelo próprio cliente e pode ser facilmente falsificado — um arquivo `.exe` renomeado para `.pdf` passaria sem problemas por qualquer verificação baseada somente em extensão ou cabeçalho HTTP. O **Apache Tika** detecta o tipo real do arquivo inspecionando seus **magic bytes** (sequência de bytes no início do arquivo que identifica o formato), independentemente do nome ou do `Content-Type` declarado.

#### Dependências

Para detecção de tipo apenas (uso mais comum em APIs):

```xml
<!-- Detecção de tipo MIME por magic bytes — leve, sem dependências externas -->
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>2.9.2</version>
</dependency>
```

Para extração de metadados e texto (opcional — aumenta significativamente o tamanho):

```xml
<!-- Parsers completos: extração de metadados, texto e suporte a 1500+ formatos -->
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers-standard-package</artifactId>
    <version>2.9.2</version>
</dependency>
```

| Funcionalidade | tika-core | tika-parsers |
|---|:---:|:---:|
| Detectar tipo MIME por magic bytes | Sim | Sim |
| Extrair metadados (autor, data) | Não | Sim |
| Extrair texto de documentos | Não | Sim |
| Suporte completo a 1500+ formatos | Parcial | Completo |
| Tamanho aproximado | ~600 KB | ~50 MB |

#### Detecção de tipo real

A classe `Tika` é thread-safe e pode ser reutilizada como bean ou constante estática.

```java
package br.com.exemplo.arquivos;

import org.apache.tika.Tika;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.io.InputStream;
import java.util.Set;

@Component
public class TikaFileValidator {

    private static final Tika TIKA = new Tika();

    private static final Set<String> TIPOS_PERMITIDOS = Set.of(
            "application/pdf",
            "image/png",
            "image/jpeg",
            "image/gif"
    );

    /**
     * Detecta o tipo MIME real do arquivo inspecionando os bytes,
     * independentemente do Content-Type declarado pelo cliente.
     */
    public String detectarTipo(MultipartFile file) throws IOException {
        try (InputStream is = file.getInputStream()) {
            return TIKA.detect(is, file.getOriginalFilename());
        }
    }

    public void validar(MultipartFile file) throws IOException {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("Arquivo vazio");
        }

        if (file.getSize() > 10 * 1024 * 1024) {
            throw new IllegalArgumentException("Arquivo acima do tamanho permitido (10 MB)");
        }

        String tipoReal = detectarTipo(file);

        if (!TIPOS_PERMITIDOS.contains(tipoReal)) {
            throw new IllegalArgumentException("Tipo de arquivo não permitido: " + tipoReal);
        }
    }
}
```

#### Detectando discrepância entre tipo declarado e tipo real

Em cenários de segurança mais rigorosos, é útil rejeitar (ou ao menos registrar) quando o tipo real difere do `Content-Type` enviado pelo cliente:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component
public class TikaFileValidator {

    private static final Logger log = LoggerFactory.getLogger(TikaFileValidator.class);
    private static final Tika TIKA = new Tika();

    private static final Set<String> TIPOS_PERMITIDOS = Set.of(
            "application/pdf",
            "image/png",
            "image/jpeg"
    );

    public void validarComVerificacaoDeTipo(MultipartFile file) throws IOException {
        String tipoDeclarado = file.getContentType();
        String tipoReal = detectarTipo(file);

        if (!tipoReal.equals(tipoDeclarado)) {
            log.warn("Tipo declarado '{}' diverge do tipo real '{}' para o arquivo '{}'",
                    tipoDeclarado, tipoReal, file.getOriginalFilename());
            throw new IllegalArgumentException(
                    "Tipo do arquivo diverge do Content-Type declarado");
        }

        if (!TIPOS_PERMITIDOS.contains(tipoReal)) {
            throw new IllegalArgumentException("Tipo não permitido: " + tipoReal);
        }
    }

    public String detectarTipo(MultipartFile file) throws IOException {
        try (InputStream is = file.getInputStream()) {
            return TIKA.detect(is, file.getOriginalFilename());
        }
    }
}
```

#### Extração de metadados com tika-parsers

Quando `tika-parsers-standard-package` estiver disponível, é possível extrair metadados de documentos como PDFs e arquivos Office:

```java
package br.com.exemplo.arquivos;

import org.apache.tika.metadata.Metadata;
import org.apache.tika.metadata.TikaCoreProperties;
import org.apache.tika.parser.AutoDetectParser;
import org.apache.tika.parser.ParseContext;
import org.apache.tika.sax.BodyContentHandler;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;
import org.xml.sax.ContentHandler;

import java.io.InputStream;

@Component
public class TikaMetadataExtractor {

    public Metadata extrairMetadados(MultipartFile file) throws Exception {
        AutoDetectParser parser = new AutoDetectParser();
        Metadata metadata = new Metadata();
        ContentHandler handler = new BodyContentHandler(-1); // -1 = sem limite de conteúdo

        try (InputStream is = file.getInputStream()) {
            parser.parse(is, handler, metadata, new ParseContext());
        }

        return metadata;
    }
}
```

Exemplo de uso dos metadados extraídos:

```java
Metadata meta = extractor.extrairMetadados(file);

String autor  = meta.get(TikaCoreProperties.CREATOR);
String criado = meta.get(TikaCoreProperties.CREATED);
String titulo = meta.get(TikaCoreProperties.TITLE);

log.info("Autor: {}, Criado: {}, Título: {}", autor, criado, titulo);
```

> Atenção: o parsing completo com `tika-parsers-standard-package` abre o conteúdo do documento. Em arquivos maliciosos (zip bombs, PDFs com macros), isso pode consumir muita memória. Configure limites de tempo e tamanho em `ParseContext` em ambientes de produção, ou use um processo separado (sidecar) para o parsing.

#### Integração no controller de upload

```java
package br.com.exemplo.arquivos;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RestController
@RequestMapping("/api/arquivos")
public class ArquivoUploadController {

    private final TikaFileValidator tikaValidator;
    private final ArquivoStorageService storageService;

    public ArquivoUploadController(TikaFileValidator tikaValidator,
                                   ArquivoStorageService storageService) {
        this.tikaValidator = tikaValidator;
        this.storageService = storageService;
    }

    @PostMapping
    public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file)
            throws IOException {
        tikaValidator.validar(file);
        String caminho = storageService.salvar(file);
        return ResponseEntity.ok("Arquivo salvo em: " + caminho);
    }
}
```

### 13.6. Upload com metadados

Muitas vezes o upload vem junto com outros campos do formulário.

Exemplo:

```java
package br.com.exemplo.arquivos;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/api/documentos")
public class DocumentoUploadController {

    @PostMapping
    public String upload(
            @RequestParam("titulo") String titulo,
            @RequestParam("file") MultipartFile file
    ) {
        return "Arquivo " + file.getOriginalFilename() + " recebido para o titulo " + titulo;
    }
}
```

### 13.7. Upload de JSON e arquivo com `@RequestPart`

Para APIs REST, essa costuma ser a abordagem mais recomendada quando a requisicao envia:

- uma part JSON estruturada;
- uma part com o arquivo binário.

Nesse modelo, o Spring usa os `HttpMessageConverters` e o Jackson para desserializar a part JSON de forma natural.

Exemplo:

```java
package br.com.exemplo.arquivos;

import jakarta.validation.Valid;
import java.util.Map;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestPart;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/upload")
public class ExemploUploadController {

    @PostMapping(
            path = "/v1",
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<Map<String, String>> handleFileUpload1(
            @RequestPart("data") @Valid SomeData dto,
            @RequestPart("binaryFile") MultipartFile file
    ) {
        return ResponseEntity.ok(Map.of("mensagem", "Upload recebido com sucesso!"));
    }
}
```

Pontos importantes:

- a part `data` deve ser enviada como JSON;
- a part `binaryFile` deve ter o mesmo nome esperado no backend;
- essa abordagem é mais explicita e previsível para APIs REST.

### 13.8. Fetch API para `/upload/v1` com `@RequestPart`

Quando o endpoint usa `@RequestPart("data")` para receber JSON e `@RequestPart("binaryFile")` para receber o arquivo, um ponto importante no frontend é que o JSON deve ser enviado como `Blob` com `type: "application/json"`.

Exemplo:

```html
<script>
  async function enviarUploadV1() {
    const fileInput = document.querySelector("#binaryFile");
    const file = fileInput.files[0];

    const data = {
      nome: "Joao da Silva",
      email: "joao@email.com"
    };

    const formData = new FormData();
    formData.append(
      "data",
      new Blob([JSON.stringify(data)], { type: "application/json" })
    );
    formData.append("binaryFile", file);

    const response = await fetch("/upload/v1", {
      method: "POST",
      body: formData
    });

    if (!response.ok) {
      throw new Error("Falha no upload");
    }

    const resultado = await response.json();
    console.log(resultado);
  }
</script>

<input type="file" id="binaryFile" />
<button type="button" onclick="enviarUploadV1()">Enviar v1</button>
```

Pontos de atenção:

- o campo `data` precisa ser enviado como `Blob` com `Content-Type` `application/json`;
- se o JSON for enviado apenas como texto puro no `FormData`, o binding com `@RequestPart` pode não ocorrer como esperado;
- o campo `binaryFile` deve ter exatamente o mesmo nome esperado no backend;
- não se deve definir manualmente o header `Content-Type` da requisição, pois o navegador monta automaticamente o `multipart/form-data` com boundary.

Esse detalhe do `Blob` e especialmente importante em integrações com Spring MVC quando a API recebe JSON estruturado e arquivo na mesma requisição.

### 13.9. Upload de JSON e arquivo com `@ModelAttribute`

Em alguns cenários, a aplicação recebe um `multipart/form-data` com:

- um campo textual contendo JSON;
- um arquivo binário;
- binding via `@ModelAttribute`.

Essa abordagem pode funcionar, mas exige mais cuidado do que `@RequestPart`, porque o Spring não vai desserializar automaticamente a string JSON para um objeto complexo apenas com base no `Content-Type` da part. Para isso, normalmente e preciso usar um binder ou conversor customizado.

Exemplo ajustado:

```java
package br.com.exemplo.arquivos;

import com.fasterxml.jackson.core.JacksonException;
import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.validation.Valid;
import java.beans.PropertyEditorSupport;
import java.util.Map;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/upload")
public class ExemploUploadController {

    private final ObjectMapper objectMapper;

    public ExemploUploadController(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    public record RegistrationMetadata(@Valid SomeData data, MultipartFile binaryFile) {
    }

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // Necessário porque, com @ModelAttribute, o campo "data" chega como texto no multipart
        // e precisa ser convertido manualmente de JSON para SomeData.
        binder.registerCustomEditor(SomeData.class, "data", new PropertyEditorSupport() {
            @Override
            public void setAsText(String json) throws IllegalArgumentException {
                try {
                    setValue(objectMapper.readValue(json, SomeData.class));
                } catch (JacksonException ex) {
                    throw new IllegalArgumentException("Erro ao converter JSON", ex);
                }
            }
        });
    }

    @PostMapping(
            path = "/v2",
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<Map<String, String>> handleFileUpload2(
            @Valid @ModelAttribute RegistrationMetadata registrationMetadata
    ) {
        SomeData dto = registrationMetadata.data();
        MultipartFile file = registrationMetadata.binaryFile();

        return ResponseEntity.ok(Map.of("mensagem", "Upload recebido com sucesso!"));
    }
}
```

Pontos importantes nessa abordagem:

- o campo `data` chega como texto no multipart;
- o binder converte esse texto JSON em `SomeData`;
- `binaryFile` continua sendo recebido como `MultipartFile`;
- o binding depende do nome do campo `data` coincidir com o nome do atributo do record ou DTO.

### 13.10. Fetch API para `/upload/v2` com `@ModelAttribute`

Quando o backend usa `@ModelAttribute`, o JSON normalmente deve ser enviado como texto simples no `FormData`, e não como `Blob` `application/json`.

Exemplo:

```html
<script>
  async function enviarUploadV2() {
    const fileInput = document.querySelector("#binaryFile");
    const file = fileInput.files[0];

    const data = {
      nome: "Joao da Silva",
      email: "joao@email.com"
    };

    const formData = new FormData();
    formData.append("data", JSON.stringify(data));
    formData.append("binaryFile", file);

    const response = await fetch("/upload/v2", {
      method: "POST",
      body: formData
    });

    if (!response.ok) {
      throw new Error("Falha no upload");
    }

    const resultado = await response.json();
    console.log(resultado);
  }
</script>

<input type="file" id="binaryFile" />
<button type="button" onclick="enviarUploadV2()">Enviar v2</button>
```

Pontos de atenção:

- o campo `data` vai como texto comum;
- a conversão depende do `@InitBinder` ou mecanismo equivalente;
- se o nome do campo não for exatamente `data`, o binding falha;
- essa abordagem é menos explicita para APIs REST.

### 13.11. Comparando `@ModelAttribute` e `@RequestPart`

`@ModelAttribute`:

- melhor para formulários multipart mais tradicionais;
- funciona bem quando os campos são simples ou achatados;
- para JSON estruturado, exige binder ou conversão customizada;
- tende a ser mais verboso e menos explicito em APIs REST.

`@RequestPart`:

- melhor para APIs que recebem uma part JSON e outra part de arquivo;
- usa os `HttpMessageConverters` do Spring e o Jackson de forma natural;
- é mais previsível para integrações SPA, mobile e clientes externos;
- costuma ser a abordagem mais recomendada para `JSON + arquivo`.

Na prática:

- se o caso for formulário clássico, `@ModelAttribute` pode ser suficiente;
- se o caso for API REST com JSON estruturado, prefira `@RequestPart`.

### 13.12. Upload resumível com TUS

TUS e um protocolo HTTP aberto para uploads resumíveis. Ele e especialmente útil quando:

- os arquivos podem ser grandes;
- a conexão do usuário pode oscilar;
- o upload precisa ser pausado e retomado;
- o cliente pode trocar de rede ou recarregar a página;
- a aplicação precisa reduzir o risco de reiniciar upload do zero.

Segundo a especificação oficial do protocolo:

- o cliente cria ou conhece um recurso de upload;
- usa `HEAD` para descobrir o `Upload-Offset`;
- usa `PATCH` para continuar enviando a partir do ponto correto;
- o header `Tus-Resumable` identifica a versão do protocolo;
- extensões como `creation`, `termination` e `checksum` podem ser negociadas.

Fluxo básico:

1. o cliente cria o upload com `POST`;
2. o servidor devolve a URL do upload;
3. o cliente envia partes com `PATCH`;
4. se houver interrupção, o cliente consulta o offset com `HEAD`;
5. o upload continua do ponto em que parou.

#### Quando preferir TUS em vez de multipart comum

`MultipartFile` tradicional costuma ser suficiente quando:

- os arquivos são pequenos ou médios;
- o upload é rápido;
- não há grande risco de interrupção.

TUS costuma ser melhor quando:

- os arquivos são grandes;
- há upload via browser em rede instável;
- a experiencia de retomada e importante;
- o upload pode durar vários minutos.

#### Exemplo de headers do protocolo

Criação do upload:

```http
POST /api/tus/files HTTP/1.1
Tus-Resumable: 1.0.0
Upload-Length: 104857600
Upload-Metadata: filename ZmF0dXJhc19tYXJjby5wZGY=
```

Resposta:

```http
HTTP/1.1 201 Created
Tus-Resumable: 1.0.0
Location: /api/tus/files/24e533e02ec3bc40c387f1a0e460e216
```

Consulta do offset:

```http
HEAD /api/tus/files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Tus-Resumable: 1.0.0
```

Resposta:

```http
HTTP/1.1 200 OK
Tus-Resumable: 1.0.0
Upload-Offset: 73400320
```

Continuação do envio:

```http
PATCH /api/tus/files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Tus-Resumable: 1.0.0
Content-Type: application/offset+octet-stream
Upload-Offset: 73400320
```

#### Exemplo de integração com Spring Boot

Uma abordagem pratica em Java é usar uma implementação pronta de servidor TUS, como a biblioteca `tus-java-server` e expor um endpoint Spring MVC que delega o protocolo para ela.

Exemplo conceitual de controller:

```java
package br.com.exemplo.arquivos.tus;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import me.desair.tus.server.TusFileUploadService;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TusUploadController {

    private final TusFileUploadService tusFileUploadService;

    public TusUploadController(TusFileUploadService tusFileUploadService) {
        this.tusFileUploadService = tusFileUploadService;
    }

    @RequestMapping("/api/tus/files/**")
    public void upload(HttpServletRequest request, HttpServletResponse response) throws Exception {
        tusFileUploadService.process(request, response);
    }
}
```

Configuração conceitual do serviço:

```java
package br.com.exemplo.arquivos.tus;

import java.nio.file.Paths;
import me.desair.tus.server.TusFileUploadService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TusConfig {

    @Bean
    public TusFileUploadService tusFileUploadService() {
        TusFileUploadService service = new TusFileUploadService();
        service.withStoragePath(Paths.get("storage-tus").toString());
        service.withUploadUri("/api/tus/files");
        service.withDownloadFeature();
        service.withThreadLocalCache(true);
        return service;
    }
}
```

Esse exemplo é ilustrativo. Dependendo da biblioteca e da versão usadas, a API exata pode variar. A ideia principal é:

- o endpoint Spring recebe as requisições do protocolo;
- o serviço TUS cuida de `POST`, `HEAD`, `PATCH` e possivelmente `DELETE`;
- os arquivos temporários ou finais ficam em um storage configurado.

#### Cliente web

No frontend, o uso de TUS normalmente aparece com bibliotecas JavaScript compatíveis, como Uppy ou `tus-js-client`. O navegador passa a:

- iniciar o upload;
- acompanhar progresso;
- retomar automaticamente após interrupção;
- persistir estado local para continuidade.

#### Cuidados de arquitetura

- validar autenticação e autorização antes de aceitar o upload;
- definir limite máximo de arquivo;
- controlar expiração de uploads incompletos;
- persistir metadados do upload separadamente quando necessário;
- decidir quando o arquivo deixa de ser temporário e passa a ser definitivo;
- integrar com antivírus ou validação posterior em fluxos sensíveis.

#### TUS com storage externo

O TUS resolve o protocolo de upload resumível, mas o backend de armazenamento ainda precisa ser definido. Em sistemas maiores, uma combinação comum e:

- TUS para transporte resumível;
- storage em disco temporário ou objeto;
- processo posterior para consolidar metadados e mover o arquivo ao destino final.
### 13.13. Download simples com `Resource`

Uma forma comum de download e retornar um `Resource` com `Content-Disposition`.

```java
package br.com.exemplo.arquivos;

import java.nio.file.Path;
import org.springframework.core.io.PathResource;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/arquivos")
public class FileDownloadController {

    private final FileStorageService fileStorageService;

    public FileDownloadController(FileStorageService fileStorageService) {
        this.fileStorageService = fileStorageService;
    }

    @GetMapping("/download/{nomeArquivo}")
    public ResponseEntity<Resource> download(@PathVariable String nomeArquivo) {
        Path arquivo = fileStorageService.obter(nomeArquivo);
        Resource resource = new PathResource(arquivo);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(resource);
    }
}
```

### 13.14. Download em streaming

Quando o arquivo pode ser grande, o ideal e evitar carregar tudo em memória.

Exemplo com `StreamingResponseBody`:

```java
package br.com.exemplo.arquivos;

import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.mvc.method.annotation.StreamingResponseBody;

@Controller
@RequestMapping("/api/stream")
public class StreamingFileDownloadController {

    private final FileStorageService fileStorageService;

    public StreamingFileDownloadController(FileStorageService fileStorageService) {
        this.fileStorageService = fileStorageService;
    }

    @GetMapping("/download/{nomeArquivo}")
    public ResponseEntity<StreamingResponseBody> download(@PathVariable String nomeArquivo) {
        Path arquivo = fileStorageService.obter(nomeArquivo);

        StreamingResponseBody body = outputStream -> {
            try (InputStream inputStream = Files.newInputStream(arquivo)) {
                inputStream.transferTo(outputStream);
            }
        };

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + arquivo.getFileName() + "\"")
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(body);
    }
}
```

Esse formato costuma ser melhor para:

- arquivos grandes;
- downloads frequentes;
- menor pressão sobre heap da aplicação.

### 13.15. Armazenamento em banco ou armazenamento externo

Em geral, existem três estratégias comuns:

- disco local;
- banco de dados;
- storage externo, como S3, MinIO ou equivalente.

Regra prática:

- disco local: simples, mas ruim para múltiplas instancias sem volume compartilhado;
- banco: pode funcionar para arquivos pequenos, mas costuma não ser ideal para volume alto;
- storage externo: melhor para escalabilidade, CDN, compartilhamento e alta disponibilidade.

Uma estratégia muito comum e:

- salvar o binário em storage externo;
- persistir no banco apenas os metadados e o identificador do arquivo.

### 13.16. Cuidados de segurança

Em upload e download, alguns cuidados são essenciais:

- validar tipo, extensão e tamanho;
- evitar usar diretamente o nome enviado pelo usuário como nome físico;
- impedir path traversal;
- restringir quem pode baixar ou subir arquivos;
- escanear arquivos em cenários sensíveis;
- nunca confiar apenas no `contentType` informado pelo cliente;
- para downloads privados, validar autorização antes de servir o arquivo.

Se a aplicação usa Spring Security, o endpoint de download deve validar se o usuário tem acesso ao recurso solicitado.

### 13.17. Tratamento de erros

Erros comuns:

- arquivo acima do limite;
- multipart malformado;
- arquivo inexistente no download;
- permissão insuficiente;
- falha de armazenamento.

Exemplo de tratamento global:

```java
package br.com.exemplo.arquivos;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MaxUploadSizeExceededException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class FileExceptionHandler {

    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ResponseEntity<String> handleMaxUploadSizeExceeded(MaxUploadSizeExceededException ex) {
        return ResponseEntity.status(HttpStatus.PAYLOAD_TOO_LARGE)
                .body("Arquivo acima do tamanho permitido");
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> handleIllegalArgument(IllegalArgumentException ex) {
        return ResponseEntity.badRequest().body(ex.getMessage());
    }
}
```

### 13.18. Boas práticas

- defina limites de upload no `application.yml`;
- valide tamanho, extensão e conteúdo;
- prefira nomes gerados internamente;
- use streaming para downloads grandes;
- mantenha metadados e binário separados quando fizer sentido;
- não salve arquivos sensíveis sem controle de autorização;
- monitore uso de disco, temporários e falhas de upload;
- em aplicações distribuídas, prefira storage compartilhado ou externo.

## 14. Concorrência com `java.util.concurrent`

Os pacotes `java.util.concurrent`, `java.util.concurrent.locks` e `java.util.concurrent.atomic` formam o núcleo de concorrência da plataforma Java. Em aplicações Spring Boot, eles costumam aparecer em cenários como:

- proteção de recursos compartilhados em serviços stateful;
- controle de acesso concorrente a caches locais;
- sincronização de operações que precisam distinguir leitura de escrita;
- coordenação entre threads em processamento paralelo.

Os principais grupos de tipos são:

**Locks (`java.util.concurrent.locks`)**

| Tipo | Finalidade |
| --- | --- |
| `ReentrantLock` | Exclusão mútua reentrant com controle explícito |
| `ReentrantReadWriteLock` | Múltiplos leitores simultâneos ou um único escritor |
| `StampedLock` | Lock otimista para leitura, além dos modos tradicional de leitura e escrita |
| `Condition` | Coordenação entre threads, substituto de `wait/notify` |
| `LockSupport` | Utilitário de baixo nível para suspender e retomar threads |

**Classes atômicas (`java.util.concurrent.atomic`)**

| Tipo | Finalidade |
| --- | --- |
| `AtomicBoolean` / `AtomicInteger` / `AtomicLong` | Tipos primitivos thread-safe com operações CAS |
| `AtomicReference<V>` | Referência de objeto trocada atomicamente |
| `LongAdder` / `DoubleAdder` | Acumulação de alta frequência com menor contenção |
| `LongAccumulator` / `DoubleAccumulator` | Acumulação com operação customizada (max, min, etc.) |

**Coleções concorrentes (`java.util.concurrent`)**

| Tipo | Finalidade |
| --- | --- |
| `ConcurrentHashMap` | Mapa thread-safe com alto paralelismo interno |
| `CopyOnWriteArrayList` / `CopyOnWriteArraySet` | Lista/conjunto otimizado para leitura frequente |
| `LinkedBlockingQueue` / `ArrayBlockingQueue` | Fila bloqueante produtor-consumidor |
| `PriorityBlockingQueue` | Fila bloqueante com prioridade |
| `DelayQueue` | Fila com elementos disponíveis após atraso |
| `ConcurrentSkipListMap` / `ConcurrentSkipListSet` | Mapa e conjunto concorrentes ordenados |

**Sincronizadores (`java.util.concurrent`)**

| Tipo | Finalidade |
| --- | --- |
| `CountDownLatch` | Aguardar N operações concluírem (uso único) |
| `CyclicBarrier` | Sincronizar threads em fases repetíveis |
| `Semaphore` | Limitar o número de acessos simultâneos |
| `Exchanger` | Trocar objetos entre dois threads |
| `Phaser` | Barreira reutilizável com participantes dinâmicos |

### 14.1. `ReentrantLock`

`ReentrantLock` é a alternativa explícita ao `synchronized`. O mesmo thread pode adquirir o lock mais de uma vez sem se bloquear (reentrada), e deve liberá-lo a mesma quantidade de vezes.

Diferenças práticas em relação ao `synchronized`:

- permite tentar adquirir o lock com timeout (`tryLock`);
- permite interromper a espera (`lockInterruptibly`);
- permite consultar quantos threads estão aguardando;
- o unlock deve ficar sempre em um bloco `finally`.

#### Situações de uso

- substituir `synchronized` quando for necessário timeout ou interrupção na espera;
- proteger operações críticas em services com estado compartilhado;
- garantir exclusividade em operações de inicialização ou reset de recursos;
- controlar acesso a recursos externos que não são thread-safe.

#### Exemplo em um service Spring Boot

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.locks.ReentrantLock;
import org.springframework.stereotype.Service;

@Service
public class ContadorService {

    private final ReentrantLock lock = new ReentrantLock();
    private int contador = 0;

    public void incrementar() {
        lock.lock();
        try {
            contador++;
        } finally {
            lock.unlock();
        }
    }

    public int obterValor() {
        lock.lock();
        try {
            return contador;
        } finally {
            lock.unlock();
        }
    }
}
```

#### Exemplo com `tryLock` e timeout

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;
import org.springframework.stereotype.Service;

@Service
public class RecursoExclusivoService {

    private final ReentrantLock lock = new ReentrantLock();

    public boolean processarSeTiver() throws InterruptedException {
        boolean adquiriu = lock.tryLock(500, TimeUnit.MILLISECONDS);
        if (!adquiriu) {
            return false;
        }
        try {
            // Operação exclusiva.
            return true;
        } finally {
            lock.unlock();
        }
    }
}
```

`tryLock` é especialmente útil em cenários onde não faz sentido bloquear indefinidamente, por exemplo em endpoints de processamento manual ou jobs concorrentes.

### 14.2. `ReentrantReadWriteLock`

`ReentrantReadWriteLock` mantém dois locks internos:

- `readLock()`: pode ser adquirido por vários threads simultaneamente, desde que não haja escritor ativo;
- `writeLock()`: exclusivo — bloqueia todos os outros leitores e escritores.

Isso permite maior paralelismo em cenários com muitas leituras e poucas escritas.

#### Situações de uso

- cache local em memória com alta taxa de leitura e baixa taxa de atualização;
- tabelas de configuração ou parâmetros consultados com frequência e alterados raramente;
- estruturas de dados compartilhadas entre threads em processamento paralelo;
- registros em memória (como listas ou mapas) que precisam de consistência nas escritas, mas podem ser lidos em paralelo.

#### Exemplo com cache local

```java
package br.com.exemplo.concorrencia;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReentrantReadWriteLock;
import org.springframework.stereotype.Service;

@Service
public class ConfiguracaoCacheService {

    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Map<String, String> cache = new HashMap<>();

    public String obter(String chave) {
        rwLock.readLock().lock();
        try {
            return cache.get(chave);
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void atualizar(String chave, String valor) {
        rwLock.writeLock().lock();
        try {
            cache.put(chave, valor);
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void limpar() {
        rwLock.writeLock().lock();
        try {
            cache.clear();
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

Nesse modelo:

- múltiplos threads podem chamar `obter` ao mesmo tempo sem bloquear uns aos outros;
- `atualizar` e `limpar` esperam todos os leitores terminarem antes de adquirir o lock de escrita;
- após o lock de escrita ser adquirido, novas leituras também ficam bloqueadas.

### 14.3. `StampedLock`

`StampedLock` foi introduzido no Java 8 e oferece três modos de operação:

- **escrita**: exclusão mútua total, similar ao `writeLock` do `ReentrantReadWriteLock`;
- **leitura pessimista**: permite vários leitores simultâneos, mas bloqueia escritores;
- **leitura otimista**: não bloqueia ninguém — o thread lê assumindo que ninguém vai escrever, e ao final valida se isso foi verdade.

A leitura otimista é o principal diferencial. Ela evita contenção em leituras quando escritas são raras, pois não adquire nenhum lock de verdade.

Atenção importante: `StampedLock` **não é reentrant**. Um thread que já possui o lock de escrita e tentar adquirir o lock de leitura vai entrar em deadlock.

#### Situações de uso

- estruturas de dados muito lidas e raramente modificadas onde performance é crítica;
- substituição de `ReentrantReadWriteLock` quando leituras otimistas forem suficientes;
- cenários de altíssimo throughput em leitura onde o custo do lock precisa ser mínimo;
- coordenação de acesso a dados voláteis em processamento paralelo intensivo.

#### Exemplo com leitura otimista

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.locks.StampedLock;
import org.springframework.stereotype.Service;

@Service
public class SaldoService {

    private final StampedLock lock = new StampedLock();
    private double saldo = 0.0;

    public double lerSaldo() {
        long stamp = lock.tryOptimisticRead();
        double valorLido = saldo;

        if (!lock.validate(stamp)) {
            // Houve escrita durante a leitura otimista; faz leitura pesimista.
            stamp = lock.readLock();
            try {
                valorLido = saldo;
            } finally {
                lock.unlockRead(stamp);
            }
        }

        return valorLido;
    }

    public void creditar(double valor) {
        long stamp = lock.writeLock();
        try {
            saldo += valor;
        } finally {
            lock.unlockWrite(stamp);
        }
    }
}
```

O padrão "tenta otimista, cai para pessimista se inválido" é o idioma mais comum com `StampedLock`. Quando escritas são raras, a maioria das leituras passa pelo caminho otimista sem qualquer bloqueio.

### 14.4. `Condition`

`Condition` é usado junto com `ReentrantLock` para coordenar threads que precisam esperar por uma condição específica antes de prosseguir. Equivale ao `wait/notify` do `synchronized`, mas com mais flexibilidade:

- é possível ter múltiplas condições por lock;
- a espera pode ter timeout;
- a espera pode ser interrompível.

Os métodos principais são `await()`, `signal()` e `signalAll()`.

#### Situações de uso

- fila de trabalho com produtor e consumidor em threads diferentes;
- controle de capacidade de buffers compartilhados;
- sincronização de fases de processamento paralelo;
- implementação de semáforos ou barreiras customizadas.

#### Exemplo de fila limitada com produtor e consumidor

```java
package br.com.exemplo.concorrencia;

import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;
import org.springframework.stereotype.Component;

@Component
public class FilaLimitada<T> {

    private final int capacidade;
    private final Queue<T> fila = new LinkedList<>();
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition naoCheia = lock.newCondition();
    private final Condition naoVazia = lock.newCondition();

    public FilaLimitada(int capacidade) {
        this.capacidade = capacidade;
    }

    public void produzir(T item) throws InterruptedException {
        lock.lock();
        try {
            while (fila.size() == capacidade) {
                naoCheia.await();
            }
            fila.add(item);
            naoVazia.signal();
        } finally {
            lock.unlock();
        }
    }

    public T consumir() throws InterruptedException {
        lock.lock();
        try {
            while (fila.isEmpty()) {
                naoVazia.await();
            }
            T item = fila.poll();
            naoCheia.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

Ter duas condições separadas (`naoCheia` e `naoVazia`) é mais eficiente do que uma única condição com `signalAll`, porque só acorda os threads relevantes para cada situação.

### 14.5. `LockSupport`

`LockSupport` é um utilitário de baixo nível que permite suspender (`park`) e retomar (`unpark`) threads diretamente, sem necessidade de monitor ou lock. É a base sobre a qual os outros mecanismos do `java.util.concurrent` são construídos.

Na prática, raramente é usado diretamente em código de aplicação. Ele é mais comum em:

- implementações customizadas de estruturas de sincronização;
- frameworks de async como `CompletableFuture`, `ForkJoinPool` e virtual threads;
- instrumentação e diagnóstico de threads.

#### Situações de uso em código de aplicação

- implementar mecanismos de sincronização muito específicos que os outros locks não cobrem;
- construir filas não bloqueantes ou semáforos customizados;
- código de infraestrutura ou utilitário de baixo nível em libraries internas.

#### Exemplo conceitual

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.locks.LockSupport;

public class ExemploLockSupport {

    public static void demonstrar() {
        Thread thread = new Thread(() -> {
            System.out.println("Thread aguardando...");
            LockSupport.park();
            System.out.println("Thread retomada.");
        });

        thread.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        LockSupport.unpark(thread);
    }
}
```

Diferente de `Thread.sleep` ou `Object.wait`, `LockSupport.park` não lança `InterruptedException` — apenas retorna quando interrompido, e o thread deve verificar `Thread.interrupted()` se precisar tratar isso.

### 14.6. Classes atômicas (`java.util.concurrent.atomic`)

O pacote `java.util.concurrent.atomic` oferece tipos que realizam operações de leitura, modificação e escrita de forma atômica, sem necessidade de lock explícito. Internamente, usam instruções CAS (compare-and-swap) da CPU, o que os torna muito eficientes em cenários de baixa a média contução.

#### `AtomicBoolean`

Booleano thread-safe com operações atômicas de leitura e escrita.

Situações de uso:

- flag de estado global acessada por múltiplos threads, como "sistema inicializado" ou "shutdown solicitado";
- controle de execução única (ex.: garantir que uma inicialização rode apenas uma vez);
- sinalização entre threads sem overhead de lock.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.atomic.AtomicBoolean;
import org.springframework.stereotype.Component;

@Component
public class InitializationGuard {

    private final AtomicBoolean inicializado = new AtomicBoolean(false);

    public boolean inicializarSeNecessario() {
        if (inicializado.compareAndSet(false, true)) {
            // Apenas o primeiro thread que chegar aqui executa.
            return true;
        }
        return false;
    }
}
```

`compareAndSet(esperado, novo)` é atômico: só muda o valor se ele ainda for `esperado`. Isso evita a janela de race condition que existiria com `get` + `set` separados.

#### `AtomicInteger` e `AtomicLong`

Inteiros thread-safe com operações atômicas de incremento, decremento, adição e CAS. `AtomicLong` é a versão de 64 bits.

Situações de uso:

- contadores de requisições, erros ou eventos acessados por múltiplos threads;
- geração de IDs sequenciais em memória;
- métricas internas sem sincronização externa;
- controle de slots disponíveis em recursos compartilhados.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicLong;
import org.springframework.stereotype.Component;

@Component
public class MetricasServico {

    private final AtomicInteger requisicoesAtivas = new AtomicInteger(0);
    private final AtomicLong totalProcessado = new AtomicLong(0);

    public void iniciarRequisicao() {
        requisicoesAtivas.incrementAndGet();
    }

    public void finalizarRequisicao(long bytes) {
        requisicoesAtivas.decrementAndGet();
        totalProcessado.addAndGet(bytes);
    }

    public int ativas() {
        return requisicoesAtivas.get();
    }

    public long totalBytes() {
        return totalProcessado.get();
    }
}
```

#### `AtomicReference<V>`

Referência de objeto thread-safe com suporte a CAS. Permite trocar atomicamente o valor apontado por uma referência.

Situações de uso:

- atualização atômica de snapshots ou configurações em memória;
- publicação segura de objeto imutável sem lock;
- substituição atômica de estruturas inteiras (ex.: trocar uma lista inteira de uma vez);
- implementação de algoritmos lock-free que dependem de CAS em objetos.

```java
package br.com.exemplo.concorrencia;

import java.util.List;
import java.util.concurrent.atomic.AtomicReference;
import org.springframework.stereotype.Component;

@Component
public class SnapshotConfiguracao {

    private final AtomicReference<List<String>> permissoesAtivas =
            new AtomicReference<>(List.of());

    public void atualizar(List<String> novasPermissoes) {
        permissoesAtivas.set(List.copyOf(novasPermissoes));
    }

    public List<String> obter() {
        return permissoesAtivas.get();
    }
}
```

Como `List.copyOf` cria uma lista imutável, a referência pode ser lida por qualquer thread sem risco — o objeto em si nunca muda, apenas a referência é trocada atomicamente.

#### `LongAdder` e `DoubleAdder`

Especializados para acumulação de alta frequência. Internamente mantêm células separadas por thread para reduzir contenção, agregando o resultado apenas na leitura (`sum()`).

Situações de uso:

- contadores com altíssimo volume de incrementos concorrentes;
- métricas de throughput em serviços sob carga;
- acumuladores de valores numéricos em processamento paralelo massivo;
- cenários onde `AtomicLong` causa contenção visível por CAS frequente.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.atomic.LongAdder;
import org.springframework.stereotype.Component;

@Component
public class ThroughputMonitor {

    private final LongAdder requisicoes = new LongAdder();

    public void registrar() {
        requisicoes.increment();
    }

    public long total() {
        return requisicoes.sum();
    }

    public void reset() {
        requisicoes.reset();
    }
}
```

Prefira `LongAdder` a `AtomicLong` quando muitos threads incrementam simultaneamente e a leitura do total ocorre com menor frequência.

#### `LongAccumulator` e `DoubleAccumulator`

Generalização de `LongAdder` para qualquer operação binária associativa e comutativa, como `max`, `min` ou `and`.

Situações de uso:

- manter o valor máximo ou mínimo observado entre threads sem lock;
- acumulação com operação customizada (diferente de soma) em cenários paralelos;
- cálculo de estatísticas como pico de latência ou menor valor de lote.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.atomic.LongAccumulator;
import org.springframework.stereotype.Component;

@Component
public class PicoLatenciaMonitor {

    private final LongAccumulator picoMs = new LongAccumulator(Long::max, 0L);

    public void registrar(long latenciaMs) {
        picoMs.accumulate(latenciaMs);
    }

    public long pico() {
        return picoMs.get();
    }

    public void reset() {
        picoMs.reset();
    }
}
```

### 14.7. Coleções concorrentes (`java.util.concurrent`)

O pacote `java.util.concurrent` traz implementações de coleções projetadas para acesso concorrente, sem necessidade de sincronização externa.

#### `ConcurrentHashMap`

Mapa thread-safe com alto grau de paralelismo interno. Ao contrário de `Collections.synchronizedMap`, não bloqueia o mapa inteiro nas operações — usa segmentação interna para permitir leituras e escritas simultâneas em partes diferentes.

Situações de uso:

- cache local compartilhado entre threads sem lock externo;
- registro de sessões, conexões ou recursos ativos em memória;
- contagem por chave em cenários de alta concorrência;
- repositório in-memory de configurações ou estados por tenant ou usuário.

```java
package br.com.exemplo.concorrencia;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import org.springframework.stereotype.Component;

@Component
public class SessionRegistry {

    private final Map<String, String> sessoes = new ConcurrentHashMap<>();

    public void registrar(String sessionId, String usuario) {
        sessoes.put(sessionId, usuario);
    }

    public String usuario(String sessionId) {
        return sessoes.get(sessionId);
    }

    public void remover(String sessionId) {
        sessoes.remove(sessionId);
    }

    public int total() {
        return sessoes.size();
    }
}
```

Operações compostas como "inserir se ausente" e "atualizar se presente" também são atômicas via `putIfAbsent`, `computeIfAbsent`, `computeIfPresent` e `merge`:

```java
// Incremento de contador por chave, sem lock externo:
contadores.merge(chave, 1L, Long::sum);

// Criar bucket apenas na primeira vez:
buckets.computeIfAbsent(clienteId, id -> Bucket.builder().build());
```

#### `CopyOnWriteArrayList`

Lista thread-safe cujas operações de escrita (add, set, remove) criam internamente uma cópia do array subjacente. As leituras operam sobre o snapshot existente, sem bloqueio.

Situações de uso:

- listas de listeners ou observers registrados em runtime;
- listas de configuração lidas com alta frequência e alteradas raramente;
- coleções de filtros, interceptors ou callbacks em memória;
- caching de resultados de consulta que são substituídos periodicamente.

```java
package br.com.exemplo.concorrencia;

import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import org.springframework.stereotype.Component;

@Component
public class EventListenerRegistry {

    private final List<EventHandler> handlers = new CopyOnWriteArrayList<>();

    public void registrar(EventHandler handler) {
        handlers.add(handler);
    }

    public void remover(EventHandler handler) {
        handlers.remove(handler);
    }

    public void notificar(String evento) {
        for (EventHandler handler : handlers) {
            handler.handle(evento);
        }
    }
}
```

Cuidado: cada escrita copia o array inteiro. Para listas grandes com escritas frequentes, o custo pode ser alto. Prefira `CopyOnWriteArrayList` apenas quando leituras superam escritas amplamente.

#### `CopyOnWriteArraySet`

Conjunto thread-safe baseado em `CopyOnWriteArrayList`. Garante ausência de duplicatas com as mesmas características de copy-on-write.

Situações de uso:

- conjunto de IPs, usuários ou tokens ativos com leitura frequente;
- lista de permissões ou bloqueios consultada a cada requisição;
- conjunto de flags ou features ativas carregadas em memória.

#### `LinkedBlockingQueue` e `ArrayBlockingQueue`

Filas bloqueantes thread-safe. `LinkedBlockingQueue` tem capacidade opcional (padrão ilimitado); `ArrayBlockingQueue` tem capacidade fixa e mantém ordem FIFO estrita.

Situações de uso:

- fila de tarefas entre threads produtoras e consumidoras;
- buffer entre componentes assíncronos no mesmo processo;
- limitação de trabalho pendente antes de processar (backpressure simples);
- fila de eventos internos consumidos por worker threads.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.LinkedBlockingQueue;
import org.springframework.stereotype.Component;

@Component
public class TaskQueue {

    private final LinkedBlockingQueue<Runnable> fila = new LinkedBlockingQueue<>(500);

    public boolean enfileirar(Runnable tarefa) {
        return fila.offer(tarefa);
    }

    public Runnable proximaTarefa() throws InterruptedException {
        return fila.take();
    }

    public int pendentes() {
        return fila.size();
    }
}
```

`offer` retorna `false` se a fila estiver cheia (não bloqueia). `take` bloqueia até haver um item.

#### `PriorityBlockingQueue`

Fila bloqueante sem limite de capacidade que entrega elementos por ordem de prioridade (via `Comparator` ou `Comparable`).

Situações de uso:

- fila de processamento com prioridade de negócio (ex.: pedidos urgentes primeiro);
- escalonamento de tarefas por custo ou prazo;
- fila de reprocessamento onde erros críticos têm precedência.

#### `DelayQueue`

Fila onde cada elemento só fica disponível após seu atraso expirar.

Situações de uso:

- agendamento de tarefas com delay sem precisar de scheduler externo;
- expiração de cache ou tokens em memória;
- implementação de retry com backoff em memória.

#### `ConcurrentSkipListMap` e `ConcurrentSkipListSet`

Versões concorrentes de `TreeMap` e `TreeSet` — mantêm os elementos em ordem natural ou por `Comparator`, com acesso thread-safe sem locks globais.

Situações de uso:

- mapa ordenado por chave acessado por múltiplos threads;
- range queries concorrentes (ex.: eventos em uma janela de tempo);
- leaderboards ou rankings em memória;
- índices in-memory ordenados com leitura e escrita simultâneas.

### 14.8. Sincronizadores

O pacote `java.util.concurrent` também inclui sincronizadores de alto nível para coordenação entre threads.

#### `CountDownLatch`

Permite que um ou mais threads esperem até que um conjunto de operações em outros threads seja concluído. O contador só decresce — após chegar a zero, não pode ser reiniciado.

Situações de uso:

- aguardar a inicialização paralela de múltiplos componentes antes de liberar tráfego;
- esperar o término de um conjunto fixo de tarefas assíncronas;
- testes de integração que precisam sincronizar threads concorrentes;
- coordenar startup de serviços dependentes.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import org.springframework.stereotype.Service;

@Service
public class InicializacaoParalelaService {

    public void inicializar() throws InterruptedException {
        int tarefas = 3;
        CountDownLatch latch = new CountDownLatch(tarefas);
        ExecutorService executor = Executors.newFixedThreadPool(tarefas);

        executor.submit(() -> {
            try {
                // Carregar cache de produtos.
            } finally {
                latch.countDown();
            }
        });

        executor.submit(() -> {
            try {
                // Carregar configurações do banco.
            } finally {
                latch.countDown();
            }
        });

        executor.submit(() -> {
            try {
                // Validar conexões externas.
            } finally {
                latch.countDown();
            }
        });

        latch.await(); // Aguarda as três tarefas terminarem.
        executor.shutdown();
    }
}
```

#### `CyclicBarrier`

Similar ao `CountDownLatch`, mas reutilizável: após todos os threads chegarem ao ponto de encontro, a barreira se reinicia automaticamente para o próximo ciclo.

Situações de uso:

- processamento paralelo em fases onde todas as threads devem terminar cada fase antes de avançar;
- simulações ou cálculos iterativos com sincronização a cada rodada;
- pipelines paralelos com checkpoints entre etapas.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class ProcessamentoFasesParalelo {

    private final CyclicBarrier barreira;

    public ProcessamentoFasesParalelo(int threads) {
        this.barreira = new CyclicBarrier(threads, () ->
                System.out.println("Fase concluída — todos os threads sincronizados.")
        );
    }

    public void executarFase(int idThread) throws InterruptedException, BrokenBarrierException {
        System.out.println("Thread " + idThread + " processando fase...");
        barreira.await(); // Espera todos chegarem antes de avançar.
    }
}
```

A ação passada ao construtor (`Runnable`) é executada uma vez quando o último thread chega — útil para consolidar resultados parciais ou registrar métricas por fase.

#### `Semaphore`

Controla o número de threads que podem acessar um recurso simultaneamente. Diferente de um lock, o `Semaphore` pode ter múltiplos "slots" de permissão.

Situações de uso:

- limitar conexões simultâneas a um recurso externo (banco, API, arquivo);
- controlar grau de paralelismo em pools de recursos não gerenciados pelo framework;
- implementar rate limiting interno por número de execuções concorrentes;
- throttle de chamadas a serviços lentos ou com cota de conexões.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.Semaphore;
import org.springframework.stereotype.Service;

@Service
public class ConexaoLimitadaService {

    private final Semaphore semaforo = new Semaphore(10);

    public String consultar(String parametro) throws InterruptedException {
        semaforo.acquire();
        try {
            // No máximo 10 threads simultâneas chegam aqui.
            return "resultado para " + parametro;
        } finally {
            semaforo.release();
        }
    }
}
```

`tryAcquire(timeout, unit)` retorna `false` se não conseguir permissão no tempo definido, permitindo rejeição explícita em vez de bloqueio indefinido.

#### `Exchanger`

Permite que dois threads troquem objetos em um ponto de encontro. Cada thread chama `exchange(valor)` e fica bloqueado até o outro também chamar — os dois recebem o valor do outro.

Situações de uso:

- pipeline com dois estágios onde cada thread preenche um buffer e os troca ao final;
- protocolo de handshake entre produtor e consumidor;
- sincronização de estado entre duas threads em testes ou simulações.

#### `Phaser`

Versão mais flexível e reusável de `CyclicBarrier` e `CountDownLatch`. Permite que threads se registrem e cancelem o registro dinamicamente, e suporta múltiplas fases com comportamento customizável ao avançar de fase.

Situações de uso:

- processamento paralelo com número variável de participantes por fase;
- pipelines onde novos workers podem ser adicionados ou removidos entre fases;
- coordenação de tarefas recursivas ou com divisão dinâmica de trabalho.

```java
package br.com.exemplo.concorrencia;

import java.util.concurrent.Phaser;

public class ProcessamentoDinamicoParalelo {

    public void executar(int threads) {
        Phaser phaser = new Phaser(1); // Registra o thread principal.

        for (int i = 0; i < threads; i++) {
            final int id = i;
            phaser.register();
            new Thread(() -> {
                System.out.println("Thread " + id + " na fase " + phaser.getPhase());
                phaser.arriveAndAwaitAdvance(); // Sincroniza com os demais.
                System.out.println("Thread " + id + " avançou para fase " + phaser.getPhase());
                phaser.arriveAndDeregister(); // Remove-se do phaser.
            }).start();
        }

        phaser.arriveAndDeregister(); // Thread principal se retira.
    }
}
```

### 14.9. Comparação prática

| Cenário | Tipo recomendado |
| --- | --- |
| Exclusão mútua com timeout ou interrupção | `ReentrantLock` |
| Muitas leituras, poucas escritas | `ReentrantReadWriteLock` |
| Leitura muito frequente, escrita rarissima, performance crítica | `StampedLock` |
| Coordenação produtor/consumidor com condição de espera | `ReentrantLock` + `Condition` |
| Infraestrutura de sincronização customizada de baixo nível | `LockSupport` |
| Contador ou flag compartilhado entre threads | `AtomicInteger` / `AtomicLong` / `AtomicBoolean` |
| Referência de objeto trocada atomicamente | `AtomicReference` |
| Contador com altíssimo volume de incrementos concorrentes | `LongAdder` |
| Acumulação com operação customizada (max, min) | `LongAccumulator` |
| Mapa compartilhado entre threads | `ConcurrentHashMap` |
| Lista com muita leitura e pouca escrita (listeners, filtros) | `CopyOnWriteArrayList` |
| Fila produtor-consumidor com backpressure | `LinkedBlockingQueue` / `ArrayBlockingQueue` |
| Fila com prioridade de negócio | `PriorityBlockingQueue` |
| Mapa concorrente com ordenação | `ConcurrentSkipListMap` |
| Aguardar N operações paralelas concluírem (uso único) | `CountDownLatch` |
| Sincronizar threads em fases repetidas | `CyclicBarrier` |
| Limitar conexões ou execuções simultâneas | `Semaphore` |
| Coordenação com número variável de participantes | `Phaser` |
| Recurso já thread-safe ou acessado por thread único | Nenhum |

### 14.10. Locks explícitos vs `synchronized`

`synchronized` continua sendo uma boa escolha quando:

- a proteção é simples e não precisa de timeout;
- não há necessidade de múltiplas condições;
- o escopo da seção crítica é bem delimitado por um bloco ou método.

Locks explícitos valem mais quando:

- é preciso tentar adquirir sem bloquear indefinidamente;
- há necessidade de distinguir leitura de escrita;
- são necessárias múltiplas condições de espera;
- o lock precisa ser transferido entre métodos (o `synchronized` está preso ao escopo léxico).

### 14.11. Boas práticas

- sempre liberar o lock em bloco `finally`;
- preferir `tryLock` com timeout quando bloqueio indefinido for inaceitável;
- não usar `StampedLock` de forma reentrant;
- manter as seções críticas tão curtas quanto possível;
- não chamar métodos externos ou de I/O dentro de seções protegidas por lock sempre que possível;
- preferir tipos atômicos e coleções concorrentes quando o Java já oferece a estrutura adequada — locks explícitos ficam para casos que esses tipos não cobrem;
- para contadores de alta frequência, preferir `LongAdder` a `AtomicLong`;
- para mapas compartilhados, preferir `ConcurrentHashMap` a `HashMap` + `synchronized`;
- usar `computeIfAbsent` e `merge` do `ConcurrentHashMap` em vez de sequências get + put;
- escolher `CopyOnWriteArrayList` apenas quando leituras superam amplamente as escritas;
- calibrar capacidade de `ArrayBlockingQueue` e `Semaphore` conforme o recurso que protegem;
- documentar qual invariante o lock ou sincronizador protege, para facilitar manutenção.

## 15. Actuator e métricas customizadas

O Spring Boot Actuator expõe endpoints operacionais que ajudam a monitorar e gerenciar a aplicação em produção. Com ele é possível verificar saúde, métricas, configurações, informações do ambiente e muito mais.

Além dos endpoints padrão, o Actuator se integra com o Micrometer, permitindo criar métricas customizadas — tanto técnicas quanto de negócio — que podem ser exportadas para sistemas como Prometheus, Datadog, New Relic ou Grafana.

### 15.1. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Para exportar métricas para Prometheus:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 15.2. Configuração básica

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,env,loggers,caches
  endpoint:
    health:
      show-details: when-authorized
      show-components: when-authorized
  info:
    env:
      enabled: true
    git:
      mode: full

info:
  app:
    nome: Minha Aplicação
    versao: 1.0.0
    ambiente: ${ENVIRONMENT:desenvolvimento}
```

Notas importantes:

- `exposure.include` controla quais endpoints ficam acessíveis via HTTP;
- em produção, exponha apenas o necessário e proteja com Spring Security;
- `show-details: when-authorized` exige autenticação para ver detalhes de saúde;
- o endpoint `/actuator/prometheus` só aparece quando o `micrometer-registry-prometheus` está no classpath.

### 15.3. Endpoints mais usados

| Endpoint | Descrição | Uso comum |
| --- | --- | --- |
| `/actuator/health` | estado de saúde da aplicação e suas dependências | readiness e liveness probes em Kubernetes, monitoramento |
| `/actuator/info` | informações sobre a aplicação (versão, git, etc.) | identificação em ambientes com várias instâncias |
| `/actuator/metrics` | lista de métricas disponíveis | exploração e depuração de métricas |
| `/actuator/metrics/{nome}` | valor de uma métrica específica | consulta direta de contadores, timers e gauges |
| `/actuator/prometheus` | métricas no formato Prometheus | scraping pelo Prometheus |
| `/actuator/env` | propriedades do ambiente | depuração de configuração |
| `/actuator/loggers` | níveis de log atuais e alteração em runtime | ajustar log sem restart |
| `/actuator/caches` | caches registrados | monitoramento de caches ativos |

### 15.4. Protegendo endpoints com Spring Security

Em produção, os endpoints do Actuator devem ser protegidos:

```java
package br.com.exemplo.actuator;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class ActuatorSecurityConfig {

    @Bean
    @Order(1)
    public SecurityFilterChain actuatorSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                .securityMatcher("/actuator/**")
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/actuator/health").permitAll()
                        .requestMatchers("/actuator/info").permitAll()
                        .anyRequest().hasRole("ACTUATOR")
                );
        return http.build();
    }
}
```

Essa configuração permite acesso público ao `/health` e `/info`, mas exige role `ACTUATOR` para os demais endpoints.

### 15.5. Health indicators customizados

O Actuator já inclui health indicators para banco de dados, Redis, disk space, JMS e outros. Para verificar dependências específicas da aplicação, é possível criar indicadores customizados.

#### Interfaces do pacote `org.springframework.boot.actuate.health`

| Interface | Quando usar | Comportamento |
| --- | --- | --- |
| `HealthIndicator` | verificações síncronas e rápidas | executa `health()` de forma bloqueante; é a opção mais comum para checar banco, API externa, fila ou recurso local |
| `ReactiveHealthIndicator` | verificações reativas (WebFlux) | retorna `Mono<Health>`; indicado para aplicações reativas ou quando a verificação envolve I/O não bloqueante |
| `HealthContributor` | interface base marcadora | `HealthIndicator` já estende esta interface; usado diretamente apenas em cenários de composição programática |
| `CompositeHealthContributor` | agrupar vários indicadores sob um único nome | útil quando uma dependência tem sub-verificações, por exemplo `broker` com sub-itens `conexao` e `filas` |

Na maioria dos projetos, `HealthIndicator` é suficiente. `ReactiveHealthIndicator` só é relevante em aplicações WebFlux. `CompositeHealthContributor` é útil quando se quer organizar indicadores relacionados como um grupo no JSON de saúde.

#### Exemplo com `ReactiveHealthIndicator`

Indicado para aplicações reativas que usam `WebClient`:

```java
package br.com.exemplo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.ReactiveHealthIndicator;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Component
public class ApiCotacaoReactiveHealthIndicator implements ReactiveHealthIndicator {

    private final WebClient webClient;

    public ApiCotacaoReactiveHealthIndicator(WebClient webClient) {
        this.webClient = webClient;
    }

    @Override
    public Mono<Health> health() {
        return webClient.get()
                .uri("/health")
                .retrieve()
                .toBodilessEntity()
                .map(response -> Health.up()
                        .withDetail("servico", "API de Cotação")
                        .build())
                .onErrorResume(ex -> Mono.just(Health.down()
                        .withDetail("servico", "API de Cotação")
                        .withDetail("erro", ex.getMessage())
                        .build()));
    }
}
```

#### Exemplo com `CompositeHealthContributor`

Quando uma dependência tem várias dimensões de saúde, pode-se agrupá-las:

```java
package br.com.exemplo.actuator;

import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.Map;
import org.springframework.boot.actuate.health.CompositeHealthContributor;
import org.springframework.boot.actuate.health.HealthContributor;
import org.springframework.boot.actuate.health.NamedContributor;
import org.springframework.stereotype.Component;

@Component("broker")
public class BrokerCompositeHealthContributor implements CompositeHealthContributor {

    private final Map<String, HealthContributor> contributors;

    public BrokerCompositeHealthContributor(
            BrokerConexaoHealthIndicator conexaoIndicator,
            BrokerFilasHealthIndicator filasIndicator
    ) {
        this.contributors = new LinkedHashMap<>();
        this.contributors.put("conexao", conexaoIndicator);
        this.contributors.put("filas", filasIndicator);
    }

    @Override
    public HealthContributor getContributor(String name) {
        return contributors.get(name);
    }

    @Override
    public Iterator<NamedContributor<HealthContributor>> iterator() {
        return contributors.entrySet().stream()
                .map(entry -> NamedContributor.of(entry.getKey(), entry.getValue()))
                .iterator();
    }
}
```

Indicadores individuais que compõem o grupo:

```java
package br.com.exemplo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class BrokerConexaoHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean conectado = verificarConexaoBroker();
        return conectado
                ? Health.up().withDetail("status", "conectado").build()
                : Health.down().withDetail("status", "sem conexão").build();
    }

    private boolean verificarConexaoBroker() {
        return true;
    }
}
```

```java
package br.com.exemplo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class BrokerFilasHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int mensagensPendentes = contarMensagensPendentes();
        if (mensagensPendentes > 50_000) {
            return Health.down()
                    .withDetail("mensagensPendentes", mensagensPendentes)
                    .withDetail("motivo", "Acúmulo acima do limiar operacional")
                    .build();
        }
        return Health.up()
                .withDetail("mensagensPendentes", mensagensPendentes)
                .build();
    }

    private int contarMensagensPendentes() {
        return 120;
    }
}
```

O resultado no endpoint `/actuator/health` aparece agrupado:

```json
{
  "status": "UP",
  "components": {
    "broker": {
      "status": "UP",
      "components": {
        "conexao": {
          "status": "UP",
          "details": { "status": "conectado" }
        },
        "filas": {
          "status": "UP",
          "details": { "mensagensPendentes": 120 }
        }
      }
    }
  }
}
```

#### Exemplo com `HealthIndicator` — verificando conectividade com API externa

```java
package br.com.exemplo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestClient;

@Component
public class ApiPagamentoHealthIndicator implements HealthIndicator {

    private final RestClient restClient;

    public ApiPagamentoHealthIndicator(RestClient restClient) {
        this.restClient = restClient;
    }

    @Override
    public Health health() {
        try {
            restClient.get()
                    .uri("/health")
                    .retrieve()
                    .toBodilessEntity();
            return Health.up()
                    .withDetail("servico", "API de Pagamento")
                    .build();
        } catch (Exception ex) {
            return Health.down()
                    .withDetail("servico", "API de Pagamento")
                    .withDetail("erro", ex.getMessage())
                    .build();
        }
    }
}
```

#### Exemplo com `HealthIndicator` — verificando recurso interno

```java
package br.com.exemplo.actuator;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class FilaProcessamentoHealthIndicator implements HealthIndicator {

    private final FilaProcessamentoService filaService;

    public FilaProcessamentoHealthIndicator(FilaProcessamentoService filaService) {
        this.filaService = filaService;
    }

    @Override
    public Health health() {
        int pendentes = filaService.contarPendentes();
        if (pendentes > 10_000) {
            return Health.down()
                    .withDetail("pendentes", pendentes)
                    .withDetail("motivo", "Fila de processamento acima do limite aceitável")
                    .build();
        }
        return Health.up()
                .withDetail("pendentes", pendentes)
                .build();
    }
}
```

O nome do componente no endpoint `/actuator/health` é derivado do nome da classe: `ApiPagamentoHealthIndicator` aparece como `apiPagamento`, `FilaProcessamentoHealthIndicator` aparece como `filaProcessamento`.

### 15.6. Métricas customizadas com Micrometer

O Micrometer é a fachada de métricas do Spring Boot. Ele fornece uma API uniforme para registrar métricas que podem ser exportadas para diferentes backends. Os tipos mais usados são:

| Tipo | Quando usar | Exemplo |
| --- | --- | --- |
| `Counter` | contar ocorrências que só crescem | pedidos criados, e-mails enviados, erros |
| `Timer` | medir duração de operações | tempo de processamento de pedido, latência de integração |
| `Gauge` | medir valor instantâneo que sobe e desce | itens na fila, conexões ativas, memória usada |
| `DistributionSummary` | medir distribuição de valores (não tempo) | tamanho de payload, valor monetário de pedidos |

### 15.7. Counter — contando ocorrências

Exemplo técnico — contando erros de integração:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class IntegracaoExternaService {

    private final Counter sucessoCounter;
    private final Counter falhaCounter;

    public IntegracaoExternaService(MeterRegistry meterRegistry) {
        this.sucessoCounter = Counter.builder("integracao.externa.chamadas")
                .tag("resultado", "sucesso")
                .description("Total de chamadas com sucesso à API externa")
                .register(meterRegistry);

        this.falhaCounter = Counter.builder("integracao.externa.chamadas")
                .tag("resultado", "falha")
                .description("Total de chamadas com falha à API externa")
                .register(meterRegistry);
    }

    public String consultar(String parametro) {
        try {
            String resultado = chamarApiExterna(parametro);
            sucessoCounter.increment();
            return resultado;
        } catch (Exception ex) {
            falhaCounter.increment();
            throw ex;
        }
    }

    private String chamarApiExterna(String parametro) {
        return "resposta";
    }
}
```

Exemplo de negócio — contando pedidos por status:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class PedidoMetricsService {

    private final Counter pedidosCriados;
    private final Counter pedidosCancelados;
    private final Counter pedidosFaturados;

    public PedidoMetricsService(MeterRegistry meterRegistry) {
        this.pedidosCriados = Counter.builder("negocio.pedidos.total")
                .tag("operacao", "criacao")
                .description("Total de pedidos criados")
                .register(meterRegistry);

        this.pedidosCancelados = Counter.builder("negocio.pedidos.total")
                .tag("operacao", "cancelamento")
                .description("Total de pedidos cancelados")
                .register(meterRegistry);

        this.pedidosFaturados = Counter.builder("negocio.pedidos.total")
                .tag("operacao", "faturamento")
                .description("Total de pedidos faturados")
                .register(meterRegistry);
    }

    public void registrarCriacao() {
        pedidosCriados.increment();
    }

    public void registrarCancelamento() {
        pedidosCancelados.increment();
    }

    public void registrarFaturamento() {
        pedidosFaturados.increment();
    }
}
```

Com tags, é possível filtrar e agrupar métricas no dashboard. A métrica `negocio.pedidos.total` aparece com a tag `operacao`, permitindo visualizar cada tipo separadamente.

### 15.8. Timer — medindo duração de operações

Exemplo técnico — medindo latência de chamada externa:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

@Service
public class ConsultaCreditoTimedService {

    private final Timer consultaTimer;

    public ConsultaCreditoTimedService(MeterRegistry meterRegistry) {
        this.consultaTimer = Timer.builder("integracao.credito.duracao")
                .description("Tempo de consulta ao serviço de crédito")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry);
    }

    public String consultar(String documento) {
        return consultaTimer.record(() -> {
            return executarConsulta(documento);
        });
    }

    private String executarConsulta(String documento) {
        return "APROVADO";
    }
}
```

Exemplo de negócio — medindo tempo de processamento de pedido:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

@Service
public class ProcessamentoPedidoTimedService {

    private final Timer processamentoTimer;

    public ProcessamentoPedidoTimedService(MeterRegistry meterRegistry) {
        this.processamentoTimer = Timer.builder("negocio.pedido.processamento.duracao")
                .description("Tempo total para processar um pedido do início ao faturamento")
                .tag("tipo", "completo")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry);
    }

    public void processar(Long pedidoId) {
        processamentoTimer.record(() -> {
            validar(pedidoId);
            calcularFrete(pedidoId);
            faturar(pedidoId);
        });
    }

    private void validar(Long pedidoId) { }
    private void calcularFrete(Long pedidoId) { }
    private void faturar(Long pedidoId) { }
}
```

O `publishPercentiles` permite acompanhar distribuição de latência: p50 (mediana), p95 e p99 ajudam a identificar problemas de cauda longa.

### 15.9. Gauge — medindo valores instantâneos

Exemplo técnico — monitorando tamanho de fila interna:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

import java.util.concurrent.BlockingQueue;

@Component
public class FilaInternaMeterBinder {

    public FilaInternaMeterBinder(MeterRegistry meterRegistry, BlockingQueue<Runnable> filaProcessamento) {
        Gauge.builder("infra.fila.tamanho", filaProcessamento, BlockingQueue::size)
                .description("Quantidade de itens aguardando na fila de processamento")
                .tag("fila", "processamento")
                .register(meterRegistry);
    }
}
```

Exemplo de negócio — monitorando pedidos pendentes:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

@Component
public class PedidosPendentesGauge {

    public PedidosPendentesGauge(MeterRegistry meterRegistry, PedidoRepository pedidoRepository) {
        Gauge.builder("negocio.pedidos.pendentes", pedidoRepository, repo -> repo.countByStatus("PENDENTE"))
                .description("Quantidade atual de pedidos pendentes de processamento")
                .register(meterRegistry);
    }
}
```

Diferente do `Counter`, o `Gauge` reflete o valor no momento da coleta. Ele não acumula — apenas lê o estado atual.

Cuidado: a função passada ao `Gauge` é chamada a cada coleta de métricas. Se a consulta for pesada, pode impactar o desempenho. Nesses casos, considere cachear o valor ou usar um `@Scheduled` para atualizar um `AtomicInteger` que o `Gauge` apenas lê.

Exemplo com `@Scheduled` e `AtomicInteger` para evitar consulta pesada a cada scrape:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.concurrent.atomic.AtomicInteger;

@Component
public class PedidosPendentesGaugeScheduled {

    private final PedidoRepository pedidoRepository;
    private final AtomicInteger pedidosPendentes = new AtomicInteger(0);

    public PedidosPendentesGaugeScheduled(MeterRegistry meterRegistry, PedidoRepository pedidoRepository) {
        this.pedidoRepository = pedidoRepository;

        Gauge.builder("negocio.pedidos.pendentes", pedidosPendentes, AtomicInteger::get)
                .description("Quantidade atual de pedidos pendentes de processamento")
                .register(meterRegistry);
    }

    @Scheduled(fixedRate = 30_000)
    public void atualizarContagem() {
        int total = pedidoRepository.countByStatus("PENDENTE");
        pedidosPendentes.set(total);
    }
}
```

Nesse modelo:

- o `@Scheduled` executa a consulta ao banco a cada 30 segundos e atualiza o `AtomicInteger`;
- o `Gauge` apenas lê o valor do `AtomicInteger`, operação praticamente sem custo;
- o intervalo do `@Scheduled` pode ser ajustado conforme a frequência de scrape do Prometheus ou a volatilidade do dado;
- requer `@EnableScheduling` na configuração da aplicação.

Essa abordagem é especialmente útil quando:

- a consulta envolve `COUNT` em tabelas grandes;
- o scrape de métricas ocorre com frequência alta;
- há vários gauges que dependem de consultas ao banco ou a serviços externos.

### 15.10. DistributionSummary — distribuição de valores

Exemplo de negócio — distribuição de valores de pedidos:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;

@Service
public class PedidoValorMetricsService {

    private final DistributionSummary valorPedidoSummary;

    public PedidoValorMetricsService(MeterRegistry meterRegistry) {
        this.valorPedidoSummary = DistributionSummary.builder("negocio.pedido.valor")
                .description("Distribuição dos valores dos pedidos faturados")
                .baseUnit("BRL")
                .publishPercentiles(0.5, 0.75, 0.95)
                .register(meterRegistry);
    }

    public void registrarValor(BigDecimal valor) {
        valorPedidoSummary.record(valor.doubleValue());
    }
}
```

Exemplo técnico — distribuição de tamanho de payload:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class PayloadMetricsService {

    private final DistributionSummary payloadSummary;

    public PayloadMetricsService(MeterRegistry meterRegistry) {
        this.payloadSummary = DistributionSummary.builder("infra.payload.tamanho")
                .description("Distribuição do tamanho dos payloads recebidos")
                .baseUnit("bytes")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry);
    }

    public void registrar(long tamanhoBytes) {
        payloadSummary.record(tamanhoBytes);
    }
}
```

O `DistributionSummary` é semelhante ao `Timer`, mas mede valores arbitrários em vez de duração.

### 15.11. Usando `@Timed` para métricas automáticas em métodos

O Micrometer oferece a anotação `@Timed` para registrar automaticamente duração e contagem de chamadas a métodos. Para que funcione, é necessário registrar um `TimedAspect`:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.aop.TimedAspect;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TimedAspectConfig {

    @Bean
    public TimedAspect timedAspect(MeterRegistry meterRegistry) {
        return new TimedAspect(meterRegistry);
    }
}
```

Uso em um service:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.annotation.Timed;
import org.springframework.stereotype.Service;

@Service
public class RelatorioService {

    @Timed(value = "negocio.relatorio.geracao", description = "Tempo de geração de relatório")
    public byte[] gerar(String tipo, String periodo) {
        // Lógica de geração.
        return new byte[0];
    }
}
```

Essa abordagem é conveniente para adicionar métricas de tempo sem alterar a lógica do método.

### 15.12. Exemplo completo — métricas de negócio integradas ao fluxo

Um cenário realista é um service de pedidos que combina counter, timer e distribution summary para capturar volume, tempo e valor de operações:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;

@Service
public class PedidoCompletoMetricsService {

    private final Counter pedidosFaturados;
    private final Timer tempoFaturamento;
    private final DistributionSummary valorFaturado;

    public PedidoCompletoMetricsService(MeterRegistry meterRegistry) {
        this.pedidosFaturados = Counter.builder("negocio.pedidos.faturados")
                .description("Total de pedidos faturados com sucesso")
                .register(meterRegistry);

        this.tempoFaturamento = Timer.builder("negocio.pedidos.faturamento.duracao")
                .description("Tempo de processamento do faturamento")
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry);

        this.valorFaturado = DistributionSummary.builder("negocio.pedidos.faturamento.valor")
                .description("Distribuição dos valores faturados")
                .baseUnit("BRL")
                .publishPercentiles(0.5, 0.95)
                .register(meterRegistry);
    }

    public void registrarFaturamento(BigDecimal valor, Runnable operacao) {
        tempoFaturamento.record(() -> {
            operacao.run();
            pedidosFaturados.increment();
            valorFaturado.record(valor.doubleValue());
        });
    }
}
```

Uso no service principal:

```java
package br.com.exemplo.actuator.metricas;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;

@Service
public class FaturamentoService {

    private final PedidoRepository pedidoRepository;
    private final PedidoCompletoMetricsService metricsService;

    public FaturamentoService(PedidoRepository pedidoRepository, PedidoCompletoMetricsService metricsService) {
        this.pedidoRepository = pedidoRepository;
        this.metricsService = metricsService;
    }

    @Transactional
    public void faturar(Long pedidoId) {
        Pedido pedido = pedidoRepository.findById(pedidoId)
                .orElseThrow(() -> new IllegalArgumentException("Pedido não encontrado"));

        BigDecimal valor = pedido.getValorTotal();

        metricsService.registrarFaturamento(valor, () -> {
            pedido.setStatus("FATURADO");
            pedidoRepository.save(pedido);
        });
    }
}
```

Com isso, cada faturamento registra automaticamente:

- quantidade total de faturamentos (`Counter`);
- tempo gasto em cada operação (`Timer`);
- distribuição dos valores faturados (`DistributionSummary`).

### 15.13. Contagem por entidade — visualizações e likes de produtos

Um caso muito comum em aplicações de negócio é contar interações por item, como visualizações e likes de produtos. O desafio é que o ID do produto é um discriminador de alta cardinalidade — criar uma série temporal por produto no Micrometer pode sobrecarregar o backend de métricas quando o catálogo é grande.

A abordagem recomendada é separar as responsabilidades:

- contadores por produto ficam na camada da aplicação, usando estruturas concorrentes;
- métricas agregadas vão para o Micrometer, com baixa cardinalidade;
- consultas por produto específico ficam disponíveis via API ou persistência.

#### Serviço de contagem por produto

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.LongAdder;

@Service
public class ProdutoInteracaoService {

    private final Map<Long, LongAdder> visualizacoesPorProduto = new ConcurrentHashMap<>();
    private final Map<Long, LongAdder> likesPorProduto = new ConcurrentHashMap<>();

    private final Counter totalVisualizacoes;
    private final Counter totalLikes;

    public ProdutoInteracaoService(MeterRegistry meterRegistry) {
        this.totalVisualizacoes = Counter.builder("negocio.produto.visualizacoes.total")
                .description("Total agregado de visualizações de produtos")
                .register(meterRegistry);

        this.totalLikes = Counter.builder("negocio.produto.likes.total")
                .description("Total agregado de likes em produtos")
                .register(meterRegistry);
    }

    public void registrarVisualizacao(Long produtoId) {
        visualizacoesPorProduto
                .computeIfAbsent(produtoId, id -> new LongAdder())
                .increment();
        totalVisualizacoes.increment();
    }

    public void registrarLike(Long produtoId) {
        likesPorProduto
                .computeIfAbsent(produtoId, id -> new LongAdder())
                .increment();
        totalLikes.increment();
    }

    public long obterVisualizacoes(Long produtoId) {
        LongAdder adder = visualizacoesPorProduto.get(produtoId);
        return adder != null ? adder.sum() : 0;
    }

    public long obterLikes(Long produtoId) {
        LongAdder adder = likesPorProduto.get(produtoId);
        return adder != null ? adder.sum() : 0;
    }
}
```

Nesse modelo:

- `ConcurrentHashMap<Long, LongAdder>` mantém contadores individuais por produto, com boa performance sob concorrência;
- `LongAdder` é mais eficiente que `AtomicLong` em cenários de alta frequência de escrita;
- os `Counter` do Micrometer registram apenas o total agregado, sem explodir a cardinalidade;
- os métodos `obterVisualizacoes` e `obterLikes` permitem consultar contagens por produto específico.

#### Uso em um controller

```java
package br.com.exemplo.actuator.metricas;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/produtos")
public class ProdutoInteracaoController {

    private final ProdutoInteracaoService interacaoService;
    private final ProdutoRepository produtoRepository;

    public ProdutoInteracaoController(
            ProdutoInteracaoService interacaoService,
            ProdutoRepository produtoRepository
    ) {
        this.interacaoService = interacaoService;
        this.produtoRepository = produtoRepository;
    }

    @GetMapping("/{id}")
    public ProdutoDto visualizar(@PathVariable Long id) {
        ProdutoDto produto = produtoRepository.findDtoById(id)
                .orElseThrow(() -> new IllegalArgumentException("Produto não encontrado"));
        interacaoService.registrarVisualizacao(id);
        return produto;
    }

    @PostMapping("/{id}/like")
    public void curtir(@PathVariable Long id) {
        interacaoService.registrarLike(id);
    }

    @GetMapping("/{id}/interacoes")
    public InteracoesDto consultarInteracoes(@PathVariable Long id) {
        return new InteracoesDto(
                id,
                interacaoService.obterVisualizacoes(id),
                interacaoService.obterLikes(id)
        );
    }
}
```

```java
package br.com.exemplo.actuator.metricas;

public record InteracoesDto(Long produtoId, long visualizacoes, long likes) {
}
```

#### Persistência periódica das contagens

Em aplicações reais, os contadores em memória devem ser persistidos periodicamente para sobreviver a restarts:

```java
package br.com.exemplo.actuator.metricas;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.concurrent.atomic.LongAdder;

@Component
public class ProdutoInteracaoFlushService {

    private final ProdutoInteracaoService interacaoService;
    private final ProdutoInteracaoRepository repository;

    public ProdutoInteracaoFlushService(
            ProdutoInteracaoService interacaoService,
            ProdutoInteracaoRepository repository
    ) {
        this.interacaoService = interacaoService;
        this.repository = repository;
    }

    @Scheduled(fixedRate = 60_000)
    public void flush() {
        Map<Long, LongAdder> visualizacoes = interacaoService.obterTodasVisualizacoes();
        Map<Long, LongAdder> likes = interacaoService.obterTodosLikes();

        visualizacoes.forEach((produtoId, adder) -> {
            long delta = adder.sumThenReset();
            if (delta > 0) {
                repository.incrementarVisualizacoes(produtoId, delta);
            }
        });

        likes.forEach((produtoId, adder) -> {
            long delta = adder.sumThenReset();
            if (delta > 0) {
                repository.incrementarLikes(produtoId, delta);
            }
        });
    }
}
```

O `sumThenReset()` lê o valor acumulado e zera o contador atomicamente, evitando perda ou contagem dupla. Com isso, o `@Scheduled` acumula os deltas em memória e persiste periodicamente via `UPDATE ... SET visualizacoes = visualizacoes + :delta`.

#### Quando usar tags dinâmicas no Micrometer

Se o catálogo for pequeno e limitado (dezenas de produtos, não milhares), é possível usar tags dinâmicas diretamente no Micrometer:

```java
package br.com.exemplo.actuator.metricas;

import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

@Service
public class ProdutoInteracaoTagService {

    private final MeterRegistry meterRegistry;

    public ProdutoInteracaoTagService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void registrarVisualizacao(Long produtoId) {
        meterRegistry.counter("negocio.produto.visualizacoes",
                "produtoId", String.valueOf(produtoId))
                .increment();
    }

    public void registrarLike(Long produtoId) {
        meterRegistry.counter("negocio.produto.likes",
                "produtoId", String.valueOf(produtoId))
                .increment();
    }
}
```

Essa abordagem é mais simples, mas cria uma série temporal por produto. Ela só é segura quando:

- o número de produtos é pequeno e previsível;
- o backend de métricas suporta a cardinalidade resultante;
- não há risco de crescimento descontrolado do catálogo.

Para catálogos grandes ou dinâmicos, a abordagem com `ConcurrentHashMap` + contadores agregados é mais adequada e escalável.

### 15.14. Convenções de nomenclatura para métricas

Uma boa prática é adotar um padrão consistente de nomes:

| Prefixo | Uso |
| --- | --- |
| `negocio.*` | métricas de domínio e regras de negócio |
| `integracao.*` | chamadas a APIs e serviços externos |
| `infra.*` | filas, pools, recursos internos de infraestrutura |

Exemplos de nomes bem formados:

- `negocio.pedidos.total` com tag `operacao=criacao`
- `negocio.pedido.processamento.duracao`
- `integracao.credito.duracao`
- `integracao.externa.chamadas` com tag `resultado=sucesso`
- `infra.fila.tamanho` com tag `fila=processamento`

Evitar:

- nomes genéricos como `contagem` ou `tempo`;
- misturar idiomas no mesmo projeto;
- tags com cardinalidade alta demais, como IDs de usuário ou UUIDs.

### 15.15. Integração com OpenTelemetry (OTEL)

OpenTelemetry é um padrão aberto e vendor-neutral para observabilidade. Ele unifica três pilares:

- **traces**: rastreamento de requisições entre serviços;
- **métricas**: contadores, timers, gauges e distribuições;
- **logs**: correlação de registros de log com traces.

O Spring Boot 3.x oferece suporte a OpenTelemetry de duas formas principais:

- via Micrometer Tracing com bridge OpenTelemetry, que integra naturalmente com o Actuator;
- via OpenTelemetry Java Agent, que instrumenta automaticamente sem mudança de código.

Ambas as abordagens podem coexistir, mas em projetos Spring Boot o caminho mais comum é usar o suporte nativo via Micrometer Tracing.

### 15.16. Dependências para OTEL via Micrometer Tracing

```xml
<dependencies>
    <!-- Actuator e métricas -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Micrometer Tracing com bridge OpenTelemetry -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-otel</artifactId>
    </dependency>

    <!-- Exportador OTLP para traces -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-otlp</artifactId>
    </dependency>

    <!-- Exportador OTLP para métricas (quando quiser exportar métricas via OTEL) -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-otlp</artifactId>
    </dependency>
</dependencies>
```

Notas:

- `micrometer-tracing-bridge-otel` conecta o Micrometer Tracing ao SDK do OpenTelemetry;
- `opentelemetry-exporter-otlp` envia traces via protocolo OTLP para coletores como Jaeger, Tempo ou o próprio OpenTelemetry Collector;
- `micrometer-registry-otlp` exporta métricas do Micrometer via OTLP, permitindo centralizar métricas e traces no mesmo coletor.

### 15.17. Configuração para exportação via OTLP

```yaml
management:
  tracing:
    enabled: true
    sampling:
      probability: 1.0
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces
    metrics:
      export:
        enabled: true
        url: http://localhost:4318/v1/metrics
        step: 30s
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true
    tags:
      application: minha-aplicacao
      environment: ${ENVIRONMENT:desenvolvimento}

spring:
  application:
    name: minha-aplicacao
```

Notas importantes:

- `sampling.probability: 1.0` envia 100% dos traces — adequado para desenvolvimento; em produção, valores como `0.1` ou `0.01` são mais comuns;
- `spring.application.name` aparece como `service.name` nos traces, identificando o serviço no backend de observabilidade;
- `management.metrics.tags` adiciona tags globais a todas as métricas, útil para distinguir ambientes e instâncias;
- o endpoint `4318` é a porta HTTP padrão do OTLP; para gRPC, use `4317` e o exporter correspondente.

### 15.18. Propagação de contexto entre serviços

Quando uma requisição passa por vários serviços, o trace é propagado automaticamente via headers HTTP. O Spring Boot 3.x suporta os formatos mais comuns:

```yaml
management:
  tracing:
    propagation:
      type: w3c,b3
```

Tipos disponíveis:

- `w3c`: padrão W3C Trace Context (`traceparent` / `tracestate`) — recomendado para novos projetos;
- `b3`: formato Zipkin B3 — útil para compatibilidade com sistemas que já usam Zipkin ou Sleuth.

Na prática, quando o serviço A chama o serviço B via `RestClient`, `RestTemplate` ou `WebClient`, os headers de propagação são injetados automaticamente. Nenhuma configuração adicional é necessária desde que os beans sejam criados via injeção de dependência do Spring.

### 15.19. Instrumentação automática com `@Observed`

O Micrometer Observation API permite instrumentar métodos com `@Observed`, gerando automaticamente traces e métricas.

Configuração necessária:

```java
package br.com.exemplo.actuator.otel;

import io.micrometer.observation.aop.ObservedAspect;
import io.micrometer.observation.ObservationRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ObservationConfig {

    @Bean
    public ObservedAspect observedAspect(ObservationRegistry observationRegistry) {
        return new ObservedAspect(observationRegistry);
    }
}
```

Uso em um service:

```java
package br.com.exemplo.actuator.otel;

import io.micrometer.observation.annotation.Observed;
import org.springframework.stereotype.Service;

@Service
public class ProcessamentoPedidoObservedService {

    @Observed(name = "negocio.pedido.processamento",
            contextualName = "processar-pedido",
            lowCardinalityKeyValues = {"tipo", "padrao"})
    public void processar(Long pedidoId) {
        validar(pedidoId);
        calcular(pedidoId);
        persistir(pedidoId);
    }

    private void validar(Long pedidoId) { }
    private void calcular(Long pedidoId) { }
    private void persistir(Long pedidoId) { }
}
```

O que `@Observed` gera automaticamente:

- um span no trace com o nome `processar-pedido`;
- um timer com o nome `negocio.pedido.processamento`;
- tags de baixa cardinalidade para filtragem.

Isso reduz código boilerplate quando o objetivo é observabilidade básica de um método.

### 15.20. Spans customizados com Observation API

Quando é necessário criar spans manualmente com mais controle, a Observation API permite isso diretamente:

```java
package br.com.exemplo.actuator.otel;

import io.micrometer.observation.Observation;
import io.micrometer.observation.ObservationRegistry;
import org.springframework.stereotype.Service;

@Service
public class IntegracaoExternaObservadaService {

    private final ObservationRegistry observationRegistry;

    public IntegracaoExternaObservadaService(ObservationRegistry observationRegistry) {
        this.observationRegistry = observationRegistry;
    }

    public String consultar(String codigo) {
        return Observation.createNotStarted("integracao.fornecedor.consulta", observationRegistry)
                .lowCardinalityKeyValue("fornecedor", "api-pagamentos")
                .highCardinalityKeyValue("codigo", codigo)
                .observe(() -> {
                    return executarChamadaExterna(codigo);
                });
    }

    private String executarChamadaExterna(String codigo) {
        return "resposta";
    }
}
```

Diferença entre chaves de baixa e alta cardinalidade:

- `lowCardinalityKeyValue`: valores com poucas variações (tipo, fornecedor, status) — seguros para métricas e tags;
- `highCardinalityKeyValue`: valores com muitas variações (IDs, códigos) — incluídos apenas nos traces, não nas métricas, para evitar explosão de séries temporais.

### 15.21. Correlação de logs com trace ID

Para correlacionar logs com traces distribuídos, o Micrometer Tracing injeta automaticamente `traceId` e `spanId` no MDC do SLF4J. Basta configurar o padrão de log:

```yaml
logging:
  pattern:
    level: "%5p [${spring.application.name},%X{traceId},%X{spanId}]"
```

Saída de exemplo:

```text
 INFO [minha-aplicacao,6a3f2b1c4d5e6f7a8b9c0d1e2f3a4b5c,1a2b3c4d5e6f7a8b] Processando pedido 42
 INFO [minha-aplicacao,6a3f2b1c4d5e6f7a8b9c0d1e2f3a4b5c,9f8e7d6c5b4a3928] Consulta de crédito concluída
```

Com isso, é possível:

- filtrar todos os logs de uma requisição distribuída pelo `traceId`;
- correlacionar logs com spans no Jaeger, Tempo ou Grafana;
- identificar qual etapa do fluxo gerou cada linha de log.

### 15.22. Infraestrutura local com Docker Compose

Para desenvolvimento local, um setup típico com OpenTelemetry Collector, Jaeger (traces) e Prometheus (métricas):

```yaml
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    ports:
      - "4317:4317"
      - "4318:4318"
    volumes:
      - ./otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14250:14250"
    environment:
      COLLECTOR_OTLP_ENABLED: "true"

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
```

Configuração mínima do OpenTelemetry Collector (`otel-collector-config.yaml`):

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

exporters:
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true

  prometheus:
    endpoint: 0.0.0.0:8889

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [otlp/jaeger]
    metrics:
      receivers: [otlp]
      exporters: [prometheus]
```

Esse setup permite:

- enviar traces e métricas da aplicação Spring Boot para o Collector via OTLP;
- visualizar traces no Jaeger em `http://localhost:16686`;
- consultar métricas no Prometheus em `http://localhost:9090`;
- montar dashboards no Grafana em `http://localhost:3000`.

### 15.23. OpenTelemetry Java Agent como alternativa

Para aplicações que precisam de instrumentação sem mudança de código, o OpenTelemetry Java Agent é uma alternativa. Ele é um agente JVM que instrumenta automaticamente bibliotecas populares (Spring MVC, JDBC, JMS, RestTemplate, etc.).

Uso:

```bash
java -javaagent:opentelemetry-javaagent.jar \
     -Dotel.service.name=minha-aplicacao \
     -Dotel.exporter.otlp.endpoint=http://localhost:4318 \
     -Dotel.exporter.otlp.protocol=http/protobuf \
     -Dotel.metrics.exporter=otlp \
     -Dotel.logs.exporter=otlp \
     -jar minha-aplicacao.jar
```

O agente instrumenta automaticamente:

- requisições HTTP de entrada e saída;
- consultas JDBC;
- operações JMS;
- chamadas a caches;
- frameworks de log.

Quando preferir o Java Agent:

- aplicações legadas onde não se quer alterar código;
- instrumentação abrangente sem configuração manual;
- padronização de observabilidade entre aplicações de diferentes stacks.

Quando preferir Micrometer Tracing nativo:

- controle fino sobre quais operações geram spans;
- métricas customizadas de negócio integradas ao mesmo pipeline;
- menos overhead e startup mais rápido;
- aplicações que já usam Actuator e Micrometer.

Na prática, muitos projetos começam com Micrometer Tracing nativo e adicionam o Java Agent apenas quando precisam de instrumentação automática de bibliotecas que o Spring não cobre diretamente.

### 15.24. Boas práticas

- exponha apenas os endpoints necessários em produção e proteja com autenticação;
- use health indicators para verificar dependências críticas;
- crie métricas de negócio que ajudem a responder perguntas reais, como quantos pedidos foram faturados, qual o ticket médio, qual a taxa de erro por integração;
- use tags para dimensionar métricas, mas evite cardinalidade alta;
- combine `Counter`, `Timer` e `DistributionSummary` para ter visão completa de volume, latência e distribuição;
- prefira `@Timed` para métricas simples de duração e o registro manual para cenários mais complexos;
- monitore o custo de gauges que fazem consultas sob demanda;
- defina alertas sobre métricas que indicam degradação, como circuit breaker aberto, taxa de fallback alta ou fila crescendo;
- em Kubernetes, use `/actuator/health/liveness` e `/actuator/health/readiness` como probes;
- mantenha as métricas alinhadas com o time de operações para que os dashboards reflitam o que realmente importa;
- ajuste `sampling.probability` em produção para evitar overhead excessivo de traces;
- use `lowCardinalityKeyValue` para tags que aparecem em métricas e `highCardinalityKeyValue` para dados que ficam apenas nos traces;
- padronize `spring.application.name` em todos os serviços para facilitar a navegação nos traces;
- prefira o formato W3C Trace Context para novos projetos;
- correlacione logs com `traceId` e `spanId` para facilitar depuração de problemas em produção.

## 16. Feature Flags e estratégias de deploy

Feature flags (ou feature toggles) permitem ativar e desativar funcionalidades sem necessidade de novo deploy, separando o conceito de **deploy** (colocar código em produção) do conceito de **release** (tornar a funcionalidade visível para os usuários). Este tópico também cobre as estratégias de deploy mais comuns para minimizar riscos em atualizações de produção.

### 16.1. Estratégias de deploy

```
Blue-Green Deploy:
┌──────────┐     ┌──────────┐
│  Blue    │ ──▶ │  Green   │     Load Balancer alterna 100% do tráfego
│  (v1.0)  │     │  (v2.0)  │     de um ambiente para outro
└──────────┘     └──────────┘

Canary Deploy:
┌──────────┐     ┌──────────┐
│  Stable  │ 90% │  Canary  │ 10%   Tráfego gradual: 10% → 25% → 50% → 100%
│  (v1.0)  │ ──▶ │  (v2.0)  │ ──▶   Rollback automático se erro > threshold
└──────────┘     └──────────┘

Rolling Update:
Pod1 [v1] → [v2]
Pod2 [v1] → [v2]     Atualiza um pod por vez sem downtime
Pod3 [v1] → [v2]
```

| Estratégia | Downtime | Rollback | Custo | Risco |
|------------|----------|----------|-------|-------|
| **Blue-Green** | Zero | Instantâneo | 2x recursos | Baixo |
| **Canary** | Zero | Rápido | +10-20% recursos | Muito baixo |
| **Rolling** | Zero | Moderado | Mesmo | Moderado |
| **Recreate** | Sim | Lento | Mesmo | Alto |

### 16.2. Feature Flags — conceito

A ideia central é encapsular uma funcionalidade em uma condição controlada externamente:

```java
package br.com.exemplo.checkout;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/checkout")
public class CheckoutController {

    private final FeatureFlagService featureFlags;
    private final CheckoutService checkoutService;

    public CheckoutController(FeatureFlagService featureFlags,
                               CheckoutService checkoutService) {
        this.featureFlags = featureFlags;
        this.checkoutService = checkoutService;
    }

    @PostMapping
    public ResponseEntity<?> checkout(@RequestBody CheckoutRequest request) {
        if (featureFlags.isEnabled("novo-gateway-pagamento")) {
            return ResponseEntity.ok(checkoutService.processarComNovoGateway(request));
        }
        return ResponseEntity.ok(checkoutService.processarComGatewayAtual(request));
    }
}
```

Isso permite:
- fazer deploy do código novo em produção sem ativá-lo;
- ativar a feature para um grupo restrito de usuários antes do rollout completo;
- desativar instantaneamente se um problema for detectado — sem rollback de deploy.

### 16.3. Implementação simples com `@ConfigurationProperties`

```java
package br.com.exemplo.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "features")
public class FeatureFlagService {

    private Map<String, Boolean> flags = new HashMap<>();

    public Map<String, Boolean> getFlags() {
        return flags;
    }

    public void setFlags(Map<String, Boolean> flags) {
        this.flags = flags;
    }

    public boolean isEnabled(String flag) {
        return flags.getOrDefault(flag, false);
    }
}
```

```yaml
features:
  flags:
    novo-gateway-pagamento: false
    redesign-checkout: true
    relatorio-v2: false
```

Para alterar uma flag sem redeploy, combine com Spring Cloud Config ou com um endpoint administrativo que atualize o valor em banco de dados.

### 16.4. Feature flags com Spring Profiles

Para cenários mais simples, Spring Profiles pode funcionar como um mecanismo básico de feature toggle:

```yaml
# application.yml
app:
  features:
    novo-relatorio: false

---
# application-homolog.yml
app:
  features:
    novo-relatorio: true
```

```java
package br.com.exemplo.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Map;

@Component
@ConfigurationProperties(prefix = "app.features")
public class ProfileFeatureFlags {

    private boolean novoRelatorio;

    public boolean isNovoRelatorio() {
        return novoRelatorio;
    }

    public void setNovoRelatorio(boolean novoRelatorio) {
        this.novoRelatorio = novoRelatorio;
    }
}
```

A limitação é que a alteração exige restart da aplicação — diferente de uma solução com banco ou serviço externo.

### 16.5. Ferramentas externas

Para cenários mais avançados — segmentação por usuário, A/B testing, porcentagem de rollout ou auditoria de quem ativou cada flag — considere ferramentas especializadas:

| Ferramenta | Modelo | Destaque |
|------------|--------|----------|
| **Unleash** | Open source | Self-hosted, SDK para Java, dashboard web |
| **Flagsmith** | Open source / SaaS | API REST, segmentação de usuários |
| **LaunchDarkly** | SaaS | Feature management completo, targeting avançado |
| **Split** | SaaS | A/B testing integrado, métricas de impacto |

Exemplo de integração com Unleash:

```xml
<dependency>
    <groupId>io.getunleash</groupId>
    <artifactId>unleash-client-java</artifactId>
    <version>9.2.4</version>
</dependency>
```

```java
package br.com.exemplo.config;

import io.getunleash.DefaultUnleash;
import io.getunleash.Unleash;
import io.getunleash.util.UnleashConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class UnleashConfiguration {

    @Bean
    public Unleash unleash() {
        UnleashConfig config = UnleashConfig.builder()
                .appName("minha-aplicacao")
                .instanceId("instancia-1")
                .unleashAPI("http://unleash-server:4242/api")
                .build();

        return new DefaultUnleash(config);
    }
}
```

```java
@Service
public class CheckoutService {

    private final Unleash unleash;

    public CheckoutService(Unleash unleash) {
        this.unleash = unleash;
    }

    public void processar(CheckoutRequest request) {
        if (unleash.isEnabled("novo-gateway-pagamento")) {
            processarComNovoGateway(request);
        } else {
            processarComGatewayAtual(request);
        }
    }
}
```

### 16.6. Boas práticas

- feature flags são código temporário — remova a flag e o branch condicional assim que a funcionalidade estiver estável em produção;
- nomeie as flags de forma descritiva e padronizada (`novo-gateway-pagamento`, não `flag1`);
- documente quem criou cada flag, quando e qual o critério para remoção;
- mantenha um inventário de flags ativas — flags esquecidas se acumulam e dificultam a manutenção;
- para flags de longa duração (kill switches, ops toggles), trate como configuração de infraestrutura e não como feature toggle;
- em testes automatizados, cubra ambos os caminhos (flag ligada e desligada).

---

## 17. Sugestão de organização do projeto

```text
src/main/java/br/com/exemplo/
  mail/
    api/
    config/
    template/
  mensageria/
    rabbit/
    kafka/
    eventos/
  graphql/
  grpc/
    server/
    client/
  api/
    openapi/
  actuator/
    metricas/
  events/
  resilience/
  batch/
  cache/
  session/
  arquivos/
  exportacao/
  quartz/
    api/
    admin/
  jms/
    config/
    pedido/
    notificacao/
    requestreply/
  config/
    featureflags/
  checkout/
```

## 18. Resumo

Os tópicos avançados deste documento seguem a mesma ideia de arquitetura:

- usar a configuração padrão do Spring Boot quando o caso for simples;
- adicionar uma camada administrativa quando for preciso parametrizar comportamento em banco;
- manter a regra de negócio desacoplada da infraestrutura;
- expor operações manuais apenas quando houver necessidade operacional clara;
- escolher o protocolo de comunicação adequado ao cenário (REST, GraphQL, gRPC, mensageria);
- documentar APIs de forma automatizada e sincronizada com o código;
- separar deploy de release com feature flags para reduzir risco de mudanças em produção.

Esse desenho deixa a aplicação mais flexível em produção, reduz a necessidade de deploy para ajustes operacionais e facilita a evolução para cenários multi-tenant, clusterizados ou orientados a eventos.

## 19. Referencias

As referências abaixo foram usadas como base oficial ou complementar relevante para a elaboração deste documento:

### Spring Boot e Spring Framework

- Spring Boot Caching: https://docs.spring.io/spring-boot/reference/io/caching.html
- Spring Boot Messaging/JMS: https://docs.spring.io/spring-boot/reference/messaging/jms.html
- Spring Boot Messaging: https://docs.spring.io/spring-boot/reference/messaging/index.html
- Spring Boot Spring Session: https://docs.spring.io/spring-boot/reference/web/spring-session.html
- Spring Framework JMS Sending: https://docs.spring.io/spring-framework/reference/integration/jms/sending.html
- Spring Framework JMS Receiving: https://docs.spring.io/spring-framework/reference/integration/jms/receiving.html
- Spring Framework Multipart Resolver: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/multipart.html
- Spring Framework Multipart Forms: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/multipart-forms.html
- Spring Framework `ApplicationListener`: https://docs.spring.io/spring-framework/docs/6.1.7/javadoc-api/org/springframework/context/ApplicationListener.html
- Spring Framework `ContextRefreshedEvent`: https://docs.spring.io/spring-framework/docs/6.1.7/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html
- Spring Framework `ContextStartedEvent`: https://docs.spring.io/spring-framework/docs/6.1.7/javadoc-api/org/springframework/context/event/ContextStartedEvent.html
- Spring Framework `RequestHandledEvent`: https://docs.spring.io/spring-framework/docs/6.1.7/javadoc-api/org/springframework/web/context/support/RequestHandledEvent.html
- Spring Framework `ServletRequestHandledEvent`: https://docs.spring.io/spring-framework/docs/6.1.7/javadoc-api/org/springframework/web/context/support/ServletRequestHandledEvent.html
- Baeldung - Spring Events: https://www.baeldung.com/spring-events
- Spring Batch Reference: https://docs.spring.io/spring-batch/reference/

### Spring Security

- Spring Security `AbstractAuthenticationEvent`: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/authentication/event/AbstractAuthenticationEvent.html
- Spring Security `AuthorizationDeniedEvent`: https://docs.spring.io/spring-security/reference/7.1-SNAPSHOT/api/java/org/springframework/security/authorization/event/AuthorizationDeniedEvent.html
- Spring Security Deprecated API Notes: https://docs.spring.io/spring-security/reference/7.1-SNAPSHOT/api/java/deprecated-list.html

### Spring Session

- Spring Session Reference: https://docs.spring.io/spring-session/reference/
- Spring Session HttpSession Integration: https://docs.spring.io/spring-session/reference/http-session.html

### ActiveMQ Artemis

- ActiveMQ Artemis Address Model: https://artemis.apache.org/components/artemis/documentation/latest/address-model.html
- ActiveMQ Artemis Using JMS: https://activemq.apache.org/components/artemis/documentation/2.31.1/using-jms.html

### Upload resumível

- TUS Protocol: https://tus.io/protocols/resumable-upload
- `tus-java-server`: https://github.com/tomdesair/tus-java-server
- `tusd`: https://github.com/tus/tusd

### Resiliência e rate limiting

- Bucket4j: https://bucket4j.com/
- Resilience4j: https://resilience4j.readme.io/
- Spring Retry: https://github.com/spring-projects/spring-retry

### Actuator, Micrometer e OpenTelemetry

- Spring Boot Actuator: https://docs.spring.io/spring-boot/reference/actuator/
- Spring Boot Actuator Endpoints: https://docs.spring.io/spring-boot/reference/actuator/endpoints.html
- Spring Boot Actuator Metrics: https://docs.spring.io/spring-boot/reference/actuator/metrics.html
- Spring Boot Observability: https://docs.spring.io/spring-boot/reference/actuator/observability.html
- Spring Boot Tracing: https://docs.spring.io/spring-boot/reference/actuator/tracing.html
- Micrometer: https://micrometer.io/
- Micrometer Concepts: https://docs.micrometer.io/micrometer/reference/concepts.html
- Micrometer Tracing: https://docs.micrometer.io/tracing/reference/
- Micrometer Prometheus: https://docs.micrometer.io/micrometer/reference/implementations/prometheus.html
- Micrometer OTLP: https://docs.micrometer.io/micrometer/reference/implementations/otlp.html
- OpenTelemetry: https://opentelemetry.io/
- OpenTelemetry Java: https://opentelemetry.io/docs/languages/java/
- OpenTelemetry Java Agent: https://opentelemetry.io/docs/zero-code/java/agent/
- OpenTelemetry Collector: https://opentelemetry.io/docs/collector/

### Cache

- Caffeine: https://github.com/ben-manes/caffeine

### Apache Tika

- Apache Tika — Getting Started: https://tika.apache.org/2.9.2/gettingstarted.html
- Apache Tika — Supported Formats: https://tika.apache.org/2.9.2/formats.html
- Apache Tika — API Javadoc: https://tika.apache.org/2.9.2/api/
- Apache Tika no Maven Central: https://mvnrepository.com/artifact/org.apache.tika/tika-core

### CSV e Excel

- Apache Commons CSV: https://commons.apache.org/proper/commons-csv/
- Apache POI Component Overview: https://poi.apache.org/components/
- Apache POI Quick Guide: https://poi.apache.org/components/spreadsheet/quick-guide.html
- Apache POI `SXSSFWorkbook`: https://poi.apache.org/apidocs/5.0/org/apache/poi/xssf/streaming/SXSSFWorkbook.html
- `pjfanning/excel-streaming-reader`: https://github.com/pjfanning/excel-streaming-reader

### Mensageria (Kafka e RabbitMQ)

- Spring for Apache Kafka: https://docs.spring.io/spring-kafka/reference/
- Spring AMQP (RabbitMQ): https://docs.spring.io/spring-amqp/reference/
- Apache Kafka Documentation: https://kafka.apache.org/documentation/
- RabbitMQ Tutorials: https://www.rabbitmq.com/tutorials

### GraphQL

- Spring for GraphQL: https://docs.spring.io/spring-graphql/reference/
- GraphQL Specification: https://spec.graphql.org/
- Spring Boot GraphQL Starter: https://docs.spring.io/spring-boot/reference/web/spring-graphql.html

### gRPC

- gRPC Official Documentation: https://grpc.io/docs/
- Protocol Buffers Language Guide (proto3): https://protobuf.dev/programming-guides/proto3/
- grpc-spring-boot-starter (net.devh): https://github.com/yidongnan/grpc-spring-boot-starter
- gRPC Java: https://grpc.io/docs/languages/java/

### SpringDoc OpenAPI

- SpringDoc OpenAPI: https://springdoc.org/
- OpenAPI 3.0 Specification: https://spec.openapis.org/oas/v3.0.3
- Swagger Annotations (io.swagger.v3): https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations

### Feature Flags

- Unleash: https://www.getunleash.io/
- Unleash SDK for Java: https://docs.getunleash.io/reference/sdks/java
- Flagsmith: https://www.flagsmith.com/
- Martin Fowler — Feature Toggles: https://martinfowler.com/articles/feature-toggles.html
