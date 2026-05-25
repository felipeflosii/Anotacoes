---
tags: [java, ano-5, staff-engineer, adr, rfc, tech-strategy, mentoria]
trimestre: T17
meses: 52-54
---

# T17 · Staff Engineer e Impacto Técnico
### Meses 52–54 · Ano 5

> **Objetivo:** Entender o que diferencia um Staff Engineer de um Sênior. Impacto organizacional, influência sem autoridade, decisões técnicas em escala.

---

## 🔵 Bloco 1 — O que é um Staff Engineer

> [!note] Staff ≠ Sênior que escreve mais código
> Staff Engineers multiplicam o impacto de outros devs. Eles resolvem problemas que afetam múltiplos times e criam sistemas que outros mantêm por anos.

### Archetypes (Will Larson — "Staff Engineer")

| Archetype | O que faz | Onde passa o tempo |
|-----------|-----------|-------------------|
| **Tech Lead** | Lidera tecnicamente um time específico | Próximo ao time, code reviews, pair programming |
| **Architect** | Define estratégia técnica cross-time | Design sessions, RFCs, prototipagem |
| **Solver** | Resolve problemas difíceis que ninguém mais consegue | Problema → resolve → move para o próximo |
| **Right Hand** | Amplifica o impacto de um líder de engenharia | Reuniões executivas, delegação técnica |

### Como Staff Engineers passam o tempo

```
Código (30-50%)          — ainda escrevem código, mas código de alavancagem
Revisões técnicas (20%)  — code reviews, design reviews, RFC reviews
Documentação (15%)       — RFCs, ADRs, runbooks, guias de arquitetura
Mentoria (15%)           — 1:1s, pair programming, career development
Reuniões estratégicas    — alinhamento com produto, gestão, outros times
(10-20%)
```

---

## 🔵 Bloco 2 — Architecture Decision Records (ADRs)

> [!tip] ADRs documentam o "por quê" das decisões técnicas
> Sem ADRs, o conhecimento de por que certas escolhas foram feitas se perde quando as pessoas saem.

```markdown
# ADR-0042: Adotar Virtual Threads para Serviços HTTP

## Status
Aceito — 2026-01-15

## Contexto
Nossos serviços Spring Boot usam modelo thread-per-request com Tomcat.
Com o crescimento de 3x em 2025, estamos gastando USD 12k/mês em infra
apenas para suportar concorrência. Testamos 3 abordagens:
1. Escalar horizontal (mais pods) — custo linear
2. Migrar para WebFlux/Reactive — alto custo de migração e treinamento
3. Habilitar Virtual Threads (Java 21) — mínima mudança de código

## Decisão
Habilitaremos Virtual Threads em todos os serviços Java 21 via
`spring.threads.virtual.enabled=true`.

## Consequências

### Positivas
- Throughput estimado 4-8x maior nos testes de carga (ver benchmark em /docs/vt-benchmark.md)
- Zero mudança no código de negócio (transparente para devs)
- Redução estimada de 60% nos pods necessários → ~USD 7k/mês de economia

### Negativas
- Não funciona com ThreadLocal para propagação de contexto em alguns casos edge
- Bibliotecas com synchronized blocks (alguns drivers JDBC antigos) perdem benefício
- Equipe precisa entender o modelo para debugar problemas de pinning

## Alternativas Consideradas
- WebFlux: descartado pelo custo de reescrita estimado em 8+ sprints
- Scaling horizontal: descartado por não resolver o custo estruturalmente

## Referências
- [JEP 444: Virtual Threads](https://openjdk.org/jeps/444)
- [Benchmark interno](/docs/vt-benchmark-jan2026.md)
- [Análise de pinning nos nossos serviços](/docs/vt-pinning-analysis.md)
```

---

## 🔵 Bloco 3 — RFC (Request for Comments)

> [!note] RFCs para mudanças grandes que afetam múltiplos times
> Diferente de ADRs (que documentam decisões já tomadas), RFCs propõem mudanças e coletam feedback antes de implementar.

