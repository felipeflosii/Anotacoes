---
tags: [devops, roadmap, estudos, carreira]
aliases: [Roadmap, Plano de Estudos DevOps]
---

# 🛣️ Roadmap DevOps — Do Zero ao Mercado

> [!abstract] Como usar este roadmap
> Este roadmap está dividido em **4 fases** progressivas. Cada fase tem pré-requisitos, objetivos claros e marcos de conclusão. Siga a ordem — ela foi pensada para construir conhecimento de forma sólida.

---

## Visão Geral das Fases

```
FASE 1 ──── FASE 2 ──── FASE 3 ──── FASE 4
Fundamentos  Prática    Containers   Avançado
  2-3 sem     3-4 sem    3-4 sem      4-6 sem
```

---

## 🟦 FASE 1 — Fundamentos Conceituais
> Duração estimada: **2 a 3 semanas**
> Pré-requisitos: noções básicas de programação e linha de comando

### Objetivos
- [ ] Entender o que é DevOps e por que existe
- [ ] Conhecer o modelo CALMS
- [ ] Entender CI/CD conceitualmente
- [ ] Entender o que é automação e IaC
- [ ] Ter visão sistêmica do fluxo DevOps

### Notas para estudar nesta fase
- [[01 - O que é DevOps]]
- [[02 - Cultura DevOps e CALMS]]
- [[03 - CI — Integração Contínua]]
- [[04 - CD — Entrega e Deploy Contínuos]]
- [[05 - Automação]]
- [[11 - O Fluxo Completo DevOps]]

### Marco de conclusão ✅
> Você consegue explicar o fluxo `commit → build → test → deploy → monitor` para alguém sem conhecimento técnico.

---

## 🟨 FASE 2 — Prática Fundamental
> Duração estimada: **3 a 4 semanas**
> Pré-requisitos: Fase 1 completa, Git básico, terminal Linux

### Objetivos
- [ ] Criar e versionar projetos com Git (branches, PRs, merges)
- [ ] Configurar um pipeline CI com GitHub Actions
- [ ] Automatizar build e testes em pipeline
- [ ] Escrever scripts Bash para automação
- [ ] Provisionar infra básica com Terraform

### Ferramentas a aprender
```
Git & GitHub ──────── versionamento e colaboração
GitHub Actions ──────── pipeline CI/CD
Bash/Shell ──────────── scripts de automação
Terraform (básico) ──── IaC
```

### Notas para estudar nesta fase
- [[06 - Infraestrutura como Código (IaC)]]
- [[07 - Escalabilidade]]
- [[08 - Alta Disponibilidade]]

### Projeto prático sugerido
> Crie um repositório no GitHub com uma aplicação simples (pode ser um "hello world" em Node.js ou Python). Configure um pipeline no GitHub Actions que:
> 1. Roda testes automaticamente a cada PR
> 2. Faz build da aplicação
> 3. Notifica o resultado

### Marco de conclusão ✅
> Você tem um pipeline CI rodando no GitHub Actions que executa testes e build automaticamente.

---

## 🟧 FASE 3 — Containers e Deploy
> Duração estimada: **3 a 4 semanas**
> Pré-requisitos: Fase 2 completa

### Objetivos
- [ ] Entender containers e por que são usados
- [ ] Criar e rodar imagens Docker
- [ ] Fazer deploy de containers em produção
- [ ] Entender Kubernetes básico
- [ ] Configurar observabilidade básica

### Ferramentas a aprender
```
Docker ─────────── criar e rodar containers
Docker Compose ─── múltiplos containers localmente
Kubernetes ──────── orquestração de containers
Prometheus ──────── coleta de métricas
Grafana ─────────── dashboards e alertas
```

### Notas para estudar nesta fase
- [[09 - Resiliência]]
- [[10 - Observabilidade]]

