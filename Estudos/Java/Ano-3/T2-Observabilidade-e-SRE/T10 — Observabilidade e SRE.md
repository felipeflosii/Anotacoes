---
tags: [java, ano-3, observabilidade, prometheus, grafana, opentelemetry, sre, logs]
trimestre: T10
meses: 31-33
---

# T10 · Observabilidade e SRE
### Meses 31–33 · Ano 3

> **Objetivo:** Operar sistemas em produção com confiança. Os três pilares: Logs, Métricas, Traces.

---

## 🔵 Bloco 1 — Os 3 Pilares da Observabilidade

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    LOGS     │    │   MÉTRICAS  │    │   TRACES    │
│             │    │             │    │             │
│ O que       │    │ Quanto      │    │ Por onde    │
│ aconteceu   │    │ está indo   │    │ passou      │
│             │    │             │    │             │
│ ELK Stack   │    │ Prometheus  │    │ Jaeger      │
│ Loki        │    │ + Grafana   │    │ Tempo       │
│ CloudWatch  │    │             │    │ Zipkin      │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 🔵 Bloco 2 — Logging Estruturado

> [!tip] Logs estruturados em JSON são obrigatórios
> Em produção com múltiplos serviços, logs em texto livre são inutilizáveis. Use JSON para que ferramentas como ELK, Loki e CloudWatch possam indexar e filtrar.

### Logback com JSON (Logstash Encoder)

```xml
<!-- logback-spring.xml -->
<configuration>
    <springProfile name="!local">
        <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>traceId</includeMdcKeyName>
                <includeMdcKeyName>spanId</includeMdcKeyName>
                <includeMdcKeyName>userId</includeMdcKeyName>
                <includeMdcKeyName>requestId</includeMdcKeyName>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON"/>
        </root>
    </springProfile>

    <springProfile name="local">
        <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```

### MDC (Mapped Diagnostic Context) — contexto em logs

```java
@Component
public class RequestContextFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {
        var requestId = Optional.ofNullable(request.getHeader("X-Request-ID"))
                                .orElse(UUID.randomUUID().toString());
        MDC.put("requestId", requestId);
        MDC.put("path", request.getRequestURI());
        response.setHeader("X-Request-ID", requestId);
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();  // CRÍTICO: limpar MDC ao final da request
        }
    }
}
```

### O que logar (e o que NÃO logar)

✅ **Logar:**
- Início e fim de operações de negócio importantes
- Erros com contexto suficiente (usuário, pedidoId, operação)
- Chamadas externas (duração, status)
- Transições de estado importantes

❌ **NÃO logar:**
- Senhas, tokens, CPFs, cartões (PII/PCI compliance)
- Query parameters de autenticação
- Stack traces completas em INFO (use ERROR)
- Logs em loops tight (performance killer)

---

## 🔵 Bloco 3 — Métricas com Micrometer + Prometheus

> [!note] Micrometer = SLF4J das métricas
> Micrometer abstrai o backend (Prometheus, Datadog, CloudWatch, etc). Escreva uma vez, exporte para qualquer sistema.

### Métricas customizadas

```java
@Service
@RequiredArgsConstructor
public class PedidoService {

    private final MeterRegistry meterRegistry;

    // Counter — eventos que só crescem
    private Counter pedidosCriados;
    private Counter pedidosCancelados;

    // Timer — duração de operações
    private Timer processamentoPagamento;

    // Gauge — valor atual (varia pra cima e pra baixo)
    private AtomicInteger pedidosEmProcessamento = new AtomicInteger(0);

    @PostConstruct
    void init() {
        pedidosCriados = Counter.builder("pedidos.criados")
            .description("Total de pedidos criados")
            .tag("ambiente", environment)
            .register(meterRegistry);

        processamentoPagamento = Timer.builder("pagamento.processamento")
            .description("Tempo de processamento de pagamento")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(meterRegistry);

        Gauge.builder("pedidos.em.processamento", pedidosEmProcessamento, AtomicInteger::get)
            .description("Pedidos atualmente em processamento")
            .register(meterRegistry);
    }

    public Pedido criar(PedidoCriacaoRequest request) {
        pedidosEmProcessamento.incrementAndGet();
        try {
            var pedido = // ...
            pedidosCriados.increment(Tags.of("canal", request.canal()));
            return pedido;
        } finally {
            pedidosEmProcessamento.decrementAndGet();
        }
    }

    public void processarPagamento(Long pedidoId) {
        processamentoPagamento.record(() -> {
            // operação cronometrada
            pagamentoGateway.processar(pedidoId);
        });
    }
}
```

