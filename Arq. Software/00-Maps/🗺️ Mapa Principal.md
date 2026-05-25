# 🗺️ Mapa Principal — Arquitetura de Software

> Vault de estudo para coordenadores de projetos acadêmicos.
> Siga o fluxo numerado ou navegue pelos mapas temáticos.

---

## Fluxo do processo completo

```mermaid
flowchart TD
    A([💡 Ideia]) --> B[Entender o Problema]
    B --> C[Definir Requisitos]
    C --> D{Tipo de projeto?}

    D -->|Simples / Curto prazo| E[MVC]
    D -->|Regras complexas| F[Clean Architecture]
    D -->|Múltiplos clientes| G[Hexagonal]

    E --> H[Escolher Stack]
    F --> H
    G --> H

    H --> I[Planejar o Time]
    I --> J([🚀 Primeiro commit])

    style A fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style J fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D fill:#FAEEDA,stroke:#854F0B,color:#633806
    style E fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style F fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style G fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

---

## Índice de notas

### Fundamentos
- [[01 - Por que Arquitetura Existe]]
- [[02 - MVC]]
- [[03 - Clean Architecture]]
- [[04 - Arquitetura Hexagonal]]
- [[05 - Comparativo de Arquiteturas]]

### Requisitos
- [[06 - Entendendo o Problema]]
- [[07 - Requisitos Funcionais e Não-Funcionais]]
- [[08 - User Stories]]
- [[09 - Priorização com MoSCoW]]

### Arquitetura
- [[10 - Árvore de Decisão de Arquitetura]]
- [[11 - Estrutura de Pastas Clean Architecture]]
- [[12 - Regra de Dependência]]

### Tecnologias
- [[13 - Como Escolher Tecnologias]]
- [[14 - Stack Recomendada para Projetos Acadêmicos]]
- [[15 - Quando Adicionar Ferramentas]]

### Time
- [[16 - Divisão de Responsabilidades]]
- [[17 - Rituais de Time]]
- [[18 - Lidando com Requisitos que Mudam]]
- [[19 - Contrato de API]]

### Templates
- [[Template - Documento de Visão]]
- [[Template - User Story]]
- [[Template - Decisão de Arquitetura]]

---

## Mapa de conceitos relacionados

```mermaid
mindmap
  root((Arquitetura))
    Padrões
      MVC
      Clean Architecture
      Hexagonal
      Microsserviços
    Requisitos
      Funcionais
      Não-funcionais
      User Stories
      MoSCoW
    Time
      Divisão de camadas
      Rituais
      Code review
      Contrato de API
    Tecnologias
      Backend
      Frontend
      Banco de dados
      Deploy
```

