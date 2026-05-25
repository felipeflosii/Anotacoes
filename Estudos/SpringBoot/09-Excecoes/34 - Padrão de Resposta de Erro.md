# 34 — Padrão de Resposta de Erro

tags: #springboot #exceções #erros #padrão #api
links: [[32 - ControllerAdvice e ExceptionHandler]] | [[33 - Exceções Customizadas]] | [[39 - Padrão REST Completo]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Por que padronizar respostas de erro

Sem padrão, cada erro da API fica diferente:

```json
// Erro de validação (Spring padrão):
{ "timestamp": "...", "status": 400, "error": "Bad Request", "path": "/api/clientes" }

// Erro de negócio (você mesmo):
{ "message": "E-mail já existe" }

// Erro do banco (vazando):
{ "erro": "could not execute statement; constraint [uk_email]..." }

// Erro 404 default do Spring:
{ "timestamp": "...", "status": 404, "error": "Not Found", "path": "..." }
```

Com padrão, **todo erro tem o mesmo formato** — o frontend sabe exatamente como tratar.

---

## O ErrorResponse completo

```java
package com.empresa.api.exception;

import com.fasterxml.jackson.annotation.JsonInclude;
import java.time.LocalDateTime;
import java.util.List;

// @JsonInclude: campos null não aparecem no JSON
@JsonInclude(JsonInclude.Include.NON_NULL)
public record ErrorResponse(

    int status,            // código HTTP numérico: 400, 404, 422...
    String erro,           // categoria curta: "Não encontrado", "Erro de validação"
    String mensagem,       // descrição legível para o usuário/desenvolvedor
    String caminho,        // URL que causou o erro: "/api/v1/clientes"
    LocalDateTime timestamp,
    List<CampoErro> erros  // null para erros simples, lista para erros de validação

) {
    // Construtor para erros simples
    public static ErrorResponse of(int status, String erro, String mensagem, String caminho) {
        return new ErrorResponse(status, erro, mensagem, caminho, LocalDateTime.now(), null);
    }

    // Construtor para erros de validação (com lista de campos)
    public static ErrorResponse ofValidation(int status, String caminho, List<CampoErro> erros) {
        return new ErrorResponse(
            status,
            "Erro de validação",
            "Um ou mais campos possuem valores inválidos. Verifique os detalhes em 'erros'.",
            caminho,
            LocalDateTime.now(),
            erros
        );
    }
}
```

```java
// Detalhe de cada campo com erro de validação
public record CampoErro(
    String campo,           // nome do campo: "email", "preco", "itens[0].quantidade"
    Object valorRejeitado,  // o valor que veio na requisição (pode ser null)
    String mensagem         // mensagem de erro: "E-mail inválido"
) {}
```

---

## Exemplos de cada tipo de resposta

### 400 — Validação falhou

```json
POST /api/v1/clientes
Body: { "nome": "", "email": "invalido", "telefone": "abc" }

Response 400:
{
  "status": 400,
  "erro": "Erro de validação",
  "mensagem": "Um ou mais campos possuem valores inválidos. Verifique os detalhes em 'erros'.",
  "caminho": "/api/v1/clientes",
  "timestamp": "2024-01-15T10:30:00",
  "erros": [
    { "campo": "email",    "valorRejeitado": "invalido", "mensagem": "Formato de e-mail inválido" },
    { "campo": "nome",     "valorRejeitado": "",          "mensagem": "Campo obrigatório" },
    { "campo": "telefone", "valorRejeitado": "abc",       "mensagem": "Telefone deve ter 10 ou 11 dígitos" }
  ]
}
```

### 404 — Recurso não encontrado

```json
GET /api/v1/clientes/999

Response 404:
{
  "status": 404,
  "erro": "Recurso não encontrado",
  "mensagem": "Cliente com id 999 não encontrado",
  "caminho": "/api/v1/clientes/999",
  "timestamp": "2024-01-15T10:30:00"
}
```

### 409 — Conflito (recurso já existe)

```json
POST /api/v1/clientes
Body: { "email": "felipe@ex.com" }  ← email já existe

Response 409:
{
  "status": 409,
  "erro": "Conflito",
  "mensagem": "Cliente com e-mail 'felipe@ex.com' já existe",
  "caminho": "/api/v1/clientes",
  "timestamp": "2024-01-15T10:30:00"
}
```

### 422 — Regra de negócio

```json
PATCH /api/v1/pedidos/10/cancelar

Response 422:
{
  "status": 422,
  "erro": "Regra de negócio violada",
  "mensagem": "Pedido não pode ser cancelado após despacho para entrega",
  "caminho": "/api/v1/pedidos/10/cancelar",
  "timestamp": "2024-01-15T10:30:00"
}
```

### 500 — Erro interno (sem vazar detalhes)

```json
Response 500:
{
  "status": 500,
  "erro": "Erro interno do servidor",
  "mensagem": "Ocorreu um erro inesperado. Por favor, tente novamente mais tarde.",
  "caminho": "/api/v1/pedidos",
  "timestamp": "2024-01-15T10:30:00"
}
```

---

## Desabilitando as respostas de erro padrão do Spring

Por padrão, o Spring Boot tem suas próprias páginas de erro (`/error`). Para APIs REST, substitua completamente:

```yaml
# application.yml
server:
  error:
    include-message: never        # não inclui mensagem técnica na resposta padrão
    include-binding-errors: never # não inclui erros de binding
    include-stacktrace: never     # NUNCA exponha stack trace
    include-exception: false      # não inclui nome da classe de exceção
```

```java
// Para desabilitar completamente o /error endpoint (opcional):
@SpringBootApplication(exclude = {ErrorMvcAutoConfiguration.class})
public class MinhaApiApplication { ... }
```

---

## Tabela de referência — status por situação

| Situação | Status | Erro |
|---|---|---|
| Dados inválidos (formato) | 400 | Bad Request |
| Token ausente / inválido | 401 | Unauthorized |
| Sem permissão | 403 | Forbidden |
| Recurso não existe | 404 | Not Found |
| Método HTTP incorreto | 405 | Method Not Allowed |
| Recurso já existe (duplicata) | 409 | Conflict |
| Regra de negócio violada | 422 | Unprocessable Entity |
| Muitas requisições | 429 | Too Many Requests |
| Erro inesperado no servidor | 500 | Internal Server Error |
| Serviço externo indisponível | 502 | Bad Gateway |
| Servidor sobrecarregado | 503 | Service Unavailable |

---

## Próximas notas
- [[35 - Introdução ao Spring Security]] — começando o módulo de segurança
- [[39 - Padrão REST Completo]] — boas práticas completas de REST
