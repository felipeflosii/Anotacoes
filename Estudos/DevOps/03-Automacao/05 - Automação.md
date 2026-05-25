---
tags: [devops, automacao, pipeline, fundamentos]
aliases: [Automação, Automation]
---

# 🤖 Automação

> [!abstract] Princípio
> Todo processo manual é uma **fonte potencial de erro**. DevOps depende de eliminar tarefas repetitivas e substituí-las por automação confiável e rastreável.

---

## Por que automatizar?

```
Manual:
→ Depende de quem executa
→ Passos pulados "por acidente"
→ Não rastreável
→ Não reproduzível
→ Escala mal

Automatizado:
→ Sempre igual
→ Todos os passos executados
→ Auditável (log de tudo)
→ Reproduzível em qualquer ambiente
→ Escala facilmente
```

---

## O que Automatizar no DevOps

### 1. Build
- Compilação do código
- Geração de artefatos (JARs, binários, pacotes)
- Build de imagens Docker

### 2. Testes
- Testes unitários e de integração
- Análise estática de código (lint)
- Scanning de vulnerabilidades (SAST/DAST)
- Testes de performance

### 3. Deploy
- Deploy em staging e produção
- Rollback automático em caso de falha
- Smoke tests pós-deploy

### 4. Provisionamento de Infraestrutura
- Criação de servidores, redes, bancos de dados
- Ver: [[06 - Infraestrutura como Código (IaC)]]

### 5. Monitoramento e Alertas
- Coleta de métricas
- Disparo de alertas
- Ver: [[10 - Observabilidade]]

---

## O Pipeline como Código

> [!important] Conceito-chave
> O próprio pipeline de automação deve ser **versionado como código** — armazenado no repositório, revisado em PRs, versionado com tags.

```
meu-projeto/
├── src/
├── tests/
├── Dockerfile
├── .github/
│   └── workflows/
│       └── pipeline.yml  ← pipeline como código
└── terraform/           ← infra como código
```

---

## Níveis de Maturidade de Automação

| Nível | Descrição |
|---|---|
| **0** | Tudo manual — deploys por SSH, scripts ad-hoc |
| **1** | Scripts de deploy, mas executados manualmente |
| **2** | CI automatizado, CD manual |
| **3** | CI/CD completo, infra manual |
| **4** | CI/CD + IaC, tudo versionado |
| **5** | Self-healing, auto-scaling, GitOps |

---

## Ferramentas de Automação

```
CI/CD:
  GitHub Actions, GitLab CI, Jenkins, CircleCI

Infra:
  Terraform, Ansible, Pulumi

Containers:
  Docker, Kubernetes

Scripts:
  Bash, Python, Makefile
```

---

## Antipadrões de Automação

> [!warning] Evite estes erros comuns
> - 🚫 **Snowflake servers** — servidores configurados manualmente que ninguém sabe como recriar
> - 🚫 **"Só eu sei fazer o deploy"** — automação quebrada que vira dependência pessoal
> - 🚫 **Automatizar o caos** — automatizar um processo ruim só torna o problema mais rápido
> - 🚫 **Scripts sem versionamento** — automação que não está no repositório é automação perdida

---

## Links relacionados

- [[03 - CI — Integração Contínua]] — automação do build e teste
- [[04 - CD — Entrega e Deploy Contínuos]] — automação do deploy
- [[06 - Infraestrutura como Código (IaC)]] — automação da infra
- [[🗺️ MOC — DevOps]] — voltar ao mapa

