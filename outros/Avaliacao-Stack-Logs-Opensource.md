# Avaliação de Stack Open-Source para Logs e Observabilidade

> **Objetivo:** Avaliar a viabilidade de criar uma stack de observabilidade alternativa ao ELK (Elasticsearch, Logstash, Kibana), motivada por restrições de licenciamento. Análise de substituição do Logstash por Fluentd/Fluent Bit, do Elasticsearch por Apache Solr, e comparativo com alternativas open-source.

---

## 1. Contexto de Licenciamento

A partir da versão 7.11, a Elastic migrou Elasticsearch, Logstash e Kibana de **Apache 2.0** para **SSPL + Elastic License 2.0** — licenças restritivas que impedem oferecer o software como serviço gerenciado e limitam uso comercial em alguns cenários.

Na versão 8.x houve uma reversão parcial: o Elasticsearch voltou a ser distribuído sob **AGPL-3.0** como opção, mas AGPL ainda impõe obrigações de compartilhar código-fonte de sistemas que interagem via rede.

| Componente | Licença original | Licença atual (8.x) | Restrição principal |
|------------|-----------------|---------------------|---------------------|
| Elasticsearch | Apache 2.0 | SSPL / Elastic License 2.0 / AGPL-3.0 | SSPL impede oferta como serviço; AGPL exige abertura de código |
| Logstash | Apache 2.0 | Elastic License 2.0 | Não pode ser oferecido como serviço gerenciado |
| Kibana | Apache 2.0 | Elastic License 2.0 / AGPL-3.0 | Mesmas restrições do Elasticsearch |

---

## 2. Substituição do Logstash por Fluentd / Fluent Bit

**Viabilidade: alta** — essa substituição é bem estabelecida no mercado.

| Critério | Fluentd | Fluent Bit | Logstash |
|----------|---------|------------|----------|
| **Licença** | Apache 2.0 | Apache 2.0 | Elastic License 2.0 |
| **CNCF** | Graduated | Graduated | N/A |
| **Linguagem** | Ruby + C | C | Java (JRuby) |
| **Consumo de memória** | ~40-100 MB | ~5-15 MB | ~500 MB+ |
| **Plugins** | ~1000+ (comunidade vasta) | ~100 nativos (core) | ~200+ |
| **Transformações complexas** | Sim (filtros, plugins Ruby) | Limitado (Lua scripting) | Sim (filtros, Ruby) |
| **Caso de uso ideal** | Agregador central | Agente leve em containers/edge | Pipeline complexo |

### 2.1. Arquitetura recomendada

Usar **Fluent Bit como agente** (coleta nos nós/containers) + **Fluentd como agregador** (transformação e roteamento central). Essa é a arquitetura mais comum em ambientes Kubernetes.

```
Aplicação → Fluent Bit (agente leve) → Fluentd (agregador) → Backend de armazenamento
```

Se o plano é ter uma **implementação própria do pipeline de ingestão**, Fluent Bit como agente de coleta é a escolha ideal — consome poucos recursos e encaminha os logs para o pipeline customizado via TCP, HTTP ou Kafka.

### 2.2. Exemplo de configuração — Fluent Bit como agente

```ini
# fluent-bit.conf
[SERVICE]
    Flush         5
    Log_Level     info
    Parsers_File  parsers.conf

[INPUT]
    Name          tail
    Path          /var/log/app/*.log
    Tag           app.*
    Parser        json
    Refresh_Interval 5
    Mem_Buf_Limit 10MB

[FILTER]
    Name          modify
    Match         app.*
    Add           service pedidos-service
    Add           environment dev

[FILTER]
    Name          nest
    Match         app.*
    Operation     lift
    Nested_under  mdc

[OUTPUT]
    Name          forward
    Match         app.*
    Host          fluentd-aggregator
    Port          24224
```

### 2.3. Exemplo de configuração — Fluentd como agregador

