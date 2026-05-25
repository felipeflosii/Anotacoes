---
tags: [java, ano-1, java-core, sintaxe]
created: 2026-03-24
trimestre: T1
meses: 1-3
horas_semana: 10-15h
status: pendente
---

# T1 · Java Core e Sintaxe Moderna
### Meses 1–3 · Ano 1

> **Objetivo:** Dominar a linguagem Java em sua versão moderna (Java 21 LTS). Entender o modelo de execução, tipagem, estruturas de controle e as features mais recentes usadas no mercado.

---

## 🔵 Bloco 1 — Setup e Modelo de Execução

> [!tip] Por que isso importa?
> Entender como o Java funciona por baixo (JVM, bytecode, classloader) diferencia candidatos em entrevistas e explica comportamentos inesperados em produção.

### Tópicos

- **JDK vs JRE vs JVM** — diferenças e o que cada um faz
- **Compilação e execução** — `javac`, `.class`, bytecode, classloading
- **SDKMAN** — gerenciamento de versões Java (use sempre)
- **IntelliJ IDEA** — configuração, atalhos essenciais, plugins úteis
  - Plugin: SonarLint, Lombok, Rainbow Brackets, GitToolBox
- **Estrutura de um projeto Java** — `src/main/java`, pacotes, `pom.xml` / `build.gradle`
- **Maven vs Gradle** — diferenças, quando usar cada um; dominar os dois
  - Maven: `mvn clean install`, `mvn dependency:tree`, ciclo de vida
  - Gradle: `build.gradle`, tasks, Kotlin DSL vs Groovy DSL

### Prática

```
[ ] Instalar Java 21 via SDKMAN
[ ] Criar projeto Maven e projeto Gradle do zero
[ ] Entender o output de "javap -c HelloWorld.class"
[ ] Configurar IntelliJ com SonarLint ativo
```

---

## 🔵 Bloco 2 — Sintaxe e Tipos

### Tipos Primitivos e Wrappers

- `int`, `long`, `double`, `boolean`, `char`, `byte`, `short`, `float`
- Autoboxing / unboxing — armadilhas de performance e NPE
- `BigDecimal` para valores monetários — **nunca use `double` para dinheiro**
- `String` — imutabilidade, String Pool, `StringBuilder`, `StringJoiner`
- `var` (local type inference, Java 10+) — quando usar e quando NÃO usar

### Estruturas de Controle

- `if/else`, `switch` clássico
- **Switch Expressions** (Java 14+) — `yield`, múltiplos labels
- **Pattern Matching for instanceof** (Java 16+) — `if (obj instanceof String s)`
- **Sealed Classes + Pattern Matching** (Java 17+/21+)
- `for`, `while`, `do-while`, enhanced `for`
- `break`, `continue`, labels (evite labels — code smell)

### Arrays e Varargs

- Arrays multidimensionais
- `Arrays.sort()`, `Arrays.copyOf()`, `Arrays.asList()`
- Varargs (`String... args`) — uso correto

---

## 🔵 Bloco 3 — Java Moderno (Features do Java 8 ao 21)

> [!important] Essencial para qualquer entrevista sênior
> Saber o que mudou em cada versão demonstra senioridade. Foque em Java 8, 11, 17 e 21 (todos LTS).

### Java 8 (ainda amplamente usado em legado)

- **Lambdas** — sintaxe, functional interfaces (`Function`, `Predicate`, `Consumer`, `Supplier`, `BiFunction`, etc.)
- **Method References** — `Classe::método`, `objeto::método`, `Classe::new`
- **Streams API** — `map`, `filter`, `reduce`, `collect`, `flatMap`, `distinct`, `sorted`, `limit`, `peek`, `findFirst`, `anyMatch`, `allMatch`, `groupingBy`, `partitioningBy`
- **Optional** — evitar NPE, uso correto (`orElse` vs `orElseGet` vs `orElseThrow`)
- **Date/Time API** — `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Duration`, `Period`, `DateTimeFormatter`
- **Default/Static Methods em Interfaces**
- **CompletableFuture** — programação assíncrona básica

### Java 11 (LTS — muito usado em produção)

- `String`: `isBlank()`, `strip()`, `repeat()`, `lines()`
- `Files.readString()`, `Files.writeString()`
- `var` em lambdas
- HTTP Client API (`HttpClient`) — substitui Apache HttpClient em muitos casos

### Java 17 (LTS — padrão atual em muitas empresas)