### SLIs, SLOs e Error Budgets

```yaml
# Exemplos de SLOs para uma API
SLOs:
  disponibilidade:
    target: 99.9%  # 8.7 horas de downtime/ano
    medição: "sum(rate(http_requests_total{status!~'5..'}[5m])) / sum(rate(http_requests_total[5m]))"

  latência_p99:
    target: < 500ms
    medição: "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))"

  taxa_de_erro:
    target: < 0.1%
    medição: "rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m])"
```

### Alertas Prometheus

```yaml
# rules/pedidos.yml
groups:
- name: pedidos
  rules:
  - alert: TaxaDeErroCritica
    expr: |
      sum(rate(http_requests_total{job="pedidos-service",status=~"5.."}[5m]))
      / sum(rate(http_requests_total{job="pedidos-service"}[5m])) > 0.05
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "Taxa de erro acima de 5% em pedidos-service"
      description: "Taxa atual: {{ $value | humanizePercentage }}"

  - alert: LatenciaAltaP99
    expr: |
      histogram_quantile(0.99,
        rate(http_request_duration_seconds_bucket{job="pedidos-service"}[5m])
      ) > 1.0
    for: 5m
    labels:
      severity: warning
```

---

## 🔵 Bloco 4 — Distributed Tracing com OpenTelemetry

> [!tip] OpenTelemetry é o padrão da indústria
> Substitui Zipkin, Jaeger proprietary SDKs, etc. Vendor-neutral.

```yaml
# build.gradle
implementation 'io.micrometer:micrometer-tracing-bridge-otel'
implementation 'io.opentelemetry:opentelemetry-exporter-otlp'
```

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0   # 100% em dev; 0.1 (10%) em prod com alto volume
  otlp:
    tracing:
      endpoint: http://otel-collector:4318/v1/traces

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

```java
// Span customizado para rastrear operação de negócio
@Service
public class PedidoService {

    @NewSpan("processar-pagamento")
    public void processarPagamento(@SpanTag("pedido.id") Long pedidoId) {
        // todo o código dentro desta operação fica num span filho
    }
}
```

### Stack de Observabilidade completa (OSS)

```
Aplicação Java
    ↓ (OTLP)
OpenTelemetry Collector
    ├─→ Prometheus (métricas)  ─→ Grafana (dashboards + alertas)
    ├─→ Grafana Tempo (traces) ─→ Grafana (traces viewer)
    └─→ Grafana Loki (logs)    ─→ Grafana (log explorer)
```

---

## 🔵 Bloco 5 — Health Checks e Actuator

```yaml
management:
  endpoints:
    web:
      base-path: /management
      exposure:
        include: health, info, metrics, prometheus, env, loggers, threaddump, heapdump
  endpoint:
    health:
      show-details: when-authorized
      show-components: when-authorized
      probes:
        enabled: true  # /health/liveness e /health/readiness para k8s
    loggers:
      enabled: true    # permite mudar nível de log em runtime via HTTP
```

```java
// Health check customizado
@Component
public class KafkaHealthIndicator implements HealthIndicator {
    @Autowired private AdminClient kafkaAdminClient;

    @Override
    public Health health() {
        try {
            var topics = kafkaAdminClient.listTopics().names().get(3, SECONDS);
            return Health.up()
                .withDetail("topicsCount", topics.size())
                .build();
        } catch (Exception ex) {
            return Health.down()
                .withDetail("error", ex.getMessage())
                .build();
        }
    }
}
```

---

## 🔗 Navegação

← [[T9 — Microservices e Spring Cloud]]  
→ [[T11 — Performance e JVM Deep Dive]]
