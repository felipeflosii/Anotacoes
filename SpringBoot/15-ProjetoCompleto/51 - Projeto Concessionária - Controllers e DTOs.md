# 51 — Projeto Concessionária — Controllers, DTOs e Services

tags: #springboot #projeto #concessionária #controller #service
links: [[50 - Projeto Concessionária - Entidades]] | [[52 - Projeto Concessionária - Segurança e JWT]] | [[🗺️ Mapa Principal]]

---

## Módulo Marca — completo

### DTOs

```java
// MarcaRequest.java
package com.concessionaria.api.marca.dto;

import jakarta.validation.constraints.*;

public record MarcaRequest(

    @NotBlank(message = "Nome é obrigatório")
    @Size(min = 2, max = 100, message = "Nome deve ter entre 2 e 100 caracteres")
    String nome,

    @Size(max = 100, message = "País de origem deve ter no máximo 100 caracteres")
    String paisOrigem
) {}
```

```java
// MarcaResponse.java
package com.concessionaria.api.marca.dto;

import com.concessionaria.api.marca.Marca;
import java.time.LocalDateTime;

public record MarcaResponse(
    Long id,
    String nome,
    String paisOrigem,
    boolean ativo,
    LocalDateTime criadoEm
) {
    public static MarcaResponse from(Marca marca) {
        return new MarcaResponse(
            marca.getId(), marca.getNome(), marca.getPaisOrigem(),
            marca.isAtivo(), marca.getCriadoEm()
        );
    }
}
```

### Repository

```java
package com.concessionaria.api.marca;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface MarcaRepository extends JpaRepository<Marca, Long> {
    boolean existsByNomeIgnoreCase(String nome);
    List<Marca> findByAtivoTrue();
    List<Marca> findByNomeContainingIgnoreCaseAndAtivoTrue(String nome);
}
```

### Service

```java
package com.concessionaria.api.marca;

import com.concessionaria.api.exception.*;
import com.concessionaria.api.marca.dto.*;
import org.springframework.data.domain.*;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
public class MarcaService {

    private final MarcaRepository repository;

    public MarcaService(MarcaRepository repository) {
        this.repository = repository;
    }

    @Transactional
    public MarcaResponse criar(MarcaRequest request) {
        if (repository.existsByNomeIgnoreCase(request.nome())) {
            throw new RecursoJaExisteException("Marca", "nome", request.nome());
        }
        var marca = new Marca(request.nome(), request.paisOrigem());
        return MarcaResponse.from(repository.save(marca));
    }

    @Transactional(readOnly = true)
    public MarcaResponse buscarPorId(Long id) {
        return repository.findById(id)
            .map(MarcaResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Marca", id));
    }

    @Transactional(readOnly = true)
    public Page<MarcaResponse> listar(Pageable pageable) {
        return repository.findAll(pageable).map(MarcaResponse::from);
    }

    @Transactional(readOnly = true)
    public List<MarcaResponse> listarAtivas() {
        return repository.findByAtivoTrue().stream()
            .map(MarcaResponse::from).toList();
    }

    @Transactional
    public MarcaResponse atualizar(Long id, MarcaRequest request) {
        var marca = repository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Marca", id));

        boolean nomeExisteEmOutraMarca = repository.existsByNomeIgnoreCase(request.nome())
            && !marca.getNome().equalsIgnoreCase(request.nome());

        if (nomeExisteEmOutraMarca) {
            throw new RecursoJaExisteException("Marca", "nome", request.nome());
        }

        marca.atualizar(request.nome(), request.paisOrigem());
        return MarcaResponse.from(marca);
    }

    @Transactional
    public void desativar(Long id) {
        var marca = repository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Marca", id));
        marca.desativar();
    }
}
```

### Controller