- **Records** — `record Point(int x, int y) {}` — substitui POJOs simples
- **Sealed Classes/Interfaces** — `sealed interface Shape permits Circle, Rectangle`
- **Text Blocks** — strings multiline com `"""`
- **Pattern Matching for switch** (preview)

### Java 21 (LTS — adote agora)

- **Virtual Threads (Project Loom)** — revolução em concorrência; `Thread.ofVirtual().start()`
- **Sequenced Collections** — `SequencedCollection`, `SequencedMap`
- **Record Patterns** — desestruturação em pattern matching
- **Pattern Matching for switch** (finalizado)
- **Structured Concurrency** (preview) — `StructuredTaskScope`

---

## 🔵 Bloco 4 — Coleções e Generics

### Java Collections Framework

| Interface | Implementações principais | Use quando |
|-----------|--------------------------|------------|
| `List` | `ArrayList`, `LinkedList` | Ordem importa, duplicatas OK |
| `Set` | `HashSet`, `LinkedHashSet`, `TreeSet` | Sem duplicatas |
| `Map` | `HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap` | Chave-valor |
| `Queue` | `ArrayDeque`, `PriorityQueue`, `LinkedList` | FIFO/LIFO/prioridade |
| `Deque` | `ArrayDeque` | Stack ou Queue dupla |

### Pontos críticos de entrevista

- `HashMap` vs `LinkedHashMap` vs `TreeMap` — diferenças de ordenação e performance
- `equals()` e `hashCode()` — contrato obrigatório; como implementar corretamente
- `Comparable` vs `Comparator`
- `Collections.unmodifiableList()` vs `List.of()` — imutabilidade real vs wrapper
- **CopyOnWriteArrayList** e **ConcurrentHashMap** — thread-safety básico

### Generics

- Bounded wildcards: `<? extends T>` (covariant — só leitura) vs `<? super T>` (contravariant — só escrita) — regra PECS
- Type erasure — por que `List<String>` e `List<Integer>` são iguais em runtime
- Generic methods

---

## 🔵 Bloco 5 — Exceções e Tratamento de Erros

- **Checked vs Unchecked** — quando usar cada tipo
- `try-with-resources` — `AutoCloseable`, fechar recursos corretamente
- Hierarquia: `Throwable` → `Error` / `Exception` → `RuntimeException`
- Criando exceções customizadas de negócio
- **Boas práticas:**
  - Nunca engolir exceções (`catch (Exception e) {}` — proibido)
  - Logar com contexto suficiente
  - Não usar exceções para controle de fluxo
  - Prefer `Optional` a retornar `null`

---

## 🔵 Bloco 6 — I/O e Serialização

- `java.nio.file` — `Path`, `Files`, `Paths` — **use NIO2, não `java.io.File`**
- Leitura/escrita de arquivos, listagem de diretórios
- Serialização Java (básico e por que evitá-la — prefira JSON/Protobuf)
- `ObjectMapper` do Jackson — introdução

---

## 📖 Recursos

### Livros
- **"Effective Java" — Joshua Bloch** ⭐⭐⭐⭐⭐ — OBRIGATÓRIO
- "Java: The Complete Reference" — Herbert Schildt (referência)
- "Modern Java in Action" — Urma, Fusco, Mycroft (Java 8–17)

### Cursos Online
| Curso | Plataforma | Custo |
|-------|-----------|-------|
| Java COMPLETO — Nélio Alves | Udemy | 💰 (R$30–60 promoção) |
| Java Programming Masterclass — Tim Buchalka | Udemy | 💰 |
| Java Brains (YouTube) | YouTube | 🆓 |
| Loiane.training — Java | loiane.training | 🆓 |

### Documentação
- [JEPs por versão](https://openjdk.org/jeps/0) — acompanhe as features oficiais
- [Baeldung.com](https://baeldung.com) — artigos práticos de alta qualidade
- [Dev.java](https://dev.java) — portal oficial Oracle

---

## 🧪 Projeto Prático do Trimestre

> [!example] Mini-projeto: Sistema de Biblioteca CLI
> - Cadastro de livros e usuários com `ArrayList` e `HashMap`
> - Leitura/gravação em arquivo JSON com Jackson
> - Menu interativo no console
> - Uso de Streams para filtragem e ordenação
> - Exceções customizadas para erros de negócio
> - Build com Maven, código no GitHub

---

## 🔗 Próximo Trimestre

→ [[T2 — OOP, SOLID e Design Patterns]]
