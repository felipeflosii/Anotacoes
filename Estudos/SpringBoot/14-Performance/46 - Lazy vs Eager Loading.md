# 46 — Lazy vs Eager Loading

tags: #springboot #jpa #performance #lazy #eager
links: [[47 - Problema N+1]] | [[48 - Cache com Spring]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O que é

Quando o JPA carrega uma entidade com relacionamentos, ele precisa decidir: carrega os dados relacionados **agora** (Eager) ou **só quando pedido** (Lazy)?

```java
@Entity
public class Pedido {

    @ManyToOne(fetch = FetchType.EAGER)  // carrega Cliente JUNTO com o Pedido
    private Cliente cliente;

    @OneToMany(fetch = FetchType.LAZY)   // carrega Itens só quando pedido.getItens() é chamado
    private List<ItemPedido> itens;
}
```

---

## Defaults do JPA — saiba de cor

| Anotação | Default | Por quê |
|---|---|---|
| `@ManyToOne` | **EAGER** | ⚠️ Perigoso — troca para LAZY sempre |
| `@OneToOne` | **EAGER** | ⚠️ Perigoso — troca para LAZY sempre |
| `@OneToMany` | **LAZY** | ✅ Correto |
| `@ManyToMany` | **LAZY** | ✅ Correto |

> ⚠️ `@ManyToOne` sendo EAGER por padrão é uma das maiores armadilhas do JPA. Um simples `findAll()` de Pedidos fará um JOIN com Clientes que talvez você nunca precise.

**Regra absoluta: sempre declare `fetch = FetchType.LAZY` em todos os relacionamentos.**

```java
// ✅ Sempre assim:
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "cliente_id")
private Cliente cliente;

@OneToOne(fetch = FetchType.LAZY)
private PerfilUsuario perfil;
```

---

## Como o Lazy funciona

```java
@Transactional
public void exemplo() {
    Pedido pedido = repository.findById(1L).orElseThrow();
    // SQL: SELECT * FROM pedidos WHERE id = 1
    // Os itens NÃO foram carregados ainda — proxy Hibernate

    // Agora você acessa os itens:
    int qtd = pedido.getItens().size();
    // SQL: SELECT * FROM itens_pedido WHERE pedido_id = 1
    // Carregado sob demanda
}

// FORA da transação:
Pedido pedido = service.buscar(1L);  // transação encerrou
pedido.getItens().size();            // ← LazyInitializationException!
// O proxy tenta buscar do banco mas não há sessão aberta
```

### A solução: sempre converter para DTO dentro da transação

```java
@Transactional  // transação aberta
public PedidoResponse buscar(Long id) {
    Pedido pedido = repository.findById(id).orElseThrow(...);
    // acessa itens AQUI, dentro da transação → sem erro
    return PedidoResponse.from(pedido);  // PedidoResponse.from acessa pedido.getItens()
}  // transação encerra — mas já convertemos para DTO, não há mais proxy para acessar
```

---

## Quando usar JOIN FETCH para carregar junto

```java
// Quando você SABE que vai precisar dos itens, use JOIN FETCH:
@Query("""
    SELECT DISTINCT p FROM Pedido p
    JOIN FETCH p.itens i
    JOIN FETCH i.produto
    WHERE p.id = :id
    """)
Optional<Pedido> findByIdComItens(@Param("id") Long id);

// Resultado: 1 query com JOIN em vez de 1 + N queries
```

---

## Próximas notas
- [[47 - Problema N+1]] — o problema de performance mais comum com JPA
- [[48 - Cache com Spring]] — reduzindo queries ao banco
