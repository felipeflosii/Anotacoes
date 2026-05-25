# 16 — Divisão de Responsabilidades

tags: #time #organização
links: [[17 - Rituais de Time]] | [[19 - Contrato de API]]

---

## Divisão por camada (Clean Architecture)

| Pessoa | Responsabilidade | Entregáveis |
|---|---|---|
| **Dev 1** | Domínio + Casos de uso | Entidades, interfaces, use-cases |
| **Dev 2** | Infraestrutura + Banco | Repositórios, migrations, queries |
| **Dev 3** | Frontend | Telas, componentes, consumo de API |
| **Coordenador** | Controllers + Integração | Rotas, DTOs, revisão de PRs |

---

## Por que dividir por camada

Cada pessoa trabalha em arquivos diferentes — conflitos de merge são raros. E cada um desenvolve profundidade em uma área, sem bloquear os outros.

---

## O contrato que une o time

Antes de codar, frontend e backend combinam os endpoints. Isso permite trabalho paralelo desde o dia 1.

```
GET  /users/:id         → retorna dados do usuário
POST /users             → cria novo usuário
POST /auth/login        → autentica e retorna JWT
GET  /orders?userId=X   → lista pedidos do usuário
```

Veja detalhes em [[19 - Contrato de API]].

---

## Rotação de responsabilidades

Em projetos longos, considere rodar as responsabilidades a cada sprint. Isso distribui conhecimento e evita que alguém seja o único que entende uma parte do sistema (*bus factor*).

---

## Próximas notas
- [[17 - Rituais de Time]]
- [[19 - Contrato de API]]
