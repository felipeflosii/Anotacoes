# 54 — Logs com SLF4J

tags: #springboot #logs #slf4j #extras
links: [[55 - Profiles dev e prod]] | [[🗺️ Mapa Principal]]

---

## SLF4J — a fachada de logging

**SLF4J** é uma fachada (interface) para frameworks de log. O Spring Boot usa **Logback** como implementação padrão, mas você escreve código contra o SLF4J — se trocar a implementação, seu código não muda.

```java
// Com Lombok — forma recomendada
@Slf4j  // gera private static final Logger log = ...
@Service
public class VendaService {

    public VendaResponse realizar(VendaRequest request) {
        log.info("Iniciando venda — carro: {}, cliente: {}", request.carroId(), request.clienteId());

        // ... lógica ...

        log.info("Venda {} realizada com sucesso — valor: R$ {}", venda.getId(), venda.getValorVenda());
        return VendaResponse.from(venda);
    }
}

// Sem Lombok
public class MarcaService {
    private static final Logger log = LoggerFactory.getLogger(MarcaService.class);
}
```

---

## Níveis de log — quando usar cada um

| Nível | Quando usar | Exemplo |
|---|---|---|
| `TRACE` | Detalhe extremo (raramente) | Cada linha de loop |
| `DEBUG` | Info de desenvolvimento | SQL gerado, valores de variáveis |
| `INFO` | Eventos de negócio normais | "Venda realizada", "Usuário logado" |
| `WARN` | Algo inesperado mas recuperável | "Token expirado", "Tentativa de acesso negada" |
| `ERROR` | Erro que precisa de atenção | "Falha ao enviar e-mail", exceções inesperadas |

```java
@Service
@Slf4j
public class CarroService {

    public CarroResponse criar(CarroRequest request) {
        log.debug("Criando carro — request: {}", request);              // debug: detalhes

        var carro = new Carro(/* ... */);
        carroRepository.save(carro);

        log.info("Carro {} criado — {} {} {}", carro.getId(),          // info: evento de negócio
            carro.getAno(), carro.getMarca().getNome(), carro.getModelo());

        return CarroResponse.from(carro);
    }
}

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public ErrorResponse handle404(RecursoNaoEncontradoException ex, HttpServletRequest req) {
        log.warn("Recurso não encontrado em {}: {}", req.getRequestURI(), ex.getMessage()); // warn
        return ErrorResponse.of(404, "Não encontrado", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(Exception.class)
    public ErrorResponse handle500(Exception ex, HttpServletRequest req) {
        log.error("Erro inesperado em {}: {}", req.getRequestURI(), ex.getMessage(), ex); // error + stack
        return ErrorResponse.of(500, "Erro interno", "Tente mais tarde", req.getRequestURI());
    }
}
```

---

## Configuração de logs por ambiente

```yaml
# application-dev.yml
logging:
  level:
    root: INFO
    com.concessionaria: DEBUG          # seu código em DEBUG
    org.hibernate.SQL: DEBUG           # ver SQLs gerados
    org.hibernate.orm.jdbc.bind: TRACE # ver parâmetros das queries
  pattern:
    console: "%d{HH:mm:ss} [%thread] %-5level %logger{30} - %msg%n"

# application-prod.yml
logging:
  level:
    root: WARN
    com.concessionaria: INFO           # só eventos de negócio
    org.hibernate.SQL: OFF             # sem SQL em produção
  file:
    name: /var/log/concessionaria/app.log
  logback:
    rollingpolicy:
      max-file-size: 50MB
      max-history: 30                  # 30 dias de histórico
```

---

## Boas práticas de log

```java
// ✅ Use placeholders {} em vez de concatenação de string
log.info("Carro {} criado com preço {}", carro.getId(), carro.getPreco());
// ❌ Evite:
log.info("Carro " + carro.getId() + " criado com preço " + carro.getPreco());
// (A string é construída mesmo se o nível não for logado — desperdício)

// ✅ Log de início e fim de operações críticas
log.info("Iniciando processamento de lote de {} registros", lista.size());
// ... processa ...
log.info("Lote concluído em {}ms", Duration.between(inicio, LocalDateTime.now()).toMillis());

// ✅ Nunca logue dados sensíveis
log.info("Login do usuário: {}", usuario.getEmail());      // OK
log.info("Senha do usuário: {}", usuario.getSenha());      // ❌ NUNCA!
log.info("Token: {}", token);                              // ❌ NUNCA em produção!

// ✅ Estruture logs importantes para facilitar busca
log.info("[VENDA] id={} carro={} cliente={} valor={}", ...);
log.info("[AUTH] login email={} ip={}", email, ip);
```
