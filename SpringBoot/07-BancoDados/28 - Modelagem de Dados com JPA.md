# 28 — Modelagem de Dados com JPA

tags: #springboot #jpa #modelagem #banco
links: [[26 - Configuração de Banco Relacional]] | [[27 - Flyway Migrations]] | [[23 - Relacionamentos OneToMany ManyToOne ManyToMany]] | [[🗺️ Mapa Principal]]

---

## Boas práticas de modelagem

### 1. Convenções de nomenclatura no banco

```sql
-- Tabelas: snake_case, plural
clientes, pedidos, itens_pedido, categorias_produto

-- Colunas: snake_case, singular
nome, email, criado_em, cliente_id, preco_unitario

-- Chaves primárias: sempre "id"
id BIGSERIAL PRIMARY KEY

-- Chaves estrangeiras: entidade_id
cliente_id, produto_id, categoria_id

-- Índices: idx_{tabela}_{coluna}
idx_clientes_email, idx_pedidos_cliente_id

-- Constraints: {tipo}_{tabela}_{coluna}
uk_clientes_email, chk_produtos_preco_positivo, fk_pedidos_clientes
```

### 2. Tipos de dados adequados

```sql
-- IDs: BIGSERIAL (auto-increment Long)
id BIGSERIAL PRIMARY KEY

-- Textos curtos: VARCHAR com limite
nome       VARCHAR(150)  NOT NULL
email      VARCHAR(200)  NOT NULL
status     VARCHAR(20)   NOT NULL

-- Textos longos: TEXT (sem limite)
descricao  TEXT
observacao TEXT

-- Valores monetários: DECIMAL (nunca FLOAT/DOUBLE — imprecisão)
preco      DECIMAL(10, 2) NOT NULL   -- até R$ 99.999.999,99

-- Booleano: BOOLEAN
ativo      BOOLEAN NOT NULL DEFAULT TRUE

-- Datas: TIMESTAMP (sem timezone para dados locais, TIMESTAMPTZ para UTC)
criado_em  TIMESTAMP NOT NULL DEFAULT NOW()

-- UUID: UUID (PostgreSQL tem tipo nativo)
id         UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

### 3. Campos de auditoria em toda tabela

```sql
-- Toda tabela de negócio deve ter:
criado_em     TIMESTAMP NOT NULL DEFAULT NOW()
atualizado_em TIMESTAMP
-- opcional: deletado_em TIMESTAMP (soft delete)
-- opcional: criado_por VARCHAR(200), atualizado_por VARCHAR(200)
```

---

## Soft Delete — exclusão lógica

Nunca deletar dados de negócio fisicamente. Use soft delete:

```java
@Entity
@Table(name = "clientes")
@SQLRestriction("deletado_em IS NULL")  // Hibernate 6 — filtra automaticamente
// Hibernate 5: @Where(clause = "deletado_em IS NULL")
public class Cliente {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @Column(name = "deletado_em")
    private LocalDateTime deletadoEm;

    // Método de negócio — soft delete
    public void deletar() {
        this.deletadoEm = LocalDateTime.now();
    }

    public boolean isDeletado() {
        return deletadoEm != null;
    }
}
```

```sql
-- Migration correspondente
ALTER TABLE clientes ADD COLUMN deletado_em TIMESTAMP;
CREATE INDEX idx_clientes_deletado_em ON clientes(deletado_em)
    WHERE deletado_em IS NULL;  -- índice parcial — só registros ativos
```

```java
// Com @SQLRestriction, todas as queries ignoram registros deletados automaticamente:
clienteRepository.findAll();              // só retorna não-deletados
clienteRepository.findByEmail(email);     // idem
// Sem precisar de nenhum filtro extra

// Para incluir deletados (admin, auditoria):
@Query(value = "SELECT * FROM clientes WHERE email = ?1", nativeQuery = true)
Optional<Cliente> findByEmailIncluindoDeletados(String email);
```

---

## Chave composta — @EmbeddedId

Quando a chave primária é composta por múltiplas colunas:

```java
// Classe da chave composta — deve implementar Serializable
@Embeddable
public class ItemPedidoId implements Serializable {

    @Column(name = "pedido_id")
    private Long pedidoId;

    @Column(name = "produto_id")
    private Long produtoId;

    protected ItemPedidoId() {}

