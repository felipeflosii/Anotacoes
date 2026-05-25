---
tags: [java, ano-2, kafka, rabbitmq, redis, cache, mensageria]
trimestre: T7
meses: 22-24
---

# T7 · Mensageria e Cache
### Meses 22–24 · Ano 2

---

## 🔵 Bloco 1 — Apache Kafka

> [!tip] Kafka é o padrão do mercado para event streaming
> Nubank, iFood, Mercado Livre, Amazon, LinkedIn — todos usam Kafka. Dominar Kafka é diferencial para qualquer vaga sênior.

### Conceitos Fundamentais

- **Topic** — canal de mensagens (como uma tabela imutável e append-only)
- **Partition** — subdivisão paralela de um topic; ordem garantida DENTRO da partição
- **Offset** — posição de uma mensagem na partição
- **Producer** — quem publica mensagens
- **Consumer** — quem consome mensagens
- **Consumer Group** — grupo de consumers que compartilham carga; cada partição → 1 consumer do grupo
- **Broker** — servidor Kafka
- **Replication Factor** — quantas cópias de cada partição (tolerância a falhas)

### Producer com Spring Kafka

```java
@Service
@RequiredArgsConstructor
public class PedidoEventPublisher {

    private final KafkaTemplate<String, Object> kafkaTemplate;

    public void publicarPedidoCriado(PedidoCriadoEvent event) {
        kafkaTemplate.send("pedidos.criados", event.pedidoId().toString(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Falha ao publicar evento pedidoId={}", event.pedidoId(), ex);
                    // dead letter, retry, alerta
                } else {
                    log.info("Evento publicado topic={} partition={} offset={}",
                             result.getRecordMetadata().topic(),
                             result.getRecordMetadata().partition(),
                             result.getRecordMetadata().offset());
                }
            });
    }
}
```

