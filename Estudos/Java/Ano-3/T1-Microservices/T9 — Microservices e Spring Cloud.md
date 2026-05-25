---
tags: [java, ano-3, microservices, spring-cloud, resilience4j, service-mesh]
trimestre: T9
meses: 28-30
---

# T9 · Microservices e Spring Cloud
### Meses 28–30 · Ano 3

---

## 🔵 Bloco 1 — Microservices na Prática

### Quando NÃO usar microservices

> [!warning] Microservices não são bala de prata
> Comece monolito modular. Extraia microservices quando tiver:
> - Times independentes que precisam deployar em cadências diferentes
> - Componentes com requisitos de escala muito distintos
> - Domínios com acoplamento baixo natural

### Domain-Driven Design aplicado

- **Bounded Context** — limite de um microservice
- **Aggregate** — consistência transacional dentro do contexto
- **Domain Events** — comunicação entre contextos
- **Anti-Corruption Layer** — adaptar modelos externos ao seu domínio
- **Context Map** — como bounded contexts se relacionam

### Comunicação entre Serviços

| Estilo | Protocolo | Quando usar |
|--------|-----------|-------------|
| Síncrono | REST/HTTP | Leitura imediata, response necessário |
| Síncrono | gRPC | Alta performance, schemas estritos, streaming |
| Assíncrono | Kafka | Eventos de domínio, alta resiliência |
| Assíncrono | RabbitMQ | Task queues, processamento background |

---

## 🔵 Bloco 2 — Spring Cloud

### Service Discovery — Eureka

```java
// servidor
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApp { ... }

// cliente
@SpringBootApplication
@EnableDiscoveryClient
public class PedidosServiceApp { ... }
```

```yaml
# cliente
eureka:
  client:
    service-url:
      defaultZone: http://discovery:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
```

> [!note] Kubernetes substitui Eureka
> Se você já tem k8s, use DNS nativo do cluster. Eureka só faz sentido fora de k8s.

### API Gateway — Spring Cloud Gateway

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("pedidos-service", r -> r
                .path("/api/pedidos/**")
                .filters(f -> f
                    .circuitBreaker(c -> c
                        .setName("pedidosCB")
                        .setFallbackUri("forward:/fallback/pedidos"))
                    .requestRateLimiter(rl -> rl
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(ipKeyResolver()))
                    .retry(3)
                    .addRequestHeader("X-Gateway-Version", "2.0")
                )
                .uri("lb://pedidos-service"))
            .build();
    }
}
```

### Config Server

```yaml
# config server — serve configs do Git
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/empresa/config-repo
          search-paths: "{application}"
          default-label: main
          clone-on-start: true

# cliente — busca config no startup
spring:
  config:
    import: "configserver:http://config-server:8888"
  application:
    name: pedidos-service
  profiles:
    active: prod
```

---

## 🔵 Bloco 3 — Resilience4j

> [!tip] Resilience4j substituiu Hystrix
> Hystrix está em manutenção. Use Resilience4j — é mais leve, funcional e bem integrado com Spring Boot.

### Circuit Breaker

```java
@Service
public class EstoqueClient {

    @CircuitBreaker(name = "estoque", fallbackMethod = "fallbackEstoque")
    @Retry(name = "estoque")
    @TimeLimiter(name = "estoque")
    public CompletableFuture<EstoqueDTO> verificarEstoque(Long produtoId) {
        return CompletableFuture.supplyAsync(() ->
            estoqueApi.getEstoque(produtoId));
    }

