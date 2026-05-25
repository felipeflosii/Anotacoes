---
tags: [devops, fundamentos, cultura, calms]
aliases: [Cultura DevOps, CALMS, Modelo CALMS]
---

# 🤝 Cultura DevOps e Modelo CALMS

> [!abstract] Ideia central
> A base de tudo no DevOps é **cultural**. Sem cultura, ferramentas não resolvem. Times podem ter Jenkins, Kubernetes e Terraform, e ainda assim serem disfuncionais se a cultura for ruim.

---

## Os 4 Princípios Centrais

### 1. Responsabilidade Compartilhada
O time inteiro — dev, ops e QA — é **co-dono** do sistema.
- Não existe "meu código" e "seu servidor"
- Falha em produção é problema de todos
- Sucesso em produção também é de todos

### 2. Colaboração Contínua
Quebrar o silo entre as equipes.
- Devs entendem de operação
- Ops entendem do produto
- QA trabalha junto desde o início, não no final

### 3. Feedback Rápido (Shift-Left)
Detectar erros o mais cedo possível.

```
❌  Lento:  Code → Code → Code → Code → TESTE → erro encontrado tarde

✅  DevOps: Code → TESTE → Code → TESTE → Code → TESTE
```

> [!tip] Shift-left
> "Shift-left" significa mover o teste e a validação para mais **à esquerda** no fluxo — ou seja, mais cedo. Quanto mais cedo o erro é encontrado, mais barato é corrigi-lo.

### 4. Melhoria Contínua (Kaizen)
Pequenas otimizações constantes no processo.
- Não esperar pelo "grande projeto de melhorias"
- Cada sprint melhora um pouco o processo
- Retrospectivas são ferramentas DevOps

---

## Modelo CALMS

O CALMS é o **framework conceitual** que organiza os pilares de uma cultura DevOps madura.

| Letra | Pilar | O que significa na prática |
|---|---|---|
| **C** | Culture | Cultura colaborativa entre todos os times |
| **A** | Automation | Automatizar tudo o que for repetível e manual |
| **L** | Lean | Eliminar desperdício, entregar em pequenos incrementos |
| **M** | Measurement | Medir tudo: deploy freq., MTTR, taxa de falha |
| **S** | Sharing | Compartilhar conhecimento, postmortems, runbooks |

---

## Destrinchando cada pilar

### C — Culture (Cultura)
- Times com trust mútuo
- Postmortems sem culpa (*blameless postmortem*)
- Experimentos são encorajados

### A — Automation (Automação)
- CI/CD pipelines
- Infraestrutura como código
- Testes automatizados
- Ver: [[05 - Automação]]

### L — Lean (Enxuto)
- Inspirado no Sistema Toyota de Produção
- Reduzir WIP (Work in Progress)
- Fluxo contínuo ao invés de entregas em batch

### M — Measurement (Métricas)
Métricas essenciais DevOps (DORA Metrics):
- **Deployment Frequency** — com que frequência se faz deploy
- **Lead Time for Changes** — do commit ao deploy, quanto tempo?
- **Change Failure Rate** — % de deploys que causam falha
- **MTTR** — tempo médio para se recuperar de um incidente

### S — Sharing (Compartilhamento)
- Wikis e runbooks acessíveis
- Postmortems públicos internamente
- Pair programming entre dev e ops
- Comunidades de prática

---

## Links relacionados

- [[01 - O que é DevOps]] — contexto e definição
- [[05 - Automação]] — o pilar A do CALMS em detalhe
- [[10 - Observabilidade]] — o pilar M (Measurement) em detalhe
- [[🗺️ MOC — DevOps]] — voltar ao mapa

