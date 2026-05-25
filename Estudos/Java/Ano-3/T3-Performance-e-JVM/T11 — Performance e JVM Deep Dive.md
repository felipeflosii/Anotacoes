---
tags: [java, ano-3, jvm, performance, gc, profiling, memory]
trimestre: T11
meses: 34-36
---

# T11 · Performance e JVM Deep Dive
### Meses 34–36 · Ano 3

> **Objetivo:** Entender a JVM por dentro. Diagnosticar e resolver problemas de memória, GC e CPU. Diferencial de nível sênior/staff.

---

## 🔵 Bloco 1 — Arquitetura da JVM

```
┌──────────────────────────────────────────────────────────┐
│                     JVM Process                          │
│  ┌──────────────┐  ┌────────────────────────────────┐   │
│  │  Class Loader │  │         Runtime Data Areas      │   │
│  │  Bootstrap    │  │  ┌──────┐ ┌─────────────────┐ │   │
│  │  Extension    │  │  │Stack │ │      Heap        │ │   │
│  │  Application  │  │  │(por  │ │  ┌────────────┐ │ │   │
│  └──────────────┘  │  │thread│ │  │Young Gen   │ │ │   │
│                     │  │      │ │  │ Eden+S0+S1 │ │ │   │
│  ┌──────────────┐  │  └──────┘ │  ├────────────┤ │ │   │
│  │ Execution    │  │           │  │Old Gen     │ │ │   │
│  │ Engine       │  │  ┌──────┐ │  └────────────┘ │ │   │
│  │ - JIT (C1/C2)│  │  │ PC   │ └─────────────────┘ │   │
│  │ - GC         │  │  │Reg.  │ ┌──────────────────┐ │   │
│  │ - Native     │  │  └──────┘ │ Metaspace         │ │   │
│  └──────────────┘  └────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### Memory Areas

| Área | O que guarda | GC? | Tamanho |
|------|-------------|-----|---------|
| **Heap — Eden** | Novos objetos | Sim (Minor GC) | Configurável |
| **Heap — Survivors** | Objetos sobreviveram ao Eden | Sim | Configurável |
| **Heap — Old Gen** | Objetos de longa vida | Sim (Major/Full GC) | Configurável |
| **Metaspace** | Metadados de classes, bytecode | Sim (raramente) | Cresce dinamicamente |
| **Stack** | Frames de método, variáveis locais | Não | Por thread |
| **Code Cache** | Código JIT compilado | Não | `-XX:ReservedCodeCacheSize` |
| **Native Memory** | JVM internals, DirectByteBuffer | Não | Variável |

---

## 🔵 Bloco 2 — Garbage Collectors

### Algoritmos disponíveis

| GC | Flag | Melhor para | Característica |
|----|------|------------|----------------|
| **G1GC** | `-XX:+UseG1GC` | Aplicações gerais (padrão JVM 9+) | Previsível, baixa latência |
| **ZGC** | `-XX:+UseZGC` | Baixíssima latência (<1ms) | JVM 15+ produção, heap enorme |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Baixa latência, menor footprint | GraalVM, OpenJDK |
| **Serial** | `-XX:+UseSerialGC` | Apps pequenas, containers tiny | Single thread |
| **Parallel** | `-XX:+UseParallelGC` | Throughput máximo, batch | Pausas maiores |

### Configuração G1GC (o mais usado em produção)

```bash
java \
  -Xms2g -Xmx2g \                              # heap fixo (evita resize em prod)
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \                   # meta de pausa (não garantia)
  -XX:G1HeapRegionSize=16m \                   # tamanho da região
  -XX:G1NewSizePercent=20 \
  -XX:G1MaxNewSizePercent=40 \
  -XX:InitiatingHeapOccupancyPercent=45 \      # quando iniciar concurrent marking
  -XX:+G1UseAdaptiveIHOP \
  -XX:+ExplicitGCInvokesConcurrent \           # System.gc() não faz Full GC
  -XX:+ParallelRefProcEnabled \
  -XX:+AlwaysPreTouch \                        # alocar memória no startup
  -Xlog:gc*:file=gc.log:time,uptime:filecount=5,filesize=50m \
  -jar app.jar
```

### Configuração ZGC (para latência ultra-baixa)

```bash
java \
  -Xmx4g \
  -XX:+UseZGC \
  -XX:ZCollectionInterval=5 \    # forçar GC a cada 5 segundos
  -XX:ZUncommitDelay=300 \       # devolver memória ao OS após 5min idle
  -jar app.jar
```

---

## 🔵 Bloco 3 — Diagnóstico e Profiling

### JVM Flags para diagnóstico

```bash
# GC logging detalhado
-Xlog:gc*,safepoint:gc.log:time,uptime,pid,level,tags

# Heap dump automático em OOM
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/dumps/heap-$(date +%Y%m%d-%H%M%S).hprof

# JFR — Java Flight Recorder (built-in, baixíssimo overhead)
-XX:StartFlightRecording=duration=60s,filename=app.jfr,settings=profile
```

### async-profiler — o melhor profiler Java

```bash
# CPU profiling (flame graph)
./profiler.sh -d 30 -f profile.html $(pgrep java)

# Heap allocation profiling
./profiler.sh -e alloc -d 30 -f alloc.html $(pgrep java)

# Lock contention
./profiler.sh -e lock -d 30 -f locks.html $(pgrep java)

