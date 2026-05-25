---
tags: [java, ano-2, docker, kubernetes, aws, gcp, ci-cd, cloud]
trimestre: T8
meses: 25-27
---

# T8 · Containers, Kubernetes e Cloud
### Meses 25–27 · Ano 2

---

## 🔵 Bloco 1 — Docker para Java

### Dockerfile otimizado para Spring Boot

```dockerfile
# Multi-stage build — imagem final sem JDK, só JRE
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY .mvn .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -q   # cacheia dependências
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Extrair layers para cache Docker eficiente
FROM eclipse-temurin:21-jdk-alpine AS extract
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# Imagem final
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=extract /app/dependencies/ ./
COPY --from=extract /app/spring-boot-loader/ ./
COPY --from=extract /app/snapshot-dependencies/ ./
COPY --from=extract /app/application/ ./

# Usuário não-root (segurança)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-XX:+ExitOnOutOfMemoryError", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "org.springframework.boot.loader.launch.JarLauncher"]
```

> [!tip] Flags JVM para containers
> - `-XX:+UseContainerSupport` — JVM respeita limits de CPU/RAM do container (Java 10+)
> - `-XX:MaxRAMPercentage=75.0` — usa 75% da RAM do container para heap
> - `-XX:+ExitOnOutOfMemoryError` — container reinicia em OOM (Kubernetes faz restart)

### Docker Compose para desenvolvimento

```yaml
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_PROFILES_ACTIVE: docker
      DATABASE_URL: jdbc:postgresql://db:5432/appdb
      REDIS_HOST: redis
      KAFKA_BROKERS: kafka:9092
    depends_on:
      db: { condition: service_healthy }
      kafka: { condition: service_healthy }
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s; timeout: 5s; retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --save 60 1 --requirepass redispass

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: CONTROLLER://0.0.0.0:9093,PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092

volumes:
  pgdata:
```

---

## 🔵 Bloco 2 — Kubernetes (k8s)

### Objetos Essenciais

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pedidos-service
  labels:
    app: pedidos-service
    version: "1.0.0"
spec:
  replicas: 3
  selector:
    matchLabels: { app: pedidos-service }
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0      # zero downtime deploy
  template:
    metadata:
      labels: { app: pedidos-service }
    spec:
      containers:
      - name: pedidos-service
        image: registry.company.com/pedidos-service:1.0.0
        ports: [{containerPort: 8080}]
        resources:
          requests: {cpu: 250m, memory: 512Mi}
          limits: {cpu: 1000m, memory: 1Gi}
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef: {name: db-secret, key: url}
        readinessProbe:
          httpGet: {path: /actuator/health/readiness, port: 8080}
          initialDelaySeconds: 20
          periodSeconds: 10
          failureThreshold: 3
        livenessProbe:
          httpGet: {path: /actuator/health/liveness, port: 8080}
          initialDelaySeconds: 60
          periodSeconds: 30
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 10"]  # graceful shutdown
```

### ConfigMap e Secrets

```yaml
# ConfigMap — configurações não-sensíveis
apiVersion: v1
kind: ConfigMap
metadata: {name: pedidos-config}
data:
  SPRING_PROFILES_ACTIVE: "prod"
  APP_PEDIDOS_MAX_ITENS: "50"

# Secret — dados sensíveis (base64 encoded)
apiVersion: v1
kind: Secret
metadata: {name: db-secret}
type: Opaque
data:
  url: amRiYzpwb3N0Z3Jlc3FsLy8...   # base64
  password: c3VwZXJzZWNyZXQ=
```

### HPA — Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: {name: pedidos-hpa}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pedidos-service
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target: {type: Utilization, averageUtilization: 70}
  - type: Resource
    resource:
      name: memory
      target: {type: Utilization, averageUtilization: 80}
```

---

## 🔵 Bloco 3 — CI/CD com GitHub Actions

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with: {java-version: '21', distribution: 'temurin', cache: 'maven'}
    - name: Run Tests
      run: ./mvnw verify -P coverage
    - name: SonarCloud Analysis
      run: ./mvnw sonar:sonar
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    - name: Upload Coverage
      uses: codecov/codecov-action@v3

  build-and-push:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Build Docker Image
      run: |
        docker build -t ${{ secrets.REGISTRY }}/pedidos-service:${{ github.sha }} .
        docker push ${{ secrets.REGISTRY }}/pedidos-service:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/pedidos-service \
          pedidos-service=${{ secrets.REGISTRY }}/pedidos-service:${{ github.sha }}
        kubectl rollout status deployment/pedidos-service --timeout=5m
```

---

## 🔵 Bloco 4 — Cloud (AWS/GCP/Azure)

### AWS — Serviços Essenciais para Java Dev

| Serviço | Para que usar |
|---------|--------------|
| **ECS / EKS** | Deploy de containers Spring Boot |
| **RDS (PostgreSQL)** | Banco gerenciado |
| **ElastiCache (Redis)** | Cache gerenciado |
| **MSK (Kafka)** | Kafka gerenciado |
| **SQS/SNS** | Mensageria serverless |
| **ECR** | Registry Docker |
| **Secrets Manager** | Secrets (DB passwords, API keys) |
| **CloudWatch** | Logs, métricas, alertas |
| **ALB** | Load balancer para APIs |
| **API Gateway** | Gateway para microservices |

### Spring Cloud AWS

```java
// Lendo secrets do AWS Secrets Manager
// application.yml
spring:
  cloud:
    aws:
      secretsmanager:
        import-keys:
          - /prod/myapp/database
          - /prod/myapp/redis
```

### Infrastructure as Code com Terraform

```hcl
# main.tf — ECS + RDS básico
resource "aws_ecs_service" "pedidos_service" {
  name            = "pedidos-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.pedidos.arn
  desired_count   = 3

  load_balancer {
    target_group_arn = aws_lb_target_group.pedidos.arn
    container_name   = "pedidos-service"
    container_port   = 8080
  }
}
```

---

## 🔗 Navegação

← [[T7 — Mensageria e Cache]]  
→ [[Ano 3 — Especialização]]
