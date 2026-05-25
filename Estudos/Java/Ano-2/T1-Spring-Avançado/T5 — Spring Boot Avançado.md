---
tags: [java, ano-2, spring-security, aop, async, reactive, jwt]
trimestre: T5
meses: 13-18
---

# T5 · Spring Boot Avançado
### Meses 13–18 · Ano 2

---

## 🔵 Bloco 1 — Spring Security 6

> [!warning] Spring Security mudou muito no 6.x
> `WebSecurityConfigurerAdapter` foi removido. A configuração agora é via `SecurityFilterChain` beans. Não aprenda pelo legado.

### Autenticação com JWT (o padrão do mercado)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
                                            JwtAuthFilter jwtAuthFilter) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/actuator/health").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/produtos/**").hasRole("USER")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
                .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
            )
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // work factor 12
    }
}
```

### JWT Filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {
        var authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }
        var token = authHeader.substring(7);
        var username = jwtService.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            var userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtService.isTokenValid(token, userDetails)) {
                var auth = new UsernamePasswordAuthenticationToken(
                    userDetails, null, userDetails.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(request, response);
    }
}
```

### OAuth2 e OpenID Connect

- **Resource Server** — validar tokens emitidos por Keycloak/Auth0/Okta
- **Keycloak** — identity provider open source; use em projetos enterprise
- **Spring Authorization Server** — emitir tokens OAuth2 (substituiu Spring Security OAuth)
- Fluxos: Authorization Code + PKCE (SPAs), Client Credentials (M2M), Device Flow

### Method Security

```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.name")
public UserDTO getUser(String userId) { ... }

@PostAuthorize("returnObject.ownerId == authentication.name")
public Document getDocument(Long id) { ... }

@PreFilter("filterObject.active == true")
public void processItems(List<Item> items) { ... }
```

---

## 🔵 Bloco 2 — AOP (Aspect-Oriented Programming)

> [!note] AOP é a base do Spring
> `@Transactional`, `@Cacheable`, Spring Security, `@Async` — tudo usa AOP internamente. Entender AOP resolve 80% dos "comportamentos mágicos" do Spring.

### Conceitos

- **Aspect** — módulo que encapsula cross-cutting concern
- **Join Point** — ponto de execução interceptável (método, no Spring)
- **Pointcut** — expressão que seleciona join points
- **Advice** — ação executada (Before, After, Around, AfterReturning, AfterThrowing)
- **Weaving** — processo de aplicar aspectos ao código (Spring usa proxies em runtime)

```java
@Aspect
@Component
@Slf4j
public class AuditAspect {

    // Pointcut: todos os métodos em services com @Auditable
    @Pointcut("@annotation(br.com.app.annotation.Auditable)")
    public void auditableMethods() {}

    @Around("auditableMethods()")
    public Object auditMethod(ProceedingJoinPoint pjp) throws Throwable {
        var start = System.currentTimeMillis();
        var method = pjp.getSignature().getName();
        var user = getCurrentUser();

        try {
            var result = pjp.proceed();
            log.info("AUDIT: user={} method={} duration={}ms status=SUCCESS",
                     user, method, System.currentTimeMillis() - start);
            return result;
        } catch (Exception ex) {
            log.error("AUDIT: user={} method={} duration={}ms status=ERROR error={}",
                      user, method, System.currentTimeMillis() - start, ex.getMessage());
            throw ex;
        }
    }
}
```

### Usos práticos de AOP customizado

- Logging de auditoria automático
- Rate limiting por anotação
- Retry automático com backoff
- Validação de regras de negócio cross-cutting
- Circuit breaker manual
- Métricas automáticas (Micrometer)

---

## 🔵 Bloco 3 — Programação Assíncrona

### @Async e ThreadPool

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean("emailTaskExecutor")
    public Executor emailTaskExecutor() {
        var executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("email-");
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    @Async("emailTaskExecutor")
    public CompletableFuture<Void> enviarEmail(String destinatario) {
        // executa em thread separada
        emailClient.send(destinatario);
        return CompletableFuture.completedFuture(null);
    }
}
```

### CompletableFuture — API moderna de async

```java
// Composição de operações assíncronas
CompletableFuture<DadosPedido> resultado = CompletableFuture
    .supplyAsync(() -> buscarPedido(id))
    .thenApplyAsync(pedido -> enriquecerComCliente(pedido))
    .thenCombineAsync(
        CompletableFuture.supplyAsync(() -> buscarEstoque(id)),
        (pedidoEnriquecido, estoque) -> combinar(pedidoEnriquecido, estoque)
    )
    .exceptionally(ex -> {
        log.error("Erro ao processar pedido", ex);
        return DadosPedido.empty();
    });
