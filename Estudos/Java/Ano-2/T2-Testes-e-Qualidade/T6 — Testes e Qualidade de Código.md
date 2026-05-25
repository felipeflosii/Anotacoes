---
tags: [java, ano-2, testes, junit, mockito, testcontainers, tdd, contract-testing]
trimestre: T6
meses: 19-21
---

# T6 · Testes e Qualidade de Código
### Meses 19–21 · Ano 2

> **Objetivo:** Escrever testes que realmente protegem o sistema. Unit, Integration, Contract e E2E. Entender o que testar e por quê — não apenas buscar % de cobertura.

---

## 🔵 Bloco 1 — Filosofia de Testes

> [!tip] A pirâmide de testes
> ```
>        /E2E\          ← poucos, lentos, caros
>       /------\
>      / Integ. \       ← médio volume
>     /----------\
>    /    Unit    \     ← muitos, rápidos, baratos
>   /--------------\
> ```
> **Regra prática:** 70% unit / 20% integration / 10% E2E

### O que testar de verdade

- Lógica de negócio (regras, cálculos, validações)
- Casos de borda (null, vazio, limite mínimo/máximo)
- Casos de erro (exceções esperadas)
- Contratos de API (não mudar sem quebrar clientes)
- **NÃO testar:** getters/setters, frameworks, configurações triviais

### TDD (Test-Driven Development)

- **Red** → **Green** → **Refactor**
- Não é sobre 100% de cobertura; é sobre design
- TDD força pensar na interface antes da implementação
- Use em código novo com lógica complexa

---

## 🔵 Bloco 2 — JUnit 5 + Mockito

### JUnit 5 — o essencial

```java
@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {

    @Mock private PedidoRepository pedidoRepository;
    @Mock private EstoqueService estoqueService;
    @InjectMocks private PedidoService pedidoService;

    @Test
    @DisplayName("Deve criar pedido quando estoque disponível")
    void deveCriarPedido_quandoEstoqueDisponivel() {
        // Given (Arrange)
        var request = new PedidoCriacaoRequest(1L, List.of(new ItemRequest(10L, 2)), null);
        when(estoqueService.verificar(10L, 2)).thenReturn(true);
        when(pedidoRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));

        // When (Act)
        var resultado = pedidoService.criar(request);

        // Then (Assert)
        assertThat(resultado).isNotNull();
        assertThat(resultado.status()).isEqualTo(StatusPedido.CRIADO);
        verify(pedidoRepository).save(any(Pedido.class));
        verify(estoqueService).reservar(10L, 2);
    }

    @Test
    @DisplayName("Deve lançar exceção quando produto sem estoque")
    void deveLancarExcecao_quandoSemEstoque() {
        var request = new PedidoCriacaoRequest(1L, List.of(new ItemRequest(10L, 99)), null);
        when(estoqueService.verificar(10L, 99)).thenReturn(false);

        assertThatThrownBy(() -> pedidoService.criar(request))
            .isInstanceOf(EstoqueInsuficienteException.class)
            .hasMessageContaining("produto 10");

        verifyNoInteractions(pedidoRepository);
    }

    @ParameterizedTest
    @MethodSource("descontosProvider")
    @DisplayName("Deve calcular desconto corretamente")
    void deveCalcularDesconto(BigDecimal total, int quantidadeItens, BigDecimal expectedDesconto) {
        var desconto = pedidoService.calcularDesconto(total, quantidadeItens);
        assertThat(desconto).isEqualByComparingTo(expectedDesconto);
    }

    static Stream<Arguments> descontosProvider() {
        return Stream.of(
            Arguments.of(new BigDecimal("100"), 1, BigDecimal.ZERO),
            Arguments.of(new BigDecimal("500"), 5, new BigDecimal("25")),
            Arguments.of(new BigDecimal("1000"), 10, new BigDecimal("100"))
        );
    }
}
```

### Mockito — técnicas avançadas

```java
// Capturar argumentos
var captor = ArgumentCaptor.forClass(Pedido.class);
verify(repository).save(captor.capture());
assertThat(captor.getValue().getStatus()).isEqualTo(PAGO);

// Spy (objeto real com mocks parciais)
var realService = spy(new EmailService());
doNothing().when(realService).enviarEmail(anyString());

// Verificar ordem de chamadas
var inOrder = inOrder(estoqueService, pedidoRepository);
inOrder.verify(estoqueService).reservar(any(), anyInt());
inOrder.verify(pedidoRepository).save(any());

// Mock de método estático (use com moderação — indica design ruim)
try (var mocked = mockStatic(LocalDateTime.class)) {
    mocked.when(LocalDateTime::now).thenReturn(fixedDateTime);
    // ...
}
```

### AssertJ — asserções fluentes (prefira ao JUnit puro)

```java
// Coleções
assertThat(pedidos)
    .hasSize(3)
    .extracting(Pedido::getStatus)
    .containsExactlyInAnyOrder(CRIADO, PAGO, ENTREGUE);

// Exceções
assertThatThrownBy(() -> service.buscar(-1L))
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("ID inválido");

// Objetos complexos
assertThat(pedido)
    .satisfies(p -> {
        assertThat(p.getNumero()).startsWith("PED-");
        assertThat(p.getTotal()).isPositive();
        assertThat(p.getItens()).isNotEmpty();
    });
```

