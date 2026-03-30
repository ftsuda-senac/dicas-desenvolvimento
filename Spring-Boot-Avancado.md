# Spring Boot Avancado

Este documento reune informacoes e exemplos praticos sobre recursos avancados usados com frequencia em aplicacoes Spring Boot:

- envio de e-mails com configuracao padrao, configuracao dinamica de servidores SMTP e templates Thymeleaf;
- execucao de jobs com Quartz, incluindo persistencia em banco, controle dinamico e disparo manual;
- uso de JMS com Artemis, cobrindo filas, pub-sub, filas duraveis, uso de `pooled-jms` e mensagens com resposta.
- uso de Spring Events para comunicacao interna desacoplada, com listeners sincronos, assincronos e transacionais;
- mecanismos de resiliencia, incluindo timeout, retry, circuit breaker, bulkhead, rate limiter e fallback;
- rate limiting com Bucket4j para controle de acesso por IP, usuario, tenant ou chave de negocio.
- processamento em lote com Spring Batch, incluindo `Job`, `Step`, chunk processing, restart, skip e retry.
- cache e Spring Session, incluindo `@Cacheable`, `@CachePut`, `@CacheEvict`, Caffeine, Redis e sessao distribuida.
- upload e download de arquivos, incluindo `MultipartFile`, streaming, armazenamento local e validacoes de seguranca.
- leitura e exportacao de dados em CSV e Excel com Apache Commons CSV, Apache POI e `excel-streaming-reader`.

Os exemplos abaixo seguem um estilo compativel com Spring Boot 3.x e Jakarta EE.

## 1. Envio de e-mails

### 1.1. Dependencias

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

### 1.2. Configuracao padrao

Quando a aplicacao usa um unico servidor SMTP, a forma padrao e configurar tudo em `application.yml`.

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

Template estatico:

```text
src/main/resources/templates/mail/boas-vindas.html
```

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <h1>Bem-vindo(a), <span th:text="${nome}">Nome</span>!</h1>
    <p>Seu cadastro foi realizado com sucesso.</p>
    <p>Codigo de ativacao: <strong th:text="${codigoAtivacao}">0000</strong></p>
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

Servico:

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

### 1.5. Cadastro e obtencao dinamica dos servidores de e-mail

Quando a aplicacao precisa suportar varios clientes, tenants ou unidades, e comum armazenar as configuracoes SMTP no banco.

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

Repositorio:

```java
package br.com.exemplo.mail.config;

import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MailServerConfigRepository extends JpaRepository<MailServerConfig, Long> {

    Optional<MailServerConfig> findByCodigoAndAtivoTrue(String codigo);
}
```

Fabrica dinamica de `JavaMailSender`:

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

Servico de envio usando configuracao dinamica:

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
                .orElseThrow(() -> new IllegalArgumentException("Servidor SMTP nao encontrado"));

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

API basica para cadastro e consulta:

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
                .orElseThrow(() -> new IllegalArgumentException("Configuracao nao encontrada"));
    }
}
```

Boas praticas:

- proteger senha com criptografia ou secret manager;
- validar conectividade antes de ativar a configuracao;
- manter cache com invalidacao se houver muitas leituras;
- nunca expor credenciais em logs ou APIs administrativas.

### 1.6. Gerenciamento de templates com Thymeleaf

#### Templates estaticos

Os templates estaticos ficam versionados com a aplicacao:

```text
src/main/resources/templates/email/
  notificacao.html
  redefinicao-senha.html
  cobranca.html
```

Eles sao ideais quando:

- o layout muda pouco;
- a equipe quer revisar alteracoes pelo Git;
- a publicacao do template faz parte do deploy.

#### Templates dinamicos

Quando o conteudo precisa ser alterado sem novo deploy, pode-se salvar o HTML no banco e processa-lo com `StringTemplateResolver`.

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

Configuracao do renderizador (ou usar configuração mostrada anteriormente)

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
                .orElseThrow(() -> new IllegalArgumentException("Template nao encontrado"));

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

Exemplo de template dinamico salvo em banco:

```html
<html>
<body>
    <h2>Ola, <span th:text="${nomeCliente}">Cliente</span></h2>
    <p>Seu pedido <strong th:text="${numeroPedido}">123</strong> foi faturado.</p>
    <p>Valor total: <strong th:text="${valorTotal}">0,00</strong></p>
</body>
</html>
```

Cuidados com templates dinamicos:

- validar a sintaxe antes de publicar;
- manter historico e versao do template;
- limitar quem pode editar;
- higienizar HTML quando houver edicao rica;
- considerar fallback para template estatico.

Uma estrategia muito pratica e manter templates tecnicos como estaticos e comunicacoes mais volateis como dinamicas.

## 2. Jobs com Quartz

### 2.1. Dependencias

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

Quando houver persistencia funcional das definicoes:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

### 2.2. Configuracao padrao

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
- o ideal em producao e criar as tabelas do Quartz por migration;
- `isClustered: true` e importante quando ha mais de uma instancia da aplicacao.

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

O uso de `@DisallowConcurrentExecution` evita que duas execucoes do mesmo job rodem ao mesmo tempo.

### 2.4. Registro padrao de `JobDetail` e `Trigger`

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

Essa e a forma padrao quando a agenda faz parte fixa da aplicacao.

### 2.5. Gerenciamento dinamico de jobs e triggers com dados em banco

Quando horarios, parametros e ativacao precisam mudar sem deploy, uma estrategia muito comum e manter uma tabela funcional da aplicacao e sincronizar isso com o Quartz.

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

Servico administrativo:

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

Na pratica, os dados dinamicos podem ficar:

- no `JobDataMap`, quando forem pequenos e de uso direto pelo Quartz;
- em uma tabela funcional da aplicacao, quando exigirem auditoria, filtros e historico;
- nas duas camadas, usando a tabela funcional como fonte principal e o Quartz como runtime.

Campos comuns em base funcional:

- expressao cron;
- identificador do tenant;
- parametros de negocio;
- usuario que criou ou alterou;
- flag de ativo/inativo;
- ultima e proxima execucao;
- resultado da ultima tentativa.

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

Tambem e comum manter `ativo = false` na tabela funcional e sincronizar isso com o scheduler ao salvar ou inicializar a aplicacao.

### 2.7. Execucao manual de jobs

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
- execucao sob demanda via painel administrativo;
- teste de agenda em homologacao;
- reenvio de relatorios e integracoes.

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

Boas praticas com Quartz:

- use `@DisallowConcurrentExecution` quando houver risco de concorrencia;
- mantenha o job fino e delegue regra de negocio para services;
- trate `misfire` explicitamente em agendas criticas;
- monitore falhas, tempo medio e quantidade de reprocessamentos;
- evite objetos grandes no `JobDataMap`.

## 3. JMS com Artemis

### 3.1. Dependencias

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

### 3.2. Configuracao basica

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

Para desenvolvimento, tambem e possivel usar broker embarcado se o projeto incluir as dependencias necessarias.

#### Exemplo para Artemis embarcado em ambiente dev

Segundo a documentacao oficial do Spring Boot, para usar Artemis embarcado a aplicacao precisa ter o starter JMS do Artemis e tambem a dependencia do broker embarcado no classpath.

Dependencia adicional:

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-jakarta-server</artifactId>
</dependency>
```

Configuracao recomendada para desenvolvimento:

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

O que essa configuracao faz:

- `mode: embedded` exige broker embarcado;
- `persistent: false` evita gravacao em disco, o que costuma ser adequado para dev;
- `queues` e `topics` criam destinos basicos com opcoes padrao;
- `session-cache-size` mantem o uso simples e eficiente durante desenvolvimento.

Se houver necessidade de configuracao mais avancada das filas e topicos, o proprio Spring Boot recomenda declarar beans do tipo `JMSQueueConfiguration` e `TopicConfiguration`.

Exemplo com fila duravel e topico predefinido:

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

Para ambiente dev, esse modelo embarcado e util quando:

- a equipe quer subir tudo com a propria aplicacao;
- nao compensa manter um broker externo local;
- os testes manuais precisam de filas e topicos ja disponiveis;
- o objetivo e reduzir dependencia de infraestrutura no setup local.

Para homologacao ou producao, normalmente vale migrar para um broker externo dedicado.

### 3.3. Filas duraveis

Em ActiveMQ Artemis, uma fila duravel e uma fila que sobrevive a restart ou falha do broker. Entretanto, para que a mensagem em si sobreviva, ela tambem precisa ser enviada como mensagem duravel ou persistente.

Na pratica:

- fila duravel protege a existencia da fila;
- mensagem duravel protege o conteudo da mensagem;
- para persistencia completa, os dois pontos precisam estar alinhados.

#### O que considerar em Artemis

- filas auto-criadas pelo broker sao duraveis, nao temporarias e nao transientes;
- mesmo assim, elas podem ser removidas automaticamente se as politicas de `auto-delete-queues` estiverem habilitadas;
- para filas de negocio, normalmente vale predefinir a fila ou desabilitar auto-delete para aquele conjunto de enderecos.

#### Exemplo para broker embarcado no Spring Boot

Quando a aplicacao usa Artemis embarcado, o Spring Boot permite declarar beans do tipo `JMSQueueConfiguration` para configuracoes avancadas de fila.

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

