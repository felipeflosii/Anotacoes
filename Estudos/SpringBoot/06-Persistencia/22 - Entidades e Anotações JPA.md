# 22 — Entidades e Anotações JPA

tags: #springboot #jpa #entidade #anotações
links: [[21 - JPA e Hibernate Fundamentos]] | [[23 - Relacionamentos OneToMany ManyToOne ManyToMany]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## @Entity e @Table

```java
@Entity                         // obrigatório — marca como entidade JPA
@Table(
    name = "produtos",          // nome da tabela (padrão: nome da classe em minúsculo)
    schema = "public",          // schema do banco
    uniqueConstraints = {
        @UniqueConstraint(
            name = "uk_produtos_codigo",
            columnNames = {"codigo"}
        )
    },
    indexes = {
        @Index(name = "idx_produtos_categoria", columnList = "categoria_id"),
        @Index(name = "idx_produtos_ativo", columnList = "ativo")
    }
)
public class Produto {
    // ...
}
```

---

## @Id e @GeneratedValue — chave primária

```java
@Entity
public class Produto {

    // Opção 1: AUTO_INCREMENT / SERIAL (mais comum)
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // PostgreSQL: BIGSERIAL / SERIAL
    // MySQL: AUTO_INCREMENT
    // Não usa sequence — eficiente para inserts simples
    private Long id;

    // Opção 2: Sequence (recomendada para PostgreSQL em alta performance)
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "produto_seq")
    @SequenceGenerator(
        name = "produto_seq",
        sequenceName = "produto_id_seq",
        allocationSize = 50  // busca 50 IDs por vez do banco — mais eficiente
    )
    private Long id;

    // Opção 3: UUID (para sistemas distribuídos)
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)  // Java 17+, Hibernate 6+
    private UUID id;

    // Opção 4: Atribuído manualmente (não há geração automática)
    @Id
    private String codigoExterno;  // ex: código vindo de sistema legado
}
```

---

## @Column — configurando colunas

```java
@Entity
@Table(name = "produtos")
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(
        name = "nome",           // nome da coluna (padrão: nome do campo)
        nullable = false,        // NOT NULL no banco
        length = 200,            // VARCHAR(200)
        unique = false           // sem UNIQUE constraint aqui (use @Table para isso)
    )
    private String nome;

    @Column(name = "descricao", columnDefinition = "TEXT")  // tipo explícito
    private String descricao;

    @Column(
        name = "preco",
        nullable = false,
        precision = 10,          // total de dígitos
        scale = 2                // dígitos após a vírgula — DECIMAL(10,2)
    )
    private BigDecimal preco;

    @Column(name = "estoque", nullable = false)
    private Integer estoque = 0;  // valor padrão Java (não SQL)

    @Column(name = "ativo", nullable = false)
    private boolean ativo = true;

    @Column(name = "codigo", length = 50, unique = true)
    private String codigo;

    // Coluna criada com nome diferente do campo
    @Column(name = "criado_em", nullable = false, updatable = false)
    // updatable = false: nunca gera UPDATE para esta coluna
    private LocalDateTime criadoEm;

    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    // Campo ignorado pelo JPA — não tem coluna no banco
    @Transient
    private BigDecimal precoComDesconto;  // calculado em memória
}
```

---

## Enums — @Enumerated

```java
public enum StatusPedido {
    PENDENTE,
    PAGO,
    EM_SEPARACAO,
    ENVIADO,
    ENTREGUE,
    CANCELADO
}

@Entity
public class Pedido {

    // ❌ Evitar: salva o índice numérico (0, 1, 2...)
    // Frágil — reordenar o enum quebra os dados históricos
    @Enumerated(EnumType.ORDINAL)
    private StatusPedido status;

    // ✅ Correto: salva a String ("PENDENTE", "PAGO", etc.)
    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false, length = 20)
    private StatusPedido status;
}
```

---

## Datas e @Temporal

Com Java 8+ (LocalDate, LocalDateTime), não precisa de `@Temporal`:

```java
@Entity
public class Evento {

    // LocalDateTime — timestamp sem timezone
    @Column(name = "criado_em")
    private LocalDateTime criadoEm;

    // LocalDate — só a data, sem hora
    @Column(name = "data_evento")
    private LocalDate dataEvento;

    // LocalTime — só a hora
    @Column(name = "hora_inicio")
    private LocalTime horaInicio;

    // OffsetDateTime — timestamp COM timezone (recomendado para sistemas globais)
    @Column(name = "criado_em_utc", columnDefinition = "TIMESTAMPTZ")
    private OffsetDateTime criadoEmUtc;

    // Instant — timestamp em UTC (também bom para sistemas globais)
    @Column(name = "timestamp_utc")
    private Instant timestampUtc;
}
```

---

## @PrePersist e @PreUpdate — callbacks de ciclo de vida

```java
@Entity
@Table(name = "clientes")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private String email;

    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    // Executado antes do INSERT
    @PrePersist
    protected void prePersist() {
        this.criadoEm = LocalDateTime.now();
        this.atualizadoEm = LocalDateTime.now();
    }

    // Executado antes de cada UPDATE
    @PreUpdate
    protected void preUpdate() {
        this.atualizadoEm = LocalDateTime.now();
    }
}
```

### Alternativa: @EntityListeners com Auditing do Spring

```java
// Configuração — habilita auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {}

// Entidade base com campos de auditoria — outras entidades herdam
@MappedSuperclass  // não é entidade — não tem tabela própria
@EntityListeners(AuditingEntityListener.class)
public abstract class EntidadeBase {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate                     // preenchido automaticamente pelo Spring
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate                // atualizado automaticamente pelo Spring
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    // Getters...
}

// Entidade que herda os campos de auditoria
@Entity
@Table(name = "clientes")
public class Cliente extends EntidadeBase {
    // só os campos específicos do Cliente
    private String nome;
    private String email;
}

@Entity
@Table(name = "produtos")
public class Produto extends EntidadeBase {
    private String nome;
    private BigDecimal preco;
}
```

---

## Entidade completa — exemplo de referência

```java
@Entity
@Table(
    name = "produtos",
    indexes = {
        @Index(name = "idx_produtos_categoria_id", columnList = "categoria_id"),
        @Index(name = "idx_produtos_ativo", columnList = "ativo")
    }
)
@EntityListeners(AuditingEntityListener.class)
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String nome;

    @Column(columnDefinition = "TEXT")
    private String descricao;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal preco;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal custo;

    @Column(nullable = false)
    private Integer estoque = 0;

    @Column(name = "estoque_minimo", nullable = false)
    private Integer estoqueMinimo = 5;

    @Column(nullable = false)
    private boolean ativo = true;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 30)
    private CategoriaProduto categoria;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    // Construtor protegido — JPA precisa, mas não queremos uso externo
    protected Produto() {}

    // Construtor de negócio
    public Produto(String nome, String descricao, BigDecimal preco,
                   BigDecimal custo, Integer estoque, CategoriaProduto categoria) {
        this.nome = nome;
        this.descricao = descricao;
        this.preco = preco;
        this.custo = custo;
        this.estoque = estoque;
        this.categoria = categoria;
    }

    // Métodos de negócio na entidade
    public void adicionarEstoque(int quantidade) {
        if (quantidade <= 0) throw new IllegalArgumentException("Quantidade deve ser positiva");
        this.estoque += quantidade;
    }

    public void removerEstoque(int quantidade) {
        if (quantidade > this.estoque) throw new EstoqueInsuficienteException(this.id);
        this.estoque -= quantidade;
    }

    public boolean temEstoqueBaixo() {
        return this.estoque <= this.estoqueMinimo;
    }

    public void desativar() { this.ativo = false; }
    public void ativar() { this.ativo = true; }

    // Getters (sem setters públicos — usa métodos de negócio)
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public BigDecimal getPreco() { return preco; }
    public BigDecimal getCusto() { return custo; }
    public Integer getEstoque() { return estoque; }
    public boolean isAtivo() { return ativo; }
    public CategoriaProduto getCategoria() { return categoria; }
    public LocalDateTime getCriadoEm() { return criadoEm; }
    public LocalDateTime getAtualizadoEm() { return atualizadoEm; }

    // Para PUT/UPDATE — método controlado
    public void atualizar(String nome, String descricao, BigDecimal preco) {
        this.nome = nome;
        this.descricao = descricao;
        this.preco = preco;
    }
}
```

---

## Próximas notas
- [[23 - Relacionamentos OneToMany ManyToOne ManyToMany]] — o tópico mais complexo do JPA
- [[24 - JpaRepository e Query Methods]] — repositórios e queries