```

### Virtual Threads (Java 21) + Spring Boot 3.2+

```yaml
spring:
  threads:
    virtual:
      enabled: true  # habilita virtual threads para Tomcat e @Async
```

> [!tip] Game changer para performance
> Virtual threads eliminam a necessidade de reactive programming para a maioria dos casos de uso de I/O-bound. Com virtual threads, você escreve código imperativo com performance próxima ao reativo.

### WebFlux / Reactive Programming (quando usar)

- **Quando NÃO usar:** maioria dos CRUDs, APIs internas, quando a equipe não conhece bem
- **Quando usar:** streaming de dados, SSE (Server-Sent Events), WebSocket, alto volume de I/O paralelo onde virtual threads ainda não bastam
- `Mono<T>` — 0 ou 1 elemento
- `Flux<T>` — 0 a N elementos
- Operators: `map`, `flatMap`, `filter`, `zip`, `merge`, `buffer`, `window`
- **Backpressure** — controle de fluxo entre producer e consumer

---

## 🔵 Bloco 4 — Spring Batch

> [!note] Essencial para empresas enterprise
> Processamento em lote — ETL, relatórios, importações, conciliações financeiras. Muito usado em bancos, e-commerce, logística.

### Conceitos

- **Job** — unidade de trabalho batch
- **Step** — etapa dentro do Job (pode ter N steps)
- **ItemReader** → **ItemProcessor** → **ItemWriter** — pipeline de processamento
- **Chunk-oriented processing** — processa em lotes de N itens (controle de transação)
- **JobRepository** — persiste estado do Job no banco

```java
@Configuration
@EnableBatchProcessing
public class ImportacaoProdutosBatch {

    @Bean
    public Job importacaoProdutosJob(JobRepository repo, Step step) {
        return new JobBuilder("importacaoProdutos", repo)
            .incrementer(new RunIdIncrementer())
            .start(step)
            .build();
    }

    @Bean
    @StepScope
    public FlatFileItemReader<ProdutoCSV> reader(
            @Value("#{jobParameters['arquivo']}") String arquivo) {
        return new FlatFileItemReaderBuilder<ProdutoCSV>()
            .name("produtoReader")
            .resource(new FileSystemResource(arquivo))
            .delimited().delimiter(",")
            .names("sku", "nome", "preco", "estoque")
            .targetType(ProdutoCSV.class)
            .linesToSkip(1)
            .build();
    }

    @Bean
    public Step importarStep(JobRepository repo,
                              PlatformTransactionManager tx,
                              FlatFileItemReader<ProdutoCSV> reader,
                              ItemProcessor<ProdutoCSV, Produto> processor,
                              JpaItemWriter<Produto> writer) {
        return new StepBuilder("importar", repo)
            .<ProdutoCSV, Produto>chunk(500, tx)  // commit a cada 500
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .faultTolerant()
            .skipLimit(100)
            .skip(ParseException.class)
            .retryLimit(3)
            .retry(DeadlockLoserDataAccessException.class)
            .build();
    }
}
```

---

## 📖 Recursos

| Recurso | Nota |
|---------|------|
| **Spring Security in Action — Spilcă** | Melhor livro Spring Security |
| **Documentação Spring Security 6.x** | Oficial, bem escrita |
| Amigoscode Spring Security JWT (YouTube) | 🆓 |
| **Spring Batch Reference Documentation** | Oficial |

---

## 🧪 Projeto Prático

> [!example] Projeto: Plataforma de Vendas com Auth Completa
> - JWT com refresh token + revogação
> - OAuth2 com Google (Social Login)
> - Roles e permissões granulares
> - AOP para auditoria automática de operações sensíveis
> - Job Batch para importação de catálogo de produtos via CSV
> - Async para envio de notificações
> - Virtual threads habilitadas

---

## 🔗 Navegação

← [[T4 — Web, HTTP e Spring Boot Básico]]  
→ [[T6 — Testes e Qualidade de Código]]
