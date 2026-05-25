# 🔗 Mapa de Conexões — Gestão de Times

tags: #moc #grafo
links: [[Estudos/Gest. Pessoas/00-Maps/🗺️ Mapa Principal]]

---

> Esta nota cria conexões explícitas entre todas as notas do vault para o Graph View do Obsidian.

---

## Fluxo completo

```mermaid
flowchart TD
    INICIO([Novo projeto ou time]) --> N01[[01 - Tech Lead]]
    INICIO --> N02[[02 - Product Owner]]
    INICIO --> N03[[03 - Perfis de Devs]]
    N01 & N02 & N03 --> N05[[05 - Mapa de Papéis]]

    N05 --> N06[[06 - Montar Time do Zero]]
    N06 --> N07[[07 - Camada vs Feature]]
    N06 --> N08[[08 - Bus Factor]]
    N06 --> N09[[09 - Contratar ou Realocar]]

    N06 --> N10[[10 - Canais de Comunicação]]
    N10 --> N11[[11 - Reuniões]]
    N11 --> N12[[12 - Feedback]]
    N12 --> N13[[13 - Conflitos]]

    N06 --> N14[[14 - Velocity e Burndown]]
    N14 --> N15[[15 - Lead e Cycle Time]]
    N15 --> N16[[16 - Qualidade de Código]]
    N16 --> N17[[17 - Métricas sem Pressão]]

    N06 --> N18[[18 - Por que Onboarding]]
    N18 --> N19[[19 - Plano 30 Dias]]
    N19 --> N20[[20 - Documentação]]

    N17 & N13 & N20 --> FIM([Time autônomo e produtivo])

    style INICIO fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style FIM fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

---

## Mapa temático — Papéis

```mermaid
graph LR
    N01[[Tech Lead]] --- N03[[Perfis de Devs]]
    N01 --- N04[[Scrum Master vs Coord.]]
    N02[[Product Owner]] --- N04
    N03 --- N05[[Mapa de Papéis]]
    N04 --- N05
    N05 --- N06[[Montar Time]]
    N05 --- TRACI[[RACI]]
    N05 --- TPAPEIS[[Template Papéis]]
```

---

## Mapa temático — Comunicação

```mermaid
graph LR
    N10[[Canais]] --- N11[[Reuniões]]
    N11 --- N12[[Feedback]]
    N12 --- N13[[Conflitos]]
    N12 --- TFB[[Template 1-1]]
    N11 --- N17[[Métricas sem Pressão]]
```

---

## Mapa temático — Onboarding

```mermaid
graph LR
    N18[[Por que Onboarding]] --- N19[[Plano 30 Dias]]
    N19 --- N20[[Documentação]]
    N19 --- TON[[Template Onboarding]]
    N20 --- N08[[Bus Factor]]
```

---

## Todos os links do vault

[[01 - O que é um Tech Lead]]
[[02 - O que é um Product Owner]]
[[03 - Perfis de Desenvolvedores]]
[[04 - Scrum Master vs Coordenador de Projeto]]
[[05 - Mapa de Papéis por Tamanho de Time]]
[[06 - Como Montar um Time do Zero]]
[[07 - Divisão por Camada vs por Feature]]
[[08 - Bus Factor e Conhecimento Compartilhado]]
[[09 - Quando Contratar ou Realocar]]
[[10 - Canais de Comunicação e Quando Usar]]
[[11 - Reuniões que Funcionam]]
[[12 - Como Dar e Receber Feedback]]
[[13 - Gestão de Conflitos no Time]]
[[14 - Velocity e Burndown]]
[[15 - Lead Time e Cycle Time]]
[[16 - Métricas de Qualidade de Código]]
[[17 - Como Usar Métricas sem Criar Pressão Tóxica]]
[[18 - Por que Onboarding Importa]]
[[19 - Plano de Onboarding em 30 Dias]]
[[20 - Documentação que o Time Realmente Lê]]
[[Template - Plano de Onboarding]]
[[Template - Feedback 1-1]]
[[Template - Definição de Papéis do Time]]
[[Template - Matriz RACI]]