    public ItemPedidoId(Long pedidoId, Long produtoId) {
        this.pedidoId = pedidoId;
        this.produtoId = produtoId;
    }

    // equals e hashCode OBRIGATÓRIOS para chaves compostas
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ItemPedidoId that)) return false;
        return Objects.equals(pedidoId, that.pedidoId) &&
               Objects.equals(produtoId, that.produtoId);
    }

    @Override
    public int hashCode() {
        return Objects.hash(pedidoId, produtoId);
    }
}

@Entity
@Table(name = "itens_pedido")
public class ItemPedido {

    @EmbeddedId
    private ItemPedidoId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("pedidoId")          // mapeia o campo "pedidoId" da chave composta
    @JoinColumn(name = "pedido_id")
    private Pedido pedido;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("produtoId")
    @JoinColumn(name = "produto_id")
    private Produto produto;

    private Integer quantidade;
    private BigDecimal precoUnitario;
}
```

---

## Herança de entidades — @MappedSuperclass

A forma mais simples e prática de reuso:

```java
// Classe base — não é entidade, não tem tabela
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class EntidadeAuditavel {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    @Column(name = "deletado_em")
    private LocalDateTime deletadoEm;

    public Long getId() { return id; }
    public LocalDateTime getCriadoEm() { return criadoEm; }
    public boolean isDeletado() { return deletadoEm != null; }

    public void deletar() { this.deletadoEm = LocalDateTime.now(); }
}

// Entidades herdam os campos
@Entity @Table(name = "clientes")
public class Cliente extends EntidadeAuditavel {
    private String nome;
    private String email;
    // criadoEm, atualizadoEm, deletadoEm — herdados
}

@Entity @Table(name = "produtos")
public class Produto extends EntidadeAuditavel {
    private String nome;
    private BigDecimal preco;
    // criadoEm, atualizadoEm, deletadoEm — herdados
}
```

---

## Modelagem de domínio — exemplo concessionária

```sql
-- Modelo de dados do projeto completo (Parte 15 da apostila)

CREATE TABLE marcas (
    id        BIGSERIAL    PRIMARY KEY,
    nome      VARCHAR(100) NOT NULL UNIQUE,
    pais      VARCHAR(100),
    ativo     BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE TABLE carros (
    id            BIGSERIAL       PRIMARY KEY,
    modelo        VARCHAR(150)    NOT NULL,
    ano           INTEGER         NOT NULL,
    preco         DECIMAL(12, 2)  NOT NULL,
    cor           VARCHAR(50),
    quilometragem INTEGER         NOT NULL DEFAULT 0,
    status        VARCHAR(20)     NOT NULL DEFAULT 'DISPONIVEL',
    marca_id      BIGINT          NOT NULL REFERENCES marcas(id),
    criado_em     TIMESTAMP       NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP,
    CONSTRAINT chk_ano_valido CHECK (ano BETWEEN 1900 AND 2100),
    CONSTRAINT chk_preco_positivo CHECK (preco > 0)
);

CREATE TABLE clientes (
    id            BIGSERIAL    PRIMARY KEY,
    nome          VARCHAR(150) NOT NULL,
    email         VARCHAR(200) NOT NULL UNIQUE,
    telefone      VARCHAR(20),
    cpf           VARCHAR(14)  UNIQUE,
    ativo         BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em     TIMESTAMP    NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP
);

CREATE TABLE vendas (
    id            BIGSERIAL       PRIMARY KEY,
    carro_id      BIGINT          NOT NULL REFERENCES carros(id),
    cliente_id    BIGINT          NOT NULL REFERENCES clientes(id),
    valor_venda   DECIMAL(12, 2)  NOT NULL,
    data_venda    TIMESTAMP       NOT NULL DEFAULT NOW(),
    forma_pagamento VARCHAR(30)   NOT NULL,
    observacao    TEXT,
    criado_em     TIMESTAMP       NOT NULL DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_carros_marca     ON carros(marca_id);
CREATE INDEX idx_carros_status    ON carros(status);
CREATE INDEX idx_vendas_carro     ON vendas(carro_id);
CREATE INDEX idx_vendas_cliente   ON vendas(cliente_id);
CREATE INDEX idx_vendas_data      ON vendas(data_venda DESC);
```

---

## Próximas notas
- [[29 - Bean Validation]] — validação de dados de entrada
- [[49 - Projeto Concessionária - Visão Geral]] — projeto completo com este modelo
