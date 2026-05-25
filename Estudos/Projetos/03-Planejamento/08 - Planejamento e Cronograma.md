# 08 — Planejamento e Cronograma

tags: #planejamento #cronograma #projeto #tempo
links: [[07 - Montar o Time]] | [[09 - Estrutura Financeira]] | [[13 - Gestão de Tarefas e Sprints]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Por que cronogramas falham

A maioria dos cronogramas falha por três razões:
1. **Estimativas otimistas** — tempo estimado no melhor cenário possível
2. **Sem folga** — qualquer imprevisto quebra tudo
3. **Nível errado de detalhe** — ou detalha demais o início (que está claro) ou deixa o fim vago (que é onde os problemas ficam)

A solução não é planejar mais — é planejar diferente.

---

## O fluxo de construção do cronograma

```mermaid
flowchart LR
    A[Lista de entregas\ndo escopo] --> B[Quebrar em tarefas\nmenores]
    B --> C[Estimar cada tarefa\nem horas]
    C --> D[Agrupar em sprints\nou fases]
    D --> E[Mapear no calendário\nreal]
    E --> F[Adicionar folga\nde 20%]
    F --> G([Cronograma pronto])

    style A fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style G fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

---

## Passo 1 — Quebre as entregas em tarefas

Toda entrega grande precisa ser quebrada em tarefas de **no máximo 4 horas**. Se uma tarefa não cabe em 4 horas, ela ainda está grande demais — quebre mais.

```
Entrega: "Protótipo do app"
❌ Muito grande — uma semana de trabalho não sabe se está andando

Tarefas:
✅ Desenhar tela de login no papel (2h)
✅ Digitalizar tela de login no Figma (3h)
✅ Desenhar tela principal no papel (2h)
✅ Digitalizar tela principal no Figma (3h)
✅ Conectar as telas em fluxo clicável (2h)
✅ Teste com 3 pessoas e anotações de feedback (3h)
```

---

## Passo 2 — Estime com honestidade

Use a regra dos **três pontos**:

| Estimativa | O que significa | Fator |
|---|---|---|
| Otimista (O) | Tudo corre perfeitamente | Base |
| Provável (P) | Cenário realista com pequenos imprevistos | ×1,5 |
| Pessimista (Pe) | Algo dá errado, mas não catastrófico | ×2 |

**Fórmula sugerida:** `(O + 4P + Pe) / 6`

```
Exemplo — "Desenvolver tela de login":
Otimista:   3h (sei exatamente o que fazer)
Provável:   5h (algumas dúvidas técnicas)
Pessimista: 9h (tecnologia travou, precisei de ajuda)

Estimativa: (3 + 4×5 + 9) / 6 = (3 + 20 + 9) / 6 = 32 / 6 = ~5,3h
```

---

## Passo 3 — Organize em fases ou sprints

### Modelo por fases (projetos lineares)

```
Fase 1 — Fundação (semanas 1-2)
  Pesquisa, validação, escopo, setup inicial

Fase 2 — Construção (semanas 3-6)
  Entregáveis principais

Fase 3 — Refinamento (semanas 7-8)
  Testes, ajustes, documentação

Fase 4 — Entrega (semanas 9-10)
  Preparação da apresentação, entrega final
```

### Modelo por sprints (projetos iterativos — recomendado)

```
Sprint 1 (2 semanas) — [Objetivo da sprint]
  Tarefa A — Responsável: X — Prazo: DD/MM
  Tarefa B — Responsável: Y — Prazo: DD/MM
  Tarefa C — Responsável: X — Prazo: DD/MM
  Review: DD/MM

Sprint 2 (2 semanas) — [Objetivo da sprint]
  ...
```

**Por que sprints?** Porque você revisa o que funcionou e ajusta o plano a cada ciclo. Projetos longos sem pontos de revisão derivam silenciosamente.

---

## Passo 4 — Mapeie no calendário real

Coloque as tarefas no calendário considerando:

```
❌ Não calcule capacidade com 8h/dia, 5 dias/semana
✅ Use a capacidade real disponível

Exemplo (universitário):
- Disponibilidade total: 4h/dia útil
- Aulas e obrigações: -2h
- Capacidade real no projeto: 2h/dia útil = 10h/semana

Com 3 pessoas → 30h/semana de trabalho no projeto
```

---

## Passo 5 — Adicione a folga de 20%

Toda estimativa está errada. A folga não é sinal de ineficiência — é sinal de maturidade.

```
Total estimado:    80 horas
Folga de 20%:    + 16 horas
Total planejado:   96 horas

Se você entregar em 80h, ótimo — você está adiantado.
Se aparecer um imprevisto de 10h, ainda entrega no prazo.
```

---

## Template de cronograma simplificado

```markdown
## Cronograma — [Nome do Projeto]

**Início:** DD/MM  |  **Entrega final:** DD/MM  |  **Duração total:** X semanas

### Sprint 1 — [DD/MM até DD/MM] — Objetivo: ___________
| Tarefa | Responsável | Prazo | Status |
|--------|-------------|-------|--------|
| [tarefa] | [pessoa] | DD/MM | ⬜ Pendente |

### Sprint 2 — [DD/MM até DD/MM] — Objetivo: ___________
| Tarefa | Responsável | Prazo | Status |
|--------|-------------|-------|--------|

### Marcos importantes
| Marco | Data |
|-------|------|
| Escopo aprovado | DD/MM |
| Protótipo pronto | DD/MM |
| Versão beta testada | DD/MM |
| Entrega final | DD/MM |
| Apresentação | DD/MM |
```

---

## Próximas notas
- [[09 - Estrutura Financeira]] — custos e orçamento do projeto
- [[13 - Gestão de Tarefas e Sprints]] — executar o cronograma
