# 39 — Padrão REST Completo

tags: #springboot #rest #boaspraticas #api
links: [[40 - Versionamento de API]] | [[41 - Paginação e Ordenação]] | [[42 - Filtros e Idempotência]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Os pilares de uma API REST bem projetada

```
1. URLs expressivas e padronizadas
2. Uso correto dos métodos HTTP
3. Status codes corretos
4. Respostas consistentes
5. Paginação em listas
6. Versionamento
7. Documentação
```

---

## URLs — nomenclatura definitiva

```
✅ CORRETO:
GET    /api/v1/clientes             listar clientes
GET    /api/v1/clientes/42          buscar cliente por id
POST   /api/v1/clientes             criar cliente
PUT    /api/v1/clientes/42          atualizar cliente completo
PATCH  /api/v1/clientes/42          atualizar campos específicos
DELETE /api/v1/clientes/42          deletar cliente
GET    /api/v1/clientes/42/pedidos  pedidos do cliente 42
POST   /api/v1/clientes/42/pedidos  criar pedido para cliente 42

❌ ERRADO — verbos na URL:
GET  /api/v1/buscarCliente?id=42
POST /api/v1/criarCliente
GET  /api/v1/cliente/listarTodos
POST /api/v1/deletarCliente/42
POST /api/v1/cliente/atualizar

❌ ERRADO — singular:
/api/v1/cliente      (deveria ser /clientes)
/api/v1/produto/42   (deveria ser /produtos/42)

❌ ERRADO — camelCase ou underscores:
/api/v1/minhsListaDeClientes
/api/v1/minha_lista_de_clientes  (use hífen se precisar separar)

✅ Se precisar separar palavras: use hífen
/api/v1/categorias-produto
/api/v1/itens-pedido
```

---

## Ações que não se encaixam no CRUD

Às vezes você precisa representar ações que não são simples CRUD:

```
# Ações de estado — use PATCH com sub-recurso descritivo
PATCH /api/v1/pedidos/10/cancelar
PATCH /api/v1/pedidos/10/confirmar
PATCH /api/v1/usuarios/5/ativar
PATCH /api/v1/usuarios/5/desativar

# Ações mais complexas — POST com verbo no path (aceitável como exceção)
POST /api/v1/pedidos/10/reprocessar
POST /api/v1/relatorios/vendas/gerar
POST /api/v1/emails/boas-vindas/enviar

# Buscas especiais — use query params
GET /api/v1/produtos?nome=teclado&categoria=perifericos
GET /api/v1/pedidos?status=PENDENTE&dataInicio=2024-01-01
```

---

## Respostas — o que retornar em cada situação

```java
// POST — criar → 201 + Location + corpo
@PostMapping
public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest req) {
    var response = service.criar(req);
    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
        .path("/{id}").buildAndExpand(response.id()).toUri();
    return ResponseEntity.created(location).body(response);
}
// HTTP 201 Created
// Location: /api/v1/clientes/42
// Body: { "id": 42, "nome": "Felipe", ... }

// GET → 200 + corpo
// PUT/PATCH → 200 + corpo atualizado
// DELETE → 204 (sem corpo)

// Recurso não encontrado → 404 + corpo de erro
// Dados inválidos → 400 + lista de campos inválidos
// Não autenticado → 401
// Sem permissão → 403
// Conflito (duplicata) → 409
// Regra de negócio → 422
// Erro interno → 500 (sem detalhes internos)
```

---

## Paginação — padrão de resposta

```json
// Toda listagem deve ser paginada
// Retorne Page<T> — o Spring serializa automaticamente assim:
{
  "content": [
    { "id": 1, "nome": "Produto A" },
    { "id": 2, "nome": "Produto B" }
  ],
  "pageable": {
    "sort": { "sorted": true, "orders": [{ "property": "nome", "direction": "ASC" }] },
    "pageNumber": 0,
    "pageSize": 20,
    "offset": 0
  },
  "totalElements": 150,
  "totalPages": 8,
  "last": false,
  "first": true,
  "numberOfElements": 20,
  "empty": false
}
```

---

## Headers importantes a incluir nas respostas

```java
@PostMapping
public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest req) {
    var response = service.criar(req);
    return ResponseEntity.status(201)
        // Location: onde encontrar o recurso criado
        .header(HttpHeaders.LOCATION, "/api/v1/clientes/" + response.id())
        // Cache-Control: dado muda frequentemente, não cachear
        .header(HttpHeaders.CACHE_CONTROL, "no-cache")
        .body(response);
}

@GetMapping("/{id}")
public ResponseEntity<ClienteResponse> buscar(@PathVariable Long id) {
    var response = service.buscarPorId(id);
    return ResponseEntity.ok()
        // Cache por 60 segundos — dado não muda tão rápido
        .header(HttpHeaders.CACHE_CONTROL, "max-age=60")
        .body(response);
}
```

---

## HATEOAS — links nos recursos (nível avançado)

HATEOAS (Hypermedia As The Engine Of Application State) adiciona links de navegação nos recursos:

```json
// Sem HATEOAS (comum e aceitável):
{ "id": 42, "nome": "Felipe", "email": "felipe@ex.com" }

// Com HATEOAS (REST puro — nível 3):
{
  "id": 42,
  "nome": "Felipe",
  "email": "felipe@ex.com",
  "_links": {
    "self":    { "href": "/api/v1/clientes/42" },
    "pedidos": { "href": "/api/v1/clientes/42/pedidos" },
    "update":  { "href": "/api/v1/clientes/42", "method": "PUT" },
    "delete":  { "href": "/api/v1/clientes/42", "method": "DELETE" }
  }
}
```

```xml
<!-- Para HATEOAS no Spring Boot: -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

> Na prática do mercado brasileiro, HATEOAS é raramente implementado. A maioria das APIs usa o nível 2 (recursos + métodos HTTP corretos) e é considerado suficiente.

---

## Checklist de uma boa API REST

```
✅ URLs com substantivos no plural
✅ Métodos HTTP corretos (GET, POST, PUT, PATCH, DELETE)
✅ Status codes corretos (201 para criação, 204 para deleção, etc.)
✅ Paginação em todas as listagens
✅ Location header no POST de criação
✅ Mensagens de erro padronizadas e sem detalhes internos
✅ Validação de entrada com mensagens claras
✅ Versionamento de URL (/api/v1/)
✅ Autenticação JWT em endpoints privados
✅ CORS configurado para o domínio correto
✅ Documentação Swagger atualizada
✅ Respostas consistentes em toda a API
```

---

## Próximas notas
- [[40 - Versionamento de API]] — como versionar sua API
- [[41 - Paginação e Ordenação]] — paginação em detalhe
- [[42 - Filtros e Idempotência]] — filtros e operações seguras
