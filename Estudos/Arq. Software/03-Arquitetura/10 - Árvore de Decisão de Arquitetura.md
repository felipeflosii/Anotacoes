# 10 — Árvore de Decisão de Arquitetura

tags: #arquitetura #decisão
links: [[05 - Comparativo de Arquiteturas]] | [[11 - Estrutura de Pastas Clean Architecture]]

---

## A árvore completa

```mermaid
flowchart TD
    A{Qual o prazo?} -->|Menos de 4 semanas| MVC[MVC simples]
    A -->|Mais de 1 mês| B{Regras de negócio\ncomplexa?}
    B -->|Não| MVC
    B -->|Sim| C{Múltiplos clientes\nou canais?}
    C -->|Não| Clean[Clean Architecture]
    C -->|Sim| Hexa[Hexagonal]
    Clean -.->|pode evoluir| Hexa

    A -->|Múltiplos times,\nescala grande| MS[Microsserviços]
    MS -->|⚠️ alto overhead| Aviso[Não comece aqui]

    style MVC fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style Clean fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Hexa fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style MS fill:#FAEEDA,stroke:#854F0B,color:#633806
    style Aviso fill:#FAECE7,stroke:#993C1D,color:#712B13
```

---

## Critérios de decisão resumidos

### Use MVC se...
- Prazo < 4 semanas
- Time está aprendendo
- Funcionalidades são CRUD simples
- Framework já impõe MVC (Laravel, Django, Rails)

### Use Clean Architecture se...
- Regras de negócio são complexas
- Precisa de cobertura de testes alta
- Sistema vai durar e evoluir por meses/anos
- Time tem experiência com injeção de dependências

### Use Hexagonal se...
- Mesmo que Clean Architecture, e adicionalmente:
- O sistema é acionado por múltiplos canais (HTTP, fila, CLI, cron)
- Vai trocar de banco ou serviços externos com frequência

### Não use Microsserviços se...
- Time tem menos de 6-8 pessoas
- Não há equipe de infraestrutura dedicada
- O projeto tem menos de 1 ano
- Você está em ambiente acadêmico

---

## Próximas notas
- [[11 - Estrutura de Pastas Clean Architecture]]
- [[13 - Como Escolher Tecnologias]]
