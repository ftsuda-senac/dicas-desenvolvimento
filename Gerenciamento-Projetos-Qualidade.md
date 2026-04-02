# Gerenciamento de Projetos e Qualidade de Software

> **Objetivo:** Apresentar de forma abrangente os principais padrões, metodologias e frameworks utilizados no gerenciamento de projetos e na garantia de qualidade em desenvolvimento de software: PMI/PMBOK, PMI-ACP/Agile, Scrum, Kanban, Lean, Six Sigma, CMMI, MPS-BR e ITIL.

---

## Sumário

1. [PMI e PMBOK — Gerenciamento Tradicional de Projetos](#1-pmi-e-pmbok--gerenciamento-tradicional-de-projetos)
   - [1.1 O PMI e o PMBOK Guide](#11-o-pmi-e-o-pmbok-guide)
   - [1.2 PMBOK 7ª Edição — Visão Geral](#12-pmbok-7ª-edição--visão-geral)
   - [1.3 Os 12 Princípios de Gerenciamento de Projetos](#13-os-12-princípios-de-gerenciamento-de-projetos)
   - [1.4 Os 8 Domínios de Desempenho](#14-os-8-domínios-de-desempenho)
   - [1.5 Tailoring (Adaptação)](#15-tailoring-adaptação)
   - [1.6 Modelos, Métodos e Artefatos](#16-modelos-métodos-e-artefatos)
   - [1.7 Papéis e Responsabilidades](#17-papéis-e-responsabilidades)
   - [1.8 Grupos de Processos — Legado PMBOK 6](#18-grupos-de-processos--legado-pmbok-6)
   - [1.9 Áreas de Conhecimento — Legado PMBOK 6](#19-áreas-de-conhecimento--legado-pmbok-6)
   - [1.10 Artefatos e Documentos por Fase do Projeto](#110-artefatos-e-documentos-por-fase-do-projeto)
   - [1.11 Documentos de Licitação e Aquisição Pública](#111-documentos-de-licitação-e-aquisição-pública)
2. [Gerenciamento Ágil de Projetos — PMI-ACP](#2-gerenciamento-ágil-de-projetos--pmi-acp)
   - [2.1 O PMI-ACP](#21-o-pmi-acp)
   - [2.2 O Manifesto Ágil](#22-o-manifesto-ágil)
   - [2.3 Domínios do PMI-ACP](#23-domínios-do-pmi-acp)
   - [2.4 Scrum](#24-scrum)
   - [2.5 Extreme Programming (XP)](#25-extreme-programming-xp)
   - [2.6 Lean Software Development](#26-lean-software-development)
   - [2.7 SAFe — Scaled Agile Framework](#27-safe--scaled-agile-framework)
   - [2.8 LeSS — Large Scale Scrum](#28-less--large-scale-scrum)
   - [2.9 Crystal](#29-crystal)
   - [2.10 Feature Driven Development (FDD)](#210-feature-driven-development-fdd)
3. [Kanban](#3-kanban)
   - [3.1 Origem e Conceito](#31-origem-e-conceito)
   - [3.2 Princípios e Práticas do Método Kanban](#32-princípios-e-práticas-do-método-kanban)
   - [3.3 O Quadro Kanban](#33-o-quadro-kanban)
   - [3.4 Métricas e Cadências](#34-métricas-e-cadências)
   - [3.5 Kanban vs. Scrum](#35-kanban-vs-scrum)
   - [3.6 Scrumban](#36-scrumban)
4. [Six Sigma](#4-six-sigma)
   - [4.1 Conceito e Origem](#41-conceito-e-origem)
   - [4.2 DMAIC](#42-dmaic)
   - [4.3 DMADV (DFSS)](#43-dmadv-dfss)
   - [4.4 Papéis no Six Sigma](#44-papéis-no-six-sigma)
   - [4.5 Six Sigma em Desenvolvimento de Software](#45-six-sigma-em-desenvolvimento-de-software)
   - [4.6 Lean Six Sigma](#46-lean-six-sigma)
5. [CMMI — Capability Maturity Model Integration](#5-cmmi--capability-maturity-model-integration)
   - [5.1 Visão Geral](#51-visão-geral)
   - [5.2 Níveis de Maturidade](#52-níveis-de-maturidade)
   - [5.3 Níveis de Capacidade](#53-níveis-de-capacidade)
   - [5.4 Áreas de Processo](#54-áreas-de-processo)
   - [5.5 CMMI v2.0](#55-cmmi-v20)
   - [5.6 CMMI em Organizações de Software](#56-cmmi-em-organizações-de-software)
6. [MPS-BR — Melhoria de Processo do Software Brasileiro](#6-mps-br--melhoria-de-processo-do-software-brasileiro)
   - [6.1 Origem e Objetivo](#61-origem-e-objetivo)
   - [6.2 Níveis de Maturidade do MPS-BR](#62-níveis-de-maturidade-do-mps-br)
   - [6.3 Processos do MPS-BR](#63-processos-do-mps-br)
   - [6.4 MPS-BR vs. CMMI](#64-mps-br-vs-cmmi)
   - [6.5 Implementação Prática](#65-implementação-prática)
7. [ITIL — Information Technology Infrastructure Library](#7-itil--information-technology-infrastructure-library)
   - [7.1 O que é o ITIL](#71-o-que-é-o-itil)
   - [7.2 ITIL 4 — Fundamentos](#72-itil-4--fundamentos)
   - [7.3 Sistema de Valor de Serviço (SVS)](#73-sistema-de-valor-de-serviço-svs)
   - [7.4 Cadeia de Valor de Serviço](#74-cadeia-de-valor-de-serviço)
   - [7.5 As Quatro Dimensões](#75-as-quatro-dimensões)
   - [7.6 Práticas do ITIL 4](#76-práticas-do-itil-4)
   - [7.7 ITIL e DevOps](#77-itil-e-devops)
8. [Comparativo entre Frameworks](#8-comparativo-entre-frameworks)
9. [Escolhendo o Framework Certo](#9-escolhendo-o-framework-certo)

---

## 1. PMI e PMBOK — Gerenciamento Tradicional de Projetos

### 1.1 O PMI e o PMBOK Guide

O **Project Management Institute (PMI)** é a maior associação de gerenciamento de projetos do mundo, fundada em 1969 nos EUA. Produz guias, padrões e certificações reconhecidos internacionalmente.

O **PMBOK Guide** (*A Guide to the Project Management Body of Knowledge*) é o principal padrão do PMI. Atualmente na **7ª edição (2021)**, passou por uma mudança significativa de abordagem: deixou de ser um guia prescritivo baseado em processos e tornou-se um guia baseado em **princípios** e **domínios de desempenho**, reconhecendo a diversidade de abordagens (preditivas, ágeis e híbridas).

```
Evolução do PMBOK:
1ª ed. (1996) → 5 grupos de processo + 9 áreas de conhecimento
6ª ed. (2017) → 5 grupos de processo + 10 áreas de conhecimento + capítulo ágil
7ª ed. (2021) → 12 princípios + 8 domínios de desempenho (abordagem agnóstica)
```

### 1.2 PMBOK 7ª Edição — Visão Geral

A 7ª edição é acompanhada pelo **PMIstandards+**, uma plataforma digital que fornece conteúdo complementar, modelos e artefatos. O guia reconhece explicitamente que não existe uma única metodologia correta — o gerente de projeto deve adaptar a abordagem ao contexto.

**Conceito central — Entrega de Valor:**

```
Sistema de Entrega de Valor
┌─────────────────────────────────────────────────────────────────────┐
│  Portfólio → Programas → Projetos → Produtos/Serviços → Valor       │
│                                                                     │
│  Governança ←──────────────────────────────────────────────────→   │
│  Funções de Apoio (PMO, RH, Finanças, Jurídico...)                 │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.3 Os 12 Princípios de Gerenciamento de Projetos

Os princípios são orientações comportamentais que guiam as decisões do gerente e da equipe. São intencionalmente amplos para serem aplicáveis a qualquer metodologia.

| # | Princípio | Descrição |
|---|-----------|-----------|
| 1 | **Diligência e respeito** | Ser um administrador diligente, respeitoso e cuidadoso; agir com integridade e cuidado com os recursos |
| 2 | **Ambiente colaborativo** | Criar um ambiente de equipe colaborativo que fomente comprometimento e resultados coletivos |
| 3 | **Envolvimento de stakeholders** | Engajar partes interessadas de forma efetiva para contribuir com sucesso e satisfação do projeto |
| 4 | **Foco em valor** | Orientar o projeto para a entrega contínua de valor ao negócio e às partes interessadas |
| 5 | **Pensamento sistêmico** | Reconhecer, avaliar e responder às interações internas e externas do sistema do projeto |
| 6 | **Liderança** | Demonstrar comportamentos de liderança adaptados ao contexto, motivando a equipe |
| 7 | **Tailoring** | Adaptar a abordagem com base no contexto do projeto, não aplicar processos de forma rígida |
| 8 | **Qualidade** | Incorporar qualidade nos processos e nas entregas; não tratar qualidade como atividade final |
| 9 | **Complexidade** | Navegar na complexidade usando conhecimento, experiência e aprendizado contínuo |
| 10 | **Risco** | Otimizar respostas a riscos (ameaças e oportunidades) de forma contínua |
| 11 | **Adaptabilidade e resiliência** | Ser adaptável e resiliente para acomodar mudanças e recuperar-se de adversidades |
| 12 | **Mudança** | Facilitar a mudança para alcançar o estado futuro desejado pelo projeto |

### 1.4 Os 8 Domínios de Desempenho

Os domínios descrevem **áreas de foco** interativas e interdependentes que operam simultaneamente durante todo o projeto.

```
┌──────────────────────────────────────────────────────────┐
│               8 Domínios de Desempenho                   │
│                                                          │
│  1. Stakeholders          5. Trabalho do Projeto         │
│  2. Equipe                6. Entrega                     │
│  3. Abordagem e Ciclo     7. Mensuração                  │
│     de Vida               8. Incerteza                   │
│  4. Planejamento                                         │
└──────────────────────────────────────────────────────────┘
```

**1. Domínio de Stakeholders**
Foca no engajamento contínuo das partes interessadas. O gerente deve identificar, analisar e construir relacionamentos que gerem apoio e minimizem resistências.

- Identificação e análise contínua (poder, interesse, impacto)
- Estratégias diferenciadas por perfil de stakeholder
- Comunicação bidirecional e feedbacks constantes

**2. Domínio de Equipe**
Trata da liderança e do desenvolvimento da equipe de projeto.

- Liderança situacional — adaptar o estilo ao nível de maturidade da equipe
- Cultura de alta performance: confiança, respeito, responsabilidade compartilhada
- Desenvolvimento contínuo de competências técnicas e interpessoais

**3. Domínio de Abordagem e Ciclo de Vida**
Define como o trabalho será realizado: abordagem preditiva, adaptativa (ágil) ou híbrida; e como o projeto está estruturado em fases.

```
Abordagens possíveis:
┌────────────────┬──────────────────────────────────────────────┐
│ Preditiva      │ Escopo bem definido, baixa incerteza         │
│ (Waterfall)    │ Ex: construção civil, projetos regulatórios  │
├────────────────┼──────────────────────────────────────────────┤
│ Adaptativa     │ Escopo emergente, alta mudança               │
│ (Ágil)         │ Ex: desenvolvimento de produto digital       │
├────────────────┼──────────────────────────────────────────────┤
│ Híbrida        │ Combinação conforme as necessidades          │
│                │ Ex: fase de requisitos preditiva + entregas  │
│                │ iterativas                                   │
└────────────────┴──────────────────────────────────────────────┘
```

**4. Domínio de Planejamento**
O planejamento é contínuo e progressivo, não apenas uma atividade inicial. Abrange:

- Estimativas de esforço, custo e cronograma (rolling wave)
- Gerenciamento de escopo e requisitos
- Priorização baseada em valor
- Identificação de dependências e restrições

**5. Domínio do Trabalho do Projeto**
Garante que os processos e o ambiente do projeto funcionem adequadamente para entregar valor.

- Comunicação eficaz
- Gerenciamento de contratos e fornecedores
- Monitoramento de desempenho operacional
- Manutenção dos artefatos do projeto

**6. Domínio de Entrega**
Foca na qualidade e na relevância das entregas geradas pelo projeto.

- Requisitos vinculados a metas de negócio
- Critérios de aceitação claramente definidos
- Gestão de defeitos e dívida técnica
- Alinhamento entre entregas e valor esperado

**7. Domínio de Mensuração**
Avalia o desempenho do projeto para tomada de decisão baseada em dados.

- KPIs e métricas de entrega de valor
- Métricas de qualidade, velocidade e previsibilidade
- Earned Value Management (EVM) em abordagens preditivas
- Burndown/Burnup em abordagens ágeis
- Revisão periódica e ajustes

**8. Domínio de Incerteza**
Trata do gerenciamento de riscos, ambiguidades e complexidades.

- Risco: incerteza com impacto potencial (positivo ou negativo)
- Ambiguidade: falta de clareza sobre requisitos ou contexto
- Complexidade: número de variáveis e interdependências
- Estratégias: evitar, mitigar, transferir, aceitar (ameaças); explorar, melhorar, compartilhar, aceitar (oportunidades)

### 1.5 Tailoring (Adaptação)

Tailoring é o processo deliberado de adaptar a abordagem, os processos, os artefatos e as práticas ao contexto específico de cada projeto. O PMBOK 7 eleva o tailoring a um princípio fundamental.

```
Fatores que influenciam o tailoring:
┌───────────────────┬────────────────────────────────────────────┐
│ Organizacionais   │ Cultura, maturidade, estrutura, portfólio  │
│ Do Projeto        │ Complexidade, tamanho, duração, criticidade│
│ Da Equipe         │ Tamanho, distribuição, experiência         │
│ Do Produto        │ Tipo, tecnologia, inovação, riscos         │
│ Regulatórios      │ Compliance, setor, padrões externos        │
└───────────────────┴────────────────────────────────────────────┘
```

**Processo de Tailoring:**
1. Selecionar a abordagem inicial (preditiva, ágil, híbrida)
2. Identificar restrições e requisitos organizacionais
3. Customizar para o contexto do projeto
4. Implementar e adaptar continuamente
5. Incorporar melhoria contínua

### 1.6 Modelos, Métodos e Artefatos

O PMBOK 7 inclui um conjunto de modelos, métodos e artefatos (MMA) como referência:

**Modelos:**
- Modelo de Comunicação (emissor-receptor)
- Modelo de Motivação (Maslow, Herzberg, McGregor)
- Modelo de Maturidade de Equipe (Forming, Storming, Norming, Performing — Tuckman)
- Modelo de Gestão de Mudanças (Kotter, ADKAR)
- Complexity Model (Cynefin — simples, complicado, complexo, caótico)

**Métodos:**
- Reuniões e Facilitação
- Estimativas (análoga, paramétrica, 3 pontos, planning poker)
- Retrospectivas
- Gerenciamento de Valor Agregado (EVM)
- Análise de Variância

**Artefatos:**
- Termo de Abertura do Projeto (TAP)
- Registro de Stakeholders
- Plano de Gerenciamento do Projeto
- WBS (Estrutura Analítica do Projeto)
- Cronograma
- Registro de Riscos
- Relatórios de Desempenho
- Backlog (em abordagens ágeis)

### 1.7 Papéis e Responsabilidades

**Gerente de Projeto (GP)**
Principal responsável pelo sucesso do projeto. No PMBOK 7, atua como líder servidor, facilitador e integrador — não apenas como controlador.

Competências necessárias (Triângulo de Talentos PMI):
```
         ▲
        / \
       /   \
      / Lide-\
     / rança  \
    /───────────\
   / Habilidades \
  /  de Negócio   \
 /─────────────────\
/ Gestão Técnica de \
/ Projetos          \
└───────────────────┘
```

**Sponsor (Patrocinador)**
Executivo que autoriza o projeto, fornece recursos, remove obstáculos organizacionais e garante o alinhamento estratégico.

**Equipe de Projeto**
Conjunto de profissionais que executam o trabalho do projeto. Inclui especialistas técnicos, analistas, arquitetos, testadores etc.

**Escritório de Projetos (PMO)**
Estrutura que padroniza processos de gerenciamento, oferece suporte e governa o portfólio de projetos. Pode ser:
- **Apoio**: fornece templates e treinamento
- **Controle**: exige conformidade com processos
- **Diretivo**: gerencia diretamente os projetos

### 1.8 Grupos de Processos — Legado PMBOK 6

Embora o PMBOK 7 tenha migrado para o modelo de domínios, os **5 grupos de processos** da 6ª edição ainda são referência amplamente utilizada em certificações e organizações.

```
Iniciação → Planejamento → Execução → Monitoramento/Controle → Encerramento
    ↑                           ↑              ↓
    └───────────────────────────┴──────────────┘
                    (iterativo)
```

| Grupo | Atividades Típicas |
|-------|--------------------|
| **Iniciação** | TAP, identificação de stakeholders, autorização formal |
| **Planejamento** | Definição de escopo, cronograma, orçamento, riscos, planos de gerenciamento |
| **Execução** | Gerenciar equipe, conduzir aquisições, gerenciar comunicações, implementar mudanças |
| **Monitoramento/Controle** | Controle de cronograma, custos, escopo, qualidade, riscos |
| **Encerramento** | Aceitar entregas, documentar lições aprendidas, encerrar contratos |

### 1.9 Áreas de Conhecimento — Legado PMBOK 6

As 10 áreas de conhecimento (PMBOK 6) organizam o conhecimento por disciplina técnica:

| # | Área de Conhecimento | Foco Principal |
|---|---------------------|---------------|
| 1 | **Integração** | Coordenação de todos os planos e processos; gestão de mudanças |
| 2 | **Escopo** | Definir o que está e o que não está no projeto (WBS) |
| 3 | **Cronograma** | Sequenciamento, estimativas, caminho crítico (CPM/PERT) |
| 4 | **Custos** | Orçamento, controle de custos, EVM |
| 5 | **Qualidade** | Garantia e controle de qualidade; métricas e inspeções |
| 6 | **Recursos** | Equipe, equipamentos, materiais; desenvolvimento de times |
| 7 | **Comunicações** | Plano de comunicação; relatórios; reuniões |
| 8 | **Riscos** | Identificação, análise qualitativa/quantitativa, resposta |
| 9 | **Aquisições** | Contratos, fornecedores, RFP, RFI, SOW |
| 10 | **Stakeholders** | Identificação, análise e engajamento de partes interessadas |

### 1.10 Artefatos e Documentos por Fase do Projeto

Os artefatos do PMBOK são produzidos ao longo das fases do projeto para registrar decisões, comunicar o estado, orientar a execução e servir de evidência de conformidade. A tabela a seguir usa a estrutura de **5 grupos de processos** (legado PMBOK 6), que ainda é a referência mais utilizada na prática e nas certificações.

> **Nota:** O PMBOK 7 não prescreve artefatos obrigatórios — a lista abaixo representa o conjunto canônico consolidado pelo mercado. O tailoring define quais documentos fazem sentido para cada projeto.

---

#### Fase 1 — Iniciação

**Objetivo:** Autorizar formalmente o projeto ou fase; identificar as principais partes interessadas.

```
Entradas                 Atividades              Saídas (Artefatos)
─────────────────────────────────────────────────────────────────────
Business Case            Reunião de kick-off      Termo de Abertura
Estudo de Viabilidade    Análise de stakeholders  Registro de Stakeholders
Contratos/SOW            Avaliação de riscos      Premissas e Restrições
                         iniciais                 (documento inicial)
```

| Artefato | Sigla | Descrição | Autor Típico |
|----------|-------|-----------|--------------|
| **Termo de Abertura do Projeto** | TAP / Project Charter | Documento que autoriza formalmente o projeto. Define o gerente, objetivos de alto nível, premissas, restrições, orçamento preliminar, stakeholders-chave e critérios de sucesso. Sem ele o projeto não existe formalmente. | Sponsor / PMO |
| **Registro de Stakeholders** | — | Lista de todas as partes interessadas com nome, cargo, organização, papel no projeto, nível de interesse, nível de influência, expectativas e estratégia de engajamento. | Gerente de Projeto |
| **Business Case** | — | Justificativa de negócio que demonstra por que o projeto é necessário. Contém análise de ROI, custo-benefício, alternativas consideradas e alinhamento estratégico. Precede o TAP. | Sponsor / Analista |
| **Premissas e Restrições (iniciais)** | — | Lista documentada de premissas adotadas (fatos assumidos como verdadeiros) e restrições (limites que não podem ser ultrapassados). Alimentam o planejamento. | Gerente de Projeto |

**Exemplo de estrutura do TAP (Termo de Abertura):**

```
TERMO DE ABERTURA DO PROJETO
════════════════════════════════════════════════════════════
Projeto:          Sistema de Gestão de Pedidos v2.0
Data:             2026-04-01
Gerente do Projeto: Ana Silva
Sponsor:          Carlos Mendes (Diretor de TI)
────────────────────────────────────────────────────────────
1. JUSTIFICATIVA
   O sistema atual não suporta o volume de pedidos projetado
   para 2026 (+40%), causando lentidão e perda de vendas.

2. OBJETIVOS
   - Desenvolver nova versão suportando 10.000 pedidos/dia
   - Reduzir tempo de processamento de pedidos em 60%
   - Integrar com os 3 ERPs da empresa

3. ESCOPO DE ALTO NÍVEL
   Inclui: módulos de pedido, estoque e faturamento
   Exclui: módulo de RH e folha de pagamento

4. MARCOS PRINCIPAIS
   - Kickoff:         2026-04-15
   - Entrega Alpha:   2026-07-01
   - Go-Live:         2026-10-01

5. ORÇAMENTO ESTIMADO: R$ 850.000

6. PREMISSAS
   - APIs do ERP legado estarão disponíveis para integração
   - Equipe de 6 desenvolvedores alocados full-time

7. RESTRIÇÕES
   - Data de go-live imutável (prazo regulatório)
   - Stack tecnológica: Java / Spring Boot

8. RISCOS INICIAIS
   - Risco de indisponibilidade das APIs do ERP (Alto)
   - Turnover da equipe durante o projeto (Médio)

9. APROVAÇÕES
   Sponsor: ________________  Data: ____
   GP:      ________________  Data: ____
════════════════════════════════════════════════════════════
```

---

#### Fase 2 — Planejamento

**Objetivo:** Definir o caminho completo para alcançar os objetivos do projeto. É a fase com maior volume de artefatos, pois os planos subsidiários de cada área de conhecimento são elaborados aqui.

```
Plano de Gerenciamento do Projeto
        │
        ├── Plano de Gerenciamento do Escopo
        │       ├── Enunciado do Escopo
        │       └── WBS + Dicionário da WBS
        ├── Plano de Gerenciamento do Cronograma
        │       └── Cronograma do Projeto
        ├── Plano de Gerenciamento dos Custos
        │       └── Linha de Base dos Custos (Orçamento)
        ├── Plano de Gerenciamento da Qualidade
        ├── Plano de Gerenciamento dos Recursos
        │       └── Matriz RACI
        ├── Plano de Gerenciamento das Comunicações
        ├── Plano de Gerenciamento dos Riscos
        │       └── Registro de Riscos
        ├── Plano de Gerenciamento das Aquisições
        └── Plano de Engajamento de Stakeholders
```

| Artefato | Sigla | Descrição |
|----------|-------|-----------|
| **Plano de Gerenciamento do Projeto** | PGP | Documento mestre que integra todos os planos subsidiários. Define como o projeto será executado, monitorado e encerrado. É a referência central de governança. |
| **Enunciado do Escopo do Projeto** | EEP | Descreve em detalhe o que está incluído e excluído no projeto. Inclui entregas, critérios de aceitação, premissas e restrições validadas. Base para controle de escopo. |
| **Estrutura Analítica do Projeto** | WBS / EAP | Decomposição hierárquica de todo o trabalho do projeto em pacotes de trabalho (work packages) gerenciáveis. Cada nó da WBS representa uma entrega, não uma atividade. |
| **Dicionário da WBS** | — | Descreve cada elemento da WBS: identificador, descrição, critérios de aceitação, estimativas de custo e prazo, recursos necessários e marcos associados. |
| **Cronograma do Projeto** | — | Lista de atividades com dependências, estimativas de duração, datas de início/fim, recursos alocados e caminho crítico. Gerado por ferramentas como MS Project ou equivalente. |
| **Linha de Base do Cronograma** | — | Versão aprovada do cronograma, congelada como referência para medição de desempenho. Alterações exigem controle formal de mudanças. |
| **Estimativa de Custos** | — | Previsão quantitativa dos custos de cada atividade ou pacote de trabalho. Usa técnicas como analogia, paramétrica, bottom-up ou três pontos (PERT). |
| **Linha de Base dos Custos** | — | Orçamento aprovado, distribuído ao longo do tempo (curva S). Base para o cálculo do EVM (Earned Value Management). |
| **Plano de Gerenciamento da Qualidade** | — | Define padrões de qualidade, critérios de aceitação, métricas, ferramentas (revisões, testes, auditorias) e responsabilidades de garantia e controle da qualidade. |
| **Métricas de Qualidade** | — | Indicadores mensuráveis de qualidade: cobertura de testes, taxa de defeitos, MTBF, tempo de resposta de sistema, índice de satisfação etc. |
| **Plano de Gerenciamento dos Recursos** | — | Define como os recursos humanos e físicos serão adquiridos, gerenciados e liberados. Inclui organograma do projeto, descrições de papéis e responsabilidades. |
| **Matriz de Responsabilidades** | RACI | Vincula cada atividade/entrega a papéis. RACI = Responsible (executa), Accountable (responde), Consulted (consultado), Informed (informado). |
| **Plano de Gerenciamento das Comunicações** | — | Define quem recebe qual informação, em qual formato, com qual frequência e por qual canal. Inclui matriz de comunicação. |
| **Registro de Stakeholders (atualizado)** | — | Versão expandida com estratégia de engajamento, nível de engajamento atual vs. desejado (matriz de avaliação de engajamento). |
| **Plano de Gerenciamento dos Riscos** | — | Define como os riscos serão identificados, analisados, tratados e monitorados. Estabelece categorias de risco (RBS), escalas de probabilidade e impacto, limites de tolerância. |
| **Registro de Riscos** | — | Inventário de todos os riscos identificados com: descrição, causa, probabilidade, impacto, pontuação (P×I), estratégia de resposta, responsável e status. |
| **Plano de Resposta aos Riscos** | — | Detalhamento das estratégias para cada risco: evitar, mitigar, transferir ou aceitar (ameaças); explorar, melhorar, compartilhar ou aceitar (oportunidades). |
| **Plano de Gerenciamento das Aquisições** | — | Define quando e como comprar produtos/serviços externos, tipos de contrato, critérios de seleção de fornecedores e responsabilidades. |
| **Plano de Engajamento de Stakeholders** | — | Estratégias detalhadas para engajar cada grupo de stakeholders, mantê-los informados e gerenciar expectativas ao longo do projeto. |
| **Registro de Premissas** | — | Lista formalizada de premissas, com a respectiva análise: se a premissa for falsa, qual é o impacto? Alimenta o registro de riscos. |

**Earned Value Management (EVM) — métricas de linha de base:**

```
Linha de Base de Custos (BAC = Budget at Completion):
Define o orçamento total aprovado distribuído no tempo (curva S).

          Custo
            │       /── Curva S (BAC)
            │      /
            │     /
            │    /
            │   /
            │  /
            └──────────────────────── Tempo
               Início           Fim

Indicadores EVM calculados na execução:
PV  (Planned Value)   = Valor orçado do trabalho planejado
EV  (Earned Value)    = Valor orçado do trabalho realizado
AC  (Actual Cost)     = Custo real incorrido

SV  = EV - PV  (Schedule Variance: + adiantado, - atrasado)
CV  = EV - AC  (Cost Variance:     + abaixo do orçamento, - acima)
SPI = EV / PV  (Schedule Performance Index: > 1 = adiantado)
CPI = EV / AC  (Cost Performance Index:     > 1 = eficiente)
EAC = BAC / CPI (Estimate at Completion)
```

**Exemplo de Matriz RACI para desenvolvimento de software:**

```
Atividade / Entrega        Dev  Arq  QA   GP   PO   Sponsor
────────────────────────────────────────────────────────────
Levantamento de Requisitos  C    C    C    A    R    I
Arquitetura Técnica         C    R    C    I    A    I
Desenvolvimento de Feature  R    C    C    I    I    –
Code Review                 C    A    C    I    –    –
Testes de Aceitação         I    I    R    A    C    I
Deploy em Produção          R    C    C    A    I    I
Relatório de Status         I    I    I    R    I    A
────────────────────────────────────────────────────────────
R=Responsible  A=Accountable  C=Consulted  I=Informed
```

---

#### Fase 3 — Execução

**Objetivo:** Coordenar pessoas e recursos para realizar o trabalho definido no Plano de Gerenciamento do Projeto.

| Artefato | Sigla | Descrição |
|----------|-------|-----------|
| **Entregas do Projeto** | — | Produtos, serviços ou resultados gerados durante a execução. Em projetos de software: incrementos de código, módulos, APIs, integrações, documentação técnica etc. |
| **Dados de Desempenho do Trabalho** | — | Dados brutos coletados durante a execução: horas trabalhadas, tarefas concluídas, defeitos encontrados, custo incorrido, reuniões realizadas. |
| **Solicitações de Mudança** | SCM | Documento formal para solicitar alterações no escopo, cronograma, custo, qualidade ou qualquer linha de base aprovada. Toda mudança deve ser solicitada formalmente. |
| **Registro de Problemas (Issue Log)** | — | Registro de problemas em andamento que precisam de resolução: descrição, data de identificação, responsável, prioridade, status e data de resolução. |
| **Registro de Lições Aprendidas** | — | Captura contínua (não apenas no encerramento) de conhecimentos sobre o que funcionou e o que não funcionou, para uso imediato e em projetos futuros. |
| **Atas de Reunião** | — | Registros formais de reuniões: pauta, participantes, decisões tomadas, encaminhamentos e responsáveis. Fundamentais para rastreabilidade de decisões. |
| **Comunicações do Projeto** | — | Relatórios, e-mails, apresentações e demais comunicações produzidas e distribuídas conforme o Plano de Comunicações. |
| **Relatório de Desempenho** | — | Comunicação periódica do estado do projeto para stakeholders: progresso, marcos, desvios, riscos ativos, problemas e próximos passos. |
| **Relatório de Status** | — | Versão mais frequente (semanal/quinzenal) do relatório de desempenho, focando em período recente: o que foi feito, impedimentos, próximos passos. |
| **Plano de Gerenciamento do Projeto (atualizado)** | — | Revisões periódicas dos planos subsidiários com base em mudanças aprovadas e novas informações obtidas durante a execução. |
| **Avaliação de Desempenho da Equipe** | — | Documentação de avaliações periódicas dos membros da equipe, feedback, reconhecimentos e planos de melhoria individual. |
| **Documentação de Aquisições** | — | Contratos assinados, ordens de compra, pedidos de proposta (RFP/RFI/RFQ), avaliações de fornecedores e registros de correspondência com fornecedores. |

**Exemplo de Relatório de Status:**

```
RELATÓRIO DE STATUS — SEMANA 12
════════════════════════════════════════════════════════════
Projeto: Sistema de Gestão de Pedidos v2.0
Período: 2026-06-23 a 2026-06-27
Status Geral: 🟡 ATENÇÃO

INDICADORES EVM:
  SPI: 0,92 (leve atraso)    CPI: 1,05 (dentro do orçamento)
  EV: R$ 280.000             AC: R$ 266.000    PV: R$ 304.000

O QUE FOI FEITO:
  ✓ Módulo de Pedidos — desenvolvimento concluído (100%)
  ✓ 47 testes de integração implementados
  ✓ Reunião de revisão com stakeholders realizada

IMPEDIMENTOS:
  ⚠ API do ERP Legado instável — bloqueando módulo de Estoque
    Ação: Reunião agendada com equipe do ERP (2026-07-01)
    Responsável: Ana Silva

PRÓXIMA SEMANA:
  - Iniciar módulo de Faturamento
  - Resolver bloqueio da API do ERP
  - Revisão de segurança com time de InfoSec

RISCOS ATIVOS:
  R-003: Instabilidade API ERP (Prob: Alta / Impacto: Alto)
════════════════════════════════════════════════════════════
```

---

#### Fase 4 — Monitoramento e Controle

**Objetivo:** Rastrear, revisar e regular o progresso e o desempenho do projeto; identificar áreas que requerem mudanças e iniciar as correspondentes.

> Esta fase ocorre **em paralelo com todas as outras fases** — é contínua, não sequencial.

| Artefato | Sigla | Descrição |
|----------|-------|-----------|
| **Informações sobre o Desempenho do Trabalho** | — | Dados de desempenho analisados e contextualizados: variações de cronograma e custo (SV, CV, SPI, CPI), status das entregas, índice de defeitos, status dos riscos. |
| **Relatórios de Desempenho do Trabalho** | — | Consolidação das informações de desempenho para distribuição às partes interessadas. Inclui análise de tendências, previsões (EAC, ETC) e recomendações. |
| **Solicitações de Mudança Aprovadas** | — | Mudanças formalmente analisadas pelo CCB (*Change Control Board*) e aprovadas, com impactos avaliados em escopo, prazo, custo e qualidade. |
| **Registro de Mudanças (Change Log)** | — | Histórico de todas as solicitações de mudança: número, descrição, data, solicitante, status (pendente/aprovada/rejeitada/implementada) e impacto. |
| **Previsões de Cronograma** | — | Estimativa revisada das datas de término das atividades e marcos com base no desempenho atual (SPI, tendências). |
| **Previsões de Custos** | — | EAC (*Estimate at Completion*) e ETC (*Estimate to Complete*) recalculados com base no CPI e no trabalho restante. |
| **Relatório de Qualidade** | — | Resultado das auditorias de qualidade, inspeções, revisões de código e testes. Registra não-conformidades encontradas e ações corretivas tomadas. |
| **Resultados de Testes e Avaliações** | — | Relatórios de testes unitários, integração, sistema, aceitação (UAT) e desempenho. Inclui taxa de defeitos, cobertura, casos aprovados/reprovados. |
| **Relatório de Riscos** | — | Atualização periódica do status dos riscos identificados: novos riscos, riscos mitigados, riscos materializados, eficácia das respostas implementadas. |
| **Registro de Riscos (atualizado)** | — | Versão revisada do registro, com status atual de cada risco, resposta implementada e resultado. |
| **Relatório de Aquisições** | — | Status dos contratos em vigor: conformidade do fornecedor com os termos, desempenho, pagamentos realizados e pendências. |
| **Registro de Controle de Qualidade** | — | Resultados detalhados das medições de qualidade realizadas, incluindo dados de inspeção e resultados de testes comparados com as métricas planejadas. |

**Fluxo de Controle de Mudanças:**

```
Solicitação de Mudança
         │
         ▼
  Registro no Change Log
         │
         ▼
  Análise de Impacto
  (Escopo / Prazo / Custo / Qualidade / Riscos)
         │
         ▼
  Revisão pelo CCB
  (Change Control Board)
         │
    ┌────┴────┐
    │         │
 Aprovada   Rejeitada
    │           │
    ▼           ▼
Atualizar     Notificar
Linhas de     solicitante
Base +        e registrar
Planos        motivo
    │
    ▼
Implementar
    │
    ▼
Verificar e
Encerrar SCM
```

**Métricas de Controle para Projetos de Software:**

```
Métricas de Cronograma:
  SPI (Schedule Performance Index) = EV / PV
  SPI > 1,0 → adiantado
  SPI = 0,9 → 10% atrasado
  Variância de Cronograma (SV) = EV - PV

Métricas de Custo:
  CPI (Cost Performance Index) = EV / AC
  CPI > 1,0 → abaixo do orçamento (eficiente)
  CPI = 0,85 → gastando 18% a mais que o planejado
  EAC = BAC / CPI  (previsão de custo final)

Métricas de Qualidade em Software:
  Taxa de Defeitos  = Defeitos / KLOC (mil linhas de código)
  Cobertura de Testes (code coverage) = % linhas cobertas
  Defeitos Escapados = bugs em produção / total de bugs
  Densidade de Defeitos por Módulo
```

---

#### Fase 5 — Encerramento

**Objetivo:** Finalizar formalmente o projeto ou fase, entregar o produto ao cliente, liberar recursos e documentar lições aprendidas.

| Artefato | Sigla | Descrição |
|----------|-------|-----------|
| **Termo de Aceite das Entregas** | — | Documento assinado pelo cliente/sponsor confirmando que as entregas foram recebidas, avaliadas e aceitas conforme os critérios de aceitação definidos. |
| **Relatório Final do Projeto** | — | Sumário executivo do projeto: objetivos alcançados, desempenho de prazo e custo (EVM final), qualidade das entregas, riscos materializados, lições aprendidas e recomendações. |
| **Documento de Lições Aprendidas (consolidado)** | — | Compilação estruturada de todas as lições aprendidas durante o projeto. Inclui o que funcionou bem (repetir), o que não funcionou (evitar) e sugestões de melhoria de processo. |
| **Plano de Transição / Handover** | — | Descreve como o produto/sistema será transferido para operação: treinamento de usuários, transferência de documentação técnica, suporte pós-implementação e runbooks. |
| **Documentação Técnica Final** | — | Conjunto de documentos técnicos entregues junto ao produto: manual do usuário, manual do sistema, arquitetura final, configurações de ambiente, procedimentos de operação. |
| **Avaliação de Desempenho dos Fornecedores** | — | Avaliação formal dos fornecedores que participaram do projeto: qualidade das entregas, conformidade contratual, relacionamento e recomendação para projetos futuros. |
| **Encerramento de Contratos** | — | Documentação formal do encerramento de todos os contratos com fornecedores: confirmação de entregas, liquidação de pagamentos, distrato e arquivo do contrato. |
| **Atualização dos Ativos de Processo** | — | Incorporação de templates, checklists, estimativas históricas, dados de desempenho e lições aprendidas ao repositório organizacional (biblioteca de ativos de processo / PMO). |
| **Comunicado de Encerramento** | — | Comunicação formal às partes interessadas anunciando o encerramento do projeto, destacando resultados alcançados, agradecimentos e próximos passos (ex: operação do produto). |

**Conteúdo típico do Relatório Final:**

```
RELATÓRIO FINAL DO PROJETO
════════════════════════════════════════════════════════════
1. RESUMO EXECUTIVO
   Projeto concluído em 2026-10-01 com todas as entregas
   principais aprovadas. Custo final: R$ 820.000 (3,5%
   abaixo do orçamento de R$ 850.000). Prazo: dentro do
   cronograma aprovado.

2. OBJETIVOS ALCANÇADOS
   ✓ Sistema suporta 12.000 pedidos/dia (meta: 10.000)
   ✓ Tempo de processamento reduzido em 68% (meta: 60%)
   ✓ Integração com os 3 ERPs operacional

3. DESEMPENHO
   CPI Final: 1,037  SPI Final: 0,98
   Taxa de Defeitos em Produção (1º mês): 0,3 bugs/KLOC
   Cobertura de Testes: 84%

4. RISCOS MATERIALIZADOS
   R-003: Instabilidade API ERP → impacto de 2 semanas
   Ação efetiva: contorno via queue de mensagens (Kafka)

5. LIÇÕES APRENDIDAS
   (+) Pair programming reduziu defeitos em code review em 40%
   (+) Demo semanal com PO eliminou retrabalho de UI
   (–) Estimativa de integração com ERP subestimada em 35%
   (–) Onboarding de novo dev no mês 5 custou 1 sprint

6. RECOMENDAÇÕES
   - Incorporar buffer de 20% para integrações com legado
   - Adotar contrato de nível de serviço com fornecedor ERP
════════════════════════════════════════════════════════════
```

---

#### Visão Consolidada: Artefatos × Fase × Área de Conhecimento

```
                    INIC  PLAN  EXEC  M&C   ENC
────────────────────────────────────────────────
INTEGRAÇÃO
  Termo de Abertura  ●
  Plano Geral do PJ        ●     ○     ○
  Solicitações de Mudança              ●
  Registro de Mudanças                 ●
  Relatório Final                            ●
  Lições Aprendidas               ●    ●     ●

ESCOPO
  Enunciado do Escopo      ●
  WBS / EAP                ●
  Dicionário da WBS         ●
  Termo de Aceite                             ●

CRONOGRAMA
  Cronograma               ●     ○     ○
  Linha de Base do Crono.  ●
  Previsões de Cronograma              ●

CUSTOS
  Estimativa de Custos     ●
  Linha de Base de Custos  ●
  Previsões de Custos                  ●

QUALIDADE
  Plano de Qualidade       ●
  Métricas de Qualidade    ●     ○     ○
  Relatório de Qualidade              ●
  Resultado de Testes            ●    ●

RECURSOS
  Plano de Recursos        ●
  Matriz RACI              ●
  Avaliação de Equipe            ●

COMUNICAÇÕES
  Plano de Comunicações    ●
  Relatório de Status            ●    ●
  Comunicado de Encerr.                       ●

RISCOS
  Plano de Riscos          ●
  Registro de Riscos       ●     ○     ○
  Relatório de Riscos                  ●

AQUISIÇÕES
  Plano de Aquisições      ●
  Contratos / Docs. Aquis.       ●    ●
  Avaliação de Fornecedores                   ●
  Encerramento de Contratos                   ●

STAKEHOLDERS
  Registro de Stakeholders  ●    ○     ○
  Plano de Engajamento      ●    ○     ○

────────────────────────────────────────────────
● Criado na fase    ○ Atualizado na fase
INIC=Iniciação  PLAN=Planejamento  EXEC=Execução
M&C=Monitoramento/Controle  ENC=Encerramento
```

---

### 1.11 Documentos de Licitação e Aquisição Pública

Em projetos do setor público brasileiro, o Gerenciamento das Aquisições (PMBOK) articula-se diretamente com o processo licitatório, regulado atualmente pela **Lei nº 14.133/2021** (Nova Lei de Licitações e Contratos Administrativos), que substituiu a Lei nº 8.666/93 e a Lei nº 10.520/2002 (Pregão). Também coexistem contratos regidos pela **Lei nº 8.987/95** (concessões) e legislação específica para empresas estatais (**Lei nº 13.303/2016**).

> Em projetos privados os mesmos documentos aparecem com denominação diferente: o Edital equivale ao **RFP** (*Request for Proposal*), o Termo de Referência equivale ao **SOW** (*Statement of Work*), e o Contrato Administrativo equivale ao **Contrato de Prestação de Serviços**.

#### Modalidades de Licitação (Lei 14.133/2021)

| Modalidade | Uso Típico em TI | Critério de Julgamento |
|------------|------------------|------------------------|
| **Pregão** | Aquisição de bens e serviços comuns (licenças, hardware, suporte) | Menor preço ou maior desconto |
| **Concorrência** | Contratos de grande valor; obras complexas; concessões | Menor preço, melhor técnica ou técnica e preço |
| **Concurso** | Trabalhos técnicos, científicos ou artísticos | Melhor técnica |
| **Leilão** | Alienação de bens; concessões | Maior lance |
| **Diálogo Competitivo** | Soluções inovadoras ou de alta complexidade técnica | Técnica e preço |

> O **Pregão Eletrônico** é a modalidade dominante em contratações de TI pelo governo federal, operacionalizado pela plataforma **ComprasGov** (PNCP — Portal Nacional de Contratações Públicas).

#### Fases do Processo Licitatório e seus Documentos

```
Fase Preparatória
      │
      ▼
  Divulgação do Edital
      │
      ▼
  Apresentação de Propostas
      │
      ▼
  Julgamento e Habilitação
      │
      ▼
  Homologação e Adjudicação
      │
      ▼
  Celebração do Contrato
      │
      ▼
  Execução Contratual
      │
      ▼
  Encerramento / Prestação de Contas
```

#### Documentos da Fase Preparatória (Planejamento da Contratação)

Corresponde à fase de **Planejamento das Aquisições** no PMBOK.

| Documento | Descrição |
|-----------|-----------|
| **Documento de Oficialização da Demanda (DOD)** | Solicitação formal da área requisitante ao setor de TI ou de compras, descrevendo a necessidade de negócio. Ponto de partida do processo. |
| **Estudo Técnico Preliminar (ETP)** | Análise técnica que demonstra a viabilidade da contratação, alternativas avaliadas, riscos identificados, mercado existente e estimativa de volume. Obrigatório pela Lei 14.133/2021. |
| **Análise de Riscos** | Identificação e análise dos riscos da contratação (técnicos, operacionais, legais), com estratégias de mitigação. Alimenta o ETP e o Termo de Referência. |
| **Pesquisa de Preços** | Levantamento de preços de mercado para embasar o valor estimado da contratação. Pode ser realizada por cotação direta, pesquisa em sistemas de preços (painel de preços do governo federal), contratos similares anteriores ou referências de mercado. |
| **Termo de Referência (TR)** | Principal documento técnico da licitação de serviços. Equivale ao **SOW** (Statement of Work). Contém: objeto da contratação, justificativa, requisitos técnicos, especificações do serviço, critérios de aceitação, métricas de desempenho (ANS/SLA), modelo de execução, obrigações das partes, prazo, forma de pagamento e valor estimado. |
| **Projeto Básico** | Utilizado em obras e engenharia no lugar do TR. Contém especificações técnicas detalhadas, plantas, orçamento detalhado e cronograma físico-financeiro. |
| **Anteprojeto** | Versão simplificada do Projeto Básico, utilizada no Diálogo Competitivo quando os detalhes técnicos serão definidos com os licitantes. |
| **Autorização de Abertura** | Despacho ou portaria da autoridade competente autorizando o início do processo licitatório. |

**Estrutura típica de um Termo de Referência para serviços de software:**

```
TERMO DE REFERÊNCIA — DESENVOLVIMENTO DE SISTEMA
══════════════════════════════════════════════════════════════
1. OBJETO
   Contratação de empresa especializada para desenvolvimento
   e manutenção do Sistema de Gestão de Processos (SGP).

2. JUSTIFICATIVA E MOTIVAÇÃO
   O sistema atual (legado) não atende aos requisitos da
   Resolução nº XX/2025. O ETP demonstrou inviabilidade
   de adaptação — nova solução é necessária.

3. ESPECIFICAÇÕES TÉCNICAS
   3.1 Requisitos Funcionais (listagem ou referência a anexo)
   3.2 Requisitos Não-Funcionais
       - Disponibilidade: 99,5% em horário comercial
       - Tempo de resposta: < 3s para 95% das requisições
       - Suporte a 500 usuários simultâneos
   3.3 Stack tecnológica exigida / recomendada
   3.4 Integração com sistemas existentes

4. MODELO DE EXECUÇÃO
   4.1 Metodologia de desenvolvimento aceita (Ágil/Scrum)
   4.2 Entregas e marcos (sprints, releases)
   4.3 Ambiente de desenvolvimento, homologação e produção
   4.4 Gestão de mudanças e versionamento

5. ACORDO DE NÍVEL DE SERVIÇO (ANS / SLA)
   5.1 Prazos para correção de defeitos por severidade:
       Crítico (P1): 4 horas / Alto (P2): 8 horas
       Médio (P3): 3 dias úteis / Baixo (P4): 10 dias úteis
   5.2 Disponibilidade mínima garantida
   5.3 Penalidades por descumprimento do ANS (glosas)

6. HABILITAÇÃO TÉCNICA EXIGIDA
   - Atestado de capacidade técnica (sistema similar)
   - Certificações exigidas (ex: ISO 27001, CMMI Nível 3)
   - Qualificação da equipe técnica mínima

7. FORMA DE PRESTAÇÃO DO SERVIÇO
   Unidade de Serviço Técnico (UST) ou Ponto de Função (PF)
   ou Squad Dedicado — justificar a métrica adotada

8. CRITÉRIO DE JULGAMENTO
   Técnica e Preço (peso técnico: 70% / preço: 30%)
   ou Menor Preço (para serviços de menor complexidade)

9. ESTIMATIVA DE VALOR
   Baseada em: Painel de Preços MPOG, SIORG, cotações
   Valor estimado: R$ XXX.XXX (sigiloso até abertura)

10. PRAZO DE VIGÊNCIA
    12 meses, prorrogável até 60 meses (art. 107, Lei 14.133)
══════════════════════════════════════════════════════════════
```

#### Documentos da Fase de Divulgação e Propostas

| Documento | Descrição |
|-----------|-----------|
| **Edital** | Documento público que convoca os interessados e estabelece as regras da licitação. Contém o TR ou Projeto Básico como anexo, critérios de habilitação, julgamento, recursos, penalidades e minuta do contrato. Publicado no PNCP. |
| **Aviso de Licitação** | Comunicado resumido publicado no Diário Oficial e no PNCP anunciando a abertura da licitação, com link para o Edital completo. |
| **Impugnação ao Edital** | Documento apresentado por interessados contestando alguma cláusula do Edital antes do prazo de entrega de propostas. A administração deve responder formalmente. |
| **Pedido de Esclarecimento** | Questão formal de licitante sobre o Edital, respondida publicamente pela administração e incorporada ao processo. |
| **Proposta Técnica** | Documento do licitante descrevendo como executará o objeto: metodologia, equipe, cronograma, experiência comprovada, plano de trabalho e, em alguns casos, protótipo ou prova de conceito. |
| **Proposta Comercial (de Preços)** | Documento do licitante com o preço total e o detalhamento dos custos (planilha de composição de preços). Deve ser compatível com as especificações do TR. |
| **Declarações Obrigatórias** | Conjunto de declarações exigidas dos licitantes: idoneidade, inexistência de impedimentos, cumprimento de requisitos de habilitação, reserva de cargos para pessoas com deficiência, entre outras. |

#### Documentos da Fase de Julgamento e Habilitação

| Documento | Descrição |
|-----------|-----------|
| **Ata da Sessão Pública** | Registro oficial de todo o andamento da sessão de julgamento: participantes, lances, recursos interpostos, classificação final e decisões do pregoeiro/comissão. |
| **Documentos de Habilitação** | Comprovantes exigidos do vencedor: regularidade fiscal e trabalhista (CND, FGTS, INSS, débitos trabalhistas), qualificação técnica (atestados, certidões), qualificação econômico-financeira (balanço, índices contábeis) e habilitação jurídica (contrato social, CNPJ). |
| **Resultado do Julgamento** | Publicação oficial com a classificação final das propostas e o licitante vencedor. |
| **Recurso Administrativo** | Contestação formal de licitante inconformado com a decisão, apresentada dentro do prazo legal. Suspende o processo até decisão da autoridade. |
| **Decisão de Recurso** | Resposta formal da administração ao recurso, mantendo ou reformando a decisão impugnada. |
| **Adjudicação** | Ato pelo qual a administração declara o vencedor e atribui formalmente o objeto da licitação a ele. Praticado pelo pregoeiro (pregão) ou pela autoridade competente. |
| **Homologação** | Ato da autoridade superior ratificando a regularidade do processo e autorizando a contratação. |

#### Documentos da Execução Contratual

Correspondem à fase de **Execução e Monitoramento/Controle das Aquisições** no PMBOK.

| Documento | Descrição |
|-----------|-----------|
| **Contrato Administrativo** | Instrumento jurídico que formaliza a relação entre a administração e o contratado. Deve conter: objeto, prazo, valor, obrigações das partes, ANS/SLA, penalidades, cláusula de rescisão, foro e demais condições. |
| **Nota de Empenho** | Documento contábil que reserva recursos orçamentários para honrar o contrato. Emitida antes do início da execução. Em contratos simples pode substituir o Contrato Administrativo. |
| **Ordem de Serviço (OS)** | Documento que autoriza formalmente o início de uma demanda específica dentro do contrato. Define escopo, prazo, responsável e valor da OS. Essencial em contratos por demanda (UST/Ponto de Função). |
| **Plano de Trabalho** | Documento detalhado apresentado pelo contratado antes do início de cada demanda ou fase, descrevendo atividades, responsáveis, cronograma e recursos. |
| **Relatório de Execução / Progresso** | Documento periódico do contratado descrevendo o andamento dos serviços: atividades realizadas, entregas, horas consumidas e desvios em relação ao planejado. |
| **Relatório de Medição de Serviços** | Comprovação mensal (ou por período) dos serviços efetivamente prestados, com base nas métricas do ANS. Fundamenta o pagamento. |
| **Nota Fiscal / Fatura** | Documento fiscal emitido pelo contratado após a execução dos serviços, vinculado ao Relatório de Medição e à Ordem de Serviço. Inicia o processo de pagamento. |
| **Termo de Recebimento Provisório** | Assinado pelo fiscal técnico ao receber as entregas, indicando que estão aparentemente conformes. Inicia o prazo de verificação definitiva. |
| **Termo de Recebimento Definitivo** | Assinado após verificação técnica completa, atestando a conformidade das entregas com o TR e os critérios de aceitação. Libera o pagamento definitivo. |
| **Registro de Ocorrências / Diário de Obra** | Livro ou registro eletrônico mantido pelo fiscal, documentando ocorrências relevantes, visitas técnicas, irregularidades e providências tomadas. Fundamental como evidência em disputas contratuais. |
| **Notificação / Advertência** | Comunicação formal ao contratado sobre descumprimento de obrigações. Primeiro passo antes da aplicação de penalidades. |
| **Auto de Infração / Aplicação de Penalidade** | Documento formal aplicando sanção ao contratado (advertência, multa, suspensão, inidoneidade), com fundamento legal, descrição da infração e valor. |
| **Termo Aditivo** | Instrumento que altera formalmente o contrato: prorrogação de prazo, acréscimo ou supressão de escopo (limitado a 25% ou 50% do valor original), reequilíbrio econômico-financeiro. |
| **Apostila** | Atualização unilateral do contrato em situações previstas em lei (ex: reajuste de preços), sem necessidade de aditivo formal. |

#### Documentos do Encerramento Contratual

| Documento | Descrição |
|-----------|-----------|
| **Relatório Final de Execução** | Documento do contratado sumarizando tudo que foi entregue no contrato: escopo executado, métricas de ANS atingidas, lições aprendidas, pendências resolvidas. |
| **Termo de Encerramento do Contrato** | Documento formal registrando o encerramento do contrato por conclusão do objeto ou expiração do prazo. Quita obrigações mútuas. |
| **Termo de Rescisão** | Utilizado quando o contrato é encerrado antes do prazo: por acordo mútuo, por inadimplência do contratado, por interesse da administração ou por caso fortuito. |
| **Avaliação do Contratado** | Registro formal do desempenho do fornecedor ao longo do contrato, alimentando o cadastro de fornecedores do órgão e, se publicado, o SICAF (Sistema de Cadastramento Unificado de Fornecedores). |
| **Prestação de Contas / TCU** | Para contratos de maior valor ou financiados com recursos federais, documentação enviada ao Tribunal de Contas da União (TCU) ou ao órgão de controle competente. |

#### Fiscalização Contratual (Lei 14.133/2021)

A Lei 14.133/2021 instituiu a figura do **Gestor** e do **Fiscal** de contrato como papéis distintos e obrigatórios:

```
Gestor de Contrato
├── Responsável pela gestão administrativa do contrato
├── Emite ordens de serviço, acompanha prazos, autoriza pagamentos
├── Coordena os fiscais técnico e administrativo
└── Ponto de contato oficial com o contratado

Fiscal Técnico
├── Verifica a conformidade técnica das entregas
├── Assina Termos de Recebimento Provisório e Definitivo
├── Registra não-conformidades e solicita correções
└── Alimenta o Relatório de Medição de Serviços

Fiscal Administrativo
├── Acompanha obrigações trabalhistas e previdenciárias
├── Verifica recolhimento de FGTS, INSS, IRRF
└── Garante que os funcionários do contratado têm
    direitos trabalhistas em dia
```

#### Correspondência PMBOK × Documentos de Licitação

```
PMBOK — Processos de Aquisição        Documentos Licitatórios
──────────────────────────────────────────────────────────────
Planejar o Gerenciamento das     →  ETP, Pesquisa de Preços,
Aquisições                           Termo de Referência / PB

Conduzir as Aquisições           →  Edital, Aviso de Licitação,
(Solicitar propostas, selecionar)    Propostas Técnica e Comercial,
                                     Ata de Sessão, Julgamento,
                                     Habilitação, Homologação

Controlar as Aquisições          →  Contrato, Nota de Empenho,
(Monitorar desempenho)               OS, Relatórios de Medição,
                                     Termos de Recebimento,
                                     Termo Aditivo, Penalidades

Encerrar as Aquisições           →  Termo de Encerramento,
                                     Avaliação do Contratado,
                                     Prestação de Contas
```

---

## 2. Gerenciamento Ágil de Projetos — PMI-ACP

### 2.1 O PMI-ACP

A certificação **PMI-ACP** (*Agile Certified Practitioner*) foi lançada em 2011 em resposta ao crescimento das metodologias ágeis. Diferente de certificações de framework único (como o CSM do Scrum Alliance), o PMI-ACP abrange um conjunto amplo de práticas ágeis.

**Pré-requisitos para a certificação:**
- 2.000 horas de experiência geral em projetos
- 1.500 horas trabalhando com metodologias ágeis
- 21 horas de treinamento em práticas ágeis

### 2.2 O Manifesto Ágil

Publicado em 2001 por 17 profissionais, o **Manifesto Ágil** estabelece os valores fundamentais do desenvolvimento ágil:

**4 Valores:**
```
Indivíduos e interações    ACIMA DE  processos e ferramentas
Software em funcionamento  ACIMA DE  documentação abrangente
Colaboração com o cliente  ACIMA DE  negociação de contratos
Responder a mudanças       ACIMA DE  seguir um plano
```

> Os itens à direita têm valor, mas os itens à esquerda são valorizados **mais**.

**12 Princípios:**
1. Satisfazer o cliente com entrega contínua e antecipada de software valioso
2. Mudanças de requisitos são bem-vindas, mesmo tardias — agilidade como vantagem competitiva
3. Entrega frequente de software funcionando, de semanas a meses
4. Pessoas de negócio e desenvolvedores devem trabalhar juntos diariamente
5. Construir projetos ao redor de indivíduos motivados; confiar que farão o trabalho
6. Comunicação face a face é o método mais eficiente de transmitir informações
7. Software funcionando é a medida primária de progresso
8. Processos ágeis promovem desenvolvimento sustentável — ritmo constante indefinidamente
9. Atenção contínua à excelência técnica e ao bom design aumenta a agilidade
10. Simplicidade — a arte de maximizar a quantidade de trabalho não realizado — é essencial
11. As melhores arquiteturas emergem de times auto-organizados
12. Em intervalos regulares, a equipe reflete sobre como se tornar mais eficaz e ajusta seu comportamento

### 2.3 Domínios do PMI-ACP

O PMI-ACP organiza o conhecimento em 7 domínios:

| Domínio | Descrição |
|---------|-----------|
| **Mentalidade e Princípios Ágeis** | Valores, princípios, empirismo |
| **Entrega orientada a valor** | Backlog, priorização, MVP, ROI |
| **Engajamento de Stakeholders** | Colaboração, expectativas, comunicação |
| **Desempenho de Equipe** | Autogestão, colaboração, facilitação |
| **Planejamento Adaptativo** | Estimativas ágeis, release planning, roadmap |
| **Detecção e Resolução de Problemas** | Gestão de impedimentos, melhoria contínua |
| **Melhoria Contínua** | Retrospectivas, kaizen, aprendizagem |

### 2.4 Scrum

O Scrum é o framework ágil mais adotado no mundo. Definido no **Scrum Guide** (última versão: 2020, por Ken Schwaber e Jeff Sutherland), é baseado em empirismo e Lean thinking.

**Pilares do Scrum (inspeção, adaptação, transparência):**
```
        Transparência
             │
      ┌──────┴──────┐
      │             │
  Inspeção     Adaptação
```

**Valores do Scrum:**
- Comprometimento (*Commitment*)
- Foco (*Focus*)
- Abertura (*Openness*)
- Respeito (*Respect*)
- Coragem (*Courage*)

#### Papéis do Scrum

**Product Owner (PO)**
- Maximiza o valor do produto
- Gerencia e prioriza o Product Backlog
- Responsável por expressar os itens do backlog claramente
- Único responsável pelo ROI do produto
- Porta-voz dos stakeholders para a equipe

**Scrum Master (SM)**
- Líder servidor — facilita, remove impedimentos, ensina
- Garante que o Scrum seja entendido e praticado
- Protege a equipe de interferências externas
- Facilita os eventos Scrum
- Ajuda a organização a adotar o Scrum

**Developers (Equipe de Desenvolvimento)**
- Auto-organizados e multifuncionais
- Responsáveis por criar o Incremento a cada Sprint
- Definem o "Como" a partir do "O Quê" do PO
- Tamanho recomendado: 3 a 9 pessoas
- Sem hierarquia interna — todos são "Developers"

#### Eventos do Scrum

```
Product Backlog
      │
      ▼
Sprint Planning ──────────────────────────────────────────┐
      │                                                    │
      ▼                                                    │ Sprint
Sprint Backlog                                             │ (1-4 semanas)
      │                                                    │
      ▼                                                    │
Daily Scrum (15 min/dia) ─────────────────────────────────┤
      │                                                    │
      ▼                                                    │
Sprint Review ────────────────────────────────────────────┤
      │                                                    │
      ▼                                                    │
Sprint Retrospective ─────────────────────────────────────┘
      │
      ▼
Incremento potencialmente entregável
```

**Sprint**
- Ciclo de desenvolvimento fixo: 1 a 4 semanas
- Objetivo imutável durante a Sprint
- Não há mudanças que coloquem em risco o Sprint Goal
- Cancelamento apenas pelo PO se o objetivo se tornar obsoleto

**Sprint Planning** (máx. 8h para Sprint de 1 mês)
- Por que esta Sprint é valiosa? → Sprint Goal
- O que pode ser feito nesta Sprint? → Seleção do Backlog
- Como o trabalho selecionado será realizado? → Plano de execução

**Daily Scrum** (15 min, mesma hora, mesmo local)
- Inspeção do progresso em direção ao Sprint Goal
- Adaptação do Sprint Backlog
- Não é reunião de status para gerentes — é da equipe
- Formato livre (não obrigatoriamente 3 perguntas)

**Sprint Review** (máx. 4h para Sprint de 1 mês)
- Inspeção do Incremento com stakeholders
- Feedback sobre o produto entregue
- Atualização do Product Backlog com base no que foi aprendido
- Discussão sobre próximos passos

**Sprint Retrospective** (máx. 3h para Sprint de 1 mês)
- Inspeção de pessoas, interações, processos e ferramentas
- Identificação de melhorias concretas para a próxima Sprint
- O item mais importante vai para o Sprint Backlog
- Cultura de melhoria contínua

#### Artefatos do Scrum

**Product Backlog**
- Lista ordenada de tudo que é necessário no produto
- Gerenciado pelo PO; nunca está completo
- Itens mais detalhados no topo (refinamento contínuo)
- *Commitment*: **Product Goal** — objetivo de longo prazo do produto

**Sprint Backlog**
- Subconjunto do Product Backlog selecionado para a Sprint + plano de entrega
- Pertence à equipe de desenvolvimento
- Atualizado diariamente
- *Commitment*: **Sprint Goal** — por que a Sprint tem valor

**Incremento**
- Soma de todos os itens concluídos na Sprint + incrementos anteriores
- Deve ser utilizável e atender à *Definition of Done* (DoD)
- *Commitment*: **Definition of Done** — critérios de qualidade obrigatórios

#### Definition of Done (DoD) vs. Critérios de Aceitação

```
Definition of Done (DoD):
├── Padrão de qualidade para TODOS os itens
├── Código revisado (code review)
├── Testes unitários passando
├── Cobertura de testes mínima (ex: 80%)
├── Análise estática sem violações críticas
├── Deploy no ambiente de homologação
└── Documentação atualizada

Critérios de Aceitação:
├── Específicos de cada User Story
├── Definem quando a story está "pronta"
├── Escritos antes do desenvolvimento (BDD/Gherkin)
└── Validados pelo PO
```

#### User Stories e Refinamento

```
Formato padrão:
"Como [persona], eu quero [ação] para que [benefício]"

Exemplo:
"Como cliente, eu quero filtrar produtos por categoria
 para que eu encontre mais rapidamente o que preciso."

Critérios de Aceitação (Gherkin):
DADO que estou na página de listagem de produtos
QUANDO seleciono a categoria "Eletrônicos"
ENTÃO vejo apenas produtos da categoria Eletrônicos
E o total de itens exibidos é atualizado
```

**INVEST (critérios para boas User Stories):**
- **I**ndependent — independente de outras stories
- **N**egotiable — detalhes são negociáveis
- **V**aluable — entrega valor ao usuário
- **E**stimable — pode ser estimada pela equipe
- **S**mall — pequena o suficiente para caber em uma Sprint
- **T**estable — possui critérios de aceitação verificáveis

#### Estimativas no Scrum

**Story Points**
Unidade relativa de esforço, complexidade e incerteza. Não é tempo.

**Planning Poker**
```
1. PO apresenta a User Story
2. Cada membro escolhe uma carta (fibonacci: 1, 2, 3, 5, 8, 13, 21, ?)
3. Todos revelam simultaneamente
4. Discuss divergências extremas
5. Re-vota até consenso
```

**Velocidade (Velocity)**
Média de Story Points entregues por Sprint. Usada para release planning:
```
Total de Story Points no Backlog: 200
Velocidade média da equipe: 40 pontos/Sprint
Estimativa de conclusão: ~5 Sprints (~10 semanas com Sprints de 2 semanas)
```

### 2.5 Extreme Programming (XP)

O XP, criado por Kent Beck nos anos 1990, foca em **práticas de engenharia de software** para melhorar a qualidade do código e a capacidade de resposta a mudanças.

**Valores do XP:**
- Comunicação, Simplicidade, Feedback, Coragem, Respeito

**Práticas Técnicas do XP:**

| Prática | Descrição |
|---------|-----------|
| **TDD** | Escrever o teste antes do código de produção |
| **Pair Programming** | Dois desenvolvedores em uma máquina — driver e navigator |
| **Refactoring** | Melhoria contínua do código sem alterar comportamento |
| **Integração Contínua** | Integrar e verificar código múltiplas vezes por dia |
| **Collective Ownership** | Qualquer membro pode modificar qualquer parte do código |
| **Coding Standards** | Convenções de código compartilhadas por toda a equipe |
| **Simple Design** | Fazer o design mais simples que funcione agora |
| **Sustainable Pace** | Ritmo de 40h/semana — sem horas extras crônicas |
| **On-Site Customer** | Representante do cliente disponível o tempo todo |
| **Small Releases** | Entregas frequentes e pequenas para validação rápida |

### 2.6 Lean Software Development

Adaptação dos princípios do **Toyota Production System** para software, pelos irmãos Poppendieck.

**7 Princípios do Lean:**

1. **Eliminar Desperdícios** (*Eliminate Waste*) — código não entregue, funcionalidades desnecessárias, retrabalho, espera, defeitos
2. **Ampliar o Aprendizado** (*Amplify Learning*) — feedback curto, iterações, testes rápidos
3. **Decidir o Mais Tarde Possível** (*Decide as Late as Possible*) — adiar decisões irreversíveis
4. **Entregar o Mais Rápido Possível** (*Deliver as Fast as Possible*) — fluxo contínuo, lotes pequenos
5. **Empoderar a Equipe** (*Empower the Team*) — times auto-organizados, decisões locais
6. **Construir Qualidade** (*Build Integrity In*) — qualidade como processo, não inspeção final
7. **Ver o Todo** (*See the Whole*) — otimizar o sistema inteiro, não subpartes

**Tipos de Desperdício em Software (Muda):**

```
1. Trabalho parcialmente concluído (WIP excessivo)
2. Funcionalidades extras (over-engineering)
3. Reaprendizado (falta de documentação/conhecimento)
4. Handoffs desnecessários (dependências entre times)
5. Atrasos (espera por aprovação, revisão, infraestrutura)
6. Defeitos (bugs, retrabalho)
7. Burocracia (processos sem valor agregado)
```

### 2.7 SAFe — Scaled Agile Framework

O SAFe é um framework para aplicar práticas ágeis em **organizações grandes** com múltiplos times. Criado por Dean Leffingwell.

**Níveis do SAFe (Full SAFe):**

```
Portfolio Level
      │
      ▼
Large Solution Level
      │
      ▼
Program Level (ART — Agile Release Train)
      │
      ▼
Team Level (Scrum, Kanban, XP)
```

**ART — Agile Release Train**
- 50-125 pessoas organizadas em times ágeis (5-12 por time)
- PI (Program Increment) — ciclo de planejamento de 8-12 semanas
- PI Planning — evento de 2 dias com todos os times do ART
- System Demo a cada Sprint; Solution Demo no final do PI

**Conceitos-chave do SAFe:**
- **PI Planning**: cerimônia de planejamento em nível de programa, alinha todos os times
- **Lean Portfolio Management**: conecta estratégia à execução ágil
- **DevSecOps**: pipeline de entrega contínua integrado com segurança
- **Business Agility**: agilidade em toda a organização, não apenas em TI

### 2.8 LeSS — Large Scale Scrum

Alternativa ao SAFe para escalar o Scrum com **menos processos adicionais**. Criado por Craig Larman e Bas Vodde.

- Princípio: mais Scrum, menos framework
- Um Product Backlog, um PO para todo o produto
- Múltiplos times Scrum trabalhando no mesmo produto
- LeSS (2-8 times) e LeSS Huge (8+ times)
- Times Feature — cada time entrega de ponta a ponta

### 2.9 Crystal

Família de metodologias criada por Alistair Cockburn, variando em intensidade de acordo com o **tamanho e criticidade** do projeto.

```
Crystal Clear   → até 8 pessoas, baixa criticidade
Crystal Yellow  → 10-20 pessoas
Crystal Orange  → 20-50 pessoas
Crystal Red     → 50-100 pessoas
Crystal Maroon  → projetos críticos/com risco de vida
```

**Princípios Crystal:**
- Pessoas são o fator mais importante
- Comunicação frequente minimiza documentação
- Foco na entrega de valor, não em processos

### 2.10 Feature Driven Development (FDD)

Metodologia de Peter Coad e Jeff De Luca, orientada a **features** (funcionalidades) pequenas e entregáveis.

**5 Processos do FDD:**
1. Desenvolver o Modelo Global
2. Construir a Lista de Features
3. Planejar por Feature
4. Design por Feature
5. Build por Feature

Features seguem o padrão: `<ação> o <resultado> <objeto>` (ex: "Calcular o total de uma Venda")

---

## 3. Kanban

### 3.1 Origem e Conceito

O **Kanban** (看板 — *quadro visual*) originou-se no sistema de produção da Toyota nos anos 1940. David J. Anderson adaptou o conceito para desenvolvimento de software no início dos anos 2000, criando o **Método Kanban**.

O Kanban não é um processo de desenvolvimento — é um **método de gestão de mudanças** que melhora incrementalmente o fluxo de trabalho existente, tornando o trabalho visível e limitando o trabalho em progresso.

### 3.2 Princípios e Práticas do Método Kanban

**Princípios Fundamentais (Foundation Principles):**

*Princípios de Gestão da Mudança:*
1. Comece com o que você faz agora
2. Busque mudança incremental e evolucionária
3. Inicialmente, respeite papéis, responsabilidades e cargos atuais

*Princípios de Entrega de Serviço:*
4. Foque nas necessidades e expectativas do cliente
5. Gerencie o trabalho; deixe as pessoas se auto-organizarem
6. Evolua políticas regularmente para melhorar resultados

**6 Práticas do Método Kanban:**

| Prática | Descrição |
|---------|-----------|
| **1. Visualizar** | Tornar o trabalho visível no quadro Kanban |
| **2. Limitar WIP** | Work In Progress: número máximo de itens simultâneos por etapa |
| **3. Gerenciar o Fluxo** | Monitorar e otimizar o fluxo contínuo de trabalho |
| **4. Tornar Políticas Explícitas** | Documentar e comunicar as regras do processo |
| **5. Feedback Loops** | Reuniões cadenciadas para inspecionar e adaptar |
| **6. Melhorar Colaborativamente** | Evolução baseada em dados e modelos científicos |

### 3.3 O Quadro Kanban

```
┌──────────────┬──────────────────────────┬──────────────┬──────────────┐
│   Backlog    │    Em Desenvolvimento     │  Em Revisão  │   Pronto     │
│              │         (WIP: 3)          │   (WIP: 2)   │              │
│              ├────────────┬─────────────┤              │              │
│ [Item 7]     │  Em Prog.  │  Aguardando │  [Item 3]    │  [Item 1]    │
│ [Item 8]     │  [Item 4]  │  [Item 5]   │  [Item 6]    │  [Item 2]    │
│ [Item 9]     │  [Item 10] │             │              │              │
│ [Item 11]    │            │             │              │              │
└──────────────┴────────────┴─────────────┴──────────────┴──────────────┘
                         WIP LIMIT: 3                  WIP LIMIT: 2
```

**Elementos de um card Kanban:**
- Descrição do trabalho
- Responsável
- Data de entrada na etapa
- Classe de serviço (urgente, normal, data fixa)
- Indicadores de bloqueio

**Classes de Serviço:**
```
Expedita        → Máxima urgência, bypass de limites WIP, custo de atraso alto
Data Fixa       → Prazo imutável (regulatório, marketing)
Alto Valor      → Prioridade alta, custo de atraso elevado
Padrão          → Fluxo normal, custo de atraso linear
Intangível      → Baixa urgência, dívida técnica, melhorias internas
```

### 3.4 Métricas e Cadências

**Métricas Principais:**

**Lead Time**
Tempo total desde que o pedido foi feito até a entrega ao cliente.
```
Lead Time = Data de Saída - Data de Entrada no Fluxo
```

**Cycle Time**
Tempo desde que o trabalho efetivamente começou até a conclusão.
```
Cycle Time = Data de Conclusão - Data de Início do Trabalho Ativo
```

**Throughput**
Taxa de entrega: quantos itens são concluídos por unidade de tempo.
```
Throughput = Itens Concluídos / Período (ex: semana)
```

**Lei de Little:**
```
WIP = Throughput × Lead Time

Exemplo:
Se o throughput é 5 itens/semana e o WIP é 20 itens:
Lead Time = WIP / Throughput = 20 / 5 = 4 semanas
```
Reduzir o WIP é a forma mais direta de reduzir o Lead Time.

**Diagrama de Fluxo Cumulativo (CFD):**
Visualiza o fluxo de trabalho ao longo do tempo, mostrando acúmulos (gargalos) e tendências de entrega.

**Cadências do Kanban:**
```
Reunião Diária (standup)    → ~15 min — sincronização do time
Revisão de Fila             → Semanal — priorização do backlog
Revisão de Entrega          → Semanal — inspecionar o que foi entregue
Revisão de Operações        → Mensal — saúde do fluxo e métricas
Retrospectiva               → Trimestral — melhorias de processo
Revisão de Risco            → Conforme necessário
```

### 3.5 Kanban vs. Scrum

| Aspecto | Scrum | Kanban |
|---------|-------|--------|
| **Iterações** | Sprints fixas (1-4 semanas) | Fluxo contínuo, sem iterações obrigatórias |
| **Papéis** | PO, SM, Developers | Não prescreve papéis |
| **Mudanças** | Não durante a Sprint | A qualquer momento (respeitando WIP) |
| **WIP** | Implícito (tamanho da Sprint) | Explícito e central |
| **Métricas** | Velocity, Burndown | Lead Time, Cycle Time, Throughput |
| **Estimativas** | Obrigatórias (Story Points) | Opcionais |
| **Retrospectiva** | Obrigatória por Sprint | Cadência opcional |
| **Melhor para** | Projetos com iterações de produto | Suporte, manutenção, fluxo contínuo |

### 3.6 Scrumban

Híbrido que combina a estrutura do Scrum com a visualização e os limites de WIP do Kanban. Útil em transições ou em equipes que precisam de ambas as características.

- Quadro Kanban + eventos do Scrum (planejamento, retrospectiva)
- WIP limits explícitos dentro das Sprints
- Planning on-demand em vez de planejamento por Sprint rígido

---

## 4. Six Sigma

### 4.1 Conceito e Origem

O **Six Sigma** é uma metodologia de melhoria de processos baseada em **dados estatísticos** para reduzir defeitos e variabilidade. Criada pela Motorola nos anos 1980 e popularizada pela General Electric (Jack Welch) nos anos 1990.

**O que significa Sigma:**
O sigma (σ) é o desvio padrão estatístico. "Six Sigma" significa que o processo opera a 6 desvios-padrão da média, resultando em:

```
Nível Six Sigma: 3,4 defeitos por milhão de oportunidades (DPMO)

Comparação de níveis:
┌──────────┬──────────────────────────┬────────────────────────┐
│  Sigma   │  DPMO (Defeitos/Milhão)  │  Yield (% sem defeito) │
├──────────┼──────────────────────────┼────────────────────────┤
│ 1σ       │ 690.000                  │ 31,0%                  │
│ 2σ       │ 308.000                  │ 69,2%                  │
│ 3σ       │ 66.800                   │ 93,3%                  │
│ 4σ       │ 6.210                    │ 99,4%                  │
│ 5σ       │ 233                      │ 99,977%                │
│ 6σ       │ 3,4                      │ 99,9997%               │
└──────────┴──────────────────────────┴────────────────────────┘
```

### 4.2 DMAIC

O **DMAIC** é o ciclo de melhoria do Six Sigma para **processos existentes**:

```
      D           M           A           I           C
  ┌───────┐   ┌───────┐   ┌───────┐   ┌───────┐   ┌───────┐
  │Define │→  │Measure│→  │Analyze│→  │Improve│→  │Control│
  └───────┘   └───────┘   └───────┘   └───────┘   └───────┘
```

**D — Define (Definir)**
- Definir o problema e o escopo do projeto
- Identificar clientes e seus requisitos críticos (CTQ — Critical to Quality)
- Elaborar o Project Charter
- Mapear o processo de alto nível (SIPOC)

*Ferramentas: SIPOC, VOC (Voz do Cliente), CTQ Tree, Project Charter*

```
SIPOC:
Suppliers → Inputs → Process → Outputs → Customers
```

**M — Measure (Medir)**
- Mapear o processo atual em detalhes
- Identificar e coletar dados relevantes
- Calcular a capacidade atual do processo (Sigma Baseline)
- Validar o sistema de medição (MSA — Measurement System Analysis)

*Ferramentas: Fluxogramas, Plano de Coleta de Dados, Cartas de Controle, Cp/Cpk*

**A — Analyze (Analisar)**
- Identificar causas-raiz dos defeitos
- Analisar dados para confirmar hipóteses
- Priorizar causas com maior impacto

*Ferramentas: Diagrama de Ishikawa (Espinha de Peixe), 5 Porquês, Análise de Pareto, FMEA, Regressão, Testes de Hipótese*

```
Diagrama de Ishikawa (6M):
                          Defeito/Problema
                               ↑
Máquinas    Métodos    Mão-de-Obra
    \           |          /
     \          |         /
──────────────────────────────
     /          |         \
    /           |          \
Materiais  Medição    Meio Ambiente
```

**I — Improve (Melhorar)**
- Gerar soluções para as causas-raiz identificadas
- Pilotar e validar as soluções
- Implementar as melhorias

*Ferramentas: Design of Experiments (DOE), Poka-Yoke (à prova de erros), Kaizen, Simulação*

**C — Control (Controlar)**
- Padronizar as melhorias implementadas
- Monitorar o processo para garantir a sustentação dos ganhos
- Criar planos de controle e documentação
- Transferir responsabilidade ao dono do processo

*Ferramentas: Cartas de Controle (CEP), Plano de Controle, SOPs, Visual Management*

### 4.3 DMADV (DFSS)

O **DMADV** (*Design for Six Sigma*) é usado para **criar novos processos ou produtos** que atinjam o nível Six Sigma desde o início:

| Fase | Descrição |
|------|-----------|
| **D** — Define | Definir objetivos, requisitos dos clientes e metas de projeto |
| **M** — Measure | Medir requisitos críticos (CTQ), capacidades do processo, riscos |
| **A** — Analyze | Analisar alternativas de design; identificar o melhor conceito |
| **D** — Design | Desenvolver o design detalhado; otimizar e planejar verificação |
| **V** — Verify | Verificar o design via simulações, pilotos e validação com cliente |

### 4.4 Papéis no Six Sigma

```
Executive Champion (Líder Executivo)
      │
      ▼
  Deployment Champion
      │
      ▼
  Master Black Belt (MBB)
  ├── Mentor de Black Belts
  ├── Desenvolvimento de treinamentos
  └── Consultoria estratégica
      │
      ▼
  Black Belt (BB)
  ├── Líder de projetos Six Sigma full-time
  ├── Conhecimento profundo de estatística
  └── Facilita projetos complexos
      │
      ▼
  Green Belt (GB)
  ├── Trabalha em projetos Six Sigma part-time
  ├── Lidera projetos menores
  └── Suporta Black Belts
      │
      ▼
  Yellow Belt / White Belt
  └── Conhecimento básico; membro de equipes
```

### 4.5 Six Sigma em Desenvolvimento de Software

O Six Sigma pode ser adaptado para medir e melhorar processos de software:

**Definições de Defeito em Software:**
- Bugs por funcionalidade entregue
- Bugs escapados para produção
- Story Points não entregues por Sprint
- Falhas em deploys (Change Failure Rate)

**Métricas de Processo:**
```
DPMO em Desenvolvimento:
Defeitos = bugs encontrados em produção
Oportunidades = linhas de código × pontos de complexidade

Se uma release tem 50 bugs em 500.000 LOC:
DPMO = (50 / 500.000) × 1.000.000 = 100 DPMO ≈ 5σ
```

**Aplicações práticas:**
- Melhoria do processo de code review (reduzir defeitos escapados)
- Otimização do pipeline de CI/CD (reduzir tempo de build/falhas)
- Melhoria de processos de testes (aumentar cobertura, reduzir regressões)
- Redução do Lead Time de entrega de features

### 4.6 Lean Six Sigma

Combinação do **Lean** (foco em eliminar desperdícios e acelerar o fluxo) com o **Six Sigma** (foco em reduzir variabilidade e defeitos). Os dois se complementam:

```
Lean Six Sigma = Eliminação de Desperdícios + Redução de Variação
               = Velocidade + Qualidade
```

O ciclo DMAIC é mantido, com ferramentas Lean integradas (Value Stream Mapping, 5S, Kaizen, Kanban).

---

## 5. CMMI — Capability Maturity Model Integration

### 5.1 Visão Geral

O **CMMI** (*Capability Maturity Model Integration*) é um framework de melhoria de processos desenvolvido pelo **SEI** (*Software Engineering Institute*) da Carnegie Mellon University. Define práticas de processo que as organizações de software devem ter para melhorar a qualidade e previsibilidade de seus produtos.

**Versões:**
- **CMMI v1.3 (2010)**: amplamente adotado globalmente
- **CMMI v2.0 (2018)**: reescrita focada em negócios e resultados, mais flexível e agnóstica a metodologia

**Modelos do CMMI:**
- **CMMI-DEV**: desenvolvimento de produtos e serviços
- **CMMI-SVC**: entrega e gerenciamento de serviços
- **CMMI-ACQ**: aquisição de produtos e serviços
- **CMMI v2.0**: unificou os três modelos anteriores

### 5.2 Níveis de Maturidade

O CMMI define **5 níveis de maturidade** para organizações:

```
Nível 5 — Otimização
    │  Foco em melhoria contínua e inovação
    │  Prevenção de defeitos; gestão quantitativa de mudanças de processo
    │
Nível 4 — Gerenciado Quantitativamente
    │  Processos medidos e controlados estatisticamente
    │  Variabilidade do processo entendida e gerenciada
    │
Nível 3 — Definido
    │  Processos organizacionais padronizados e documentados
    │  Projetos adaptam o processo padrão
    │
Nível 2 — Gerenciado
    │  Processos básicos definidos e seguidos por projeto
    │  Planejamento, rastreamento, revisão e controle de versão
    │
Nível 1 — Inicial
       Processos ad hoc, imprevisíveis
       Sucesso depende de indivíduos heroicos
```

**Progressão típica:**
- Empresas pequenas e novas geralmente estão no **Nível 1**
- Atingir o **Nível 2** exige ~1-2 anos de esforço sustentado
- Cada nível subsequente demanda mais 1-3 anos
- **Nível 3** é o alvo comum para empresas que desenvolvem software crítico
- **Nível 5** é atingido por poucas organizações globalmente

### 5.3 Níveis de Capacidade

Além dos níveis de maturidade (visão organizacional), o CMMI define **níveis de capacidade** por área de processo individual:

| Nível | Nome | Descrição |
|-------|------|-----------|
| 0 | Incompleto | Processo não implementado ou parcialmente implementado |
| 1 | Executado | Processo atinge os objetivos específicos da área de processo |
| 2 | Gerenciado | Processo planejado, monitorado, controlado e revisado |
| 3 | Definido | Processo baseado em processo padrão organizacional (tailoring) |

### 5.4 Áreas de Processo

O CMMI-DEV v1.3 organiza 22 áreas de processo em categorias:

**Gerenciamento de Processos:**
- OPD — Organizational Process Definition
- OPF — Organizational Process Focus
- OPM — Organizational Performance Management (N5)
- OPP — Organizational Process Performance (N4)
- OT  — Organizational Training

**Gerenciamento de Projetos:**
- IPM — Integrated Project Management
- PMC — Project Monitoring and Control
- PP  — Project Planning
- QPM — Quantitative Project Management (N4)
- REQM — Requirements Management
- RSKM — Risk Management
- SAM — Supplier Agreement Management

**Engenharia:**
- PI  — Product Integration
- RD  — Requirements Development
- TS  — Technical Solution
- VAL — Validation
- VER — Verification

**Suporte:**
- CAR — Causal Analysis and Resolution (N5)
- CM  — Configuration Management
- DAR — Decision Analysis and Resolution
- MA  — Measurement and Analysis
- PPQA — Process and Product Quality Assurance

### 5.5 CMMI v2.0

O CMMI v2.0 introduziu mudanças significativas:

**Novas características:**
- Foco em **resultados de negócio**, não apenas em conformidade de processo
- **Práticas de Alta Maturidade** (níveis 4 e 5) integradas em todas as áreas de processo
- **Vista de Capacidade** permite avaliar áreas individualmente
- Suporte explícito a **abordagens ágeis** e DevOps
- **Guias de implementação** mais práticos e orientados a exemplos

**Categorias do CMMI v2.0:**
```
Doing         → Práticas técnicas e de engenharia
Managing      → Gerenciamento de trabalho e projetos
Enabling      → Suporte e infraestrutura de processo
Improving     → Melhoria de processo organizacional
```

### 5.6 CMMI em Organizações de Software

**Implementação do Nível 2 (foco em projetos):**

```
Práticas mínimas do Nível 2:
✓ Plano de projeto formal (escopo, cronograma, estimativas)
✓ Rastreamento de status (relatórios periódicos, revisões)
✓ Gestão de requisitos (rastreabilidade, controle de mudanças)
✓ Controle de configuração (versionamento de artefatos)
✓ Revisões de qualidade (inspeções, auditorias)
✓ Gestão de acordos com fornecedores
```

**Implementação do Nível 3 (foco na organização):**

```
Adicionalmente ao Nível 2:
✓ Processo padrão organizacional definido e documentado
✓ Biblioteca de ativos de processo (templates, checklists)
✓ Programa de treinamento organizacional
✓ Gestão de riscos formal
✓ Revisões de pares (peer reviews) sistemáticas
✓ Validação e verificação formais
```

**Avaliação CMMI:**
Realizada por **Lead Appraisers** certificados usando o método **SCAMPI** (*Standard CMMI Appraisal Method for Process Improvement*):
- **SCAMPI A**: avaliação formal para obtenção de nível (mais rigorosa)
- **SCAMPI B**: avaliação intermediária, menor rigor
- **SCAMPI C**: avaliação inicial, diagnóstico rápido

---

## 6. MPS-BR — Melhoria de Processo do Software Brasileiro

### 6.1 Origem e Objetivo

O **MPS-BR** (*Melhoria de Processo do Software Brasileiro*) é um modelo de qualidade desenvolvido pela **SOFTEX** (*Associação para Promoção da Excelência do Software Brasileiro*) com apoio do **MCT**, **FINEP** e **BID**. Lançado em 2003, foi criado para atender às necessidades específicas das **micro, pequenas e médias empresas (MPMEs) brasileiras**.

**Motivação:**
- O CMMI era caro e complexo para PMEs brasileiras
- Necessidade de um modelo nacional, acessível e certificável
- Fomentar a competitividade da indústria de software brasileira

**Família MPS.BR:**
- **MPS para Software (MPS-SW)**: foco em processos de desenvolvimento de software
- **MPS para Serviços (MPS-SV)**: gestão de serviços de TI
- **MPS para Recursos Humanos (MPS-RH)**: gestão de pessoas em TI

### 6.2 Níveis de Maturidade do MPS-BR

O MPS-BR define **7 níveis** (A a G), o que permite uma progressão mais granular que o CMMI (5 níveis). Isso facilita a implementação em empresas pequenas.

```
Nível A — Otimizado
    │  Inovação e análise de causas organizacional
    │  (equivalente ao CMMI Nível 5)
    │
Nível B — Gerenciado Quantitativamente
    │  Processos medidos e controlados estatisticamente
    │  (equivalente ao CMMI Nível 4)
    │
Nível C — Definido
    │  Processo organizacional padrão; gerência de riscos
    │  (equivalente ao CMMI Nível 3 completo)
    │
Nível D — Largamente Definido
    │  Verificação, validação, desenvolvimento de req.
    │  (equivalente à parte do CMMI Nível 3)
    │
Nível E — Parcialmente Definido
    │  Treinamento, definição e avaliação do processo
    │  (equivalente à parte do CMMI Nível 3)
    │
Nível F — Gerenciado
    │  Medição, garantia de qualidade, gerência de configuração,
    │  aquisição, gerência de portfólio de projetos
    │  (equivalente ao CMMI Nível 2 com mais processos)
    │
Nível G — Parcialmente Gerenciado
       Gerência de projetos, gerência de requisitos
       (subconjunto do CMMI Nível 2)
```

### 6.3 Processos do MPS-BR

| Nível | Processos |
|-------|-----------|
| G | Gerência de Projetos (GPR), Gerência de Requisitos (GRE) |
| F | Garantia da Qualidade (GQA), Gerência de Configuração (GCO), Gerência de Portfólio de Projetos (GPP), Medição (MED), Aquisição (AQU) |
| E | Avaliação e Melhoria do Processo Organizacional (AMP), Definição do Processo Organizacional (DFP), Gerência de Recursos Humanos (GRH) |
| D | Desenvolvimento de Requisitos (DRE), Integração do Produto (ITP), Projeto e Construção do Produto (PCP), Validação (VAL), Verificação (VER) |
| C | Gerência de Decisões (GDE), Gerência de Riscos (GRI) |
| B | Desempenho do Processo Organizacional (DPO), Gerência Quantitativa do Projeto (GQP) |
| A | Análise de Causas e Resolução (ACR), Inovação e Implantação na Organização (IIO) |

**Detalhamento dos processos de maior relevância:**

**Gerência de Projetos (GPR) — Nível G:**
- Planejar o projeto (escopo, cronograma, recursos, riscos)
- Monitorar e controlar o andamento do projeto
- Gerenciar comprometimentos com stakeholders
- Documentar lições aprendidas

**Gerência de Requisitos (GRE) — Nível G:**
- Obter entendimento dos requisitos com as partes interessadas
- Manter rastreabilidade bidirecional dos requisitos
- Gerenciar mudanças nos requisitos
- Garantir alinhamento entre planos e requisitos

**Verificação (VER) — Nível D:**
- Preparar plano e critérios de verificação
- Realizar revisões por pares (*peer reviews*)
- Executar verificação conforme planejado
- Registrar e tratar resultados das verificações

**Gerência de Riscos (GRI) — Nível C:**
- Identificar e categorizar riscos
- Analisar impacto e probabilidade
- Definir e executar estratégias de mitigação
- Monitorar riscos ao longo do projeto

### 6.4 MPS-BR vs. CMMI

| Aspecto | MPS-BR | CMMI |
|---------|--------|------|
| **Origem** | Brasil (SOFTEX/MCT) | EUA (SEI/Carnegie Mellon) |
| **Níveis** | 7 (A a G) | 5 (1 a 5) |
| **Foco** | MPMEs brasileiras | Grandes organizações |
| **Custo** | Menor (subsidiado) | Elevado |
| **Certificação** | Avaliação MPS | SCAMPI (A/B/C) |
| **Reconhecimento** | Nacional + América Latina | Global |
| **Idioma** | Português | Inglês |
| **Compatibilidade** | Compatível com CMMI | Referência para o MPS-BR |

**Equivalência aproximada:**
```
MPS-BR G   ≈  CMMI Nível 2 (parcial)
MPS-BR F   ≈  CMMI Nível 2
MPS-BR E   ≈  CMMI Nível 3 (parcial)
MPS-BR D   ≈  CMMI Nível 3 (parcial)
MPS-BR C   ≈  CMMI Nível 3
MPS-BR B   ≈  CMMI Nível 4
MPS-BR A   ≈  CMMI Nível 5
```

### 6.5 Implementação Prática

**Roteiro típico para empresas brasileiras:**

```
1. Diagnóstico Inicial
   └─ Avaliação SCAMPI C ou diagnóstico MPS-BR (identifica lacunas)

2. Planejamento da Implementação
   ├─ Definir nível-alvo (geralmente G ou F para início)
   ├─ Formar equipe de processo (SEPG — Software Engineering Process Group)
   └─ Obter patrocínio da alta gestão

3. Definição e Implantação dos Processos
   ├─ Documentar processos, templates e procedimentos
   ├─ Treinar equipes
   └─ Executar projetos-piloto

4. Avaliação Oficial
   ├─ Realizada por Instituição Avaliadora (IA) credenciada pela SOFTEX
   └─ Resultados publicados no site da SOFTEX

5. Manutenção e Evolução
   └─ Buscar próximo nível após consolidação
```

**Artefatos tipicamente produzidos:**
- Plano de projeto (PP)
- Plano de garantia da qualidade
- Plano de gerência de configuração
- Relatórios de medição
- Atas de reunião e relatórios de revisão por pares
- Registro de riscos
- Base de lições aprendidas

---

## 7. ITIL — Information Technology Infrastructure Library

### 7.1 O que é o ITIL

O **ITIL** (*Information Technology Infrastructure Library*) é o framework de **gerenciamento de serviços de TI (ITSM)** mais adotado no mundo. Criado pelo governo britânico nos anos 1980 (CCTA/OGC), é atualmente mantido pela **Axelos** (desde 2013) e pela **PeopleCert** (desde 2021).

**Versões:**
- ITIL v1 (1989-1995): biblioteca de livros sobre operações de TI
- ITIL v2 (2000-2006): foco em processos (Service Support e Service Delivery)
- ITIL v3 (2007) / ITIL 2011: ciclo de vida do serviço (5 volumes)
- **ITIL 4 (2019)**: abordagem holística, agnóstica a metodologia, incorpora Agile, Lean e DevOps

### 7.2 ITIL 4 — Fundamentos

O ITIL 4 introduz o **Sistema de Valor de Serviço (SVS)** como modelo central, reconhecendo que a TI deve co-criar valor com o negócio de forma colaborativa e adaptativa.

**Conceitos-chave:**

**Valor** (Value)
O ITIL 4 define valor como o **benefício percebido, utilidade e importância de algo** para o cliente. A TI não entrega valor sozinha — o valor é co-criado com o cliente.

**Serviço**
Meio de entregar valor ao cliente, facilitando os resultados que os clientes querem alcançar sem que eles gerenciem custos e riscos específicos.

**Produto**
Configuração de recursos de uma organização, projetada para oferecer valor a um consumidor.

**Utilidade e Garantia:**
```
Valor = Utilidade (Fitness for Purpose) + Garantia (Fitness for Use)

Utilidade: O serviço faz o que o cliente precisa?
           (remove restrições, suporta desempenho)

Garantia:  O serviço está disponível quando e como o cliente precisa?
           (disponibilidade, capacidade, segurança, continuidade)
```

### 7.3 Sistema de Valor de Serviço (SVS)

O SVS descreve como todos os componentes e atividades de uma organização trabalham juntos para criar valor:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Sistema de Valor de Serviço                  │
│                                                                 │
│  Oportunidade/Demanda → [SVS] → Valor                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Cadeia de Valor de Serviço                 │   │
│  │  Engage → Plan → Design/Transition → Obtain/Build →    │   │
│  │  Deliver/Support → Improve                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐    │
│  │ Princípios  │  │ Governança  │  │ Melhoria Contínua   │    │
│  │ Orientadores│  │             │  │                     │    │
│  └─────────────┘  └─────────────┘  └─────────────────────┘    │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    Práticas (34)                          │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**7 Princípios Orientadores do ITIL 4:**

| # | Princípio | Descrição |
|---|-----------|-----------|
| 1 | **Foco no Valor** | Tudo deve ser vinculado, direta ou indiretamente, à criação de valor |
| 2 | **Comece Onde Você Está** | Não começar do zero sem antes avaliar o que já existe |
| 3 | **Progrida Iterativamente com Feedback** | Pequenas melhorias incrementais com feedback contínuo |
| 4 | **Colabore e Promova Visibilidade** | Trabalho transparente, com as partes certas envolvidas |
| 5 | **Pense e Trabalhe Holisticamente** | Nenhum serviço ou prática funciona isoladamente |
| 6 | **Mantenha a Simplicidade e Praticidade** | Use o mínimo de passos necessários para alcançar o objetivo |
| 7 | **Otimize e Automatize** | Maximizar o valor do trabalho humano; automatizar o que for possível |

### 7.4 Cadeia de Valor de Serviço

A **Cadeia de Valor de Serviço (CVS)** é o modelo central de atividades do ITIL 4. Composta de 6 atividades interconectadas:

```
                    ┌─────────────────────────┐
    Demanda/     →  │         Planejar         │  →  Valor
    Oportunidade    └─────────────────────────┘
                             ↕ ↕ ↕
    ┌──────────┐   ┌──────────────────┐   ┌──────────────┐
    │ Engajar  │   │ Design e         │   │ Entregar e   │
    │          │   │ Transição        │   │ Suportar     │
    └──────────┘   └──────────────────┘   └──────────────┘
                             ↕
    ┌──────────┐   ┌──────────────────┐
    │ Obter/   │   │   Melhorar       │
    │ Construir│   │                  │
    └──────────┘   └──────────────────┘
```

| Atividade | Objetivo |
|-----------|----------|
| **Planejar** | Garantir entendimento compartilhado da visão, direção e melhoria contínua |
| **Melhorar** | Melhoria contínua de produtos, serviços e práticas |
| **Engajar** | Boa compreensão das necessidades das partes interessadas; transparência e relações contínuas |
| **Design e Transição** | Garantir que produtos e serviços atendam às expectativas de qualidade, custo e mercado |
| **Obter/Construir** | Garantir que componentes de serviço estejam disponíveis quando e onde necessário |
| **Entregar e Suportar** | Garantir que serviços sejam entregues e suportados conforme acordado |

**Fluxos de Valor (Value Streams):**
Sequências específicas de atividades da CVS para atender a um cenário. Exemplos:
- Fluxo de resolução de incidentes
- Fluxo de implantação de novo serviço
- Fluxo de atendimento de requisição

### 7.5 As Quatro Dimensões

O ITIL 4 define 4 dimensões que devem ser consideradas em qualquer serviço ou prática:

```
        Organizações e Pessoas
               ↑
               │
               │
Parceiros ─────┼───── Informação e Tecnologia
e Fornecedores │
               │
               ↓
     Fluxos de Valor e Processos

    (Todas cercadas por Fatores Externos: PESTLE)
```

**1. Organizações e Pessoas**
- Cultura, estrutura, papéis e responsabilidades
- Habilidades e competências necessárias
- Modelos de comunicação e colaboração

**2. Informação e Tecnologia**
- Sistemas de informação e bases de dados
- Tecnologias de suporte (IA, cloud, automação)
- Integração e interoperabilidade entre sistemas

**3. Parceiros e Fornecedores**
- Contratos e acordos de serviço
- Estratégia de terceirização
- Gestão de relacionamentos e risco de fornecedores

**4. Fluxos de Valor e Processos**
- Como as atividades são coordenadas para entregar valor
- Processos, procedimentos e fluxos de trabalho
- Identificação e eliminação de desperdícios

**Fatores Externos (PESTLE):**
- Político, Econômico, Social, Tecnológico, Legal, Ambiental

### 7.6 Práticas do ITIL 4

O ITIL 4 substitui os "processos" por **34 práticas**, organizadas em três categorias:

**Práticas Gerais de Gestão (14):**

| Prática | Descrição |
|---------|-----------|
| Gerenciamento de Estratégia | Definir direção e objetivos de longo prazo |
| Gerenciamento de Portfólio | Gestão de investimentos em produtos e serviços |
| Gerenciamento de Arquitetura | Governança da arquitetura técnica e de negócio |
| Gerenciamento Financeiro | Orçamento, contabilidade e cobrança de serviços |
| **Gerenciamento de Força de Trabalho e Talentos** | RH, competências e capacitação |
| **Melhoria Contínua** | Alinhamento das práticas de melhoria com a estratégia |
| **Gerenciamento de Segurança da Informação** | Proteção de informações (CIA Triad) |
| Gerenciamento de Conhecimento | Captura, organização e uso do conhecimento |
| **Medição e Relatórios** | KPIs, dashboards, indicadores de desempenho |
| Gerenciamento de Mudanças Organizacionais | Adoção e absorção de mudanças pela organização |
| Gerenciamento de Projetos | Entrega de projetos dentro de prazo, custo e qualidade |
| **Gerenciamento de Relacionamento** | Estabelecer e nutrir relacionamentos com stakeholders |
| Gerenciamento de Riscos | Identificar, analisar e tratar riscos |
| Gerenciamento de Fornecedores | Contratos e parcerias com fornecedores externos |

**Práticas de Gerenciamento de Serviços (17):**

| Prática | Descrição |
|---------|-----------|
| **Gerenciamento de Disponibilidade** | Garantir disponibilidade conforme acordado |
| Análise de Negócio | Analisar necessidades do negócio e propor soluções |
| **Gerenciamento de Capacidade e Desempenho** | Garantir capacidade suficiente para demanda atual e futura |
| **Gerenciamento de Mudanças** | Maximizar mudanças bem-sucedidas; minimizar riscos |
| **Gerenciamento de Incidentes** | Restaurar serviço o mais rápido possível |
| **Gerenciamento de Ativos de TI** | Inventário e ciclo de vida de ativos |
| **Monitoramento e Gerenciamento de Eventos** | Detectar e responder a eventos automaticamente |
| **Gerenciamento de Problemas** | Eliminar causas-raiz de incidentes recorrentes |
| Gerenciamento de Release | Disponibilizar versões de serviços para uso |
| **Gerenciamento de Catálogo de Serviços** | Fontes de informação sobre serviços disponíveis |
| **Gerenciamento de Configuração** | CMDB — base de dados de ativos e suas relações |
| **Gerenciamento de Continuidade** | Garantir disponibilidade mínima após desastre |
| Design de Serviço | Projetar serviços adequados para atingir valor |
| Service Desk | Ponto único de contato para usuários |
| **Gerenciamento de Nível de Serviço** | SLAs — acordos de nível de serviço |
| Gerenciamento de Requisições | Atender requisições padrão de usuários |
| Validação e Testes de Serviço | Garantir que serviços novos/alterados atendam requisitos |

**Práticas de Gerenciamento Técnico (3):**

| Prática | Descrição |
|---------|-----------|
| Gerenciamento de Implantação | Mover componentes para produção |
| Gerenciamento de Infraestrutura e Plataforma | Infraestrutura de TI e plataformas |
| Desenvolvimento e Gerenciamento de Software | Desenvolvimento, teste e evolução de aplicações |

#### Práticas Mais Críticas para Desenvolvimento de Software

**Gerenciamento de Incidentes:**
```
Incidente = Interrupção não planejada ou redução da qualidade de serviço

Fluxo de Incidentes:
Detecção → Registro → Classificação → Priorização → Diagnóstico
→ Escalação (se necessário) → Resolução → Fechamento

Categorias de prioridade (matriz Impacto × Urgência):
P1 — Crítico: serviço completamente indisponível, alto impacto no negócio
P2 — Alto: degradação significativa
P3 — Médio: impacto moderado
P4 — Baixo: impacto mínimo

SLA de atendimento (exemplo):
P1: Resposta em 15 min, Resolução em 4h
P2: Resposta em 30 min, Resolução em 8h
P3: Resposta em 2h, Resolução em 24h
P4: Resposta em 4h, Resolução em 72h
```

**Gerenciamento de Problemas:**
```
Problema = Causa (ou causa potencial) de um ou mais incidentes

Reativo: pós-incidente → análise de causa raiz (RCA)
Proativo: antes do incidente → análise de tendências, revisão de logs

Ciclo:
Identificação → Controle do Problema (análise RCA) →
Controle de Erros Conhecidos (workarounds) → Eliminação da Causa

Known Error = problema com causa-raiz identificada e workaround documentado
```

**Gerenciamento de Mudanças:**
```
Tipos de Mudança no ITIL 4:

Normal Change:
  Segue processo formal de aprovação
  CAB (Change Advisory Board) revisa e aprova
  Implantado em janela planejada

Standard Change:
  Mudança pré-aprovada, baixo risco, procedimento definido
  Ex: criação de conta de usuário, deploy de patch aprovado

Emergency Change:
  Necessária imediatamente para restaurar serviço crítico
  Processo de aprovação acelerado (ECAB)
  Documentação pós-implantação
```

**Gerenciamento de Nível de Serviço (SLM):**
```
SLA (Service Level Agreement):
  Acordo entre provedor de TI e cliente sobre nível de serviço

OLA (Operational Level Agreement):
  Acordo interno entre equipes de TI

UC (Underpinning Contract):
  Contrato com fornecedores externos

Exemplos de métricas de SLA:
  Disponibilidade: 99,9% (= máx. 8,76h de downtime/ano)
  MTTR (Mean Time to Repair): < 4 horas para P1
  MTBF (Mean Time Between Failures): > 720 horas
  Tempo de resposta a requisições: < 2 dias úteis
```

**Melhoria Contínua:**

O ITIL 4 usa o modelo de **Melhoria Contínua** baseado no ciclo PDCA de Deming, adaptado em 7 passos:

```
1. Qual é a visão?        → Estratégia e objetivos de negócio
2. Onde estamos agora?    → Avaliação do estado atual (baselines)
3. Onde queremos chegar?  → Metas e objetivos mensuráveis
4. Como chegamos lá?      → Planejamento de iniciativas
5. Agir!                  → Implementar as melhorias
6. Chegamos lá?           → Medição e avaliação dos resultados
7. Como mantemos o ritmo? → Consolidação; próxima rodada de melhoria
```

### 7.7 ITIL e DevOps

O ITIL 4 foi redesenhado para integrar-se com práticas modernas de desenvolvimento:

**ITIL 4 + DevOps:**
- **VSM** (Value Stream Mapping) — identificar fluxos de entrega de ponta a ponta
- **Feedback rápido** — telemetria, observabilidade, monitoramento de produção
- **Automação** — pipelines CI/CD, IaC, testes automatizados
- **Mudanças de baixo risco** — pipeline de CD como "standard change"

```
DevOps Pipeline integrado com ITIL 4:

Código → Build → Teste → Deploy → Monitoramento
  ↑                                      │
  │    Gerenciamento de Incidentes       │
  │    Gerenciamento de Problemas   ←────┘
  │    Melhoria Contínua
  └────────────────────────────────────────
```

**DORA Metrics (complementares ao ITIL):**
```
Deployment Frequency:      Com que frequência deployamos em produção?
Lead Time for Changes:     Quanto tempo do commit ao deploy em produção?
Change Failure Rate:       % de mudanças que causam incidentes/rollback?
Time to Restore Service:   Quanto tempo para recuperar de uma falha?
```

---

## 8. Comparativo entre Frameworks

```
┌──────────────┬──────────────┬──────────────┬─────────────���┬──────────────┐
│  Framework   │   Foco       │  Abordagem   │  Aplicação   │ Certificação │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ PMBOK 7      │ Gestão de    │ Princípios + │ Qualquer     │ PMP          │
│              │ Projetos     │ Domínios     │ setor        │              │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ PMI-ACP /    │ Ágil         │ Valores,     │ Produto      │ PMI-ACP,     │
│ Scrum        │              │ iterativo    │ digital      │ CSM, PSM     │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ Kanban       │ Fluxo de     │ Evolutivo,   │ Manutenção,  │ KMP, PSK     │
│              │ trabalho     │ visual       │ suporte, ops │              │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ Six Sigma    │ Qualidade e  │ Estatístico, │ Processos    │ BB, GB, MBB  │
│              │ processos    │ DMAIC        │ de qualidade │              │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ CMMI         │ Maturidade   │ Prescritivo, │ Organizações │ Appraisal    │
│              │ organizac.   │ níveis       │ de software  │ SCAMPI       │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ MPS-BR       │ Maturidade   │ Prescritivo, │ MPMEs        │ Avaliação    │
│              │ (BR)         │ 7 níveis     │ brasileiras  │ MPS-BR       │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ ITIL 4       │ Serviços     │ Holístico,   │ Gestão de    │ ITIL 4       │
│              │ de TI        │ valor        │ serviços TI  │ Foundation   │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘
```

**Complementaridade dos Frameworks:**

```
                         ┌─────────────┐
                         │    PMBOK    │
                         │  (Projetos) │
                         └──────┬──────┘
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
      ┌──────┴──────┐   ┌───────┴──────┐   ┌──────┴──────┐
      │  Scrum/XP   │   │    Kanban    │   │   SAFe      │
      │  (Times)    │   │   (Fluxo)   │   │  (Escala)   │
      └──────┬──────┘   └──────┬───────┘   └──────┬──────┘
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
             ┌──────┴──────┐         ┌──────┴──────┐
             │  CMMI/MPS   │         │    ITIL 4   │
             │  (Processo  │         │  (Serviços) │
             │   Org.)     │         │             │
             └──────┬──────┘         └──────┬──────┘
                    │                       │
                    └───────────┬───────────┘
                                │
                       ┌────────┴────────┐
                       │   Six Sigma     │
                       │  (Qualidade /   │
                       │  Melhoria)      │
                       └─────────────────┘
```

---

## 9. Escolhendo o Framework Certo

A escolha do framework ou metodologia depende de vários fatores contextuais. Os frameworks **não são mutuamente exclusivos** — a maioria das organizações maduras combina elementos de múltiplos modelos.

### Quando usar abordagem Preditiva (PMBOK/Waterfall)

- Requisitos bem definidos e estáveis
- Alta criticidade regulatória (governo, saúde, aeroespacial)
- Contratos de preço fixo com escopo fechado
- Equipes distribuídas com comunicação assíncrona
- Projetos de infraestrutura física com dependências rígidas

### Quando usar Scrum

- Produto digital com escopo emergente
- Cliente disponível e engajado para feedback frequente
- Time dedicado de 5-9 pessoas no mesmo produto
- Necessidade de entregas frequentes e visíveis
- Ambiente com alta mudança de prioridades

### Quando usar Kanban

- Trabalho contínuo sem ciclos definidos (manutenção, suporte, operações)
- Equipe já tem processo estabelecido e quer melhorar gradualmente
- Alta variabilidade no tipo e tamanho dos itens de trabalho
- Time não pode se comprometer com Sprints fixas

### Quando usar Six Sigma

- Processos com alta variabilidade e custo de defeito elevado
- Necessidade de redução de defeitos baseada em dados
- Processos repetitivos e mensuráveis
- Projetos de melhoria de qualidade em produção de software

### Quando implementar CMMI

- Contratos governamentais que exigem nível de maturidade
- Organizações que precisam de previsibilidade e padronização
- Empresas de defesa, aeroespacial ou saúde
- Necessidade de demonstrar capacidade a clientes internacionais

### Quando implementar MPS-BR

- Empresas brasileiras de pequeno e médio porte
- Necessidade de certificação nacional a custo acessível
- Empresas que querem evoluir gradualmente (7 níveis)
- Organizações que concorrem em editais governamentais brasileiros

### Quando adotar ITIL

- Organização com papel de provedor de serviços de TI
- Necessidade de SLAs formais com clientes internos/externos
- Ambientes de produção com requisitos de disponibilidade elevada
- Equipes de operação e suporte que precisam de processos claros
- Integração de desenvolvimento com operações (DevOps)

### Exemplo de Combinação Prática

Uma organização de software de médio porte pode adotar:

```
Nível de Portfólio:    PMBOK (priorização e governança de projetos)
                              +
Nível de Programa:     SAFe (coordenação entre times ágeis)
                              +
Nível de Time:         Scrum (desenvolvimento de produto)
                              +
Nível de Operações:    Kanban + ITIL 4 (manutenção e suporte)
                              +
Qualidade Org.:        MPS-BR Nível F (maturidade de processo)
                              +
Melhoria Contínua:     Lean / Six Sigma (redução de defeitos)
```

### Indicadores de Sucesso

Independente do framework escolhido, o sucesso é medido pelo impacto no negócio:

```
Entrega de Valor:
  ├── % de features entregues no prazo e dentro do orçamento
  ├── NPS (Net Promoter Score) do produto/serviço
  └── Revenue attribution das features entregues

Qualidade:
  ├── Defect Escape Rate (defeitos em produção / total de defeitos)
  ├── MTTR (tempo médio de recuperação)
  └── Change Failure Rate

Eficiência:
  ├── Lead Time (ideia ao deploy em produção)
  ├── Deployment Frequency
  └── Cost of Delay

Satisfação:
  ├── eNPS (Employee Net Promoter Score) da equipe
  ├── Retenção de talentos
  └── Satisfação de stakeholders (pesquisa periódica)
```

---

> **Nota:** Os frameworks e metodologias apresentados neste documento evoluem continuamente. Recomenda-se consultar as fontes oficiais (PMI, Scrum.org, Kanban University, ISACA, SOFTEX, Axelos/PeopleCert) para obter as versões mais atualizadas dos guias e normas.
