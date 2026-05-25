# 31 — Tratamento de Erros de Validação

tags: #springboot #validação #erros #handler
links: [[29 - Bean Validation]] | [[30 - Anotações de Validação]] | [[32 - ControllerAdvice e ExceptionHandler]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O que acontece quando a validação falha

Quando `@Valid` detecta violações, o Spring lança:

| Situação | Exceção lançada |
|---|---|
| `@RequestBody @Valid` falha | `MethodArgumentNotValidException` |
| `@RequestParam @Validated` falha | `ConstraintViolationException` |
| `@PathVariable @Validated` falha | `ConstraintViolationException` |
| Tipo de parâmetro incompatível | `MethodArgumentTypeMismatchException` |

Sem um handler, o Spring retorna uma resposta padrão pouco amigável. Você precisa capturar e formatar.

---

## Estrutura de resposta de erro padronizada

Antes de capturar os erros, defina como a resposta vai ser:

```java
// Resposta de erro padrão da API
public record ErrorResponse(
    int status,
    String erro,
    String mensagem,
    String caminho,
    LocalDateTime timestamp,
    List<CampoErro> erros    // null quando não há campos específicos
) {
    // Construtor para erros simples (sem campos)
    public static ErrorResponse of(int status, String erro, String mensagem, String caminho) {
        return new ErrorResponse(status, erro, mensagem, caminho, LocalDateTime.now(), null);
    }

    // Construtor para erros de validação (com campos)
    public static ErrorResponse ofValidation(int status, String caminho, List<CampoErro> erros) {
        return new ErrorResponse(
            status,
            "Erro de validação",
            "Um ou mais campos possuem valores inválidos",
            caminho,
            LocalDateTime.now(),
            erros
        );
    }
}

// Detalhe de cada campo inválido
public record CampoErro(
    String campo,
    Object valorRejeitado,
    String mensagem
) {}
```

### Resposta JSON resultante

```json
// Erro de validação:
{
  "status": 400,
  "erro": "Erro de validação",
  "mensagem": "Um ou mais campos possuem valores inválidos",
  "caminho": "/api/v1/clientes",
  "timestamp": "2024-01-15T10:30:00",
  "erros": [
    {
      "campo": "email",
      "valorRejeitado": "email-invalido",
      "mensagem": "Formato de e-mail inválido"
    },
    {
      "campo": "nome",
      "valorRejeitado": "",
      "mensagem": "Campo obrigatório"
    }
  ]
}

// Erro de negócio (sem campos):
{
  "status": 409,
  "erro": "Conflito",
  "mensagem": "E-mail 'felipe@ex.com' já está cadastrado",
  "caminho": "/api/v1/clientes",
  "timestamp": "2024-01-15T10:30:00",
  "erros": null
}
```

---

## Handler completo de erros de validação

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
@Slf4j
public class GlobalExceptionHandler {

    // ===== ERROS DE VALIDAÇÃO (@RequestBody @Valid) =====
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(
        MethodArgumentNotValidException ex,
        HttpServletRequest request
    ) {
        List<CampoErro> campoErros = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(fieldError -> new CampoErro(
                fieldError.getField(),
                fieldError.getRejectedValue(),
                fieldError.getDefaultMessage()
            ))
            .sorted(Comparator.comparing(CampoErro::campo))  // ordem alfabética
            .toList();

        log.warn("Erro de validação em {}: {} campos inválidos",
            request.getRequestURI(), campoErros.size());

        return ErrorResponse.ofValidation(400, request.getRequestURI(), campoErros);
    }

    // ===== ERROS DE VALIDAÇÃO (@PathVariable, @RequestParam @Validated) =====
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(
        ConstraintViolationException ex,
        HttpServletRequest request
    ) {
        List<CampoErro> campoErros = ex.getConstraintViolations()
            .stream()
            .map(cv -> new CampoErro(
                extrairNomeCampo(cv.getPropertyPath().toString()),
                cv.getInvalidValue(),
                cv.getMessage()
            ))
            .toList();

        return ErrorResponse.ofValidation(400, request.getRequestURI(), campoErros);
    }

    // ===== TIPO DE PARÂMETRO INVÁLIDO (ex: texto onde esperava Long) =====
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleTypeMismatch(
        MethodArgumentTypeMismatchException ex,
        HttpServletRequest request
    ) {
        String mensagem = String.format(
            "Parâmetro '%s' recebeu valor '%s' que não pode ser convertido para %s",
            ex.getName(),
            ex.getValue(),
            ex.getRequiredType() != null ? ex.getRequiredType().getSimpleName() : "tipo esperado"
        );

        return ErrorResponse.of(400, "Parâmetro inválido", mensagem, request.getRequestURI());
    }

    // ===== JSON MALFORMADO =====
    @ExceptionHandler(HttpMessageNotReadableException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleJsonInvalido(
        HttpMessageNotReadableException ex,
        HttpServletRequest request
    ) {
        return ErrorResponse.of(
            400,
            "JSON inválido",
            "O corpo da requisição contém JSON malformado ou tipo de dado incorreto",
            request.getRequestURI()
        );
    }

    // ===== MÉTODO HTTP NÃO SUPORTADO =====
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    public ErrorResponse handleMethodNotAllowed(
        HttpRequestMethodNotSupportedException ex,
        HttpServletRequest request
    ) {
        return ErrorResponse.of(
            405,
            "Método não permitido",
            String.format("Método %s não é suportado para %s", ex.getMethod(), request.getRequestURI()),
            request.getRequestURI()
        );
    }

    // ===== MEDIA TYPE NÃO SUPORTADO =====
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)
    public ErrorResponse handleMediaTypeNotSupported(
        HttpMediaTypeNotSupportedException ex,
        HttpServletRequest request
    ) {
        return ErrorResponse.of(
            415,
            "Media type não suportado",
            String.format("Content-Type '%s' não é suportado. Use 'application/json'", ex.getContentType()),
            request.getRequestURI()
        );
    }

    // Utilitário — extrai nome do campo do path de violação
    private String extrairNomeCampo(String propertyPath) {
        String[] parts = propertyPath.split("\\.");
        return parts[parts.length - 1];
    }
}
```

---

## Testando a validação manualmente

```bash
# Enviar dados inválidos para testar
curl -X POST http://localhost:8080/api/v1/clientes \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "",
    "email": "email-invalido",
    "telefone": "abc"
  }'

# Resposta esperada: 400 Bad Request
{
  "status": 400,
  "erro": "Erro de validação",
  "mensagem": "Um ou mais campos possuem valores inválidos",
  "caminho": "/api/v1/clientes",
  "timestamp": "2024-01-15T10:30:00",
  "erros": [
    { "campo": "email", "valorRejeitado": "email-invalido", "mensagem": "Formato de e-mail inválido" },
    { "campo": "nome",  "valorRejeitado": "", "mensagem": "Campo obrigatório" },
    { "campo": "telefone", "valorRejeitado": "abc", "mensagem": "Telefone deve ter 10 ou 11 dígitos" }
  ]
}
```

---

## Próximas notas
- [[32 - ControllerAdvice e ExceptionHandler]] — handler completo de todas as exceções
- [[33 - Exceções Customizadas]] — criando suas próprias exceções
- [[34 - Padrão de Resposta de Erro]] — padronizando todas as respostas de erro