Esse modelo e util quando a propria aplicacao sobe o broker e precisa garantir que determinadas filas de negocio existam desde a inicializacao.

#### Exemplo de configuracao no `broker.xml`

Para broker externo ou administrado separadamente, uma configuracao tipica e:

- em uma instalacao standalone do Artemis, o arquivo fica em `<broker-instance>/etc/broker.xml`;
- ou seja, ele nao deve ser colocado em `${ARTEMIS_HOME}`, mas no diretorio da instancia criada do broker;
- em Docker, o caminho padrao da imagem oficial e `/var/lib/artemis-instance/etc/broker.xml`;
- quando for usado mecanismo de sobrescrita da imagem oficial, os arquivos customizados podem ser colocados em `/var/lib/artemis-instance/etc-override`, de onde serao copiados para `etc`.

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

Se a fila for de negocio e nao puder desaparecer automaticamente, uma configuracao complementar muito comum e:

```xml
<address-settings>
   <address-setting match="queue.#">
      <auto-create-queues>true</auto-create-queues>
      <auto-delete-queues>false</auto-delete-queues>
   </address-setting>
</address-settings>
```

#### Garantindo envio persistente da mensagem

Se a fila e duravel, mas a mensagem for enviada como nao persistente, ela nao estara protegida em caso de reinicio ou falha do broker.

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

Resumo pratico:

- fila duravel + mensagem persistente: indicado para pedidos, pagamentos, integracoes criticas;
- fila duravel + mensagem nao persistente: a fila continua existindo, mas a mensagem pode se perder;
- fila temporaria ou nao duravel: mais adequada para cenarios efemeros, testes ou respostas temporarias.

### 3.4. Uso de `pooled-jms`

Por padrao, o Spring Boot envolve a `ConnectionFactory` nativa com uma `CachingConnectionFactory`, configuravel por `spring.jms.*`. Se for preferivel usar pooling nativo de JMS, o Spring Boot suporta isso com a dependencia `org.messaginghub:pooled-jms` e propriedades `spring.artemis.pool.*`.

Segundo a documentacao do Spring Boot, o caminho padrao para Artemis e:

- cache simples via `spring.jms.cache.session-cache-size`;
- pooling nativo via `spring.artemis.pool.enabled=true`.

Segundo a documentacao do `pooled-jms`, o `JmsPoolConnectionFactory` faz pooling de `Connection`, `Session` e `MessageProducer`, reduzindo o custo de criacao repetida desses recursos.

#### Dependencia

```xml
<dependency>
    <groupId>org.messaginghub</groupId>
    <artifactId>pooled-jms</artifactId>
</dependency>
```

#### Configuracao com Spring Boot

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

- `max-connections` controla quantas conexoes fisicas podem ser mantidas;
- `max-sessions-per-connection` ajuda em cenarios com muitas sessoes simultaneas;
- `idle-timeout` evita manter recursos ociosos por tempo excessivo.

#### Quando `pooled-jms` e util

Ele costuma ser especialmente util quando ha:

- muitos envios JMS de curta duracao;
- alto volume de produtores concorrentes;
- request-reply com criacao frequente de sessoes;
- uso intensivo de `JmsTemplate` ou `JmsClient` em chamadas sincronas ou bursts.

Exemplos tipicos:

- API HTTP que publica muitas mensagens por requisicao;
- servico integrador que envia mensagens para varias filas;
- aplicacao com alto throughput de producao e baixa necessidade de listeners customizados.

#### Quando avaliar com mais cuidado

Para listeners de longa duracao, a vantagem pode ser menor. A propria documentacao do Spring Boot destaca que, na maioria dos cenarios, os message listener containers devem trabalhar contra a `ConnectionFactory` nativa, pois assim cada listener container mantem sua propria conexao e responsabilidade de recuperacao local.

Em outras palavras, `pooled-jms` tende a ser mais interessante para o lado produtor do que como principal preocupacao para consumidores de longa vida.

#### Exemplo de uso com `JmsClient`

Nao ha mudanca no codigo de envio. O ganho fica na infraestrutura da `ConnectionFactory`.

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

Ou seja, a troca esta na configuracao do pool, nao no estilo do producer.

### 3.5. Padronizando serializacao JSON

Em projetos reais, e recomendavel padronizar a troca de mensagens em JSON. Assim, os exemplos com `JmsTemplate`, `JmsClient` e `@JmsListener` passam a trabalhar com DTOs de forma consistente, sem depender de `ObjectMessage`.

Uma configuracao comum e registrar um `MappingJackson2MessageConverter`:

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

Com essa configuracao:

- os DTOs podem ser enviados como JSON em `TextMessage`;
- o header `_type` ajuda na desserializacao do tipo;
- o `JmsTemplate` auto-configurado passa a usar esse converter;
- o `JmsClient` auto-configurado tambem acompanha esse comportamento, pois reutiliza a infraestrutura JMS configurada pela aplicacao.

### 3.6. Exemplos com `JmsClient` no Spring Boot 4

No Spring Boot 4, o `JmsClient` e auto-configurado e pode ser injetado diretamente nos beans da aplicacao. Ele oferece uma API fluente para envio, recebimento sincrono e request-reply, funcionando sobre a infraestrutura tradicional do Spring JMS.

Pontos importantes:

- o `JmsClient` e muito pratico para envio e recebimento sincrono;
- para consumo assincrono, `@JmsListener` continua sendo a abordagem mais comum;
- o cliente auto-configurado reutiliza a configuracao do `JmsTemplate`;
- quando a aplicacao mistura fila e topico, pode ser util ter clientes diferentes para cada dominio.

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

#### Recebimento sincrono com timeout

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

Esse tipo de consumo sincrono costuma ser util em rotinas administrativas, testes de integracao ou fluxos request-reply.

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

#### Criando um `JmsClient` dedicado para topicos

Quando o projeto trabalha com fila e pub-sub na mesma aplicacao, o cliente auto-configurado costuma ser suficiente para filas. Para topicos, uma abordagem clara e criar um `JmsClient` baseado em um `JmsTemplate` com `pubSubDomain = true`.

Se existirem dois beans do tipo `JmsClient`, use `@Qualifier` na injecao para deixar explicito qual cliente deve ser usado em cada fluxo.

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

### 3.7. Suportando fila e pub-sub na mesma aplicacao

A mesma aplicacao pode trabalhar com:

- filas para processamento ponto a ponto;
- topicos para eventos e notificacoes para varios consumidores.

Uma configuracao clara e separar `JmsTemplate` e `JmsListenerContainerFactory` para cada modelo.

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

Configuracao:

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

### 3.9. Exemplo com topicos

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

Quando o emissor envia uma mensagem e aguarda resposta, o modelo usado normalmente e request-reply.

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

- o produtor envia a requisicao;
- o Spring usa o destino de resposta adequado;
- a resposta volta ao emissor;
- o `JMSCorrelationID` ajuda a amarrar ida e volta.

### 3.11. Exemplo com correlacao explicita

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

Esse formato e util quando:

- ha integracao com sistemas legados;
- a resposta nao sera imediata;
- o identificador de correlacao precisa ser persistido.

### 3.12. Boas praticas com Artemis e JMS

- usar nomes claros para destinations, como `queue.` e `topic.`;
- para filas criticas, alinhar fila duravel com mensagens persistentes;
- evitar `auto-delete` em filas de negocio que precisam sobreviver ao ciclo de vida do consumidor;
- definir politica de redelivery e dead letter queue;
- padronizar a serializacao em JSON com `MessageConverter`;
- evitar payloads muito grandes;
- monitorar filas, tempos de consumo e erros;
- usar correlation ID em fluxos criticos;
- avaliar `pooled-jms` principalmente quando houver muitos producers e criacao frequente de sessoes;
- documentar quais consumers sao de fila e quais sao de topico.

## 4. Events do Spring

Os Events do Spring sao um mecanismo simples para comunicacao interna entre componentes da mesma aplicacao. Eles ajudam a reduzir acoplamento entre services e sao muito uteis quando uma acao de negocio precisa disparar reacoes adicionais, como auditoria, notificacao ou integracoes internas.

Casos comuns:

- apos concluir um cadastro;
- apos confirmar um pagamento;
- para auditoria interna;
- para disparar tarefas complementares sem acoplar tudo no mesmo service.

### 4.1. Quando usar Spring Events

Spring Events funcionam muito bem quando:

- produtor e consumidor estao na mesma aplicacao;
- nao ha necessidade de broker externo;
- o objetivo principal e desacoplamento interno;
- o fluxo pode ser tratado de forma simples no mesmo processo.

Nao sao a melhor escolha quando:

- a mensagem precisa sobreviver a queda da aplicacao;
- varios sistemas externos precisam consumir o mesmo evento;
- ha necessidade de garantia de entrega entre processos;
- o caso pede mensageria distribuida, como JMS, Kafka ou AMQP.

### 4.2. Eventos padrao do Spring mais usados

O Spring Framework publica alguns eventos padrao ligados ao ciclo de vida do `ApplicationContext` e, em aplicacoes web baseadas em `DispatcherServlet`, tambem eventos ligados ao processamento de requests.

