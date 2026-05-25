---
tags: [java, recursos, livros, cursos, ferramentas, comunidade]
---

# 📚 Recursos Essenciais

---

## 📖 Livros Obrigatórios por Fase

### Fase 1 — Fundamentos (Ano 1–2)

| Livro | Autor | Por que ler |
|-------|-------|------------|
| ⭐⭐⭐⭐⭐ **Effective Java, 3rd Ed.** | Joshua Bloch | O livro definitivo de Java. Leitura obrigatória. |
| ⭐⭐⭐⭐⭐ **Clean Code** | Robert C. Martin | Escrever código que humanos conseguem manter |
| ⭐⭐⭐⭐ **Head First Design Patterns** | Freeman et al. | Patterns com didática visual — melhor para iniciantes |
| ⭐⭐⭐⭐ **Modern Java in Action** | Urma, Fusco, Mycroft | Java 8–17 na prática (streams, lambdas, records) |
| ⭐⭐⭐⭐ **Java Persistence with Hibernate** | Bauer, King, Gregory | Biblia do JPA/Hibernate |

### Fase 2 — Sênior (Ano 2–4)

| Livro | Autor | Por que ler |
|-------|-------|------------|
| ⭐⭐⭐⭐⭐ **Designing Data-Intensive Applications** | Martin Kleppmann | O livro mais importante de sistemas distribuídos |
| ⭐⭐⭐⭐⭐ **Building Microservices, 2nd Ed.** | Sam Newman | Microservices na prática |
| ⭐⭐⭐⭐⭐ **Clean Architecture** | Robert C. Martin | Hexagonal, SOLID, estrutura de projetos |
| ⭐⭐⭐⭐ **Release It!, 2nd Ed.** | Michael Nygard | Resiliência e patterns de produção |
| ⭐⭐⭐⭐ **The Pragmatic Programmer, 20th** | Hunt & Thomas | Craft de desenvolvimento |
| ⭐⭐⭐⭐ **Refactoring, 2nd Ed.** | Martin Fowler | Melhorar código existente sistematicamente |
| ⭐⭐⭐⭐ **Spring Security in Action** | Laurentiu Spilcă | Spring Security 6.x completo |

### Fase 3 — Staff/Arquiteto (Ano 4–5)

| Livro | Autor | Por que ler |
|-------|-------|------------|
| ⭐⭐⭐⭐⭐ **Staff Engineer** | Will Larson | O que é e como se tornar Staff |
| ⭐⭐⭐⭐⭐ **System Design Interview Vol. 1 e 2** | Alex Xu | Preparação para System Design em big techs |
| ⭐⭐⭐⭐ **Fundamentals of Software Architecture** | Richards & Ford | Estilos arquiteturais e trade-offs |
| ⭐⭐⭐⭐ **Software Architecture: The Hard Parts** | Richards & Ford | Decisões difíceis de arquitetura |
| ⭐⭐⭐⭐ **An Elegant Puzzle** | Will Larson | Engenharia e liderança técnica |
| ⭐⭐⭐ **The Manager's Path** | Camille Fournier | Se você vai para gestão técnica |

---

## 🎓 Cursos e Plataformas

### 🆓 Gratuitos — Melhores

| Recurso | Conteúdo | Link |
|---------|---------|------|
| **loiane.training** | Java completo em PT-BR, grátis e excelente | loiane.training |
| **Baeldung.com** | Melhor site de artigos Java/Spring em inglês | baeldung.com |
| **Spring Guides** | Guias oficiais curtos e práticos | spring.io/guides |
| **Java Brains (YouTube)** | Spring, Java, Microservices | youtube @JavaBrains |
| **Amigoscode (YouTube)** | Spring Boot, Docker, K8s, React | youtube @amigoscode |
| **Dan Vega (YouTube)** | Spring Boot moderno, dicas oficiais | youtube @danvega |
| **Marco Behler (YouTube)** | Java explicado de forma prática | youtube @MarcoBehler |
| **dev.java** | Portal oficial Oracle — tutoriais Java 21+ | dev.java |

### 💰 Pagos — Vale o investimento

| Plataforma/Curso | Destaque | Custo |
|-----------------|---------|-------|
| **Udemy — Nélio Alves (Java COMPLETO)** | Melhor curso Java completo em PT-BR | R$30–80 promoção |
| **Udemy — Stéphane Maarek (Kafka)** | Melhor curso Kafka do mercado | R$30–80 promoção |
| **Udemy — Mumshad Mannambeth (K8s)** | Melhor curso Kubernetes | R$30–80 promoção |
| **Pluralsight** | Java, Spring, arquitetura | USD 29/mês |
| **O'Reilly Learning** | Livros + cursos + sandbox | USD 49/mês |
| **A Cloud Guru** | AWS, GCP, K8s hands-on | USD 49/mês |

