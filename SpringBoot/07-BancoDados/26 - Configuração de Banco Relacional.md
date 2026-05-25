# 26 — Configuração de Banco Relacional

tags: #springboot #banco #postgresql #configuração
links: [[27 - Flyway Migrations]] | [[28 - Modelagem de Dados com JPA]] | [[21 - JPA e Hibernate Fundamentos]] | [[🗺️ Mapa Principal]]

---

## PostgreSQL — o banco recomendado

O PostgreSQL é o banco relacional mais completo e robusto disponível como software livre. Para projetos Spring Boot no mercado, é a escolha padrão.

```
Por que PostgreSQL?
✅ ACID completo
✅ Suporte nativo a JSON (JSONB)
✅ Full-text search embutido
✅ Window functions, CTEs
✅ Tipos avançados (arrays, UUID, intervalo)
✅ Excelente suporte pelo Hibernate
✅ Gratuito e open source
✅ Disponível em todos os cloud providers
```

---

## Instalação local — Docker (recomendado)

A forma mais prática de ter PostgreSQL localmente é via Docker:

```bash
# Subir PostgreSQL isolado, sem instalar no sistema
docker run -d \
  --name postgres-dev \
  -e POSTGRES_DB=meubanco \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:16-alpine

# Verificar se está rodando
docker ps

# Conectar via psql
docker exec -it postgres-dev psql -U postgres -d meubanco

# Parar
docker stop postgres-dev

# Iniciar novamente
docker start postgres-dev
```

---

## Configuração completa no Spring Boot

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meubanco
    username: postgres
    password: ${DB_PASSWORD:postgres}   # env var ou padrão "postgres"
    driver-class-name: org.postgresql.Driver

    # HikariCP — pool de conexões padrão do Spring Boot
    hikari:
      pool-name: HikariPool-Principal
      maximum-pool-size: 10          # máx conexões simultâneas
      minimum-idle: 5                # conexões mínimas ociosas mantidas
      idle-timeout: 300000           # 5 min: remove conexão ociosa acima do mínimo
      connection-timeout: 30000      # 30s: tempo máx para obter conexão do pool
      max-lifetime: 1800000          # 30 min: vida máxima de uma conexão
      connection-test-query: SELECT 1 # valida conexão antes de usar

  jpa:
    hibernate:
      ddl-auto: none                 # produção: deixa o Flyway gerenciar o schema
    show-sql: false
    open-in-view: false              # SEMPRE false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        default_schema: public
        jdbc:
          batch_size: 25             # agrupa INSERTs em lote
          fetch_size: 50             # fetch do banco em blocos
        order_inserts: true          # otimiza ordem dos INSERTs para batching
        order_updates: true
        generate_statistics: false   # true apenas para diagnóstico de performance
```

---

## ddl-auto — entendendo as opções

Esta é uma das configurações mais críticas e mais mal usadas:

| Valor | O que faz | Quando usar |
|---|---|---|
| `none` | Não faz nada | ✅ **Produção** — use sempre com Flyway |
| `validate` | Valida schema vs entidades (falha se diferente) | Desenvolvimento com Flyway |
| `update` | Adiciona colunas/tabelas novas, não remove | ⚠️ Dev rápido — pode perder dados |
| `create` | Recria o schema no startup | Testes — apaga dados ao reiniciar |
| `create-drop` | Cria no startup, apaga no shutdown | Testes integração |

```yaml
# Configuração por profile (padrão recomendado):

# application-dev.yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate   # valida mas não altera — Flyway gerencia
  flyway:
    enabled: true

# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop   # recria a cada teste
  flyway:
    enabled: false            # H2 não precisa de Flyway nos testes

# application-prod.yml
spring:
  jpa:
    hibernate:
      ddl-auto: none          # NUNCA deixe o Hibernate alterar prod
  flyway:
    enabled: true
```

---

## Múltiplos DataSources — quando necessário

```java
// Para sistemas com banco principal + banco de leitura (read replica):
@Configuration
public class DataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.write")
    public DataSource writeDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.read")
    public DataSource readDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

```yaml
spring:
  datasource:
    write:
      url: jdbc:postgresql://primary:5432/meubanco
      username: app_write
      password: ${DB_WRITE_PASSWORD}
    read:
      url: jdbc:postgresql://replica:5432/meubanco
      username: app_read
      password: ${DB_READ_PASSWORD}
```

---

## Conexão com H2 para testes

```xml
<!-- pom.xml — só para testes -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

```yaml
# src/test/resources/application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    database-platform: org.hibernate.dialect.H2Dialect
  flyway:
    enabled: false
```

---

## Monitorando conexões — actuator

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, metrics
  endpoint:
    health:
      show-details: always

# Acessar: GET /actuator/health
# Retorna status do banco: { "db": { "status": "UP", "details": { ... } } }

# Métricas do pool HikariCP: GET /actuator/metrics/hikaricp.connections.active
```

---

## Próximas notas
- [[27 - Flyway Migrations]] — gerenciamento de schema com versionamento
- [[28 - Modelagem de Dados com JPA]] — boas práticas de modelagem
