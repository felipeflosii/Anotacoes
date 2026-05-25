# 16 — @PathVariable e @RequestParam

tags: #springboot #controllers #parâmetros
links: [[14 - RestController e RequestMapping]] | [[15 - Métodos HTTP GET POST PUT DELETE]] | [[17 - RequestBody e ResponseEntity]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Os dois jeitos de passar dados via URL

```
GET /produtos/42                     ← PathVariable: id = 42
GET /produtos?categoria=eletronicos  ← RequestParam: categoria = "eletronicos"
GET /produtos/42/avaliacoes?nota=5   ← ambos juntos
```

---

## @PathVariable — variável no caminho da URL

Extrai um valor da URL que faz parte do **identificador do recurso**.

```java
// URL: GET /clientes/42
@GetMapping("/{id}")
public ResponseEntity<ClienteResponse> buscar(@PathVariable Long id) {
    // Spring converte "42" para Long automaticamente
    return ResponseEntity.ok(service.buscarPorId(id));
}

// URL: GET /clientes/42/pedidos/7
@GetMapping("/{clienteId}/pedidos/{pedidoId}")
public ResponseEntity<PedidoResponse> buscarPedidoDoCliente(
    @PathVariable Long clienteId,
    @PathVariable Long pedidoId
) {
    return ResponseEntity.ok(service.buscarPedido(clienteId, pedidoId));
}
```

### Quando o nome do parâmetro difere da URL

```java
// URL: GET /produtos/abc123
@GetMapping("/{codigo}")
public ResponseEntity<ProdutoResponse> buscarPorCodigo(
    @PathVariable("codigo") String codigoProduto  // nome diferente → especifica explicitamente
) {
    return ResponseEntity.ok(service.buscarPorCodigo(codigoProduto));
}
```

### Tipos suportados automaticamente

```java
@GetMapping("/{id}")
public ResponseEntity<?> exemplo(
    @PathVariable Long id,      // Long
    // ou:
    @PathVariable Integer id,   // Integer
    @PathVariable String slug,  // String
    @PathVariable UUID uuid     // UUID (ex: /recursos/550e8400-e29b-41d4-a716)
) { ... }
```

---

## @RequestParam — parâmetros de query string

Extrai valores dos **query parameters** — a parte depois do `?` na URL.

```
URL: GET /produtos?categoria=eletronicos&precoMin=100&precoMax=500&page=0&size=20
```

```java
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @RequestParam String categoria,           // obrigatório por padrão
    @RequestParam BigDecimal precoMin,
    @RequestParam BigDecimal precoMax
) { ... }
```

### Parâmetros opcionais com valor padrão

```java
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @RequestParam(required = false) String categoria,        // opcional, null se ausente
    @RequestParam(defaultValue = "0") BigDecimal precoMin,  // padrão: 0
    @RequestParam(defaultValue = "9999") BigDecimal precoMax,
    @RequestParam(defaultValue = "0") int pagina,
    @RequestParam(defaultValue = "20") int tamanho,
    @RequestParam(defaultValue = "nome") String ordenarPor
) {
    return ResponseEntity.ok(
        service.listar(categoria, precoMin, precoMax, pagina, tamanho, ordenarPor)
    );
}
```

### Lista de valores

```java
// URL: GET /produtos?ids=1&ids=2&ids=3
@GetMapping("/em-lote")
public ResponseEntity<List<ProdutoResponse>> buscarVarios(
    @RequestParam List<Long> ids
) {
    return ResponseEntity.ok(service.buscarPorIds(ids));
}

// URL: GET /produtos?tags=novo&tags=promoção
@GetMapping
public ResponseEntity<List<ProdutoResponse>> buscarPorTags(
    @RequestParam List<String> tags
) { ... }
```

---

## PathVariable vs RequestParam — quando usar cada um

| Situação | Use | Exemplo |
|---|---|---|
| Identifica um recurso específico | `@PathVariable` | `/clientes/42` |
| Filtra uma coleção | `@RequestParam` | `/clientes?ativo=true` |
| Parâmetro obrigatório e faz parte da hierarquia | `@PathVariable` | `/pedidos/10/itens` |
| Parâmetro opcional | `@RequestParam` | `/produtos?categoria=X` |
| Paginação e ordenação | `@RequestParam` ou `Pageable` | `/produtos?page=0&size=20` |
| Busca/filtro | `@RequestParam` | `/produtos?nome=teclado` |

```java
// ✅ Correto — id como PathVariable, filtros como RequestParam
@GetMapping("/{clienteId}/pedidos")
public ResponseEntity<Page<PedidoResponse>> pedidosDoCliente(
    @PathVariable Long clienteId,
    @RequestParam(required = false) String status,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size
) { ... }

// ❌ Evitar — filtros como PathVariable
@GetMapping("/{clienteId}/pedidos/{status}")  // status no path fica estranho
public ResponseEntity<?> pedidosPorStatus(...) { }
```

---

## @RequestHeader — lendo headers da requisição

```java
@GetMapping("/perfil")
public ResponseEntity<UsuarioResponse> perfil(
    @RequestHeader("Authorization") String authHeader,
    // ou com nome diferente:
    @RequestHeader(value = "X-API-Version", defaultValue = "1") String apiVersion,
    @RequestHeader(required = false) String locale
) {
    // authHeader = "Bearer eyJhbGci..."
    String token = authHeader.replace("Bearer ", "");
    return ResponseEntity.ok(service.perfil(token));
}
```

> 💡 Na prática, **não leia o token JWT manualmente no controller**. O Spring Security faz isso via filtro. Use `@RequestHeader` para outros headers customizados.

---

## @CookieValue — lendo cookies

```java
@GetMapping("/sessao")
public ResponseEntity<?> verificarSessao(
    @CookieValue(value = "session-id", required = false) String sessionId
) {
    if (sessionId == null) return ResponseEntity.status(401).build();
    return ResponseEntity.ok(service.verificarSessao(sessionId));
}
```

---

## Exemplo completo — endpoint de busca com múltiplos filtros

```java
// URL: GET /api/v1/produtos?nome=teclado&categoria=perifericos&precoMin=50&precoMax=500&ativo=true&page=0&size=20&sort=preco,asc
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> buscar(
    @RequestParam(required = false) String nome,
    @RequestParam(required = false) String categoria,
    @RequestParam(required = false) BigDecimal precoMin,
    @RequestParam(required = false) BigDecimal precoMax,
    @RequestParam(defaultValue = "true") boolean ativo,
    @PageableDefault(size = 20, sort = "nome") Pageable pageable
) {
    var filtro = new ProdutoFiltro(nome, categoria, precoMin, precoMax, ativo);
    return ResponseEntity.ok(service.buscar(filtro, pageable));
}

// DTO de filtro — agrupa todos os parâmetros
public record ProdutoFiltro(
    String nome,
    String categoria,
    BigDecimal precoMin,
    BigDecimal precoMax,
    boolean ativo
) {}
```

---

## Validação de PathVariable e RequestParam

```java
// Bean Validation funciona em PathVariables e RequestParams também
// Precisa de @Validated na classe (não @Valid)
@Validated  // ← obrigatório para validar parâmetros de método
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    @GetMapping("/{id}")
    public ResponseEntity<ProdutoResponse> buscar(
        @PathVariable @Positive(message = "ID deve ser positivo") Long id
    ) { ... }

    @GetMapping
    public ResponseEntity<?> listar(
        @RequestParam @Min(0) @Max(100) int pagina,
        @RequestParam @Size(max = 100) String nome
    ) { ... }
}
```

---

## Próximas notas
- [[17 - RequestBody e ResponseEntity]] — body e resposta HTTP
