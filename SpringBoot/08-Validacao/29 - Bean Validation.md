# 29 — Bean Validation

tags: #springboot #validação #beanvalidation #jakarta
links: [[30 - Anotações de Validação]] | [[31 - Tratamento de Erros de Validação]] | [[19 - Request vs Response DTO]] | [[🗺️ Mapa Principal]]

---

## O que é Bean Validation

**Bean Validation** (Jakarta Validation) é a especificação padrão Java para validação de objetos via anotações. O **Hibernate Validator** é a implementação mais usada (já inclusa no `spring-boot-starter-validation`).

A ideia: declare as regras **no próprio DTO** com anotações. O Spring valida automaticamente quando você usa `@Valid` ou `@Validated`.

```xml
<!-- pom.xml — adicionar ao projeto -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## Como a validação funciona no fluxo

```
Cliente envia:  POST /clientes  { "nome": "", "email": "nao-é-email" }
                     ↓
@RequestBody @Valid ClienteRequest request
                     ↓
Spring executa Bean Validation no objeto
                     ↓
Encontra violações: nome em branco, email inválido
                     ↓
Lança: MethodArgumentNotValidException
                     ↓
@ControllerAdvice captura e retorna 400 Bad Request com detalhes
```

---

## @Valid vs @Validated

```java
// @Valid — padrão Jakarta, validação simples de objetos
@PostMapping
public ResponseEntity<?> criar(@RequestBody @Valid ClienteRequest request) { }

// @Validated — Spring, suporta grupos de validação + valida parâmetros de método
@Validated  // colocar na CLASSE para habilitar validação de @PathVariable e @RequestParam
@RestController
public class ClienteController {

    @GetMapping("/{id}")
    public ResponseEntity<?> buscar(
        @PathVariable @Positive Long id  // valida o PathVariable
    ) { }
}

// @Validated também aceita grupos:
@PostMapping
public ResponseEntity<?> criar(
    @RequestBody @Validated(CriacaoGroup.class) ClienteRequest request
) { }
```

---

## Validando objetos aninhados

```java
public record PedidoRequest(

    @NotNull Long clienteId,

    @NotEmpty
    @Valid  // ← @Valid nos objetos aninhados para validar cada item da lista
    List<ItemPedidoRequest> itens
) {}

public record ItemPedidoRequest(
    @NotNull Long produtoId,
    @NotNull @Min(1) Integer quantidade
) {}

// Sem @Valid na lista: os campos de ItemPedidoRequest NÃO são validados
// Com @Valid na lista: cada ItemPedidoRequest é validado individualmente
```

---

## Grupos de validação — validações condicionais

Às vezes você quer validações diferentes na criação vs na atualização:

```java
// Definição dos grupos (interfaces vazias — só marcadores)
public interface CriacaoGroup {}
public interface AtualizacaoGroup {}

// DTO com grupos
public record UsuarioRequest(

    @NotNull(groups = CriacaoGroup.class)       // só na criação
    @Null(groups = AtualizacaoGroup.class)       // na atualização, deve ser null
    String senha,

    @NotBlank(groups = {CriacaoGroup.class, AtualizacaoGroup.class})
    String nome,

    @NotBlank(groups = CriacaoGroup.class)       // só obrigatório na criação
    String email
) {}

// Controller usando grupos
@PostMapping
public ResponseEntity<?> criar(
    @RequestBody @Validated(CriacaoGroup.class) UsuarioRequest request
) { }

@PutMapping("/{id}")
public ResponseEntity<?> atualizar(
    @PathVariable Long id,
    @RequestBody @Validated(AtualizacaoGroup.class) UsuarioRequest request
) { }
```

---

## Criando validações customizadas

Quando as anotações padrão não são suficientes:

```java
// 1. Defina a anotação
@Documented
@Constraint(validatedBy = CpfValidator.class)  // classe que implementa a lógica
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface CPF {
    String message() default "CPF inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implemente o validador
public class CpfValidator implements ConstraintValidator<CPF, String> {

    @Override
    public void initialize(CPF annotation) {}

    @Override
    public boolean isValid(String cpf, ConstraintValidatorContext context) {
        if (cpf == null || cpf.isBlank()) return true;  // deixe @NotBlank tratar o null

        String apenasDigitos = cpf.replaceAll("[^0-9]", "");
        if (apenasDigitos.length() != 11) return false;
        if (apenasDigitos.matches("(\\d)\\1{10}")) return false;  // todos iguais

        return validarDigitosCPF(apenasDigitos);
    }

    private boolean validarDigitosCPF(String cpf) {
        int soma = 0;
        for (int i = 0; i < 9; i++) soma += (cpf.charAt(i) - '0') * (10 - i);
        int primeiroDigito = 11 - (soma % 11);
        if (primeiroDigito >= 10) primeiroDigito = 0;
        if (primeiroDigito != (cpf.charAt(9) - '0')) return false;

        soma = 0;
        for (int i = 0; i < 10; i++) soma += (cpf.charAt(i) - '0') * (11 - i);
        int segundoDigito = 11 - (soma % 11);
        if (segundoDigito >= 10) segundoDigito = 0;
        return segundoDigito == (cpf.charAt(10) - '0');
    }
}

// 3. Use no DTO
public record ClienteRequest(
    @NotBlank String nome,
    @NotBlank @Email String email,
    @CPF String cpf  // sua anotação customizada
) {}
```

### Validação com acesso a banco — Bean de Spring no Validador

```java
// Validação que verifica unicidade no banco
@Documented
@Constraint(validatedBy = EmailUnicoValidator.class)
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface EmailUnico {
    String message() default "E-mail já cadastrado";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Component  // precisa ser @Component para injetar dependências
public class EmailUnicoValidator implements ConstraintValidator<EmailUnico, String> {

    private final ClienteRepository repository;

    public EmailUnicoValidator(ClienteRepository repository) {
        this.repository = repository;
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) return true;
        return !repository.existsByEmail(email);
    }
}

// No DTO:
public record ClienteRequest(
    @NotBlank @Email @EmailUnico String email,  // valida unicidade automaticamente
    @NotBlank String nome
) {}
```

> ⚠️ Validações de unicidade no DTO criam acoplamento entre o DTO e o banco. Muitos projetos preferem lançar exceção no Service (mais explícito). Ambas as abordagens funcionam.

---

## Validação em Service — @Validated no Service

```java
// Não só controllers — Services também podem ter validação
@Service
@Validated
public class ClienteService {

    public ClienteResponse buscarPorId(
        @NotNull(message = "ID não pode ser nulo")
        @Positive(message = "ID deve ser positivo")
        Long id
    ) {
        return repository.findById(id)
            .map(ClienteResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", id));
    }
}
```

---

## Próximas notas
- [[30 - Anotações de Validação]] — catálogo completo de anotações
- [[31 - Tratamento de Erros de Validação]] — capturando e formatando erros de validação
