---
tags: [java, ano-1, oop, solid, design-patterns]
created: 2026-03-24
trimestre: T2
meses: 4-6
status: pendente
---

# T2 · OOP, SOLID e Design Patterns
### Meses 4–6 · Ano 1

> **Objetivo:** Dominar orientação a objetos de verdade (não só herança/encapsulamento superficial), aplicar SOLID no código do dia a dia e conhecer os Design Patterns mais cobrados em entrevistas e mais usados no Spring.

---

## 🔵 Bloco 1 — OOP Profundo

> [!warning] Armadilha comum
> A maioria dos devs iniciantes acha que sabe OOP depois de aprender herança. OOP real é sobre **modelagem de domínio**, **delegação** e **composição sobre herança**.

### Os 4 Pilares (revisão crítica)

#### Encapsulamento
- Não é só `private` + getter/setter — é esconder **decisões de implementação**
- Getters e setters indiscriminados violam encapsulamento
- **Tell, Don't Ask** — o objeto deve agir sobre seus dados, não expô-los
- Imutabilidade como ferramenta de encapsulamento — `record`, `final`, `Collections.unmodifiableList()`

#### Herança
- **Herança de implementação vs herança de interface** — prefira sempre interface
- Fragile Base Class Problem — por que herança profunda é perigosa
- `final class` para impedir herança desnecessária
- Quando herança faz sentido (is-a real)
- `abstract class` vs `interface` — decisão arquitetural, não técnica

#### Polimorfismo
- Polimorfismo de subtipo (herança/interface) — o mais comum
- Polimorfismo paramétrico (Generics)
- Sobrecarga (`overloading`) — polimorfismo de compilação — armadilhas
- **Dispatch dinâmico** — como a JVM escolhe o método em runtime
- `instanceof` e casts — quando (raramente) usá-los

#### Abstração
- Programar para interfaces, não para implementações
- Separar **o que** do **como**
- Interfaces como contratos de domínio

### Composição vs Herança

```java
// ❌ Herança para reuso (errado)
class Stack extends ArrayList { ... }

// ✅ Composição (correto)
class Stack<T> {
    private final Deque<T> storage = new ArrayDeque<>();
    public void push(T item) { storage.push(item); }
    public T pop() { return storage.pop(); }
}
```

---

## 🔵 Bloco 2 — SOLID na Prática

> [!tip] SOLID em entrevistas
> Não basta decorar os nomes. Saiba identificar violações em código real e refatorar para corrigir.

### S — Single Responsibility Principle

- Uma classe, uma razão para mudar
- Como identificar responsabilidades misturadas
- **Refatoração:** extrair classes de serviço, repositório, transformador (mapper)

```java
// ❌ Violação: OrderService faz tudo
class OrderService {
    void createOrder(Order o) { /* valida + persiste + envia email + gera PDF */ }
}

// ✅ Correto: responsabilidades separadas
class OrderService { void createOrder(Order o) { /* orquestra */ } }
class OrderValidator { void validate(Order o) { ... } }
class OrderRepository { void save(Order o) { ... } }
class OrderNotificationService { void notify(Order o) { ... } }
```

### O — Open/Closed Principle

- Aberto para extensão, fechado para modificação
- Use interfaces + polimorfismo ao invés de `if/else` ou `switch` com tipos
- Strategy Pattern é a implementação clássica de OCP

### L — Liskov Substitution Principle

- Subtipos devem ser substituíveis pelos seus tipos base
- Violação clássica: `Rectangle` → `Square` (pré-condições mais fortes)
- Como detectar: teste se o código do cliente precisa saber o tipo concreto

### I — Interface Segregation Principle

- Clientes não devem depender de métodos que não usam
- Interfaces "gordas" → problemas de coesão
- Spring usa muito isso: `ApplicationEventPublisher`, `ResourceLoader`, etc.

### D — Dependency Inversion Principle

- Módulos de alto nível não devem depender de módulos de baixo nível
- Dependa de abstrações (interfaces), não de concretos
- **Injeção de Dependência** (DI) é a aplicação prática do DIP
- Spring IoC Container — fundamento do Spring

---

## 🔵 Bloco 3 — Design Patterns (os que o mercado usa de verdade)

> [!note] Foco no mercado
> Não tente decorar todos os 23 patterns do GoF. Foque nos que aparecem constantemente no Spring, em entrevistas e em código enterprise.

### Creational Patterns

#### Singleton
- Implementação correta com double-checked locking ou enum
- Por que o Spring gerencia Singletons para você (beans)
- Problemas com Singleton em testes