| Evento | Quando ocorre | Uso comum | Observacoes |
| --- | --- | --- | --- |
| `ContextRefreshedEvent` | Quando o `ApplicationContext` e inicializado ou atualizado com `refresh()` | carregar caches, aquecer recursos, validar configuracoes apos subida | pode ocorrer mais de uma vez em contextos que suportam refresh |
| `ContextStartedEvent` | Quando o contexto recebe `start()` | iniciar componentes que dependem do ciclo de vida do contexto | menos comum em apps Spring Boot do dia a dia |
| `ContextStoppedEvent` | Quando o contexto recebe `stop()` | parar componentes de forma controlada | tambem e menos comum em apps Spring Boot convencionais |
| `ContextClosedEvent` | Quando o contexto e fechado com `close()` | liberar recursos, encerrar agendadores, flush final | representa encerramento do ciclo de vida do contexto |
| `RequestHandledEvent` | Quando uma requisicao HTTP foi concluida | auditoria, log tecnico, metricas simples por request | evento especifico de aplicacoes web usando `DispatcherServlet` |
| `ServletRequestHandledEvent` | Variante web com mais detalhes sobre a request servida | observabilidade e auditoria com metodo, URL, tempo e status | herda de `RequestHandledEvent` e costuma ser mais util em apps MVC |

Na pratica:

- `ContextRefreshedEvent` e um dos mais usados para tarefas de inicializacao;
- `ContextClosedEvent` e bastante util para shutdown controlado;
- `RequestHandledEvent` e `ServletRequestHandledEvent` ajudam em cenarios web;
- `ContextStartedEvent` e `ContextStoppedEvent` aparecem mais em cenarios que usam explicitamente o ciclo de vida do contexto.

Exemplo ouvindo um evento padrao do contexto:

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

### 4.3. Publicando eventos com `ApplicationEventPublisher`

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

### 4.4. Consumindo eventos com `@EventListener`

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

Com isso, o `PedidoService` nao precisa conhecer nenhum desses consumidores.

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

Nesse caso, o listener so e executado quando a expressao SpEL retornar `true`.

### 4.5. Eventos sincronos

Por padrao, o processamento de eventos do Spring e sincrono. Isso significa:

- o listener roda na mesma thread do publisher;
- se um listener falhar, a excecao pode afetar o fluxo principal;
- o tempo do listener impacta diretamente o tempo da operacao original.

Esse comportamento e adequado para:

- regras internas pequenas;
- auditoria simples;
- validacoes complementares;
- fluxos que precisam participar da mesma transacao logica.

### 4.6. Eventos assincronos com `@Async`

Quando o processamento nao deve bloquear o fluxo principal, pode-se combinar eventos com execucao assincrona.

Configuracao:

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

Listener assincrono:

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

- excecoes assincronas nao sobem naturalmente para o publisher;
- e importante monitorar falhas e tempo de execucao;
- para tarefas criticas, pode ser melhor usar fila ou broker.

### 4.7. Eventos transacionais com `@TransactionalEventListener`

Quando o evento depende do sucesso da transacao, a melhor opcao costuma ser `@TransactionalEventListener`.

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

- `AFTER_COMMIT`: executa apenas se a transacao confirmar;
- `AFTER_ROLLBACK`: executa se houver rollback;
- `AFTER_COMPLETION`: executa ao final, independentemente do resultado;
- `BEFORE_COMMIT`: executa antes do commit.

Na maioria dos casos de integracao ou notificacao, `AFTER_COMMIT` e a opcao mais segura.

### 4.8. Ordenacao e condicoes

Se houver varios listeners para o mesmo evento, e possivel controlar ordem:

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

Tambem e possivel usar condicoes:

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

### 4.9. Event listeners retornando novos eventos

Um listener tambem pode retornar outro evento, permitindo encadear reacoes:

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

Esse recurso e util, mas deve ser usado com moderacao para nao tornar o fluxo dificil de rastrear.

### 4.10. Comparando Spring Events e JMS

Use Spring Events quando:

- a comunicacao for interna;
- o processamento puder ocorrer no mesmo processo;
- o objetivo principal for desacoplamento entre beans.

Use JMS quando:

- a comunicacao precisar ser distribuida;
- a entrega precisar ser desacoplada do ciclo de vida da aplicacao;
- houver necessidade de broker, retry, fila, topico ou persistencia de mensagens.

Em muitos sistemas corporativos, os dois coexistem:

- Spring Events para comunicacao interna entre services;
- JMS para integracao externa ou comunicacao entre sistemas.

### 4.11. Boas praticas com Spring Events

- mantenha os eventos pequenos e focados no fato de negocio;
- nao coloque regra complexa diretamente no listener;
- prefira nomes orientados a acontecimentos, como `PedidoFaturadoEvent`;
- para integracoes dependentes de commit, use `@TransactionalEventListener`;
- para carga alta ou necessidade de resiliencia, considere mensageria externa;
- documente quais eventos sao internos e quais disparam integracoes.

### 4.12. Eventos padrao relacionados ao Spring Security

O Spring Security tambem publica eventos padrao para autenticacao, autorizacao, logout e aspectos da sessao. Eles sao muito uteis para:

- auditoria de login e logout;
- metricas de falha por credencial invalida, conta bloqueada ou usuario desabilitado;
- monitoramento de negacoes de acesso;
- rastreamento de eventos de seguranca;
- reacoes tecnicas como alertas e trilhas de auditoria.

Na pratica, vale separar mentalmente esses eventos em dois grupos:

- eventos de autenticacao;
- eventos de autorizacao.

#### Configuracao para publicacao dos eventos

Para eventos de autenticacao, a documentacao oficial do Spring Security recomenda registrar um `AuthenticationEventPublisher`, normalmente usando `DefaultAuthenticationEventPublisher`.

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

Para eventos de autorizacao, a documentacao oficial recomenda registrar um `AuthorizationEventPublisher`, por exemplo com `SpringAuthorizationEventPublisher`.

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

#### Eventos padrao mais usados

| Evento | Quando ocorre | Uso comum | Observacoes |
| --- | --- | --- | --- |
| `AuthenticationSuccessEvent` | autenticacao concluida com sucesso | auditoria de login, metricas de sucesso, registro de ultimo acesso | representa sucesso de autenticacao |
| `AbstractAuthenticationFailureEvent` | autenticacao falhou | tratamento centralizado de falhas | classe base para eventos de falha |
| `AuthenticationFailureBadCredentialsEvent` | credenciais invalidas | contador de tentativas erradas, alertas, bloqueio progressivo | muito comum em login por usuario e senha |
| `AuthenticationFailureDisabledEvent` | usuario desabilitado tentou autenticar | auditoria e suporte operacional | bom para monitorar contas desativadas ainda em uso |
| `AuthenticationFailureLockedEvent` | autenticacao falhou porque a conta esta bloqueada | seguranca e suporte | util em fluxos de bloqueio por excesso de tentativas |
| `AuthenticationFailureCredentialsExpiredEvent` | autenticacao falhou porque a credencial expirou | forcar redefinicao de senha | comum em politicas corporativas de senha |
| `InteractiveAuthenticationSuccessEvent` | autenticacao interativa foi bem-sucedida | auditoria de login web real | nao estende `AuthenticationSuccessEvent` para evitar duplicidade |
| `LogoutSuccessEvent` | logout realizado com sucesso | trilha de auditoria, encerramento de sessao | disponivel desde Spring Security 5.2 |
| `SessionFixationProtectionEvent` | o ID da sessao mudou por protecao contra session fixation | monitoramento de seguranca de sessao | informa sessao antiga e nova |
| `AuthorizationDeniedEvent` | uma autorizacao foi negada | auditoria de acesso negado, alertas, metricas | recomendado para negacoes em modelos atuais de autorizacao |
| `AuthorizationGrantedEvent` | uma autorizacao foi concedida | trilha de acesso permitido quando necessario | pode ser habilitado conforme a estrategia de publicacao |
| `AuthenticationSwitchUserEvent` | houve troca de usuario via switch user | auditoria administrativa | util em ambientes com impersonation controlada |

Na maioria dos sistemas, os eventos mais valiosos no inicio sao:

- `AuthenticationSuccessEvent`;
- `AuthenticationFailureBadCredentialsEvent`;
- `LogoutSuccessEvent`;
- `AuthorizationDeniedEvent`.

#### Exemplo de listener para autenticacao

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

#### Exemplo de listener para autorizacao negada

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

#### Exemplo de listener para logout e protecao de sessao

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

#### Eventos legados e eventos atuais de autorizacao

Nas versoes atuais do Spring Security, os eventos legados do pacote `org.springframework.security.access.event` aparecem como descontinuados em varios casos. Para novos projetos, a recomendacao e preferir os eventos modernos:

- `AuthorizationDeniedEvent`;
- `AuthorizationGrantedEvent`.

Isso e especialmente importante para evitar exemplos baseados em APIs antigas de autorizacao.

#### Boas praticas com eventos do Spring Security

- nunca registre senha ou token bruto em listeners;
- trate listeners de auditoria como componentes tecnicos e enxutos;
- use esses eventos para metricas, alertas e trilha de seguranca;
- em cenarios criticos, persista a auditoria de forma assincrona ou desacoplada;
- diferencie falha de autenticacao de negacao de autorizacao;
- ao contar tentativas de login, considere IP, usuario, horario e contexto da aplicacao.

## 5. Mecanismos de resiliencia

