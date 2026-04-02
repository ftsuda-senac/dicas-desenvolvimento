# Spring Boot Avançado

Este documento reúne informações e exemplos práticos sobre recursos avançados usados com frequência em aplicações Spring Boot:

- envio de e-mails com configuração padrão, configuração dinâmica de servidores SMTP e templates Thymeleaf;
- execução de jobs com Quartz, incluindo persistencia em banco, controle dinâmico e disparo manual;
- uso de JMS com Artemis, cobrindo filas, pub-sub, filas duráveis, uso de `pooled-jms` e mensagens com resposta.
- uso de Spring Events para comunicação interna desacoplada, com listeners síncronos, assíncronos e transacionais;
- mecanismos de resiliência, incluindo timeout, retry, circuit breaker, bulkhead, rate limiter e fallback;
- rate limiting com Bucket4j para controle de acesso por IP, usuário, tenant ou chave de negocio.
- processamento em lote com Spring Batch, incluindo `Job`, `Step`, chunk processing, restart, skip e retry.
- cache e Spring Session, incluindo `@Cacheable`, `@CachePut`, `@CacheEvict`, Caffeine, Redis e sessão distribuída.
- upload e download de arquivos, incluindo `MultipartFile`, streaming, armazenamento local, validações de segurança e detecção de tipo real com Apache Tika.
- leitura e exportação de dados em CSV e Excel com Apache Commons CSV, Apache POI e `excel-streaming-reader`.

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

## 4. Events do Spring

Os Events do Spring são um mecanismo simples para comunicação interna entre componentes da mesma aplicação. Eles ajudam a reduzir acoplamento entre services e são muito uteis quando uma ação de negócio precisa disparar reações adicionais, como auditoria, notificação ou integrações internas.

Casos comuns:

- após concluir um cadastro;
- após confirmar um pagamento;
- para auditoria interna;
- para disparar tarefas complementares sem acoplar tudo no mesmo service.

### 4.1. Quando usar Spring Events

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

### 4.2. Eventos padrão do Spring mais usados

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

### 4.5. Eventos síncronos

Por padrão, o processamento de eventos do Spring é síncrono. Isso significa:

- o listener roda na mesma thread do publisher;
- se um listener falhar, a exceção pode afetar o fluxo principal;
- o tempo do listener impacta diretamente o tempo da operação original.

Esse comportamento é adequado para:

- regras internas pequenas;
- auditoria simples;
- validações complementares;
- fluxos que precisam participar da mesma transação logica.

### 4.6. Eventos assíncronos com `@Async`

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

### 4.7. Eventos transacionais com `@TransactionalEventListener`

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

### 4.8. Ordenação e condições

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

### 4.9. Event listeners retornando novos eventos

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

### 4.10. Comparando Spring Events e JMS

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

### 4.11. Boas práticas com Spring Events

- mantenha os eventos pequenos e focados no fato de negócio;
- não coloque regra complexa diretamente no listener;
- prefira nomes orientados a acontecimentos, como `PedidoFaturadoEvent`;
- para integrações dependentes de commit, use `@TransactionalEventListener`;
- para carga alta ou necessidade de resiliência, considere mensageria externa;
- documente quais eventos são internos e quais disparam integrações.

### 4.12. Eventos padrão relacionados ao Spring Security

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

## 5. Mecanismos de resiliência

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

### 5.1. Quando aplicar resiliência

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

### 5.2. Dependências mais comuns

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

### 5.3. Timeout

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

### 5.4. Retry com Resilience4j

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

### 5.5. Retry com Spring Retry

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

### 5.6. Circuit breaker

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

### 5.7. Bulkhead

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

### 5.9. Rate limiter com Bucket4j

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

### 5.10. Fallback

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

### 5.11. Combinando mecanismos

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

### 5.12. Resiliencia em consumidores JMS e jobs

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

### 5.13. Observabilidade e monitoramento

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

### 5.14. Boas práticas

- aplique resiliência apenas onde existe dependência instável ou remota;
- diferencie erro transiente de erro funcional;
- prefira operações idempotentes quando houver retry;
- use timeout curto o suficiente para proteger a aplicação;
- não use fallback enganoso em fluxos críticos;
- monitore tudo o que puder degradar silenciosamente;
- revise periodicamente os parâmetros de retry, timeout e circuit breaker.


## 6. Cache e Spring Session

Cache e gerenciamento de sessão são dois temas muito relevantes em aplicações Spring Boot tradicionais. Embora resolvam problemas diferentes, eles frequentemente usam tecnologias parecidas, como Redis:

- cache melhora desempenho e reduz chamadas repetidas;
- sessão distribuída permite que o estado do usuário sobreviva entre instancias da aplicação.

Em geral:

- use cache para dados repetidos e relativamente estáveis;
- use Spring Session para estado de usuário em aplicações web stateful;
- evite misturar os dois conceitos no desenho de chaves e TTLs.