    private CompletableFuture<EstoqueDTO> fallbackEstoque(Long produtoId, Throwable t) {
        log.warn("Circuit breaker aberto para estoque: {}", t.getMessage());
        return CompletableFuture.completedFuture(EstoqueDTO.indisponivel());
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      estoque:
        sliding-window-type: COUNT_BASED
        sliding-window-size: 10
        failure-rate-threshold: 50          # 50% de falhas abre o CB
        slow-call-rate-threshold: 80
        slow-call-duration-threshold: 2s
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 5
        automatic-transition-from-open-to-half-open-enabled: true
  retry:
    instances:
      estoque:
        max-attempts: 3
        wait-duration: 500ms
        exponential-backoff-multiplier: 2   # 500ms, 1s, 2s
        retry-exceptions:
          - java.net.ConnectException
          - java.util.concurrent.TimeoutException
  timelimiter:
    instances:
      estoque:
        timeout-duration: 3s
```

### Bulkhead

```java
@Bulkhead(name = "dbCalls", type = Bulkhead.Type.SEMAPHORE)
public Produto buscarProduto(Long id) { ... }

@Bulkhead(name = "asyncCalls", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<Void> processarAsync(Long id) { ... }
```

---

## 🔵 Bloco 4 — Comunicação Síncrona Avançada

### gRPC com Java

```protobuf
// produto.proto
syntax = "proto3";
package br.com.app;

service ProdutoService {
    rpc BuscarProduto (BuscarProdutoRequest) returns (ProdutoResponse);
    rpc ListarProdutos (ListarRequest) returns (stream ProdutoResponse);
    rpc AtualizarEstoque (stream AtualizarEstoqueRequest) returns (ResultadoResponse);
}

message ProdutoResponse {
    int64 id = 1;
    string nome = 2;
    string sku = 3;
    double preco = 4;
    int32 estoque = 5;
}
```

```java
// implementação do server
@GrpcService
public class ProdutoGrpcService extends ProdutoServiceGrpc.ProdutoServiceImplBase {

    @Override
    public void buscarProduto(BuscarProdutoRequest request,
                               StreamObserver<ProdutoResponse> observer) {
        produtoRepository.findById(request.getId())
            .map(this::toProto)
            .ifPresentOrElse(
                proto -> { observer.onNext(proto); observer.onCompleted(); },
                () -> observer.onError(Status.NOT_FOUND.asException())
            );
    }
}
```

### OpenFeign — HTTP Client declarativo

```java
@FeignClient(
    name = "clientes-service",
    url = "${services.clientes.url}",
    configuration = FeignConfig.class,
    fallbackFactory = ClienteClientFallbackFactory.class
)
public interface ClienteClient {

    @GetMapping("/api/clientes/{id}")
    ClienteDTO buscarPorId(@PathVariable Long id);

    @PostMapping("/api/clientes")
    ClienteDTO criar(@RequestBody ClienteCriacaoDTO dto);
}
```

---

## 🔵 Bloco 5 — Sagas e Consistência Eventual

> [!note] O maior desafio de microservices
> Transações distribuídas são impossíveis com ACID. Use Sagas para orquestrar operações distribuídas.

### Saga Pattern — Coreografia vs Orquestração

**Coreografia (via Kafka):**
```
Pedido Criado → Reservar Estoque → Processar Pagamento → Confirmar Envio
     ↓ falhou          ↓ falhou
Cancelar Pedido ← Liberar Estoque ← Estornar Pagamento
```

**Orquestração (Saga Orchestrator):**
```java
// O Orchestrator controla o fluxo explicitamente
public class CriarPedidoSaga {
    public void execute(CriarPedidoCommand cmd) {
        try {
            var pedidoId = pedidoService.criar(cmd);
            var reservaId = estoqueService.reservar(pedidoId, cmd.itens());
            var pagamentoId = pagamentoService.processar(pedidoId, cmd.pagamento());
            pedidoService.confirmar(pedidoId);
        } catch (EstoqueException ex) {
            pedidoService.cancelar(cmd.pedidoId(), "Sem estoque");
        } catch (PagamentoException ex) {
            estoqueService.liberar(reservaId);
            pedidoService.cancelar(cmd.pedidoId(), "Pagamento falhou");
        }
    }
}
```

### Outbox Pattern — garantia de entrega de eventos

```java
@Transactional
public Pedido criar(PedidoCriacaoRequest request) {
    var pedido = pedidoRepository.save(new Pedido(request));
    // salva evento na mesma transação do DB
    outboxRepository.save(new OutboxEvent(
        "pedidos.criados",
        pedido.getId().toString(),
        objectMapper.writeValueAsString(new PedidoCriadoEvent(pedido))
    ));
    return pedido;
    // Debezium ou Polling publisher lê outbox e publica no Kafka
}
```

---

## 🔗 Navegação

← [[T8 — Containers e Cloud]]  
→ [[T10 — Observabilidade e SRE]]
