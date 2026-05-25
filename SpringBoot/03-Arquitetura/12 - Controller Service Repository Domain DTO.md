# 12 — Controller · Service · Repository · Domain · DTO

tags: #springboot #arquitetura #camadas #padrão
links: [[10 - Arquitetura em Camadas]] | [[11 - Clean Architecture no Spring Boot]] | [[14 - RestController e RequestMapping]] | [[🗺️ Mapa Principal]]

---

## Visão geral do fluxo completo

Veja um CRUD completo de `Cliente` passando por todas as camadas, com o código de cada uma:

```
POST /clientes
  │
  ▼
ClienteController          ← recebe HTTP, valida formato, converte DTO
  │ chama
  ▼
ClienteService             ← aplica regras de negócio
  │ usa
  ▼
ClienteRepository          ← acessa o banco via JPA
  │
  ▼
Banco de dados (PostgreSQL)
  │
  ▲ retorna Cliente (entidade)
  │
ClienteResponse.from(cliente) ← converte entidade para DTO de saída
  │
  ▲ retorna ao controller
  │
ResponseEntity.status(201).body(response) ← monta resposta HTTP
```

---

## Domain — a entidade

```java
package com.empresa.api.cliente;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "clientes")
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nome;

    @Column(nullable = false, unique = true, length = 200)
    private String email;

    @Column(nullable = false, length = 20)
    private String telefone;

    @Column(name = "ativo", nullable = false)
    private boolean ativo = true;

    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    // JPA precisa de construtor sem args — protected evita uso acidental
    protected Cliente() {}

    public Cliente(String nome, String email, String telefone) {
        this.nome = nome;
        this.email = email;
        this.telefone = telefone;
        this.criadoEm = LocalDateTime.now();
    }

    // Método de negócio na entidade — simples
    public void desativar() {
        this.ativo = false;
        this.atualizadoEm = LocalDateTime.now();
    }

    public void atualizar(String nome, String telefone) {
        this.nome = nome;
        this.telefone = telefone;
        this.atualizadoEm = LocalDateTime.now();
    }

    // Getters — sem setters públicos (controle via métodos de negócio)
    public Long getId() { return id; }
    public String getNome() { return nome; }
    public String getEmail() { return email; }
    public String getTelefone() { return telefone; }
    public boolean isAtivo() { return ativo; }
    public LocalDateTime getCriadoEm() { return criadoEm; }
    public LocalDateTime getAtualizadoEm() { return atualizadoEm; }
}
```

---

## Repository — acesso a dados

```java
package com.empresa.api.cliente;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;
import java.util.Optional;

public interface ClienteRepository extends JpaRepository<Cliente, Long> {

    // Spring Data gera a query pelo nome do método
    boolean existsByEmail(String email);

    Optional<Cliente> findByEmail(String email);

    // Busca por nome (case insensitive, busca parcial)
    List<Cliente> findByNomeContainingIgnoreCaseAndAtivoTrue(String nome);

    // Query JPQL customizada
    @Query("SELECT c FROM Cliente c WHERE c.ativo = true ORDER BY c.nome")
    List<Cliente> findAllAtivos();

    // Query nativa SQL (quando JPQL não é suficiente)
    @Query(value = """
        SELECT * FROM clientes
        WHERE ativo = true
        AND criado_em >= :dataInicio
        """, nativeQuery = true)
    List<Cliente> findAtivosCriadosDepoisDe(@Param("dataInicio") LocalDateTime dataInicio);
}
```

---

## DTOs — transferência de dados

```java
package com.empresa.api.cliente.dto;

import jakarta.validation.constraints.*;

// DTO de entrada — dados que chegam do cliente HTTP
public record ClienteRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 150, message = "Nome deve ter entre 2 e 150 caracteres")
    String nome,

    @NotBlank(message = "E-mail é obrigatório")
    @Email(message = "E-mail inválido")
    String email,

    @NotBlank(message = "Telefone é obrigatório")
    @Pattern(regexp = "\\d{10,11}", message = "Telefone deve ter 10 ou 11 dígitos")
    String telefone
) {}
```

```java
package com.empresa.api.cliente.dto;

import com.empresa.api.cliente.Cliente;
import java.time.LocalDateTime;

// DTO de saída — dados retornados ao cliente HTTP
public record ClienteResponse(
    Long id,
    String nome,
    String email,
    String telefone,
    boolean ativo,
    LocalDateTime criadoEm
) {
    // Factory method estático — converte entidade para DTO
    public static ClienteResponse from(Cliente cliente) {
        return new ClienteResponse(
            cliente.getId(),
            cliente.getNome(),
            cliente.getEmail(),
            cliente.getTelefone(),
            cliente.isAtivo(),
            cliente.getCriadoEm()
        );
    }
}
```

