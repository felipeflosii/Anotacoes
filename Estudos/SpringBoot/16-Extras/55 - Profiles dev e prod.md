# 55 — Profiles: dev e prod

tags: #springboot #profiles #configuração #extras
links: [[54 - Logs com SLF4J]] | [[56 - Docker para Spring Boot]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## O que são profiles

Profiles permitem ter configurações diferentes por ambiente. Você ativa um profile e o Spring usa automaticamente as configurações correspondentes.

```
application.yml           ← configurações comuns (todos os ambientes)
application-dev.yml       ← sobrescreve para desenvolvimento local
application-test.yml      ← sobrescreve para testes automatizados
application-prod.yml      ← sobrescreve para produção
```

---

## Configuração completa por ambiente

```yaml
# application.yml — configurações compartilhadas
spring:
  application:
    name: concessionaria-api
  jpa:
    open-in-view: false
  flyway:
    enabled: true
    locations: classpath:db/migration

server:
  port: 8080

app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 86400000

springdoc:
  swagger-ui:
    path: /swagger-ui.html
```

```yaml
# application-dev.yml — desenvolvimento local
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/concessionaria_dev
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true

  # Swagger liberado em dev
springdoc:
  swagger-ui:
    enabled: true

logging:
  level:
    com.concessionaria: DEBUG
    org.hibernate.SQL: DEBUG

app:
  jwt:
    secret: ZGV2LXNlY3JldC1rZXktbXVpdG8tbG9uZ2EtcGFyYS10ZXN0ZQ==
```

```yaml
# application-test.yml — testes automatizados
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    database-platform: org.hibernate.dialect.H2Dialect
  flyway:
    enabled: false

app:
  jwt:
    secret: dGVzdC1zZWNyZXQta2V5LW11aXRvLWxvbmdhLXBhcmEtdGVzdGU=
    expiration: 3600000
```

```yaml
# application-prod.yml — produção
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT:5432}/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
  jpa:
    hibernate:
      ddl-auto: none      # NUNCA alterar schema em prod
    show-sql: false

springdoc:
  swagger-ui:
    enabled: false        # desabilitado em produção

logging:
  level:
    root: WARN
    com.concessionaria: INFO
  file:
    name: /var/log/app/concessionaria.log

server:
  error:
    include-stacktrace: never
    include-message: never
```

---

## Ativando profiles

```bash
# Linha de comando
java -jar concessionaria-api.jar --spring.profiles.active=prod

# Variável de ambiente (preferido para containers)
export SPRING_PROFILES_ACTIVE=prod
java -jar concessionaria-api.jar

# No IntelliJ: Run Configuration → Environment variables
# SPRING_PROFILES_ACTIVE=dev

# Em testes automatizados
@SpringBootTest
@ActiveProfiles("test")
class MinhaTest { }
```

---

## Beans condicionais por profile

```java
// Bean só existe no perfil dev
@Component
@Profile("dev")
public class DataSeeder implements CommandLineRunner {

    @Override
    public void run(String... args) {
        log.info("Populando banco de desenvolvimento com dados de exemplo...");
        // insere dados de teste
    }
}

// Bean diferente por ambiente
@Profile("dev")
@Bean
public EmailService emailServiceDev() {
    return new LogEmailService();  // só loga, não envia de verdade
}

@Profile("prod")
@Bean
public EmailService emailServiceProd() {
    return new SmtpEmailService();  // envia e-mail real
}
```

---

## Próximas notas
- [[56 - Docker para Spring Boot]] — containerizando a aplicação
