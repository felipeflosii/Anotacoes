# 48 — Cache com Spring

tags: #springboot #cache #performance #redis
links: [[46 - Lazy vs Eager Loading]] | [[47 - Problema N+1]] | [[49 - Projeto Concessionária - Visão Geral]] | [[🗺️ Mapa Principal]]

---

## Quando usar cache

```
Use cache quando:
✅ Dados mudam raramente (categorias, configurações, tabelas de preço)
✅ Query é cara (JOIN complexo, cálculo pesado)
✅ Mesmo dado é lido muitas vezes em curto período

Não use cache quando:
❌ Dados mudam com frequência (estoque em tempo real)
❌ Consistência imediata é crítica (saldo bancário)
❌ Dados são específicos por usuário (a menos que use cache por usuário)
```

---

## Cache simples — @Cacheable

```xml
<!-- Adicionar ao pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```java
// Habilitar cache na aplicação
@SpringBootApplication
@EnableCaching
public class MinhaApiApplication { ... }
```

```java
@Service
public class CategoriaService {

    private final CategoriaRepository repository;

    // @Cacheable: na 1ª chamada, executa o método e armazena no cache
    // Nas chamadas seguintes com o mesmo argumento, retorna do cache
    @Cacheable(value = "categorias", key = "#id")
    public CategoriaResponse buscarPorId(Long id) {
        // Só executa se não estiver no cache
        return repository.findById(id)
            .map(CategoriaResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Categoria", id));
    }

    // Cache de lista completa
    @Cacheable(value = "categorias-todas")
    public List<CategoriaResponse> listarTodas() {
        return repository.findAll().stream()
            .map(CategoriaResponse::from)
            .toList();
    }

    // @CacheEvict: invalida o cache quando o dado muda
    @CacheEvict(value = {"categorias", "categorias-todas"}, allEntries = true)
    @Transactional
    public CategoriaResponse criar(CategoriaRequest request) {
        var categoria = new Categoria(request.nome());
        return CategoriaResponse.from(repository.save(categoria));
    }

    // @CachePut: atualiza o cache após a operação (sem invalidar)
    @CachePut(value = "categorias", key = "#id")
    @Transactional
    public CategoriaResponse atualizar(Long id, CategoriaRequest request) {
        var categoria = repository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Categoria", id));
        categoria.atualizar(request.nome());
        return CategoriaResponse.from(categoria);
    }
}
```

---

## Cache com Redis — produção

```yaml
# application.yml com Redis
spring:
  cache:
    type: redis
    redis:
      time-to-live: 600000   # 10 minutos em ms
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```java
// Configuração do Redis com TTL por cache
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheConfiguration cacheConfig() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .disableCachingNullValues()
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
    }

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        Map<String, RedisCacheConfiguration> configs = Map.of(
            "categorias",       cacheConfig().entryTtl(Duration.ofHours(1)),
            "categorias-todas", cacheConfig().entryTtl(Duration.ofHours(1)),
            "produtos-destaque",cacheConfig().entryTtl(Duration.ofMinutes(15))
        );

        return RedisCacheManager.builder(factory)
            .cacheDefaults(cacheConfig())
            .withInitialCacheConfigurations(configs)
            .build();
    }
}
```

---

## Cache simples (sem Redis) — Caffeine para dev

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=500,expireAfterWrite=600s
```

---

## Próximas notas
- [[49 - Projeto Concessionária - Visão Geral]] — projeto completo
