# 15 — Métodos HTTP: GET, POST, PUT, PATCH, DELETE

tags: #springboot #controllers #http #crud
links: [[14 - RestController e RequestMapping]] | [[16 - PathVariable e RequestParam]] | [[17 - RequestBody e ResponseEntity]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Mapeamento CRUD → HTTP → Spring

| Operação CRUD | Método HTTP | Anotação Spring | Status de sucesso |
|---|---|---|---|
| **C**reate | POST | `@PostMapping` | 201 Created |
| **R**ead (lista) | GET | `@GetMapping` | 200 OK |
| **R**ead (um) | GET | `@GetMapping("/{id}")` | 200 OK |
| **U**pdate (completo) | PUT | `@PutMapping("/{id}")` | 200 OK |
| **U**pdate (parcial) | PATCH | `@PatchMapping("/{id}")` | 200 OK |
| **D**elete | DELETE | `@DeleteMapping("/{id}")` | 204 No Content |

---

## GET — Leitura

O método mais simples. Não tem body. Retorna dados.

```java
// GET /produtos — lista com paginação
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @PageableDefault(size = 20, sort = "nome") Pageable pageable,
    @RequestParam(required = false) String categoria,
    @RequestParam(required = false) BigDecimal precoMin,
    @RequestParam(required = false) BigDecimal precoMax
) {
    return ResponseEntity.ok(
        produtoService.listar(pageable, categoria, precoMin, precoMax)
    );
}

// GET /produtos/99 — busca por ID
@GetMapping("/{id}")
public ResponseEntity<ProdutoResponse> buscarPorId(@PathVariable Long id) {
    return ResponseEntity.ok(produtoService.buscarPorId(id));
}

// GET /clientes/99/pedidos — sub-recurso (pedidos de um cliente específico)
@GetMapping("/{clienteId}/pedidos")
public ResponseEntity<Page<PedidoResponse>> pedidosDoCliente(
    @PathVariable Long clienteId,
    @PageableDefault(size = 10) Pageable pageable
) {
    return ResponseEntity.ok(pedidoService.listarPorCliente(clienteId, pageable));
}
```

**Características do GET:**
- Sem body na requisição
- Idempotente e seguro — pode ser repetido sem efeitos
- Resultado pode ser cacheado
- `@Transactional(readOnly = true)` no service

---

## POST — Criação

Cria um novo recurso. Body obrigatório. Retorna 201 com o recurso criado.

```java
@PostMapping
public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest request) {

    ClienteResponse response = clienteService.criar(request);

    // ✅ Boas práticas:
    // 1. Retornar 201 Created (não 200 OK)
    // 2. Incluir Location header com a URL do novo recurso
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()            // http://localhost:8080/api/v1/clientes
        .path("/{id}")
        .buildAndExpand(response.id())   // http://localhost:8080/api/v1/clientes/42
        .toUri();

    return ResponseEntity.created(location).body(response);
    // HTTP 201 Created
    // Location: /api/v1/clientes/42
    // Body: { "id": 42, "nome": "Felipe", ... }
}
```

**Características do POST:**
- Tem body com os dados do novo recurso
- **Não** é idempotente — chamar duas vezes cria dois recursos
- Retorna 201 + Location header
- O ID é gerado pelo servidor

---

## PUT — Atualização completa

Substitui **completamente** o recurso. Todos os campos devem ser enviados.

```java
@PutMapping("/{id}")
public ResponseEntity<ProdutoResponse> atualizar(
    @PathVariable Long id,
    @RequestBody @Valid ProdutoRequest request
) {
    // Service verifica se existe, substitui todos os campos
    return ResponseEntity.ok(produtoService.atualizar(id, request));
}
```

```java
// Service — PUT substitui tudo
@Transactional
public ProdutoResponse atualizar(Long id, ProdutoRequest request) {
    Produto produto = produtoRepository.findById(id)
        .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));

    // Atualiza TODOS os campos — se não vier no request, vai para null/default
    produto.setNome(request.nome());
    produto.setDescricao(request.descricao());
    produto.setPreco(request.preco());
    produto.setEstoque(request.estoque());
    produto.setCategoria(request.categoria());
    // atualizadoEm é atualizado automaticamente pelo @PreUpdate

    return ProdutoResponse.from(produto);
    // Não precisa chamar save() — mudanças persistem automaticamente (@Transactional)
}
```

**Quando usar PUT vs PATCH:**
```
PUT  /produtos/99    → body: { nome, descricao, preco, estoque, categoria }
                       → TODOS os campos são substituídos
                       → campos ausentes viram null

PATCH /produtos/99   → body: { preco: 49.90 }
                       → APENAS o preço é atualizado
                       → outros campos permanecem iguais
```

---

## PATCH — Atualização parcial

Atualiza apenas os campos enviados. Mais complexo de implementar, mas mais correto semanticamente.

```java
@PatchMapping("/{id}")
public ResponseEntity<ProdutoResponse> atualizarParcial(
    @PathVariable Long id,
    @RequestBody Map<String, Object> campos  // campos dinâmicos
) {
    return ResponseEntity.ok(produtoService.atualizarParcial(id, campos));
}

// Ou com um DTO específico para PATCH (abordagem mais tipada):
@PatchMapping("/{id}/preco")
public ResponseEntity<ProdutoResponse> atualizarPreco(
    @PathVariable Long id,
    @RequestBody @Valid AtualizarPrecoRequest request
) {
    return ResponseEntity.ok(produtoService.atualizarPreco(id, request.novoPreco()));
}
```

```java
// DTOs específicos para PATCH — mais limpo que Map<String, Object>
public record AtualizarPrecoRequest(
    @NotNull @Positive BigDecimal novoPreco
) {}

public record AtualizarEstoqueRequest(
    @NotNull @Min(0) Integer quantidade
) {}
```

```java
// Service — PATCH atualiza só o que foi enviado
@Transactional
public ProdutoResponse atualizarPreco(Long id, BigDecimal novoPreco) {
    Produto produto = produtoRepository.findById(id)
        .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));

    // Regra de negócio: preço não pode ser menor que custo
    if (novoPreco.compareTo(produto.getCusto()) < 0) {
        throw new RegraDeNegocioException("Preço não pode ser menor que o custo");
    }

    produto.setPreco(novoPreco);  // só atualiza o preço
    return ProdutoResponse.from(produto);
}
```

---

## DELETE — Remoção

Remove o recurso. Retorna 204 No Content (sem body na resposta).

```java
// DELETE direto (hard delete)
@DeleteMapping("/{id}")
public ResponseEntity<Void> deletar(@PathVariable Long id) {
    produtoService.deletar(id);
    return ResponseEntity.noContent().build();  // 204
}

// Soft delete — desativa em vez de apagar
@DeleteMapping("/{id}")
public ResponseEntity<Void> desativar(@PathVariable Long id) {
    produtoService.desativar(id);
    return ResponseEntity.noContent().build();  // 204
}
```

```java
// Service — dois tipos de delete
@Transactional
public void deletar(Long id) {
    if (!produtoRepository.existsById(id)) {
        throw new RecursoNaoEncontradoException("Produto", id);
    }
    produtoRepository.deleteById(id);  // remove do banco
}

@Transactional
public void desativar(Long id) {
    Produto produto = produtoRepository.findById(id)
        .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", id));
    produto.desativar();  // ativo = false — mantém no banco
}
```

**Hard delete vs Soft delete:**

| | Hard Delete | Soft Delete |
|---|---|---|
| **O que faz** | Remove do banco | Marca como inativo |
| **Recuperação** | Impossível | Possível |
| **Histórico** | Perdido | Mantido |
| **Complexidade** | Simples | Exige filtros nas queries |
| **Uso** | Dados temporários | Clientes, pedidos, dados críticos |

---

## Tabela de referência — controller completo padrão de mercado

```java
@RestController
@RequestMapping("/api/v1/produtos")
public class ProdutoController {

    private final ProdutoService service;

    public ProdutoController(ProdutoService service) {
        this.service = service;
    }

    // ========================= LEITURA ========================= //

    @GetMapping                                      // GET /produtos
    public ResponseEntity<Page<ProdutoResponse>> listar(
        @PageableDefault(size = 20) Pageable pageable
    ) {
        return ResponseEntity.ok(service.listar(pageable));
    }

    @GetMapping("/{id}")                             // GET /produtos/{id}
    public ResponseEntity<ProdutoResponse> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    // ========================= ESCRITA ========================= //

    @PostMapping                                     // POST /produtos → 201
    public ResponseEntity<ProdutoResponse> criar(
        @RequestBody @Valid ProdutoRequest request
    ) {
        var response = service.criar(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(response.id()).toUri();
        return ResponseEntity.created(location).body(response);
    }

    @PutMapping("/{id}")                             // PUT /produtos/{id} → 200
    public ResponseEntity<ProdutoResponse> atualizar(
        @PathVariable Long id,
        @RequestBody @Valid ProdutoRequest request
    ) {
        return ResponseEntity.ok(service.atualizar(id, request));
    }

    @PatchMapping("/{id}/ativar")                    // PATCH /produtos/{id}/ativar
    public ResponseEntity<Void> ativar(@PathVariable Long id) {
        service.ativar(id);
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/{id}")                          // DELETE /produtos/{id} → 204
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        service.deletar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Próximas notas
- [[16 - PathVariable e RequestParam]] — parâmetros na URL
- [[17 - RequestBody e ResponseEntity]] — body e resposta em detalhe
