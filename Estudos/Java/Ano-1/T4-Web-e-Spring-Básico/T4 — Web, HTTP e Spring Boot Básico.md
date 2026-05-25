---
tags: [java, ano-1, spring-boot, rest, api, http]
created: 2026-03-24
trimestre: T4
meses: 10-12
status: pendente
---

# T4 · Web, HTTP e Spring Boot Básico
### Meses 10–12 · Ano 1

> **Objetivo:** Construir APIs REST profissionais com Spring Boot, entendendo o ciclo completo de uma requisição HTTP, validação, tratamento de erros e documentação.

---

## 🔵 Bloco 1 — HTTP Profundo

> [!tip] Devs sênior conhecem HTTP de cor
> Toda API REST é HTTP. Entender profundamente o protocolo é diferencial em entrevistas e resolve problemas de integração.

### Métodos HTTP e Semântica

| Método | Idempotente? | Safe? | Uso correto |
|--------|-------------|-------|-------------|
| `GET` | ✅ | ✅ | Buscar recursos |
| `POST` | ❌ | ❌ | Criar recursos / ações |
| `PUT` | ✅ | ❌ | Substituição completa |
| `PATCH` | ❌ | ❌ | Atualização parcial |
| `DELETE` | ✅ | ❌ | Remover recursos |
| `HEAD` | ✅ | ✅ | Verificar existência |
| `OPTIONS` | ✅ | ✅ | CORS preflight |

### Status Codes — os que todo dev deve saber

```
2xx — Sucesso
  200 OK              — resposta padrão GET/PUT
  201 Created         — recurso criado (POST), inclua Location header
  202 Accepted        — processamento assíncrono
  204 No Content      — DELETE, PATCH sem body
  206 Partial Content — paginação, range requests

3xx — Redirecionamento
  301 Moved Permanently
  302 Found (redirect temporário)
  304 Not Modified    — cache (ETag/If-None-Match)

4xx — Erro do Cliente
  400 Bad Request     — payload inválido
  401 Unauthorized    — não autenticado (nome confuso)
  403 Forbidden       — autenticado mas sem permissão
  404 Not Found       — recurso não existe
  405 Method Not Allowed
  409 Conflict        — violação de unicidade
  410 Gone            — recurso deletado permanentemente
  422 Unprocessable Entity — validação de negócio
  429 Too Many Requests    — rate limiting

5xx — Erro do Servidor
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable  — circuit breaker aberto
  504 Gateway Timeout
```

### Headers Importantes

```
Request Headers:
  Authorization: Bearer <jwt>
  Content-Type: application/json
  Accept: application/json
  X-Request-ID: <uuid>      (rastreabilidade)
  X-Correlation-ID: <uuid>  (rastreabilidade cross-service)
  If-None-Match: "<etag>"   (cache condicional)
  Idempotency-Key: <uuid>   (idempotência em POST)

Response Headers:
  Content-Type: application/json; charset=UTF-8
  Location: /api/pedidos/123   (após 201)
  ETag: "abc123"
  Cache-Control: no-cache, no-store
  X-Request-ID: <uuid>  (echoback)
```

---

## 🔵 Bloco 2 — Spring Core (IoC e DI)

### IoC Container

- `ApplicationContext` — o container Spring
- Bean lifecycle: instanciação → injeção → `@PostConstruct` → uso → `@PreDestroy`
- Escopos: `singleton` (padrão), `prototype`, `request`, `session`

### Injeção de Dependência

```java
// 1. Constructor Injection — PREFERIDA (imutável, testável)
@Service
public class PedidoService {
    private final PedidoRepository repository;
    private final NotificationService notificationService;

    public PedidoService(PedidoRepository repository,
                         NotificationService notificationService) {
        this.repository = repository;
        this.notificationService = notificationService;
    }
}

// 2. Field Injection — EVITAR (dificulta testes, oculta dependências)
@Autowired  // ❌ não faça isso
private PedidoRepository repository;

// 3. Setter Injection — para dependências opcionais
@Autowired(required = false)
public void setMetricsCollector(MetricsCollector collector) { ... }
```

### Configuração e Profiles

```java
@Configuration
public class AppConfig {
    @Bean
    @ConditionalOnProperty(name = "feature.notifications", havingValue = "true")
    public NotificationService notificationService() {
        return new EmailNotificationService();
    }
}

// application.yml
spring:
  profiles:
    active: local  # local, dev, staging, prod

// application-prod.yml
spring:
  datasource:
    url: ${DATABASE_URL}  # variável de ambiente
```

---

## 🔵 Bloco 3 — Spring MVC e REST Controllers

### Estrutura de um Controller

```java
@RestController
@RequestMapping("/api/v1/pedidos")
@Tag(name = "Pedidos", description = "Gerenciamento de pedidos")  // OpenAPI
public class PedidoController {

    private final PedidoService pedidoService;

    public PedidoController(PedidoService pedidoService) {
        this.pedidoService = pedidoService;
    }

    @GetMapping
    public ResponseEntity<Page<PedidoResumoDTO>> listar(
            @RequestParam(required = false) StatusPedido status,
            @PageableDefault(size = 20, sort = "criadoEm", direction = DESC)
            Pageable pageable) {
        return ResponseEntity.ok(pedidoService.listar(status, pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<PedidoDetalheDTO> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(pedidoService.buscarPorId(id));
    }

    @PostMapping
    public ResponseEntity<PedidoCriadoDTO> criar(
            @RequestBody @Valid PedidoCriacaoRequest request,
            UriComponentsBuilder uriBuilder) {
        var pedido = pedidoService.criar(request);
        var location = uriBuilder.path("/api/v1/pedidos/{id}")
                                 .buildAndExpand(pedido.id()).toUri();
        return ResponseEntity.created(location).body(pedido);
    }

    @PatchMapping("/{id}/status")
    public ResponseEntity<Void> atualizarStatus(
            @PathVariable Long id,
            @RequestBody @Valid AtualizarStatusRequest request) {
        pedidoService.atualizarStatus(id, request.status());
        return ResponseEntity.noContent().build();
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancelar(@PathVariable Long id) {
        pedidoService.cancelar(id);
        return ResponseEntity.noContent().build();
    }
}
```

