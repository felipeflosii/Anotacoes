# 19 — Contrato de API

tags: #time #api #processo
links: [[16 - Divisão de Responsabilidades]] | [[11 - Estrutura de Pastas Clean Architecture]]

---

## O que é e por que importa

O contrato de API é um documento que define os endpoints *antes* de qualquer código ser escrito. Ele permite que frontend e backend trabalhem em **paralelo** desde o primeiro dia.

Sem contrato: o frontend fica bloqueado esperando o backend ficar pronto.
Com contrato: frontend mocka os dados e desenvolve independente.

---

## Formato mínimo (Markdown)

```markdown
## Endpoints da API — v1

Base URL: /api/v1

---

### Autenticação

POST /auth/login
Body:    { email: string, password: string }
200 OK:  { token: string, user: { id, name, email } }
401:     { error: "Credenciais inválidas" }

POST /auth/register
Body:    { name: string, email: string, password: string }
201:     { token: string, user: { id, name, email } }
409:     { error: "E-mail já cadastrado" }

---

### Usuários

GET /users/:id
Header:  Authorization: Bearer <token>
200 OK:  { id, name, email, createdAt }
404:     { error: "Usuário não encontrado" }

---

### Pedidos

GET /orders?userId=:id&page=1&limit=20
200 OK:  { data: Order[], total: number, page: number }

POST /orders
Body:    { userId: string, items: [{ productId, qty }] }
201:     { id, status: "pending", total: number }
```

---

## Como o frontend mocka enquanto o backend não está pronto

```javascript
// mock durante desenvolvimento
const api = {
  login: async (email, password) => {
    // simula delay de rede
    await new Promise(r => setTimeout(r, 300))
    return { token: 'fake-jwt', user: { id: '1', name: 'Dev User', email } }
  }
}

// quando o backend ficar pronto, troca para:
const api = {
  login: async (email, password) => {
    const res = await fetch('/api/v1/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    })
    return res.json()
  }
}
```

O resto do código frontend não muda nada.

---

## Formato avançado: OpenAPI (Swagger)

Para projetos maiores, defina o contrato em OpenAPI — gera documentação interativa automaticamente:

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: API do Projeto
  version: 1.0.0
paths:
  /auth/login:
    post:
      summary: Autenticar usuário
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                email: { type: string }
                password: { type: string }
      responses:
        '200':
          description: Token JWT retornado
```

Com FastAPI (Python), isso é gerado automaticamente. Com Express, use o pacote `swagger-jsdoc`.

---

## Próximas notas
- [[16 - Divisão de Responsabilidades]]
- [[17 - Rituais de Time]]
