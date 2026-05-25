---
tags: [devops, cicd, cd, deploy, pipeline]
aliases: [CD, Continuous Delivery, Continuous Deployment, Deploy Contínuo]
---

# 🚢 CD — Entrega e Deploy Contínuos

> [!abstract] Definição
> **CD** é a extensão natural da [[03 - CI — Integração Contínua|CI]]. Após o código ser validado, o CD garante que ele seja **entregue ao ambiente de destino** de forma confiável e automatizada.

---

## Continuous Delivery vs Continuous Deployment

> [!important] Diferença crucial
> São dois conceitos diferentes, mas complementares!

### Continuous Delivery (Entrega Contínua)
- Código **sempre pronto** para ser deployado
- O deploy em produção é **acionado manualmente**
- Um humano decide **quando** vai para produção
- Ideal quando há aprovações regulatórias ou de negócio

```
CI ✅ → Staging → [👤 Aprovação humana] → Produção
```

### Continuous Deployment (Deploy Contínuo)
- Deploy **totalmente automático** até produção
- Se todos os testes passam → vai para produção sozinho
- Zero intervenção humana
- Exige altíssima cobertura de testes e confiança no pipeline

```
CI ✅ → Staging → Testes automatizados ✅ → Produção 🚀
```

---

## O Pipeline Completo CI/CD

```
commit
  │
  ▼
build ──────── ❌ falha → notifica dev, para tudo
  │
  ▼
testes ─────── ❌ falha → notifica dev, para tudo
  │
  ▼
package (imagem Docker, artefato, etc.)
  │
  ▼
deploy em staging / homologação
  │
  ▼
testes de fumaça (smoke tests)
  │
  ▼
[Delivery] aprovação manual
ou
[Deployment] automático
  │
  ▼
deploy em produção 🚀
  │
  ▼
monitoramento ← [[10 - Observabilidade]]
```

---

## Ambientes Típicos

| Ambiente | Propósito | Quem usa |
|---|---|---|
| **Dev/Local** | Desenvolvimento ativo | Desenvolvedor |
| **CI** | Testes automatizados | Pipeline |
| **Staging/Homolog** | Validação pré-produção | Time + QA |
| **Produção** | Usuário final | Todos |

> [!tip] Paridade de ambientes
> Ambientes devem ser o mais **idênticos possível** entre si. [[06 - Infraestrutura como Código (IaC)|IaC]] é fundamental para garantir isso.

---

## Estratégias de Deploy

### Blue-Green
```
[Blue] → produção atual (v1)
[Green] → nova versão (v2) sobe paralelamente
→ tráfego redirecionado para Green
→ Blue fica em standby (rollback rápido)
```

### Canary Release
```
[100% usuários] → v1
[  5% usuários] → v2 (canary)
→ métricas ok? → aumenta gradualmente
→ erro? → reverte o canary
```

### Rolling Update
```
instância 1: v1 → v2
instância 2: v1 → v2
instância 3: v1 → v2  (uma de cada vez, sem downtime)
```

---

## Métricas de CD (DORA)

- **Deployment Frequency** — quantas vezes por dia/semana se faz deploy?
- **Lead Time** — do commit ao deploy em produção, quanto tempo?
- **Change Failure Rate** — % de deploys que causam incidente
- **MTTR** — quanto tempo para restaurar após falha?

---

## Links relacionados

- [[03 - CI — Integração Contínua]] — etapa anterior ao CD
- [[05 - Automação]] — base técnica do pipeline
- [[08 - Alta Disponibilidade]] — deploy sem downtime
- [[10 - Observabilidade]] — monitorar após o deploy
- [[🗺️ MOC — DevOps]] — voltar ao mapa

