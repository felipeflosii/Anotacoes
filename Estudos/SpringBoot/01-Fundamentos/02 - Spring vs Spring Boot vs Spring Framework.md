# 02 — Spring vs Spring Boot vs Spring Framework

tags: #springboot #fundamentos #java
links: [[01 - O que é Spring Boot]] | [[03 - IoC e Injeção de Dependência]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## A confusão comum

Quando alguém diz "eu uso Spring", pode estar falando de três coisas diferentes. Entender a distinção é fundamental para não se perder na documentação e nas conversas técnicas.

```
Spring (marca / ecossistema)
│
├── Spring Framework       ← O núcleo. Existe desde 2003.
│   ├── Core Container (IoC, DI)
│   ├── Spring MVC
│   ├── Spring Data
│   ├── Spring Security
│   └── ... (dezenas de módulos)
│
└── Spring Boot            ← Camada de automação. Existe desde 2014.
    ├── Auto-configuration
    ├── Starters
    ├── Embedded server
    └── Usa o Spring Framework por baixo
```

---

## Comparação direta

| | Spring Framework | Spring Boot |
|---|---|---|
| **O que é** | Conjunto de módulos e bibliotecas | Ferramenta de bootstrap e auto-configuração |
| **Objetivo** | Prover infraestrutura para aplicações Java | Eliminar configuração boilerplate |
| **Configuração** | Manual (XML ou Java) | Automática com defaults inteligentes |
| **Servidor web** | Externo (deploy de WAR) | Embutido (JAR executável) |
| **Curva de aprendizado** | Alta — muito para configurar | Menor — foca na lógica de negócio |
| **Lançamento** | 2003 | 2014 |
| **Usa o outro?** | — | Sim, usa o Spring Framework |

---

## Analogia para fixar

Pense assim:

- **Spring Framework** = motor de um carro. Potente, mas você monta tudo do zero.
- **Spring Boot** = carro completo e pronto para dirigir, com o mesmo motor.

Você pode aprender a montar o motor (Spring Framework puro), mas na prática do mercado, você dirige o carro (Spring Boot).

---

## O ecossistema Spring completo

O Spring vai além do Framework e do Boot. É um ecossistema inteiro de projetos:

| Projeto | Para que serve |
|---|---|
| **Spring Framework** | Núcleo: IoC, MVC, JDBC, etc. |
| **Spring Boot** | Auto-configuração e bootstrap |
| **Spring Data** | Acesso a dados (JPA, MongoDB, Redis...) |
| **Spring Security** | Autenticação e autorização |
| **Spring Cloud** | Microsserviços, service discovery, circuit breaker |
| **Spring Batch** | Processamento em lote |
| **Spring WebFlux** | Programação reativa |
| **Spring Integration** | Integração com sistemas externos |

> 💡 **No dia a dia:** você usa Spring Boot + Spring Data JPA + Spring Security. Os outros entram conforme a necessidade do projeto.

---

## Por que o Spring Boot não substitui o Spring Framework

Spring Boot **não é** uma versão "simplificada" do Spring Framework. Ele é uma camada em cima do Framework.

Ao usar Spring Boot, você está usando 100% do Spring Framework. A diferença é que o Boot configura tudo automaticamente para você.

```java
// COM Spring Framework puro (configuração manual):
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Bean
    public ViewResolver viewResolver() { ... }
    @Bean
    public MessageSource messageSource() { ... }
    // mais 20 beans de configuração...
}

// COM Spring Boot (auto-configuração):
// Não precisa escrever nada. O Boot detecta suas dependências
// e configura tudo automaticamente.
// Se precisar sobrescrever algum comportamento:
@Configuration
public class MeuConfig {
    @Bean
    public MimeMessage minhaConfiguracaoCustomizada() {
        // sobrescreve apenas o que precisa
    }
}
```

---

## Jakarta EE vs Spring — contexto histórico

É comum ver referências a "Java EE" ou "Jakarta EE" quando se estuda Java para backend. Vale entender a diferença:

| | Spring Framework | Jakarta EE (ex-Java EE) |
|---|---|---|
| **Mantido por** | VMware / comunidade | Eclipse Foundation |
| **Modelo** | Injeção de dependência com anotações próprias | Especificações padronizadas |
| **Adoção no mercado** | Dominante em novos projetos | Legado e servidores de aplicação |
| **Servidor** | Embutido (Tomcat, Jetty) | Externo (WildFly, GlassFish) |

O Spring usa **partes** do Jakarta EE — por exemplo, as anotações JPA (`@Entity`, `@Id`) são especificações Jakarta. O Hibernate implementa essas especificações.

> No Spring Boot 3.x, a migração de `javax.*` para `jakarta.*` foi consolidada. Se você ver código antigo com `import javax.persistence.*`, é Spring Boot 2.x ou inferior.

---

## Resumo para levar

```
Quando alguém pede "Spring" no currículo de vaga → quer Spring Boot
Quando a doc fala de "módulos Spring" → fala do Spring Framework
Quando você cria um projeto novo → use Spring Boot sempre
Quando precisa de segurança → Spring Security (parte do ecossistema)
Quando precisa de acesso a dados → Spring Data JPA (parte do ecossistema)
```

---

## Próximas notas
- [[03 - IoC e Injeção de Dependência]] — o conceito mais importante do Spring
- [[06 - Criando Projeto com Spring Initializr]] — mão na massa