```java
package com.concessionaria.api.marca;

import com.concessionaria.api.marca.dto.*;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import org.springframework.data.domain.*;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.*;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;
import java.net.URI;
import java.util.List;

@RestController
@RequestMapping("/api/v1/marcas")
@Tag(name = "Marcas", description = "Gestão de marcas de veículos")
@SecurityRequirement(name = "JWT")
public class MarcaController {

    private final MarcaService service;

    public MarcaController(MarcaService service) {
        this.service = service;
    }

    @Operation(summary = "Criar marca")
    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<MarcaResponse> criar(@RequestBody @Valid MarcaRequest request) {
        var response = service.criar(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(response.id()).toUri();
        return ResponseEntity.created(location).body(response);
    }

    @Operation(summary = "Buscar marca por ID")
    @GetMapping("/{id}")
    public ResponseEntity<MarcaResponse> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @Operation(summary = "Listar marcas (paginado)")
    @GetMapping
    public ResponseEntity<Page<MarcaResponse>> listar(
        @PageableDefault(size = 20, sort = "nome") Pageable pageable
    ) {
        return ResponseEntity.ok(service.listar(pageable));
    }

    @Operation(summary = "Listar marcas ativas (para dropdowns)")
    @GetMapping("/ativas")
    public ResponseEntity<List<MarcaResponse>> listarAtivas() {
        return ResponseEntity.ok(service.listarAtivas());
    }

    @Operation(summary = "Atualizar marca")
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<MarcaResponse> atualizar(
        @PathVariable Long id, @RequestBody @Valid MarcaRequest request
    ) {
        return ResponseEntity.ok(service.atualizar(id, request));
    }

    @Operation(summary = "Desativar marca")
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> desativar(@PathVariable Long id) {
        service.desativar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Módulo Venda — o mais complexo (regras de negócio)

### DTOs

```java
// VendaRequest.java
public record VendaRequest(

    @NotNull(message = "Carro é obrigatório")
    @Positive Long carroId,

    @NotNull(message = "Cliente é obrigatório")
    @Positive Long clienteId,

    @NotNull(message = "Valor de venda é obrigatório")
    @Positive(message = "Valor deve ser positivo")
    BigDecimal valorVenda,

    @NotBlank(message = "Forma de pagamento é obrigatória")
    @Pattern(
        regexp = "DINHEIRO|FINANCIAMENTO|CARTAO_CREDITO|PIX|CHEQUE",
        message = "Forma de pagamento inválida"
    )
    String formaPagamento,

    @Size(max = 500)
    String observacao
) {}
```

```java
// VendaResponse.java
public record VendaResponse(
    Long id,
    Long carroId,
    String carroDescricao,
    Long clienteId,
    String clienteNome,
    BigDecimal valorVenda,
    LocalDateTime dataVenda,
    String formaPagamento,
    String observacao,
    LocalDateTime criadoEm
) {
    public static VendaResponse from(Venda venda) {
        return new VendaResponse(
            venda.getId(),
            venda.getCarro().getId(),
            venda.getCarro().getAno() + " " +
                venda.getCarro().getMarca().getNome() + " " +
                venda.getCarro().getModelo(),
            venda.getCliente().getId(),
            venda.getCliente().getNome(),
            venda.getValorVenda(),
            venda.getDataVenda(),
            venda.getFormaPagamento(),
            venda.getObservacao(),
            venda.getCriadoEm()
        );
    }
}
```

### Repository

```java
public interface VendaRepository extends JpaRepository<Venda, Long> {

    @Query("""
        SELECT v FROM Venda v
        JOIN FETCH v.carro c
        JOIN FETCH c.marca m
        JOIN FETCH v.cliente cl
        WHERE v.id = :id
        """)
    Optional<Venda> findByIdComDetalhes(@Param("id") Long id);

    @Query("""
        SELECT v FROM Venda v
        JOIN FETCH v.carro c
        JOIN FETCH c.marca
        JOIN FETCH v.cliente
        WHERE v.cliente.id = :clienteId
        ORDER BY v.dataVenda DESC
        """)
    List<Venda> findByClienteId(@Param("clienteId") Long clienteId);

