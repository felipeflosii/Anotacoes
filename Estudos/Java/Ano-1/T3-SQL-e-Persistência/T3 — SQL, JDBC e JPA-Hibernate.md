---
tags: [java, ano-1, sql, jpa, hibernate, banco-de-dados]
created: 2026-03-24
trimestre: T3
meses: 7-9
status: pendente
---

# T3 · SQL, JDBC e JPA/Hibernate
### Meses 7–9 · Ano 1

> **Objetivo:** Dominar banco de dados relacional com Java — desde SQL puro até JPA/Hibernate de forma eficiente. Saber identificar e resolver N+1 query problem, lazy/eager loading e otimizar queries.

---

## 🔵 Bloco 1 — SQL Avançado

> [!tip] SQL é obrigatório, mesmo usando ORM
> Devs que só sabem usar JPA sem entender SQL são perigosos em produção. Toda query Hibernate vira SQL — você precisa saber o que foi gerado.

### DDL e Modelagem

- `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE`
- **Tipos de dados:** `VARCHAR`, `TEXT`, `INTEGER`, `BIGINT`, `DECIMAL(p,s)`, `BOOLEAN`, `DATE`, `TIMESTAMP`, `UUID`
- **Constraints:** `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`, `DEFAULT`
- Chaves primárias: `SERIAL`/`BIGSERIAL` (PostgreSQL), `AUTO_INCREMENT` (MySQL), UUIDs
- Índices: `CREATE INDEX`, índices compostos, índices parciais, índice único

### DML

- `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- **JOINs profundos:** `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `CROSS JOIN`, `SELF JOIN`
- Subqueries: correlacionadas vs não-correlacionadas
- `EXISTS` vs `IN` — diferença de performance
- CTEs (Common Table Expressions) — `WITH ... AS`
- Window Functions — `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`, `SUM() OVER(PARTITION BY ...)` — **muito cobrado em big techs**

### Agregações e Agrupamentos

- `GROUP BY`, `HAVING`, `DISTINCT`
- `COUNT(*)` vs `COUNT(coluna)` — diferença importante
- Funções de agregação: `SUM`, `AVG`, `MIN`, `MAX`, `STRING_AGG`

### Transações e Isolamento

- `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`
- **Níveis de isolamento:** `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`
- Problemas: Dirty Read, Non-Repeatable Read, Phantom Read
- Deadlocks — como detectar e evitar

### Performance e Otimização

- `EXPLAIN` e `EXPLAIN ANALYZE` — ler plano de execução
- Quando índices ajudam e quando não ajudam
- `VACUUM` e `ANALYZE` no PostgreSQL
- Connection pooling — por que é crítico

### Banco de dados a usar

- **PostgreSQL** — padrão de mercado, use em todos os projetos
- MySQL/MariaDB — legado corporativo, conhecer o básico
- H2 — apenas para testes

---

## 🔵 Bloco 2 — JDBC (a base que o JPA usa)

> [!note] Por que aprender JDBC se temos JPA?
> JPA é uma abstração sobre JDBC. Saber JDBC ajuda a debugar problemas de conexão, entender connection pools e escrever queries nativas quando necessário.

### Conceitos Fundamentais

- `DriverManager` vs `DataSource` — sempre prefira `DataSource`
- `Connection`, `Statement`, `PreparedStatement`, `ResultSet`, `CallableStatement`
- **SQL Injection** — por que `PreparedStatement` é obrigatório
- `try-with-resources` para fechar conexões corretamente

### Connection Pooling

- **HikariCP** — o pool mais rápido, padrão do Spring Boot
- Configurações críticas: `maximumPoolSize`, `minimumIdle`, `connectionTimeout`, `idleTimeout`, `maxLifetime`
- Como dimensionar o pool: **fórmula PostgreSQL** = `(núcleos * 2) + discos spindle`

```yaml
# application.yml — configuração HikariCP
spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

---

## 🔵 Bloco 3 — JPA e Hibernate

### JPA vs Hibernate

- JPA é a **especificação** (javax.persistence / jakarta.persistence)
- Hibernate é a **implementação** mais usada
- EclipseLink existe mas Hibernate domina o mercado

### Entidades e Mapeamentos

#### Mapeamentos Básicos

```java
@Entity
@Table(name = "pedidos")
public class Pedido {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // ou UUID
    private Long id;

