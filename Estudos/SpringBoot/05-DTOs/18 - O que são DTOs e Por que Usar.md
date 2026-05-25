# 18 — O que são DTOs e Por que Usar

tags: #springboot #dto #arquitetura #boaspraticas
links: [[19 - Request vs Response DTO]] | [[20 - Mapeamento Manual e MapStruct]] | [[12 - Controller Service Repository Domain DTO]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Definição

**DTO (Data Transfer Object)** é um objeto simples usado para transportar dados entre camadas ou entre sistemas. Ele não tem lógica de negócio — só carrega dados.

> "Um DTO é um envelope. Você coloca os dados que quer enviar dentro dele, envia, e o destinatário abre e usa."

---

## O problema que o DTO resolve

Sem DTOs, você expõe diretamente a entidade JPA na API:

```java
// ❌ SEM DTO — expondo entidade diretamente
@GetMapping("/{id}")
public ResponseEntity<Usuario> buscar(@PathVariable Long id) {
    return ResponseEntity.ok(usuarioRepository.findById(id).orElseThrow());
}

// JSON retornado: TODO o objeto Usuario, incluindo:
// {
//   "id": 1,
//   "nome": "Felipe",
//   "email": "felipe@ex.com",
//   "senhaHash": "$2a$10$abc123...",    ← SENHA EXPOSTA! CRÍTICO
//   "role": "ADMIN",
//   "tokenReset": "xyz789",             ← token interno exposto
//   "pedidos": [...],                   ← pode causar LazyInitializationException
//   "criadoEm": "...",
//   "atualizadoEm": "...",
//   "deletadoEm": null,                 ← campo interno exposto
//   "versao": 3                         ← campo de controle interno
// }
```

**Problemas concretos:**

| Problema | Consequência |
|---|---|
| Expõe `senhaHash` | Falha grave de segurança |
| Expõe campos internos | Vaza lógica interna da aplicação |
| Acoplamento API ↔ banco | Mudar o banco quebra os clientes da API |
| `LazyInitializationException` | Relacionamentos lazy fora da transação |
| Ciclos infinitos | `@OneToMany` + `@ManyToOne` geram JSON infinito |
| Sem controle do shape | Não dá pra ter formatos diferentes por endpoint |

---

## A solução com DTOs

```java
// ✅ COM DTO — controle total do que é exposto
public record UsuarioResponse(
    Long id,
    String nome,
    String email
    // senha? não! role? depende. campos internos? nunca.
) {
    public static UsuarioResponse from(Usuario usuario) {
        return new UsuarioResponse(
            usuario.getId(),
            usuario.getNome(),
            usuario.getEmail()
        );
    }
}

@GetMapping("/{id}")
public ResponseEntity<UsuarioResponse> buscar(@PathVariable Long id) {
    Usuario usuario = service.buscarPorId(id);
    return ResponseEntity.ok(UsuarioResponse.from(usuario)); // só o que importa
}
```

---

## Os 5 benefícios dos DTOs

### 1. Segurança — controle do que é exposto
```java
// Entidade tem 20 campos. DTO expõe só 5.
// Nenhum campo interno ou sensível vaza para o cliente.
```

### 2. Flexibilidade — shapes diferentes por contexto
```java
// Mesmo recurso, representações diferentes:
ProdutoResumoResponse     // para listagens: { id, nome, preco }
ProdutoDetalheResponse    // para tela de detalhe: todos os campos
ProdutoAdminResponse      // para admin: inclui custo, margem, estoque
```

### 3. Desacoplamento — banco pode mudar sem quebrar a API
```java
// Renomeou coluna "nome_completo" para "nome" no banco?
// Atualiza a entidade e o mapeamento no DTO — a API continua igual.
// Os clientes (mobile, frontend) não percebem.
```

### 4. Validação separada da entidade
```java
// DTO de entrada: valida o que o CLIENTE manda
public record ClienteRequest(
    @NotBlank String nome,
    @Email String email
) {}

// Entidade: define as restrições do BANCO
@Entity
public class Cliente {
    @Column(nullable = false)
    private String nome;
}
```

### 5. Evita problemas de serialização JPA
```java
// Sem DTO, Jackson tenta serializar toda a entidade,
// incluindo relacionamentos lazy não carregados → exceção.
// Com DTO, você só inclui o que já foi carregado.
```

---

## Records Java — a forma moderna de DTOs

Desde Java 16 (disponível desde Java 14 como preview), `record` é a forma ideal para DTOs:

```java
// Antes (Java <16): classe com muito boilerplate
public class ClienteResponse {
    private Long id;
    private String nome;
    private String email;

    public ClienteResponse(Long id, String nome, String email) {
        this.id = id;
        this.nome = nome;
        this.email = email;
    }

    public Long getId() { return id; }
    public String getNome() { return nome; }
    public String getEmail() { return email; }

    @Override
    public boolean equals(Object o) { ... }
    @Override
    public int hashCode() { ... }
    @Override
    public String toString() { ... }
}

// Depois (Java 16+): record gera tudo automaticamente
public record ClienteResponse(Long id, String nome, String email) {
    // construtor, getters (id(), nome(), email()), equals, hashCode, toString
    // tudo gerado pelo compilador
}
```

**Records são imutáveis** — perfeito para DTOs que só transportam dados. Uma vez criado, não muda.

```java
// Acessando campos de um record:
ClienteResponse response = new ClienteResponse(1L, "Felipe", "felipe@ex.com");
response.id();      // 1L  (não getId() — records usam nome do campo diretamente)
response.nome();    // "Felipe"
response.email();   // "felipe@ex.com"
```

---

## Lombok como alternativa (Java <16 ou preferência)

```java
// Com Lombok — elimina boilerplate em classes normais
@Data           // gera getters, setters, equals, hashCode, toString
@Builder        // gera builder pattern
@NoArgsConstructor
@AllArgsConstructor
public class ClienteResponse {
    private Long id;
    private String nome;
    private String email;
}

// Usando o builder:
ClienteResponse response = ClienteResponse.builder()
    .id(1L)
    .nome("Felipe")
    .email("felipe@ex.com")
    .build();

// Para DTOs de leitura (imutável), prefira:
@Value  // equivalente a @Data mas imutável — todos os campos são final
@Builder
public class ClienteResponse {
    Long id;
    String nome;
    String email;
}
```

> 💡 **Recomendação:** use `record` para DTOs em projetos com Java 17+. Use Lombok `@Data/@Value` se o time já usa Lombok extensivamente ou se precisa de mais flexibilidade.

---

## Quando NÃO usar DTO

Há situações onde o overhead de DTO não compensa:

```java
// ✅ Pode retornar tipos simples diretamente
@GetMapping("/contagem")
public ResponseEntity<Long> contar() {
    return ResponseEntity.ok(service.contar());
}

@GetMapping("/{id}/existe")
public ResponseEntity<Boolean> existe(@PathVariable Long id) {
    return ResponseEntity.ok(service.existe(id));
}

// ✅ Pode usar Map para respostas simples e pontuais
@GetMapping("/status")
public ResponseEntity<Map<String, String>> status() {
    return ResponseEntity.ok(Map.of("status", "ok", "versao", "1.0.0"));
}
```

Mas para **entidades de negócio** (Cliente, Produto, Pedido), sempre use DTO.

---

## Próximas notas
- [[19 - Request vs Response DTO]] — os dois tipos de DTO em detalhe
- [[20 - Mapeamento Manual e MapStruct]] — como converter entidade ↔ DTO