### 8.1. Dependências

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

Com `@EnableCaching`, o Spring passa a interceptar os métodos anotados com `@Cacheable`, `@CachePut` e `@CacheEvict`.

### 6.3. Uso de `@Cacheable`

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

### 6.4. Uso de `@CachePut`

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

### 6.5. Uso de `@CacheEvict`

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

### 6.6. Configuração com Caffeine

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

### 6.7. Configuração com Redis para cache

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

### 6.8. Estratégias de invalidação

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

### 6.9. Cache em aplicações Spring Boot tradicionais

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

### 6.10. Spring Session e sessão distribuída

Spring Session abstrai o armazenamento da sessão HTTP e permite mover esse estado para um backend compartilhado, como Redis ou JDBC.

Isso é especialmente útil quando:

- há mais de uma instancia da aplicação;
- o load balancer não usa sticky session;
- o usuário precisa manter a sessão ao trocar de instancia;
- o estado da autenticação precisa sobreviver em ambiente distribuído.

Segundo a documentação oficial do Spring Boot, em aplicações servlet o store de sessão pode ser autoconfigurado com Redis ou JDBC, sem necessidade de `@Enable*HttpSession` quando há um único modulo Spring Session no classpath.

### 6.11. Configurando Spring Session com Redis

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

### 6.12. Persistência em Redis e integração com Spring Security

Em aplicações web com login tradicional, o Spring Session integra-se muito bem com Spring Security. Quando o `SecurityContext` e salvo na sessão, ele também passa a ser persistido no Redis.

Isso permite:

- manter autenticação entre instancias;
- invalidar sessões de forma centralizada;
- acompanhar sessões ativas de forma mais controlada.

Segundo a documentação do Spring Session, um cookie `SESSION` e usado para apontar para a sessão persistida. O backend Redis armazena os atributos da sessão e cuida da expiração conforme a configuração.

### 6.13. Estratégias para aplicações web tradicionais

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

### 6.14. Encontrando e invalidando sessões por usuário

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

### 6.15. Eventos de sessão

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

### 6.16. Boas práticas com cache e Spring Session

- escolha Caffeine para cache local e Redis para cache ou sessão compartilhada;
- não use cache como substituto de consistência de negócio;
- defina TTL e estratégia de invalidação antes de ativar o cache;
- monitore hit ratio, latência e volume de chaves;
- mantenha sessões pequenas e controladas;
- em aplicações distribuídas, prefira Spring Session com backend compartilhado;
- teste invalidação de cache e expiração de sessão em cenários reais;
- evite salvar objetos não serializáveis ou muito grandes na sessão.

## 7. Spring Batch

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

### 7.1. Dependências

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

Se o projeto vai usar persistencia real dos metadados do Batch, normalmente também havera um banco com `spring-boot-starter-data-jdbc` ou `spring-boot-starter-data-jpa`, dependendo da arquitetura adotada.

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

### 7.3. Configuração basica de Job e Step

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

### 7.5. Leitura de arquivo CSV com Spring Batch

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
- não houver processamento item a item;
- a logica for unica, como mover arquivo, limpar diretorio ou gerar marcador de execução.

### 7.10. Boas praticas com Spring Batch

- persista metadados do Batch em banco relacional;
- mantenha jobs idempotentes sempre que possivel;
- use `JobParameters` conscientemente para distinguir instancias;
- prefira chunk para alto volume e `Tasklet` para tarefas pontuais;
- trate skip e retry com regras de negocio claras;
- registre metricas e auditoria das execucoes;
- para jobs pesados, considere disparo por scheduler ou operacao administrativa;
- para arquivos grandes, combine Spring Batch com leitores aprópriados, como CSV ou leitura streaming de XLSX.

### 7.11. Quando Spring Batch faz mais sentido

Spring Batch costuma ser uma excelente escolha quando:

- há volume relevante de dados;
- o processamento precisa de restart controlado;
- existe necessidade de trilha de execução;
- o fluxo precisa de chunk, skip, retry e gerenciamento de falha.

Em contrapartida, talvez ele seja excessivo quando:

- a tarefa é muito simples e eventual;
- um service comum ou job Quartz resolve com menos complexidade;
- não há estado de execução, checkpoint ou necessidade de restart.

## 8. Exportação de dados em CSV e Excel

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

### 6.1. Dependências

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

### 8.2. Quando usar CSV e quando usar Excel

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

### 8.3. Exemplo de DTO para exportação

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

- o delimitador pode ser `,` ou `;`, dependendo do padrão esperado;
- em cenarios brasileiros, `;` e comum por compatibilidade com Excel em algumas configuracoes regionais;
- usar UTF-8 evita problemas com acentos;
- `CSVPrinter` ajuda a lidar corretamente com escape e aspas.

