# 43 — Testes Unitários com JUnit e Mockito

tags: #springboot #testes #junit #mockito
links: [[44 - Testes de Integração Spring Boot Test]] | [[🗺️ Mapa Principal]]

---

## Por que testar

```
Sem testes:
→ Você testa manualmente no Postman após cada mudança
→ Refactoring é arriscado — pode quebrar sem perceber
→ Bugs chegam em produção
→ Time tem medo de mudar código legado

Com testes:
→ Confiança para refatorar
→ Documentação viva do comportamento esperado
→ Bugs detectados antes do commit
→ Deploy com segurança
```

---

## Dependências (já inclusas no spring-boot-starter-test)

```xml
<!-- Incluído automaticamente com spring-boot-starter-test: -->
<!-- JUnit 5 (Jupiter) — framework de testes -->
<!-- Mockito — mocking de dependências -->
<!-- AssertJ — assertions fluentes -->
<!-- Hamcrest — matchers -->
<!-- Spring Test — utilitários de teste Spring -->
```

---

## Estrutura de um teste unitário

```java
// Padrão AAA: Arrange (preparar), Act (agir), Assert (verificar)

@ExtendWith(MockitoExtension.class)  // habilita Mockito sem subir o Spring
class ClienteServiceTest {

    @Mock                                // mock — substitui a dependência real
    private ClienteRepository repository;

    @Mock
    private EmailService emailService;

    @InjectMocks                         // injeta os mocks acima no service
    private ClienteService service;

    @Test
    @DisplayName("Deve criar cliente com sucesso quando dados são válidos")
    void deveCriarClienteComSucesso() {
        // === ARRANGE — preparar dados e comportamento dos mocks ===
        var request = new ClienteRequest("Felipe", "felipe@ex.com", "11999887766");
        var clienteSalvo = new Cliente("Felipe", "felipe@ex.com", "11999887766");
        // Simula ID gerado pelo banco:
        ReflectionTestUtils.setField(clienteSalvo, "id", 1L);

        when(repository.existsByEmail("felipe@ex.com")).thenReturn(false);
        when(repository.save(any(Cliente.class))).thenReturn(clienteSalvo);

        // === ACT — executar a ação a testar ===
        ClienteResponse response = service.criar(request);

        // === ASSERT — verificar o resultado ===
        assertThat(response).isNotNull();
        assertThat(response.id()).isEqualTo(1L);
        assertThat(response.nome()).isEqualTo("Felipe");
        assertThat(response.email()).isEqualTo("felipe@ex.com");

        // Verificar interações com os mocks
        verify(repository).existsByEmail("felipe@ex.com");
        verify(repository).save(any(Cliente.class));
        verify(emailService).enviarBoasVindas("felipe@ex.com", "Felipe");
    }

    @Test
    @DisplayName("Deve lançar exceção quando e-mail já está cadastrado")
    void deveLancarExcecaoQuandoEmailJaCadastrado() {
        // Arrange
        var request = new ClienteRequest("Felipe", "felipe@ex.com", "11999887766");
        when(repository.existsByEmail("felipe@ex.com")).thenReturn(true);

        // Act + Assert
        assertThatThrownBy(() -> service.criar(request))
            .isInstanceOf(RecursoJaExisteException.class)
            .hasMessageContaining("felipe@ex.com");

        // Garantir que o banco não foi chamado para salvar
        verify(repository, never()).save(any());
        verify(emailService, never()).enviarBoasVindas(any(), any());
    }
}
```

---

## Mockito — principais métodos

```java
// === CONFIGURAR COMPORTAMENTO (when/thenReturn) ===

// Retornar valor
when(repository.findById(1L)).thenReturn(Optional.of(cliente));
when(repository.existsByEmail(anyString())).thenReturn(false);
when(service.calcular(any(BigDecimal.class))).thenReturn(BigDecimal.TEN);

// Lançar exceção
when(repository.findById(999L))
    .thenThrow(new RecursoNaoEncontradoException("Cliente", 999L));

// Retornar valores diferentes em chamadas sucessivas
when(repository.count())
    .thenReturn(0L)      // 1ª chamada
    .thenReturn(1L);     // 2ª chamada

// Método void — usar doNothing ou doThrow
doNothing().when(emailService).enviar(anyString(), anyString());
doThrow(new RuntimeException("SMTP falhou"))
    .when(emailService).enviar(anyString(), anyString());

// === VERIFICAR INTERAÇÕES (verify) ===

verify(repository).save(any(Cliente.class));           // chamado 1x
verify(repository, times(2)).findById(anyLong());      // chamado 2x
verify(repository, never()).deleteById(anyLong());     // nunca chamado
verify(repository, atLeastOnce()).findAll();            // pelo menos 1x
verify(repository, atMost(3)).findById(anyLong());     // no máximo 3x

// Capturar argumento passado ao mock
ArgumentCaptor<Cliente> captor = ArgumentCaptor.forClass(Cliente.class);
verify(repository).save(captor.capture());
Cliente clienteSalvo = captor.getValue();
assertThat(clienteSalvo.getEmail()).isEqualTo("felipe@ex.com");

// === MATCHERS ===
any()                          // qualquer objeto
any(Cliente.class)             // qualquer Cliente
anyString()                    // qualquer String
anyLong()                      // qualquer Long
eq("valor exato")              // valor específico
isNull()                       // null
isNotNull()                    // não null
argThat(c -> c.isAtivo())      // condição customizada
```

