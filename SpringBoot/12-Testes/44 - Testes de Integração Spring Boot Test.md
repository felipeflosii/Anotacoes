# 44 — Testes de Integração com Spring Boot Test

tags: #springboot #testes #integração #testcontainers
links: [[43 - Testes Unitários com JUnit e Mockito]] | [[45 - Swagger e OpenAPI]] | [[🗺️ Mapa Principal]]

---

## Teste de integração vs unitário

| | Unitário | Integração |
|---|---|---|
| **Escopo** | Classe isolada | Múltiplas camadas juntas |
| **Banco** | Mock | Real (H2 ou Testcontainers) |
| **Velocidade** | Rápido (ms) | Lento (segundos) |
| **Confiança** | Lógica isolada | Fluxo completo |
| **Quando usar** | Services, regras de negócio | API end-to-end |

---

## @SpringBootTest — sobe o contexto completo

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")  // usa application-test.yml com H2
@Transactional           // rollback após cada teste — banco limpo
class ClienteIntegracaoTest {

    @Autowired
    private TestRestTemplate restTemplate;  // cliente HTTP para testes

    @Autowired
    private ClienteRepository repository;

    @Test
    @DisplayName("Fluxo completo: criar → buscar → atualizar → deletar")
    void fluxoCompletoCrud() {
        // === CRIAR ===
        var request = new ClienteRequest("Felipe", "felipe@ex.com", "11999887766");

        ResponseEntity<ClienteResponse> criarResponse = restTemplate.postForEntity(
            "/api/v1/clientes",
            request,
            ClienteResponse.class
        );

        assertThat(criarResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(criarResponse.getHeaders().getLocation()).isNotNull();
        Long id = criarResponse.getBody().id();

        // === BUSCAR ===
        ResponseEntity<ClienteResponse> buscarResponse = restTemplate.getForEntity(
            "/api/v1/clientes/" + id,
            ClienteResponse.class
        );

        assertThat(buscarResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(buscarResponse.getBody().nome()).isEqualTo("Felipe");

        // === DELETAR ===
        restTemplate.delete("/api/v1/clientes/" + id);
        assertThat(repository.existsById(id)).isFalse();
    }
}
```

---

## @DataJpaTest — testa só a camada de persistência

```java
@DataJpaTest           // sobe só JPA + H2 — sem controllers, sem services
@ActiveProfiles("test")
class ClienteRepositoryTest {

    @Autowired
    private ClienteRepository repository;

    @Autowired
    private TestEntityManager entityManager;  // utilitário para setup dos testes

    @Test
    @DisplayName("findByEmail deve retornar cliente quando existe")
    void findByEmailDeveRetornarCliente() {
        // Arrange — persiste no banco de teste
        var cliente = new Cliente("Felipe", "felipe@ex.com", "11999887766");
        entityManager.persistAndFlush(cliente);

        // Act
        Optional<Cliente> resultado = repository.findByEmail("felipe@ex.com");

        // Assert
        assertThat(resultado).isPresent();
        assertThat(resultado.get().getNome()).isEqualTo("Felipe");
    }

    @Test
    @DisplayName("existsByEmail deve retornar false quando não existe")
    void existsByEmailDeveRetornarFalse() {
        assertThat(repository.existsByEmail("naoexiste@ex.com")).isFalse();
    }

    @Test
    @DisplayName("findByNomeContainingIgnoreCase deve buscar parcial case-insensitive")
    void deveBuscarPorNomeParcial() {
        entityManager.persist(new Cliente("Felipe Santos", "felipe@ex.com", "11999887766"));
        entityManager.persist(new Cliente("Maria Silva", "maria@ex.com", "11988776655"));
        entityManager.flush();

        List<Cliente> resultado = repository.findByNomeContainingIgnoreCase("feli");

        assertThat(resultado).hasSize(1);
        assertThat(resultado.get(0).getNome()).isEqualTo("Felipe Santos");
    }
}
```

---

## Testcontainers — banco real nos testes

Para testes de integração com PostgreSQL real (não H2):

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@Testcontainers  // habilita Testcontainers
class PedidoIntegracaoTest {

    @Container  // sobe um container Docker PostgreSQL
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource  // injeta as propriedades do container no Spring
    static void configureDataSource(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void deveExecutarComPostgresReal() {
        // Testa com PostgreSQL de verdade — detecta incompatibilidades com H2
    }
}
```

---

## Próximas notas
- [[45 - Swagger e OpenAPI]] — documentando a API
