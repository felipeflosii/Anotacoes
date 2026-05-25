---
tags: [devops, observabilidade, logs, metricas, tracing, prometheus, grafana]
aliases: [Observabilidade, Observability, Monitoring, Logs, Métricas, Tracing]
---

# 📊 Observabilidade

> [!abstract] Definição
> **Observabilidade** é a capacidade de entender o que está acontecendo **dentro do sistema** a partir dos seus outputs externos — sem precisar adicionar novo código para cada nova pergunta.

---

## A pergunta central

> Quando algo dá errado em produção, você consegue responder:
> - **O que** está acontecendo?
> - **Onde** está o problema?
> - **Por que** aconteceu?
> - **Quando** começou?

Se não consegue responder sem adicionar código novo, sua observabilidade está incompleta.

---

## Monitoramento vs Observabilidade

| | Monitoramento | Observabilidade |
|---|---|---|
| **Abordagem** | Verifica métricas conhecidas | Responde perguntas desconhecidas |
| **Pergunta** | "Esse threshold foi ultrapassado?" | "Por que os usuários estão lentos?" |
| **Uso** | Alertas reativos | Debugging proativo |

---

## Os 3 Pilares

### 1. 📋 Logs

Registros textuais de eventos que aconteceram no sistema.

```json
{
  "timestamp": "2024-01-15T14:23:01Z",
  "level": "ERROR",
  "service": "api-pagamento",
  "traceId": "abc123",
  "message": "Timeout ao conectar com banco de dados",
  "duration_ms": 5001
}
```

**Boas práticas:**
- Sempre estruturado (JSON, não texto livre)
- Incluir `traceId` para correlacionar com tracing
- Nível de log adequado (DEBUG, INFO, WARN, ERROR)
- Centralizar em uma plataforma (não ficar em cada servidor)

**Ferramentas:** ELK Stack (Elasticsearch + Logstash + Kibana), Loki + Grafana

---

### 2. 📈 Métricas

Dados numéricos coletados ao longo do tempo.

```
Taxa de requests por segundo: 1200 req/s
Latência P99: 450ms
Taxa de erro: 0.3%
CPU usage: 67%
Memória: 4.2 GB / 8 GB
```

**Tipos de métricas:**
| Tipo | Descrição | Exemplo |
|---|---|---|
| **Counter** | Só aumenta | Total de requests |
| **Gauge** | Sobe e desce | CPU, memória atual |
| **Histogram** | Distribuição | Latência por percentil (P50, P99) |
| **Summary** | Percentis calculados | Tempo de resposta |

**Ferramentas:** Prometheus (coleta), Grafana (visualização)

---

### 3. 🔍 Tracing Distribuído

Rastreia o caminho de uma request através de múltiplos serviços.

```
Request do usuário
    │
    ├─ [API Gateway] 12ms
    │       │
    │       ├─ [Serviço de Auth] 8ms ✅
    │       │
    │       └─ [Serviço de Pedidos] 450ms ⚠️
    │               │
    │               └─ [Banco de Dados] 420ms ❌ ← AQUI está o problema!
    │
    └─ Total: 470ms
```

**Ferramentas:** Jaeger, Zipkin, OpenTelemetry (padrão), Datadog APM

---

## SLI, SLO e SLA

> [!note] Vocabulário de SRE
> Esses termos definem como medir e comprometer-se com a confiabilidade.

| Sigla | Nome | Significado |
|---|---|---|
| **SLI** | Service Level Indicator | Métrica que mede o que importa (ex: % de requests com < 300ms) |
| **SLO** | Service Level Objective | Meta interna (ex: 99,9% das requests < 300ms) |
| **SLA** | Service Level Agreement | Compromisso contratual com cliente (ex: 99,5% uptime) |

---

## Alertas

> [!warning] Alerta bom vs alerta ruim
> - ❌ **Alerta ruim:** dispara toda hora, equipe ignora ("alert fatigue")
> - ✅ **Alerta bom:** dispara apenas quando precisa de ação humana, com contexto claro

**Regra de ouro:** Alertas devem ser **acionáveis**. Se não há ação possível, não deve alertar.

---

## Stack Típica de Observabilidade

```
Aplicação
  │ exporta métricas/logs/traces
  ▼
OpenTelemetry (coleta padronizada)
  │
  ├──► Prometheus (métricas) ──► Grafana (dashboards + alertas)
  │
  ├──► Loki (logs) ──────────► Grafana
  │
  └──► Jaeger (traces) ───────► Grafana / UI do Jaeger
```

---

## Links relacionados

- [[09 - Resiliência]] — observabilidade detecta falhas para acionar respostas
- [[08 - Alta Disponibilidade]] — medir uptime e disponibilidade
- [[11 - O Fluxo Completo DevOps]] — observabilidade como parte do ciclo
- [[🗺️ MOC — DevOps]] — voltar ao mapa