# Wall clock (inclui I/O wait)
./profiler.sh -e wall -d 30 -f wall.html $(pgrep java)
```

### JVM Tools

```bash
# Heap dump manual
jcmd <pid> GC.heap_dump /tmp/heapdump.hprof
jmap -dump:format=b,file=heap.hprof <pid>  # legado

# Thread dump
jcmd <pid> Thread.print
kill -3 <pid>  # envia SIGQUIT, JVM imprime thread dump no stdout

# JVM info
jcmd <pid> VM.flags                 # flags efetivas
jcmd <pid> VM.system_properties
jcmd <pid> GC.class_histogram       # classes por instâncias/memória
jcmd <pid> VM.native_memory         # native memory tracking

# jstat — estatísticas JVM em tempo real
jstat -gcutil <pid> 1000  # GC stats a cada 1 segundo
```

### Análise de Heap Dump

Ferramentas: **Eclipse MAT** (Memory Analyzer Tool), VisualVM, JProfiler

**Checklist de análise:**
1. **Dominator tree** — quais objetos retêm mais memória?
2. **Retained heap** — quanto de memória seria liberado se o objeto fosse coletado?
3. **GC roots** — por que o objeto não está sendo coletado?
4. **Leak suspects report** — MAT identifica automaticamente candidatos a leak

---

## 🔵 Bloco 4 — Problemas Comuns e Soluções

### Memory Leak — causas comuns em Java

```java
// 1. Static collections crescendo sem limpeza
class Cache {
    private static final Map<String, Object> CACHE = new HashMap<>(); // cresce para sempre
    // solução: use WeakHashMap, Caffeine, ou limite tamanho
}

// 2. ThreadLocal não removido
class RequestContext {
    private static final ThreadLocal<User> USER = new ThreadLocal<>();
    // solução: sempre chame USER.remove() no finally do filter
}

// 3. Listeners não registrados
class EventService {
    public void subscribe(EventListener listener) {
        listeners.add(listener);
        // listeners.remove() nunca é chamado → listener é mantido na memória
    }
}

// 4. Intern strings excessivo
// String.intern() em Java 6- → PermGen overflow
// Em Java 8+ → Metaspace, mas ainda pode ser excessivo
```

### CPU Spike — diagnóstico

```bash
# 1. Identificar thread com alto CPU
top -H -p <pid>  # -H mostra threads

# 2. Converter PID da thread para hex
printf "%x\n" <thread-pid>

# 3. Buscar no thread dump
jcmd <pid> Thread.print | grep -A 20 "<hex-id>"
```

### Thread Deadlock — detecção

```bash
jcmd <pid> Thread.print 2>&1 | grep -A 5 "deadlock"
# ou use VisualVM / JProfiler para visualizar
```

### OutOfMemoryError — tipos

| Mensagem | Causa | Solução |
|---------|-------|---------|
| `Java heap space` | Heap cheio | Aumentar heap, corrigir memory leak |
| `GC overhead limit exceeded` | GC passa >98% do tempo sem liberar >2% | Memory leak, heap muito pequeno |
| `Metaspace` | Muitas classes carregadas | Aumentar Metaspace, verificar class leaks |
| `Direct buffer memory` | DirectByteBuffer esgotado | Aumentar `-XX:MaxDirectMemorySize` |
| `Unable to create native thread` | Muitas threads | Reduzir threads, usar virtual threads |

---

## 🔵 Bloco 5 — Performance de Código Java

### Benchmarking com JMH

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 10, time = 1)
@Fork(2)
@State(Scope.Benchmark)
public class StringConcatBenchmark {

    @Benchmark
    public String stringPlus() {
        return "Hello" + " " + "World";  // compilador otimiza em compile time
    }

    @Benchmark
    public String stringBuilder() {
        return new StringBuilder().append("Hello").append(" ").append("World").toString();
    }

    @Benchmark
    public String stringFormat() {
        return String.format("Hello %s", "World");  // mais lento — não use em loops
    }
}
```

### Dicas de Performance

```java
// 1. ArrayList vs LinkedList — ArrayList quase sempre melhor
// LinkedList tem overhead de ponteiros, péssimo cache locality

// 2. HashMap sizing — evite resizes
new HashMap<>(expectedSize * 2);  // ou use Guava's Maps.newHashMapWithExpectedSize

// 3. Stream vs for loop — para coleções pequenas (<1000), for é mais rápido
// Para coleções grandes, Stream.parallel() pode ajudar (mas medir antes)

// 4. String concatenation em loops
// ❌
String result = "";
for (var item : list) result += item;  // O(n²)

// ✅
var sb = new StringBuilder(list.size() * 10);
for (var item : list) sb.append(item);  // O(n)

// 5. Evite boxing/unboxing em loops críticos
// ❌
List<Integer> nums = new ArrayList<>();
int sum = 0;
for (Integer n : nums) sum += n;  // unboxing a cada iteração

// ✅ Use primitivos quando possível
int[] nums = ...;
int sum = 0;
for (int n : nums) sum += n;

// 6. Caffeine para cache local de alta performance
Cache<Long, Produto> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(Duration.ofMinutes(10))
    .recordStats()
    .build();
```

---

## 🔗 Navegação

← [[T10 — Observabilidade e SRE]]  
→ [[T12 — Segurança em Java]]
