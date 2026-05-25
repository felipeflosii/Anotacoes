---
tags: [java, setup, ferramentas, ambiente]
---

# 🛠️ Setup do Ambiente de Desenvolvimento

> Setup completo do zero para Java dev. Execute na ordem.

---

## 1. SDKMAN — Gerenciador de versões Java

```bash
# Instalar SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Instalar Java 21 (LTS atual — use sempre)
sdk install java 21.0.4-tem   # Eclipse Temurin (OpenJDK)

# Manter Java 17 também (legado em muitas empresas)
sdk install java 17.0.12-tem

# Definir padrão
sdk default java 21.0.4-tem

# Build tools
sdk install maven 3.9.9
sdk install gradle 8.10

# Verificar
java -version
mvn -version
gradle -version
```

---

## 2. Docker e Docker Compose

```bash
# macOS: instalar Docker Desktop
# https://www.docker.com/products/docker-desktop

# Linux (Ubuntu)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Verificar
docker --version
docker compose version
```

---

## 3. Kubernetes Tools

```bash
# macOS
brew install kubectl helm k9s kubectx stern

# Linux
# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# Para desenvolvimento local
brew install minikube   # ou kind (Kubernetes in Docker)

# Iniciar cluster local
minikube start --cpus 4 --memory 8g --driver docker
```

---

## 4. Ferramentas CLI Essenciais

```bash
# macOS
brew install git gh httpie terraform

# Linux
sudo apt install git httpie
# gh: https://cli.github.com

# Configurar git
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
git config --global init.defaultBranch main
git config --global pull.rebase true

# Autenticar GitHub CLI
gh auth login
```

---

## 5. IntelliJ IDEA

```bash
# macOS via Homebrew
brew install --cask intellij-idea-ce   # Community (grátis)
# ou
brew install --cask intellij-idea      # Ultimate (pago — 30 dias trial)

# Linux: baixar em https://www.jetbrains.com/idea/download
```

### Configurações recomendadas

```
Settings > Editor > Code Style > Java:
  - Scheme: Google Style (importar google-java-format)

Settings > Editor > Inspections:
  - Ativar todas SonarLint suggestions

Settings > Build > Build Tools > Maven:
  - Maven home path: apontar para $SDKMAN_DIR/candidates/maven/current

Settings > Keymap:
  - Shift+Shift: Search Everywhere
  - Ctrl+Alt+L: Reformat Code
  - Ctrl+Shift+T: Go to Test
```

### Plugins obrigatórios

```
Marketplace → buscar e instalar:
□ SonarLint
□ Lombok
□ Checkstyle-IDEA
□ Rainbow Brackets Lite
□ GitToolBox
□ .env files support
□ Kubernetes (Ultimate)
□ Docker (Ultimate)
```

---

## 6. Docker Compose para desenvolvimento local

Salve como `docker-compose.dev.yml` na raiz dos seus projetos:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    ports: ["5432:5432"]
    environment:
      POSTGRES_DB: devdb
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev -d devdb"]
      interval: 5s

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
    command: redis-server --save "" --appendonly no

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    ports: ["9092:9092"]
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_LISTENERS: CONTROLLER://0.0.0.0:9093,PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports: ["8090:8080"]
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092

volumes:
  pgdata:
```

```bash
# Iniciar todos os serviços de infra
docker compose -f docker-compose.dev.yml up -d

# Parar
docker compose -f docker-compose.dev.yml down
```

---

## 7. application.yml padrão para desenvolvimento

```yaml
# src/main/resources/application-local.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/devdb
    username: dev
    password: dev123
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    open-in-view: false
  kafka:
    bootstrap-servers: localhost:9092
  data:
    redis:
      host: localhost
      port: 6379

logging:
  level:
    root: INFO
    br.com.sua.empresa: DEBUG
    org.hibernate.SQL: DEBUG
    org.springframework.security: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
```

---

## 🔗 Navegação

→ [[Recursos Essenciais]]