    @Query("""
        SELECT v FROM Venda v
        JOIN FETCH v.carro c
        JOIN FETCH c.marca
        JOIN FETCH v.cliente
        WHERE v.dataVenda BETWEEN :inicio AND :fim
        """)
    Page<Venda> findByPeriodo(
        @Param("inicio") LocalDateTime inicio,
        @Param("fim") LocalDateTime fim,
        Pageable pageable
    );
}
```

### Service

```java
@Service
public class VendaService {

    private final VendaRepository vendaRepository;
    private final CarroRepository carroRepository;
    private final ClienteRepository clienteRepository;

    public VendaService(VendaRepository vendaRepository,
                        CarroRepository carroRepository,
                        ClienteRepository clienteRepository) {
        this.vendaRepository = vendaRepository;
        this.carroRepository = carroRepository;
        this.clienteRepository = clienteRepository;
    }

    @Transactional
    public VendaResponse realizar(VendaRequest request) {

        // 1. Buscar e validar o carro
        Carro carro = carroRepository.findById(request.carroId())
            .orElseThrow(() -> new RecursoNaoEncontradoException("Carro", request.carroId()));

        if (!carro.isDisponivel()) {
            throw new RegraDeNegocioException(
                "O carro '" + carro.getMarca().getNome() + " " + carro.getModelo() +
                "' não está disponível. Status: " + carro.getStatus()
            );
        }

        // 2. Buscar e validar o cliente
        Cliente cliente = clienteRepository.findById(request.clienteId())
            .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", request.clienteId()));

        if (!cliente.isAtivo()) {
            throw new RegraDeNegocioException("Cliente está inativo e não pode realizar compras");
        }

        // 3. Registrar a venda
        var venda = new Venda(
            carro, cliente,
            request.valorVenda(),
            request.formaPagamento(),
            request.observacao()
        );
        vendaRepository.save(venda);

        // 4. Atualizar status do carro
        carro.marcarComoVendido();

        return VendaResponse.from(venda);
    }

    @Transactional(readOnly = true)
    public VendaResponse buscarPorId(Long id) {
        return vendaRepository.findByIdComDetalhes(id)
            .map(VendaResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Venda", id));
    }

    @Transactional(readOnly = true)
    public Page<VendaResponse> listar(Pageable pageable) {
        return vendaRepository.findAll(pageable).map(VendaResponse::from);
    }

    @Transactional(readOnly = true)
    public List<VendaResponse> buscarPorCliente(Long clienteId) {
        if (!clienteRepository.existsById(clienteId)) {
            throw new RecursoNaoEncontradoException("Cliente", clienteId);
        }
        return vendaRepository.findByClienteId(clienteId)
            .stream().map(VendaResponse::from).toList();
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/api/v1/vendas")
@Tag(name = "Vendas", description = "Registro e consulta de vendas")
@SecurityRequirement(name = "JWT")
public class VendaController {

    private final VendaService service;

    public VendaController(VendaService service) { this.service = service; }

    @PostMapping
    @PreAuthorize("hasAnyRole('ADMIN', 'VENDEDOR')")
    public ResponseEntity<VendaResponse> realizar(@RequestBody @Valid VendaRequest request) {
        var response = service.realizar(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(response.id()).toUri();
        return ResponseEntity.created(location).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<VendaResponse> buscar(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Page<VendaResponse>> listar(
        @PageableDefault(size = 20, sort = "dataVenda", direction = Sort.Direction.DESC)
        Pageable pageable
    ) {
        return ResponseEntity.ok(service.listar(pageable));
    }

    @GetMapping("/cliente/{clienteId}")
    public ResponseEntity<List<VendaResponse>> porCliente(@PathVariable Long clienteId) {
        return ResponseEntity.ok(service.buscarPorCliente(clienteId));
    }
}
```

---

## Próximas notas
- [[52 - Projeto Concessionária - Segurança e JWT]] — configuração de segurança completa