```markdown
# RFC-0018: Padronização de Event Schema com AsyncAPI

## Resumo
Proposta para adotar AsyncAPI 3.0 como padrão de documentação e validação
de schemas de eventos Kafka em todos os times.

## Motivação
Atualmente temos 3 times usando formatos diferentes de documentação de eventos:
- Pedidos: README no repositório
- Clientes: Confluence page
- Pagamentos: Nenhuma documentação formal

Isso causou 4 incidentes de incompatibilidade em 2025 quando serviços assumiram
formatos errados de eventos.

## Proposta Detalhada
[...]

## Impactos por Time
| Time | Esforço estimado | Prazo sugerido |
|------|-----------------|----------------|
| Pedidos | 2 dias | Sprint 48 |
| Clientes | 3 dias | Sprint 49 |
| Pagamentos | 1 dia | Sprint 48 |
| Platform (infra) | 5 dias (CI integration) | Sprint 50 |

## Riscos
1. Adoção parcial pode piorar o problema
   → Mitigação: Gate obrigatório no CI a partir de Sprint 52

## Período de Feedback
Até 2026-02-01. Comentários via PR ou reunião de design review (quinta 14h).

## Decisão
[A ser preenchido após feedback]
```

---

## 🔵 Bloco 4 — Technical Strategy

### Tech Radar interno

Baseado no [Thoughtworks Tech Radar](https://www.thoughtworks.com/radar):

```markdown
## Tech Radar — Time de Backend Java (2026 Q1)

### ADOTE (use em novos projetos)
- Java 21 + Virtual Threads
- Spring Boot 3.3+
- Testcontainers para testes de integração
- OpenTelemetry para observabilidade
- Kafka para event streaming

### EXPERIMENTE (piloto em 1-2 serviços)
- Spring AI para funcionalidades com LLM
- Apache Flink para streaming complexo
- GraalVM Native Image para Lambda functions

### AVALIE (fique de olho)
- Project Loom Structured Concurrency (ainda preview)
- Quarkus para workloads serverless
- CRaC (Coordinated Restore at Checkpoint) para startup rápido

### EVITE (não use em novos projetos)
- Hystrix (use Resilience4j)
- Spring Cloud Netflix Ribbon (use Spring Cloud LoadBalancer)
- Java 8 (upgrade para 21 LTS)
- PermGen flags JVM (era Java 7)
```

### Skill Matrix do time

```
Skill                | Nível time atual | Meta 6 meses
---------------------|-----------------|-------------
Java 21 features     | 2/5             | 4/5
Spring Boot 3.x      | 4/5             | 5/5
Testcontainers       | 2/5             | 4/5
Kafka avançado       | 3/5             | 4/5
Kubernetes           | 2/5             | 3/5
Observabilidade      | 2/5             | 4/5
```

---

## 🔵 Bloco 5 — Mentoria Técnica

### Framework de Mentoria 1:1

```
Estrutura de 1h de 1:1 com mentorado:

1. Check-in (10 min)
   - Como está indo? Bloqueios?
   - Progress nos objetivos da última semana?

2. Tópico técnico (30 min)
   - Code review profundo de algo que escreveram
   - Pair programming em algo desafiador
   - Explicação de conceito que eles pediram

3. Desenvolvimento de carreira (15 min)
   - Onde querem chegar em 6 meses/1 ano?
   - Quais habilidades precisam desenvolver?
   - Feedback honesto sobre o que está bem e o que melhorar

4. Próximos passos (5 min)
   - Definir 1-2 ações concretas para a próxima semana
```

### Como dar feedback técnico efetivo

```
Feedback fraco:
"Este código não está muito bom."

Feedback forte:
"Esta classe tem 3 responsabilidades diferentes (SRP violation):
1. Busca dados do banco (linha 45-67)
2. Transforma o DTO (linha 70-95)
3. Envia email (linha 98-120)

Sugiro extrair EmailNotificationService e PedidoMapper.
Veja este exemplo de como ficaria: [link para PR]

O que você acha? Tem alguma razão para manter assim?"
```

---

## 🔗 Navegação

← [[Ano 5 — Liderança Técnica]]  
→ [[T18 — Open Source e Liderança Técnica]]