### 8.5. Lendo arquivos CSV com Apache Commons CSV

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

Esse controller mostra os dois formatos no mesmo endpoint de exportação.

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

### 8.9. Diferencas entre `XSSFWorkbook` e `SXSSFWorkbook`

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

### 8.10. Exemplo de exportação com `SXSSFWorkbook`

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

### 8.11. Leitura de arquivos grandes com `excel-streaming-reader`

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

### 8.12. Comparando abordagens de leitura para XLSX

Resumo prática:

- `WorkbookFactory` ou `XSSFWorkbook`: melhor para arquivos pequenos e medios, com API completa;
- `excel-streaming-reader`: melhor para leitura de arquivos grandes com menor consumo de memória;
- `SXSSFWorkbook`: não e biblioteca de leitura; ele e voltado para escrita streaming.

Escolha rápida:

- importar planilha pequena ou média: `XSSFWorkbook` ou `WorkbookFactory`;
- importar planilha muito grande: `excel-streaming-reader`;
- gerar planilha muito grande: `SXSSFWorkbook`.

### 8.13. Exportação com mais de uma aba

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

### 8.14. Cuidados com memória e volume

Arquivos CSV geralmente são leves, mas planilhas Excel podem consumir bastante memória, principalmente com muitas linhas e estilos.

Boas praticas:

- para grandes volumes, preferir CSV quando possível;
- limitar quantidade de estilos diferentes no Excel;
- evitar manter muitos arquivos grandes em memória ao mesmo tempo;
- considerar exportação assíncrona para arquivos muito grandes;
- para escrita massiva em `.xlsx`, avaliar `SXSSFWorkbook`;
- lembrar que `SXSSFWorkbook` cria arquivos temporários e exige limpeza adequada.

### 8.15. Diferencas práticas entre CSV e Excel

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

### 8.16. Boas práticas para exportação

- usar nomes de arquivo claros, por exemplo `pedidos-2026-03-19.xlsx`;
- aplicar `Content-Disposition: attachment`;
- definir o `Content-Type` correto;
- evitar expor colunas sensíveis sem necessidade;
- considerar paginação, filtros e exportação assíncrona para grandes volumes;
- registrar auditoria quando a exportação envolver dados sensíveis.

## 9. Upload e download de arquivos

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

### 9.1. Suporte básico a multipart no Spring Boot

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

### 9.4. Validação de tipo e tamanho

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

### 9.5. Validação de conteúdo com Apache Tika

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

### 9.6. Upload com metadados

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

### 9.7. Upload de JSON e arquivo com `@RequestPart`

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

### 9.8. Fetch API para `/upload/v1` com `@RequestPart`

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

### 9.9. Upload de JSON e arquivo com `@ModelAttribute`

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

### 9.10. Fetch API para `/upload/v2` com `@ModelAttribute`

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

### 9.11. Comparando `@ModelAttribute` e `@RequestPart`

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

### 9.12. Upload resumível com TUS

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
### 9.13. Download simples com `Resource`

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

### 9.14. Download em streaming

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

### 9.15. Armazenamento em banco ou armazenamento externo

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

### 9.16. Cuidados de segurança

Em upload e download, alguns cuidados são essenciais:

- validar tipo, extensão e tamanho;
- evitar usar diretamente o nome enviado pelo usuário como nome físico;
- impedir path traversal;
- restringir quem pode baixar ou subir arquivos;
- escanear arquivos em cenários sensíveis;
- nunca confiar apenas no `contentType` informado pelo cliente;
- para downloads privados, validar autorização antes de servir o arquivo.

Se a aplicação usa Spring Security, o endpoint de download deve validar se o usuário tem acesso ao recurso solicitado.

### 9.17. Tratamento de erros

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

### 9.18. Boas práticas

- defina limites de upload no `application.yml`;
- valide tamanho, extensão e conteúdo;
- prefira nomes gerados internamente;
- use streaming para downloads grandes;
- mantenha metadados e binário separados quando fizer sentido;
- não salve arquivos sensíveis sem controle de autorização;
- monitore uso de disco, temporários e falhas de upload;
- em aplicações distribuídas, prefira storage compartilhado ou externo.

## 10. Sugestão de organização do projeto

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

Os tópicos avançados deste documento seguem a mesma ideia de arquitetura:

- usar a configuração padrão do Spring Boot quando o caso for simples;
- adicionar uma camada administrativa quando for preciso parametrizar comportamento em banco;
- manter a regra de negócio desacoplada da infraestrutura;
- expor operações manuais apenas quando houver necessidade operacional clara.

Esse desenho deixa a aplicação mais flexível em produção, reduz a necessidade de deploy para ajustes operacionais e facilita a evolução para cenários multi-tenant, clusterizados ou orientados a eventos.

## 12. Referencias

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
