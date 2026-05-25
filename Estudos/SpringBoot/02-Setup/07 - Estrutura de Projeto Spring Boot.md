# 07 — Estrutura de Projeto Spring Boot

tags: #springboot #setup #estrutura
links: [[06 - Criando Projeto com Spring Initializr]] | [[08 - Maven vs Gradle]] | [[13 - Organização de Pacotes e Boas Práticas]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Estrutura gerada pelo Initializr

Após gerar e extrair o projeto, você verá:

```
minha-api/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── empresa/
│   │   │           └── minhaapi/
│   │   │               └── MinhaApiApplication.java  ← Classe principal
│   │   └── resources/
│   │       ├── application.properties                ← Configurações
│   │       ├── static/                               ← Arquivos estáticos (HTML, CSS, JS)
│   │       └── templates/                            ← Templates Thymeleaf (se usar)
│   └── test/
│       └── java/
│           └── com/
│               └── empresa/
│                   └── minhaapi/
│                       └── MinhaApiApplicationTests.java  ← Testes
├── pom.xml                                           ← Dependências Maven
├── mvnw                                              ← Maven Wrapper (Linux/Mac)
├── mvnw.cmd                                          ← Maven Wrapper (Windows)
└── .gitignore                                        ← Arquivos ignorados pelo Git
```

---

## Cada arquivo e pasta explicado

### `MinhaApiApplication.java` — o ponto de entrada

```java
package com.empresa.minhaapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
// Equivale a:
// @Configuration         — esta classe pode definir beans
// @EnableAutoConfiguration — ativa a auto-configuração do Spring Boot
// @ComponentScan         — escaneia o pacote atual e subpacotes por @Component
public class MinhaApiApplication {

    public static void main(String[] args) {
        // Inicia o contexto Spring, sobe o servidor Tomcat, registra todos os beans
        SpringApplication.run(MinhaApiApplication.class, args);
    }
}
```

> ⚠️ **Regra importante:** esta classe deve estar no **pacote raiz** do projeto. O `@ComponentScan` escaneia o pacote dela e todos os subpacotes. Se suas classes estiverem fora do pacote raiz, não serão detectadas.

### `src/main/resources/application.properties`

Configurações da aplicação. Veja [[09 - application.properties e application.yml]] para detalhes.

### `src/main/resources/static/`

Arquivos servidos diretamente (HTML, CSS, JS, imagens). Para APIs REST puras, geralmente fica vazio.

### `src/main/resources/templates/`

Templates para renderização server-side com Thymeleaf. Para APIs REST puras, não é usado.

### `src/test/`

Espelha a estrutura do `src/main/`. Cada classe de teste fica no mesmo pacote da classe que testa.

### `pom.xml` / `build.gradle`

Define as dependências, versão do Java, plugins de build.

### `mvnw` e `mvnw.cmd`

Maven Wrapper — permite rodar Maven sem instalá-lo globalmente. Use sempre `./mvnw` em vez de `mvn` para garantir a versão correta.

---

## Estrutura recomendada para uma API REST real

A estrutura gerada é mínima. Para um projeto real, organize por funcionalidade:

```
src/main/java/com/empresa/minhaapi/
│
├── MinhaApiApplication.java
│
├── config/                          ← Configurações do Spring
│   ├── SecurityConfig.java
│   ├── CorsConfig.java
│   └── SwaggerConfig.java
│
├── cliente/                         ← Módulo Cliente (por domínio)
│   ├── Cliente.java                 ← Entidade JPA
│   ├── ClienteRepository.java       ← Interface de acesso a dados
│   ├── ClienteService.java          ← Regras de negócio
│   ├── ClienteController.java       ← Endpoints HTTP
│   └── dto/
│       ├── ClienteRequest.java      ← Dados de entrada
│       └── ClienteResponse.java     ← Dados de saída
│
├── pedido/                          ← Módulo Pedido
│   ├── Pedido.java
│   ├── PedidoRepository.java
│   ├── PedidoService.java
│   ├── PedidoController.java
│   └── dto/
│       ├── PedidoRequest.java
│       └── PedidoResponse.java
│
├── auth/                            ← Módulo de Autenticação
│   ├── AuthController.java
│   ├── AuthService.java
│   └── dto/
│       ├── LoginRequest.java
│       └── TokenResponse.java
│
├── security/                        ← Infraestrutura de segurança
│   ├── JwtService.java
│   ├── JwtAuthFilter.java
│   └── UserDetailsServiceImpl.java
│
└── exception/                       ← Tratamento global de exceções
    ├── GlobalExceptionHandler.java
    ├── ResourceNotFoundException.java
    └── ErrorResponse.java
```

Esta organização por **módulo/domínio** é a mais usada no mercado. Cada pasta agrupa tudo relacionado a um conceito de negócio.

> 💡 **Alternativa: organização por camada** (controller/, service/, repository/). Funciona para projetos pequenos, mas escala mal — ao crescer, cada pasta fica gigante e difícil de navegar.

---

## O Maven Wrapper em detalhe

O `mvnw` garante que todos no time usem a **mesma versão** do Maven, sem necessidade de instalação.

```bash
# Compilar o projeto
./mvnw compile

# Rodar os testes
./mvnw test

# Gerar o JAR (ignorando testes)
./mvnw package -DskipTests

# Rodar a aplicação
./mvnw spring-boot:run

# Limpar o projeto (apaga /target)
./mvnw clean

# Limpar e gerar JAR
./mvnw clean package
```

O JAR gerado fica em `target/minha-api-0.0.1-SNAPSHOT.jar` e pode ser executado com:

```bash
java -jar target/minha-api-0.0.1-SNAPSHOT.jar
```

---

## Próximas notas
- [[08 - Maven vs Gradle]] — qual ferramenta de build usar
- [[09 - application.properties e application.yml]] — configurar a aplicação
- [[13 - Organização de Pacotes e Boas Práticas]] — estrutura avançada
