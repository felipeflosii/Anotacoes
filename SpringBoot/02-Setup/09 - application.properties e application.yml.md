# 09 — application.properties e application.yml

tags: #springboot #setup #configuração
links: [[06 - Criando Projeto com Spring Initializr]] | [[55 - Profiles dev e prod]] | [[🗺️ Mapa Principal]]

---

## O que são

São os arquivos de configuração da aplicação Spring Boot. Ficam em `src/main/resources/` e controlam tudo: porta do servidor, conexão com banco, configurações de segurança, logs, e qualquer propriedade customizada.

---

## Properties vs YAML — qual usar?

```properties
# application.properties — formato chave=valor
spring.datasource.url=jdbc:postgresql://localhost:5432/meubanco
spring.datasource.username=postgres
spring.datasource.password=senha123
spring.jpa.hibernate.ddl-auto=update
```

```yaml
# application.yml — formato hierárquico
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meubanco
    username: postgres
    password: senha123
  jpa:
    hibernate:
      ddl-auto: update
```

| Critério | Properties | YAML |
|---|---|---|
| **Legibilidade** | Repetitivo em prefixos longos | Hierárquico e limpo |
| **Suporte a listas** | Limitado | Nativo |
| **Complexidade** | Menor | Um pouco maior |
| **Padrão** | Default do Spring Boot | Muito popular no mercado |

> **Minha recomendação:** use YAML (`application.yml`). É mais legível, especialmente para configurações aninhadas. Ambos funcionam exatamente igual.

---

## Configuração completa comentada

```yaml
# application.yml — configuração completa para uma API REST

# ===== SERVIDOR =====
server:
  port: 8080                        # porta padrão — troque em produção
  servlet:
    context-path: /api              # prefixo de todas as URLs (opcional)

# ===== BANCO DE DADOS =====
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/concessionaria
    username: postgres
    password: ${DB_PASSWORD}        # ${} lê variável de ambiente
    driver-class-name: org.postgresql.Driver
    
    # Pool de conexões (HikariCP — padrão do Spring Boot)
    hikari:
      maximum-pool-size: 10         # máx conexões simultâneas
      minimum-idle: 5               # conexões mínimas mantidas ativas
      connection-timeout: 30000     # ms para obter conexão do pool

  # ===== JPA / HIBERNATE =====
  jpa:
    hibernate:
      # Opções de ddl-auto:
      # none     — não faz nada (produção com Flyway/Liquibase)
      # validate — valida schema mas não altera
      # update   — atualiza schema automaticamente (desenvolvimento)
      # create   — recria o schema a cada start (testes)
      # create-drop — cria no start, apaga no shutdown
      ddl-auto: none
      
    show-sql: false                 # true para ver SQL gerado no console
    properties:
      hibernate:
        format_sql: true            # formata o SQL exibido
        dialect: org.hibernate.dialect.PostgreSQLDialect
        
        # Performance: habilita batching de queries
        jdbc:
          batch_size: 25
        order_inserts: true
        order_updates: true

  # ===== FLYWAY (migrations) =====
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

# ===== SEGURANÇA / JWT =====
app:
  jwt:
    secret: ${JWT_SECRET}           # variável de ambiente — nunca hardcode!
    expiration: 86400000            # 24 horas em milissegundos

# ===== LOGS =====
logging:
  level:
    root: INFO                      # nível padrão
    com.empresa.minhaapi: DEBUG     # debug só do seu código
    org.hibernate.SQL: DEBUG        # ver SQLs (alternativa ao show-sql)
    org.springframework.security: DEBUG  # debug de segurança
  
  file:
    name: logs/application.log      # salvar logs em arquivo
    
  pattern:
    console: "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"

# ===== SWAGGER / OPENAPI =====
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method

# ===== ACTUATOR (monitoramento) =====
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when-authorized
```

---

## Variáveis de ambiente — segurança em produção

**Nunca** coloque senhas e segredos no `application.yml` que vai para o Git.

```yaml
# ❌ ERRADO — senha visível no repositório
spring:
  datasource:
    password: minhasenha123

# ✅ CORRETO — lê de variável de ambiente
spring:
  datasource:
    password: ${DB_PASSWORD}

# ✅ Com valor padrão (para desenvolvimento)
spring:
  datasource:
    password: ${DB_PASSWORD:postgres}
    # Se DB_PASSWORD não existir, usa "postgres"
```

Definindo variáveis de ambiente:

```bash
# Linux/Mac
export DB_PASSWORD=minhasenha
export JWT_SECRET=chavemuitorandômicoaqui

# Windows
set DB_PASSWORD=minhasenha

# No IntelliJ: Run/Debug Configurations → Environment variables
```

---

## Profiles — diferentes configurações por ambiente

```yaml
# application.yml — configurações comuns
spring:
  application:
    name: minha-api

---
# Ativa com: spring.profiles.active=dev
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:testdb     # banco em memória no dev
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop
  
logging:
  level:
    com.empresa: DEBUG

---
# Ativa com: spring.profiles.active=prod
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://${DB_HOST}:5432/${DB_NAME}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: none

logging:
  level:
    root: WARN
    com.empresa: INFO
```

Para ativar um profile:

```bash
# Via variável de ambiente
export SPRING_PROFILES_ACTIVE=dev

# Via linha de comando
java -jar minha-api.jar --spring.profiles.active=prod

# Via application.properties (não recomendado para produção)
spring.profiles.active=dev
```

---

## Criando propriedades customizadas

```yaml
# application.yml
app:
  nome: Minha API
  versao: 1.0.0
  upload:
    diretorio: /tmp/uploads
    tamanho-maximo: 10MB
  email:
    remetente: noreply@empresa.com
    smtp:
      host: smtp.gmail.com
      port: 587
```

```java
// Lendo com @Value (simples)
@Service
public class EmailService {
    
    @Value("${app.email.remetente}")
    private String remetente;
    
    @Value("${app.email.smtp.host}")
    private String smtpHost;
}

// Lendo com @ConfigurationProperties (recomendado para grupos)
@ConfigurationProperties(prefix = "app.email")
@Component
public class EmailProperties {
    private String remetente;
    private Smtp smtp;
    
    // getters e setters (ou @Data do Lombok)
    
    public static class Smtp {
        private String host;
        private int port;
        // getters e setters
    }
}

// Usando:
@Service
public class EmailService {
    
    private final EmailProperties emailProperties;
    
    public EmailService(EmailProperties emailProperties) {
        this.emailProperties = emailProperties;
    }
    
    public void enviar(String destino, String assunto, String corpo) {
        String host = emailProperties.getSmtp().getHost();
        // ...
    }
}
```

---

## Próximas notas
- [[10 - Arquitetura em Camadas]] — como estruturar o código
- [[55 - Profiles dev e prod]] — gerenciamento avançado de profiles