---

## AssertJ — assertions fluentes

```java
// Objetos
assertThat(response).isNotNull();
assertThat(response).isNull();
assertThat(response).isInstanceOf(ClienteResponse.class);
assertThat(response).isEqualTo(esperado);

// Strings
assertThat(nome).isEqualTo("Felipe");
assertThat(nome).isNotBlank();
assertThat(email).contains("@");
assertThat(email).startsWith("felipe");
assertThat(mensagem).containsIgnoringCase("não encontrado");

// Números
assertThat(preco).isGreaterThan(BigDecimal.ZERO);
assertThat(preco).isBetween(new BigDecimal("10"), new BigDecimal("100"));
assertThat(count).isEqualTo(5);

// Coleções
assertThat(lista).isNotEmpty();
assertThat(lista).hasSize(3);
assertThat(lista).contains(item1, item2);
assertThat(lista).doesNotContain(itemIndesejado);
assertThat(lista).allMatch(item -> item.isAtivo());
assertThat(lista).extracting("nome").containsExactly("A", "B", "C");

// Exceções
assertThatThrownBy(() -> service.criar(requestInvalido))
    .isInstanceOf(RecursoJaExisteException.class)
    .hasMessageContaining("já existe");

assertThatCode(() -> service.criar(requestValido))
    .doesNotThrowAnyException();
```

---

## Testando o Controller com MockMvc

```java
@WebMvcTest(ClienteController.class)  // sobe só a camada web — sem banco, sem service real
class ClienteControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean                          // mock do Spring — substitui o bean real
    private ClienteService service;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("POST /clientes deve retornar 201 quando dados válidos")
    void deveCriarCliente() throws Exception {
        var request = new ClienteRequest("Felipe", "felipe@ex.com", "11999887766");
        var response = new ClienteResponse(1L, "Felipe", "felipe@ex.com", "11999887766", true, LocalDateTime.now());

        when(service.criar(any(ClienteRequest.class))).thenReturn(response);

        mockMvc.perform(
            post("/api/v1/clientes")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request))
        )
        .andExpect(status().isCreated())
        .andExpect(header().exists("Location"))
        .andExpect(jsonPath("$.id").value(1))
        .andExpect(jsonPath("$.nome").value("Felipe"))
        .andExpect(jsonPath("$.email").value("felipe@ex.com"));
    }

    @Test
    @DisplayName("POST /clientes deve retornar 400 quando e-mail inválido")
    void deveRetornar400QuandoEmailInvalido() throws Exception {
        var request = new ClienteRequest("Felipe", "email-invalido", "11999887766");

        mockMvc.perform(
            post("/api/v1/clientes")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request))
        )
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.erros[0].campo").value("email"));
    }

    @Test
    @DisplayName("GET /clientes/999 deve retornar 404 quando não existe")
    void deveRetornar404QuandoNaoEncontrado() throws Exception {
        when(service.buscarPorId(999L))
            .thenThrow(new RecursoNaoEncontradoException("Cliente", 999L));

        mockMvc.perform(get("/api/v1/clientes/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.status").value(404));
    }

    @Test
    @WithMockUser(roles = "ADMIN")   // simula usuário autenticado como ADMIN
    @DisplayName("DELETE /clientes/1 deve retornar 204 para ADMIN")
    void deveDeletarComoAdmin() throws Exception {
        doNothing().when(service).deletar(1L);

        mockMvc.perform(delete("/api/v1/clientes/1"))
            .andExpect(status().isNoContent());
    }
}
```

---

## Próximas notas
- [[44 - Testes de Integração Spring Boot Test]] — testes com banco real