### Projeto prático sugerido
> Evolua o projeto da Fase 2:
> 1. Dockerize a aplicação (crie um `Dockerfile`)
> 2. Adicione o build da imagem Docker ao pipeline CI
> 3. Faça deploy da imagem em um ambiente (pode ser Fly.io, Render, ou um VPS)
> 4. Configure logs centralizados

### Marco de conclusão ✅
> Você tem uma aplicação containerizada, com pipeline CI/CD completo e deploy automatizado.

---

## 🟥 FASE 4 — Nível Avançado
> Duração estimada: **4 a 6 semanas**
> Pré-requisitos: Fases 1, 2 e 3 completas

### Objetivos
- [ ] Aprofundar Kubernetes (HPA, namespaces, Helm)
- [ ] Implementar estratégias avançadas de deploy (blue-green, canary)
- [ ] Implementar GitOps com ArgoCD ou Flux
- [ ] Segurança no pipeline (SAST, DAST, scanning de imagens)
- [ ] Chaos Engineering básico
- [ ] SRE e métricas de confiabilidade (SLI/SLO/SLA)

### Ferramentas a aprender
```
Kubernetes avançado ─── HPA, Helm, namespaces
ArgoCD / Flux ─────────── GitOps
Trivy / Snyk ──────────── segurança no pipeline
Jaeger / OpenTelemetry ── tracing distribuído
```

### Marco de conclusão ✅
> Você consegue fazer um deploy zero-downtime com rollback automático, tem observabilidade completa (logs, métricas, traces) e segurança integrada ao pipeline.

---

## 📚 Recursos de Estudo

### Livros
- **The Phoenix Project** — Gene Kim *(leitura obrigatória, narrativa fácil)*
- **The DevOps Handbook** — Gene Kim et al.
- **Site Reliability Engineering** — Google (disponível online grátis)
- **Accelerate** — Nicole Forsgren *(baseado em dados reais sobre DevOps)*

### Cursos e Plataformas
- KodeKloud — prático, labs hands-on
- Linux Foundation (LFS258 para CKA)
- A Cloud Guru / Pluralsight
- YouTube: TechWorld with Nana (excelente para Kubernetes)

### Certificações (em ordem de mercado)
```
1. CKA  — Certified Kubernetes Administrator
2. AWS  — Solutions Architect Associate
3. CKAD — Certified Kubernetes Application Developer
4. GCP  — Professional Cloud DevOps Engineer
5. HashiCorp — Terraform Associate
```

---

## 🔧 Ferramentas por Categoria

### Source Control
`Git` · `GitHub` · `GitLab` · `Bitbucket`

### CI/CD
`GitHub Actions` · `GitLab CI` · `Jenkins` · `CircleCI` · `ArgoCD`

### Containers
`Docker` · `Kubernetes` · `Helm` · `Docker Compose`

### IaC
`Terraform` · `Ansible` · `Pulumi` · `CloudFormation`

### Cloud
`AWS` · `GCP` · `Azure` · `DigitalOcean` · `Fly.io`

### Observabilidade
`Prometheus` · `Grafana` · `Loki` · `Jaeger` · `Datadog` · `ELK Stack`

### Segurança
`Trivy` · `Snyk` · `SonarQube` · `OWASP ZAP`

---

## Checklist Final do Profissional DevOps Júnior

> [!success] Você está pronto para o mercado quando:
> - [ ] Consegue criar um pipeline CI/CD do zero
> - [ ] Sabe Dockerizar uma aplicação
> - [ ] Entende como funciona o Kubernetes (pods, services, deployments)
> - [ ] Sabe provisionar infra básica com Terraform
> - [ ] Consegue ler logs e métricas para investigar um problema
> - [ ] Conhece os conceitos de HA, resiliência e observabilidade
> - [ ] Tem projetos práticos no GitHub demonstrando essas habilidades

---

## Links relacionados

- [[🗺️ MOC — DevOps]] — mapa completo de conteúdo
- [[01 - O que é DevOps]] — começo do estudo
- [[11 - O Fluxo Completo DevOps]] — visão sistêmica