```java
// DTO para atualização parcial
public record ClienteUpdateRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 150)
    String nome,

    @NotBlank(message = "Telefone é obrigatório")
    @Pattern(regexp = "\\d{10,11}")
    String telefone
) {}
```

---

## Service — regras de negócio

```java
package com.empresa.api.cliente;

import com.empresa.api.cliente.dto.*;
import com.empresa.api.exception.RecursoNaoEncontradoException;
import com.empresa.api.exception.EmailJaCadastradoException;
import org.springframework.data.domain.*;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class ClienteService {

    private final ClienteRepository clienteRepository;

    public ClienteService(ClienteRepository clienteRepository) {
        this.clienteRepository = clienteRepository;
    }

    // ===== CRIAR =====
    @Transactional
    public ClienteResponse criar(ClienteRequest request) {
        // Regra de negócio: e-mail único
        if (clienteRepository.existsByEmail(request.email())) {
            throw new EmailJaCadastradoException(request.email());
        }

        var cliente = new Cliente(request.nome(), request.email(), request.telefone());
        var salvo = clienteRepository.save(cliente);
        return ClienteResponse.from(salvo);
    }

    // ===== BUSCAR POR ID =====
    @Transactional(readOnly = true)
    public ClienteResponse buscarPorId(Long id) {
        return clienteRepository.findById(id)
            .map(ClienteResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", id));
    }

    // ===== LISTAR (com paginação) =====
    @Transactional(readOnly = true)
    public Page<ClienteResponse> listar(Pageable pageable) {
        return clienteRepository.findAll(pageable)
            .map(ClienteResponse::from);
    }

    // ===== ATUALIZAR =====
    @Transactional
    public ClienteResponse atualizar(Long id, ClienteUpdateRequest request) {
        var cliente = clienteRepository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", id));

        cliente.atualizar(request.nome(), request.telefone());
        // Não precisa chamar save() — @Transactional persiste automaticamente
        return ClienteResponse.from(cliente);
    }

    // ===== DESATIVAR (soft delete) =====
    @Transactional
    public void desativar(Long id) {
        var cliente = clienteRepository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", id));

        cliente.desativar();
        // Persiste automaticamente pela transação
    }

    // ===== DELETAR (hard delete) =====
    @Transactional
    public void deletar(Long id) {
        if (!clienteRepository.existsById(id)) {
            throw new RecursoNaoEncontradoException("Cliente", id);
        }
        clienteRepository.deleteById(id);
    }
}
```

> 💡 **`@Transactional(readOnly = true)`**: use em operações de leitura. O Hibernate otimiza a sessão — não registra mudanças, melhora performance.

---

## Controller — camada web

```java
package com.empresa.api.cliente;

import com.empresa.api.cliente.dto.*;
import jakarta.validation.Valid;
import org.springframework.data.domain.*;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/clientes")
public class ClienteController {

    private final ClienteService clienteService;

    public ClienteController(ClienteService clienteService) {
        this.clienteService = clienteService;
    }

    @PostMapping
    public ResponseEntity<ClienteResponse> criar(@RequestBody @Valid ClienteRequest request) {
        var response = clienteService.criar(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<ClienteResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(clienteService.buscarPorId(id));
    }

    @GetMapping
    public ResponseEntity<Page<ClienteResponse>> listar(
        @PageableDefault(size = 20, sort = "nome") Pageable pageable
    ) {
        return ResponseEntity.ok(clienteService.listar(pageable));
    }

    @PutMapping("/{id}")
    public ResponseEntity<ClienteResponse> atualizar(
        @PathVariable Long id,
        @RequestBody @Valid ClienteUpdateRequest request
    ) {
        return ResponseEntity.ok(clienteService.atualizar(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        clienteService.deletar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Resumo — papel de cada camada

| Camada | Anotação principal | Responsabilidade | Conhece |
|---|---|---|---|
| Controller | `@RestController` | HTTP in/out, routing | Service, DTOs |
| Service | `@Service` | Regras de negócio | Repository, Domain |
| Repository | `@Repository` / JpaRepository | Acesso a dados | Domain (Entidade) |
| Domain | `@Entity` | Modelo de dados + regras simples | Nada externo |
| DTO | Record / POJO | Transporte de dados | Nada (ou só o Domain para converter) |

---

## Próximas notas
- [[13 - Organização de Pacotes e Boas Práticas]] — como organizar as pastas
- [[14 - RestController e RequestMapping]] — controllers em profundidade
- [[18 - O que são DTOs e Por que Usar]] — DTOs em profundidade
