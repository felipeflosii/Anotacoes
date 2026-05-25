# 🔗 Mapa de Conexões — Grafo do Vault

tags: #moc #grafo
links: [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

> Esta nota existe para criar conexões explícitas entre todas as notas do vault.
> O Obsidian Graph View vai mostrar o grafo completo ao abrir esta nota.

---

## Fluxo principal — da ideia ao projeto entregue

```mermaid
flowchart TD
    I01[[01 - Capturar a Ideia]] --> I02[[02 - Definir o Problema]]
    I02 --> I03[[03 - Pesquisa e Benchmark]]
    I03 --> V04[[04 - Validação]]
    V04 --> V05[[05 - Escopo e Requisitos]]
    V05 --> V06[[06 - Riscos e Premissas]]
    V06 --> P07[[07 - Montar o Time]]
    P07 --> P08[[08 - Cronograma]]
    P08 --> P09[[09 - Estrutura Financeira]]
    P09 --> PI10[[10 - Anatomia do Pitch]]
    PI10 --> PI11[[11 - Storytelling]]
    PI11 --> PI12[[12 - Apresentação Visual]]
    PI12 --> E13[[13 - Gestão de Tarefas]]
    E13 --> E14[[14 - Comunicação]]
    E14 --> E15[[15 - Bloqueios]]
    E15 --> EN16[[16 - Preparar Entrega]]
    EN16 --> EN17[[17 - Apresentação Final]]
    EN17 --> EN18[[18 - Retrospectiva]]

    style I01 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style EN18 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style PI10 fill:#FAECE7,stroke:#993C1D,color:#712B13
```

---

## Mapa temático — Ideação e Validação

```mermaid
graph LR
    I01[[01 - Ideia]] --- I02[[02 - Problema]]
    I02 --- I03[[03 - Benchmark]]
    I03 --- V04[[04 - Validação]]
    V04 --- V05[[05 - Escopo]]
    V05 --- V06[[06 - Riscos]]
    I01 --- TLC[[Template - Lean Canvas]]
    I02 --- TDV[[Template - Visão]]
```

---

## Mapa temático — Pitch

```mermaid
graph LR
    PI10[[10 - Pitch]] --- PI11[[11 - Storytelling]]
    PI11 --- PI12[[12 - Visual]]
    PI10 --- TPD[[Template - Pitch Deck]]
    PI11 --- TDV[[Template - Visão]]
```

---

## Mapa temático — Execução e Entrega

```mermaid
graph LR
    E13[[13 - Tarefas]] --- E14[[14 - Comunicação]]
    E14 --- E15[[15 - Bloqueios]]
    E15 --- EN16[[16 - Entrega]]
    EN16 --- EN17[[17 - Apresentação]]
    EN17 --- EN18[[18 - Retrospectiva]]
    E13 --- TC[[Template - Cronograma]]
    EN17 --- TA[[Template - Ata]]
```

---

## Links para todos os nós

[[01 - Capturar e Refinar a Ideia]]
[[02 - Definir o Problema Real]]
[[03 - Pesquisa e Benchmark]]
[[04 - Validação com Pessoas Reais]]
[[05 - Definir Escopo e Requisitos]]
[[06 - Riscos e Premissas]]
[[07 - Montar o Time]]
[[08 - Planejamento e Cronograma]]
[[09 - Estrutura Financeira]]
[[10 - Anatomia de um Pitch]]
[[11 - Storytelling do Projeto]]
[[12 - Apresentação Visual]]
[[13 - Gestão de Tarefas e Sprints]]
[[14 - Comunicação no Time]]
[[15 - Lidando com Bloqueios]]
[[16 - Preparar a Entrega]]
[[17 - Apresentação Final]]
[[18 - Retrospectiva e Aprendizados]]
[[Template - Documento de Visão do Projeto]]
[[Template - Lean Canvas]]
[[Template - Cronograma]]
[[Template - Pitch Deck]]
[[Template - Ata de Reunião]]
