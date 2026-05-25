---
tags: [devops, cicd, ci, pipeline, automacao]
aliases: [CI, Integração Contínua, Continuous Integration]
---

# ⚙️ CI — Integração Contínua

> [!abstract] Definição
> **Continuous Integration (CI)** é a prática de integrar o código de todos os membros do time frequentemente — idealmente várias vezes por dia — com validação automatizada a cada integração.

---

## O problema que a CI resolve

```
❌ Sem CI (antigamente):
Dev A trabalha 2 semanas isolado
Dev B trabalha 2 semanas isolado
Ambos tentam juntar o código → MERGE HELL 🔥

✅ Com CI:
Dev A faz commit → pipeline roda → ok
Dev B faz commit → pipeline roda → conflito detectado imediatamente
```

---

## Fluxo Básico de CI

```
1. Dev faz commit (ou abre PR)
        ↓
2. Pipeline CI é disparado automaticamente
        ↓
3. Build — código compilado / empacotado
        ↓
4. Testes automatizados rodam
   ├── Testes unitários
   ├── Testes de integração
   └── Análise estática (lint, SAST)
        ↓
5a. ✅ Passou → código aprovado, pronto para CD
5b. ❌ Falhou → dev é notificado, branch bloqueada
```

---

## Tipos de Testes no Pipeline CI

| Tipo | O que valida | Velocidade |
|---|---|---|
| **Unitário** | Funções e classes isoladas | Muito rápido |
| **Integração** | Interação entre componentes | Médio |
| **E2E (End-to-End)** | Fluxo completo do usuário | Lento |
| **SAST** | Vulnerabilidades no código-fonte | Médio |
| **Lint** | Estilo e qualidade do código | Rápido |

> [!tip] Pirâmide de testes
> A boa prática é ter **muitos testes unitários** (base), **alguns de integração** (meio) e **poucos E2E** (topo). Isso mantém o pipeline rápido.

---

## Boas Práticas de CI

- ✅ Pipeline deve rodar em **menos de 10 minutos**
- ✅ Todo commit deve disparar o pipeline
- ✅ Branch com teste falhando **não pode ser mergeada**
- ✅ Resultados devem ser visíveis para todo o time
- ✅ Pipeline deve ser tratado como código (versionado)

> [!warning] Antipadrão
> Não commitar direto na branch `main` sem passar por um pipeline. Isso elimina o benefício da CI.

---

## Ferramentas de CI

```
GitHub Actions   → integrado ao GitHub, YAML, ecossistema rico
GitLab CI/CD     → nativo do GitLab, runners customizáveis
Jenkins          → open source, altamente flexível, legado
CircleCI         → fácil configuração, boa performance
Tekton           → nativo Kubernetes, para quem usa K8s
```

---

## Exemplo de Pipeline (GitHub Actions)

```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Instalar dependências
        run: npm install

      - name: Rodar lint
        run: npm run lint

      - name: Rodar testes
        run: npm test

      - name: Build
        run: npm run build
```

---

## Links relacionados

- [[04 - CD — Entrega e Deploy Contínuos]] — o próximo passo após a CI
- [[05 - Automação]] — a automação que sustenta a CI
- [[02 - Cultura DevOps e CALMS]] — o princípio de feedback rápido
- [[🗺️ MOC — DevOps]] — voltar ao mapa

