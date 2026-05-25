# Template — Matriz RACI

tags: #template #papéis #responsabilidade
links: [[05 - Mapa de Papéis por Tamanho de Time]] | [[Template - Definição de Papéis do Time]]

---

## O que é RACI

Matriz que define o nível de envolvimento de cada pessoa em cada atividade do projeto.

| Letra | Significado | Quantas por atividade |
|---|---|---|
| **R** — Responsible | Quem executa a tarefa | 1 ou mais |
| **A** — Accountable | Quem responde pelo resultado final | Exatamente 1 |
| **C** — Consulted | Quem deve ser consultado antes | 0 ou mais |
| **I** — Informed | Quem deve ser informado depois | 0 ou mais |

> Toda atividade precisa de exatamente 1 **A**. Sem isso, ninguém é responsável de verdade.

---

## Matriz preenchida — exemplo

| Atividade | Coord. | Tech Lead | PO | Dev 1 | Dev 2 |
|---|---|---|---|---|---|
| Definir requisitos | A | C | R | I | I |
| Decisão de arquitetura | I | A | C | R | C |
| Escrever user stories | C | I | A/R | I | I |
| Code review | I | A | — | R | R |
| Deploy em produção | A | R | I | C | — |
| Facilitar daily | R/A | I | I | I | I |
| Comunicação com professor | A/R | C | C | I | I |
| Aceitar entrega do sprint | C | C | A/R | I | I |

---

## Sua matriz — preencha aqui

| Atividade | [Pessoa 1] | [Pessoa 2] | [Pessoa 3] | [Pessoa 4] |
|---|---|---|---|---|
| Definir requisitos | | | | |
| Decisão de arquitetura | | | | |
| Escrever user stories | | | | |
| Code review | | | | |
| Deploy | | | | |
| Facilitar daily | | | | |
| Comunicação externa | | | | |
| Aceitar entrega do sprint | | | | |
| Onboarding de novos membros | | | | |
| Documentação | | | | |

---

## Sinais de RACI problemático

- Mais de um **A** na mesma atividade → ninguém é realmente responsável
- Nenhum **A** em uma atividade → tarefa órfã, vai cair no esquecimento
- Uma pessoa com **A** em tudo → centralização, gargalo
- **R** sem **A** → execução sem accountability
