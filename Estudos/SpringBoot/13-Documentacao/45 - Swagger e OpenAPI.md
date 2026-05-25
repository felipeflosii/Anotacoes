# 45 — Swagger / OpenAPI

tags: #springboot #swagger #openapi #documentação
links: [[43 - Testes Unitários com JUnit e Mockito]] | [[46 - Lazy vs Eager Loading]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O que é e por que usar

**OpenAPI** é a especificação para descrever APIs REST. **Swagger UI** é a interface visual gerada a partir dessa especificação.

Com Springdoc OpenAPI, sua documentação é **gerada automaticamente** a partir dos controllers e DTOs.

---

## Configuração

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```yaml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method
    tagsSorter: alpha
    try-it-out-enabled: true   # habilita "Try it out" na UI
```

```java
// Informações gerais da API
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("API Concessionária")
                .description("API REST para gestão de concessionária de veículos")
                .version("v1.0.0")
                .contact(new Contact()
                    .name("Felipe Santos")
                    .email("felipe@empresa.com"))
                .license(new License().name("MIT")))
            .addSecurityItem(new SecurityRequirement().addList("JWT"))
            .components(new Components()
                .addSecuritySchemes("JWT", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")
                    .description("Informe o token JWT no formato: Bearer {token}")));
    }
}
```

---

## Documentando Controllers e DTOs

```java
@RestController
@RequestMapping("/api/v1/clientes")
@Tag(name = "Clientes", description = "Operações de gestão de clientes")
public class ClienteController {

    @Operation(
        summary = "Criar cliente",
        description = "Cria um novo cliente no sistema. O e-mail deve ser único.",
        responses = {
            @ApiResponse(responseCode = "201", description = "Cliente criado com sucesso",
                content = @Content(schema = @Schema(implementation = ClienteResponse.class))),
            @ApiResponse(responseCode = "400", description = "Dados inválidos",
                content = @Content(schema = @Schema(implementation = ErrorResponse.class))),
            @ApiResponse(responseCode = "409", description = "E-mail já cadastrado")
        }
    )
    @PostMapping
    public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest request) { ... }

    @Operation(summary = "Buscar cliente por ID", security = @SecurityRequirement(name = "JWT"))
    @Parameter(name = "id", description = "ID do cliente", example = "42")
    @GetMapping("/{id}")
    public ResponseEntity<ClienteResponse> buscar(@PathVariable Long id) { ... }
}
```

```java
// Documentando DTOs
@Schema(description = "Dados para criação de cliente")
public record ClienteRequest(

    @Schema(description = "Nome completo", example = "Felipe Santos", minLength = 2, maxLength = 150)
    @NotBlank String nome,

    @Schema(description = "E-mail único do cliente", example = "felipe@empresa.com")
    @NotBlank @Email String email,

    @Schema(description = "Telefone (somente dígitos)", example = "11999887766")
    @NotBlank @Pattern(regexp = "\\d{10,11}") String telefone
) {}
```

Acesse em: `http://localhost:8080/swagger-ui.html`

---

## Liberando Swagger no Spring Security

```java
// No SecurityConfig — liberar as URLs do Swagger
.authorizeHttpRequests(auth -> auth
    .requestMatchers(
        "/swagger-ui/**",
        "/swagger-ui.html",
        "/api-docs/**",
        "/api-docs.yaml"
    ).permitAll()
    // ...
)
```

---

## Próximas notas
- [[46 - Lazy vs Eager Loading]] — performance com JPA
- [[49 - Projeto Concessionária - Visão Geral]] — projeto completo