    @Column(name = "numero", nullable = false, unique = true, length = 20)
    private String numero;

    @Enumerated(EnumType.STRING) // NUNCA use ORDINAL
    private StatusPedido status;

    @CreationTimestamp  // Hibernate
    private LocalDateTime criadoEm;

    @UpdateTimestamp
    private LocalDateTime atualizadoEm;
}
```

#### Relacionamentos — o ponto mais crítico

| Anotação | Cardinalidade | Lado dono | Problema típico |
|----------|--------------|-----------|-----------------|
| `@OneToOne` | 1:1 | quem tem FK | lazy não funciona bem |
| `@ManyToOne` | N:1 | lado N (tem FK) | — |
| `@OneToMany` | 1:N | lado 1 (mappedBy) | **N+1, use @BatchSize ou JOIN FETCH** |
| `@ManyToMany` | N:N | quem define @JoinTable | cartesian product explosions |

#### FetchType — decisão crítica de performance

```java
// @ManyToOne e @OneToOne: padrão EAGER — mude para LAZY
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "cliente_id")
private Cliente cliente;

// @OneToMany e @ManyToMany: padrão LAZY — mantenha LAZY
@OneToMany(mappedBy = "pedido", fetch = FetchType.LAZY)
private List<ItemPedido> itens;
```

> [!danger] Regra de ouro
> **Sempre use LAZY para tudo**. Carregue explicitamente quando precisar. EAGER em @ManyToOne pode parecer conveniente mas causa explosão de queries em listas.

#### Cascade e Orphan Removal

```java
@OneToMany(mappedBy = "pedido",
           cascade = CascadeType.ALL,
           orphanRemoval = true)
private List<ItemPedido> itens = new ArrayList<>();
```

- `CascadeType.ALL` — persist, merge, remove, refresh, detach
- `orphanRemoval = true` — remove filhos quando removidos da lista
- Evite Cascade em `@ManyToOne` — perigoso

### N+1 Query Problem

> [!danger] O bug de performance #1 em Java
> Ocorre quando você carrega uma lista de N entidades e o Hibernate faz mais N queries para carregar relacionamentos. Em produção com N=10.000 = 10.001 queries.

**Como detectar:** ative o log de SQL

```yaml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        use_sql_comments: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql: TRACE
```

**Soluções:**

```java
// 1. JOIN FETCH (JPQL)
@Query("SELECT p FROM Pedido p JOIN FETCH p.itens WHERE p.cliente = :cliente")
List<Pedido> findByClienteWithItens(Cliente cliente);

// 2. @EntityGraph
@EntityGraph(attributePaths = {"itens", "itens.produto"})
List<Pedido> findByCliente(Cliente cliente);

// 3. @BatchSize (Hibernate-specific)
@BatchSize(size = 30)
@OneToMany(mappedBy = "pedido")
private List<ItemPedido> itens;

// 4. DTO Projection (melhor performance)
@Query("SELECT new br.com.app.dto.PedidoDTO(p.id, p.numero, c.nome) " +
       "FROM Pedido p JOIN p.cliente c")
List<PedidoDTO> findAllProjected();
```

### JPQL e Criteria API

```java
// JPQL
@Query("SELECT p FROM Pedido p WHERE p.status = :status AND p.total > :minTotal")
List<Pedido> findByStatusAndMinTotal(@Param("status") StatusPedido status,
                                     @Param("minTotal") BigDecimal minTotal);

// Native Query (quando JPQL não basta)
@Query(value = "SELECT * FROM pedidos WHERE ...", nativeQuery = true)
List<Pedido> findNative();
```

### Herança em JPA

| Strategy | Tabelas | Performance | Use quando |
|----------|---------|-------------|------------|
| `SINGLE_TABLE` | 1 | Melhor query | Hierarquia simples, poucas colunas opcionais |
| `TABLE_PER_CLASS` | N (sem join) | Pior para polimorfismo | Raro |
| `JOINED` | N (com join) | Balanceada | Hierarquia complexa, sem nulos |

---

## 🔵 Bloco 4 — Spring Data JPA

> [!note] O que você usará no dia a dia
> Spring Data JPA adiciona repositories automáticos sobre JPA, eliminando boilerplate.

### Repository Hierarchy

```
Repository (marker)
  └── CrudRepository<T, ID>
        └── PagingAndSortingRepository<T, ID>
              └── JpaRepository<T, ID>  ← use este
