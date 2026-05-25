# 05 — Mapa de Papéis por Tamanho de Time

tags: #papéis #estrutura #times
links: [[06 - Como Montar um Time do Zero]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## A regra dos papéis mínimos

Todo time de software, independente do tamanho, precisa garantir que três perguntas tenham um dono:

| Pergunta | Papel responsável |
|---|---|
| **O que construir?** | Product Owner |
| **Como construir?** | Tech Lead |
| **O time está funcionando?** | Scrum Master / Coordenador |

Em times pequenos, uma pessoa pode acumular papéis — mas nunca deixe uma pergunta sem dono.

---

## Configurações por tamanho

```mermaid
flowchart TD
    subgraph T2["2 pessoas"]
        P1A["Pessoa 1
        Dev + Tech Lead"]
        P2A["Pessoa 2
        Dev + PO"]
    end

    subgraph T4["3–4 pessoas"]
        P1B["Pessoa 1
        Tech Lead + Dev"]
        P2B["Pessoa 2
        PO + Coordenador"]
        P3B["Pessoa 3–4
        Devs"]
    end

    subgraph T8["5–8 pessoas"]
        P1C["Tech Lead"]
        P2C["Product Owner"]
        P3C["Scrum Master"]
        P4C["Devs Sênior/Pleno/Júnior"]
    end

    subgraph T15["10+ pessoas"]
        P1D["Engineering Manager"]
        P2D["Tech Lead por squad"]
        P3D["PO por produto"]
        P4D["Scrum Master dedicado"]
        P5D["Devs especializados"]
    end

    style T2 fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style T4 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style T8 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style T15 fill:#FAEEDA,stroke:#854F0B,color:#633806
```

---

## Para times acadêmicos (2–4 pessoas)

A configuração mais eficiente:

```
Coordenador do grupo
├── Acumula: PO + Scrum Master + Dev
├── Foco: requisitos, comunicação com professor, rituais
└── Coda menos, facilita mais

Dev mais experiente
├── Acumula: Tech Lead + Dev Sênior
├── Foco: arquitetura, code review, padrões
└── Tem voz final em decisões técnicas

Demais membros
├── Papel: Dev Pleno ou Júnior
└── Foco: implementação das user stories
```

---

## Quando os papéis ficam difusos

Sinal de alerta: quando ninguém sabe quem decide o quê.

**Sintomas:**
- Decisões técnicas são tomadas pelo coordenador (que não é tech lead)
- O dev mais vocal vira PO informal sem ter visão do usuário
- Ninguém facilita os rituais — as reuniões viram bagunça
- Conflitos não são resolvidos porque não há autoridade clara

**Solução:** reunião de 30 minutos para declarar explicitamente quem é responsável por cada pergunta. Use o [[Template - Definição de Papéis do Time]].

---

## Próximas notas
- [[06 - Como Montar um Time do Zero]]
- [[Template - Definição de Papéis do Time]]
