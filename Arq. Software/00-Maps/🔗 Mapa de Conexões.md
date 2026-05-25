# 🔗 Mapa de Conexões — Grafo do Vault

tags: #moc #grafo
links: [[🗺️ Mapa Principal]]

---

> Esta nota existe para criar conexões explícitas entre todas as notas do vault.
> O Obsidian Graph View vai mostrar o grafo completo ao abrir esta nota.

---

## Fluxo principal (da ideia ao código)

```mermaid
flowchart TD
    IDEIA([Ideia de projeto]) --> E06[[06 - Entendendo o Problema]]
    E06 --> E07[[07 - Req. Funcionais e NF]]
    E07 --> E08[[08 - User Stories]]
    E08 --> E09[[09 - MoSCoW]]
    E09 --> E10[[10 - Árvore de Decisão]]

    E10 --> MVC[[02 - MVC]]
    E10 --> CLEAN[[03 - Clean Architecture]]
    E10 --> HEXA[[04 - Arquitetura Hexagonal]]

    CLEAN --> E11[[11 - Estrutura de Pastas]]
    CLEAN --> E12[[12 - Regra de Dependência]]

    E10 --> E13[[13 - Como Escolher Tech]]
    E13 --> E14[[14 - Stack Recomendada]]
    E13 --> E15[[15 - Quando Adicionar Ferramentas]]

    E14 --> E16[[16 - Divisão de Responsabilidades]]
    E16 --> E17[[17 - Rituais de Time]]
    E16 --> E19[[19 - Contrato de API]]
    E17 --> E18[[18 - Req. que Mudam]]

    E19 --> COMMIT([🚀 Primeiro commit])

    style IDEIA fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style COMMIT fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style CLEAN fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style HEXA fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style MVC fill:#F1EFE8,stroke:#5F5E5A,color:#444441
```

---

## Mapa temático — Arquitetura

```mermaid
graph LR
    E01[[01 - Por que Arquitetura]] --- E02[[02 - MVC]]
    E01 --- CLEAN[[03 - Clean Architecture]]
    E01 --- HEXA[[04 - Arquitetura Hexagonal]]
    E02 --- E05[[05 - Comparativo]]
    CLEAN --- E05
    HEXA --- E05
    CLEAN --- E11[[11 - Pastas]]
    CLEAN --- E12[[12 - Regra de Dep.]]
    E05 --- E10[[10 - Árvore de Decisão]]
```

---

## Mapa temático — Requisitos

```mermaid
graph LR
    E06[[06 - Problema]] --- E07[[07 - RF e RNF]]
    E07 --- E08[[08 - User Stories]]
    E08 --- E09[[09 - MoSCoW]]
    E09 --- E18[[18 - Req. que Mudam]]
    E06 --- TDV[[Template - Visão]]
    E08 --- TUS[[Template - User Story]]
```

---

## Mapa temático — Time e processo

```mermaid
graph LR
    E16[[16 - Divisão]] --- E17[[17 - Rituais]]
    E17 --- E18[[18 - Req. Mudam]]
    E16 --- E19[[19 - Contrato API]]
    E19 --- E11[[11 - Pastas]]
    E17 --- TADR[[Template - ADR]]
```

---

## Links para todos os nós (para o grafo do Obsidian)

[[01 - Por que Arquitetura Existe]]
[[02 - MVC]]
[[03 - Clean Architecture]]
[[04 - Arquitetura Hexagonal]]
[[05 - Comparativo de Arquiteturas]]
[[06 - Entendendo o Problema]]
[[07 - Requisitos Funcionais e Não-Funcionais]]
[[08 - User Stories]]
[[09 - Priorização com MoSCoW]]
[[10 - Árvore de Decisão de Arquitetura]]
[[11 - Estrutura de Pastas Clean Architecture]]
[[12 - Regra de Dependência]]
[[13 - Como Escolher Tecnologias]]
[[14 - Stack Recomendada para Projetos Acadêmicos]]
[[15 - Quando Adicionar Ferramentas]]
[[16 - Divisão de Responsabilidades]]
[[17 - Rituais de Time]]
[[18 - Lidando com Requisitos que Mudam]]
[[19 - Contrato de API]]
[[Template - Documento de Visão]]
[[Template - User Story]]
[[Template - Decisão de Arquitetura]]
