# 47 — O Problema N+1

tags: #springboot #jpa #performance #n1
links: [[46 - Lazy vs Eager Loading]] | [[48 - Cache com Spring]] | [[🗺️ Mapa Principal]]

---

## O que é o N+1

É o problema de performance mais frequente com JPA. Acontece quando você faz 1 query para buscar uma lista e depois N queries para buscar os relacionamentos de cada item.

```java
// ❌ Código com N+1
List<Pedido> pedidos = pedidoRepository.findAll();  // 1 query
// SELECT * FROM pedidos  → retorna 100 pedidos

for (Pedido pedido : pedidos) {
    String nomeCliente = pedido.getCliente().getNome();
    // Para cada pedido: SELECT * FROM clientes WHERE id = ?
    // São 100 queries adicionais!
}

// Total: 1 + 100 = 101 queries para 100 pedidos
// Com 1000 pedidos: 1001 queries → aplicação lenta
```

---

## Detectando N+1

```yaml
# application.yml — habilitar log de SQL para detectar
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        generate_statistics: true  # mostra total de queries no log
```

Se você ver muitas queries `SELECT * FROM clientes WHERE id = ?` repetidas, é N+1.

---

## Soluções

### 1. JOIN FETCH — solução principal

```java
// ✅ Uma query com JOIN — sem N+1
@Query("""
    SELECT DISTINCT p FROM Pedido p
    JOIN FETCH p.cliente
    """)
List<Pedido> findAllComCliente();
// SELECT p.*, c.* FROM pedidos p JOIN clientes c ON c.id = p.cliente_id

// Com paginação + JOIN FETCH (precisa de countQuery separado)
@Query(
    value = "SELECT DISTINCT p FROM Pedido p JOIN FETCH p.cliente",
    countQuery = "SELECT COUNT(DISTINCT p) FROM Pedido p"
)
Page<Pedido> findAllComClientePaginado(Pageable pageable);
```

### 2. @EntityGraph — carregar relacionamentos por configuração

```java
@Entity
@NamedEntityGraph(
    name = "Pedido.comClienteEItens",
    attributeNodes = {
        @NamedAttributeNode("cliente"),
        @NamedAttributeNode(value = "itens", subgraph = "itens"),
    },
    subgraphs = @NamedSubgraph(
        name = "itens",
        attributeNodes = @NamedAttributeNode("produto")
    )
)
public class Pedido { ... }

// No repositório:
@EntityGraph("Pedido.comClienteEItens")
List<Pedido> findAll();  // carrega cliente + itens + produto em 1 query
```

### 3. Projeção com @Query — buscar só o que precisa

```java
// Em vez de carregar toda a entidade com relacionamentos,
// busque direto o DTO:
@Query("""
    SELECT new com.empresa.api.pedido.dto.PedidoResumoDto(
        p.id, p.total, p.status,
        c.nome, c.email
    )
    FROM Pedido p
    JOIN p.cliente c
    WHERE p.status = :status
    """)
List<PedidoResumoDto> findResumosPorStatus(@Param("status") StatusPedido status);
```

---

## Regra prática anti-N+1

```
1. Sempre use FetchType.LAZY em todos os relacionamentos
2. Identifique no Service quais relacionamentos vai acessar no DTO
3. Use JOIN FETCH ou @EntityGraph para carregá-los em 1 query
4. Com @DataJpaTest, escreva testes contando o número de queries
5. Em produção, monitore queries lentas com pg_stat_statements
```

---

## Próxima nota
- [[48 - Cache com Spring]]