```

### Derived Query Methods

```java
public interface PedidoRepository extends JpaRepository<Pedido, Long> {

    // Spring gera a query automaticamente
    List<Pedido> findByStatus(StatusPedido status);
    List<Pedido> findByClienteNome(String nome);
    List<Pedido> findByTotalGreaterThanAndStatus(BigDecimal total, StatusPedido status);
    Optional<Pedido> findByNumero(String numero);
    boolean existsByNumero(String numero);
    long countByStatus(StatusPedido status);

    // Paginação
    Page<Pedido> findByStatus(StatusPedido status, Pageable pageable);

    // Ordenação
    List<Pedido> findByClienteOrderByTotalDesc(Cliente cliente);
}
```

### Specifications (para queries dinâmicas)

```java
// Para filtros dinâmicos (ex: busca com múltiplos parâmetros opcionais)
public class PedidoSpecs {
    public static Specification<Pedido> comStatus(StatusPedido s) {
        return (root, q, cb) -> s == null ? null : cb.equal(root.get("status"), s);
    }
    public static Specification<Pedido> comTotalAcimaDe(BigDecimal min) {
        return (root, q, cb) -> min == null ? null : cb.greaterThan(root.get("total"), min);
    }
}

// Uso
pedidoRepo.findAll(comStatus(PAGO).and(comTotalAcimaDe(new BigDecimal("100"))));
```

### Transações com Spring

```java
@Service
@Transactional(readOnly = true)  // padrão para leitura
public class PedidoService {

    @Transactional  // sobrescreve para escrita
    public Pedido criarPedido(PedidoCriacaoDTO dto) { ... }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void auditarAcao(String acao) { ... }  // própria transação

    @Transactional(rollbackFor = PedidoException.class)
    public void processarPagamento(Long pedidoId) { ... }
}
```

**Propagations importantes:**
- `REQUIRED` (padrão) — usa existente ou cria nova
- `REQUIRES_NEW` — sempre nova transação
- `SUPPORTS` — usa se existir, sem transação se não
- `MANDATORY` — exige que exista (lança exceção se não)
- `NEVER` — proibido ter transação ativa

> [!warning] @Transactional e proxies
> `@Transactional` só funciona em chamadas externas (via Spring proxy). Se método A chama método B na mesma classe, a transação de B não funciona. Isso é o "self-invocation problem".

---

## 🔵 Bloco 5 — Migrations com Flyway

```java
// Estrutura de migrations
src/main/resources/
  db/migration/
    V1__create_clientes.sql
    V2__create_pedidos.sql
    V3__add_index_pedidos_status.sql
    V4__alter_clientes_add_telefone.sql
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration
```

> [!tip] Regras de ouro para migrations
> 1. Nunca altere uma migration já aplicada em produção
> 2. Migrations devem ser idempotentes quando possível
> 3. Separe migration de schema de migration de dados
> 4. Sempre teste migrations em staging primeiro

---

## 📖 Recursos

| Recurso | Nota |
|---------|------|
| **"Java Persistence with Hibernate" — Bauer/King** | Biblia do Hibernate ⭐⭐⭐⭐⭐ |
| Vlad Mihalcea Blog (vladmihalcea.com) | Melhor blog sobre JPA/Hibernate ⭐⭐⭐⭐⭐ |
| Documentação Spring Data JPA | Oficial |
| PostgreSQL Documentation | Referência SQL completa |
| Use The Index, Luke (use-the-index-luke.com) | Performance de índices |

---

## 🧪 Projeto Prático do Trimestre

> [!example] Mini-projeto: API de Pedidos com PostgreSQL
> - Modelar: Cliente, Produto, Pedido, ItemPedido, Categoria
> - Migrations completas com Flyway
> - Spring Data JPA com queries otimizadas (sem N+1)
> - Paginação e ordenação nos endpoints de listagem
> - Transactions com diferentes propagations
> - DTO Projections para endpoints de listagem
> - Testes de repository com `@DataJpaTest` + H2

---

## 🔗 Navegação

← [[T2 — OOP, SOLID e Design Patterns]]  
→ [[T4 — Web, HTTP e Spring Boot Básico]]