### Consumer com Spring Kafka

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class PedidoCriadoConsumer {

    private final EstoqueService estoqueService;

    @KafkaListener(
        topics = "pedidos.criados",
        groupId = "estoque-service",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consumir(
            @Payload PedidoCriadoEvent event,
            @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment ack) {
        log.info("Processando pedidoId={} partition={} offset={}", event.pedidoId(), partition, offset);
        try {
            estoqueService.reservarItens(event.pedidoId(), event.itens());
            ack.acknowledge();  // commit manual
        } catch (EstoqueInsuficienteException ex) {
            log.warn("Estoque insuficiente, enviando para DLT: {}", ex.getMessage());
            ack.acknowledge();  // acknowledge para não reprocessar infinitamente
            // publicar em pedidos.criados.DLT
        } catch (Exception ex) {
            log.error("Erro temporário, NÃO fazendo acknowledge", ex);
            // NÃO ack = mensagem será reprocessada
        }
    }
}
```

### Configuração Crítica

```yaml
spring:
  kafka:
    bootstrap-servers: ${KAFKA_BROKERS:localhost:9092}
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all           # espera confirmação de todos os replicas (durabilidade máxima)
      retries: 3
      properties:
        enable.idempotence: true  # exatamente uma entrega (producer idempotente)
        max.in.flight.requests.per.connection: 5
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false   # SEMPRE manual commit em produção
      max-poll-records: 500
    listener:
      ack-mode: manual_immediate
```

### Padrões importantes com Kafka

- **Outbox Pattern** — garantir consistência entre DB e Kafka (transactional outbox)
- **Dead Letter Topic (DLT)** — mensagens que falharam após N retries
- **Idempotent Consumer** — processar a mesma mensagem múltiplas vezes sem efeito duplicado
- **Event Sourcing** — Kafka como source of truth, não como cache
- **CQRS + Kafka** — separar writes de reads via eventos
- **Compacted Topics** — manter apenas último valor por chave (materializar estado)

---

## 🔵 Bloco 2 — RabbitMQ

> [!note] RabbitMQ vs Kafka
> RabbitMQ = broker tradicional, mensagens são consumidas e descartadas. Melhor para task queues, RPC, filas de trabalho.
> Kafka = log distribuído, mensagens são retidas. Melhor para event streaming, auditoria, replay.

```java
// Configuração de exchanges, queues e bindings
@Configuration
public class RabbitMQConfig {

    @Bean
    public TopicExchange pedidosExchange() {
        return ExchangeBuilder.topicExchange("pedidos.exchange")
            .durable(true).build();
    }

    @Bean
    public Queue filaNotificacoes() {
        return QueueBuilder.durable("notificacoes.pedidos")
            .withArgument("x-dead-letter-exchange", "dlx.exchange")
            .withArgument("x-message-ttl", 300000)
            .build();
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(filaNotificacoes())
            .to(pedidosExchange())
            .with("pedido.#");  // routing key pattern
    }
}

// Publisher
@Service
public class PedidoNotificationPublisher {
    @Autowired private RabbitTemplate rabbitTemplate;

    public void publicar(PedidoEvent event) {
        rabbitTemplate.convertAndSend(
            "pedidos.exchange",
            "pedido.criado",
            event,
            message -> {
                message.getMessageProperties().setContentType("application/json");
                message.getMessageProperties().setDeliveryMode(PERSISTENT);
                return message;
            });
    }
}
```

---

## 🔵 Bloco 3 — Redis e Caching

### Redis — além de cache

| Uso | Estrutura Redis | Exemplo |
|-----|----------------|---------|
| Cache de objetos | String (JSON) | Resultado de query |
| Session storage | Hash | Dados de sessão do usuário |
| Rate limiting | String + INCR + EXPIRE | Limite de requests por IP |
| Distributed lock | String + SET NX | Lock para operações críticas |
| Leaderboard | Sorted Set | Ranking de vendedores |
| Pub/Sub | Pub/Sub | Notificações em tempo real |
| Geolocalização | Geo | Entregadores próximos |
| Fila de trabalho | List | Simples task queue |

### @Cacheable com Spring

```java
@Service
@CacheConfig(cacheNames = "produtos")
public class ProdutoService {

    @Cacheable(key = "#id", unless = "#result == null")
    public ProdutoDTO buscarPorId(Long id) {
        return repository.findById(id)
            .map(mapper::toDTO)
            .orElseThrow(() -> new NotFoundException("Produto " + id));
    }

    @CacheEvict(key = "#produto.id")
    public ProdutoDTO atualizar(ProdutoUpdateDTO produto) { ... }

    @CachePut(key = "#result.id")
    public ProdutoDTO criar(ProdutoCriacaoDTO dto) { ... }

    @Caching(evict = {
        @CacheEvict(key = "#id"),
        @CacheEvict(cacheNames = "catalogos", allEntries = true)
    })
    public void deletar(Long id) { ... }
}
```

```yaml
spring:
  cache:
    type: redis
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: 6379
      password: ${REDIS_PASSWORD:}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 16
          max-idle: 8

  cache:
    redis:
      time-to-live: 3600000  # 1 hora padrão
      cache-null-values: false
      key-prefix: "app:cache:"
```

### Redis com RedisTemplate (controle total)

```java
@Service
@RequiredArgsConstructor
public class RateLimiterService {

    private final StringRedisTemplate redisTemplate;

    public boolean isAllowed(String clientId, int maxRequests, Duration window) {
        var key = "rate:" + clientId;
        var script = """
            local current = redis.call('INCR', KEYS[1])
            if current == 1 then
                redis.call('PEXPIRE', KEYS[1], ARGV[2])
            end
            if current > tonumber(ARGV[1]) then
                return 0
            end
            return 1
            """;
        Long result = redisTemplate.execute(
            RedisScript.of(script, Long.class),
            List.of(key),
            String.valueOf(maxRequests),
            String.valueOf(window.toMillis())
        );
        return result != null && result == 1L;
    }
}
```

### Distributed Lock com Redis

```java
// Use Redisson — biblioteca de alto nível para Redis
@Service
public class PagamentoService {
    @Autowired private RedissonClient redisson;

    public void processarPagamento(Long pedidoId) {
        var lock = redisson.getLock("lock:pagamento:" + pedidoId);
        try {
            if (!lock.tryLock(5, 30, TimeUnit.SECONDS)) {
                throw new ConcurrencyException("Pagamento já em processamento");
            }
            // operação crítica aqui
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

### Cache Patterns Avançados

- **Cache-Aside** (Lazy Loading) — aplicação gerencia cache (padrão Spring @Cacheable)
- **Write-Through** — escreve em cache E banco simultaneamente
- **Write-Behind** (Write-Back) — escreve em cache, persiste assincronamente
- **Read-Through** — cache busca no banco automaticamente
- **Cache Stampede / Thundering Herd** — problema quando cache expira com alta concorrência → use probabilistic early expiration

---

## 🔗 Navegação

← [[T6 — Testes e Qualidade de Código]]  
→ [[T8 — Containers e Cloud]]