Aplicacoes corporativas normalmente dependem de APIs, bancos, brokers e servicos externos. Por isso, e importante aplicar mecanismos de resiliencia para evitar que falhas temporarias derrubem fluxos inteiros ou sobrecarreguem o sistema.

Os mecanismos mais comuns sao:

- timeout;
- retry;
- circuit breaker;
- bulkhead;
- rate limiter;
- fallback.

Em Spring Boot, a combinacao mais comum para esses casos e:

- `spring-retry` para repeticao de tentativas em cenarios simples;
- Resilience4j para circuit breaker, bulkhead, rate limiter, retry e timeout;
- observabilidade para acompanhar falhas, latencia e degradacao.

### 5.1. Quando aplicar resiliencia

Esses mecanismos costumam ser aplicados em chamadas para:

- APIs REST externas;
- filas, brokers e integracoes assincronas;
- operacoes remotas via HTTP ou mensageria;
- consultas a recursos mais lentos ou instaveis.

Nem toda operacao deve ter retry ou fallback. Em geral:

- use retry para falhas transientes;
- use timeout para evitar espera indefinida;
- use circuit breaker quando um servico externo estiver instavel;
- use bulkhead para isolar consumo de recursos;
- use rate limiter quando for necessario controlar volume de chamadas;
- use fallback apenas quando houver uma resposta degradada aceitavel.

### 5.2. Dependencias mais comuns

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

O starter AOP e necessario porque boa parte dessas anotacoes funciona por proxy.

### 5.3. Timeout

O timeout impede que uma chamada fique presa por tempo indeterminado.

Em clientes HTTP, o ideal e combinar:

- timeout no cliente;
- timeout de resiliencia;
- monitoramento de latencia.

Exemplo de configuracao com `RestClient`:

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

Configuracao:

```yaml
resilience4j:
  timelimiter:
    instances:
      consultaExterna:
        timeoutDuration: 3s
        cancelRunningFuture: true
```

### 5.4. Retry com Resilience4j

O retry deve ser usado quando a falha e temporaria, por exemplo:

- timeout ocasional;
- erro 503 de servico momentaneamente indisponivel;
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

Configuracao:

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

- evitar retry em erro funcional, como validacao;
- evitar retry infinito;
- combinar retry com timeout;
- observar o impacto de repeticao em operacoes nao idempotentes.

### 5.5. Retry com Spring Retry

Para casos mais simples, o Spring Retry ainda e uma opcao bastante pratica.

Habilitacao:

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
        throw new IllegalStateException("Falha temporaria no servico de estoque");
    }

    @Recover
    public String recuperar(Exception ex, String sku) {
        return "Nao foi possivel atualizar o saldo do SKU " + sku;
    }
}
```

Essa abordagem e boa quando o foco e apenas repeticao de tentativa em um service local.

### 5.6. Circuit breaker

O circuit breaker evita que a aplicacao continue insistindo em um servico que esta falhando de forma recorrente.

Estados principais:

- `CLOSED`: operacao normal;
- `OPEN`: chamadas bloqueadas temporariamente;
- `HALF_OPEN`: algumas chamadas de teste sao liberadas.

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

Configuracao:

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

Esse mecanismo protege a aplicacao e tambem reduz carga sobre o sistema externo problemático.

### 5.7. Bulkhead

O bulkhead isola recursos para que a falha ou lentidao de uma integracao nao consuma todas as threads da aplicacao.

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
        return "servico temporariamente sobrecarregado";
    }
}
```

Configuracao:

```yaml
resilience4j:
  bulkhead:
    instances:
      consultaDocumento:
        maxConcurrentCalls: 10
        maxWaitDuration: 100ms
```

Quando usar:

- integracoes lentas;
- operacoes que podem monopolizar threads;
- isolamento entre fluxos criticos e secundarios.

### 5.8. Rate limiter

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

Configuracao:

```yaml
resilience4j:
  ratelimiter:
    instances:
      consultaCep:
        limitForPeriod: 5
        limitRefreshPeriod: 1s
        timeoutDuration: 0
```

Esse mecanismo e util para:

- proteger APIs com quota;
- evitar estouro de consumo em parceiros externos;
- reduzir picos causados por rajadas de requisicoes.

### 5.9. Rate limiter com Bucket4j

Outra abordagem bastante popular em Spring Boot e usar Bucket4j, uma biblioteca baseada no algoritmo de token bucket. Ela e especialmente util quando o limite precisa ser aplicado diretamente sobre endpoints HTTP, usuarios, IPs ou chaves de negocio.

Bucket4j funciona bem quando:

- o controle de taxa precisa ser muito explicito;
- o limite deve ser calculado por IP, usuario, token ou tenant;
- o projeto quer uma politica de rate limit independente de um cliente HTTP especifico;
- ha necessidade de evoluir depois para buckets distribuidos.

Segundo a documentacao oficial do Bucket4j, a dependencia principal para Java 17+ e `com.bucket4j:bucket4j_jdk17-core`. O proprio projeto tambem destaca que, para casos genericos com Spring Boot, existe um starter dedicado baseado em configuracao, mas o uso direto da biblioteca e muito bom quando queremos controle fino em codigo.

#### Dependencia

```xml
<dependency>
    <groupId>com.bucket4j</groupId>
    <artifactId>bucket4j_jdk17-core</artifactId>
    <version>8.15.0</version>
</dependency>
```

#### Exemplo simples em memoria

Configuracao de um bucket com capacidade de 20 requisicoes por minuto:

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

Servico para obter buckets por chave, por exemplo IP ou usuario:

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

- proteger endpoints publicos;
- limitar chamadas por IP;
- aplicar politicas simples sem depender de gateway externo.

#### Rate limit por usuario, token ou tenant

Em vez de usar IP, a chave pode ser:

- login do usuario autenticado;
- API key;
- identificador do tenant;
- combinacao de usuario e endpoint.

Exemplo conceitual:

```java
String chave = tenantId + ":" + username + ":" + request.getRequestURI();
Bucket bucket = rateLimitService.resolveBucket(chave);
```

Isso permite politicas diferentes por perfil ou plano comercial.

#### Exemplo de resposta HTTP 429 enriquecida

Uma boa pratica e devolver cabecalhos que ajudem o cliente a entender o bloqueio:

- `X-Rate-Limit-Remaining`;
- `X-Rate-Limit-Retry-After-Seconds`;
- eventualmente um corpo JSON com detalhes.

Exemplo:

```json
{
  "status": 429,
  "erro": "TOO_MANY_REQUESTS",
  "mensagem": "Limite de requisicoes excedido para este cliente."
}
```

#### Buckets distribuidos

O exemplo anterior usa memoria local e funciona bem em:

- aplicacoes simples;
- ambientes de desenvolvimento;
- uma unica instancia.

Em producao com varias instancias, o ideal e compartilhar o estado do bucket usando tecnologia distribuida, como cache ou armazenamento compativel com os modulos distribuidos suportados pelo Bucket4j. A documentacao oficial destaca suporte para cenarios distribuidos com integracoes como JCache, Hazelcast, Ignite, Infinispan e Coherence.

Nesses casos, o objetivo e fazer todas as instancias enxergarem o mesmo consumo de tokens.

#### Quando preferir Bucket4j em vez de `RateLimiter` do Resilience4j

Bucket4j costuma ser uma escolha melhor quando:

- o rate limit esta ligado a requisicoes HTTP de entrada;
- a chave do limite depende do cliente chamador;
- o controle precisa ser por IP, usuario ou API key;
- o bloqueio deve ocorrer antes de chegar na regra de negocio.

O `RateLimiter` do Resilience4j costuma ser mais natural quando:

- queremos limitar chamadas de saida para um servico externo;
- o controle esta acoplado a um metodo especifico;
- a politica de resiliencia faz parte de uma chamada remota do service.

### 5.10. Fallback

O fallback define o comportamento degradado quando a chamada principal falha.

Exemplos de fallback aceitavel:

- retornar cache;
- retornar status simplificado;
- marcar processamento como pendente;
- enfileirar para tentativa posterior;
- responder com informacao parcial.

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

- fallback nao deve mascarar erro grave sem observabilidade;
- a resposta degradada precisa ser coerente com o negocio;
- em fluxos financeiros ou criticos, talvez o correto seja falhar explicitamente.

### 5.11. Combinando mecanismos

Em muitos cenarios, os mecanismos sao usados juntos. Um arranjo comum para chamadas HTTP externas e:

1. timeout curto;
2. retry controlado;
3. circuit breaker;
4. fallback;
5. monitoramento por metricas e logs.

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

Ao combinar mecanismos, evite exagero. Muitas camadas de resiliencia mal calibradas podem aumentar latencia, mascarar problemas ou gerar tempestade de tentativas.

### 5.12. Resiliencia em consumidores JMS e jobs

Os mesmos conceitos valem para jobs e mensageria:

- listeners JMS podem usar retry controlado no processamento;
- jobs Quartz podem aplicar circuit breaker ao chamar servicos externos;
- falhas recorrentes podem redirecionar para dead letter queue ou reprocessamento posterior;
- operacoes demoradas devem ter timeout e isolamento de recursos.

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

### 5.13. Observabilidade e monitoramento

Nao existe resiliencia boa sem visibilidade operacional. O ideal e acompanhar:

