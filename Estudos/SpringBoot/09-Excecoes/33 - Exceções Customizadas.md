# 33 — Exceções Customizadas

tags: #springboot #exceções #customizadas #boaspraticas
links: [[32 - ControllerAdvice e ExceptionHandler]] | [[34 - Padrão de Resposta de Erro]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Por que criar exceções próprias

```java
// ❌ Sem exceções customizadas — genérico e sem semântica
throw new RuntimeException("Cliente não encontrado com id: " + id);
throw new IllegalArgumentException("E-mail já cadastrado");
throw new RuntimeException("Estoque insuficiente");

// ✅ Com exceções customizadas — semântico, rastreável e tratável
throw new RecursoNaoEncontradoException("Cliente", id);
throw new EmailJaCadastradoException(email);
throw new EstoqueInsuficienteException(produto.getId(), quantidadeSolicitada);
```

Com exceções customizadas, o `GlobalExceptionHandler` consegue tratar cada tipo adequadamente e retornar o HTTP status correto.

---

## Hierarquia de exceções do projeto

```
RuntimeException
│
├── ApiException (base do projeto)
│   │
│   ├── RecursoNaoEncontradoException        → 404
│   ├── RegraDeNegocioException              → 422
│   ├── RecursoJaExisteException             → 409
│   ├── AcessoNegadoException                → 403
│   └── ServicoExternoException              → 502
│
└── (outras do Java/Spring: tratadas pelo fallback)
```

---

## Implementação da hierarquia

```java
// Base — todas as exceções do projeto herdam desta
public abstract class ApiException extends RuntimeException {

    private final int httpStatus;

    protected ApiException(String mensagem, int httpStatus) {
        super(mensagem);
        this.httpStatus = httpStatus;
    }

    protected ApiException(String mensagem, int httpStatus, Throwable causa) {
        super(mensagem, causa);
        this.httpStatus = httpStatus;
    }

    public int getHttpStatus() {
        return httpStatus;
    }
}
```

```java
// 404 — recurso não encontrado
public class RecursoNaoEncontradoException extends ApiException {

    public RecursoNaoEncontradoException(String recurso, Long id) {
        super(String.format("%s com id %d não encontrado", recurso, id), 404);
    }

    public RecursoNaoEncontradoException(String recurso, String identificador) {
        super(String.format("%s '%s' não encontrado", recurso, identificador), 404);
    }

    // Uso:
    // throw new RecursoNaoEncontradoException("Cliente", 42L);
    // throw new RecursoNaoEncontradoException("Cliente", "felipe@ex.com");
}
```

```java
// 409 — recurso já existe
public class RecursoJaExisteException extends ApiException {

    public RecursoJaExisteException(String recurso, String campo, String valor) {
        super(String.format("%s com %s '%s' já existe", recurso, campo, valor), 409);
    }

    // Uso:
    // throw new RecursoJaExisteException("Cliente", "e-mail", "felipe@ex.com");
}
```

```java
// 422 — regra de negócio violada
public class RegraDeNegocioException extends ApiException {

    public RegraDeNegocioException(String mensagem) {
        super(mensagem, 422);
    }

    // Uso:
    // throw new RegraDeNegocioException("Pedido não pode ser cancelado após envio");
    // throw new RegraDeNegocioException("Preço não pode ser menor que o custo");
}
```

```java
// 422 — estoque insuficiente (caso específico de negócio)
public class EstoqueInsuficienteException extends RegraDeNegocioException {

    public EstoqueInsuficienteException(Long produtoId, int solicitado, int disponivel) {
        super(String.format(
            "Estoque insuficiente para o produto %d: solicitado=%d, disponível=%d",
            produtoId, solicitado, disponivel
        ));
    }
}
```

```java
// 403 — sem permissão
public class AcessoNegadoException extends ApiException {

    public AcessoNegadoException(String mensagem) {
        super(mensagem, 403);
    }

    public AcessoNegadoException() {
        super("Você não tem permissão para realizar esta operação", 403);
    }
}
```

```java
// 502 — serviço externo falhou
public class ServicoExternoException extends ApiException {

    public ServicoExternoException(String servico, Throwable causa) {
        super(String.format("Falha na comunicação com o serviço '%s'", servico), 502, causa);
    }
}
```

---

## Usando no Service

```java
@Service
public class ProdutoService {

    @Transactional
    public void adicionarAoCarrinho(Long produtoId, int quantidade) {

        Produto produto = produtoRepository.findById(produtoId)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Produto", produtoId));

        if (!produto.isAtivo()) {
            throw new RegraDeNegocioException(
                String.format("Produto '%s' não está disponível para compra", produto.getNome())
            );
        }

        if (produto.getEstoque() < quantidade) {
            throw new EstoqueInsuficienteException(produtoId, quantidade, produto.getEstoque());
        }

        produto.removerEstoque(quantidade);
    }

    @Transactional
    public ProdutoResponse criar(ProdutoRequest request) {

        if (produtoRepository.existsByCodigo(request.codigo())) {
            throw new RecursoJaExisteException("Produto", "código", request.codigo());
        }

        var produto = new Produto(request.nome(), request.preco(), request.codigo());
        return ProdutoResponse.from(produtoRepository.save(produto));
    }
}
```

---

## Handler simplificado para ApiException

Com a hierarquia base `ApiException`, o handler fica mais limpo:

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // Um handler para TODAS as exceções do projeto (que herdam de ApiException)
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<ErrorResponse> handleApiException(
        ApiException ex, HttpServletRequest req
    ) {
        log.warn("{} - {}: {}", ex.getHttpStatus(), req.getRequestURI(), ex.getMessage());

        ErrorResponse body = ErrorResponse.of(
            ex.getHttpStatus(),
            HttpStatus.valueOf(ex.getHttpStatus()).getReasonPhrase(),
            ex.getMessage(),
            req.getRequestURI()
        );

        return ResponseEntity.status(ex.getHttpStatus()).body(body);
    }

    // Handlers específicos para exceções do Spring/Java...
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex, HttpServletRequest req) { ... }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenerico(Exception ex, HttpServletRequest req) { ... }
}
```

---

## Onde criar os arquivos

```
src/main/java/com/empresa/api/
└── exception/
    ├── ApiException.java                    ← base abstrata
    ├── RecursoNaoEncontradoException.java   ← 404
    ├── RecursoJaExisteException.java        ← 409
    ├── RegraDeNegocioException.java         ← 422
    ├── EstoqueInsuficienteException.java    ← 422 (específica)
    ├── AcessoNegadoException.java           ← 403
    ├── ServicoExternoException.java         ← 502
    ├── GlobalExceptionHandler.java          ← handler
    └── ErrorResponse.java                  ← DTO de erro
```

---

## Próximas notas
- [[34 - Padrão de Resposta de Erro]] — o ErrorResponse completo e padronizado
