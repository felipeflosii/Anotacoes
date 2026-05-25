# 25 — JPQL e @Query

tags: #springboot #jpa #jpql #query
links: [[24 - JpaRepository e Query Methods]] | [[26 - Configuração de Banco Relacional]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O que é JPQL

**JPQL (Jakarta Persistence Query Language)** é uma linguagem de query parecida com SQL, mas que opera sobre **entidades Java** — não tabelas do banco. Isso significa que você usa nomes de classe e atributos Java, não nomes de tabelas e colunas SQL.

```sql
-- SQL puro (tabelas e colunas do banco)
SELECT p.id, p.nome, p.preco FROM produtos p WHERE p.ativo = true

-- JPQL (classes e atributos Java)
SELECT p FROM Produto p WHERE p.ativo = true
```

---

## @Query com JPQL

```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    // JPQL simples — usa o nome da CLASSE Java, não da tabela
    @Query("SELECT p FROM Produto p WHERE p.ativo = true ORDER BY p.nome ASC")
    List<Produto> findAllAtivos();

    // Com parâmetro nomeado — preferível
    @Query("SELECT p FROM Produto p WHERE p.nome LIKE %:nome% AND p.ativo = true")
    List<Produto> findByNomeContendo(@Param("nome") String nome);

    // Com parâmetro posicional (menos legível — evite)
    @Query("SELECT p FROM Produto p WHERE p.nome LIKE %?1% AND p.ativo = true")
    List<Produto> findByNomeContendo(String nome);

    // Busca com JOIN — carrega o relacionamento junto
    @Query("""
        SELECT p FROM Produto p
        JOIN FETCH p.categoria c
        WHERE p.ativo = true
        AND c.nome = :nomeCategoria
        """)
    List<Produto> findByCategoriaAtivos(@Param("nomeCategoria") String nomeCategoria);

    // Retornando campos específicos — projeção
    @Query("SELECT new com.empresa.api.produto.dto.ProdutoResumoDto(p.id, p.nome, p.preco) " +
           "FROM Produto p WHERE p.ativo = true")
    List<ProdutoResumoDto> findResumos();

    // Com paginação — @Query + Pageable
    @Query(value = "SELECT p FROM Produto p WHERE p.ativo = true",
           countQuery = "SELECT COUNT(p) FROM Produto p WHERE p.ativo = true")
    Page<Produto> findAtivos(Pageable pageable);
    // countQuery é necessário quando o @Query tem JOIN FETCH (para o COUNT correto)

    // Condicionais com JPQL — busca de datas
    @Query("""
        SELECT p FROM Pedido p
        WHERE p.cliente.id = :clienteId
        AND p.criadoEm BETWEEN :inicio AND :fim
        AND p.status IN :statuses
        ORDER BY p.criadoEm DESC
        """)
    List<Pedido> findPedidosDoCliente(
        @Param("clienteId") Long clienteId,
        @Param("inicio") LocalDateTime inicio,
        @Param("fim") LocalDateTime fim,
        @Param("statuses") List<StatusPedido> statuses
    );
}
```

---

## @Query com SQL nativo

Quando JPQL não consegue expressar a query (funções específicas do banco, CTEs, window functions):

```java
@Query(
    value = """
        SELECT
            c.id,
            c.nome,
            c.email,
            COUNT(p.id) AS total_pedidos,
            SUM(p.total) AS valor_total
        FROM clientes c
        LEFT JOIN pedidos p ON p.cliente_id = c.id
        WHERE c.ativo = true
        GROUP BY c.id, c.nome, c.email
        HAVING COUNT(p.id) > :minPedidos
        ORDER BY valor_total DESC
        """,
    nativeQuery = true
)
List<Object[]> findClientesComEstatisticas(@Param("minPedidos") int minPedidos);

// Melhor: use interface de projeção para tipagem
@Query(
    value = """
        SELECT
            c.id AS id,
            c.nome AS nome,
            COUNT(p.id) AS totalPedidos,
            COALESCE(SUM(p.total), 0) AS valorTotal
        FROM clientes c
        LEFT JOIN pedidos p ON p.cliente_id = c.id
        WHERE c.ativo = true
        GROUP BY c.id, c.nome
        ORDER BY valor_total DESC
        """,
    nativeQuery = true
)
List<ClienteEstatisticaProjection> findEstatisticasClientes();

// Interface de projeção para query nativa
public interface ClienteEstatisticaProjection {
    Long getId();
    String getNome();
    Long getTotalPedidos();
    BigDecimal getValorTotal();
}
```

---

## JOIN FETCH — carregando relacionamentos eficientemente

```java
// Problema: buscar pedidos com seus itens — gera N+1 queries
List<Pedido> pedidos = pedidoRepository.findAll();
pedidos.forEach(p -> p.getItens().size()); // N queries para N pedidos!

// Solução: JOIN FETCH carrega tudo em 1 query
@Query("""
    SELECT DISTINCT p FROM Pedido p
    JOIN FETCH p.itens i
    JOIN FETCH i.produto
    WHERE p.cliente.id = :clienteId
    """)
List<Pedido> findPedidosComItens(@Param("clienteId") Long clienteId);
// 1 query com JOIN — muito mais eficiente
```

> ⚠️ **DISTINCT é obrigatório com JOIN FETCH em coleções** — sem ele, você obtém um Pedido duplicado para cada item.

---

## Queries com agregação e grupos

```java
@Query("""
    SELECT p.status AS status, COUNT(p) AS quantidade, SUM(p.total) AS totalValor
    FROM Pedido p
    WHERE p.criadoEm >= :dataInicio
    GROUP BY p.status
    """)
List<PedidoEstatisticaProjection> findEstatisticasPorStatus(@Param("dataInicio") LocalDateTime dataInicio);

public interface PedidoEstatisticaProjection {
    StatusPedido getStatus();
    Long getQuantidade();
    BigDecimal getTotalValor();
}
```

---

## JPQL vs Query Methods vs SQL nativo — quando usar cada um

| Situação | Abordagem | Exemplo |
|---|---|---|
| Busca simples por campo | Query Method | `findByEmail(email)` |
| Busca com texto | Query Method | `findByNomeContainingIgnoreCase` |
| Paginação simples | Query Method | `findByAtivo(true, pageable)` |
| JOIN simples | @Query JPQL | `SELECT p FROM Pedido p JOIN FETCH p.itens` |
| Filtros dinâmicos | Specifications | Veja nota 24 |
| Agregações | @Query JPQL | `SELECT COUNT(p), SUM(p.total) FROM Pedido p` |
| Funções específicas do banco | @Query nativeQuery | `date_trunc`, `json_agg`, CTEs |
| Window functions | @Query nativeQuery | `ROW_NUMBER() OVER (...)` |
| Relatórios complexos | @Query nativeQuery | Múltiplos JOINs e subqueries |

---

## Text Blocks — queries legíveis (Java 15+)

```java
// Antes — concatenação ilegível
@Query("SELECT p FROM Pedido p " +
       "JOIN FETCH p.cliente c " +
       "WHERE p.status = :status " +
       "AND c.ativo = true " +
       "ORDER BY p.criadoEm DESC")

// Depois — text block (muito mais legível)
@Query("""
    SELECT p FROM Pedido p
    JOIN FETCH p.cliente c
    WHERE p.status = :status
    AND c.ativo = true
    ORDER BY p.criadoEm DESC
    """)
List<Pedido> findByStatusComCliente(@Param("status") StatusPedido status);
```

---

## @NamedQuery — queries na entidade (menos comum)

```java
// Na entidade
@Entity
@NamedQuery(
    name = "Produto.findAtivosBaratos",
    query = "SELECT p FROM Produto p WHERE p.ativo = true AND p.preco <= :precoMax"
)
public class Produto { ... }

// No repositório — referencia pelo nome
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    List<Produto> findAtivosBaratos(@Param("precoMax") BigDecimal precoMax);
    // Spring detecta o @NamedQuery pelo nome do método
}
```

> Na prática: prefira `@Query` no repositório — mais próximo de onde é usado, mais fácil de encontrar.

---

## Próximas notas
- [[26 - Configuração de Banco Relacional]] — configuração completa do banco
- [[27 - Flyway Migrations]] — gerenciamento de schema com migrations