- quantidade de retries;
- circuit breakers abertos;
- chamadas lentas ou expiradas;
- taxa de fallback;
- volume rejeitado por bulkhead ou rate limiter.

Isso ajuda a responder perguntas como:

- o servico externo esta instavel ou apenas lento?
- o retry esta ajudando ou piorando?
- o fallback esta sendo usado demais?
- o gargalo esta no provedor externo ou na propria aplicacao?

### 5.14. Boas praticas

- aplique resiliencia apenas onde existe dependencia instavel ou remota;
- diferencie erro transiente de erro funcional;
- prefira operacoes idempotentes quando houver retry;
- use timeout curto o suficiente para proteger a aplicacao;
- nao use fallback enganoso em fluxos criticos;
- monitore tudo o que puder degradar silenciosamente;
- revise periodicamente os parametros de retry, timeout e circuit breaker.

## 6. Cache e Spring Session

Cache e gerenciamento de sessao sao dois temas muito relevantes em aplicacoes Spring Boot tradicionais. Embora resolvam problemas diferentes, eles frequentemente usam tecnologias parecidas, como Redis:

- cache melhora desempenho e reduz chamadas repetidas;
- sessao distribuida permite que o estado do usuario sobreviva entre instancias da aplicacao.

Em geral:

- use cache para dados repetidos e relativamente estaveis;
- use Spring Session para estado de usuario em aplicacoes web stateful;
- evite misturar os dois conceitos no desenho de chaves e TTLs.

### 8.1. Dependencias

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

Para cache com Redis e sessao distribuida:

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

### 6.2. Habilitando cache

```java
package br.com.exemplo.cache;

import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CacheConfig {
}
```

Com `@EnableCaching`, o Spring passa a interceptar os metodos anotados com `@Cacheable`, `@CachePut` e `@CacheEvict`.

### 6.3. Uso de `@Cacheable`

`@Cacheable` e usado quando queremos:

- consultar primeiro no cache;
- executar o metodo apenas se a chave ainda nao existir;
- armazenar o retorno para reutilizacao posterior.

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
                .orElseThrow(() -> new IllegalArgumentException("Produto nao encontrado"));
    }
}
```

Observacoes importantes:

- a primeira chamada consulta a fonte original;
- chamadas seguintes com a mesma chave usam o cache;
- o metodo precisa ser chamado via proxy Spring para a anotacao funcionar.

Tambem e possivel usar `unless` e `condition`:

```java
@Cacheable(cacheNames = "produtos", key = "#id", unless = "#result == null")
public ProdutoDto buscarPorId(Long id) {
    // ...
}
```

### 6.4. Uso de `@CachePut`

`@CachePut` sempre executa o metodo e atualiza o cache com o retorno mais recente.

Ele e util quando:

- o dado acabou de ser alterado;
- queremos manter o cache sincronizado sem depender de nova leitura;
- o retorno do metodo ja representa o estado final salvo.

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
                .orElseThrow(() -> new IllegalArgumentException("Produto nao encontrado"));

        produto.setNome(request.nome());
        produto.setPreco(request.preco());

        Produto salvo = produtoRepository.save(produto);
        return new ProdutoDto(salvo.getId(), salvo.getNome(), salvo.getPreco());
    }
}
```

### 6.5. Uso de `@CacheEvict`

`@CacheEvict` remove entradas do cache, o que e importante quando o dado original mudou ou foi excluido.

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

### 6.6. Configuracao com Caffeine

Caffeine e uma excelente opcao para cache local em memoria:

- muito rapido;
- simples de operar;
- ideal para uma unica instancia ou para caches que podem divergir brevemente entre instancias;
- nao serve como cache compartilhado entre nos.

Configuracao simples:

```yaml
spring:
  cache:
    type: caffeine
    cache-names: produtos,categorias
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=10m,recordStats
```

Tambem e possivel customizar via `CaffeineCacheManager`:

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

- aplicacao monolitica em uma instancia;
- dados muito consultados e pouco alterados;
- necessidade de latencia minima;
- cache local como camada complementar.

### 6.7. Configuracao com Redis para cache

Redis e uma boa opcao quando:

- ha varias instancias da aplicacao;
- o cache precisa ser compartilhado;
- o TTL precisa ser centralizado;
- a invalidação precisa refletir entre varios nos.

Configuracao base:

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

Customizacao mais detalhada:

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

- manter `use-key-prefix` ajuda a evitar colisao entre caches;
- diferentes caches podem ter TTLs diferentes;
- serializacao JSON costuma ser mais amigavel para manutencao e diagnostico.

### 6.8. Estrategias de invalidação

Cache sem estrategia de invalidacao costuma gerar inconsistencias.

Estrategias comuns:

- TTL curto para dados mais volateis;
- `@CacheEvict` ao atualizar ou excluir dados;
- `@CachePut` para refresh imediato;
- `allEntries = true` para invalidacoes amplas;
- invalidacao por evento de negocio;
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

Boas praticas:

- nao cachear tudo indiscriminadamente;
- separar cache de leitura de cache derivado;
- documentar TTL e estrategia de remocao;
- medir hit ratio antes de expandir uso de cache.

### 6.9. Cache em aplicacoes Spring Boot tradicionais

Em aplicacoes web tradicionais, o cache costuma ser aplicado em:

- consultas de catalogo;
- configuracoes de negocio;
- parametros e tabelas auxiliares;
- resultados de chamadas externas;
- dados frequentemente acessados por controllers e services.

Uma combinacao pratica e:

- Caffeine para cache local de baixa latencia;
- Redis para cache compartilhado entre instancias;
- invalidacao orientada a evento ou alteracao de dados.

### 6.10. Spring Session e sessao distribuida

Spring Session abstrai o armazenamento da sessao HTTP e permite mover esse estado para um backend compartilhado, como Redis ou JDBC.

Isso e especialmente util quando:

- ha mais de uma instancia da aplicacao;
- o load balancer nao usa sticky session;
- o usuario precisa manter a sessao ao trocar de instancia;
- o estado da autenticacao precisa sobreviver em ambiente distribuido.

Segundo a documentacao oficial do Spring Boot, em aplicacoes servlet o store de sessao pode ser auto-configurado com Redis ou JDBC, sem necessidade de `@Enable*HttpSession` quando ha um unico modulo Spring Session no classpath.

### 6.11. Configurando Spring Session com Redis

Exemplo para aplicacao web tradicional:

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

Em muitos cenarios, apenas adicionar a dependencia `spring-session-data-redis` ja faz o Spring Boot configurar automaticamente a substituicao do `HttpSession`.

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
- a aplicacao segue com a mesma API de sessao.

### 6.12. Persistencia em Redis e integracao com Spring Security

Em aplicacoes web com login tradicional, o Spring Session integra-se muito bem com Spring Security. Quando o `SecurityContext` e salvo na sessao, ele tambem passa a ser persistido no Redis.

Isso permite:

- manter autenticacao entre instancias;
- invalidar sessoes de forma centralizada;
- acompanhar sessoes ativas de forma mais controlada.

Segundo a documentacao do Spring Session, um cookie `SESSION` e usado para apontar para a sessao persistida. O backend Redis armazena os atributos da sessao e cuida da expiracao conforme a configuracao.

### 6.13. Estrategias para aplicacoes web tradicionais

Em aplicacoes web stateful, algumas estrategias sao bastante comuns:

- sessao local do servlet container para ambientes simples e uma unica instancia;
- sticky session no balanceador para reduzir troca de sessao entre nos;
- Spring Session com Redis para alta disponibilidade e escalabilidade horizontal;
- reducao do tamanho da sessao para evitar excesso de serializacao.

Recomendacoes praticas:

- manter na sessao apenas o necessario;
- nao guardar objetos grandes ou grafo pesado;
- preferir IDs e dados essenciais em vez de entidades completas;
- configurar timeout coerente com a experiencia esperada;
- invalidar sessao explicitamente no logout.

### 6.14. Encontrando e invalidando sessoes por usuario

Quando a aplicacao precisa listar sessoes ativas de um usuario ou derrubar sessoes remotamente, o Spring Session com repositorio indexado ajuda bastante.

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

Esse tipo de recurso e especialmente util em:

- painis administrativos;
- forca de logout;
- controle de sessoes simultaneas;
- auditoria de acesso.

### 6.15. Eventos de sessao

Quando o repositorio indexado esta configurado, o Spring Session tambem pode publicar eventos como:

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

### 6.16. Boas praticas com cache e Spring Session

- escolha Caffeine para cache local e Redis para cache ou sessao compartilhada;
- nao use cache como substituto de consistencia de negocio;
- defina TTL e estrategia de invalidacao antes de ativar o cache;
- monitore hit ratio, latencia e volume de chaves;
- mantenha sessoes pequenas e controladas;
- em aplicacoes distribuidas, prefira Spring Session com backend compartilhado;
- teste invalidacao de cache e expiracao de sessao em cenarios reais;
- evite salvar objetos nao serializaveis ou muito grandes na sessao.

## 7. Spring Batch

Spring Batch e o modulo do ecossistema Spring voltado para processamento em lote. Ele e muito util quando a aplicacao precisa processar grande volume de dados de forma controlada, reexecutavel e auditavel.

Casos comuns:

