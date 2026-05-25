# 56 — Docker para Spring Boot

tags: #springboot #docker #containerização #extras
links: [[55 - Profiles dev e prod]] | [[57 - Deploy]] | [[🗺️ Mapa Principal]]

---

## Por que Docker

Docker empacota sua aplicação e todas as dependências em um **container** — um ambiente isolado e reproduzível. Elimina o "funciona na minha máquina".

```
Sem Docker:
- Dev tem Java 17, prod tem Java 21 → comportamento diferente
- Dev tem PostgreSQL 14, CI tem PostgreSQL 16 → erro de sintaxe
- Cada desenvolvedor configura o ambiente na mão

Com Docker:
- Mesmo container em todos os lugares
- Ambiente declarado em arquivo versionado
- Subir o ambiente completo com 1 comando
```

---

## Dockerfile — construindo a imagem

```dockerfile
# Dockerfile na raiz do projeto

# === STAGE 1: BUILD ===
FROM maven:3.9-eclipse-temurin-21-alpine AS build
WORKDIR /app

# Copia o pom.xml primeiro (cache de dependências)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copia o código e compila
COPY src ./src
RUN mvn clean package -DskipTests -B

# === STAGE 2: RUNTIME ===
# Imagem menor para produção (só JRE, sem Maven)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Usuário não-root por segurança
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copia apenas o JAR gerado
COPY --from=build /app/target/*.jar app.jar

# Porta da aplicação
EXPOSE 8080

# Configurações JVM para container
ENV JAVA_OPTS="-Xmx512m -Xms256m -XX:+UseContainerSupport"

# Comando de execução
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

## .dockerignore — o que ignorar

```
# .dockerignore
target/
.git/
.gitignore
*.md
.mvn/
!.mvn/wrapper/
```

---

## docker-compose.yml — ambiente completo de desenvolvimento

```yaml
# docker-compose.yml
version: '3.8'

services:

  # Banco de dados
  postgres:
    image: postgres:16-alpine
    container_name: concessionaria-db
    environment:
      POSTGRES_DB: concessionaria
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Aplicação Spring Boot
  api:
    build: .
    container_name: concessionaria-api
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: concessionaria
      DB_USER: postgres
      DB_PASSWORD: postgres
      JWT_SECRET: dGhpcyBpcyBhIHZlcnkgbG9uZyBzZWNyZXQga2V5IGZvciBqd3Q=
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## Comandos Docker essenciais

```bash
# Construir a imagem
docker build -t concessionaria-api:latest .

# Subir o ambiente completo (banco + API)
docker-compose up -d

# Ver logs da aplicação
docker-compose logs -f api

# Parar tudo
docker-compose down

# Parar e apagar volumes (dados do banco)
docker-compose down -v

# Acessar o container
docker exec -it concessionaria-api sh

# Ver containers rodando
docker ps

# Reconstruir após mudanças
docker-compose up -d --build api
```

---

## Próximas notas
- [[57 - Deploy]] — publicando em produção
