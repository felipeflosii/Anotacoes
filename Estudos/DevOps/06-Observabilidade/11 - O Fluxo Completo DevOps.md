---
tags: [devops, fundamentos, fluxo, visao-geral]
aliases: [Fluxo DevOps, Ciclo DevOps, Visão Geral DevOps]
---

# 🔄 O Fluxo Completo DevOps

> [!abstract] Visão sistêmica
> DevOps não é um conjunto de ferramentas isoladas. É um **ciclo contínuo** que conecta código, qualidade, infraestrutura e operação em um loop de melhoria constante.

---

## O Ciclo Infinito (∞)

```
         PLAN ──────────────────► CODE
          ▲                         │
          │                         ▼
       MONITOR                    BUILD
          │                         │
          ▼                         ▼
       OPERATE                    TEST
          │                         │
          ▼                         ▼
        DEPLOY ◄────────────── RELEASE
```

Cada fase tem suas práticas e ferramentas DevOps.

---

## As Fases em Detalhe

### 🗂️ PLAN (Planejamento)
- Definição de requisitos e prioridades
- Backlog e sprints
- **Ferramentas:** Jira, Linear, GitHub Issues

### 💻 CODE (Desenvolvimento)
- Escrita de código e testes
- Code review e PRs
- **Ferramentas:** Git, GitHub, GitLab, VS Code

### 🔨 BUILD (Construção)
- Compilação e empacotamento
- Build de imagens Docker
- **Ferramentas:** Maven, Gradle, Docker, npm
- Ver: [[03 - CI — Integração Contínua]]

### 🧪 TEST (Testes)
- Testes automatizados em pipeline
- Análise de qualidade e segurança
- **Ferramentas:** Jest, JUnit, SonarQube, Snyk
- Ver: [[03 - CI — Integração Contínua]]

### 📦 RELEASE (Entrega)
- Geração do artefato versionado
- Aprovação e promoção entre ambientes
- **Ferramentas:** GitHub Releases, Helm, ArgoCD
- Ver: [[04 - CD — Entrega e Deploy Contínuos]]

### 🚀 DEPLOY (Implantação)
- Deploy nos ambientes alvo
- Estratégias sem downtime
- **Ferramentas:** Kubernetes, Terraform, Ansible
- Ver: [[04 - CD — Entrega e Deploy Contínuos]] | [[06 - Infraestrutura como Código (IaC)]]

### ⚙️ OPERATE (Operação)
- Gestão dos sistemas em produção
- Escalabilidade e disponibilidade
- **Ferramentas:** Kubernetes, AWS, GCP, Azure
- Ver: [[07 - Escalabilidade]] | [[08 - Alta Disponibilidade]]

### 📊 MONITOR (Monitoramento)
- Observar métricas, logs e traces
- Detectar anomalias e alertar
- **Ferramentas:** Prometheus, Grafana, Datadog, Jaeger
- Ver: [[10 - Observabilidade]]

---

## O Fluxo Técnico Linear

```
Dev faz commit
      │
      ▼
Pipeline CI dispara automaticamente
      │
      ├── Build ────────────── ❌ erro? notifica dev
      ├── Lint/SAST ─────────── ❌ erro? notifica dev
      ├── Testes Unitários ──── ❌ erro? notifica dev
      └── Testes Integração ─── ❌ erro? notifica dev
      │ ✅ tudo ok
      ▼
Artefato gerado (imagem Docker, jar, etc.)
      │
      ▼
Deploy automático em Staging
      │
      ▼
Smoke Tests em Staging
      │ ✅ ok
      ▼
[Continuous Delivery] → aprovação humana
ou
[Continuous Deployment] → automático
      │
      ▼
Deploy em Produção
      │
      ├── Logs sendo coletados
      ├── Métricas sendo monitoradas
      └── Alertas configurados
      │
      ▼
Feedback → melhoria → novo commit → ciclo recomeça
```

---

## O que o Mercado Espera de Você

> [!important] Expectativas reais do mercado
> Mais do que saber ferramentas, esperam que você:

- ✅ Entenda o **pipeline de ponta a ponta**
- ✅ Saiba **automatizar processos** repetitivos
- ✅ Consiga **subir uma aplicação do zero até produção**
- ✅ Entenda **como sistemas falham** e como evitar isso
- ✅ Saiba **ler logs e métricas** para diagnosticar problemas
- ✅ Tenha **visão sistêmica** — não apenas ferramentas isoladas

---

## Conexão entre Todos os Tópicos

```
[[01 - O que é DevOps]]
         │
[[02 - Cultura DevOps e CALMS]]
         │
    ┌────┴────┐
    │         │
[[03 - CI]] [[05 - Automação]]
    │
[[04 - CD]]
    │
    ├── [[06 - IaC]]
    ├── [[07 - Escalabilidade]]
    ├── [[08 - Alta Disponibilidade]]
    ├── [[09 - Resiliência]]
    └── [[10 - Observabilidade]]
         │
    (feedback volta ao início do ciclo)
```

---

## Links relacionados

- [[🗺️ MOC — DevOps]] — mapa completo do vault
- [[🛣️ Roadmap DevOps]] — caminho de aprendizado

