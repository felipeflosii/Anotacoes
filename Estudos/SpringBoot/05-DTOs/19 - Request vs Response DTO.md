# 19 — Request vs Response DTO

tags: #springboot #dto #request #response
links: [[18 - O que são DTOs e Por que Usar]] | [[20 - Mapeamento Manual e MapStruct]] | [[29 - Bean Validation]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Os dois papéis do DTO

```
Cliente HTTP
    │
    │  POST /produtos  { "nome": "Teclado", "preco": 149.90 }
    ▼
ProdutoRequest      ← DTO de ENTRADA (o que o cliente manda para o servidor)
    │
    ▼  (Service processa, cria entidade, persiste)
    │
    ▼
ProdutoResponse     ← DTO de SAÍDA (o que o servidor retorna ao cliente)
    │
    ▼
Cliente HTTP
    { "id": 1, "nome": "Teclado", "preco": 149.90, "criadoEm": "2024-01-15" }
```

---

## Request DTO — entrada de dados

O Request DTO representa os dados que **chegam do cliente**. Suas responsabilidades:
- Definir o contrato de entrada da API
- Carregar as anotações de validação (`@NotBlank`, `@Email`, etc.)
- Nunca ter campos gerados pelo servidor (id, criadoEm, etc.)

```java
// Criação — campos obrigatórios + validações
public record ProdutoRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 200, message = "Nome deve ter entre 2 e 200 caracteres")
    String nome,

    @Size(max = 1000, message = "Descrição deve ter no máximo 1000 caracteres")
    String descricao,  // opcional — sem @NotBlank

    @NotNull(message = "Preço é obrigatório")
    @Positive(message = "Preço deve ser positivo")
    @DecimalMax(value = "99999.99", message = "Preço máximo excedido")
    BigDecimal preco,

    @NotNull(message = "Estoque é obrigatório")
    @Min(value = 0, message = "Estoque não pode ser negativo")
    Integer estoque,

    @NotNull(message = "Categoria é obrigatória")
    Long categoriaId  // referência por ID, não o objeto completo
) {}
```

```java
// Atualização completa (PUT) — mesmos campos, pode ter validações diferentes
public record ProdutoUpdateRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 200)
    String nome,

    String descricao,

    @NotNull @Positive
    BigDecimal preco,

    @NotNull @Min(0)
    Integer estoque
    // categoriaId pode não ser atualizável — omitido intencionalmente
) {}
```

```java
// Atualização parcial (PATCH) — DTOs específicos por operação
public record AtualizarPrecoRequest(
    @NotNull(message = "Novo preço é obrigatório")
    @Positive(message = "Preço deve ser positivo")
    BigDecimal novoPreco
) {}

public record AtualizarEstoqueRequest(
    @NotNull @Min(0)
    Integer novaQuantidade,

    @NotBlank
    String motivo  // rastreabilidade da mudança de estoque
) {}
```

---

## Response DTO — saída de dados

O Response DTO representa o que **o servidor retorna**. Suas responsabilidades:
- Expor apenas o que o cliente precisa ver
- Nunca expor dados sensíveis
- Pode ter formatos diferentes por contexto

```java
// Resposta completa — tela de detalhe
public record ProdutoResponse(
    Long id,
    String nome,
    String descricao,
    BigDecimal preco,
    Integer estoque,
    boolean ativo,
    CategoriaResponse categoria,   // objeto aninhado
    LocalDateTime criadoEm,
    LocalDateTime atualizadoEm
) {
    // Factory method — converte entidade para DTO
    public static ProdutoResponse from(Produto produto) {
        return new ProdutoResponse(
            produto.getId(),
            produto.getNome(),
            produto.getDescricao(),
            produto.getPreco(),
            produto.getEstoque(),
            produto.isAtivo(),
            CategoriaResponse.from(produto.getCategoria()),
            produto.getCriadoEm(),
            produto.getAtualizadoEm()
        );
    }
}
```

```java
// Resposta resumida — listagens e dropdowns
public record ProdutoResumoResponse(
    Long id,
    String nome,
    BigDecimal preco,
    boolean ativo
) {
    public static ProdutoResumoResponse from(Produto produto) {
        return new ProdutoResumoResponse(
            produto.getId(),
            produto.getNome(),
            produto.getPreco(),
            produto.isAtivo()
        );
    }
}
```

```java
// Resposta para admin — campos extras que só admin vê
public record ProdutoAdminResponse(
    Long id,
    String nome,
    BigDecimal preco,
    BigDecimal custo,           // custo do produto — só admin vê
    BigDecimal margemLucro,     // calculado
    Integer estoque,
    Integer estoqueMinimo,
    boolean ativo,
    String fornecedor           // dados internos
) {
    public static ProdutoAdminResponse from(Produto produto) {
        BigDecimal margem = produto.getPreco()
            .subtract(produto.getCusto())
            .divide(produto.getPreco(), 2, RoundingMode.HALF_UP)
            .multiply(BigDecimal.valueOf(100));

        return new ProdutoAdminResponse(
            produto.getId(),
            produto.getNome(),
            produto.getPreco(),
            produto.getCusto(),
            margem,
            produto.getEstoque(),
            produto.getEstoqueMinimo(),
            produto.isAtivo(),
            produto.getFornecedor()
        );
    }
}
```

---

## DTOs aninhados — relacionamentos

Quando a entidade tem relacionamentos, represente-os com DTOs aninhados:

```java
// Entidades
public class Pedido {
    private Long id;
    private Cliente cliente;
    private List<ItemPedido> itens;
    private BigDecimal total;
    private StatusPedido status;
    private LocalDateTime criadoEm;
}

public class ItemPedido {
    private Long id;
    private Produto produto;
    private Integer quantidade;
    private BigDecimal precoUnitario;
}
```

```java
// DTO de resposta com relacionamentos aninhados
public record PedidoResponse(
    Long id,
    ClienteResumoResponse cliente,    // DTO do cliente (só id e nome)
    List<ItemPedidoResponse> itens,
    BigDecimal total,
    String status,
    LocalDateTime criadoEm
) {
    public static PedidoResponse from(Pedido pedido) {
        return new PedidoResponse(
            pedido.getId(),
            ClienteResumoResponse.from(pedido.getCliente()),
            pedido.getItens().stream()
                .map(ItemPedidoResponse::from)
                .toList(),
            pedido.getTotal(),
            pedido.getStatus().name(),
            pedido.getCriadoEm()
        );
    }
}

public record ItemPedidoResponse(
    Long id,
    Long produtoId,
    String produtoNome,
    Integer quantidade,
    BigDecimal precoUnitario,
    BigDecimal subtotal
) {
    public static ItemPedidoResponse from(ItemPedido item) {
        return new ItemPedidoResponse(
            item.getId(),
            item.getProduto().getId(),
            item.getProduto().getNome(),
            item.getQuantidade(),
            item.getPrecoUnitario(),
            item.getPrecoUnitario().multiply(BigDecimal.valueOf(item.getQuantidade()))
        );
    }
}

// DTO de criação com relacionamentos por ID
public record PedidoRequest(

    @NotNull(message = "Cliente é obrigatório")
    Long clienteId,

    @NotEmpty(message = "Pedido deve ter pelo menos 1 item")
    @Valid  // valida cada item da lista
    List<ItemPedidoRequest> itens
) {}

public record ItemPedidoRequest(

    @NotNull(message = "Produto é obrigatório")
    Long produtoId,

    @NotNull @Min(value = 1, message = "Quantidade mínima é 1")
    Integer quantidade
) {}
```

---

## Projeções com interface — alternativa aos DTOs

O Spring Data JPA suporta **projeções de interface** que geram DTOs automaticamente a partir de queries:

```java
// Interface de projeção — Spring gera a implementação
public interface ProdutoResumoProjection {
    Long getId();
    String getNome();
    BigDecimal getPreco();
}

// Repository usando projeção
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    List<ProdutoResumoProjection> findByAtivoTrue();  // retorna só id, nome, preco
}

// Projeção com campo calculado
public interface ProdutoComMargemProjection {
    Long getId();
    String getNome();
    BigDecimal getPreco();
    BigDecimal getCusto();

    // Campo calculado na projeção
    default BigDecimal getMargem() {
        return getPreco().subtract(getCusto());
    }
}
```

> Projeções são úteis para queries de leitura onde você quer evitar carregar a entidade completa. Para operações de escrita, use DTOs normais.

---

## Padrão de nomenclatura

```
Entidade: Produto

Request DTOs (entrada):
  ProdutoRequest           → criação (POST)
  ProdutoUpdateRequest     → atualização completa (PUT)
  AtualizarPrecoRequest    → atualização parcial (PATCH)

Response DTOs (saída):
  ProdutoResponse          → detalhe completo (GET /{id})
  ProdutoResumoResponse    → listagem resumida (GET /)
  ProdutoAdminResponse     → view de admin
```

---

## Próximas notas
- [[20 - Mapeamento Manual e MapStruct]] — como converter entidade ↔ DTO
- [[29 - Bean Validation]] — validações nos Request DTOs em detalhe