---

## 🛠️ Ferramentas Essenciais

### IDEs e Editores

| Ferramenta | Uso |
|-----------|-----|
| **IntelliJ IDEA Community** (grátis) | IDE principal — melhor para Java |
| **IntelliJ IDEA Ultimate** (pago) | Spring Boot, banco, Docker integrados |
| **VS Code** | Scripts, Markdown, YAML, Terraform |

### Plugins IntelliJ indispensáveis

- **SonarLint** — análise estática em tempo real
- **Lombok** — suporte a anotações Lombok
- **Checkstyle-IDEA** — estilo de código
- **Rainbow Brackets** — colchetes coloridos
- **GitToolBox** — Git inline
- **HTTP Client** — testar APIs (substitui Postman)
- **Database Tools** — cliente SQL integrado (Ultimate)
- **Docker** — gerenciar containers (Ultimate)
- **Kubernetes** — visualizar recursos k8s (Ultimate)

### Ferramentas de Linha de Comando

```bash
# Gerenciamento de versões Java
sdk install java 21-tem      # SDKMAN — obrigatório

# Build
mvn, gradle                  # Maven e Gradle

# HTTP
httpie, curl                 # Testar APIs na linha de comando

# Docker
docker, docker compose       # Containers

# Kubernetes
kubectl                      # CLI principal
helm                         # Package manager para k8s
k9s                          # UI terminal para k8s — essencial
kubectx + kubens             # Troca rápida de contexto/namespace
stern                        # Tail de logs de múltiplos pods

# Banco de dados
pgcli                        # Cliente PostgreSQL com autocomplete
redis-cli                    # Cliente Redis

# Monitoramento
htop, btop                   # Sistema
jcmd, jstat                  # JVM tools
async-profiler               # Profiling JVM

# Terraform
terraform, terragrunt         # IaC

# Git
git, gh (GitHub CLI)         # Controle de versão
```

---

## 🌐 Comunidades

### Brasileiras

| Comunidade | Plataforma | Link |
|-----------|-----------|------|
| Java Brasil | Telegram | t.me/JavaBrasil |
| Spring Brasil | Discord | buscar "Spring Brasil" no Discord |
| Kotlin Brasil | Telegram | t.me/kotlinbrasil |
| Dev. BR | Discord | discord.gg/devbr |
| Café com Java | Meetup/YouTube | youtube @CafecomJava |

### Internacionais

| Comunidade | Plataforma | Link |
|-----------|-----------|------|
| r/java | Reddit | reddit.com/r/java |
| Spring Community | Discord | discord.gg/spring-framework |
| JVM Weekly newsletter | Email | jvmweekly.com |
| InfoQ Java | Site | infoq.com/java |
| Baeldung newsletter | Email | baeldung.com |

---

## 📰 Newsletters para se manter atualizado

| Newsletter | Frequência | Foco |
|-----------|-----------|------|
| **This Week in Spring** (Josh Long) | Semanal | Spring ecosystem news |
| **JVM Weekly** | Semanal | Java/Kotlin/Scala |
| **Java Annotated Monthly** (JetBrains) | Mensal | Java ecosystem overview |
| **InfoQ Java** | Semanal | Arquitetura, tendências |
| **Last Week in AWS** | Semanal | AWS news |
| **TLDR** (dev) | Diária | Tech news geral |

---

## 🔧 Setup do Ambiente de Desenvolvimento

```bash
# macOS/Linux — setup completo

# 1. Homebrew (macOS) ou apt (Ubuntu)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. SDKMAN para Java
curl -s "https://get.sdkman.io" | bash
sdk install java 21.0.4-tem
sdk install java 17.0.12-tem
sdk install maven 3.9.9
sdk install gradle 8.10

# 3. Ferramentas essenciais
brew install git gh httpie k9s kubectx stern terraform

# 4. Docker Desktop
# Download: https://www.docker.com/products/docker-desktop

# 5. IntelliJ IDEA
# Download: https://www.jetbrains.com/idea/download

# Verificar setup
java -version          # deve ser 21.x
mvn -version
gradle -version
docker --version
kubectl version --client
```

---

## 🔗 Links Rápidos

```
Documentação oficial Spring Boot:
https://docs.spring.io/spring-boot/

Spring Initializr (criar projetos):
https://start.spring.io

Baeldung (artigos Java/Spring):
https://baeldung.com

Java SE Documentation:
https://docs.oracle.com/en/java/javase/21/

Docker Hub:
https://hub.docker.com

Kubernetes Documentation:
https://kubernetes.io/docs

AWS Documentation:
https://docs.aws.amazon.com

Maven Central:
https://search.maven.org

MVNRepository (buscar dependências):
https://mvnrepository.com
```
