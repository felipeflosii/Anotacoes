# 08 — Maven vs Gradle

tags: #springboot #setup #maven #gradle
links: [[06 - Criando Projeto com Spring Initializr]] | [[07 - Estrutura de Projeto Spring Boot]] | [[🗺️ Mapa Principal]]

---

## O que são

Ambos são **ferramentas de build** para projetos Java. Elas gerenciam:
- Download e versionamento de dependências
- Compilação do código
- Execução de testes
- Geração do artefato (JAR ou WAR)
- Deploy e publicação

---

## Comparação direta

| Critério | Maven | Gradle |
|---|---|---|
| **Formato de configuração** | XML (`pom.xml`) | Groovy ou Kotlin DSL (`build.gradle`) |
| **Curva de aprendizado** | Menor | Maior |
| **Legibilidade** | Verboso, mas previsível | Mais conciso |
| **Performance** | Mais lento em projetos grandes | Mais rápido (cache incremental) |
| **Convenção** | Forte — segue opiniões do Maven | Flexível — mais customizável |
| **Mercado Spring Boot** | Dominante (70%+ dos projetos) | Crescendo (preferido no Android) |
| **Suporte Spring Boot** | Excelente | Excelente |
| **Documentação/tutoriais** | Abundante | Boa, mas menos que Maven |

---

## A mesma dependência em cada um

```xml
<!-- Maven — pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

```groovy
// Gradle — build.gradle (Groovy DSL)
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'org.postgresql:postgresql'
}
```

```kotlin
// Gradle — build.gradle.kts (Kotlin DSL)
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    runtimeOnly("org.postgresql:postgresql")
}
```

---

## Escopos de dependência

### Maven

| Escopo | Quando usar |
|---|---|
| `compile` (padrão) | Disponível em compilação, teste e runtime |
| `runtime` | Só em runtime (drivers de banco) |
| `test` | Só nos testes |
| `provided` | Fornecido pelo servidor (não inclui no JAR) |
| `optional` | Opcional — não é transitivo |

### Gradle

| Configuração | Equivalente Maven |
|---|---|
| `implementation` | `compile` |
| `runtimeOnly` | `runtime` |
| `testImplementation` | `test` |
| `compileOnly` | `provided` |
| `annotationProcessor` | Para Lombok, MapStruct |

---

## Estrutura do pom.xml — completo e comentado

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>

    <!-- Herda do Spring Boot Parent — gerencia versões automaticamente -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!-- Identidade do projeto -->
    <groupId>com.empresa</groupId>
    <artifactId>minha-api</artifactId>
    <version>1.0.0</version>

    <!-- Propriedades globais -->
    <properties>
        <java.version>21</java.version>
        <!-- Versão do MapStruct, se usar -->
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>

    <dependencies>
        <!-- ===== CORE ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- Sem <version> — herdada do parent -->
        </dependency>

        <!-- ===== BANCO DE DADOS ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Migrations de banco -->
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>

        <!-- ===== VALIDAÇÃO ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- ===== SEGURANÇA ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>

        <!-- ===== UTILITÁRIOS ===== -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Swagger UI -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.3.0</version>
        </dependency>

        <!-- ===== DESENVOLVIMENTO ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- ===== TESTES ===== -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Qual escolher?

**Use Maven** se você:
- Está iniciando e quer foco em Spring Boot (menos fricção)
- A empresa usa Maven (verifique antes)
- Quer mais tutoriais e exemplos disponíveis

**Use Gradle** se você:
- Tem projeto grande e quer build mais rápido
- Já usa Gradle em outros projetos
- Trabalha com Android (onde Gradle é padrão)

> Para o seu contexto de aprendizado e primeiros projetos no mercado: **Maven**. É o que a maioria das vagas de Java/Spring usa e tem mais material de apoio.

---

## Próximas notas
- [[09 - application.properties e application.yml]] — configurando a aplicação
