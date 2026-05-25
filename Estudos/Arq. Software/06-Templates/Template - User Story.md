# Template — User Story

tags: #template
links: [[08 - User Stories]] | [[09 - Priorização com MoSCoW]]

---

> Use uma nota por user story. Nomeie como: `US-001 - [título curto]`

---

## Identificação

**ID:** US-000  
**Título:**  
**Prioridade:** `Must Have` / `Should Have` / `Could Have` / `Won't Have`  
**Sprint:**  
**Responsável:**  
**Status:** `Backlog` / `Em progresso` / `Em revisão` / `Concluído`

---

## A story

```
Como [tipo de usuário],
quero [ação],
para que [benefício].
```

---

## Critérios de aceitação

### Cenário 1 — Caminho feliz
```
Dado que [contexto],
quando [ação do usuário],
então [resultado esperado].
```

### Cenário 2 — Caso de erro
```
Dado que [contexto com dados inválidos ou edge case],
quando [ação do usuário],
então [mensagem de erro ou comportamento esperado].
```

### Cenário 3 — (adicione se necessário)
```
Dado que [contexto],
quando [ação],
então [resultado].
```

---

## Notas técnicas

> Observações de implementação, links para decisões de arquitetura, dependências de outras stories.

- Depende de: 
- Decisão de arquitetura relacionada: 
- Observações: 

---

## Definição de pronto (DoD)

- [ ] Código implementado e funcionando localmente
- [ ] Testes unitários escritos e passando
- [ ] PR aberto e revisado por pelo menos 1 pessoa
- [ ] Merge feito na branch `main`
- [ ] Critérios de aceitação validados manualmente
