# 32 — @ControllerAdvice e @ExceptionHandler

tags: #springboot #exceções #controlleradvice #handler
links: [[31 - Tratamento de Erros de Validação]] | [[33 - Exceções Customizadas]] | [[34 - Padrão de Resposta de Erro]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O problema sem um handler global

```java
// Sem handler global, cada controller trata seus próprios erros:
@GetMapping("/{id}")
public ResponseEntity<?> buscar(@PathVariable Long id) {
    try {
        return ResponseEntity.ok(service.buscarPorId(id));
    } catch (RecursoNaoEncontradoException e) {
        return ResponseEntity.status(404).body(Map.of("erro", e.getMessage()));
    } catch (Exception e) {
        return ResponseEntity.status(500).body(Map.of("erro", "Erro interno"));
    }
}

// Resultado: try/catch em TODOS os métodos de TODOS os controllers
// Repetição enorme, resposta inconsistente, difícil manutenção
```

---

## @ControllerAdvice — o handler global

`@ControllerAdvice` é uma classe que intercepta exceções de **todos** os controllers centralizando o tratamento:

```java
@RestControllerAdvice   // = @ControllerAdvice + @ResponseBody
@Slf4j                  // Lombok: injeta logger 'log'
public class GlobalExceptionHandler {

    // Cada método @ExceptionHandler trata um tipo de exceção
    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleRecursoNaoEncontrado(
        RecursoNaoEncontradoException ex,
        HttpServletRequest request
    ) {
        log.warn("Recurso não encontrado: {}", ex.getMessage());
        return ErrorResponse.of(404, "Não encontrado", ex.getMessage(), request.getRequestURI());
    }
}
```

Com isso, o controller fica **limpo**:

```java
@GetMapping("/{id}")
public ResponseEntity<ClienteResponse> buscar(@PathVariable Long id) {
    return ResponseEntity.ok(service.buscarPorId(id));
    // Se lançar RecursoNaoEncontradoException → GlobalExceptionHandler captura
    // Sem try/catch no controller
}
```

---

## GlobalExceptionHandler completo

```java
package com.empresa.api.exception;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.validation.ConstraintViolationException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.*;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;

import java.util.Comparator;
import java.util.List;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // ===================== EXCEÇÕES DE NEGÓCIO ===================== //

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleRecursoNaoEncontrado(
        RecursoNaoEncontradoException ex, HttpServletRequest req
    ) {
        log.warn("404 - {}: {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(404, "Recurso não encontrado", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(RegraDeNegocioException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleRegraDeNegocio(
        RegraDeNegocioException ex, HttpServletRequest req
    ) {
        log.warn("422 - {}: {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(422, "Regra de negócio violada", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(RecursoJaExisteException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleRecursoJaExiste(
        RecursoJaExisteException ex, HttpServletRequest req
    ) {
        log.warn("409 - {}: {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(409, "Conflito", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(AcessoNegadoException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleAcessoNegado(
        AcessoNegadoException ex, HttpServletRequest req
    ) {
        log.warn("403 - Acesso negado em {}: {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(403, "Acesso negado", ex.getMessage(), req.getRequestURI());
    }

    // =================== ERROS DE VALIDAÇÃO =================== //

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(
        MethodArgumentNotValidException ex, HttpServletRequest req
    ) {
        List<CampoErro> erros = ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> new CampoErro(fe.getField(), fe.getRejectedValue(), fe.getDefaultMessage()))
            .sorted(Comparator.comparing(CampoErro::campo))
            .toList();

        log.warn("400 - Validação falhou em {}: {} erro(s)", req.getRequestURI(), erros.size());
        return ErrorResponse.ofValidation(400, req.getRequestURI(), erros);
    }

    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraintViolation(
        ConstraintViolationException ex, HttpServletRequest req
    ) {
        List<CampoErro> erros = ex.getConstraintViolations().stream()
            .map(cv -> {
                String path = cv.getPropertyPath().toString();
                String campo = path.contains(".") ? path.substring(path.lastIndexOf('.') + 1) : path;
                return new CampoErro(campo, cv.getInvalidValue(), cv.getMessage());
            })
            .toList();

        return ErrorResponse.ofValidation(400, req.getRequestURI(), erros);
    }

    // =================== ERROS DE TIPO E FORMATO =================== //

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleTypeMismatch(
        MethodArgumentTypeMismatchException ex, HttpServletRequest req
    ) {
        String msg = String.format(
            "O parâmetro '%s' com valor '%s' não pôde ser convertido para o tipo '%s'",
            ex.getName(), ex.getValue(),
            ex.getRequiredType() != null ? ex.getRequiredType().getSimpleName() : "desconhecido"
        );
        return ErrorResponse.of(400, "Tipo de parâmetro inválido", msg, req.getRequestURI());
    }

    @ExceptionHandler(org.springframework.http.converter.HttpMessageNotReadableException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleJsonInvalido(HttpServletRequest req) {
        return ErrorResponse.of(
            400, "JSON inválido",
            "O corpo da requisição está malformado ou contém tipos de dados incorretos",
            req.getRequestURI()
        );
    }

    // =================== ERROS DE BANCO =================== //

    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDataIntegrity(
        DataIntegrityViolationException ex, HttpServletRequest req
    ) {
        log.error("409 - Violação de integridade em {}: {}", req.getRequestURI(), ex.getMessage());

        // Tenta identificar a causa pela mensagem
        String mensagem = "Operação violou uma restrição de integridade dos dados";
        if (ex.getMessage() != null && ex.getMessage().contains("unique")) {
            mensagem = "Já existe um registro com esses dados";
        }
        return ErrorResponse.of(409, "Conflito de dados", mensagem, req.getRequestURI());
    }

    // =================== SEGURANÇA =================== //

    @ExceptionHandler(org.springframework.security.access.AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleSpringAccessDenied(HttpServletRequest req) {
        return ErrorResponse.of(
            403, "Acesso negado",
            "Você não tem permissão para realizar esta operação",
            req.getRequestURI()
        );
    }

    // =================== FALLBACK — ERRO GENÉRICO =================== //

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenerico(Exception ex, HttpServletRequest req) {
        // SEMPRE logue erros inesperados com stack trace completo
        log.error("500 - Erro inesperado em {}: {}", req.getRequestURI(), ex.getMessage(), ex);

        // NUNCA exponha detalhes internos ao cliente
        return ErrorResponse.of(
            500,
            "Erro interno do servidor",
            "Ocorreu um erro inesperado. Por favor, tente novamente mais tarde.",
            req.getRequestURI()
        );
    }
}
```

---

## Escopo do @ControllerAdvice

Por padrão, `@ControllerAdvice` se aplica a todos os controllers. Você pode limitar:

```java
// Só para controllers de um pacote específico
@ControllerAdvice("com.empresa.api.financeiro")

// Só para controllers de uma anotação específica
@ControllerAdvice(annotations = RestController.class)

// Só para controllers de classes específicas
@ControllerAdvice(assignableTypes = {ClienteController.class, ProdutoController.class})
```

---

## Precedência entre handlers

Quando múltiplas exceções poderiam ser capturadas por diferentes handlers, o Spring escolhe o **mais específico**:

```java
// Hierarquia de exceções:
// Exception (mais genérico)
//   └── RuntimeException
//         └── RecursoNaoEncontradoException (mais específico)

// Spring escolhe o handler de RecursoNaoEncontradoException
// O handler de Exception só pega o que nenhum outro handler capturou
```

---

## Próximas notas
- [[33 - Exceções Customizadas]] — criando a hierarquia de exceções do projeto
- [[34 - Padrão de Resposta de Erro]] — o ErrorResponse completo