```xml
<!-- fluentd.conf -->
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<!-- Parsear campos JSON -->
<filter app.**>
  @type parser
  key_name log
  reserve_data true
  <parse>
    @type json
  </parse>
</filter>

<!-- Anonimizar dados sensíveis -->
<filter app.**>
  @type record_transformer
  enable_ruby true
  <record>
    message ${record["message"]&.gsub(/\d{3}\.\d{3}\.\d{3}-\d{2}/, "***.***.***-**") rescue record["message"]}
  </record>
</filter>

<!-- Saída para o backend de armazenamento -->
<match app.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name logs-${tag}-%Y.%m.%d
  <buffer tag, time>
    @type memory
    timekey 1d
    timekey_wait 10m
    flush_interval 5s
    chunk_limit_size 8MB
  </buffer>
</match>
```

### 2.4. docker-compose.yml — Fluent Bit + Fluentd

```yaml
services:
  fluent-bit:
    image: fluent/fluent-bit:3.0
    container_name: fluent-bit
    volumes:
      - ./fluentbit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
      - ./fluentbit/parsers.conf:/fluent-bit/etc/parsers.conf:ro
      - /var/log/app:/var/log/app:ro
    networks: [obs]
    depends_on: [fluentd]

  fluentd:
    image: fluentd:v1.17-1
    container_name: fluentd
    ports:
      - "24224:24224"
    volumes:
      - ./fluentd/fluentd.conf:/fluentd/etc/fluentd.conf:ro
    networks: [obs]
```

---

## 3. Substituição do Elasticsearch por Apache Solr

**Viabilidade: baixa a moderada** — possível tecnicamente, mas com limitações significativas para o caso de uso de logs.

### 3.1. O que funciona

- **Licença:** Apache 2.0, sem restrições.
- **Busca full-text:** Solr é excelente; baseado no mesmo Lucene que o Elasticsearch.
- **Maturidade:** projeto Apache de longa data, estável.
- **Faceting e agregações:** suporta, com sintaxe diferente.
- **SolrCloud:** modo distribuído com sharding e réplicas.

### 3.2. O que não funciona bem para logs

| Aspecto | Elasticsearch | Solr | Impacto |
|---------|--------------|------|---------|
| **Otimização para time-series** | Data streams, ILM nativo, rollover automático | Não tem equivalente nativo; requer implementação manual | Alto |
| **Ingestão em massa** | Bulk API otimizada para logs | Menos otimizado para write-heavy workloads | Alto |
| **Retenção automática** | ILM (hot/warm/cold/delete) | Precisa de scripts externos ou solução custom | Alto |
| **Dashboard nativo** | Kibana (Discover, Lens, Maps) | Banana (descontinuado), Solr Admin UI (básico) | Crítico |
| **Esquema dinâmico** | Schemaless por padrão, ideal para logs heterogêneos | Schema-driven; schemaless possível mas não é o default | Moderado |
| **Integração com Fluentd/Fluent Bit** | Plugin nativo e maduro | Plugin existe mas é menos testado | Moderado |
| **Comunidade de observabilidade** | Enorme (ELK é padrão de facto) | Pequena para este caso de uso | Alto |
| **Alertas sobre logs** | Kibana Rules, ElastAlert | Não tem solução integrada | Crítico |

### 3.3. O problema central

O Solr foi projetado para **busca em catálogos** (e-commerce, documentos, CMS) — cenários de leitura intensiva com dados relativamente estáveis. Logs são um workload oposto: **escrita contínua e massiva**, consultas por janela temporal, retenção com expiração automática. Adaptar o Solr para esse perfil exige construir uma camada significativa de infraestrutura ao redor.

A ausência de uma **interface de visualização** comparável ao Kibana é o ponto mais crítico. O Solr Admin UI é uma ferramenta de administração, não de análise de logs. Seria necessário construir ou adotar um frontend separado (Grafana com plugin, aplicação custom, etc.).

