# 24 — JpaRepository e Query Methods

tags: #springboot #jpa #repository #querymethods
links: [[21 - JPA e Hibernate Fundamentos]] | [[25 - JPQL e @Query]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## A hierarquia dos repositórios Spring Data

```
Repository<T, ID>                   ← interface base vazia
  └── CrudRepository<T, ID>         ← operações CRUD básicas
        └── PagingAndSortingRepository<T, ID>  ← paginação e ordenação
              └── JpaRepository<T, ID>         ← JPA específico + flushing
```

Use **sempre `JpaRepository`** — inclui tudo dos anteriores e adiciona métodos JPA específicos como `flush()`, `saveAndFlush()`, `deleteAllInBatch()`.

---

## JpaRepository — métodos prontos

```java
public interface ClienteRepository extends JpaRepository<Cliente, Long> {
    // Todos esses métodos já existem — não precisa implementar:
}

// O que você ganha de graça:
clienteRepository.save(cliente);           // INSERT ou UPDATE
clienteRepository.saveAll(clientes);       // INSERT ou UPDATE em lote
clienteRepository.findById(1L);            // SELECT por PK → Optional<Cliente>
clienteRepository.findAll();               // SELECT * (cuidado com tabelas grandes)
clienteRepository.findAll(pageable);       // SELECT com paginação e ordenação
clienteRepository.findAllById(ids);        // SELECT por lista de IDs
clienteRepository.existsById(1L);          // SELECT EXISTS
clienteRepository.count();                 // SELECT COUNT(*)
clienteRepository.deleteById(1L);          // DELETE por PK
clienteRepository.delete(cliente);         // DELETE por entidade
clienteRepository.deleteAll();             // DELETE todos (cuidado!)
clienteRepository.deleteAllInBatch(list);  // DELETE em batch (mais eficiente)
```

---

## Query Methods — queries por nome de método

O Spring Data gera SQL automaticamente a partir do **nome do método**. Você escreve o nome, o Spring faz o SQL.

### Palavras-chave suportadas

| Palavra | SQL gerado | Exemplo |
|---|---|---|
| `findBy` | `SELECT ... WHERE` | `findByEmail` |
| `findAll` | `SELECT ... WHERE` | `findAllByAtivo` |
| `existsBy` | `SELECT EXISTS` | `existsByEmail` |
| `countBy` | `SELECT COUNT(*)` | `countByAtivo` |
| `deleteBy` | `DELETE ... WHERE` | `deleteByEmail` |
| `And` | `AND` | `findByNomeAndEmail` |
| `Or` | `OR` | `findByNomeOrEmail` |
| `Not` | `<>` | `findByNomeNot` |
| `Like` | `LIKE` | `findByNomeLike` |
| `Containing` | `LIKE %valor%` | `findByNomeContaining` |
| `StartingWith` | `LIKE valor%` | `findByNomeStartingWith` |
| `EndingWith` | `LIKE %valor` | `findByNomeEndingWith` |
| `IgnoreCase` | `UPPER/LOWER` | `findByNomeIgnoreCase` |
| `Between` | `BETWEEN` | `findByPrecoBetween` |
| `LessThan` | `<` | `findByPrecoLessThan` |
| `GreaterThan` | `>` | `findByPrecoGreaterThan` |
| `In` | `IN (...)` | `findByStatusIn` |
| `IsNull` | `IS NULL` | `findByDeletadoEmIsNull` |
| `IsNotNull` | `IS NOT NULL` | `findByDeletadoEmIsNotNull` |
| `OrderBy` | `ORDER BY` | `findAllOrderByNomeAsc` |
| `Top` / `First` | `LIMIT` | `findTop5ByOrderByPrecoDesc` |

### Exemplos completos por entidade

```java
public interface ClienteRepository extends JpaRepository<Cliente, Long> {

    // ===== BUSCAS POR CAMPO =====

    Optional<Cliente> findByEmail(String email);
    // SELECT * FROM clientes WHERE email = ?

    boolean existsByEmail(String email);
    // SELECT COUNT(*) > 0 FROM clientes WHERE email = ?

    List<Cliente> findByAtivo(boolean ativo);
    // SELECT * FROM clientes WHERE ativo = ?

    // ===== BUSCA COM TEXTO =====

    List<Cliente> findByNomeContainingIgnoreCase(String nome);
    // SELECT * FROM clientes WHERE UPPER(nome) LIKE UPPER('%?%')

    List<Cliente> findByNomeStartingWithIgnoreCase(String prefixo);
    // SELECT * FROM clientes WHERE UPPER(nome) LIKE UPPER('?%')

    // ===== COMBINAÇÕES COM AND / OR =====

    List<Cliente> findByNomeContainingAndAtivo(String nome, boolean ativo);
    // SELECT * FROM clientes WHERE nome LIKE '%?%' AND ativo = ?

    Optional<Cliente> findByEmailOrCpf(String email, String cpf);
    // SELECT * FROM clientes WHERE email = ? OR cpf = ?

    // ===== DATAS =====

    List<Cliente> findByCriadoEmBetween(LocalDateTime inicio, LocalDateTime fim);
    // SELECT * FROM clientes WHERE criado_em BETWEEN ? AND ?

    List<Cliente> findByCriadoEmAfter(LocalDateTime data);
    // SELECT * FROM clientes WHERE criado_em > ?

    // ===== ORDENAÇÃO =====

    List<Cliente> findByAtivoOrderByNomeAsc(boolean ativo);
    // SELECT * FROM clientes WHERE ativo = ? ORDER BY nome ASC

    // ===== PAGINAÇÃO =====

    Page<Cliente> findByAtivo(boolean ativo, Pageable pageable);
    // SELECT * FROM clientes WHERE ativo = ? + LIMIT + OFFSET

    // ===== LIMITAR RESULTADOS =====

    List<Cliente> findTop10ByAtivoOrderByCriadoEmDesc(boolean ativo);
    // SELECT * FROM clientes WHERE ativo = ? ORDER BY criado_em DESC LIMIT 10

    Optional<Cliente> findFirstByAtivoOrderByCriadoEmDesc(boolean ativo);
    // SELECT * FROM clientes WHERE ativo = ? ORDER BY criado_em DESC LIMIT 1

    // ===== RELACIONAMENTOS =====

    List<Cliente> findByPedidosStatus(StatusPedido status);
    // SELECT c.* FROM clientes c JOIN pedidos p ON p.cliente_id = c.id WHERE p.status = ?

    // ===== CAMPOS NULOS =====

    List<Cliente> findByTelefoneIsNull();
    List<Cliente> findByTelefoneIsNotNull();
}
```

---

## Paginação com Pageable — essencial para APIs

```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    Page<Produto> findByAtivo(boolean ativo, Pageable pageable);
    Page<Produto> findByNomeContainingIgnoreCase(String nome, Pageable pageable);
}
```

```java
// No Service — usando Pageable
@Transactional(readOnly = true)
public Page<ProdutoResponse> listar(Pageable pageable) {
    return produtoRepository.findAll(pageable)
        .map(ProdutoResponse::from);
}

// No Controller — Pageable vem automaticamente dos query params
@GetMapping
public ResponseEntity<Page<ProdutoResponse>> listar(
    @PageableDefault(size = 20, sort = "nome", direction = Sort.Direction.ASC)
    Pageable pageable
) {
    return ResponseEntity.ok(service.listar(pageable));
}

// Cliente faz: GET /produtos?page=0&size=20&sort=nome,asc
//              GET /produtos?page=2&size=10&sort=preco,desc
```

### O que a resposta Page contém

```json
{
  "content": [
    { "id": 1, "nome": "Produto A", "preco": 10.00 },
    { "id": 2, "nome": "Produto B", "preco": 20.00 }
  ],
  "pageable": {
    "sort": { "sorted": true, "unsorted": false },
    "pageNumber": 0,
    "pageSize": 20
  },
  "totalElements": 150,
  "totalPages": 8,
  "last": false,
  "first": true,
  "numberOfElements": 20
}
```

---

## Slice vs Page

```java
// Page — faz COUNT para saber o total de páginas (2 queries)
Page<Cliente> findByAtivo(boolean ativo, Pageable pageable);

// Slice — não faz COUNT, só sabe se tem próxima página (1 query, mais rápido)
// Use para feeds infinitos / scroll infinito
Slice<Cliente> findByAtivo(boolean ativo, Pageable pageable);
```

---

## @Modifying — queries de escrita via método

```java
public interface ClienteRepository extends JpaRepository<Cliente, Long> {

    // UPDATE
    @Modifying
    @Transactional
    @Query("UPDATE Cliente c SET c.ativo = false WHERE c.id = :id")
    int desativarPorId(@Param("id") Long id);

    // DELETE customizado
    @Modifying
    @Transactional
    void deleteByAtivoFalse();  // DELETE FROM clientes WHERE ativo = false

    // UPDATE em lote
    @Modifying
    @Transactional
    @Query("UPDATE Produto p SET p.ativo = false WHERE p.criadoEm < :data")
    int desativarProdutosAntes(@Param("data") LocalDateTime data);
}
```

> ⚠️ `@Modifying` sem `clearAutomatically = true` pode deixar o contexto de persistência inconsistente após o UPDATE. Use `@Modifying(clearAutomatically = true)` para limpar o cache.

---

## Specifications — queries dinâmicas

Quando os filtros variam (a combinação muda conforme os parâmetros), use Specifications:

```java
// Adicionar dependência: JpaSpecificationExecutor
public interface ProdutoRepository extends JpaRepository<Produto, Long>,
    JpaSpecificationExecutor<Produto> { }

// Classe de specifications
public class ProdutoSpecs {

    public static Specification<Produto> ativo() {
        return (root, query, cb) -> cb.isTrue(root.get("ativo"));
    }

    public static Specification<Produto> nomeContendo(String nome) {
        return nome == null ? null :
            (root, query, cb) -> cb.like(
                cb.lower(root.get("nome")),
                "%" + nome.toLowerCase() + "%"
            );
    }

    public static Specification<Produto> precoEntre(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min == null && max == null) return null;
            if (min == null) return cb.lessThanOrEqualTo(root.get("preco"), max);
            if (max == null) return cb.greaterThanOrEqualTo(root.get("preco"), min);
            return cb.between(root.get("preco"), min, max);
        };
    }
}

// Usando no Service — combina specs dinamicamente
@Transactional(readOnly = true)
public Page<ProdutoResponse> buscar(String nome, BigDecimal precoMin,
                                     BigDecimal precoMax, Pageable pageable) {
    var spec = Specification.where(ProdutoSpecs.ativo())
        .and(ProdutoSpecs.nomeContendo(nome))
        .and(ProdutoSpecs.precoEntre(precoMin, precoMax));

    return produtoRepository.findAll(spec, pageable).map(ProdutoResponse::from);
}
```

---

## Próximas notas
- [[25 - JPQL e @Query]] — queries customizadas em JPQL e SQL nativo
- [[47 - Problema N+1]] — performance em queries com relacionamentos