### DTOs e Mapeamento

```java
// Request DTO (entrada)
public record PedidoCriacaoRequest(
    @NotNull Long clienteId,
    @NotEmpty List<@Valid ItemRequest> itens,
    String observacao
) {}

// Response DTO (saída)
public record PedidoResumoDTO(Long id, String numero, StatusPedido status,
                               BigDecimal total, LocalDateTime criadoEm) {}

// Mapper com MapStruct (gera código em compile time — muito eficiente)
@Mapper(componentModel = "spring")
public interface PedidoMapper {
    PedidoResumoDTO toResumo(Pedido pedido);
    List<PedidoResumoDTO> toResumoList(List<Pedido> pedidos);
}
```

### Bean Validation

```java
public record ClienteCriacaoRequest(
    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 100)
    String nome,

    @NotBlank
    @Email(message = "E-mail inválido")
    String email,

    @NotBlank
    @Pattern(regexp = "\\d{11}", message = "CPF deve ter 11 dígitos")
    String cpf,

    @NotNull
    @Past(message = "Data de nascimento deve ser no passado")
    LocalDate dataNascimento,

    @Valid  // valida o objeto aninhado
    @NotNull
    EnderecoRequest endereco
) {}
```

---

## 🔵 Bloco 4 — Tratamento de Erros Global

> [!tip] Padrão Problem Details (RFC 9457)
> Resposta padronizada de erro — adotada por Spring 6+ com `ProblemDetail`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        pd.setTitle("Recurso não encontrado");
        pd.setType(URI.create("https://api.empresa.com/errors/not-found"));
        return pd;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        var pd = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        pd.setTitle("Erro de validação");
        var errors = ex.getBindingResult().getFieldErrors().stream()
            .collect(toMap(FieldError::getField, FieldError::getDefaultMessage));
        pd.setProperty("errors", errors);
        return pd;
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    public ProblemDetail handleDataIntegrity(DataIntegrityViolationException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT,
                                                   "Violação de integridade de dados");
        pd.setTitle("Conflito");
        return pd;
    }
}
```

---

## 🔵 Bloco 5 — Documentação com OpenAPI/Swagger

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.x</version>
</dependency>
```

```java
@OpenAPIDefinition(
    info = @Info(title = "API de Pedidos", version = "v1",
                 description = "Documentação da API de Pedidos"),
    servers = @Server(url = "https://api.empresa.com")
)
@Configuration
public class OpenApiConfig {}
```

Acesse: `http://localhost:8080/swagger-ui.html`

---

## 🔵 Bloco 6 — Configuração e Externalização

```yaml
# application.yml — estrutura completa
spring:
  application:
    name: pedidos-service
  datasource:
    url: jdbc:postgresql://localhost:5432/pedidos
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate  # NUNCA create/update em prod
    open-in-view: false   # SEMPRE false

server:
  port: 8080
  compression:
    enabled: true
  error:
    include-message: never  # segurança: não expor detalhes de erro

management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when-authorized

app:
  pedidos:
    max-itens: 50
    prazo-entrega-dias: 7
```

```java
@ConfigurationProperties(prefix = "app.pedidos")
@Validated
public record PedidosConfig(
    @Min(1) @Max(100) int maxItens,
    @Min(1) int prazoEntregaDias
) {}
```

---

## 📖 Recursos

| Recurso | Nota |
|---------|------|
| **Documentação Spring Boot oficial** | Sempre atualizada |
| **"Spring Boot in Action" — Craig Walls** | Introdução sólida |
| **Spring Guides (spring.io/guides)** | Tutoriais práticos curtos |
| Amigoscode Spring Boot (YouTube) | 🆓 Excelente didática |
| Daily Code Buffer (YouTube) | 🆓 Projetos completos |

---

## 🧪 Projeto Final do Ano 1

> [!example] Projeto: API REST Completa de E-commerce
> - **Domínio:** Produtos, Categorias, Clientes, Pedidos, Pagamentos
> - **Stack:** Spring Boot 3.x + PostgreSQL + Flyway + MapStruct
> - **APIs:** CRUD completo com paginação, filtros, ordenação
> - **Validação:** Bean Validation em todos os endpoints
> - **Erros:** GlobalExceptionHandler com ProblemDetail
> - **Docs:** Swagger/OpenAPI completo
> - **Config:** Profiles (local/dev/prod), ConfigurationProperties
> - **Deploy:** Jar executável no Railway ou Render (grátis)
> - **CI:** GitHub Actions com build e testes automáticos
> 
> **Este projeto deve estar no seu GitHub. É o que você mostra em entrevistas.**

---

## 🔗 Navegação

← [[T3 — SQL, JDBC e JPA-Hibernate]]  
→ [[Ano 2 — Produção e Qualidade]]
