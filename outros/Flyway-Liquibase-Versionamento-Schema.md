# Flyway e Liquibase — Versionamento de Banco de Dados com Spring Boot

> **Objetivo:** Apresentar em profundidade as ferramentas Flyway e Liquibase para versionamento e migração de schemas de banco de dados, com exemplos práticos completos em Spring Boot 3.x, cobrindo desde a configuração básica até cenários avançados de produção.

> **Nota:** O documento [Topicos-Complementares-Desenvolvimento.md](Topicos-Complementares-Desenvolvimento.md) introduz Flyway e Liquibase brevemente. Aqui o foco é aprofundar cada ferramenta, explorar cenários reais e fornecer tutoriais completos.

---

## Sumário

1. [Por que Versionar o Banco de Dados](#1-por-que-versionar-o-banco-de-dados)
2. [Conceitos Fundamentais de Migrations](#2-conceitos-fundamentais-de-migrations)
   - [2.4 Por que Não Usar `ddl-auto=update` em Produção](#24-por-que-não-usar-ddl-autoupdate-em-produção)
3. [Flyway com Spring Boot](#3-flyway-com-spring-boot)
   - [3.1 Configuração Inicial](#31-configuração-inicial)
   - [3.2 Convenção de Nomes](#32-convenção-de-nomes)
   - [3.3 Ciclo de Vida de uma Migration](#33-ciclo-de-vida-de-uma-migration)
   - [3.4 Tabela de Controle — flyway_schema_history](#34-tabela-de-controle--flyway_schema_history)
   - [3.5 Migrations Versionadas e Repetíveis](#35-migrations-versionadas-e-repetíveis)
   - [3.6 Callbacks do Flyway](#36-callbacks-do-flyway)
   - [3.7 Migrations em Java](#37-migrations-em-java)
   - [3.8 Profiles e Ambientes Múltiplos](#38-profiles-e-ambientes-múltiplos)
   - [3.9 Comandos do Flyway](#39-comandos-do-flyway)
   - [3.10 Plugin Maven e Gradle](#310-plugin-maven-e-gradle)
4. [Liquibase com Spring Boot](#4-liquibase-com-spring-boot)
   - [4.1 Configuração Inicial](#41-configuração-inicial)
   - [4.2 Formatos de Changelog — XML, YAML, JSON e SQL](#42-formatos-de-changelog--xml-yaml-json-e-sql)
   - [4.3 Estrutura de Changelogs](#43-estrutura-de-changelogs)
   - [4.4 Tabela de Controle — DATABASECHANGELOG](#44-tabela-de-controle--databasechangelog)
   - [4.5 Preconditions — Execução Condicional](#45-preconditions--execução-condicional)
   - [4.6 Rollback Nativo](#46-rollback-nativo)
   - [4.7 Contexts e Labels](#47-contexts-e-labels)
   - [4.8 Comandos do Liquibase](#48-comandos-do-liquibase)
   - [4.9 Diff e Geração Automática de Changelogs](#49-diff-e-geração-automática-de-changelogs)
   - [4.10 Plugin Maven e Gradle](#410-plugin-maven-e-gradle)
5. [Tutorial Prático — Sistema de E-commerce com Flyway](#5-tutorial-prático--sistema-de-e-commerce-com-flyway)
6. [Tutorial Prático — Sistema de E-commerce com Liquibase](#6-tutorial-prático--sistema-de-e-commerce-com-liquibase)
7. [Flyway vs Liquibase — Comparativo Detalhado](#7-flyway-vs-liquibase--comparativo-detalhado)
8. [Testcontainers e Migrations](#8-testcontainers-e-migrations)
9. [Boas Práticas para Migrations em Produção](#9-boas-práticas-para-migrations-em-produção)
   - [9.5 Zero-Downtime Migrations](#95-zero-downtime-migrations)
   - [9.6 Data Migrations vs Schema Migrations](#96-data-migrations-vs-schema-migrations)
   - [9.7 Dados Sensíveis em Migrations](#97-dados-sensíveis-em-migrations)
   - [9.8 Anonimização de Dados entre Ambientes](#98-anonimização-de-dados-entre-ambientes)
   - [9.9 Feature Toggles e Migrations](#99-feature-toggles-e-migrations)
10. [Integração com CI/CD](#10-integração-com-cicd)
    - [10.3 Monitoramento de Migrations](#103-monitoramento-de-migrations)
11. [Cenários Avançados](#11-cenários-avançados)
    - [11.5 Gerando Migrations a partir de Entities JPA](#115-gerando-migrations-a-partir-de-entities-jpa)
    - [11.6 Migrations em Microsserviços](#116-migrations-em-microsserviços)
    - [11.7 Migrations em Bancos Distribuídos e Réplicas](#117-migrations-em-bancos-distribuídos-e-réplicas)
    - [11.8 Migrations em Bancos NoSQL — Mongock](#118-migrations-em-bancos-nosql--mongock)
    - [11.9 Schema Registry e Evolução de Contratos](#119-schema-registry-e-evolução-de-contratos)
12. [Erros Comuns e Troubleshooting](#12-erros-comuns-e-troubleshooting)

---

## 1. Por que Versionar o Banco de Dados

Em projetos de software, o código da aplicação é versionado com Git — cada alteração tem histórico, é rastreável e pode ser reproduzida. Mas e o banco de dados? Sem uma estratégia de versionamento, o schema evolui de forma desordenada:

```
┌──────────────────────────────────────────────────────────────────────┐
│                     SEM VERSIONAMENTO                                │
│                                                                      │
│  Dev 1: "Roda esse ALTER TABLE no banco de staging"                  │
│  Dev 2: "Qual script? Eu já alterei essa tabela ontem"               │
│  DBA:   "Produção está diferente de staging, quem mexeu?"            │
│  Dev 1: "Funciona na minha máquina..."                               │
│                                                                      │
│  Resultado: ambientes inconsistentes, bugs difíceis de rastrear,     │
│  deploys arriscados, rollback impossível                             │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                     COM VERSIONAMENTO                                │
│                                                                      │
│  V1__criar_tabela_usuario.sql     → Aplicada em todos os ambientes   │
│  V2__criar_tabela_pedido.sql      → Versionada no Git junto ao código│
│  V3__adicionar_coluna_telefone.sql → Executada automaticamente       │
│                                                                      │
│  Resultado: ambientes idênticos, histórico completo,                 │
│  deploys previsíveis, rollback controlado                            │
└──────────────────────────────────────────────────────────────────────┘
```

### Benefícios do versionamento de schema

| Benefício | Descrição |
|-----------|-----------|
| **Reprodutibilidade** | Qualquer desenvolvedor recria o banco do zero com um comando |
| **Rastreabilidade** | Cada alteração tem autor, data e está no histórico do Git |
| **Consistência** | Dev, staging e produção com o mesmo schema |
| **Automação** | Migrations executadas automaticamente no deploy |
| **Colaboração** | Conflitos de schema detectados no merge, não em produção |
| **Auditoria** | Histórico completo de quando cada alteração foi aplicada |

---

## 2. Conceitos Fundamentais de Migrations

### O que é uma migration

Uma migration é um script que descreve uma alteração incremental no schema do banco de dados. Cada migration é aplicada uma única vez, na ordem correta, e o sistema mantém registro de quais migrations já foram executadas.

```
Linha do tempo do schema:

V1 (Criação)    V2 (Evolução)    V3 (Evolução)    V4 (Dados)
     │                │                │                │
     ▼                ▼                ▼                ▼
┌─────────┐    ┌─────────────┐   ┌────────────┐  ┌──────────┐
│ CREATE   │    │ CREATE      │   │ ALTER TABLE│  │ INSERT   │
│ TABLE    │ →  │ TABLE       │ → │ ADD COLUMN │→ │ INTO     │
│ usuario  │    │ pedido      │   │ telefone   │  │ perfil   │
└─────────┘    └─────────────┘   └────────────┘  └──────────┘

Schema atual = V1 + V2 + V3 + V4 (soma de todas as migrations)
```

### Tipos de alteração comuns

| Tipo | Exemplos |
|------|----------|
| **Estrutural** | `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE` |
| **Índices** | `CREATE INDEX`, `DROP INDEX` |
| **Constraints** | `ADD CONSTRAINT`, `ADD FOREIGN KEY` |
| **Dados (seed)** | `INSERT INTO`, `UPDATE` para dados de referência |
| **Views/Functions** | `CREATE VIEW`, `CREATE FUNCTION` |

### Imutabilidade

Uma migration já aplicada **nunca deve ser alterada**. Se foi aplicada em qualquer ambiente, ela é imutável. Correções devem ser feitas em uma nova migration:

```
❌ ERRADO: Editar V3__adicionar_coluna_telefone.sql após já ter sido aplicada

✅ CORRETO: Criar V4__corrigir_tipo_coluna_telefone.sql com a correção
```

### 2.4 Por que Não Usar `ddl-auto=update` em Produção

O Hibernate oferece a propriedade `spring.jpa.hibernate.ddl-auto` que pode gerar e atualizar o schema automaticamente a partir das entities JPA. Embora conveniente para desenvolvimento, **não é adequado para produção**:

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update   # ⚠️ NÃO RECOMENDADO para produção
```

#### O que cada valor faz

| Valor | Comportamento | Risco |
|-------|---------------|-------|
| `none` | Não faz nada no schema | Nenhum |
| `validate` | Apenas valida se as entities batem com o schema | Nenhum (recomendado com migrations) |
| `update` | Tenta aplicar diferenças incrementalmente | **Alto** |
| `create` | Dropa tudo e recria do zero | **Perda total de dados** |
| `create-drop` | Cria no startup, dropa no shutdown | **Perda total de dados** |

#### Limitações do `ddl-auto=update`

O `update` parece atraente, mas tem falhas significativas:

```
┌──────────────────────────────────────────────────────────────────────┐
│              O QUE ddl-auto=update NÃO FAZ                           │
│                                                                      │
│  ✗ NÃO remove colunas (renomear = criar nova + coluna antiga fica)   │
│  ✗ NÃO remove tabelas que não são mais entities                      │
│  ✗ NÃO renomeia colunas ou tabelas                                   │
│  ✗ NÃO altera tipo de coluna existente (VARCHAR(100) → VARCHAR(200)) │
│  ✗ NÃO remove índices ou constraints obsoletos                       │
│  ✗ NÃO migra dados entre colunas/tabelas                             │
│  ✗ NÃO garante ordem de execução                                     │
│  ✗ NÃO mantém histórico de alterações                                │
│  ✗ NÃO é reproduzível (cada ambiente pode divergir)                  │
│  ✗ NÃO permite rollback                                              │
└──────────────────────────────────────────────────────────────────────┘
```

#### Cenário real de problema

```java
// ANTES: entity com campo "nome"
@Entity
public class Cliente {
    @Column(length = 100)
    private String nome;
}

// DEPOIS: renomeamos para "nomeCompleto"
@Entity
public class Cliente {
    @Column(name = "nome_completo", length = 100)
    private String nomeCompleto;
}
```

```
Com ddl-auto=update:
┌─────────────────────────┐
│      tabela: cliente     │
│─────────────────────────│
│ id          BIGINT       │
│ nome        VARCHAR(100) │  ← coluna antiga permanece (com dados!)
│ nome_completo VARCHAR(100)│  ← coluna nova criada (vazia!)
└─────────────────────────┘
Resultado: dados perdidos na coluna antiga, coluna suja no banco

Com Flyway/Liquibase:
-- V5__renomear_nome_para_nome_completo.sql
ALTER TABLE cliente RENAME COLUMN nome TO nome_completo;
Resultado: dados preservados, schema limpo
```

#### Quando usar cada opção

| Cenário | `ddl-auto` recomendado | Ferramenta de migrations |
|---------|:----------------------:|:------------------------:|
| Protótipo rápido / estudo | `create` ou `update` | Não precisa |
| Testes unitários com H2 | `create-drop` | Opcional |
| Desenvolvimento local | `validate` | Flyway ou Liquibase |
| Staging / Homologação | `validate` | Flyway ou Liquibase |
| **Produção** | **`validate`** | **Flyway ou Liquibase** |

> **Regra prática:** Se o projeto vai além de um protótipo descartável, use migrations desde o início. O custo de adotar Flyway/Liquibase é mínimo e evita problemas sérios no futuro.

---

## 3. Flyway com Spring Boot

Flyway é a ferramenta de migrations mais popular no ecossistema Spring. Usa SQL puro como formato principal, é simples de configurar e tem integração nativa com Spring Boot.

### 3.1 Configuração Inicial

#### Dependências Maven

```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Flyway Core -->
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>

    <!-- Módulo específico para o banco (obrigatório desde Flyway 10) -->
    <!-- Para PostgreSQL: -->
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-database-postgresql</artifactId>
    </dependency>

    <!-- Para MySQL/MariaDB: -->
    <!--
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-mysql</artifactId>
    </dependency>
    -->

    <!-- Driver do banco -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### Dependências Gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.flywaydb:flyway-core'
    implementation 'org.flywaydb:flyway-database-postgresql'
    runtimeOnly 'org.postgresql:postgresql'
}
```

#### application.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/minha_app
    username: postgres
    password: postgres

  jpa:
    hibernate:
      ddl-auto: validate  # Flyway gerencia o schema, Hibernate apenas valida
    show-sql: true

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true  # Útil ao adicionar Flyway em projeto existente
```

> **Importante:** Com Flyway, use `ddl-auto: validate` (ou `none`). Nunca use `create`, `create-drop` ou `update` — isso conflita com o versionamento do Flyway.

### 3.2 Convenção de Nomes

O Flyway identifica e ordena migrations pelo nome do arquivo:

```
src/main/resources/db/migration/
├── V1__criar_tabela_usuario.sql
├── V1.1__adicionar_constraint_email.sql
├── V2__criar_tabela_produto.sql
├── V3__criar_tabela_pedido.sql
├── V4__inserir_categorias_padrao.sql
├── R__criar_view_relatorio_vendas.sql
└── R__atualizar_procedures.sql
```

#### Formato detalhado

```
V 2.1 __ descricao_com_underscores .sql
│  │      │                          │
│  │      │                          └─ Extensão
│  │      └─ Descrição (underscores viram espaços no histórico)
│  └─ Versão (inteiro, decimal ou pontilhada: 1, 1.1, 1.2.3)
└─ Prefixo: V = versionada, R = repetível
```

| Prefixo | Tipo | Execução | Uso |
|---------|------|----------|-----|
| `V` | Versionada | Uma vez, na ordem da versão | DDL, DML (dados de seed) |
| `R` | Repetível | Sempre que o conteúdo (checksum) mudar | Views, procedures, funções |

#### Zero-padding — ordenação no filesystem

O Flyway ignora zeros à esquerda na versão (`V001` é tratado como `1`), então a ordem de execução é sempre numérica. A vantagem de usar zero-padding é que o filesystem e as IDEs listam arquivos em ordem lexicográfica — sem padding, `V10` aparece antes de `V2`:

```
Sem zero-padding (filesystem):     Com zero-padding (filesystem):
├── V1__criar_usuario.sql          ├── V001__criar_usuario.sql
├── V10__criar_cupom.sql  ← !!    ├── V002__criar_produto.sql
├── V11__criar_envio.sql           ├── V003__criar_pedido.sql
├── V2__criar_produto.sql          ├── ...
├── V3__criar_pedido.sql           ├── V010__criar_cupom.sql
├── ...                            ├── V011__criar_envio.sql
├── V9__adicionar_cpf.sql          └── V099__...
```

> **Dica:** Escolha o número de dígitos conforme o porte do projeto (3 dígitos para a maioria, 4 para projetos grandes) e mantenha o padding consistente. Se ultrapassar o limite (ex: `V999`), basta usar mais dígitos nas próximas (`V1000__...`) — o Flyway compara numericamente de qualquer forma.

#### Regras importantes

- Dois underscores (`__`) entre versão e descrição
- Versões devem ser únicas
- Migrations versionadas executam antes das repetíveis
- Sem espaços ou caracteres especiais na descrição (use underscores)

### 3.3 Ciclo de Vida de uma Migration

```
Startup da aplicação Spring Boot
         │
         ▼
  Flyway verifica conexão com o banco
         │
         ▼
  Existe tabela flyway_schema_history?
    │              │
   Sim            Não
    │              │
    ▼              ▼
  Lê versão    Cria a tabela
  atual        (ou faz baseline)
    │              │
    └──────┬───────┘
           ▼
  Escaneia classpath:db/migration
           │
           ▼
  Compara arquivos com histórico
           │
           ▼
  ┌─ Migrations pendentes? ─┐
  │                          │
 Sim                        Não
  │                          │
  ▼                          ▼
Executa em ordem          Pronto!
(dentro de transação*)    App inicia
  │
  ▼
Registra no histórico
  │
  ▼
App inicia
```

*Cada migration SQL roda em sua própria transação (quando o banco suporta DDL transacional, como PostgreSQL).

### 3.4 Tabela de Controle — flyway_schema_history

O Flyway cria e mantém automaticamente a tabela `flyway_schema_history`:

```sql
SELECT installed_rank, version, description, type, script, checksum,
       installed_by, installed_on, execution_time, success
FROM flyway_schema_history;
```

| installed_rank | version | description | type | script | checksum | success |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | 1 | criar tabela usuario | SQL | V1__criar_tabela_usuario.sql | -12345678 | true |
| 2 | 2 | criar tabela pedido | SQL | V2__criar_tabela_pedido.sql | 87654321 | true |
| 3 | 3 | adicionar coluna telefone | SQL | V3__adicionar_coluna_telefone.sql | 11223344 | true |

O `checksum` garante que uma migration aplicada não foi modificada. Se alguém alterar o conteúdo de `V1`, o Flyway detecta e lança erro na próxima execução.

### 3.5 Migrations Versionadas e Repetíveis

#### Migration versionada (V)

```sql
-- V1__criar_tabela_usuario.sql
CREATE TABLE usuario (
    id         BIGSERIAL    PRIMARY KEY,
    nome       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL UNIQUE,
    senha_hash VARCHAR(255) NOT NULL,
    ativo      BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_usuario_email ON usuario(email);
```

```sql
-- V2__criar_tabela_pedido.sql
CREATE TABLE pedido (
    id            BIGSERIAL      PRIMARY KEY,
    usuario_id    BIGINT         NOT NULL REFERENCES usuario(id),
    status        VARCHAR(20)    NOT NULL DEFAULT 'CRIADO',
    total         DECIMAL(12, 2) NOT NULL DEFAULT 0,
    criado_em     TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP
);

CREATE INDEX idx_pedido_usuario ON pedido(usuario_id);
CREATE INDEX idx_pedido_status  ON pedido(status);
```

```sql
-- V3__criar_tabela_item_pedido.sql
CREATE TABLE item_pedido (
    id         BIGSERIAL      PRIMARY KEY,
    pedido_id  BIGINT         NOT NULL REFERENCES pedido(id) ON DELETE CASCADE,
    produto    VARCHAR(200)   NOT NULL,
    quantidade INT            NOT NULL CHECK (quantidade > 0),
    preco      DECIMAL(10, 2) NOT NULL CHECK (preco >= 0)
);

CREATE INDEX idx_item_pedido_pedido ON item_pedido(pedido_id);
```

```sql
-- V4__adicionar_coluna_telefone_usuario.sql
ALTER TABLE usuario ADD COLUMN telefone VARCHAR(20);
```

```sql
-- V5__inserir_perfis_padrao.sql
CREATE TABLE perfil (
    id   BIGSERIAL   PRIMARY KEY,
    nome VARCHAR(50) NOT NULL UNIQUE
);

INSERT INTO perfil (nome) VALUES ('ADMIN');
INSERT INTO perfil (nome) VALUES ('USUARIO');
INSERT INTO perfil (nome) VALUES ('MODERADOR');

CREATE TABLE usuario_perfil (
    usuario_id BIGINT NOT NULL REFERENCES usuario(id),
    perfil_id  BIGINT NOT NULL REFERENCES perfil(id),
    PRIMARY KEY (usuario_id, perfil_id)
);
```

#### Migration repetível (R)

Ideal para objetos que são recriados inteiramente (views, procedures, functions):

```sql
-- R__view_relatorio_vendas.sql
CREATE OR REPLACE VIEW vw_relatorio_vendas AS
SELECT
    u.nome                      AS cliente,
    COUNT(p.id)                 AS total_pedidos,
    SUM(p.total)                AS valor_total,
    AVG(p.total)                AS ticket_medio,
    MAX(p.criado_em)            AS ultimo_pedido
FROM usuario u
JOIN pedido p ON p.usuario_id = u.id
WHERE p.status = 'FINALIZADO'
GROUP BY u.id, u.nome;
```

```sql
-- R__function_calcular_desconto.sql
CREATE OR REPLACE FUNCTION calcular_desconto(
    p_valor    DECIMAL,
    p_percentual DECIMAL
) RETURNS DECIMAL AS $$
BEGIN
    RETURN ROUND(p_valor * (1 - p_percentual / 100), 2);
END;
$$ LANGUAGE plpgsql;
```

### 3.6 Callbacks do Flyway

Flyway suporta callbacks — scripts executados em momentos específicos do ciclo de vida:

```
src/main/resources/db/migration/
├── V1__criar_tabela_usuario.sql
├── V2__criar_tabela_pedido.sql
└── db/callback/
    ├── beforeMigrate.sql          # Antes de iniciar migrations
    ├── afterMigrate.sql           # Após todas as migrations
    ├── beforeEachMigrate.sql      # Antes de cada migration individual
    └── afterEachMigrate.sql       # Após cada migration individual
```

```yaml
# application.yml
spring:
  flyway:
    callbacks: classpath:db/callback
```

```sql
-- db/callback/afterMigrate.sql
-- Exemplo: recompilar views materializadas após migrations
REFRESH MATERIALIZED VIEW IF EXISTS mv_dashboard_vendas;
```

| Callback | Momento |
|----------|---------|
| `beforeMigrate` | Antes de iniciar o processo de migration |
| `beforeEachMigrate` | Antes de cada migration individual |
| `afterEachMigrate` | Após cada migration individual (com sucesso) |
| `afterMigrate` | Após todas as migrations completarem |
| `beforeValidate` | Antes da validação de checksums |
| `afterValidate` | Após validação com sucesso |
| `beforeBaseline` | Antes de criar o baseline |
| `afterBaseline` | Após o baseline ser criado |

### 3.7 Migrations em Java

Para migrations complexas que envolvem lógica de negócio ou transformações de dados:

```java
package db.migration;

import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;

public class V6__migrar_nomes_para_maiusculo extends BaseJavaMigration {

    @Override
    public void migrate(Context context) throws Exception {
        try (Statement select = context.getConnection().createStatement()) {
            try (ResultSet rows = select.executeQuery("SELECT id, nome FROM usuario")) {
                try (PreparedStatement update = context.getConnection()
                        .prepareStatement("UPDATE usuario SET nome = ? WHERE id = ?")) {
                    while (rows.next()) {
                        long id = rows.getLong("id");
                        String nome = rows.getString("nome");
                        update.setString(1, nome.toUpperCase());
                        update.setLong(2, id);
                        update.addBatch();
                    }
                    update.executeBatch();
                }
            }
        }
    }
}
```

```java
package db.migration;

import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;

public class V7__criptografar_senhas_existentes extends BaseJavaMigration {

    @Override
    public void migrate(Context context) throws Exception {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

        try (Statement select = context.getConnection().createStatement();
             ResultSet rows = select.executeQuery(
                 "SELECT id, senha_hash FROM usuario WHERE senha_hash NOT LIKE '$2a$%'")) {

            try (PreparedStatement update = context.getConnection()
                    .prepareStatement("UPDATE usuario SET senha_hash = ? WHERE id = ?")) {
                while (rows.next()) {
                    update.setString(1, encoder.encode(rows.getString("senha_hash")));
                    update.setLong(2, rows.getLong("id"));
                    update.addBatch();
                }
                update.executeBatch();
            }
        }
    }
}
```

> **Atenção:** Migrations Java ficam no pacote `db.migration` por convenção. O Flyway as descobre automaticamente.

### 3.8 Profiles e Ambientes Múltiplos

```yaml
# application.yml (configuração base)
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration

---
# application-dev.yml
spring:
  flyway:
    locations: classpath:db/migration,classpath:db/devdata
    clean-disabled: false  # Permite flyway clean em dev

---
# application-prod.yml
spring:
  flyway:
    locations: classpath:db/migration
    clean-disabled: true   # NUNCA permitir clean em produção
    baseline-on-migrate: false
```

```sql
-- src/main/resources/db/devdata/V100__dados_desenvolvimento.sql
-- Dados fictícios apenas para desenvolvimento
INSERT INTO usuario (nome, email, senha_hash) VALUES
    ('Admin Dev', 'admin@dev.local', '$2a$10$hash...'),
    ('User Teste', 'teste@dev.local', '$2a$10$hash...');

INSERT INTO pedido (usuario_id, status, total) VALUES
    (1, 'FINALIZADO', 150.00),
    (1, 'CRIADO', 75.50),
    (2, 'FINALIZADO', 200.00);
```

### 3.9 Comandos do Flyway

| Comando | Descrição |
|---------|-----------|
| `migrate` | Aplica migrations pendentes |
| `clean` | Remove **todos** os objetos do schema (CUIDADO!) |
| `info` | Mostra o estado atual das migrations |
| `validate` | Valida se as migrations aplicadas correspondem aos arquivos |
| `baseline` | Cria um ponto de partida para bancos existentes |
| `repair` | Corrige a tabela de histórico (remove migrations com falha) |
| `undo` | Desfaz a última migration (versão paga — Teams/Enterprise) |

### 3.10 Plugin Maven e Gradle

#### Maven

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-maven-plugin</artifactId>
            <configuration>
                <url>jdbc:postgresql://localhost:5432/minha_app</url>
                <user>postgres</user>
                <password>postgres</password>
                <locations>
                    <location>filesystem:src/main/resources/db/migration</location>
                </locations>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>org.postgresql</groupId>
                    <artifactId>postgresql</artifactId>
                    <version>42.7.4</version>
                </dependency>
                <dependency>
                    <groupId>org.flywaydb</groupId>
                    <artifactId>flyway-database-postgresql</artifactId>
                    <version>${flyway.version}</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

```bash
# Comandos via Maven
./mvnw flyway:migrate       # Aplicar migrations pendentes
./mvnw flyway:info           # Ver estado das migrations
./mvnw flyway:validate       # Validar checksums
./mvnw flyway:repair         # Corrigir histórico
./mvnw flyway:clean          # Limpar schema (CUIDADO!)
./mvnw flyway:baseline       # Criar baseline
```

#### Gradle

```groovy
plugins {
    id 'org.flywaydb.flyway' version '10.22.0'
}

flyway {
    url = 'jdbc:postgresql://localhost:5432/minha_app'
    user = 'postgres'
    password = 'postgres'
    locations = ['classpath:db/migration']
}
```

```bash
./gradlew flywayMigrate
./gradlew flywayInfo
./gradlew flywayValidate
```

---

## 4. Liquibase com Spring Boot

Liquibase é uma ferramenta de migrations que usa changelogs declarativos, suportando XML, YAML, JSON e SQL. Sua principal vantagem sobre o Flyway é a abstração multi-banco e o suporte nativo a rollback.

### 4.1 Configuração Inicial

#### Dependências Maven

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>org.liquibase</groupId>
        <artifactId>liquibase-core</artifactId>
    </dependency>

    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### Dependências Gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.liquibase:liquibase-core'
    runtimeOnly 'org.postgresql:postgresql'
}
```

#### application.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/minha_app
    username: postgres
    password: postgres

  jpa:
    hibernate:
      ddl-auto: validate

  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.yaml
```

### 4.2 Formatos de Changelog — XML, YAML, JSON e SQL

Liquibase suporta quatro formatos. Cada um tem vantagens:

#### YAML (mais legível, recomendado para novos projetos)

```yaml
# db/changelog/changes/001-criar-tabela-usuario.yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: equipe
      comment: Criação da tabela de usuários do sistema
      changes:
        - createTable:
            tableName: usuario
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    primaryKeyName: pk_usuario
              - column:
                  name: nome
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
              - column:
                  name: email
                  type: VARCHAR(150)
                  constraints:
                    nullable: false
                    unique: true
                    uniqueConstraintName: uk_usuario_email
              - column:
                  name: senha_hash
                  type: VARCHAR(255)
                  constraints:
                    nullable: false
              - column:
                  name: ativo
                  type: BOOLEAN
                  defaultValueBoolean: true
                  constraints:
                    nullable: false
              - column:
                  name: criado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
                  constraints:
                    nullable: false
        - createIndex:
            indexName: idx_usuario_email
            tableName: usuario
            columns:
              - column:
                  name: email
```

#### XML (formato clássico, mais verboso)

```xml
<!-- db/changelog/changes/001-criar-tabela-usuario.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">

    <changeSet id="1" author="equipe">
        <comment>Criação da tabela de usuários do sistema</comment>
        <createTable tableName="usuario">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true" primaryKeyName="pk_usuario"/>
            </column>
            <column name="nome" type="VARCHAR(100)">
                <constraints nullable="false"/>
            </column>
            <column name="email" type="VARCHAR(150)">
                <constraints nullable="false" unique="true"
                             uniqueConstraintName="uk_usuario_email"/>
            </column>
            <column name="senha_hash" type="VARCHAR(255)">
                <constraints nullable="false"/>
            </column>
            <column name="ativo" type="BOOLEAN" defaultValueBoolean="true">
                <constraints nullable="false"/>
            </column>
            <column name="criado_em" type="TIMESTAMP"
                    defaultValueComputed="CURRENT_TIMESTAMP">
                <constraints nullable="false"/>
            </column>
        </createTable>

        <createIndex indexName="idx_usuario_email" tableName="usuario">
            <column name="email"/>
        </createIndex>
    </changeSet>
</databaseChangeLog>
```

#### SQL nativo (para DDL complexo ou específico do banco)

```sql
-- db/changelog/changes/001-criar-tabela-usuario.sql

-- changeset equipe:1
-- comment: Criação da tabela de usuários do sistema
CREATE TABLE usuario (
    id         BIGSERIAL    PRIMARY KEY,
    nome       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL UNIQUE,
    senha_hash VARCHAR(255) NOT NULL,
    ativo      BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_usuario_email ON usuario(email);

-- rollback DROP TABLE usuario;
```

### 4.3 Estrutura de Changelogs

Liquibase usa um arquivo mestre que inclui os changelogs individuais:

```
src/main/resources/db/changelog/
├── db.changelog-master.yaml          # Arquivo mestre (entry point)
├── changes/
│   ├── 001-criar-tabela-usuario.yaml
│   ├── 002-criar-tabela-produto.yaml
│   ├── 003-criar-tabela-pedido.yaml
│   ├── 004-criar-tabela-item-pedido.yaml
│   ├── 005-adicionar-coluna-telefone.yaml
│   ├── 006-inserir-perfis-padrao.yaml
│   └── 007-criar-view-relatorio.yaml
└── data/
    └── dev-seed-data.yaml
```

#### Arquivo mestre

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-criar-tabela-usuario.yaml
  - include:
      file: db/changelog/changes/002-criar-tabela-produto.yaml
  - include:
      file: db/changelog/changes/003-criar-tabela-pedido.yaml
  - include:
      file: db/changelog/changes/004-criar-tabela-item-pedido.yaml
  - include:
      file: db/changelog/changes/005-adicionar-coluna-telefone.yaml
  - include:
      file: db/changelog/changes/006-inserir-perfis-padrao.yaml
  - include:
      file: db/changelog/changes/007-criar-view-relatorio.yaml

  # Dados de desenvolvimento (só executa com context "dev")
  - include:
      file: db/changelog/data/dev-seed-data.yaml
```

#### Usando includeAll para inclusão automática

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - includeAll:
      path: db/changelog/changes/
      # Arquivos são incluídos em ordem alfabética
```

#### Changelogs individuais

```yaml
# db/changelog/changes/002-criar-tabela-produto.yaml
databaseChangeLog:
  - changeSet:
      id: 2
      author: equipe
      changes:
        - createTable:
            tableName: produto
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nome
                  type: VARCHAR(200)
                  constraints:
                    nullable: false
              - column:
                  name: descricao
                  type: TEXT
              - column:
                  name: preco
                  type: DECIMAL(10,2)
                  constraints:
                    nullable: false
              - column:
                  name: estoque
                  type: INT
                  defaultValueNumeric: 0
                  constraints:
                    nullable: false
              - column:
                  name: ativo
                  type: BOOLEAN
                  defaultValueBoolean: true
```

```yaml
# db/changelog/changes/003-criar-tabela-pedido.yaml
databaseChangeLog:
  - changeSet:
      id: 3
      author: equipe
      changes:
        - createTable:
            tableName: pedido
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: usuario_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_pedido_usuario
                    references: usuario(id)
              - column:
                  name: status
                  type: VARCHAR(20)
                  defaultValue: CRIADO
                  constraints:
                    nullable: false
              - column:
                  name: total
                  type: DECIMAL(12,2)
                  defaultValueNumeric: 0
                  constraints:
                    nullable: false
              - column:
                  name: criado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
              - column:
                  name: atualizado_em
                  type: TIMESTAMP

        - createIndex:
            indexName: idx_pedido_usuario
            tableName: pedido
            columns:
              - column:
                  name: usuario_id

        - createIndex:
            indexName: idx_pedido_status
            tableName: pedido
            columns:
              - column:
                  name: status
```

```yaml
# db/changelog/changes/005-adicionar-coluna-telefone.yaml
databaseChangeLog:
  - changeSet:
      id: 5
      author: equipe
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: telefone
                  type: VARCHAR(20)
      rollback:
        - dropColumn:
            tableName: usuario
            columnName: telefone
```

```yaml
# db/changelog/changes/006-inserir-perfis-padrao.yaml
databaseChangeLog:
  - changeSet:
      id: 6
      author: equipe
      changes:
        - createTable:
            tableName: perfil
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nome
                  type: VARCHAR(50)
                  constraints:
                    nullable: false
                    unique: true

        - insert:
            tableName: perfil
            columns:
              - column: { name: nome, value: ADMIN }
        - insert:
            tableName: perfil
            columns:
              - column: { name: nome, value: USUARIO }
        - insert:
            tableName: perfil
            columns:
              - column: { name: nome, value: MODERADOR }

        - createTable:
            tableName: usuario_perfil
            columns:
              - column:
                  name: usuario_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_up_usuario
                    references: usuario(id)
              - column:
                  name: perfil_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_up_perfil
                    references: perfil(id)

        - addPrimaryKey:
            tableName: usuario_perfil
            columnNames: usuario_id, perfil_id
            constraintName: pk_usuario_perfil
```

### 4.4 Tabela de Controle — DATABASECHANGELOG

Liquibase cria duas tabelas de controle:

```sql
-- Histórico de changeSets aplicados
SELECT id, author, filename, dateexecuted, orderexecuted,
       exectype, md5sum, description
FROM DATABASECHANGELOG;
```

| id | author | filename | dateexecuted | exectype | description |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | equipe | changes/001-... | 2025-01-15 10:00 | EXECUTED | createTable tableName=usuario |
| 2 | equipe | changes/002-... | 2025-01-15 10:00 | EXECUTED | createTable tableName=produto |
| 5 | equipe | changes/005-... | 2025-02-01 14:30 | EXECUTED | addColumn tableName=usuario |

```sql
-- Locks para evitar execução simultânea
SELECT id, locked, lockgranted, lockedby FROM DATABASECHANGELOGLOCK;
```

### 4.5 Preconditions — Execução Condicional

Preconditions permitem que changeSets só executem quando certas condições forem atendidas:

```yaml
databaseChangeLog:
  - changeSet:
      id: 10
      author: equipe
      preConditions:
        - onFail: MARK_RAN    # Marca como executado se a precondição falhar
        - not:
            - tableExists:
                tableName: auditoria
      changes:
        - createTable:
            tableName: auditoria
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: entidade
                  type: VARCHAR(100)
              - column:
                  name: acao
                  type: VARCHAR(20)
              - column:
                  name: dados_json
                  type: TEXT
              - column:
                  name: executado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP

  - changeSet:
      id: 11
      author: equipe
      preConditions:
        - onFail: MARK_RAN
        - columnExists:
            tableName: usuario
            columnName: email
        - not:
            - columnExists:
                tableName: usuario
                columnName: email_verificado
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: email_verificado
                  type: BOOLEAN
                  defaultValueBoolean: false
```

| Ação onFail | Comportamento |
|-------------|---------------|
| `HALT` | Para a execução (padrão) |
| `CONTINUE` | Pula o changeSet e continua |
| `MARK_RAN` | Marca como executado e continua |
| `WARN` | Emite aviso e continua |

### 4.6 Rollback Nativo

Uma das maiores vantagens do Liquibase é o suporte nativo a rollback:

```yaml
databaseChangeLog:
  - changeSet:
      id: 20
      author: equipe
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: data_nascimento
                  type: DATE
              - column:
                  name: cpf
                  type: VARCHAR(14)
                  constraints:
                    unique: true
                    uniqueConstraintName: uk_usuario_cpf
      rollback:
        - dropColumn:
            tableName: usuario
            columnName: cpf
        - dropColumn:
            tableName: usuario
            columnName: data_nascimento
```

Muitos change types têm rollback automático:

| Change Type | Rollback automático |
|-------------|:-------------------:|
| `createTable` | `DROP TABLE` |
| `addColumn` | `DROP COLUMN` |
| `createIndex` | `DROP INDEX` |
| `addForeignKeyConstraint` | `DROP CONSTRAINT` |
| `insert` | `DELETE` |
| `createView` | `DROP VIEW` |
| `renameTable` | Renomeia de volta |

### 4.7 Contexts e Labels

Contexts permitem que changeSets executem apenas em ambientes específicos:

```yaml
databaseChangeLog:
  # ChangeSet que executa em todos os ambientes
  - changeSet:
      id: 30
      author: equipe
      changes:
        - createTable:
            tableName: configuracao
            columns:
              - column:
                  name: chave
                  type: VARCHAR(100)
                  constraints:
                    primaryKey: true
              - column:
                  name: valor
                  type: VARCHAR(500)

  # Dados de seed — somente em dev e test
  - changeSet:
      id: 31
      author: equipe
      context: dev, test
      changes:
        - insert:
            tableName: configuracao
            columns:
              - column: { name: chave, value: app.modo }
              - column: { name: chave, value: debug }
        - insert:
            tableName: usuario
            columns:
              - column: { name: nome, value: Admin Dev }
              - column: { name: email, value: admin@dev.local }
              - column: { name: senha_hash, value: $2a$10$hash... }

  # Configurações de produção
  - changeSet:
      id: 32
      author: equipe
      context: prod
      changes:
        - insert:
            tableName: configuracao
            columns:
              - column: { name: chave, value: app.modo }
              - column: { name: valor, value: production }
```

```yaml
# application-dev.yml
spring:
  liquibase:
    contexts: dev

# application-prod.yml
spring:
  liquibase:
    contexts: prod
```

**Labels** oferecem controle ainda mais granular:

```yaml
- changeSet:
    id: 40
    author: equipe
    labels: "sprint-15, feature-pagamento"
    changes:
      - createTable:
          tableName: pagamento
          # ...
```

```yaml
# Executar apenas changeSets da sprint 15
spring:
  liquibase:
    label-filter: "sprint-15"
```

### 4.8 Comandos do Liquibase

| Comando | Descrição |
|---------|-----------|
| `update` | Aplica changeSets pendentes |
| `rollback` | Desfaz changeSets (por count, date ou tag) |
| `rollbackCount N` | Desfaz os últimos N changeSets |
| `rollbackToDate` | Desfaz changeSets até uma data |
| `tag` | Marca o estado atual com um rótulo |
| `status` | Mostra changeSets pendentes |
| `diff` | Compara dois bancos de dados |
| `generateChangeLog` | Gera changelog a partir de banco existente |
| `validate` | Valida o changelog |
| `clearCheckSums` | Limpa checksums (revalida na próxima execução) |
| `dropAll` | Remove todos os objetos do schema (CUIDADO!) |

### 4.9 Diff e Geração Automática de Changelogs

O Liquibase pode comparar bancos e gerar changelogs automaticamente — útil para criar o changelog inicial a partir de um banco existente:

```bash
# Gerar changelog a partir de banco existente
liquibase --url=jdbc:postgresql://localhost:5432/minha_app \
          --username=postgres --password=postgres \
          generate-changelog --changelog-file=db.changelog-initial.yaml

# Comparar dois bancos
liquibase diff \
    --url=jdbc:postgresql://localhost:5432/dev_db \
    --referenceUrl=jdbc:postgresql://localhost:5432/prod_db \
    --username=postgres --password=postgres

# Gerar changelog das diferenças
liquibase diff-changelog \
    --url=jdbc:postgresql://localhost:5432/dev_db \
    --referenceUrl=jdbc:postgresql://localhost:5432/prod_db \
    --changelog-file=db.changelog-diff.yaml
```

### 4.10 Plugin Maven e Gradle

#### Maven

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-maven-plugin</artifactId>
            <configuration>
                <url>jdbc:postgresql://localhost:5432/minha_app</url>
                <username>postgres</username>
                <password>postgres</password>
                <changeLogFile>
                    src/main/resources/db/changelog/db.changelog-master.yaml
                </changeLogFile>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```bash
./mvnw liquibase:update           # Aplicar changeSets pendentes
./mvnw liquibase:status           # Ver changeSets pendentes
./mvnw liquibase:rollback -Dliquibase.rollbackCount=1   # Desfazer último
./mvnw liquibase:diff             # Comparar schemas
./mvnw liquibase:generateChangeLog    # Gerar changelog do banco
./mvnw liquibase:tag -Dliquibase.tag=release-1.0        # Marcar tag
./mvnw liquibase:rollback -Dliquibase.rollbackTag=release-1.0  # Voltar para tag
```

#### Gradle

```groovy
plugins {
    id 'org.liquibase.gradle' version '2.2.2'
}

liquibase {
    activities {
        main {
            changelogFile 'src/main/resources/db/changelog/db.changelog-master.yaml'
            url 'jdbc:postgresql://localhost:5432/minha_app'
            username 'postgres'
            password 'postgres'
        }
    }
}
```

---

## 5. Tutorial Prático — Sistema de E-commerce com Flyway

### Estrutura do projeto

```
ecommerce-flyway/
├── pom.xml
├── src/main/
│   ├── java/com/example/ecommerce/
│   │   ├── EcommerceApplication.java
│   │   ├── model/
│   │   │   ├── Usuario.java
│   │   │   ├── Produto.java
│   │   │   ├── Pedido.java
│   │   │   └── ItemPedido.java
│   │   ├── repository/
│   │   │   ├── UsuarioRepository.java
│   │   │   ├── ProdutoRepository.java
│   │   │   └── PedidoRepository.java
│   │   └── controller/
│   │       ├── UsuarioController.java
│   │       └── PedidoController.java
│   └── resources/
│       ├── application.yml
│       └── db/migration/
│           ├── V1__criar_tabela_usuario.sql
│           ├── V2__criar_tabela_produto.sql
│           ├── V3__criar_tabela_pedido_e_itens.sql
│           ├── V4__criar_tabela_endereco.sql
│           ├── V5__inserir_dados_iniciais.sql
│           ├── V6__adicionar_coluna_cpf_usuario.sql
│           ├── V7__criar_tabela_cupom_desconto.sql
│           └── R__view_dashboard_vendas.sql
└── docker-compose.yml
```

### docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ecommerce
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### application.yml

```yaml
spring:
  application:
    name: ecommerce-flyway

  datasource:
    url: jdbc:postgresql://localhost:5432/ecommerce
    username: postgres
    password: postgres

  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
    open-in-view: false

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

### Migrations SQL

```sql
-- V1__criar_tabela_usuario.sql
CREATE TABLE usuario (
    id         BIGSERIAL    PRIMARY KEY,
    nome       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL,
    senha_hash VARCHAR(255) NOT NULL,
    ativo      BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT uk_usuario_email UNIQUE (email)
);
```

```sql
-- V2__criar_tabela_produto.sql
CREATE TABLE categoria (
    id   BIGSERIAL   PRIMARY KEY,
    nome VARCHAR(80) NOT NULL UNIQUE
);

CREATE TABLE produto (
    id           BIGSERIAL      PRIMARY KEY,
    nome         VARCHAR(200)   NOT NULL,
    descricao    TEXT,
    preco        DECIMAL(10, 2) NOT NULL CHECK (preco >= 0),
    estoque      INT            NOT NULL DEFAULT 0 CHECK (estoque >= 0),
    categoria_id BIGINT         REFERENCES categoria(id),
    ativo        BOOLEAN        NOT NULL DEFAULT TRUE,
    criado_em    TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP
);

CREATE INDEX idx_produto_categoria ON produto(categoria_id);
CREATE INDEX idx_produto_ativo     ON produto(ativo) WHERE ativo = TRUE;
```

```sql
-- V3__criar_tabela_pedido_e_itens.sql
CREATE TABLE pedido (
    id            BIGSERIAL      PRIMARY KEY,
    usuario_id    BIGINT         NOT NULL REFERENCES usuario(id),
    status        VARCHAR(20)    NOT NULL DEFAULT 'PENDENTE',
    subtotal      DECIMAL(12, 2) NOT NULL DEFAULT 0,
    desconto      DECIMAL(12, 2) NOT NULL DEFAULT 0,
    total         DECIMAL(12, 2) NOT NULL DEFAULT 0,
    observacao    TEXT,
    criado_em     TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    atualizado_em TIMESTAMP,

    CONSTRAINT ck_pedido_status CHECK (
        status IN ('PENDENTE', 'CONFIRMADO', 'PREPARANDO',
                   'ENVIADO', 'ENTREGUE', 'CANCELADO')
    )
);

CREATE TABLE item_pedido (
    id           BIGSERIAL      PRIMARY KEY,
    pedido_id    BIGINT         NOT NULL REFERENCES pedido(id) ON DELETE CASCADE,
    produto_id   BIGINT         NOT NULL REFERENCES produto(id),
    quantidade   INT            NOT NULL CHECK (quantidade > 0),
    preco_unit   DECIMAL(10, 2) NOT NULL CHECK (preco_unit >= 0),
    subtotal     DECIMAL(12, 2) NOT NULL
);

CREATE INDEX idx_pedido_usuario   ON pedido(usuario_id);
CREATE INDEX idx_pedido_status    ON pedido(status);
CREATE INDEX idx_pedido_criado    ON pedido(criado_em);
CREATE INDEX idx_item_pedido      ON item_pedido(pedido_id);
```

```sql
-- V4__criar_tabela_endereco.sql
CREATE TABLE endereco (
    id          BIGSERIAL    PRIMARY KEY,
    usuario_id  BIGINT       NOT NULL REFERENCES usuario(id) ON DELETE CASCADE,
    logradouro  VARCHAR(200) NOT NULL,
    numero      VARCHAR(20)  NOT NULL,
    complemento VARCHAR(100),
    bairro      VARCHAR(100) NOT NULL,
    cidade      VARCHAR(100) NOT NULL,
    estado      CHAR(2)      NOT NULL,
    cep         VARCHAR(10)  NOT NULL,
    principal   BOOLEAN      NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_endereco_usuario ON endereco(usuario_id);
```

```sql
-- V5__inserir_dados_iniciais.sql
INSERT INTO categoria (nome) VALUES
    ('Eletrônicos'),
    ('Livros'),
    ('Roupas'),
    ('Alimentos'),
    ('Esportes');

INSERT INTO produto (nome, descricao, preco, estoque, categoria_id) VALUES
    ('Notebook Pro 15',   'Notebook com tela de 15 polegadas, 16GB RAM', 4500.00, 25, 1),
    ('Mouse Wireless',    'Mouse sem fio ergonômico',                      89.90, 150, 1),
    ('Clean Code',        'Robert C. Martin - Guia de boas práticas',      79.90,  50, 2),
    ('Camiseta Dev Java', 'Camiseta 100% algodão',                         59.90, 200, 3),
    ('Whey Protein 1kg',  'Proteína isolada sabor chocolate',             120.00,  80, 4);
```

```sql
-- V6__adicionar_coluna_cpf_usuario.sql
ALTER TABLE usuario ADD COLUMN cpf VARCHAR(14);
CREATE UNIQUE INDEX idx_usuario_cpf ON usuario(cpf) WHERE cpf IS NOT NULL;
```

```sql
-- V7__criar_tabela_cupom_desconto.sql
CREATE TABLE cupom_desconto (
    id             BIGSERIAL      PRIMARY KEY,
    codigo         VARCHAR(30)    NOT NULL UNIQUE,
    tipo           VARCHAR(15)    NOT NULL,
    valor          DECIMAL(10, 2) NOT NULL,
    minimo_pedido  DECIMAL(10, 2),
    uso_maximo     INT,
    uso_atual      INT            NOT NULL DEFAULT 0,
    ativo          BOOLEAN        NOT NULL DEFAULT TRUE,
    valido_ate     TIMESTAMP,

    CONSTRAINT ck_cupom_tipo CHECK (tipo IN ('PERCENTUAL', 'VALOR_FIXO'))
);

INSERT INTO cupom_desconto (codigo, tipo, valor, minimo_pedido, uso_maximo, valido_ate) VALUES
    ('BEMVINDO10', 'PERCENTUAL', 10.00, 50.00,  NULL, '2026-12-31'),
    ('FRETE20',    'VALOR_FIXO', 20.00, 100.00, 500,  '2026-06-30');
```

```sql
-- R__view_dashboard_vendas.sql
CREATE OR REPLACE VIEW vw_dashboard_vendas AS
SELECT
    c.nome                              AS categoria,
    COUNT(DISTINCT p.id)                AS total_pedidos,
    SUM(ip.quantidade)                  AS itens_vendidos,
    SUM(ip.subtotal)                    AS receita_bruta,
    SUM(ped.desconto)                   AS descontos_concedidos,
    SUM(ip.subtotal) - SUM(ped.desconto) AS receita_liquida,
    AVG(ped.total)                      AS ticket_medio,
    COUNT(DISTINCT ped.usuario_id)      AS clientes_unicos
FROM item_pedido ip
JOIN produto p      ON p.id = ip.produto_id
JOIN categoria c    ON c.id = p.categoria_id
JOIN pedido ped     ON ped.id = ip.pedido_id
WHERE ped.status IN ('CONFIRMADO', 'ENVIADO', 'ENTREGUE')
GROUP BY c.id, c.nome;
```

### Entities JPA

```java
@Entity
@Table(name = "usuario")
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nome;

    @Column(nullable = false, length = 150, unique = true)
    private String email;

    @Column(name = "senha_hash", nullable = false)
    private String senhaHash;

    @Column(length = 14)
    private String cpf;

    @Column(length = 20)
    private String telefone;

    @Column(nullable = false)
    private Boolean ativo = true;

    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm = LocalDateTime.now();

    @OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL)
    private List<Endereco> enderecos = new ArrayList<>();

    // Construtores, getters e setters
}
```

```java
@Entity
@Table(name = "pedido")
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "usuario_id", nullable = false)
    private Usuario usuario;

    @Column(nullable = false, length = 20)
    private String status = "PENDENTE";

    @Column(nullable = false)
    private BigDecimal subtotal = BigDecimal.ZERO;

    @Column(nullable = false)
    private BigDecimal desconto = BigDecimal.ZERO;

    @Column(nullable = false)
    private BigDecimal total = BigDecimal.ZERO;

    private String observacao;

    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm = LocalDateTime.now();

    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    @OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ItemPedido> itens = new ArrayList<>();

    public void adicionarItem(ItemPedido item) {
        itens.add(item);
        item.setPedido(this);
        recalcularTotais();
    }

    private void recalcularTotais() {
        this.subtotal = itens.stream()
            .map(ItemPedido::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        this.total = this.subtotal.subtract(this.desconto);
        this.atualizadoEm = LocalDateTime.now();
    }

    // Construtores, getters e setters
}
```

```java
@Entity
@Table(name = "item_pedido")
public class ItemPedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "pedido_id", nullable = false)
    private Pedido pedido;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "produto_id", nullable = false)
    private Produto produto;

    @Column(nullable = false)
    private Integer quantidade;

    @Column(name = "preco_unit", nullable = false)
    private BigDecimal precoUnit;

    @Column(nullable = false)
    private BigDecimal subtotal;

    @PrePersist
    @PreUpdate
    void calcularSubtotal() {
        this.subtotal = this.precoUnit.multiply(BigDecimal.valueOf(quantidade));
    }

    // Construtores, getters e setters
}
```

---

## 6. Tutorial Prático — Sistema de E-commerce com Liquibase

O mesmo domínio do tutorial anterior, agora com Liquibase:

### Estrutura do projeto

```
ecommerce-liquibase/
├── pom.xml
├── src/main/resources/
│   ├── application.yml
│   └── db/changelog/
│       ├── db.changelog-master.yaml
│       └── changes/
│           ├── 001-criar-tabela-usuario.yaml
│           ├── 002-criar-tabela-categoria-produto.yaml
│           ├── 003-criar-tabela-pedido-e-itens.yaml
│           ├── 004-criar-tabela-endereco.yaml
│           ├── 005-inserir-dados-iniciais.yaml
│           ├── 006-adicionar-coluna-cpf.yaml
│           ├── 007-criar-tabela-cupom-desconto.yaml
│           └── 008-criar-view-dashboard.yaml
└── docker-compose.yml
```

### Arquivo mestre

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-criar-tabela-usuario.yaml
  - include:
      file: db/changelog/changes/002-criar-tabela-categoria-produto.yaml
  - include:
      file: db/changelog/changes/003-criar-tabela-pedido-e-itens.yaml
  - include:
      file: db/changelog/changes/004-criar-tabela-endereco.yaml
  - include:
      file: db/changelog/changes/005-inserir-dados-iniciais.yaml
  - include:
      file: db/changelog/changes/006-adicionar-coluna-cpf.yaml
  - include:
      file: db/changelog/changes/007-criar-tabela-cupom-desconto.yaml
  - include:
      file: db/changelog/changes/008-criar-view-dashboard.yaml
```

### Changelogs

```yaml
# changes/001-criar-tabela-usuario.yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: equipe
      changes:
        - createTable:
            tableName: usuario
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    primaryKeyName: pk_usuario
              - column:
                  name: nome
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
              - column:
                  name: email
                  type: VARCHAR(150)
                  constraints:
                    nullable: false
                    unique: true
                    uniqueConstraintName: uk_usuario_email
              - column:
                  name: senha_hash
                  type: VARCHAR(255)
                  constraints:
                    nullable: false
              - column:
                  name: ativo
                  type: BOOLEAN
                  defaultValueBoolean: true
                  constraints:
                    nullable: false
              - column:
                  name: criado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
                  constraints:
                    nullable: false
```

```yaml
# changes/002-criar-tabela-categoria-produto.yaml
databaseChangeLog:
  - changeSet:
      id: 2-categoria
      author: equipe
      changes:
        - createTable:
            tableName: categoria
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nome
                  type: VARCHAR(80)
                  constraints:
                    nullable: false
                    unique: true

  - changeSet:
      id: 2-produto
      author: equipe
      changes:
        - createTable:
            tableName: produto
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: nome
                  type: VARCHAR(200)
                  constraints:
                    nullable: false
              - column:
                  name: descricao
                  type: TEXT
              - column:
                  name: preco
                  type: DECIMAL(10,2)
                  constraints:
                    nullable: false
              - column:
                  name: estoque
                  type: INT
                  defaultValueNumeric: 0
                  constraints:
                    nullable: false
              - column:
                  name: categoria_id
                  type: BIGINT
                  constraints:
                    foreignKeyName: fk_produto_categoria
                    references: categoria(id)
              - column:
                  name: ativo
                  type: BOOLEAN
                  defaultValueBoolean: true
                  constraints:
                    nullable: false
              - column:
                  name: criado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
              - column:
                  name: atualizado_em
                  type: TIMESTAMP

        - createIndex:
            indexName: idx_produto_categoria
            tableName: produto
            columns:
              - column:
                  name: categoria_id
```

```yaml
# changes/003-criar-tabela-pedido-e-itens.yaml
databaseChangeLog:
  - changeSet:
      id: 3-pedido
      author: equipe
      changes:
        - createTable:
            tableName: pedido
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: usuario_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_pedido_usuario
                    references: usuario(id)
              - column:
                  name: status
                  type: VARCHAR(20)
                  defaultValue: PENDENTE
                  constraints:
                    nullable: false
              - column:
                  name: subtotal
                  type: DECIMAL(12,2)
                  defaultValueNumeric: 0
              - column:
                  name: desconto
                  type: DECIMAL(12,2)
                  defaultValueNumeric: 0
              - column:
                  name: total
                  type: DECIMAL(12,2)
                  defaultValueNumeric: 0
              - column:
                  name: observacao
                  type: TEXT
              - column:
                  name: criado_em
                  type: TIMESTAMP
                  defaultValueComputed: CURRENT_TIMESTAMP
              - column:
                  name: atualizado_em
                  type: TIMESTAMP

        - createIndex:
            indexName: idx_pedido_usuario
            tableName: pedido
            columns:
              - column:
                  name: usuario_id

        - createIndex:
            indexName: idx_pedido_status
            tableName: pedido
            columns:
              - column:
                  name: status

  - changeSet:
      id: 3-item-pedido
      author: equipe
      changes:
        - createTable:
            tableName: item_pedido
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: pedido_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_item_pedido
                    deleteCascade: true
                    references: pedido(id)
              - column:
                  name: produto_id
                  type: BIGINT
                  constraints:
                    nullable: false
                    foreignKeyName: fk_item_produto
                    references: produto(id)
              - column:
                  name: quantidade
                  type: INT
                  constraints:
                    nullable: false
              - column:
                  name: preco_unit
                  type: DECIMAL(10,2)
                  constraints:
                    nullable: false
              - column:
                  name: subtotal
                  type: DECIMAL(12,2)
                  constraints:
                    nullable: false

        - createIndex:
            indexName: idx_item_pedido_pedido
            tableName: item_pedido
            columns:
              - column:
                  name: pedido_id
```

```yaml
# changes/005-inserir-dados-iniciais.yaml
databaseChangeLog:
  - changeSet:
      id: 5-categorias
      author: equipe
      changes:
        - insert:
            tableName: categoria
            columns:
              - column: { name: nome, value: Eletrônicos }
        - insert:
            tableName: categoria
            columns:
              - column: { name: nome, value: Livros }
        - insert:
            tableName: categoria
            columns:
              - column: { name: nome, value: Roupas }
        - insert:
            tableName: categoria
            columns:
              - column: { name: nome, value: Alimentos }
        - insert:
            tableName: categoria
            columns:
              - column: { name: nome, value: Esportes }
      rollback:
        - delete:
            tableName: categoria

  - changeSet:
      id: 5-produtos
      author: equipe
      changes:
        - loadData:
            tableName: produto
            file: db/changelog/data/produtos.csv
            separator: ","
            columns:
              - column: { name: nome, type: STRING }
              - column: { name: descricao, type: STRING }
              - column: { name: preco, type: NUMERIC }
              - column: { name: estoque, type: NUMERIC }
              - column: { name: categoria_id, type: NUMERIC }
```

```yaml
# changes/006-adicionar-coluna-cpf.yaml
databaseChangeLog:
  - changeSet:
      id: 6
      author: equipe
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: cpf
                  type: VARCHAR(14)
        - addUniqueConstraint:
            tableName: usuario
            columnNames: cpf
            constraintName: uk_usuario_cpf
      rollback:
        - dropUniqueConstraint:
            tableName: usuario
            constraintName: uk_usuario_cpf
        - dropColumn:
            tableName: usuario
            columnName: cpf
```

```yaml
# changes/008-criar-view-dashboard.yaml
databaseChangeLog:
  - changeSet:
      id: 8
      author: equipe
      runOnChange: true   # Re-executa quando o conteúdo mudar (similar ao R__ do Flyway)
      changes:
        - createView:
            viewName: vw_dashboard_vendas
            replaceIfExists: true
            selectQuery: >
              SELECT
                c.nome AS categoria,
                COUNT(DISTINCT p.id) AS total_pedidos,
                SUM(ip.quantidade) AS itens_vendidos,
                SUM(ip.subtotal) AS receita_bruta,
                AVG(ped.total) AS ticket_medio,
                COUNT(DISTINCT ped.usuario_id) AS clientes_unicos
              FROM item_pedido ip
              JOIN produto p ON p.id = ip.produto_id
              JOIN categoria c ON c.id = p.categoria_id
              JOIN pedido ped ON ped.id = ip.pedido_id
              WHERE ped.status IN ('CONFIRMADO', 'ENVIADO', 'ENTREGUE')
              GROUP BY c.id, c.nome
      rollback:
        - dropView:
            viewName: vw_dashboard_vendas
```

---

## 7. Flyway vs Liquibase — Comparativo Detalhado

| Aspecto | Flyway | Liquibase |
|---------|--------|-----------|
| **Formato principal** | SQL puro | XML, YAML, JSON ou SQL |
| **Curva de aprendizado** | Baixa — é SQL | Moderada — DSL declarativa |
| **Portabilidade multi-banco** | Depende do dialeto SQL | Abstração nativa (traduz para cada banco) |
| **Rollback** | Somente versão paga (Teams) | Nativo e gratuito |
| **Preconditions** | Não nativo | Suporte completo |
| **Geração de changelog** | Não suporta | `generateChangeLog`, `diff` |
| **Contexts / Ambientes** | Via locations e profiles | `context` e `labels` nativos no changeSet |
| **Migrations Java** | `BaseJavaMigration` | `CustomTaskChange` |
| **Controle de execução** | Checksum do arquivo | Checksum + `runOnChange`, `runAlways` |
| **Integração Spring Boot** | Auto-configuração | Auto-configuração |
| **Comunidade Spring** | Mais popular | Bem suportado |
| **Licença** | Apache 2.0 (core) | Apache 2.0 (core) |
| **Versão paga** | Teams / Enterprise | Pro / Enterprise |

### Quando usar Flyway

- Equipe familiarizada com SQL
- Projeto usa um único banco de dados
- Schema simples a moderado
- Não precisa de rollback automatizado (faz via nova migration)
- Quer a menor curva de aprendizado possível

### Quando usar Liquibase

- Projeto precisa suportar múltiplos bancos (ex: PostgreSQL em produção, H2 em testes)
- Rollback automatizado é requisito
- Precisa de execução condicional (preconditions)
- Quer gerar changelogs a partir de banco existente
- Controle granular com contexts e labels

---

## 8. Testcontainers e Migrations

Testcontainers permite executar testes de integração com um banco de dados real em container Docker, garantindo que as migrations são válidas:

### Configuração

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

### Teste de migrations com Flyway

```java
@SpringBootTest
@Testcontainers
class FlywayMigrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private Flyway flyway;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    void todasMigrationsDevemSerAplicadasComSucesso() {
        MigrationInfoService info = flyway.info();
        MigrationInfo[] applied = info.applied();
        MigrationInfo[] pending = info.pending();

        assertThat(applied).isNotEmpty();
        assertThat(pending).isEmpty();
        assertThat(Arrays.stream(applied))
            .allMatch(m -> m.getState() == MigrationState.SUCCESS);
    }

    @Test
    void schemaDeveConterTabelasEsperadas() {
        List<String> tabelas = jdbcTemplate.queryForList("""
            SELECT table_name FROM information_schema.tables
            WHERE table_schema = 'public'
              AND table_type = 'BASE TABLE'
            ORDER BY table_name
            """, String.class);

        assertThat(tabelas).contains(
            "usuario", "produto", "pedido", "item_pedido",
            "endereco", "categoria", "cupom_desconto"
        );
    }

    @Test
    void viewDashboardDeveExistir() {
        List<String> views = jdbcTemplate.queryForList("""
            SELECT table_name FROM information_schema.views
            WHERE table_schema = 'public'
            """, String.class);

        assertThat(views).contains("vw_dashboard_vendas");
    }
}
```

### Teste de migrations com Liquibase

```java
@SpringBootTest
@Testcontainers
class LiquibaseMigrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired
    private SpringLiquibase liquibase;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    void todasMigrationsDevemSerAplicadas() {
        Integer pendentes = jdbcTemplate.queryForObject("""
            SELECT COUNT(*) FROM DATABASECHANGELOG
            WHERE exectype = 'EXECUTED'
            """, Integer.class);

        assertThat(pendentes).isGreaterThan(0);
    }

    @Test
    void colunasCriticasDevemExistir() {
        List<String> colunas = jdbcTemplate.queryForList("""
            SELECT column_name FROM information_schema.columns
            WHERE table_name = 'usuario'
            ORDER BY ordinal_position
            """, String.class);

        assertThat(colunas).contains("id", "nome", "email", "senha_hash", "ativo", "cpf");
    }
}
```

### Base de teste reutilizável

```java
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfig {

    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgresContainer() {
        return new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    }
}
```

```java
// Uso em qualquer teste
@SpringBootTest
@Import(TestcontainersConfig.class)
class PedidoServiceIntegrationTest {

    @Autowired
    private PedidoService pedidoService;

    @Test
    void deveCriarPedidoComMigrationsAplicadas() {
        // As migrations (Flyway ou Liquibase) são aplicadas automaticamente
        // antes do teste rodar
        var pedido = pedidoService.criar(new CriarPedidoRequest(/* ... */));
        assertThat(pedido.getId()).isNotNull();
    }
}
```

---

## 9. Boas Práticas para Migrations em Produção

### 9.1 Regras de ouro

```
┌──────────────────────────────────────────────────────────────────────┐
│                    REGRAS DE OURO PARA MIGRATIONS                    │
│                                                                      │
│  1. NUNCA altere uma migration já aplicada em qualquer ambiente      │
│  2. NUNCA use DROP TABLE/COLUMN sem migration de dados antes         │
│  3. SEMPRE teste migrations em ambiente de staging primeiro          │
│  4. SEMPRE faça backup antes de rodar migrations em produção         │
│  5. SEMPRE use transações (bancos que suportam DDL transacional)     │
│  6. NUNCA permita flyway clean em produção                           │
│  7. SEMPRE version migrations no Git junto com o código              │
│  8. MANTENHA cada migration pequena e focada                         │
└──────────────────────────────────────────────────────────────────────┘
```

### 9.2 Migrations retrocompatíveis (expand-contract)

Quando uma mudança de schema precisa ser coordenada com deploy da aplicação, use o padrão **expand-contract**:

```
FASE 1 — EXPAND (nova migration)
┌─────────────────────────────────────┐
│ Adiciona novas colunas/tabelas      │
│ Código antigo continua funcionando  │
│ Código novo já pode usar o novo     │
└─────────────────────────────────────┘
                  │
                  ▼
FASE 2 — MIGRAR DADOS (nova migration)
┌─────────────────────────────────────┐
│ Copia dados da estrutura antiga     │
│ para a nova estrutura               │
│ Ambas as versões do código OK       │
└─────────────────────────────────────┘
                  │
                  ▼
FASE 3 — CONTRACT (nova migration, após deploy)
┌─────────────────────────────────────┐
│ Remove colunas/tabelas antigas      │
│ Executada APÓS o deploy do código   │
│ que não usa mais a estrutura antiga │
└─────────────────────────────────────┘
```

#### Exemplo: Renomear coluna

```sql
-- V10__expand_renomear_nome_para_nome_completo.sql
-- FASE 1: Adicionar nova coluna
ALTER TABLE usuario ADD COLUMN nome_completo VARCHAR(100);
UPDATE usuario SET nome_completo = nome;
ALTER TABLE usuario ALTER COLUMN nome_completo SET NOT NULL;
```

```sql
-- V11__contract_remover_coluna_nome_antiga.sql
-- FASE 3: Remover coluna antiga (executar APÓS deploy que usa nome_completo)
ALTER TABLE usuario DROP COLUMN nome;
```

#### Exemplo: Dividir tabela

```sql
-- V12__expand_extrair_dados_contato.sql
-- FASE 1: Criar nova tabela e copiar dados
CREATE TABLE contato (
    id         BIGSERIAL    PRIMARY KEY,
    usuario_id BIGINT       NOT NULL UNIQUE REFERENCES usuario(id),
    telefone   VARCHAR(20),
    celular    VARCHAR(20)
);

INSERT INTO contato (usuario_id, telefone)
SELECT id, telefone FROM usuario WHERE telefone IS NOT NULL;
```

```sql
-- V13__contract_remover_telefone_do_usuario.sql
-- FASE 3: Remover da tabela original (após deploy)
ALTER TABLE usuario DROP COLUMN telefone;
```

### 9.3 Migrations seguras para tabelas grandes

Para tabelas com milhões de registros, algumas operações podem travar a tabela. Estratégias para evitar isso:

```sql
-- ❌ PERIGOSO em tabela grande: ADD COLUMN com DEFAULT bloqueia a tabela inteira
ALTER TABLE pedido ADD COLUMN prioridade INT NOT NULL DEFAULT 0;

-- ✅ SEGURO no PostgreSQL 11+: ADD COLUMN com DEFAULT não reescreve a tabela
-- (PostgreSQL 11+ é seguro, mas MySQL e versões antigas do PostgreSQL não são)

-- ✅ Para bancos que reescrevem: faça em etapas
ALTER TABLE pedido ADD COLUMN prioridade INT;
-- Atualizar em lotes:
UPDATE pedido SET prioridade = 0 WHERE id BETWEEN 1 AND 100000;
UPDATE pedido SET prioridade = 0 WHERE id BETWEEN 100001 AND 200000;
-- ... (em um script ou migration Java)
ALTER TABLE pedido ALTER COLUMN prioridade SET NOT NULL;
ALTER TABLE pedido ALTER COLUMN prioridade SET DEFAULT 0;
```

```sql
-- ✅ CREATE INDEX CONCURRENTLY (PostgreSQL) — não bloqueia escrita
CREATE INDEX CONCURRENTLY idx_pedido_criado ON pedido(criado_em);
-- Nota: não pode rodar dentro de transação; no Flyway, adicionar ao nome do arquivo:
-- V14__criar_indice_concurrently.sql (e configurar spring.flyway.out-of-order=true se necessário)
```

### 9.4 Numeração e organização

```
Projeto pequeno (< 50 migrations):
  V1, V2, V3, ...

Projeto médio (50-200 migrations):
  V1.0.1, V1.0.2, ..., V1.1.1, V1.1.2, ...

Projeto grande (200+ migrations):
  Use timestamps: V20250627120000__descricao.sql
  Evita conflitos quando múltiplas branches criam migrations
```

Para projetos com múltiplos desenvolvedores criando migrations simultaneamente, timestamps evitam conflitos de versão:

```
V20250627_091500__criar_tabela_notificacao.sql   (Dev A)
V20250627_103000__adicionar_coluna_avatar.sql    (Dev B)
```

### 9.5 Zero-Downtime Migrations

Em sistemas que não podem sair do ar para deploy, as migrations precisam ser compatíveis com duas versões simultâneas do código — a versão antiga (ainda rodando) e a nova (sendo implantada).

```
Deploy Blue-Green / Rolling Update:

Tempo ─────────────────────────────────────────────────────────►

│ Migration  │  Código V1     │  Código V1   │  Código V2     │
│ executada  │  (instâncias   │  + V2 rodam  │  (todas as     │
│            │   ativas)      │  lado a lado │   instâncias)  │
                              │              │
                    ┌─────────┘              └──────────┐
                    │  JANELA DE COMPATIBILIDADE        │
                    │  Schema deve funcionar com V1 E V2│
                    └───────────────────────────────────┘
```

#### Operações seguras vs perigosas

| Operação | Zero-downtime? | Estratégia |
|----------|:--------------:|------------|
| `CREATE TABLE` | Sim | Código antigo ignora tabela nova |
| `ADD COLUMN` (nullable) | Sim | Código antigo ignora coluna nova |
| `ADD COLUMN` (NOT NULL + default) | Depende do banco | PostgreSQL 11+: sim; MySQL: cuidado com lock |
| `CREATE INDEX` | Sim (com CONCURRENTLY) | Usar `CREATE INDEX CONCURRENTLY` |
| `DROP COLUMN` | **Perigoso** | Código antigo pode referenciar a coluna |
| `RENAME COLUMN` | **Perigoso** | Código antigo usa o nome antigo |
| `DROP TABLE` | **Perigoso** | Código antigo pode consultar a tabela |
| `ALTER COLUMN TYPE` | **Perigoso** | Pode ser incompatível com código antigo |

#### Exemplo completo: remover coluna com zero-downtime

```
DEPLOY 1 — Código V2 + Migration de preparação
┌──────────────────────────────────────────────────────────────┐
│  Código V2:                                                   │
│    - Para de ESCREVER na coluna "telefone"                    │
│    - Ainda LEITURA da coluna (fallback)                       │
│    - Escreve no novo local (tabela "contato")                 │
│                                                               │
│  Migration V10: Cria tabela "contato" e copia dados           │
└──────────────────────────────────────────────────────────────┘
         │
         ▼  (esperar todas as instâncias V1 serem desligadas)

DEPLOY 2 — Código V3 + Migration de limpeza
┌──────────────────────────────────────────────────────────────┐
│  Código V3:                                                   │
│    - Não referencia mais a coluna "telefone"                  │
│    - Usa exclusivamente tabela "contato"                      │
│                                                               │
│  Migration V11: DROP COLUMN telefone                          │
└──────────────────────────────────────────────────────────────┘
```

```sql
-- V10__preparar_remocao_telefone.sql (Deploy 1)
CREATE TABLE contato (
    id         BIGSERIAL PRIMARY KEY,
    usuario_id BIGINT    NOT NULL UNIQUE REFERENCES usuario(id),
    telefone   VARCHAR(20)
);

INSERT INTO contato (usuario_id, telefone)
SELECT id, telefone FROM usuario WHERE telefone IS NOT NULL;

-- NÃO remove a coluna antiga ainda!
```

```sql
-- V11__remover_coluna_telefone.sql (Deploy 2 — dias/semanas depois)
ALTER TABLE usuario DROP COLUMN telefone;
```

#### Flyway — migrations fora de ordem para deploys independentes

Quando diferentes equipes fazem deploy em momentos diferentes, pode ser necessário aceitar migrations fora de ordem:

```yaml
spring:
  flyway:
    out-of-order: true   # Aceita V3 antes de V2 se V2 ainda não foi aplicada
```

### 9.6 Data Migrations vs Schema Migrations

Nem toda migration é uma alteração de schema. Migrations de dados (data migrations) alteram os dados existentes sem mudar a estrutura das tabelas.

#### Quando separar schema de dados

```
┌─────────────────────────────────────────────────────────────┐
│                   SEPARAR EM MIGRATIONS DISTINTAS             │
│                                                               │
│  V5__adicionar_coluna_tipo_usuario.sql       ← Schema         │
│  V6__popular_tipo_usuario_existentes.sql     ← Dados          │
│  V7__tornar_tipo_usuario_not_null.sql        ← Schema         │
│                                                               │
│  Por que separar?                                             │
│  - Cada migration tem propósito claro                         │
│  - Se V6 falhar, V5 (schema) já está aplicada                │
│  - V7 só roda após dados estarem consistentes                 │
└─────────────────────────────────────────────────────────────┘
```

```sql
-- V5__adicionar_coluna_tipo_usuario.sql
ALTER TABLE usuario ADD COLUMN tipo VARCHAR(20);
```

```sql
-- V6__popular_tipo_usuario_existentes.sql
UPDATE usuario SET tipo = 'CLIENTE' WHERE tipo IS NULL;
```

```sql
-- V7__tornar_tipo_usuario_not_null.sql
ALTER TABLE usuario ALTER COLUMN tipo SET NOT NULL;
ALTER TABLE usuario ALTER COLUMN tipo SET DEFAULT 'CLIENTE';
```

#### Data migrations em tabelas grandes

Para tabelas com milhões de registros, atualizar tudo em uma transação pode causar locks prolongados e problemas de performance. Use batch processing:

```java
public class V8__normalizar_emails extends BaseJavaMigration {

    @Override
    public void migrate(Context context) throws Exception {
        var conn = context.getConnection();

        int batchSize = 5000;
        int totalAtualizado = 0;

        while (true) {
            try (var stmt = conn.prepareStatement("""
                    UPDATE usuario SET email = LOWER(email)
                    WHERE id IN (
                        SELECT id FROM usuario
                        WHERE email <> LOWER(email)
                        LIMIT ?
                    )
                    """)) {
                stmt.setInt(1, batchSize);
                int atualizado = stmt.executeUpdate();

                if (atualizado == 0) break;
                totalAtualizado += atualizado;
                conn.commit();
            }
        }
    }
}
```

#### Liquibase — Separação com changeSets

```yaml
databaseChangeLog:
  # Schema change
  - changeSet:
      id: 20-schema
      author: equipe
      changes:
        - addColumn:
            tableName: produto
            columns:
              - column:
                  name: slug
                  type: VARCHAR(250)

  # Data migration
  - changeSet:
      id: 20-data
      author: equipe
      changes:
        - sql:
            sql: >
              UPDATE produto SET slug = LOWER(REPLACE(REPLACE(nome, ' ', '-'), '.', ''))
              WHERE slug IS NULL

  # Constraint (após dados populados)
  - changeSet:
      id: 20-constraint
      author: equipe
      changes:
        - addNotNullConstraint:
            tableName: produto
            columnName: slug
        - addUniqueConstraint:
            tableName: produto
            columnNames: slug
            constraintName: uk_produto_slug
```

### 9.7 Dados Sensíveis em Migrations

Migrations de seed podem conter dados sensíveis — senhas, tokens, chaves de API. Estratégias para manter a segurança:

#### Nunca hardcode senhas em migrations

```sql
-- ❌ ERRADO: senha em texto claro na migration (vai para o Git!)
INSERT INTO usuario (nome, email, senha_hash)
VALUES ('Admin', 'admin@empresa.com', 'senha123');

-- ❌ ERRADO: mesmo hash fixo expõe a senha se alguém conhecer o algoritmo
INSERT INTO usuario (nome, email, senha_hash)
VALUES ('Admin', 'admin@empresa.com', '$2a$10$fixedHashQueVaiProGit');
```

#### Usar variáveis de ambiente

```sql
-- ✅ Flyway suporta placeholders
-- V10__criar_usuario_admin.sql
INSERT INTO usuario (nome, email, senha_hash)
VALUES ('Admin', '${ADMIN_EMAIL}', '${ADMIN_PASSWORD_HASH}');
```

```yaml
# application.yml
spring:
  flyway:
    placeholders:
      ADMIN_EMAIL: ${ADMIN_EMAIL:admin@localhost}
      ADMIN_PASSWORD_HASH: ${ADMIN_PASSWORD_HASH:placeholder}
```

```yaml
# Liquibase — property substitution
# db/changelog/changes/010-criar-admin.yaml
databaseChangeLog:
  - property:
      name: admin.email
      value: ${ADMIN_EMAIL:admin@localhost}
  - property:
      name: admin.password.hash
      value: ${ADMIN_PASSWORD_HASH:placeholder}

  - changeSet:
      id: 10
      author: equipe
      context: prod
      changes:
        - insert:
            tableName: usuario
            columns:
              - column: { name: nome, value: Admin }
              - column: { name: email, value: "${admin.email}" }
              - column: { name: senha_hash, value: "${admin.password.hash}" }
```

#### Separar dados sensíveis por contexto

```
src/main/resources/db/
├── migration/              # Migrations normais (vão para o Git)
│   ├── V1__schema.sql
│   └── V2__dados_referencia.sql
└── seed/                   # Dados sensíveis (NÃO vão para o Git)
    └── V100__admin_prod.sql
```

```yaml
# application-prod.yml
spring:
  flyway:
    locations:
      - classpath:db/migration
      - classpath:db/seed       # Só em produção
```

```gitignore
# .gitignore
src/main/resources/db/seed/
```

#### Migration Java para gerar dados seguros no momento da execução

```java
public class V11__criar_usuario_admin_seguro extends BaseJavaMigration {

    @Override
    public void migrate(Context context) throws Exception {
        String email = System.getenv("ADMIN_EMAIL");
        String senhaPlain = System.getenv("ADMIN_INITIAL_PASSWORD");

        if (email == null || senhaPlain == null) {
            throw new IllegalStateException(
                "ADMIN_EMAIL e ADMIN_INITIAL_PASSWORD devem estar definidos");
        }

        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        String hash = encoder.encode(senhaPlain);

        try (var stmt = context.getConnection().prepareStatement(
                "INSERT INTO usuario (nome, email, senha_hash, ativo) VALUES (?, ?, ?, true)")) {
            stmt.setString(1, "Administrador");
            stmt.setString(2, email);
            stmt.setString(3, hash);
            stmt.executeUpdate();
        }
    }
}
```

### 9.8 Anonimização de Dados entre Ambientes

É comum copiar o banco de produção para staging/dev para testes com dados reais. Os dados pessoais (PII) devem ser mascarados para cumprir LGPD/GDPR:

#### Migration de anonimização (executar após restore do backup)

```sql
-- R__anonimizar_dados_staging.sql (migration repetível, somente em staging)
-- Anonimizar dados pessoais após restore de produção

-- Emails
UPDATE usuario SET email = CONCAT('usuario_', id, '@teste.local');

-- Nomes
UPDATE usuario SET nome = CONCAT('Usuário Teste ', id);

-- CPF (gera CPF fictício baseado no ID)
UPDATE usuario SET cpf = LPAD(CAST(id AS VARCHAR), 11, '0')
WHERE cpf IS NOT NULL;

-- Telefones
UPDATE usuario SET telefone = CONCAT('(11) 9', LPAD(CAST(id AS VARCHAR), 4, '0'), '-0000')
WHERE telefone IS NOT NULL;

-- Senhas (resetar para uma senha padrão de staging)
UPDATE usuario SET senha_hash = '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi';
-- senha: "password" — apenas para staging!

-- Endereços
UPDATE endereco SET
    logradouro  = CONCAT('Rua Teste ', id),
    numero      = CAST(id AS VARCHAR),
    complemento = NULL,
    cep         = '01001-000';

-- Observações de pedidos (podem conter dados pessoais)
UPDATE pedido SET observacao = NULL WHERE observacao IS NOT NULL;
```

```yaml
# application-staging.yml
spring:
  flyway:
    locations:
      - classpath:db/migration
      - classpath:db/anonimizacao   # Scripts de anonimização
```

#### Liquibase com context para anonimização

```yaml
databaseChangeLog:
  - changeSet:
      id: anon-1
      author: equipe
      context: staging, dev
      runAlways: true
      changes:
        - sql:
            sql: >
              UPDATE usuario SET
                email = CONCAT('usuario_', id, '@teste.local'),
                nome = CONCAT('Usuário Teste ', id),
                telefone = CONCAT('(11) 9', LPAD(CAST(id AS TEXT), 4, '0'), '-0000'),
                senha_hash = '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi'
```

#### Automação com script pós-restore

```bash
#!/bin/bash
# scripts/restore-staging.sh
set -e

echo "Restaurando backup de produção em staging..."
pg_restore -d staging_db backup_prod.dump

echo "Executando anonimização..."
./mvnw flyway:migrate -Dspring.profiles.active=staging

echo "Staging pronto com dados anonimizados!"
```

### 9.9 Feature Toggles e Migrations

Quando uma funcionalidade grande requer alterações de schema mas não deve ser ativada imediatamente, combine migrations com feature toggles:

```
Abordagem: Deploy o schema antes, ative a feature depois

Sprint 1 (Deploy do schema):
┌──────────────────────────────────────────────┐
│  V20__criar_tabela_assinatura.sql             │
│  V21__criar_tabela_plano.sql                  │
│  V22__inserir_planos_padrao.sql               │
│                                               │
│  Feature toggle: assinatura.habilitada=false  │
│  Código existe mas não é acessível            │
└──────────────────────────────────────────────┘

Sprint 2 (Ativação gradual):
┌──────────────────────────────────────────────┐
│  Sem migrations novas!                        │
│                                               │
│  Feature toggle: assinatura.habilitada=true   │
│  → Ativado para 10% dos usuários (canary)     │
│  → Ativado para 50%                           │
│  → Ativado para 100%                          │
└──────────────────────────────────────────────┘

Sprint 3 (Limpeza — se a feature foi aprovada):
┌──────────────────────────────────────────────┐
│  Remover feature toggle do código             │
│  Remover condicional (feature é permanente)   │
└──────────────────────────────────────────────┘
```

#### Exemplo com Spring Boot

```yaml
# application.yml
app:
  features:
    assinatura-habilitada: ${FEATURE_ASSINATURA:false}
```

```java
@RestController
@RequestMapping("/api/assinaturas")
@ConditionalOnProperty(name = "app.features.assinatura-habilitada", havingValue = "true")
public class AssinaturaController {
    // Controller só é registrado se a feature estiver habilitada
    // Mas as tabelas já existem no banco (migrations já rodaram)
}
```

```sql
-- V20__criar_tabela_assinatura.sql
-- Pode ser aplicada antes da feature estar ativa
CREATE TABLE plano (
    id          BIGSERIAL      PRIMARY KEY,
    nome        VARCHAR(50)    NOT NULL UNIQUE,
    preco_mensal DECIMAL(10,2) NOT NULL,
    ativo       BOOLEAN        NOT NULL DEFAULT TRUE
);

CREATE TABLE assinatura (
    id          BIGSERIAL   PRIMARY KEY,
    usuario_id  BIGINT      NOT NULL REFERENCES usuario(id),
    plano_id    BIGINT      NOT NULL REFERENCES plano(id),
    status      VARCHAR(20) NOT NULL DEFAULT 'ATIVA',
    inicio      DATE        NOT NULL DEFAULT CURRENT_DATE,
    fim         DATE,

    CONSTRAINT uk_assinatura_usuario UNIQUE (usuario_id)
);

INSERT INTO plano (nome, preco_mensal) VALUES
    ('Básico',       29.90),
    ('Profissional',  79.90),
    ('Enterprise',   199.90);
```

#### Migrations condicionais com Liquibase

```yaml
databaseChangeLog:
  - changeSet:
      id: feature-assinatura-schema
      author: equipe
      labels: "feature-assinatura"
      changes:
        - createTable:
            tableName: assinatura
            # ...
```

```yaml
# Executar apenas changeSets da feature
spring:
  liquibase:
    label-filter: "!feature-assinatura"  # Exclui a feature por enquanto
    # Quando ativar: remover o filtro ou trocar para "feature-assinatura"
```

---

## 10. Integração com CI/CD

### GitHub Actions — Validação de migrations

```yaml
# .github/workflows/validate-migrations.yml
name: Validate Database Migrations

on:
  pull_request:
    paths:
      - 'src/main/resources/db/**'

jobs:
  validate-flyway:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven

      - name: Validar e aplicar migrations
        run: ./mvnw flyway:validate flyway:info -B
        env:
          FLYWAY_URL: jdbc:postgresql://localhost:5432/testdb
          FLYWAY_USER: test
          FLYWAY_PASSWORD: test

      - name: Executar testes de integração
        run: ./mvnw verify -B
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: test
          SPRING_DATASOURCE_PASSWORD: test
```

### Pipeline com staging antes de produção

```yaml
# .github/workflows/deploy.yml
name: Deploy com Migrations

on:
  push:
    branches: [main]

jobs:
  migrate-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Aplicar migrations em Staging
        run: ./mvnw flyway:migrate -B
        env:
          FLYWAY_URL: ${{ secrets.STAGING_DB_URL }}
          FLYWAY_USER: ${{ secrets.STAGING_DB_USER }}
          FLYWAY_PASSWORD: ${{ secrets.STAGING_DB_PASSWORD }}

  migrate-production:
    needs: migrate-staging
    runs-on: ubuntu-latest
    environment: production    # Requer aprovação manual
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21

      - name: Backup do banco
        run: |
          pg_dump ${{ secrets.PROD_DB_URL }} > backup-$(date +%Y%m%d%H%M%S).sql

      - name: Aplicar migrations em Produção
        run: ./mvnw flyway:migrate -B
        env:
          FLYWAY_URL: ${{ secrets.PROD_DB_URL }}
          FLYWAY_USER: ${{ secrets.PROD_DB_USER }}
          FLYWAY_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
```

### 10.3 Monitoramento de Migrations

Em produção, é fundamental monitorar se as migrations foram aplicadas corretamente e detectar problemas rapidamente.

#### Spring Boot Actuator — Health Check de Migrations

O Actuator detecta automaticamente Flyway e Liquibase e adiciona verificações de saúde:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, flyway, liquibase
  endpoint:
    health:
      show-details: always
  health:
    db:
      enabled: true
```

```
GET /actuator/health

{
  "status": "UP",
  "components": {
    "flyway": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "total": 12,
        "successful": 12,
        "pending": 0
      }
    }
  }
}
```

```
GET /actuator/flyway

{
  "contexts": {
    "application": {
      "flywayBeans": {
        "flyway": {
          "migrations": [
            {
              "type": "SQL",
              "version": "1",
              "description": "criar tabela usuario",
              "state": "SUCCESS",
              "installedOn": "2025-06-01T10:00:00Z",
              "executionTime": 45
            },
            ...
          ]
        }
      }
    }
  }
}
```

#### Métricas com Micrometer

```java
@Component
public class FlywayMetrics {

    private final Flyway flyway;
    private final MeterRegistry registry;

    public FlywayMetrics(Flyway flyway, MeterRegistry registry) {
        this.flyway = flyway;
        this.registry = registry;

        Gauge.builder("flyway.migrations.applied", this::contarAplicadas)
            .description("Total de migrations aplicadas com sucesso")
            .register(registry);

        Gauge.builder("flyway.migrations.pending", this::contarPendentes)
            .description("Migrations pendentes de aplicação")
            .tag("alert", "true")
            .register(registry);

        Gauge.builder("flyway.migrations.failed", this::contarFalhas)
            .description("Migrations que falharam")
            .tag("alert", "critical")
            .register(registry);
    }

    private double contarAplicadas() {
        return Arrays.stream(flyway.info().applied())
            .filter(m -> m.getState() == MigrationState.SUCCESS)
            .count();
    }

    private double contarPendentes() {
        return flyway.info().pending().length;
    }

    private double contarFalhas() {
        return Arrays.stream(flyway.info().applied())
            .filter(m -> m.getState() == MigrationState.FAILED)
            .count();
    }
}
```

#### Alerta quando migrations falham — Evento Spring

```java
@Component
public class FlywayStartupValidator implements ApplicationListener<ApplicationReadyEvent> {

    private static final Logger log = LoggerFactory.getLogger(FlywayStartupValidator.class);

    private final Flyway flyway;

    public FlywayStartupValidator(Flyway flyway) {
        this.flyway = flyway;
    }

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        MigrationInfo[] pendentes = flyway.info().pending();
        MigrationInfo[] falhas = Arrays.stream(flyway.info().applied())
            .filter(m -> m.getState() == MigrationState.FAILED)
            .toArray(MigrationInfo[]::new);

        if (falhas.length > 0) {
            log.error("ALERTA: {} migration(s) com falha detectada(s)!", falhas.length);
            for (MigrationInfo info : falhas) {
                log.error("  FALHA: V{} - {}", info.getVersion(), info.getDescription());
            }
        }

        if (pendentes.length > 0) {
            log.warn("AVISO: {} migration(s) pendente(s).", pendentes.length);
            for (MigrationInfo info : pendentes) {
                log.warn("  PENDENTE: V{} - {}", info.getVersion(), info.getDescription());
            }
        }

        if (falhas.length == 0 && pendentes.length == 0) {
            log.info("Todas as {} migrations aplicadas com sucesso.",
                flyway.info().applied().length);
        }
    }
}
```

#### Prometheus + Grafana — Exemplo de alerta

```yaml
# prometheus/alerts.yml
groups:
  - name: database-migrations
    rules:
      - alert: MigrationsPendentes
        expr: flyway_migrations_pending > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Migrations pendentes em {{ $labels.instance }}"
          description: "{{ $value }} migration(s) não foram aplicadas."

      - alert: MigrationFalhou
        expr: flyway_migrations_failed > 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Migration falhou em {{ $labels.instance }}"
          description: "{{ $value }} migration(s) falharam. Verificar imediatamente."
```

---

## 11. Cenários Avançados

### 11.1 Multi-tenancy — Schema por tenant

```yaml
# application.yml
app:
  tenants:
    - name: empresa-a
      schema: tenant_empresa_a
    - name: empresa-b
      schema: tenant_empresa_b
```

```java
@Configuration
public class FlywayMultiTenantConfig {

    @Value("${spring.datasource.url}")
    private String url;

    @Value("${spring.datasource.username}")
    private String user;

    @Value("${spring.datasource.password}")
    private String password;

    @Bean
    CommandLineRunner flywayMigrate(
            @Value("${app.tenants[0].schema}") String schema1,
            @Value("${app.tenants[1].schema}") String schema2) {

        return args -> {
            for (String schema : List.of(schema1, schema2)) {
                Flyway.configure()
                    .dataSource(url, user, password)
                    .schemas(schema)
                    .locations("classpath:db/migration")
                    .load()
                    .migrate();
            }
        };
    }
}
```

### 11.2 Múltiplos datasources

```yaml
spring:
  flyway:
    enabled: false  # Desabilita auto-configuração; migrations manuais abaixo

app:
  datasources:
    principal:
      url: jdbc:postgresql://localhost:5432/app_principal
      username: postgres
      password: postgres
      flyway-locations: classpath:db/migration/principal
    relatorios:
      url: jdbc:postgresql://localhost:5432/app_relatorios
      username: postgres
      password: postgres
      flyway-locations: classpath:db/migration/relatorios
```

```java
@Configuration
public class MultiDataSourceFlywayConfig {

    @Bean
    @Primary
    public Flyway flywayPrincipal(
            @Qualifier("principalDataSource") DataSource dataSource) {
        Flyway flyway = Flyway.configure()
            .dataSource(dataSource)
            .locations("classpath:db/migration/principal")
            .load();
        flyway.migrate();
        return flyway;
    }

    @Bean
    public Flyway flywayRelatorios(
            @Qualifier("relatoriosDataSource") DataSource dataSource) {
        Flyway flyway = Flyway.configure()
            .dataSource(dataSource)
            .locations("classpath:db/migration/relatorios")
            .load();
        flyway.migrate();
        return flyway;
    }
}
```

### 11.3 Adicionando Flyway em projeto existente (baseline)

Quando um projeto já tem um banco de dados em produção sem versionamento:

```
PASSO 1: Gerar o schema atual como V1
$ pg_dump --schema-only -f V1__baseline_schema_atual.sql minha_app

PASSO 2: Configurar baseline
spring:
  flyway:
    baseline-on-migrate: true
    baseline-version: 1
    # Flyway marca V1 como "baseline" sem executá-la
    # Novas migrations (V2+) serão executadas normalmente

PASSO 3: Em produção, rodar uma vez
$ ./mvnw flyway:baseline
# Isso cria a tabela flyway_schema_history com V1 como baseline

PASSO 4: Novas migrations
V2__adicionar_nova_funcionalidade.sql  ← Esta será executada
```

### 11.4 Adicionando Liquibase em projeto existente

```bash
# Gerar changelog a partir do banco existente
liquibase --url=jdbc:postgresql://localhost:5432/minha_app \
          --username=postgres --password=postgres \
          generate-changelog \
          --changelog-file=src/main/resources/db/changelog/changes/000-baseline.yaml
```

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/000-baseline.yaml
  # Marca o baseline como já executado em bancos existentes
  - changeSet:
      id: baseline-mark
      author: equipe
      preConditions:
        - onFail: MARK_RAN
        - tableExists:
            tableName: usuario   # Se a tabela já existe, pula o baseline
      changes: []
```

### 11.5 Gerando Migrations a partir de Entities JPA

Criar migrations manualmente para cada entity pode ser trabalhoso. Existem formas de aproveitar o mapeamento JPA que já existe no código para gerar scripts SQL ou changelogs automaticamente — e então revisá-los antes de adicionar ao versionamento.

#### Abordagem 1 — Capturar o DDL gerado pelo Hibernate (rápida, sem dependências extras)

O Hibernate já sabe traduzir suas entities para DDL. Basta pedir que ele exporte os scripts para um arquivo em vez de executá-los no banco:

```yaml
# application-gerar-ddl.yml (profile temporário, NÃO usar em produção)
spring:
  jpa:
    hibernate:
      ddl-auto: none
    properties:
      jakarta.persistence.schema-generation:
        scripts:
          action: create          # "create" gera o schema completo;
                                  # "drop-and-create" gera DROP + CREATE
          create-target: target/generated-schema.sql
          create-source: metadata
```

```bash
# Executar com o profile para gerar o arquivo
./mvnw spring-boot:run -Dspring-boot.run.profiles=gerar-ddl

# O arquivo target/generated-schema.sql conterá todo o DDL:
#   CREATE TABLE usuario (...)
#   CREATE TABLE pedido (...)
#   CREATE INDEX ...
#   ALTER TABLE ... ADD CONSTRAINT ...
```

O arquivo gerado serve como base — copie os trechos relevantes para suas migrations Flyway ou Liquibase, revisando e ajustando conforme necessário.

> **Dica:** Para gerar sem iniciar toda a aplicação, use um teste unitário dedicado (veja Abordagem 2).

#### Abordagem 2 — SchemaExport via teste unitário (sem subir a aplicação)

Crie um teste que usa o `SchemaExport` do Hibernate para gerar o DDL diretamente em arquivo:

```java
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.tool.hbm2ddl.SchemaExport;
import org.hibernate.tool.schema.TargetType;
import org.junit.jupiter.api.Test;

import java.util.EnumSet;

class GerarSchemaTest {

    @Test
    void gerarDDLCompleto() {
        var registry = new StandardServiceRegistryBuilder()
            .applySetting("hibernate.dialect",
                          "org.hibernate.dialect.PostgreSQLDialect")
            .applySetting("hibernate.hbm2ddl.auto", "none")
            .build();

        var metadata = new MetadataSources(registry)
            .addAnnotatedClass(Usuario.class)
            .addAnnotatedClass(Produto.class)
            .addAnnotatedClass(Pedido.class)
            .addAnnotatedClass(ItemPedido.class)
            .addAnnotatedClass(Endereco.class)
            .buildMetadata();

        var schemaExport = new SchemaExport();
        schemaExport.setFormat(true);
        schemaExport.setDelimiter(";");
        schemaExport.setOutputFile("target/schema-export.sql");
        schemaExport.createOnly(EnumSet.of(TargetType.SCRIPT), metadata);

        System.out.println("Schema exportado para target/schema-export.sql");
    }
}
```

Resultado em `target/schema-export.sql`:

```sql
create table usuario (
    id bigserial not null,
    nome varchar(100) not null,
    email varchar(150) not null unique,
    senha_hash varchar(255) not null,
    cpf varchar(14),
    ativo boolean not null default true,
    criado_em timestamp not null,
    primary key (id)
);

create table pedido (
    id bigserial not null,
    usuario_id bigint not null,
    status varchar(20) not null default 'PENDENTE',
    total decimal(12,2) not null default 0,
    criado_em timestamp not null,
    atualizado_em timestamp,
    primary key (id)
);

alter table if exists pedido
    add constraint fk_pedido_usuario
    foreign key (usuario_id) references usuario;
-- ...
```

#### Abordagem 3 — Hibernate `ddl-auto=create` + capturar log SQL (a mais rápida para prototipar)

Para momentos de prototipação rápida, deixe o Hibernate gerar o schema e capture o SQL pelo log:

```yaml
# application-dev.yml (somente para desenvolvimento!)
spring:
  datasource:
    url: jdbc:h2:mem:prototipo  # Banco em memória para prototipar
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create    # Gera o schema automaticamente
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.orm.jdbc.bind: TRACE
```

```
# No console, o Hibernate imprime todo o DDL:
Hibernate:
    create table usuario (
        id bigint generated by default as identity,
        nome varchar(100) not null,
        email varchar(150) not null unique,
        ...
    )
```

Copie os statements do log, adapte o dialeto SQL para o banco de produção (ex: H2 → PostgreSQL), e transforme em migration.

> **Atenção:** Esta abordagem é útil para rascunho rápido, mas o SQL gerado pode usar tipos do H2 que não existem no PostgreSQL. Sempre revise antes de criar a migration final.

#### Abordagem 4 — Liquibase `diff` entre entities e banco (a mais robusta)

O Liquibase consegue comparar o modelo definido pelas entities JPA com o estado atual do banco e gerar automaticamente um changelog com as diferenças. Isso é ideal para criar migrations incrementais:

```xml
<!-- pom.xml — plugin Liquibase com suporte a Hibernate -->
<build>
    <plugins>
        <plugin>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-maven-plugin</artifactId>
            <configuration>
                <!-- Banco de dados atual (referência) -->
                <url>jdbc:postgresql://localhost:5432/minha_app</url>
                <username>postgres</username>
                <password>postgres</password>
                <changeLogFile>
                    src/main/resources/db/changelog/changes/nova-migration.yaml
                </changeLogFile>

                <!-- Model JPA (o que deveria existir) -->
                <referenceUrl>
                    hibernate:spring:com.example.ecommerce.model
                    ?dialect=org.hibernate.dialect.PostgreSQLDialect
                    &amp;hibernate.physical_naming_strategy=
                    org.hibernate.boot.model.naming.CamelCaseToUnderscoresNamingStrategy
                </referenceUrl>
                <referenceDriver>
                    liquibase.ext.hibernate.database.connection.HibernateDriver
                </referenceDriver>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>org.liquibase.ext</groupId>
                    <artifactId>liquibase-hibernate6</artifactId>
                    <version>4.30.0</version>
                </dependency>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-data-jpa</artifactId>
                    <version>${spring-boot.version}</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

```bash
# Gera changelog com as diferenças entre entities e banco
./mvnw liquibase:diff

# Resultado: src/main/resources/db/changelog/changes/nova-migration.yaml
# contém apenas as alterações necessárias para alinhar banco ↔ entities
```

Exemplo de saída gerada automaticamente ao adicionar um campo `dataNascimento` na entity `Usuario`:

```yaml
# Gerado automaticamente pelo liquibase:diff
databaseChangeLog:
  - changeSet:
      id: 1719484200-1
      author: liquibase (generated)
      changes:
        - addColumn:
            tableName: usuario
            columns:
              - column:
                  name: data_nascimento
                  type: DATE
```

#### Abordagem 5 — Script utilitário com Spring Boot (gerar e já salvar como migration)

Um `ApplicationRunner` que gera a migration automaticamente na pasta correta:

```java
@Component
@Profile("gerar-migration")
public class GeradorMigrationRunner implements ApplicationRunner {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        var session = entityManager.unwrap(org.hibernate.Session.class);
        var factory = session.getSessionFactory();
        var metadata = ((org.hibernate.internal.SessionFactoryImpl) factory)
            .getMetamodel();

        // Gera DDL via SchemaExport
        var integrator = new org.hibernate.tool.schema.internal.SchemaCreatorImpl(
            factory
        );

        Path migrationDir = Path.of("src/main/resources/db/migration");
        int proximaVersao = contarMigrationsExistentes(migrationDir) + 1;
        String nomeArquivo = String.format(
            "V%d__schema_gerado_automaticamente.sql", proximaVersao
        );

        Path arquivo = migrationDir.resolve(nomeArquivo);
        System.out.println("Migration gerada em: " + arquivo);
        System.out.println("REVISE o conteúdo antes de commitar!");
        System.exit(0);
    }

    private int contarMigrationsExistentes(Path dir) throws Exception {
        if (!Files.exists(dir)) return 0;
        try (var stream = Files.list(dir)) {
            return (int) stream
                .filter(p -> p.getFileName().toString().startsWith("V"))
                .count();
        }
    }
}
```

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=gerar-migration
```

#### Fluxo recomendado — Entity-first com geração assistida

```
                        ┌────────────────────────────────┐
                        │  1. Criar/alterar Entity JPA   │
                        │     @Entity, @Column, etc.     │
                        └───────────────┬────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────┐
                        │  2. Gerar DDL automaticamente   │
                        │     (qualquer abordagem acima)  │
                        └───────────────┬────────────────┘
                                        │
                                        ▼
                 ┌──────────────────────────────────────────────┐
                 │  3. REVISAR o SQL/YAML gerado                │
                 │     - Ajustar tipos específicos do banco      │
                 │     - Adicionar índices de performance        │
                 │     - Verificar constraints e defaults        │
                 │     - Remover instruções desnecessárias       │
                 │     - Separar em migrations incrementais      │
                 └───────────────────┬──────────────────────────┘
                                     │
                          ┌──────────┴──────────┐
                          ▼                     ▼
                 ┌─────────────────┐   ┌─────────────────┐
                 │  Flyway:        │   │  Liquibase:      │
                 │  V<N>__desc.sql │   │  NNN-desc.yaml   │
                 └────────┬────────┘   └────────┬────────┘
                          │                     │
                          └──────────┬──────────┘
                                     ▼
                        ┌────────────────────────────────┐
                        │  4. Testar com banco real       │
                        │     (Testcontainers / Docker)   │
                        └───────────────┬────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────┐
                        │  5. Commitar entity + migration │
                        │     juntos no mesmo commit      │
                        └────────────────────────────────┘
```

#### Resumo das abordagens

| Abordagem | Esforço | Ideal para | Limitação |
|-----------|:-------:|------------|-----------|
| **Jakarta schema-generation** (yml) | Baixo | Schema completo inicial | Gera tudo de uma vez, sem diff |
| **SchemaExport** (teste) | Baixo | Gerar sem subir a app | Requer listar entities manualmente |
| **ddl-auto=create + log** | Mínimo | Prototipação rápida | SQL pode ter dialeto errado (H2) |
| **Liquibase diff** (JPA ↔ banco) | Médio | Migrations incrementais | Requer plugin extra e banco rodando |
| **ApplicationRunner** | Médio | Automatizar no projeto | Requer manutenção do utilitário |

> **Regra de ouro:** Qualquer abordagem de geração automática produz um **rascunho**. Sempre revise o resultado antes de transformá-lo em migration definitiva. O DDL gerado pelo Hibernate pode omitir índices de performance, usar tipos genéricos, ou não refletir constraints específicas do banco de produção.

### 11.6 Migrations em Microsserviços

Em arquiteturas de microsserviços, cada serviço é dono do seu banco de dados (database-per-service). As migrations acompanham o ciclo de vida de cada serviço individualmente.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MICROSSERVIÇOS                                │
│                                                                      │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐            │
│  │ user-service  │   │ order-service │   │ product-svc  │            │
│  │              │   │              │   │              │            │
│  │ db/migration/│   │ db/migration/│   │ db/migration/│            │
│  │  V1__users   │   │  V1__orders  │   │  V1__products│            │
│  │  V2__roles   │   │  V2__items   │   │  V2__stock   │            │
│  │  V3__...     │   │  V3__...     │   │  V3__...     │            │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘            │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│    ┌─────────┐       ┌─────────┐       ┌─────────┐                │
│    │ user_db │       │ order_db│       │product_db│                │
│    └─────────┘       └─────────┘       └─────────┘                │
│                                                                      │
│  Cada serviço versiona seu schema de forma independente              │
└─────────────────────────────────────────────────────────────────────┘
```

#### Monorepo — Múltiplos módulos com Flyway

```
ecommerce-monorepo/
├── user-service/
│   ├── src/main/resources/db/migration/
│   │   ├── V1__criar_tabela_usuario.sql
│   │   └── V2__criar_tabela_perfil.sql
│   └── src/main/resources/application.yml
├── order-service/
│   ├── src/main/resources/db/migration/
│   │   ├── V1__criar_tabela_pedido.sql
│   │   └── V2__criar_tabela_item.sql
│   └── src/main/resources/application.yml
└── product-service/
    ├── src/main/resources/db/migration/
    └── src/main/resources/application.yml
```

Cada módulo tem suas próprias migrations, conecta em seu próprio banco e versões são independentes entre si.

#### Mudanças de schema que cruzam domínios

Quando uma alteração no serviço A impacta o serviço B (ex: mudança no formato de um evento), as migrations devem ser coordenadas:

```
┌─────────────────────────────────────────────────────────────────┐
│          ESTRATÉGIA: EXPAND–MIGRATE–CONTRACT ENTRE SERVIÇOS      │
│                                                                   │
│  1. Deploy do serviço CONSUMIDOR com suporte ao formato novo E    │
│     antigo (expand)                                               │
│                                                                   │
│  2. Deploy do serviço PRODUTOR com formato novo                   │
│     Migration: adicionar novas colunas/tabelas                    │
│                                                                   │
│  3. Verificar que todos os consumidores processam o formato novo  │
│                                                                   │
│  4. Deploy do serviço CONSUMIDOR removendo suporte ao formato     │
│     antigo (contract)                                             │
└─────────────────────────────────────────────────────────────────┘
```

#### Tabelas compartilhadas — o anti-pattern

```
❌ ANTI-PATTERN: Dois serviços acessam/migram a mesma tabela

  user-service ──┐
                 ├──► tabela "usuario"  ← Quem é dono? Quem migra?
  order-service ─┘

✅ CORRETO: Cada serviço acessa dados de outros via API

  order-service ──► GET /api/usuarios/{id} ──► user-service
                    (ou evento "UsuarioCriado" via mensageria)
```

### 11.7 Migrations em Bancos Distribuídos e Réplicas

Quando o banco de dados tem réplicas de leitura, as migrations devem considerar o tempo de propagação.

#### Topologia típica

```
                   ┌─────────────────┐
                   │   Primary (RW)   │
                   │                  │
  Migrations ────► │  flyway_migrate  │
                   │                  │
                   └───────┬──────┬──┘
                  Replicação│      │Replicação
              (assíncrona)  │      │(assíncrona)
                           │      │
                    ┌──────▼┐  ┌──▼──────┐
                    │Replica│  │ Replica  │
                    │  (RO) │  │  (RO)   │
                    │       │  │         │
                    │ Delay │  │ Delay   │
                    │ ~ms   │  │ ~ms     │
                    └───────┘  └─────────┘
```

#### Cuidados com réplicas

| Cenário | Problema | Solução |
|---------|----------|---------|
| `ADD COLUMN` | Réplica ainda não tem a coluna quando a app lê | Código deve tolerar coluna ausente por alguns segundos |
| `DROP COLUMN` | App pode tentar ler coluna que não existe mais na réplica | Parar de ler a coluna no código **antes** de dropar |
| `CREATE INDEX` | Lock na primary pode pausar replicação | Usar `CREATE INDEX CONCURRENTLY` |
| `ALTER TABLE` longa | Replicação acumula lag | Executar em horário de baixa carga |

#### Flyway — Executar migrations apenas no primary

```yaml
# application.yml
spring:
  datasource:
    # Primary para escrita e migrations
    url: jdbc:postgresql://primary:5432/app
    username: app_user
    password: ${DB_PASSWORD}

  flyway:
    enabled: true
    url: jdbc:postgresql://primary:5432/app   # Sempre no primary
    user: flyway_user                          # Usuário com permissão DDL
    password: ${FLYWAY_PASSWORD}

# Datasource de leitura (réplica) — configurado separadamente
app:
  datasource:
    read:
      url: jdbc:postgresql://replica:5432/app
      username: app_readonly
      password: ${DB_READONLY_PASSWORD}
```

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean("readDataSource")
    @ConfigurationProperties("app.datasource.read")
    public DataSource readDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

#### Verificar replication lag antes de liberar tráfego

```sql
-- PostgreSQL: verificar lag de replicação
SELECT
    client_addr,
    state,
    pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn) AS send_lag_bytes,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS replay_lag_bytes
FROM pg_stat_replication;
```

### 11.8 Migrations em Bancos NoSQL — Mongock

Para projetos que usam MongoDB (ou outros bancos NoSQL), o conceito de migrations também se aplica. **Mongock** é o equivalente do Flyway/Liquibase para MongoDB:

#### Configuração com Spring Boot

```xml
<dependency>
    <groupId>io.mongock</groupId>
    <artifactId>mongock-springboot-v3</artifactId>
    <version>5.5.0</version>
</dependency>
<dependency>
    <groupId>io.mongock</groupId>
    <artifactId>mongodb-springdata-v4-driver</artifactId>
    <version>5.5.0</version>
</dependency>
```

```yaml
# application.yml
mongock:
  migration-scan-package: com.example.migrations
  enabled: true
```

```java
@EnableMongock
@SpringBootApplication
public class Application { }
```

#### ChangeLogs do Mongock

```java
package com.example.migrations;

import io.mongock.api.annotations.ChangeUnit;
import io.mongock.api.annotations.Execution;
import io.mongock.api.annotations.RollbackExecution;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.index.Index;
import org.bson.Document;

@ChangeUnit(id = "001-criar-colecao-usuario", order = "001", author = "equipe")
public class V001CriarColecaoUsuario {

    @Execution
    public void execute(MongoTemplate mongoTemplate) {
        mongoTemplate.createCollection("usuarios");

        mongoTemplate.indexOps("usuarios")
            .ensureIndex(new Index().on("email", Sort.Direction.ASC).unique());

        mongoTemplate.indexOps("usuarios")
            .ensureIndex(new Index().on("criadoEm", Sort.Direction.DESC));
    }

    @RollbackExecution
    public void rollback(MongoTemplate mongoTemplate) {
        mongoTemplate.dropCollection("usuarios");
    }
}
```

```java
@ChangeUnit(id = "002-inserir-perfis", order = "002", author = "equipe")
public class V002InserirPerfis {

    @Execution
    public void execute(MongoTemplate mongoTemplate) {
        List<Document> perfis = List.of(
            new Document("nome", "ADMIN").append("permissoes",
                List.of("CRIAR", "LER", "ATUALIZAR", "DELETAR")),
            new Document("nome", "USUARIO").append("permissoes",
                List.of("LER", "ATUALIZAR")),
            new Document("nome", "MODERADOR").append("permissoes",
                List.of("LER", "ATUALIZAR", "DELETAR"))
        );

        mongoTemplate.getCollection("perfis").insertMany(perfis);
    }

    @RollbackExecution
    public void rollback(MongoTemplate mongoTemplate) {
        mongoTemplate.getCollection("perfis")
            .deleteMany(new Document("nome",
                new Document("$in", List.of("ADMIN", "USUARIO", "MODERADOR"))));
    }
}
```

```java
@ChangeUnit(id = "003-migrar-estrutura-endereco", order = "003", author = "equipe")
public class V003MigrarEstruturaEndereco {

    @Execution
    public void execute(MongoTemplate mongoTemplate) {
        // Migrar campo "endereco" (String) para subdocumento estruturado
        mongoTemplate.getCollection("usuarios").updateMany(
            Filters.type("endereco", BsonType.STRING),
            Updates.combine(
                Updates.rename("endereco", "endereco_antigo"),
                Updates.set("endereco", new Document()
                    .append("logradouro", "")
                    .append("cidade", "")
                    .append("estado", "")
                    .append("cep", ""))
            )
        );
    }

    @RollbackExecution
    public void rollback(MongoTemplate mongoTemplate) {
        mongoTemplate.getCollection("usuarios").updateMany(
            Filters.exists("endereco_antigo"),
            Updates.combine(
                Updates.rename("endereco_antigo", "endereco"),
                Updates.unset("endereco")
            )
        );
    }
}
```

#### Comparação: Migrations SQL vs NoSQL

| Aspecto | Flyway/Liquibase (SQL) | Mongock (NoSQL) |
|---------|------------------------|-----------------|
| **Formato** | SQL ou YAML/XML | Classes Java |
| **Granularidade** | Por arquivo/changeSet | Por `@ChangeUnit` |
| **Rollback** | SQL ou declarativo | Método `@RollbackExecution` |
| **Controle** | Tabela no banco | Coleção `mongockChangeLog` |
| **Schema** | Rígido (DDL) | Flexível (índices, validações) |
| **Quando usar** | Sempre com bancos SQL | Quando precisa de índices, validações ou reestruturação de documentos |

### 11.9 Schema Registry e Evolução de Contratos

Em sistemas que usam eventos (Kafka, RabbitMQ), o schema do banco de dados e o schema dos eventos precisam evoluir de forma coordenada. O **Schema Registry** garante compatibilidade entre produtores e consumidores.

#### O problema

```
┌──────────────────────────────────────────────────────────────┐
│  Sem controle de schema de eventos:                           │
│                                                               │
│  order-service (V2):                                          │
│    Migration V5 adiciona campo "cupom_id" na tabela pedido    │
│    Evento PedidoCriado agora inclui campo "cupomId"           │
│                                                               │
│  notification-service (V1):                                   │
│    Ainda não conhece "cupomId"                                │
│    → Falha ao deserializar o evento? Ignora silenciosamente?  │
│                                                               │
│  ⚠️ Schema do banco e schema do evento precisam evoluir juntos│
└──────────────────────────────────────────────────────────────┘
```

#### Avro + Schema Registry com Spring Boot

```xml
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>
    <version>7.7.1</version>
</dependency>
```

Schema Avro do evento:

```json
{
  "type": "record",
  "name": "PedidoCriado",
  "namespace": "com.example.events",
  "fields": [
    {"name": "pedidoId", "type": "long"},
    {"name": "usuarioId", "type": "long"},
    {"name": "total", "type": "string"},
    {"name": "status", "type": "string"},
    {"name": "cupomId", "type": ["null", "long"], "default": null}
  ]
}
```

#### Compatibilidade de schemas

| Tipo | Regra | Exemplo |
|------|-------|---------|
| **BACKWARD** | Consumidor novo lê dados do produtor antigo | Adicionar campo com default |
| **FORWARD** | Consumidor antigo lê dados do produtor novo | Remover campo opcional |
| **FULL** | Ambas as direções | Adicionar/remover somente campos opcionais |
| **NONE** | Sem validação | Não recomendado |

#### Coordenando migrations de banco e eventos

```
Ordem correta de deploy:

1. Migration do banco (adiciona coluna nullable)
   V10__adicionar_coluna_cupom_id.sql:
     ALTER TABLE pedido ADD COLUMN cupom_id BIGINT;

2. Novo schema Avro registrado (campo cupomId com default null)
   → Schema Registry valida compatibilidade BACKWARD

3. Deploy do CONSUMIDOR (entende o campo novo, ignora se null)
   notification-service agora processa "cupomId"

4. Deploy do PRODUTOR (começa a enviar o campo preenchido)
   order-service popula "cupomId" quando há cupom

5. (Opcional) Migration futura para NOT NULL
   V15__tornar_cupom_id_obrigatorio.sql:
     ALTER TABLE pedido ALTER COLUMN cupom_id SET NOT NULL;
   + Atualizar schema Avro removendo "null" do union type
```

#### Relação entre versionamento de banco e eventos

```
┌────────────────────┐         ┌─────────────────────┐
│   Flyway/Liquibase  │         │   Schema Registry    │
│                    │         │                     │
│  Versiona o schema │         │  Versiona o schema  │
│  do BANCO DE DADOS │         │  dos EVENTOS        │
│                    │         │                     │
│  V1, V2, V3...    │         │  v1, v2, v3...      │
│                    │         │                     │
│  Tabelas, colunas, │         │  Campos do Avro/    │
│  índices, constraints│        │  Protobuf/JSON      │
└─────────┬──────────┘         └──────────┬──────────┘
          │                               │
          └───────────┬───────────────────┘
                      │
              ┌───────▼────────┐
              │   Devem evoluir │
              │  de forma       │
              │  COORDENADA     │
              └────────────────┘
```

> **Dica:** Trate a evolução do schema de eventos com o mesmo cuidado das migrations de banco — nunca faça breaking changes sem um período de compatibilidade.

---

## 12. Erros Comuns e Troubleshooting

### Flyway

| Erro | Causa | Solução |
|------|-------|---------|
| `Migration checksum mismatch` | Migration alterada após ser aplicada | Nunca altere; crie nova migration. Ou use `flyway repair` se foi alteração intencional |
| `Found non-empty schema without metadata table` | Banco existente sem flyway_schema_history | Use `baseline-on-migrate: true` |
| `Detected failed migration` | Migration anterior falhou | `flyway repair` para limpar e corrigir o SQL |
| `Validate failed: Detected resolved migration not applied` | Migration existe localmente mas não no banco | Verifique se a migration foi deletada de outro branch |
| `Migration version not allowed` | Versão duplicada ou fora de ordem | Renumere a migration; use `out-of-order: true` se intencional |

### Liquibase

| Erro | Causa | Solução |
|------|-------|---------|
| `Checksum changed for changeSet` | ChangeSet alterado após aplicação | `clearCheckSums` ou criar novo changeSet |
| `Could not acquire change log lock` | Lock preso (crash anterior) | Delete manual: `DELETE FROM DATABASECHANGELOGLOCK WHERE locked = true` |
| `Precondition failed` | Condição não atendida | Verificar estado do banco; ajustar `onFail` |
| `changeSet already executed` | ID duplicado | Cada par `id + author` deve ser único |
| `Unknown change type` | Tipo não reconhecido no YAML/XML | Verificar ortografia e versão do Liquibase |

### Dicas de debug

```yaml
# Flyway — logs detalhados
logging:
  level:
    org.flywaydb: DEBUG

# Liquibase — logs detalhados
logging:
  level:
    liquibase: DEBUG
```

```java
// Flyway — verificar estado programaticamente
@Autowired
private Flyway flyway;

public void verificarMigrations() {
    MigrationInfo[] pendentes = flyway.info().pending();
    if (pendentes.length > 0) {
        log.warn("Migrations pendentes: {}", pendentes.length);
        for (MigrationInfo info : pendentes) {
            log.warn("  - {} ({})", info.getVersion(), info.getDescription());
        }
    }
}
```

---

> **Referências:**
> - [Flyway Documentation](https://documentation.red-gate.com/fd)
> - [Liquibase Documentation](https://docs.liquibase.com/)
> - [Spring Boot — Database Initialization](https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.schema-creation)
> - [Testcontainers](https://testcontainers.com/)
> - [Mongock — MongoDB Migrations](https://docs.mongock.io/)
> - [Confluent Schema Registry](https://docs.confluent.io/platform/current/schema-registry/)
> - [Spring Boot Actuator — Database Migrations](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html#actuator.endpoints.flyway)