#### Factory Method e Abstract Factory
- `BeanFactory` e `ApplicationContext` do Spring são Abstract Factories
- `DriverManager.getConnection()` no JDBC
- Quando criar suas próprias factories

#### Builder
- `StringBuilder`, `Stream.Builder`, Lombok `@Builder`
- Criação de objetos complexos imutáveis
- **Muito usado:** `HttpRequest.newBuilder()`, configuração de clientes HTTP

#### Prototype
- `Object.clone()` (evitar) vs cópia manual vs serialização
- Uso em templates de objetos

---

### Structural Patterns

#### Adapter
- Converter interface incompatível — muito comum em integrações
- Ex: adaptar biblioteca de pagamento externa para sua interface interna

#### Decorator
- Adicionar comportamento sem herança — alternativa a subclasses
- `BufferedReader` wrapping `FileReader` — exemplo clássico
- Spring AOP usa conceito similar

#### Facade
- Simplificar subsistemas complexos com interface unificada
- `JdbcTemplate` do Spring é uma Facade sobre JDBC puro
- Service layer como Facade do domínio

#### Proxy
- **Fundamental para o Spring** — AOP, `@Transactional`, Spring Security, caching
- Proxy dinâmico Java (`java.lang.reflect.Proxy`) vs CGLIB
- Entender proxies explica 80% dos bugs "mágicos" do Spring

#### Composite
- Estruturas em árvore — tratamento uniforme de nós e folhas
- Ex: sistema de menus, categorias hierárquicas

---

### Behavioral Patterns

#### Strategy
- Encapsular algoritmos intercambiáveis
- Ex: diferentes estratégias de desconto, diferentes meios de pagamento
- **Implementação com Spring:** beans com `@Component` + `@Qualifier`

#### Observer / Event-Driven
- `ApplicationEventPublisher` do Spring
- `@EventListener`, `@TransactionalEventListener`
- Base para arquiteturas event-driven

#### Template Method
- `JdbcTemplate`, `RestTemplate` — o Spring ama este pattern
- Definir esqueleto do algoritmo, deixar detalhes para subclasses/lambdas

#### Chain of Responsibility
- Filtros do Spring Security, Servlet Filters, HandlerInterceptors
- Pipeline de validação

#### Command
- Encapsular ação como objeto — undo/redo, filas de tarefas
- Spring Batch usa este conceito

#### State
- Máquinas de estado — pedidos (CRIADO → PAGO → ENVIADO → ENTREGUE)
- Spring State Machine (biblioteca dedicada)

---

## 🔵 Bloco 4 — Outros Princípios Essenciais

### DRY, KISS, YAGNI

- **DRY** (Don't Repeat Yourself) — mas cuidado com abstração prematura
- **KISS** (Keep It Simple, Stupid) — código simples é código profissional
- **YAGNI** (You Aren't Gonna Need It) — não construa o que não precisa agora

### Law of Demeter (Principle of Least Knowledge)

```java
// ❌ Violação: "train wreck"
customer.getAddress().getCity().getName()

// ✅ Correto: delegar ao objeto
customer.getCityName()
```

### Coesão e Acoplamento

- Alta coesão + baixo acoplamento = boa arquitetura
- Como medir: fan-in vs fan-out de dependências

---

## 📖 Recursos

| Recurso | Tipo | Nota |
|---------|------|------|
| **"Effective Java" — Bloch** (caps. sobre OOP) | Livro | ⭐⭐⭐⭐⭐ |
| **"Head First Design Patterns"** | Livro | Mais didático |
| **"Design Patterns" — GoF** | Livro | Referência clássica |
| **"Clean Code" — Robert Martin** | Livro | Obrigatório |
| Refactoring Guru (refactoring.guru) | Site | 🆓 Excelente visual |
| SourceMaking (sourcemaking.com) | Site | 🆓 |

---

## 🧪 Projeto Prático do Trimestre

> [!example] Mini-projeto: E-commerce CLI Refatorado
> - Modelar domínio: Produto, Pedido, Carrinho, Cliente
> - Aplicar SOLID em todos os serviços
> - Usar Strategy para cálculo de frete e desconto
> - Observer para eventos de pedido (email, estoque, nota fiscal)
> - Builder para criação de Pedido
> - Repository pattern para persistência (em memória por ora)
> - 100% das classes com responsabilidade única verificada

---

## 🔗 Navegação

← [[T1 — Java Core e Sintaxe Moderna]]  
→ [[T3 — SQL, JDBC e JPA-Hibernate]]
