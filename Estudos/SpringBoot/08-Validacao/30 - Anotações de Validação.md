# 30 — Anotações de Validação

tags: #springboot #validação #anotações #beanvalidation
links: [[29 - Bean Validation]] | [[31 - Tratamento de Erros de Validação]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Catálogo completo de anotações

### Nulidade e presença

| Anotação | Valida | Aceita null? |
|---|---|---|
| `@NotNull` | Não é null | ❌ |
| `@Null` | É null | ✅ |
| `@NotEmpty` | Não é null e não é vazio (`""`, `[]`, `{}`) | ❌ |
| `@NotBlank` | Não é null, não vazio e não só espaços | ❌ |

```java
public record ExemploRequest(
    @NotNull   Object qualquerCoisa,       // null → erro
    @NotEmpty  String textoOuLista,        // "" ou [] → erro, mas "  " é válido
    @NotBlank  String texto,               // "", "  " → erro
    @NotEmpty  List<String> lista          // [] → erro
) {}
```

> **Regra prática:** para Strings, prefira `@NotBlank`. Para objetos e listas, use `@NotNull` ou `@NotEmpty`.

---

### Tamanho e comprimento

```java
// Strings
@Size(min = 2, max = 150)       String nome;
@Size(min = 8)                  String senha;
@Size(max = 500)                String descricao;     // mínimo padrão é 0
@Length(min = 2, max = 150)     String nome;          // Hibernate específico — prefira @Size

// Coleções e arrays
@Size(min = 1, max = 50)        List<String> tags;    // entre 1 e 50 itens
@Size(min = 1)                  List<Item> itens;     // pelo menos 1 item
```

---

### Números

```java
// Inteiros
@Min(0)                   Integer estoque;         // >= 0
@Max(1000)                Integer limite;          // <= 1000
@Min(1) @Max(5)           Integer nota;            // entre 1 e 5
@Positive                 Long id;                 // > 0
@PositiveOrZero           Integer quantidade;      // >= 0
@Negative                 Integer debito;          // < 0
@NegativeOrZero           Integer ajuste;          // <= 0

// Decimais
@DecimalMin("0.01")       BigDecimal preco;        // >= 0.01
@DecimalMax("99999.99")   BigDecimal preco;        // <= 99999.99
@Digits(integer=10, fraction=2)  BigDecimal valor; // max 10 dígitos inteiros, 2 decimais

// Forma mais limpa para preço:
@NotNull
@Positive
@DecimalMax(value = "999999.99", message = "Preço máximo é R$ 999.999,99")
BigDecimal preco;
```

---

### Formato e padrão

```java
// E-mail
@Email                    String email;
@Email(message = "Formato de e-mail inválido")  String email;

// Expressão regular
@Pattern(regexp = "\\d{10,11}", message = "Telefone deve ter 10 ou 11 dígitos")
String telefone;

@Pattern(regexp = "^\\d{5}-\\d{3}$", message = "CEP deve estar no formato 00000-000")
String cep;

@Pattern(regexp = "^[A-Z]{2}$", message = "Estado deve ser a sigla com 2 letras maiúsculas")
String estado;

// URLs
@URL                      String website;
```

---

### Datas

```java
// Passado e futuro
@Past                   LocalDate dataNascimento;     // data passada
@PastOrPresent          LocalDateTime atualizadoEm;   // passado ou hoje
@Future                 LocalDate dataEntrega;         // data futura
@FutureOrPresent        LocalDate dataEvento;          // futuro ou hoje
```

---

### Booleano

```java
@AssertTrue     boolean termsAccepted;    // deve ser true
@AssertFalse    boolean bloqueado;        // deve ser false

// Uso comum: aceite de termos
@AssertTrue(message = "Você deve aceitar os termos de uso")
boolean aceitouTermos;
```

---

## Exemplos de DTOs completos e bem validados

### DTO de cadastro de usuário

```java
public record CadastroUsuarioRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 150, message = "Nome deve ter entre 2 e 150 caracteres")
    String nome,

    @NotBlank(message = "E-mail é obrigatório")
    @Email(message = "Formato de e-mail inválido")
    @Size(max = 200, message = "E-mail muito longo")
    String email,

    @NotBlank(message = "Senha é obrigatória")
    @Size(min = 8, max = 100, message = "Senha deve ter entre 8 e 100 caracteres")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$",
        message = "Senha deve conter ao menos: 1 minúscula, 1 maiúscula, 1 número, 1 caractere especial"
    )
    String senha,

    @NotBlank(message = "Telefone é obrigatório")
    @Pattern(regexp = "^\\d{10,11}$", message = "Telefone deve ter 10 ou 11 dígitos (somente números)")
    String telefone,

    @NotNull(message = "Data de nascimento é obrigatória")
    @Past(message = "Data de nascimento deve ser no passado")
    LocalDate dataNascimento,

    @AssertTrue(message = "Você deve aceitar os termos de uso")
    boolean aceitouTermos
) {}
```

### DTO de produto

```java
public record ProdutoRequest(

    @NotBlank(message = "Nome do produto é obrigatório")
    @Size(min = 2, max = 200, message = "Nome deve ter entre 2 e 200 caracteres")
    String nome,

    @Size(max = 1000, message = "Descrição deve ter no máximo 1000 caracteres")
    String descricao,

    @NotNull(message = "Preço é obrigatório")
    @DecimalMin(value = "0.01", message = "Preço mínimo é R$ 0,01")
    @DecimalMax(value = "999999.99", message = "Preço máximo é R$ 999.999,99")
    @Digits(integer = 6, fraction = 2, message = "Preço inválido: máximo 6 dígitos inteiros e 2 decimais")
    BigDecimal preco,

    @NotNull(message = "Estoque é obrigatório")
    @Min(value = 0, message = "Estoque não pode ser negativo")
    @Max(value = 999999, message = "Estoque máximo é 999.999 unidades")
    Integer estoque,

    @NotNull(message = "Categoria é obrigatória")
    @Positive(message = "ID da categoria deve ser positivo")
    Long categoriaId,

    @Size(max = 10, message = "Máximo de 10 tags por produto")
    List<@NotBlank @Size(max = 50) String> tags
) {}
```

### DTO de pedido com itens

```java
public record PedidoRequest(

    @NotNull(message = "Cliente é obrigatório")
    @Positive(message = "ID do cliente inválido")
    Long clienteId,

    @NotEmpty(message = "O pedido deve ter pelo menos 1 item")
    @Size(max = 50, message = "Máximo de 50 itens por pedido")
    @Valid
    List<ItemPedidoRequest> itens,

    @Size(max = 500)
    String observacao,

    @NotBlank(message = "Forma de pagamento é obrigatória")
    @Pattern(
        regexp = "CARTAO_CREDITO|CARTAO_DEBITO|PIX|BOLETO|DINHEIRO",
        message = "Forma de pagamento inválida"
    )
    String formaPagamento
) {}

public record ItemPedidoRequest(

    @NotNull(message = "Produto é obrigatório")
    @Positive(message = "ID do produto inválido")
    Long produtoId,

    @NotNull(message = "Quantidade é obrigatória")
    @Min(value = 1, message = "Quantidade mínima é 1")
    @Max(value = 100, message = "Quantidade máxima por item é 100")
    Integer quantidade
) {}
```

---

## Mensagens de validação — internacionalização

```properties
# src/main/resources/ValidationMessages.properties
# Sobrescreve as mensagens padrão do Hibernate Validator

jakarta.validation.constraints.NotBlank.message=Campo obrigatório
jakarta.validation.constraints.NotNull.message=Campo obrigatório
jakarta.validation.constraints.Size.message=Deve ter entre {min} e {max} caracteres
jakarta.validation.constraints.Email.message=E-mail inválido
jakarta.validation.constraints.Min.message=Valor mínimo é {value}
jakarta.validation.constraints.Max.message=Valor máximo é {value}
jakarta.validation.constraints.Positive.message=Deve ser um número positivo
```

---

## Próximas notas
- [[31 - Tratamento de Erros de Validação]] — como capturar e formatar os erros
- [[32 - ControllerAdvice e ExceptionHandler]] — handler global de exceções
