# 23 — Relacionamentos: @OneToMany, @ManyToOne, @ManyToMany

tags: #springboot #jpa #relacionamentos #hibernate
links: [[21 - JPA e Hibernate Fundamentos]] | [[22 - Entidades e Anotações JPA]] | [[46 - Lazy vs Eager Loading]] | [[47 - Problema N+1]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Visão geral dos relacionamentos

```
@ManyToOne   — muitos para um    — lado do "filho" (tem a FK)
@OneToMany   — um para muitos    — lado do "pai" (coleção)
@OneToOne    — um para um
@ManyToMany  — muitos para muitos (tabela de junção)
```

---

## @ManyToOne — o relacionamento mais comum

Representa o lado **filho** de um relacionamento. A tabela do filho tem a chave estrangeira (FK).

```
tabela: pedidos
  id, cliente_id (FK → clientes.id), total, status, criado_em

tabela: clientes
  id, nome, email
```

```java
@Entity
@Table(name = "pedidos")
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // @ManyToOne: muitos pedidos para um cliente
    @ManyToOne(
        fetch = FetchType.LAZY,     // ← SEMPRE use LAZY em @ManyToOne
        optional = false            // cliente é obrigatório (NOT NULL)
    )
    @JoinColumn(
        name = "cliente_id",        // nome da coluna FK na tabela pedidos
        nullable = false
    )
    private Cliente cliente;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal total;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private StatusPedido status = StatusPedido.PENDENTE;

    // construtor...
    public Pedido(Cliente cliente) {
        this.cliente = cliente;
    }

    public Long getClienteId() {
        return cliente != null ? cliente.getId() : null;
    }

    // getters...
}
```

> ⚠️ **SEMPRE use `fetch = FetchType.LAZY` em `@ManyToOne`**. O padrão é EAGER, que gera JOINs desnecessários em toda query de Pedido.

---

## @OneToMany — o lado pai

Representa o lado **pai** do relacionamento. Tem a coleção de filhos.

```java
@Entity
@Table(name = "clientes")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    // @OneToMany: um cliente tem muitos pedidos
    @OneToMany(
        mappedBy = "cliente",           // nome do campo @ManyToOne em Pedido
        fetch = FetchType.LAZY,         // padrão já é LAZY — mas deixe explícito
        cascade = CascadeType.ALL,      // operações em Cliente propagam para Pedidos
        orphanRemoval = true            // remove pedidos sem cliente
    )
    private List<Pedido> pedidos = new ArrayList<>();

    // Métodos de gerenciamento do relacionamento bidirecional
    public void adicionarPedido(Pedido pedido) {
        pedidos.add(pedido);
        pedido.setCliente(this);  // mantém consistência dos dois lados
    }

    public void removerPedido(Pedido pedido) {
        pedidos.remove(pedido);
        pedido.setCliente(null);
    }
}
```

### Bidirecional vs unidirecional

```java
// ✅ Unidirecional (mais simples) — só o lado filho conhece o pai
@Entity
public class Pedido {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;  // Pedido conhece Cliente
    // Cliente NÃO tem List<Pedido>
}

// Bidirecional — os dois lados se conhecem
// Use quando precisar navegar nos dois sentidos OU para cascade
@Entity
public class Cliente {
    @OneToMany(mappedBy = "cliente")  // mappedBy = campo no Pedido
    private List<Pedido> pedidos;
}

@Entity
public class Pedido {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;
}
```

> 💡 **Regra prática:** prefira relacionamentos **unidirecionais** — são mais simples. Use bidirecional só quando realmente precisar navegar nos dois sentidos na mesma transação ou quando precisar de `cascade`.

---

## @OneToMany com tabela de itens — caso mais comum

```java
@Entity
@Table(name = "pedidos")
public class Pedido {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    @OneToMany(
        mappedBy = "pedido",
        cascade = CascadeType.ALL,    // salva/atualiza/deleta itens junto com o pedido
        orphanRemoval = true,          // se remover item da lista, apaga do banco
        fetch = FetchType.LAZY
    )
    private List<ItemPedido> itens = new ArrayList<>();

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal total = BigDecimal.ZERO;

    // Método que mantém consistência e recalcula o total
    public void adicionarItem(Produto produto, int quantidade) {
        var item = new ItemPedido(this, produto, quantidade, produto.getPreco());
        itens.add(item);
        recalcularTotal();
    }

    public void removerItem(ItemPedido item) {
        itens.remove(item);
        item.setPedido(null);
        recalcularTotal();
    }

    private void recalcularTotal() {
        this.total = itens.stream()
            .map(ItemPedido::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

@Entity
@Table(name = "itens_pedido")
public class ItemPedido {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "pedido_id", nullable = false)
    private Pedido pedido;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "produto_id", nullable = false)
    private Produto produto;

    @Column(nullable = false)
    private Integer quantidade;

    @Column(name = "preco_unitario", nullable = false, precision = 10, scale = 2)
    private BigDecimal precoUnitario;  // snapshot do preço no momento da compra

    protected ItemPedido() {}

    public ItemPedido(Pedido pedido, Produto produto, int quantidade, BigDecimal precoUnitario) {
        this.pedido = pedido;
        this.produto = produto;
        this.quantidade = quantidade;
        this.precoUnitario = precoUnitario;
    }

    public BigDecimal getSubtotal() {
        return precoUnitario.multiply(BigDecimal.valueOf(quantidade));
    }
    // getters...
}
```

---

## @ManyToMany — relacionamento N:N

Requer uma **tabela de junção** no banco.

```
tabela: produtos           tabela: produto_tags       tabela: tags
  id, nome                   produto_id, tag_id          id, nome
                             (tabela de junção)
```

```java
@Entity
@Table(name = "produtos")
public class Produto {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @ManyToMany(
        fetch = FetchType.LAZY,
        cascade = {CascadeType.PERSIST, CascadeType.MERGE}
        // NUNCA use CascadeType.ALL em @ManyToMany — pode deletar tags usadas por outros produtos
    )
    @JoinTable(
        name = "produto_tags",                              // tabela de junção
        joinColumns = @JoinColumn(name = "produto_id"),     // FK para este lado
        inverseJoinColumns = @JoinColumn(name = "tag_id")   // FK para o outro lado
    )
    private Set<Tag> tags = new HashSet<>();  // Set evita duplicatas

    public void adicionarTag(Tag tag) {
        tags.add(tag);
        tag.getProdutos().add(this);  // mantém bidirecional
    }

    public void removerTag(Tag tag) {
        tags.remove(tag);
        tag.getProdutos().remove(this);
    }
}

@Entity
@Table(name = "tags")
public class Tag {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 50)
    private String nome;

    // Lado inverso do ManyToMany — não define a tabela de junção (quem define é Produto)
    @ManyToMany(mappedBy = "tags", fetch = FetchType.LAZY)
    private Set<Produto> produtos = new HashSet<>();
    // getters...
}
```

### @ManyToMany com atributos extras na tabela de junção

Quando a tabela de junção tem colunas extras, transforme em entidade:

```java
// Em vez de @ManyToMany direto, crie uma entidade para a tabela de junção
@Entity
@Table(name = "usuario_roles")
public class UsuarioRole {

    @EmbeddedId  // chave composta
    private UsuarioRoleId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("usuarioId")
    @JoinColumn(name = "usuario_id")
    private Usuario usuario;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("roleId")
    @JoinColumn(name = "role_id")
    private Role role;

    // Atributo extra na tabela de junção
    @Column(name = "atribuido_em", nullable = false)
    private LocalDateTime atribuidoEm = LocalDateTime.now();

    @Column(name = "atribuido_por", nullable = false)
    private String atribuidoPor;
}

@Embeddable
public class UsuarioRoleId implements Serializable {
    private Long usuarioId;
    private Long roleId;
    // equals e hashCode obrigatórios
}
```

---

## @OneToOne

```java
@Entity
public class Usuario {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    // @OneToOne: usuário tem um perfil
    @OneToOne(
        mappedBy = "usuario",     // campo em PerfilUsuario
        cascade = CascadeType.ALL,
        fetch = FetchType.LAZY,   // LAZY mesmo em @OneToOne
        optional = true           // perfil pode não existir
    )
    private PerfilUsuario perfil;
}

@Entity
@Table(name = "perfis_usuario")
public class PerfilUsuario {

    @Id
    private Long id;  // mesmo ID do usuario (shared primary key)

    @OneToOne(fetch = FetchType.LAZY)
    @MapsId  // usa o mesmo ID do usuario
    @JoinColumn(name = "usuario_id")
    private Usuario usuario;

    private String bio;
    private String avatarUrl;
    private String website;
}
```

---

## Cascade — propagação de operações

| Tipo | Efeito |
|---|---|
| `PERSIST` | `save()` no pai → `save()` nos filhos |
| `MERGE` | `save()` (update) no pai → `save()` nos filhos |
| `REMOVE` | `delete()` no pai → `delete()` nos filhos |
| `REFRESH` | `refresh()` no pai → `refresh()` nos filhos |
| `DETACH` | `detach()` no pai → `detach()` nos filhos |
| `ALL` | Todos os anteriores |

```java
// Use com cuidado:
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
// CascadeType.ALL em @OneToMany de filhos exclusivos: OK
// CascadeType.ALL em @ManyToMany: PERIGO — pode deletar entidades compartilhadas

// orphanRemoval = true: remove o filho do banco quando removido da coleção
pedido.getItens().remove(item);  // item é deletado do banco ao fazer commit
```

---

## Erros comuns com relacionamentos

```java
// ❌ ERRO 1: LazyInitializationException
// Ocorre quando você acessa um relacionamento lazy fora da transação
@Transactional
public Pedido buscar(Long id) {
    return repository.findById(id).orElseThrow();
    // Retorna a entidade — a transação TERMINA aqui
}

// No controller:
Pedido pedido = service.buscar(1L);
pedido.getItens().size();  // ← LazyInitializationException! transação já encerrou

// ✅ SOLUÇÃO: converter para DTO dentro da transação
@Transactional
public PedidoResponse buscar(Long id) {
    Pedido pedido = repository.findById(id).orElseThrow();
    return PedidoResponse.from(pedido);  // acessa itens DENTRO da transação
}

// ❌ ERRO 2: StackOverflowError por ciclo de JSON
// @OneToMany em Cliente → Pedido, @ManyToOne em Pedido → Cliente
// Jackson serializa Cliente → Pedido → Cliente → Pedido → ... infinito
// ✅ SOLUÇÃO: use DTOs (nunca retorne entidade JPA no controller)

// ❌ ERRO 3: N+1 problem em @ManyToOne EAGER
// ✅ SOLUÇÃO: veja [[47 - Problema N+1]]
```

---

## Próximas notas
- [[24 - JpaRepository e Query Methods]] — como fazer queries com Spring Data
- [[46 - Lazy vs Eager Loading]] — detalhes de carregamento de relacionamentos
- [[47 - Problema N+1]] — o problema mais comum de performance com JPA
