# 01 — O que é um Tech Lead

tags: #papéis #tech-lead #liderança
links: [[02 - O que é um Product Owner]] | [[03 - Perfis de Desenvolvedores]] | [[🗺️ Mapa Principal]]

---

## Definição

O Tech Lead é o desenvolvedor mais sênior do time que assume responsabilidade técnica coletiva — não só pelo próprio código, mas pelo código de todos.

> Tech Lead não é um título de hierarquia. É uma função de responsabilidade.

---

## O que o Tech Lead faz (e o que não faz)

```mermaid
flowchart LR
    subgraph Faz["✅ Faz"]
        F1[Define padrões de código]
        F2[Revisa arquitetura das soluções]
        F3[Desbloqueia o time]
        F4[Faz code review crítico]
        F5[Comunica decisões técnicas]
        F6[Protege o time de escopo desnecessário]
    end

    subgraph NaoFaz["❌ Não faz"]
        N1[Resolve tudo sozinho]
        N2[Aprova cada linha de código]
        N3[Faz gestão de pessoas]
        N4[Define prazo sem consultar o time]
        N5[Substitui o Product Owner]
    end

    style Faz fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style NaoFaz fill:#FAECE7,stroke:#993C1D,color:#712B13
```

---

## A tensão central do Tech Lead

O Tech Lead vive entre dois mundos:

```mermaid
flowchart TD
    TL([Tech Lead])
    TL --> C[Código — ainda precisa codar]
    TL --> P[Pessoas — precisa desbloquear o time]
    C <-->|tensão| P
    P --> D[Decisões — precisa escolher e comunicar]
    C --> D
```

**A armadilha mais comum:** o Tech Lead que vira "super dev" — escreve 80% do código, o time fica dependente, e quando ele sai o projeto para.

**O equilíbrio saudável:** Tech Lead escreve ~40–60% do código que escrevia antes de assumir o papel. O resto é review, arquitetura, desbloqueio e comunicação.

---

## Habilidades de um bom Tech Lead

| Habilidade técnica | Habilidade de gestão |
|---|---|
| Arquitetura de sistemas | Comunicação clara |
| Code review construtivo | Escuta ativa |
| Decisão de stack e ferramentas | Gestão de expectativas |
| Identificar dívida técnica | Dar e receber feedback |
| Escrever documentação | Proteger o tempo do time |

---

## Sinais de que alguém está pronto para ser Tech Lead

- Outros devs pedem a opinião dele espontaneamente
- Ele consegue explicar decisões técnicas para não-técnicos
- Quando comete um erro, assume e propõe solução — não esconde
- Pensa em manutenibilidade e não só em "funcionar"
- Sente desconforto quando o time está bloqueado (não só quando ele está bloqueado)

---

## Sinais de que o Tech Lead está falhando

- O time não sabe o porquê das decisões técnicas
- PRs ficam dias sem revisão
- Devs júnior têm medo de fazer perguntas
- A arquitetura existe na cabeça de uma pessoa só
- O Tech Lead é o único que resolve bugs em produção

---

## Próximas notas
- [[02 - O que é um Product Owner]] — quem define o que vai ser construído
- [[05 - Mapa de Papéis por Tamanho de Time]] — quando cada papel é necessário
