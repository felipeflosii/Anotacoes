---
tags: [devops, alta-disponibilidade, sre, arquitetura, resiliencia]
aliases: [Alta Disponibilidade, HA, High Availability]
---

# 🟢 Alta Disponibilidade

> [!abstract] Definição
> **Alta disponibilidade (HA)** é a capacidade do sistema de **permanecer operacional e acessível** mesmo diante de falhas de componentes individuais.

---

## O que significa "disponível"?

Disponibilidade é medida em percentual de tempo online:

| SLA | Downtime por ano | Downtime por mês |
|---|---|---|
| 99% (dois noves) | 3,65 dias | 7,2 horas |
| 99,9% (três noves) | 8,7 horas | 43,8 minutos |
| 99,99% (quatro noves) | 52,6 minutos | 4,4 minutos |
| 99,999% (cinco noves) | 5,26 minutos | 26 segundos |

> [!tip] Contexto
> A maioria das aplicações de negócio mira em **99,9%** ou **99,99%**. Cinco noves é o território de infraestrutura crítica (bancos, telecom).

---

## Técnicas de Alta Disponibilidade

### 1. Múltiplas Instâncias
Eliminar o ponto único de falha (SPOF — Single Point of Failure).

```
❌ SPOF:
[Usuário] → [Servidor único] ← se cair, tudo cai

✅ HA:
[Usuário] → [Load Balancer]
                 ├── [Servidor A]
                 └── [Servidor B] ← se A cair, B assume
```

### 2. Multi-AZ (Multi Availability Zone)
Distribuir instâncias em zonas físicas diferentes do datacenter.

```
AZ us-east-1a: [Servidor A]
AZ us-east-1b: [Servidor B]
AZ us-east-1c: [Servidor C]

→ Se o datacenter 1a tiver problema elétrico, 1b e 1c continuam
```

### 3. Redundância de Banco de Dados
```
[App] → [DB Primário (escrita)]
              ↓ replicação
         [DB Réplica (leitura)]

Se o primário cair → réplica promovida automaticamente (failover)
```

### 4. Failover Automático
O sistema detecta a falha e redireciona o tráfego automaticamente, sem intervenção humana.

### 5. Deploy sem Downtime
Ver estratégias em [[04 - CD — Entrega e Deploy Contínuos]]:
- Blue-Green deployment
- Rolling update
- Canary release

---

## Health Checks

O Load Balancer usa health checks para saber quais instâncias estão saudáveis.

```
Load Balancer pergunta a cada 30s:
GET /health → 200 OK = saudável ✅
GET /health → 500 / timeout = doente ❌ → remove do pool
```

---

## HA vs Resiliência

> [!note] Diferença importante
> São conceitos complementares, mas distintos:

| | Alta Disponibilidade | Resiliência |
|---|---|---|
| **Objetivo** | Evitar que o sistema fique indisponível | Continuar funcionando mesmo com falhas |
| **Foco** | Redundância e failover | Comportamento sob falha |
| **Quando age** | Antes da falha (prevenção) | Durante/após a falha (recuperação) |
| **Exemplo** | Dois servidores em zonas diferentes | Circuit breaker que evita cascata de erros |

Ver: [[09 - Resiliência]]

---

## Métricas de HA

- **Uptime** — % de tempo disponível
- **MTBF** (Mean Time Between Failures) — tempo médio entre falhas
- **MTTR** (Mean Time to Recover) — tempo médio para se recuperar
- **RTO** (Recovery Time Objective) — tempo máximo aceitável de downtime
- **RPO** (Recovery Point Objective) — perda máxima aceitável de dados

---

## Links relacionados

- [[09 - Resiliência]] — como se comportar quando algo falha
- [[07 - Escalabilidade]] — HA e escalabilidade andam juntos
- [[10 - Observabilidade]] — detectar falhas rapidamente
- [[04 - CD — Entrega e Deploy Contínuos]] — deploy sem downtime
- [[🗺️ MOC — DevOps]] — voltar ao mapa

