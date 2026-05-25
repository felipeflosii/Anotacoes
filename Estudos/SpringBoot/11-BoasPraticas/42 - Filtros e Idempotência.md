# 42 — Filtros e Idempotência

tags: #springboot #rest #filtros #idempotência
links: [[39 - Padrão REST Completo]] | [[41 - Paginação e Ordenação]] | [[43 - Testes Unitários com JUnit e Mockito]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Filtros — boas práticas

Filtros são query parameters que refinam uma listagem. Algumas regras:

```java
// ✅ Filtros via query params — nunca no path
GET /produtos?categoria=eletronicos&ativo=true&precoMax=500

// ❌ Filtros no path — errado
GET /produtos/categoria/eletronicos/ativo/true

// ✅ Filtros opcionais — null = sem filtro
@RequestParam(required = false) String nome   // se não enviado, busca todos

// ✅ Filtros com valores padrão sensatos
@RequestParam(defaultValue = "true") boolean ativo  // por padrão, só ativos

// ✅ Datas como query params — formato ISO-8601
GET /pedidos?dataInicio=2024-01-01&dataFim=2024-01-31
// Spring converte automaticamente com @DateTimeFormat:
@RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate dataInicio
```

---

## Idempotência — conceito essencial

Uma operação é **idempotente** se executá-la N vezes tem o mesmo efeito que executar 1 vez.

| Método | Idempotente? | Por quê |
|---|---|---|
| GET | ✅ Sim | Leitura não altera estado |
| PUT | ✅ Sim | Substituição total — mesmo resultado |
| DELETE | ✅ Sim | Deletar já deletado = mesmo resultado |
| POST | ❌ Não | Criar 2x = 2 recursos distintos |
| PATCH | ❌ Depende | Depende da operação |

### Tornando POST idempotente com Idempotency-Key

Para pagamentos e operações críticas, use um header de idempotência:

```java
// Cliente envia um UUID único por tentativa
// POST /pagamentos
// Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

@PostMapping("/pagamentos")
public ResponseEntity<PagamentoResponse> processar(
    @RequestBody @Valid PagamentoRequest request,
    @RequestHeader(value = "Idempotency-Key", required = false) String idempotencyKey
) {
    if (idempotencyKey != null) {
        // Verificar se já processamos esta chave
        Optional<PagamentoResponse> jaProcessado =
            idempotencyService.buscar(idempotencyKey);

        if (jaProcessado.isPresent()) {
            return ResponseEntity.ok(jaProcessado.get());  // retorna resultado anterior
        }
    }

    PagamentoResponse response = pagamentoService.processar(request);

    if (idempotencyKey != null) {
        idempotencyService.salvar(idempotencyKey, response);  // salva para futuras chamadas
    }

    return ResponseEntity.status(201).body(response);
}
```

---

## Rate Limiting — limitando requisições

```java
// Com Spring + Bucket4j (biblioteca de rate limiting):
@Component
public class RateLimitFilter extends OncePerRequestFilter {

    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String ip = req.getRemoteAddr();
        Bucket bucket = buckets.computeIfAbsent(ip, k -> criarBucket());

        if (bucket.tryConsume(1)) {
            chain.doFilter(req, res);
        } else {
            res.setStatus(429);
            res.setContentType("application/json");
            res.getWriter().write(
                """{"status":429,"erro":"Muitas requisições","mensagem":"Aguarde antes de tentar novamente"}"""
            );
        }
    }

    private Bucket criarBucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(100, Refill.greedy(100, Duration.ofMinutes(1))))
            // 100 requisições por minuto por IP
            .build();
    }
}
```

---

## Próximas notas
- [[43 - Testes Unitários com JUnit e Mockito]] — módulo de testes
