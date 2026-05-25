# 05 — Comparativo de Arquiteturas

tags: #fundamentos #arquitetura #comparativo
links: [[02 - MVC]] | [[03 - Clean Architecture]] | [[04 - Arquitetura Hexagonal]] | [[10 - Árvore de Decisão de Arquitetura]]

---

## Visão geral

```mermaid
quadrantChart
    title Complexidade vs Flexibilidade
    x-axis Simples --> Complexo
    y-axis Rígido --> Flexível
    quadrant-1 Ideal para escala
    quadrant-2 Over-engineering
    quadrant-3 Ponto de partida
    quadrant-4 Equilibrado
    MVC: [0.2, 0.25]
    Clean Architecture: [0.6, 0.8]
    Hexagonal: [0.65, 0.85]
    Microsserviços: [0.9, 0.9]
```

---

## Tabela comparativa

| Critério | MVC | Clean Architecture | Hexagonal |
|---|---|---|---|
| **Curva de aprendizado** | Baixa | Média | Média |
| **Testabilidade** | Média | Alta | Alta |
| **Flexibilidade** | Baixa | Alta | Alta |
| **Overhead inicial** | Baixo | Médio | Médio |
| **Bom para** | MVPs, APIs simples | Sistemas com regras complexas | Sistemas com múltiplos clientes |
| **Frameworks** | Laravel, Django, Rails | NestJS, Spring | Qualquer um |

---

## Quando usar cada um

```mermaid
flowchart TD
    A{Qual o prazo?} -->|Menos de 1 mês| B[MVC]
    A -->|Mais de 1 mês| C{Regras de negócio complexas?}
    C -->|Não| B
    C -->|Sim| D{Múltiplos clientes / canais?}
    D -->|Não| E[Clean Architecture]
    D -->|Sim| F[Hexagonal]
    E -.->|pode evoluir para| F

    style B fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

---

## Para projetos acadêmicos (2-4 pessoas)

| Duração | Recomendação |
|---|---|
| Até 4 semanas | MVC simples |
| 1-3 meses | MVC bem estruturado ou Clean Architecture leve |
| 3+ meses | Clean Architecture completa |

> **Regra prática:** nunca escolha uma arquitetura que você não consegue explicar para o time em 10 minutos. Arquitetura que só você entende é pior do que MVC simples.