---

## 🔵 Bloco 3 — Testes de Integração

### @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@ActiveProfiles("test")
class PedidoControllerIT {

    @Autowired private MockMvc mockMvc;
    @Autowired private ObjectMapper objectMapper;
    @Autowired private PedidoRepository pedidoRepository;

    @BeforeEach
    void setUp() { pedidoRepository.deleteAll(); }

    @Test
    void deveCriarPedido_retornarStatus201() throws Exception {
        var request = new PedidoCriacaoRequest(/* ... */);

        mockMvc.perform(post("/api/v1/pedidos")
                .contentType(APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request))
                .header("Authorization", "Bearer " + getTestToken()))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.numero").isNotEmpty())
            .andExpect(jsonPath("$.status").value("CRIADO"));
    }

    @Test
    void deveRetornar422_quandoPayloadInvalido() throws Exception {
        mockMvc.perform(post("/api/v1/pedidos")
                .contentType(APPLICATION_JSON)
                .content("{}"))
            .andExpect(status().isUnprocessableEntity())
            .andExpect(jsonPath("$.errors").isMap());
    }
}
```

### Testcontainers — banco de dados real em testes

```java
@SpringBootTest
@Testcontainers
class PedidoRepositoryIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")
        .withReuse(true);  // reutiliza container entre testes (performance)

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void devePersistirEBuscarPedido() {
        // testa com PostgreSQL real, não H2
    }
}
```

> [!tip] Testcontainers é o padrão nas big techs
> Testes com H2 podem passar e o código quebrar em produção por diferenças de SQL. Testcontainers garante paridade.

### @DataJpaTest + @WebMvcTest

```java
// Testa apenas a camada JPA (mais rápido)
@DataJpaTest
@AutoConfigureTestDatabase(replace = NONE)  // com Testcontainers
class PedidoRepositoryTest { ... }

// Testa apenas a camada Web (sem subir banco)
@WebMvcTest(PedidoController.class)
class PedidoControllerTest {
    @MockBean private PedidoService pedidoService;
    @Autowired private MockMvc mockMvc;
    // ...
}
```

---

## 🔵 Bloco 4 — Contract Testing com Pact

> [!note] Por que Contract Testing?
> Em microservices, como garantir que quando o serviço A chama o serviço B, os contratos (formatos de request/response) estão alinhados? Testes E2E são lentos e frágeis. Contract testing resolve isso.

### Pact — Consumer-Driven Contracts

```java
// Consumer (quem chama)
@ExtendWith(PactConsumerTestExt.class)
class PedidoServicePactTest {

    @Pact(consumer = "pedidos-service", provider = "clientes-service")
    public RequestResponsePact createPact(PactDslWithProvider builder) {
        return builder
            .given("cliente 1 existe")
            .uponReceiving("buscar cliente por id")
            .path("/api/clientes/1")
            .method("GET")
            .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .integerType("id", 1)
                .stringType("nome", "João Silva")
                .stringType("email", "joao@email.com"))
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "createPact")
    void deveRetornarCliente(MockServer mockServer) {
        var cliente = clienteServiceClient.buscarPorId(1L, mockServer.getUrl());
        assertThat(cliente.nome()).isEqualTo("João Silva");
    }
}
```

---

## 🔵 Bloco 5 — Qualidade de Código

### Ferramentas Obrigatórias

| Ferramenta | O que faz | Configuração |
|-----------|-----------|--------------|
| **SonarQube/SonarCloud** | Análise estática, bugs, vulnerabilidades, code smells | CI/CD pipeline |
| **Checkstyle** | Estilo de código (guia Google Java Style) | Maven plugin |
| **SpotBugs** | Análise estática de bugs (substitui FindBugs) | Maven plugin |
| **PMD** | Código duplicado, complexidade ciclomática | Maven plugin |
| **JaCoCo** | Cobertura de testes | Reporte no CI |
| **ArchUnit** | Testa regras arquiteturais | JUnit extension |

### ArchUnit — teste sua arquitetura

```java
@AnalyzeClasses(packages = "br.com.app")
class ArquiteturaTest {

    @ArchTest
    static final ArchRule servicesNaoDependemDeControllers =
        noClasses().that().resideInAPackage("..service..")
            .should().dependOnClassesThat()
            .resideInAPackage("..controller..");

    @ArchTest
    static final ArchRule repositoriesSaoInterfaces =
        classes().that().resideInAPackage("..repository..")
            .should().beInterfaces();

    @ArchTest
    static final ArchRule entidadesNaoSaoAcessadasDiretamentePorControllers =
        noClasses().that().resideInAPackage("..controller..")
            .should().dependOnClassesThat()
            .areAnnotatedWith(Entity.class);
}
```

### Mutation Testing com PIT

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <configuration>
        <mutationThreshold>80</mutationThreshold>
    </configuration>
</plugin>
```

> [!note] Mutation testing > cobertura
> PIT modifica seu código (cria mutantes) e verifica se seus testes detectam as mudanças. Cobertura de 80% com mutation score de 70% é melhor que cobertura de 95% fraca.

---

## 🔗 Navegação

← [[T5 — Spring Boot Avançado]]  
→ [[T7 — Mensageria e Cache]]
