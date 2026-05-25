# 41 — Paginação e Ordenação

tags: #springboot #rest #paginação #ordenação
links: [[39 - Padrão REST Completo]] | [[40 - Versionamento de API]] | [[42 - Filtros e Idempotência]] | [[🗺️ Mapa Principal]]

---

## Por que toda listagem deve ter paginação

```
Sem paginação:
GET /produtos → retorna 50.000 produtos de uma vez
→ resposta de 50MB
→ servidor sobrecarregado
→ cliente demora para carregar
→ memória estourada

Com paginação:
GET /produtos?page=0&size=20 → retorna 20 produtos
→ resposta de 2KB
→ servidor leve
→ cliente carrega rápido
```

---

## Pageable — configuração e uso

```java
// No Controller — @PageableDefault define os valores padrão
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @PageableDefault(
        size = 20,                          // itens por página (padrão)
        sort = "nome",                      // campo de ordenação padrão
        direction = Sort.Direction.ASC      // direção padrão
    )
    Pageable pageable
) {
    return ResponseEntity.ok(service.listar(pageable));
}

// Queries suportadas automaticamente:
// GET /produtos                          → page=0, size=20, sort=nome,ASC
// GET /produtos?page=2                   → page=2, size=20, sort=nome,ASC
// GET /produtos?page=0&size=10           → page=0, size=10, sort=nome,ASC
// GET /produtos?sort=preco,desc          → ordena por preço decrescente
// GET /produtos?sort=nome,asc&sort=preco,desc  → múltiplas ordenações
```

---

## Limitando o tamanho máximo de página

```java
// Sem limite, o cliente pode pedir size=999999 e travar o servidor
// Configure o máximo permitido:

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        PageableHandlerMethodArgumentResolver resolver =
            new PageableHandlerMethodArgumentResolver();
        resolver.setMaxPageSize(100);         // máximo 100 itens por página
        resolver.setFallbackPageable(
            PageRequest.of(0, 20, Sort.by("id"))  // padrão se não enviado
        );
        resolvers.add(resolver);
    }
}
```

---

## Page vs Slice vs List

```java
// Page<T> — inclui COUNT total (2 queries: dados + contagem)
// Use para: grids com paginação numérica ("Página 3 de 8")
Page<Produto> findAll(Pageable pageable);

// Slice<T> — sem COUNT, só sabe se tem próxima página (1 query)
// Use para: scroll infinito / "carregar mais"
Slice<Produto> findByAtivo(boolean ativo, Pageable pageable);

// List<T> — sem paginação (cuidado com tabelas grandes!)
// Use para: dropdowns, listas curtas com filtro que já limita os resultados
List<Produto> findByCategoria(Long categoriaId);
```

---

## Paginação com filtros — Service completo

```java
@Service
public class ProdutoService {

    private final ProdutoRepository repository;

    @Transactional(readOnly = true)
    public Page<ProdutoResponse> buscar(
        String nome,
        Long categoriaId,
        BigDecimal precoMin,
        BigDecimal precoMax,
        Boolean ativo,
        Pageable pageable
    ) {
        // Usando Specifications para combinar filtros dinamicamente
        Specification<Produto> spec = Specification
            .where(nome != null ? nomeContendo(nome) : null)
            .and(categoriaId != null ? porCategoria(categoriaId) : null)
            .and(precoEntre(precoMin, precoMax))
            .and(ativo != null ? porAtivo(ativo) : porAtivo(true));

        return repository.findAll(spec, pageable)
            .map(ProdutoResponse::from);
    }

    private Specification<Produto> nomeContendo(String nome) {
        return (root, q, cb) ->
            cb.like(cb.lower(root.get("nome")), "%" + nome.toLowerCase() + "%");
    }

    private Specification<Produto> porCategoria(Long categoriaId) {
        return (root, q, cb) ->
            cb.equal(root.get("categoria").get("id"), categoriaId);
    }

    private Specification<Produto> precoEntre(BigDecimal min, BigDecimal max) {
        return (root, q, cb) -> {
            if (min == null && max == null) return null;
            if (min == null) return cb.lessThanOrEqualTo(root.get("preco"), max);
            if (max == null) return cb.greaterThanOrEqualTo(root.get("preco"), min);
            return cb.between(root.get("preco"), min, max);
        };
    }

    private Specification<Produto> porAtivo(boolean ativo) {
        return (root, q, cb) -> cb.equal(root.get("ativo"), ativo);
    }
}
```

---

## Controller com todos os filtros e paginação

```java
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @RequestParam(required = false) String nome,
    @RequestParam(required = false) Long categoriaId,
    @RequestParam(required = false) BigDecimal precoMin,
    @RequestParam(required = false) BigDecimal precoMax,
    @RequestParam(required = false) Boolean ativo,
    @PageableDefault(size = 20, sort = "nome") Pageable pageable
) {
    return ResponseEntity.ok(
        service.buscar(nome, categoriaId, precoMin, precoMax, ativo, pageable)
    );
}

// GET /api/v1/produtos?nome=teclado&precoMax=200&page=0&size=10&sort=preco,asc
```

---

## Ordenação — campos permitidos

```java
// Para evitar que o cliente ordene por qualquer campo (inclusive internos):
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @PageableDefault(size = 20) Pageable pageable
) {
    // Valida campos de ordenação permitidos
    Set<String> camposPermitidos = Set.of("nome", "preco", "criadoEm");
    pageable.getSort().forEach(order -> {
        if (!camposPermitidos.contains(order.getProperty())) {
            throw new RegraDeNegocioException(
                "Campo de ordenação inválido: " + order.getProperty() +
                ". Permitidos: " + camposPermitidos
            );
        }
    });

    return ResponseEntity.ok(service.listar(pageable));
}
```

---

## Próximas notas
- [[42 - Filtros e Idempotência]] — filtros avançados e operações seguras
