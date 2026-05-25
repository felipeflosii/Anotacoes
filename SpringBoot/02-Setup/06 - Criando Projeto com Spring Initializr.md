# 06 — Criando Projeto com Spring Initializr

tags: #springboot #setup #initializr
links: [[07 - Estrutura de Projeto Spring Boot]] | [[08 - Maven vs Gradle]] | [[🗺️ Mapa Principal]]

---

## O que é o Spring Initializr

O **Spring Initializr** (`start.spring.io`) é o gerador oficial de projetos Spring Boot. Ele cria a estrutura completa do projeto com as dependências escolhidas, já configuradas e com versões compatíveis.

Disponível como:
- Site web: `https://start.spring.io`
- Plugin do IntelliJ IDEA (File → New → Spring Initializr)
- Plugin do VS Code (Spring Boot Extension Pack)
- CLI: `spring init`

---

## Gerando o projeto — passo a passo no site

Acesse `https://start.spring.io` e configure:

### 1. Configurações básicas

```
Project:  Maven  (padrão de mercado — use Maven)
Language: Java
Spring Boot: 3.x.x  (última versão estável — não use SNAPSHOT)

Project Metadata:
  Group:    com.empresa           (domínio reverso da sua empresa)
  Artifact: nome-do-projeto       (nome do .jar gerado)
  Name:     Nome Do Projeto       (nome exibido)
  Description: Descrição breve
  Package name: com.empresa.nomeprojeto
  Packaging: Jar                  (não War — servidor embutido)
  Java: 21                        (LTS mais recente)
```

### 2. Dependências para uma API REST básica

Clique em "Add Dependencies" e adicione:

| Dependência | Para que serve |
|---|---|
| **Spring Web** | Servidor web embutido + Spring MVC + REST |
| **Spring Data JPA** | Integração com banco via JPA/Hibernate |
| **PostgreSQL Driver** | Driver do banco (ou H2 para desenvolvimento) |
| **Spring Boot DevTools** | Hot reload durante desenvolvimento |
| **Lombok** | Elimina boilerplate (getters, setters, construtores) |
| **Validation** | Bean Validation (@NotNull, @NotBlank, etc.) |
| **Spring Security** | Segurança (adicione quando for implementar auth) |

### 3. Gerando e importando

1. Clique em **Generate** — baixa um `.zip`
2. Extraia em uma pasta de projetos
3. Abra no IntelliJ IDEA (ou VS Code): **File → Open** na pasta extraída
4. Aguarde o download das dependências pelo Maven

---

## Gerando via IntelliJ IDEA (recomendado)

1. `File → New → Project`
2. Selecione `Spring Boot` na barra lateral
3. Preencha os campos diretamente na IDE
4. Selecione as dependências
5. Clique em `Finish` — o projeto já abre pronto

---

## O pom.xml gerado — entendendo o arquivo

Para um projeto com Web + JPA + PostgreSQL + Lombok:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Parent: herda configurações do Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>

    <!-- Identificação do seu projeto -->
    <groupId>com.empresa</groupId>
    <artifactId>minha-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>minha-api</name>
    <description>API REST com Spring Boot</description>

    <!-- Versão do Java -->
    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <!-- Spring MVC + Tomcat embutido + Jackson (JSON) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- JPA + Hibernate -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Driver do PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Bean Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- Lombok — reduz boilerplate -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Hot reload durante desenvolvimento -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!-- Testes — JUnit 5 + Mockito já incluídos -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Plugin Spring Boot — cria o JAR executável -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- Remove Lombok do JAR final -->
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

## Rodando o projeto pela primeira vez

```bash
# Via terminal, na raiz do projeto:
./mvnw spring-boot:run

# Ou no IntelliJ:
# Botão verde Play na classe principal (que tem @SpringBootApplication)
# Ou: Run → Run 'NomeDoProjetoApplication'
```

Saída esperada:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.2.0)

INFO  Started MinhaApiApplication in 2.341 seconds
```

A aplicação está rodando em `http://localhost:8080`.

---

## Usando H2 (banco em memória) para desenvolvimento inicial

Se não quiser configurar PostgreSQL imediatamente, use H2 — um banco em memória que o Spring Boot configura automaticamente:

No `pom.xml`, substitua o driver PostgreSQL por:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

No `application.properties`:

```properties
# H2 não precisa de configuração — funciona por padrão
# Para acessar o console web do H2:
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:testdb
```

Acesse `http://localhost:8080/h2-console` para ver o banco em memória.

> ⚠️ H2 in-memory apaga todos os dados ao reiniciar. Use só para desenvolvimento e testes. Para produção, sempre use PostgreSQL ou outro banco real.

---

## Próximas notas
- [[07 - Estrutura de Projeto Spring Boot]] — entenda o que foi gerado
- [[08 - Maven vs Gradle]] — diferenças e quando usar cada um
- [[09 - application.properties e application.yml]] — configurando a aplicação