```
Lacunas que precisariam ser cobertas com desenvolvimento próprio:

┌──────────────────────────────────────────────────────────────────┐
│ Solr como backend de logs                                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐   CONSTRUIR   ┌─────────────────────────┐  │
│  │ Retenção / ILM  │ ◄──────────── │ Rotação de collections  │  │
│  │                 │               │ Scripts de expiração     │  │
│  └─────────────────┘               └─────────────────────────┘  │
│                                                                  │
│  ┌─────────────────┐   CONSTRUIR   ┌─────────────────────────┐  │
│  │ Visualização    │ ◄──────────── │ Frontend custom ou       │  │
│  │                 │               │ Grafana + plugin Solr    │  │
│  └─────────────────┘               └─────────────────────────┘  │
│                                                                  │
│  ┌─────────────────┐   CONSTRUIR   ┌─────────────────────────┐  │
│  │ Alertas         │ ◄──────────── │ Sistema de polling +     │  │
│  │                 │               │ notificação              │  │
│  └─────────────────┘               └─────────────────────────┘  │
│                                                                  │
│  ┌─────────────────┐   CONSTRUIR   ┌─────────────────────────┐  │
│  │ Schema dinâmico │ ◄──────────── │ Managed schema +         │  │
│  │ para logs       │               │ dynamic fields config    │  │
│  └─────────────────┘               └─────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Alternativas Open-Source Recomendadas

Considerando a motivação (licenciamento), existem opções mais adequadas que o Solr para substituir o Elasticsearch em cenários de observabilidade:

| Alternativa | Licença | Descrição | Viabilidade |
|-------------|---------|-----------|-------------|
| **OpenSearch + OpenSearch Dashboards** | Apache 2.0 | Fork da Elastic mantido pela AWS. API compatível com Elasticsearch 7.x. Dashboards = fork do Kibana. Drop-in replacement. | **Muito alta** |
| **Quickwit** | AGPL-3.0 | Motor de busca otimizado para logs e traces. Armazenamento em object storage (S3). Escrito em Rust, muito eficiente. | Alta (projeto mais jovem) |
| **ClickHouse** | Apache 2.0 | Banco colunar de alto desempenho. Excelente para log analytics e métricas. Integra com Grafana nativamente. | Alta |
| **Loki + Grafana** | AGPL-3.0 | Leve, indexa apenas labels. Integração nativa com Prometheus e Tempo. | Alta (já documentado) |
| **Zinc / ZincObserve** | Apache 2.0 | Alternativa leve ao Elasticsearch, compatível parcialmente com a API. UI própria. | Moderada (comunidade menor) |
| **Apache Solr** | Apache 2.0 | Busca full-text excelente, mas não projetado para observabilidade. | Baixa para logs |

### 4.1. OpenSearch — Drop-in replacement (recomendado)

O OpenSearch é o fork Apache 2.0 do Elasticsearch 7.10, mantido pela AWS com contribuições da comunidade. Resolve o problema de licenciamento com **esforço mínimo de migração**.

```
Stack ELK (licença restritiva)          Stack OpenSearch (Apache 2.0)
┌──────────────┐                        ┌──────────────────────┐
│ Elasticsearch│  ──── substitui ────►  │ OpenSearch            │
│ Logstash     │  ──── substitui ────►  │ Fluentd / Fluent Bit │
│ Kibana       │  ──── substitui ────►  │ OpenSearch Dashboards │
└──────────────┘                        └──────────────────────┘
```

**Vantagens:**
- API compatível com Elasticsearch 7.x (mesmas queries, mesmos clientes).
- OpenSearch Dashboards é fork do Kibana (mesma interface, mesmos dashboards).
- Plugins nativos de alertas, anomaly detection e observabilidade.
- Migração de dados possível via snapshot/restore.
- Comunidade ativa e crescente.

**docker-compose.yml:**

```yaml
services:
  opensearch:
    image: opensearchproject/opensearch:2.15.0
    container_name: opensearch
    environment:
      - discovery.type=single-node
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
      - DISABLE_SECURITY_PLUGIN=true  # desabilitar em dev
    ports:
      - "9200:9200"
    volumes:
      - opensearch_data:/usr/share/opensearch/data
    networks: [obs]

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2.15.0
    container_name: opensearch-dashboards
    ports:
      - "5601:5601"
    environment:
      - OPENSEARCH_HOSTS=["http://opensearch:9200"]
      - DISABLE_SECURITY_DASHBOARDS_PLUGIN=true
    networks: [obs]
    depends_on: [opensearch]

volumes:
  opensearch_data:

networks:
  obs:
    driver: bridge
