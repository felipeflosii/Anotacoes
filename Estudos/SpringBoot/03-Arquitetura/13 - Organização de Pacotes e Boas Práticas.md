# 13 — Organização de Pacotes e Boas Práticas

tags: #springboot #arquitetura #organização #boaspraticas
links: [[11 - Clean Architecture no Spring Boot]] | [[12 - Controller Service Repository Domain DTO]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## As duas escolas de organização

### Por camada (Layer-first)

```
com.empresa.api/
├── controller/
│   ├── ClienteController.java
│   ├── PedidoController.java
│   └── ProdutoController.java
├── service/
│   ├── ClienteService.java
│   ├── PedidoService.java
│   └── ProdutoService.java
├── repository/
│   ├── ClienteRepository.java
│   ├── PedidoRepository.java
│   └── ProdutoRepository.java
└── model/
    ├── Cliente.java
    ├── Pedido.java
    └── Produto.java
```

**Prós:** simples de entender no início
**Contras:** ao crescer, cada pasta fica com dezenas de arquivos. Para trabalhar em "Cliente", você pula entre 4 pastas diferentes.

### Por domínio/módulo (Domain-first) ✅ Recomendada

```
com.empresa.api/
├── cliente/
│   ├── Cliente.java                  ← entidade
│   ├── ClienteRepository.java        ← repositório
│   ├── ClienteService.java           ← serviço
│   ├── ClienteController.java        ← controller
│   └── dto/
│       ├── ClienteRequest.java
│       └── ClienteResponse.java
├── pedido/
│   ├── Pedido.java
│   ├── PedidoRepository.java
│   ├── PedidoService.java
│   ├── PedidoController.java
│   └── dto/
│       ├── PedidoRequest.java
│       └── PedidoResponse.java
└── config/
    ├── SecurityConfig.java
    └── SwaggerConfig.java
```

**Prós:** coesão — tudo de Cliente fica junto. Fácil de encontrar, fácil de deletar um módulo inteiro se necessário.
**Contras:** nenhum significativo para projetos Spring Boot.

---

## Estrutura completa recomendada para projetos reais

```
src/
└── main/
    ├── java/
    │   └── com/empresa/api/
    │       │
    │       ├── MinhaApiApplication.java         ← ponto de entrada
    │       │
    │       ├── config/                          ← configurações globais Spring
    │       │   ├── SecurityConfig.java
    │       │   ├── SwaggerConfig.java
    │       │   ├── CorsConfig.java
    │       │   └── JacksonConfig.java
    │       │
    │       ├── exception/                       ← tratamento global de exceções
    │       │   ├── GlobalExceptionHandler.java
    │       │   ├── RecursoNaoEncontradoException.java
    │       │   ├── EmailJaCadastradoException.java
    │       │   └── ErrorResponse.java
    │       │
    │       ├── security/                        ← infraestrutura de segurança
    │       │   ├── JwtService.java
    │       │   ├── JwtAuthFilter.java
    │       │   └── UserDetailsServiceImpl.java
    │       │
    │       ├── auth/                            ← módulo de autenticação
    │       │   ├── AuthController.java
    │       │   ├── AuthService.java
    │       │   └── dto/
    │       │       ├── LoginRequest.java
    │       │       ├── RegisterRequest.java
    │       │       └── AuthResponse.java
    │       │
    │       ├── usuario/                         ← módulo de usuários
    │       │   ├── Usuario.java
    │       │   ├── UsuarioRepository.java
    │       │   ├── UsuarioService.java
    │       │   ├── UsuarioController.java
    │       │   └── dto/
    │       │       ├── UsuarioRequest.java
    │       │       └── UsuarioResponse.java
    │       │
    │       ├── cliente/                         ← módulo de clientes
    │       │   └── (mesma estrutura acima)
    │       │
    │       └── pedido/                          ← módulo de pedidos
    │           ├── Pedido.java
    │           ├── ItemPedido.java              ← entidade relacionada
    │           ├── PedidoRepository.java
    │           ├── PedidoService.java
    │           ├── PedidoController.java
    │           └── dto/
    │               ├── PedidoRequest.java
    │               ├── PedidoResponse.java
    │               └── ItemPedidoDto.java
    │
    └── resources/
        ├── application.yml
        ├── application-dev.yml
        ├── application-prod.yml
        └── db/
            └── migration/                       ← Flyway migrations
                ├── V1__create_tables.sql
                ├── V2__add_index_email.sql
                └── V3__add_coluna_telefone.sql
```

---

## Boas práticas de nomenclatura

### Classes

```java
// Entidades — substantivo singular
Cliente.java, Pedido.java, Produto.java

// Repositories — entidade + Repository
ClienteRepository.java

// Services — entidade + Service
ClienteService.java

// Controllers — entidade + Controller
ClienteController.java

// DTOs — entidade + contexto + sufixo
ClienteRequest.java          // dados de entrada
ClienteResponse.java         // dados de saída
ClienteUpdateRequest.java    // entrada para atualização
ClienteResumoResponse.java   // saída resumida (para listas)

// Exceptions — descrição clara do problema
ClienteNaoEncontradoException.java
EmailJaCadastradoException.java
SaldoInsuficienteException.java

// Configs — componente + Config
SecurityConfig.java
SwaggerConfig.java

// Handlers — o que faz + Handler
GlobalExceptionHandler.java
JwtAuthFilter.java  (Filters → sufixo Filter)
```

### Métodos nos Services

```java
// Verbos claros e consistentes:
criar()           // POST — cria novo recurso
buscarPorId()     // GET /{id}
listar()          // GET — lista com paginação
buscarPorEmail()  // GET com filtro específico
atualizar()       // PUT /{id}
atualizarParcial() // PATCH /{id}
deletar()         // DELETE /{id}
desativar()       // soft delete (lógico)
```

### Endpoints REST

```
GET    /api/v1/clientes               → listar (paginado)
GET    /api/v1/clientes/{id}          → buscar por id
POST   /api/v1/clientes               → criar
PUT    /api/v1/clientes/{id}          → atualizar completo
PATCH  /api/v1/clientes/{id}          → atualizar parcial
DELETE /api/v1/clientes/{id}          → deletar

GET    /api/v1/clientes/{id}/pedidos  → pedidos de um cliente
POST   /api/v1/auth/login             → autenticar
POST   /api/v1/auth/register          → registrar
```

---

## Boas práticas gerais

### 1. Controllers finos, Services gordos

```java
// ✅ Controller com responsabilidade única
@PostMapping
public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest req) {
    return ResponseEntity.status(201).body(service.criar(req));
}

// ❌ Controller com lógica de negócio
@PostMapping
public ResponseEntity<?> criar(@RequestBody ClienteRequest req) {
    if (repository.existsByEmail(req.getEmail())) { // regra de negócio no controller
        return ResponseEntity.status(409).body("E-mail já existe");
    }
    // ...
}
```

### 2. Services sem referências a HTTP

```java
// ✅ Service puro — sem HttpServletRequest, ResponseEntity, etc.
@Service
public class ClienteService {
    public ClienteResponse criar(ClienteRequest request) { ... }
}

// ❌ Service com referência a HTTP
@Service
public class ClienteService {
    public ResponseEntity<ClienteResponse> criar(HttpServletRequest httpReq) { ... }
}
```

### 3. Entidades não saem do backend diretamente

```java
// ❌ Retornando entidade JPA diretamente
@GetMapping("/{id}")
public Cliente buscar(@PathVariable Long id) {
    return repository.findById(id).orElseThrow();
    // Expõe todos os campos, incluindo senha, campos internos
    // Pode causar LazyInitializationException
    // Cria acoplamento entre API e banco
}

// ✅ Sempre converter para DTO
@GetMapping("/{id}")
public ResponseEntity<ClienteResponse> buscar(@PathVariable Long id) {
    return ResponseEntity.ok(service.buscarPorId(id));
    // Só expõe o que o cliente precisa ver
}
```

### 4. `@Transactional` só no Service

```java
// ✅ Correto — transação gerenciada no Service
@Service
public class ClienteService {
    @Transactional
    public ClienteResponse criar(ClienteRequest request) { ... }
}

// ❌ Transação no Controller (errado)
@RestController
public class ClienteController {
    @Transactional  // não faça isso
    @PostMapping
    public ResponseEntity<?> criar(...) { ... }
}
```

### 5. Use `final` e injeção por construtor

```java
// ✅ final garante que a dependência não muda após construção
@Service
public class ClienteService {
    private final ClienteRepository repository;  // final

    public ClienteService(ClienteRepository repository) {  // construtor
        this.repository = repository;
    }
}
```

---

## Próximas notas
- [[14 - RestController e RequestMapping]] — começando os controllers
- [[18 - O que são DTOs e Por que Usar]] — DTOs em detalhe