- importacao de arquivos;
- conciliacao financeira;
- processamento noturno;
- geracao de relatorios pesados;
- carga ou migracao de dados;
- reprocessamento de registros com falha.

Segundo a documentacao oficial, alguns dos componentes centrais sao:

- `JobRepository`, responsavel por persistir metadados de execucao;
- `JobLauncher`, usado para disparar jobs com `JobParameters`;
- `ItemReader`, para ler itens;
- `ItemProcessor`, para transformar ou validar itens;
- `ItemWriter`, para gravar itens.

### 7.1. Dependencias

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

Se o projeto vai usar persistencia real dos metadados do Batch, normalmente tambem havera um banco com `spring-boot-starter-data-jdbc` ou `spring-boot-starter-data-jpa`, dependendo da arquitetura adotada.

### 7.2. Conceitos principais

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

### 7.3. Configuracao basica de Job e Step

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

### 7.4. Exemplo completo de `ItemReader`, `ItemProcessor` e `ItemWriter`

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

Reader simples em memoria:

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

Quando o `ItemProcessor` retorna `null`, o item e filtrado e nao segue para o writer.

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

### 7.5. Leitura de arquivo CSV com Spring Batch

Um dos cenarios mais comuns e importar dados de CSV com `FlatFileItemReader`.

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

Esse tipo de reader e bastante usado em importacoes administrativas ou integracoes legadas.

### 7.6. Writer para banco de dados

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

### 7.7. Restart e reprocessamento

Um dos grandes diferenciais do Spring Batch e a persistencia dos metadados de execucao no `JobRepository`. Isso permite:

- saber o que ja rodou;
- identificar falhas por job e step;
- retomar execucao quando o job e reiniciavel;
- evitar reprocessamento indevido do mesmo `JobInstance`.

Na pratica:

- um `JobInstance` e identificado pelo nome do job e seus `JobParameters`;
- mudar os parametros cria uma nova instancia;
- relancar com os mesmos parametros pode resultar em restart, se o job permitir.

Exemplo de disparo com parametro:

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

Se o objetivo for sempre criar uma nova execucao, adicionar um `timestamp` ou identificador unico nos parametros e uma abordagem comum.

### 7.8. Skip e retry

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

### 7.9. Tasklet step

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
- nao houver processamento item a item;
- a logica for unica, como mover arquivo, limpar diretorio ou gerar marcador de execucao.

### 7.10. Boas praticas com Spring Batch

- persista metadados do Batch em banco relacional;
- mantenha jobs idempotentes sempre que possivel;
- use `JobParameters` conscientemente para distinguir instancias;
- prefira chunk para alto volume e `Tasklet` para tarefas pontuais;
- trate skip e retry com regras de negocio claras;
- registre metricas e auditoria das execucoes;
- para jobs pesados, considere disparo por scheduler ou operacao administrativa;
- para arquivos grandes, combine Spring Batch com leitores apropriados, como CSV ou leitura streaming de XLSX.

### 7.11. Quando Spring Batch faz mais sentido

Spring Batch costuma ser uma excelente escolha quando:

- ha volume relevante de dados;
- o processamento precisa de restart controlado;
- existe necessidade de trilha de execucao;
- o fluxo precisa de chunk, skip, retry e gerenciamento de falha.

Em contrapartida, talvez ele seja excessivo quando:

- a tarefa e muito simples e eventual;
- um service comum ou job Quartz resolve com menos complexidade;
- nao ha estado de execucao, checkpoint ou necessidade de restart.

## 8. Exportacao de dados em CSV e Excel

Exportacao de dados e uma necessidade muito comum em sistemas corporativos, especialmente para:

- relatorios administrativos;
- extracao de dados para analise;
- integracao manual com planilhas;
- conferencias operacionais;
- disponibilizacao de dados para usuarios de negocio.

Em Spring Boot, duas bibliotecas muito comuns para esse cenario sao:

- Apache Commons CSV para geracao de arquivos CSV;
- Apache POI para geracao de arquivos Excel, principalmente `.xlsx`.

Conforme a documentacao oficial do Apache Commons CSV, a biblioteca e voltada para leitura e escrita de variacoes do formato CSV. Ja o Apache POI informa oficialmente que `poi` cobre o ecossistema OLE2 e `poi-ooxml` cobre os formatos OOXML, incluindo Excel `.xlsx`.

### 6.1. Dependencias

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
- manter `poi` explicito pode ser util para deixar claro o suporte ao ecossistema base do POI.

### 8.2. Quando usar CSV e quando usar Excel

Use CSV quando:

- o objetivo for simplicidade;
- o arquivo for consumido por varios sistemas;
- nao houver necessidade de formatacao visual;
- o volume de dados for grande e o arquivo precisar ser leve.

Use Excel quando:

- houver necessidade de multiplas abas;
- for importante aplicar formatacao, estilos e larguras de coluna;
- usuarios de negocio esperarem uma planilha mais amigavel;
- houver necessidade de formulas, filtros ou organizacao visual.

### 8.3. Exemplo de DTO para exportacao

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

### 8.4. Exportando CSV com Apache Commons CSV

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

- o delimitador pode ser `,` ou `;`, dependendo do padrao esperado;
- em cenarios brasileiros, `;` e comum por compatibilidade com Excel em algumas configuracoes regionais;
- usar UTF-8 evita problemas com acentos;
- `CSVPrinter` ajuda a lidar corretamente com escape e aspas.

### 8.5. Lendo arquivos CSV com Apache Commons CSV

Para importacao de dados, o Apache Commons CSV tambem oferece uma API simples e segura para parsing de arquivos CSV.

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

### 8.6. Endpoint HTTP para download de CSV

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

Esse controller mostra os dois formatos no mesmo endpoint de exportacao.

### 8.7. Exportando Excel com Apache POI

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

### 8.8. Lendo arquivos XLSX com Apache POI

Quando o arquivo Excel nao e muito grande, `XSSFWorkbook` ou `WorkbookFactory` costumam ser suficientes para leitura.

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
- e necessario aproveitar melhor a API completa do POI;
- o processamento exige mais liberdade de navegacao pela planilha.

Para leitura, `DataFormatter` e util porque ajuda a obter o valor textual exibido na celula sem precisar tratar todos os tipos manualmente em cada caso.

### 8.9. Diferencas entre `XSSFWorkbook` e `SXSSFWorkbook`

Na pratica, os dois servem para gerar arquivos `.xlsx`, mas o comportamento e o custo de memoria sao diferentes.

#### Quando usar `XSSFWorkbook`

`XSSFWorkbook` e a implementacao tradicional do Apache POI para arquivos `.xlsx`. Ela carrega e manipula a planilha em memoria, o que a torna mais simples e mais flexivel.

Use `XSSFWorkbook` quando:

- o volume de linhas for moderado;
- houver necessidade de leitura e escrita com acesso aleatorio;
- o relatorio usar muitos recursos do Excel;
- for importante aplicar estilos, formulas, comentarios, merges e outras operacoes mais ricas;
- a simplicidade do codigo for mais importante que a economia maxima de memoria.

Vantagens:

- API mais direta;
- acesso completo a linhas e celulas ja criadas;
- melhor para planilhas pequenas e medias;
- mais confortavel para cenarios com bastante formatacao.

Limites praticos:

- consome mais memoria;
- pode ficar pesado em exportacoes muito grandes;
- para arquivos extensos, o risco de lentidao e `OutOfMemoryError` aumenta.

#### Quando usar `SXSSFWorkbook`

`SXSSFWorkbook` e a versao streaming do `XSSFWorkbook`. Segundo a documentacao oficial do Apache POI, ele permite gravar arquivos muito grandes sem esgotar memoria, mantendo em memoria apenas uma janela configuravel de linhas.

Use `SXSSFWorkbook` quando:

- a exportacao tiver muitas linhas;
- o objetivo principal for escrita sequencial;
- a aplicacao precisar reduzir consumo de memoria;
- o relatorio for grande, mas relativamente simples do ponto de vista visual.

Vantagens:

- muito mais adequado para grandes volumes;
- mantem apenas parte das linhas em memoria;
- reduz risco de estouro de memoria em exportacoes extensas.

Cuidados importantes:

- depois que linhas antigas sao descarregadas, elas nao ficam mais disponiveis para acesso aleatorio;
- o POI usa arquivos temporarios em disco durante a geracao;
- recursos como comentarios e regioes mescladas ainda podem consumir memoria significativa;
- ao final, e importante limpar os arquivos temporarios com `dispose()`.

#### Resumo pratico

`XSSFWorkbook`:

- melhor para arquivos pequenos e medios;
- melhor quando ha muita formatacao e manipulacao posterior;
- mais simples para relatorios ricos.

`SXSSFWorkbook`:

- melhor para exportacao massiva;
- melhor para escrita sequencial de grandes volumes;
- melhor quando memoria e um fator critico.

Uma regra pratica bastante usada:

- ate algumas milhares de linhas, `XSSFWorkbook` costuma ser suficiente;
- para dezenas ou centenas de milhares de linhas, vale avaliar `SXSSFWorkbook`.

Essa regra e apenas uma referencia. O ponto real de troca depende da memoria disponivel, da quantidade de colunas e da complexidade da planilha.

### 8.10. Exemplo de exportacao com `SXSSFWorkbook`

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