```

### 4.2. ClickHouse — Alto desempenho para analytics

O ClickHouse é um banco de dados colunar projetado para consultas analíticas em grandes volumes de dados. Oferece compressão superior e queries extremamente rápidas sobre logs.

**Vantagens:**
- Consultas SQL padrão (menor curva de aprendizado).
- Compressão 10-20x superior ao Elasticsearch para dados de log.
- Ingestão de milhões de eventos por segundo.
- Integração nativa com Grafana via plugin oficial.
- Suporte a TTL nativo para retenção automática.

**Desvantagens:**
- Não é um motor de busca full-text (usa índices invertidos opcionais, não tão flexíveis).
- Requer modelagem de tabelas (schema fixo).
- Dashboards dependem do Grafana (não tem UI própria para análise de logs).

**Exemplo de tabela para logs:**

```sql
CREATE TABLE logs (
    timestamp DateTime64(3),
    service   LowCardinality(String),
    level     LowCardinality(String),
    logger    String,
    message   String,
    trace_id  String,
    span_id   String,
    user_id   String,
    request_id String,
    extra     Map(String, String)
) ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(timestamp)
ORDER BY (service, level, timestamp)
TTL timestamp + INTERVAL 90 DAY DELETE
SETTINGS index_granularity = 8192;
```

### 4.3. Quickwit — Projetado para logs

O Quickwit é um motor de busca projetado especificamente para logs e traces, com armazenamento em object storage (S3, MinIO).

**Vantagens:**
- Busca full-text sobre Tantivy (equivalente Rust do Lucene).
- Armazenamento em object storage — custo até 10x menor que Elasticsearch.
- Compatível com API de busca do Elasticsearch (parcial).
- Ingestão via OTLP (integra com OpenTelemetry).
- Suporte nativo a traces (alternativa ao Jaeger/Tempo).

**Desvantagens:**
- AGPL-3.0 (pode ter restrições dependendo do uso).
- Projeto mais jovem, comunidade menor.
- Sem dashboard próprio (usa Grafana).

---

## 5. Recomendação por cenário

| Cenário | Stack recomendada | Justificativa |
|---------|-------------------|---------------|
| **Máxima compatibilidade com ELK** | OpenSearch + Fluent Bit/Fluentd + OpenSearch Dashboards | Migração quase transparente, mesma API, Apache 2.0 |
| **Performance e custo de armazenamento** | ClickHouse + Fluent Bit/Fluentd + Grafana | Consultas analíticas rápidas, compressão superior, SQL padrão |
| **Já usa Prometheus/Grafana** | Loki + Fluent Bit/Fluentd + Grafana | Stack mais leve, integração nativa com Prometheus e Tempo |
| **Busca full-text + custo baixo de storage** | Quickwit + Fluent Bit/Fluentd + Grafana | Object storage, busca full-text, mas AGPL-3.0 |
| **Busca full-text + Apache 2.0 estrita** | OpenSearch + Fluent Bit/Fluentd + OpenSearch Dashboards | Única opção que combina busca full-text + Apache 2.0 + UI madura |
| **Solr como backend** | Solr + Fluent Bit/Fluentd + Grafana (custom) | Viável mas exige desenvolvimento significativo de ILM, dashboards e alertas |

---

## 6. Viabilidade da stack proposta (Fluent Bit/Fluentd + Solr)

| Aspecto | Avaliação |
|---------|-----------|
| **Coleta (Fluent Bit/Fluentd)** | Excelente — maduro, leve, Apache 2.0 |
| **Armazenamento (Solr)** | Funcional para busca, fraco para operações de log (retenção, time-series, ingestão massiva) |
| **Visualização** | Inexistente — precisaria construir ou integrar solução externa |
| **Alertas** | Inexistente — precisaria construir |
| **Esforço de desenvolvimento** | Alto — equivale a construir um "mini-Kibana" + camada de ILM + pipeline de alertas |
| **Manutenção a longo prazo** | Risco alto de débito técnico |

**Resumo:** a substituição do Logstash por Fluentd/Fluent Bit é sólida e recomendada independente do backend escolhido. Já a substituição do Elasticsearch por Solr resolve o problema de licenciamento mas introduz lacunas significativas em visualização, retenção e alertas que precisariam ser cobertas com desenvolvimento próprio. O **OpenSearch** resolve o mesmo problema de licenciamento sem essas lacunas e com esforço de migração mínimo.