- `new SXSSFWorkbook(100)` mantem apenas uma janela de 100 linhas em memoria;
- `setCompressTempFiles(true)` reduz o impacto dos arquivos temporarios em disco;
- `dispose()` remove os arquivos temporarios criados pelo streaming workbook.

Para exportacoes muito grandes, esse modelo costuma ser mais seguro que `XSSFWorkbook`.

### 8.11. Leitura de arquivos grandes com `excel-streaming-reader`

Quando o problema nao e exportar, mas sim importar ou ler planilhas `.xlsx` muito grandes, uma opcao bastante recomendada e o fork mantido por `pjfanning` da biblioteca `excel-streaming-reader`.

Esse fork e a evolucao mais atual da implementacao original e, conforme o README oficial, suporta Apache POI 5.x e Java 8+, alem de trazer correcoes e recursos adicionais.

Ela funciona como um wrapper sobre a leitura streaming do Apache POI e preserva uma API parecida com `Workbook`, `Sheet`, `Row` e `Cell`, mas com foco em baixo consumo de memoria.

Ela costuma ser util quando:

- o arquivo Excel recebido tem muitas linhas;
- a aplicacao precisa importar planilhas grandes;
- nao e viavel carregar o workbook inteiro em memoria;
- o fluxo de leitura pode ser sequencial.

Segundo o README oficial:

- a biblioteca suporta apenas arquivos `.xlsx`;
- ela foi feita para leitura streaming, nao para escrita;
- o acesso aleatorio a linhas nao esta disponivel como em um workbook tradicional;
- nem todos os metodos do POI sao suportados;
- o pacote Java mudou em relacao ao projeto original;
- o fork atual oferece opcoes extras para shared strings, comments e suporte melhorado a formatos OOXML Strict.

#### Dependencia

```xml
<dependency>
    <groupId>com.github.pjfanning</groupId>
    <artifactId>excel-streaming-reader</artifactId>
    <version>5.2.0</version>
</dependency>
```

Se o projeto usar algumas implementacoes avancadas de shared strings, o README oficial informa que pode ser necessario adicionar tambem a dependencia `poi-shared-strings`.

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

- `rowCacheSize(100)` define quantas linhas ficam em memoria;
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
- o consumo de memoria com shared strings comeca a crescer;
- a aplicacao aceita usar arquivos temporarios para aliviar RAM.

#### Comparando com `XSSFWorkbook`

Use `excel-streaming-reader` quando:

- o objetivo principal for leitura de arquivos `.xlsx` grandes;
- a leitura puder ser sequencial;
- o baixo consumo de memoria for prioridade.

Use `XSSFWorkbook` quando:

- o arquivo for pequeno ou medio;
- houver necessidade de recursos mais completos do POI;
- o codigo precisar de acesso mais livre ao workbook;
- o fluxo envolver leitura e manipulacao rica da planilha.

#### Limitacoes importantes

- funciona apenas com `.xlsx`;
- nao serve para gerar planilhas;
- alguns recursos avancados do POI nao estao disponiveis;
- como a leitura e streaming, linhas antigas nao ficam livremente acessiveis;
- a biblioteca usa arquivo temporario ao trabalhar com `InputStream`;
- para arquivos muito grandes, o proprio README recomenda favorecer o uso de arquivos temporarios.

Na pratica:

- `XSSFWorkbook` e melhor para leitura rica e planilhas menores;
- `SXSSFWorkbook` e melhor para escrita streaming de grandes arquivos;
- `excel-streaming-reader` e melhor para leitura streaming de `.xlsx` grandes.

### 8.12. Comparando abordagens de leitura para XLSX

Resumo pratico:

- `WorkbookFactory` ou `XSSFWorkbook`: melhor para arquivos pequenos e medios, com API completa;
- `excel-streaming-reader`: melhor para leitura de arquivos grandes com menor consumo de memoria;
- `SXSSFWorkbook`: nao e biblioteca de leitura; ele e voltado para escrita streaming.

Escolha rapida:

- importar planilha pequena ou media: `XSSFWorkbook` ou `WorkbookFactory`;
- importar planilha muito grande: `excel-streaming-reader`;
- gerar planilha muito grande: `SXSSFWorkbook`.

### 8.13. Exportacao com mais de uma aba

Quando o relatorio e mais complexo, uma planilha Excel pode conter varias abas.

Exemplo conceitual:

```java
Sheet resumoSheet = workbook.createSheet("Resumo");
Sheet pedidosSheet = workbook.createSheet("Pedidos");
Sheet auditoriaSheet = workbook.createSheet("Auditoria");
```

Isso e util quando:

- o arquivo precisa separar visoes do mesmo relatorio;
- existem dados analiticos e resumo executivo;
- o usuario precisa navegar por categorias ou periodos.

### 8.14. Cuidados com memoria e volume

Arquivos CSV geralmente sao leves, mas planilhas Excel podem consumir bastante memoria, principalmente com muitas linhas e estilos.

Boas praticas:

- para grandes volumes, preferir CSV quando possivel;
- limitar quantidade de estilos diferentes no Excel;
- evitar manter muitos arquivos grandes em memoria ao mesmo tempo;
- considerar exportacao assincrona para arquivos muito grandes;
- para escrita massiva em `.xlsx`, avaliar `SXSSFWorkbook`;
- lembrar que `SXSSFWorkbook` cria arquivos temporarios e exige limpeza adequada.

### 8.15. Diferencas praticas entre CSV e Excel

CSV:

- mais simples;
- mais leve;
- mais facil para integracao entre sistemas;
- sem estilos, abas ou formulas.

Excel:

- mais amigavel para usuarios finais;
- suporta formatacao, formulas e varias abas;
- normalmente mais pesado;
- mais apropriado para relatorios gerenciais.

### 8.16. Boas praticas para exportacao

- usar nomes de arquivo claros, por exemplo `pedidos-2026-03-19.xlsx`;
- aplicar `Content-Disposition: attachment`;
- definir o `Content-Type` correto;
- evitar expor colunas sensiveis sem necessidade;
- considerar paginacao, filtros e exportacao assicrona para grandes volumes;
- registrar auditoria quando a exportacao envolver dados sensiveis.

## 9. Upload e download de arquivos

Upload e download de arquivos sao requisitos muito comuns em aplicacoes Spring Boot tradicionais, especialmente em cenarios como:

- envio de documentos;
- anexos de processos;
- importacao manual de arquivos;
- download de comprovantes, relatorios e templates;
- armazenamento local ou em servicos externos.

Os cuidados principais normalmente envolvem:

- validacao de tipo e tamanho;
- seguranca do nome do arquivo;
- streaming para nao sobrecarregar memoria;
- estrategia de armazenamento;
- autorizacao de acesso ao arquivo.

### 9.1. Suporte basico a multipart no Spring Boot

Em aplicacoes servlet, o Spring Boot configura automaticamente o suporte a upload multipart quando o suporte multipart esta habilitado e as classes necessarias estao no classpath.

Configuracao comum:

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
- `max-request-size` limita o total da requisicao multipart;
- `file-size-threshold` define quando o upload vai para disco temporario;
- `location` define onde os temporarios podem ser gravados.

### 9.2. Upload simples com `MultipartFile`

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

### 9.3. Armazenamento local de arquivos

Servico de armazenamento local:

```java
package br.com.exemplo.arquivos;

import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.UUID;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
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

Esse modelo e util quando:

- o volume de arquivos e moderado;
- a aplicacao roda em um ambiente com disco acessivel;
- nao ha necessidade imediata de armazenamento externo.

### 9.4. Validacao de tipo e tamanho

Validacoes importantes no upload:

- tamanho maximo;
- content type esperado;
- extensao permitida;
- nome seguro;
- conteudo realmente compativel com o tipo declarado.

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
            throw new IllegalArgumentException("Tipo de arquivo nao permitido");
        }
    }
}
```

Em cenarios sensiveis, o ideal e validar tambem a assinatura binaria do arquivo e nao apenas a extensao ou o content type informado pelo cliente.

### 9.5. Upload com metadados

Muitas vezes o upload vem junto com outros campos do formulario.

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

### 9.6. Upload de JSON e arquivo com `@RequestPart`

Para APIs REST, essa costuma ser a abordagem mais recomendada quando a requisicao envia:

- uma part JSON estruturada;
- uma part com o arquivo binario.

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
- essa abordagem e mais explicita e previsivel para APIs REST.

### 9.7. Fetch API para `/upload/v1` com `@RequestPart`

Quando o endpoint usa `@RequestPart("data")` para receber JSON e `@RequestPart("binaryFile")` para receber o arquivo, um ponto importante no frontend e que o JSON deve ser enviado como `Blob` com `type: "application/json"`.

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

Pontos de atencao:

- o campo `data` precisa ser enviado como `Blob` com `Content-Type` `application/json`;
- se o JSON for enviado apenas como texto puro no `FormData`, o binding com `@RequestPart` pode nao ocorrer como esperado;
- o campo `binaryFile` deve ter exatamente o mesmo nome esperado no backend;
- nao se deve definir manualmente o header `Content-Type` da requisicao, pois o navegador monta automaticamente o `multipart/form-data` com boundary.

Esse detalhe do `Blob` e especialmente importante em integracoes com Spring MVC quando a API recebe JSON estruturado e arquivo na mesma requisicao.

### 9.8. Upload de JSON e arquivo com `@ModelAttribute`

Em alguns cenarios, a aplicacao recebe um `multipart/form-data` com:

- um campo textual contendo JSON;
- um arquivo binario;
- binding via `@ModelAttribute`.

Essa abordagem pode funcionar, mas exige mais cuidado do que `@RequestPart`, porque o Spring nao vai desserializar automaticamente a string JSON para um objeto complexo apenas com base no `Content-Type` da part. Para isso, normalmente e preciso usar um binder ou conversor customizado.

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
        // Necessario porque, com @ModelAttribute, o campo "data" chega como texto no multipart
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

### 9.9. Fetch API para `/upload/v2` com `@ModelAttribute`

Quando o backend usa `@ModelAttribute`, o JSON normalmente deve ser enviado como texto simples no `FormData`, e nao como `Blob` `application/json`.

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

Pontos de atencao:

- o campo `data` vai como texto comum;
- a conversao depende do `@InitBinder` ou mecanismo equivalente;
- se o nome do campo nao for exatamente `data`, o binding falha;
- essa abordagem e menos explicita para APIs REST.

### 9.10. Comparando `@ModelAttribute` e `@RequestPart`

`@ModelAttribute`:

- melhor para formularios multipart mais tradicionais;
- funciona bem quando os campos sao simples ou achatados;
- para JSON estruturado, exige binder ou conversao customizada;
- tende a ser mais verboso e menos explicito em APIs REST.

`@RequestPart`:

- melhor para APIs que recebem uma part JSON e outra part de arquivo;
- usa os `HttpMessageConverters` do Spring e o Jackson de forma natural;
- e mais previsivel para integracoes SPA, mobile e clientes externos;
- costuma ser a abordagem mais recomendada para `JSON + arquivo`.

Na pratica:

- se o caso for formulario classico, `@ModelAttribute` pode ser suficiente;
- se o caso for API REST com JSON estruturado, prefira `@RequestPart`.

### 9.11. Upload resumivel com TUS

TUS e um protocolo HTTP aberto para uploads resumiveis. Ele e especialmente util quando:

- os arquivos podem ser grandes;
- a conexao do usuario pode oscilar;
- o upload precisa ser pausado e retomado;
- o cliente pode trocar de rede ou recarregar a pagina;
- a aplicacao precisa reduzir o risco de reiniciar upload do zero.

Segundo a especificacao oficial do protocolo:

- o cliente cria ou conhece um recurso de upload;
- usa `HEAD` para descobrir o `Upload-Offset`;
- usa `PATCH` para continuar enviando a partir do ponto correto;
- o header `Tus-Resumable` identifica a versao do protocolo;
- extensoes como `creation`, `termination` e `checksum` podem ser negociadas.

Fluxo basico:

1. o cliente cria o upload com `POST`;
2. o servidor devolve a URL do upload;
3. o cliente envia partes com `PATCH`;
4. se houver interrupcao, o cliente consulta o offset com `HEAD`;
5. o upload continua do ponto em que parou.

#### Quando preferir TUS em vez de multipart comum

`MultipartFile` tradicional costuma ser suficiente quando:

- os arquivos sao pequenos ou medios;
- o upload e rapido;
- nao ha grande risco de interrupcao.

TUS costuma ser melhor quando:

- os arquivos sao grandes;
- ha upload via browser em rede instavel;
- a experiencia de retomada e importante;
- o upload pode durar varios minutos.

#### Exemplo de headers do protocolo

Criacao do upload:

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

Continuacao do envio:

```http
PATCH /api/tus/files/24e533e02ec3bc40c387f1a0e460e216 HTTP/1.1
Tus-Resumable: 1.0.0
Content-Type: application/offset+octet-stream
Upload-Offset: 73400320
```

#### Exemplo de integracao com Spring Boot

Uma abordagem pratica em Java e usar uma implementacao pronta de servidor TUS, como a biblioteca `tus-java-server`, e expor um endpoint Spring MVC que delega o protocolo para ela.

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

Configuracao conceitual do servico:

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

Esse exemplo e ilustrativo. Dependendo da biblioteca e da versao usadas, a API exata pode variar. A ideia principal e:

- o endpoint Spring recebe as requisicoes do protocolo;
- o servico TUS cuida de `POST`, `HEAD`, `PATCH` e possivelmente `DELETE`;
- os arquivos temporarios ou finais ficam em um storage configurado.

#### Cliente web

No frontend, o uso de TUS normalmente aparece com bibliotecas JavaScript compativeis, como Uppy ou `tus-js-client`. O navegador passa a:

- iniciar o upload;
- acompanhar progresso;
- retomar automaticamente apos interrupcao;
- persistir estado local para continuidade.

#### Cuidados de arquitetura

- validar autenticacao e autorizacao antes de aceitar o upload;
- definir limite maximo de arquivo;
- controlar expiracao de uploads incompletos;
- persistir metadados do upload separadamente quando necessario;
- decidir quando o arquivo deixa de ser temporario e passa a ser definitivo;
- integrar com antivirus ou validacao posterior em fluxos sensiveis.

#### TUS com storage externo

O TUS resolve o protocolo de upload resumivel, mas o backend de armazenamento ainda precisa ser definido. Em sistemas maiores, uma combinacao comum e:

- TUS para transporte resumivel;
- storage em disco temporario ou objeto;
- processo posterior para consolidar metadados e mover o arquivo ao destino final.
### 9.12. Download simples com `Resource`

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

### 9.13. Download em streaming

Quando o arquivo pode ser grande, o ideal e evitar carregar tudo em memoria.

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
- menor pressao sobre heap da aplicacao.

### 9.14. Armazenamento em banco ou armazenamento externo

Em geral, existem tres estrategias comuns:

- disco local;
- banco de dados;
- storage externo, como S3, MinIO ou equivalente.

Regra pratica:

- disco local: simples, mas ruim para multiplas instancias sem volume compartilhado;
- banco: pode funcionar para arquivos pequenos, mas costuma nao ser ideal para volume alto;
- storage externo: melhor para escalabilidade, CDN, compartilhamento e alta disponibilidade.

Uma estrategia muito comum e:

- salvar o binario em storage externo;
- persistir no banco apenas os metadados e o identificador do arquivo.

### 9.15. Cuidados de seguranca

Em upload e download, alguns cuidados sao essenciais:

- validar tipo, extensao e tamanho;
- evitar usar diretamente o nome enviado pelo usuario como nome fisico;
- impedir path traversal;
- restringir quem pode baixar ou subir arquivos;
- escanear arquivos em cenarios sensiveis;
- nunca confiar apenas no `contentType` informado pelo cliente;
- para downloads privados, validar autorizacao antes de servir o arquivo.

Se a aplicacao usa Spring Security, o endpoint de download deve validar se o usuario tem acesso ao recurso solicitado.

### 9.16. Tratamento de erros

Erros comuns:

- arquivo acima do limite;
- multipart malformado;
- arquivo inexistente no download;
- permissao insuficiente;
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

### 9.17. Boas praticas

- defina limites de upload no `application.yml`;
- valide tamanho, extensao e conteudo;
- prefira nomes gerados internamente;
- use streaming para downloads grandes;
- mantenha metadados e binario separados quando fizer sentido;
- nao salve arquivos sensiveis sem controle de autorizacao;
- monitore uso de disco, temporarios e falhas de upload;
- em aplicacoes distribuidas, prefira storage compartilhado ou externo.

## 10. Sugestao de organizacao do projeto

```text
src/main/java/br/com/exemplo/
  mail/
    api/
    config/
    template/
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
```

## 11. Resumo

Os topicos avancados deste documento seguem a mesma ideia de arquitetura:

- usar a configuracao padrao do Spring Boot quando o caso for simples;
- adicionar uma camada administrativa quando for preciso parametrizar comportamento em banco;
- manter a regra de negocio desacoplada da infraestrutura;
- expor operacoes manuais apenas quando houver necessidade operacional clara.

Esse desenho deixa a aplicacao mais flexivel em producao, reduz a necessidade de deploy para ajustes operacionais e facilita a evolucao para cenarios multi-tenant, clusterizados ou orientados a eventos.

## 12. Referencias

As referencias abaixo foram usadas como base oficial ou complementar relevante para a elaboracao deste documento:

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

### Upload resumivel

- TUS Protocol: https://tus.io/protocols/resumable-upload
- `tus-java-server`: https://github.com/tomdesair/tus-java-server
- `tusd`: https://github.com/tus/tusd

### Resiliencia e rate limiting

- Bucket4j: https://bucket4j.com/
- Resilience4j: https://resilience4j.readme.io/
- Spring Retry: https://github.com/spring-projects/spring-retry

### Cache

- Caffeine: https://github.com/ben-manes/caffeine

### CSV e Excel

- Apache Commons CSV: https://commons.apache.org/proper/commons-csv/
- Apache POI Component Overview: https://poi.apache.org/components/
- Apache POI Quick Guide: https://poi.apache.org/components/spreadsheet/quick-guide.html
- Apache POI `SXSSFWorkbook`: https://poi.apache.org/apidocs/5.0/org/apache/poi/xssf/streaming/SXSSFWorkbook.html
- `pjfanning/excel-streaming-reader`: https://github.com/pjfanning/excel-streaming-reader
