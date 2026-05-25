# 53 — Projeto Concessionária — Paginação, Documentação e Exceções

tags: #springboot #projeto #concessionária #paginação #swagger #exceções
links: [[52 - Projeto Concessionária - Segurança e JWT]] | [[54 - Logs com SLF4J]] | [[🗺️ Mapa Principal]]

---

## Módulo Carro — com filtros e paginação

### Repository com Specification

```java
package com.concessionaria.api.carro;

import org.springframework.data.jpa.repository.*;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.repository.query.Param;
import java.util.Optional;

public interface CarroRepository extends JpaRepository<Carro, Long>,
        JpaSpecificationExecutor<Carro> {

    @Query("""
        SELECT c FROM Carro c
        JOIN FETCH c.marca m
        WHERE c.id = :id
        """)
    Optional<Carro> findByIdComMarca(@Param("id") Long id);

    boolean existsByMarcaIdAndStatus(Long marcaId, StatusCarro status);
}
```

### Specifications para filtros dinâmicos

```java
package com.concessionaria.api.carro;

import org.springframework.data.jpa.domain.Specification;
import java.math.BigDecimal;

public class CarroSpecs {

    public static Specification<Carro> comStatus(StatusCarro status) {
        return status == null ? null :
            (root, q, cb) -> cb.equal(root.get("status"), status);
    }

    public static Specification<Carro> comMarca(Long marcaId) {
        return marcaId == null ? null :
            (root, q, cb) -> cb.equal(root.get("marca").get("id"), marcaId);
    }

    public static Specification<Carro> comAno(Integer ano) {
        return ano == null ? null :
            (root, q, cb) -> cb.equal(root.get("ano"), ano);
    }

    public static Specification<Carro> precoAte(BigDecimal max) {
        return max == null ? null :
            (root, q, cb) -> cb.lessThanOrEqualTo(root.get("preco"), max);
    }

    public static Specification<Carro> precoDe(BigDecimal min) {
        return min == null ? null :
            (root, q, cb) -> cb.greaterThanOrEqualTo(root.get("preco"), min);
    }

    public static Specification<Carro> modeloContendo(String modelo) {
        return modelo == null ? null :
            (root, q, cb) -> cb.like(
                cb.lower(root.get("modelo")), "%" + modelo.toLowerCase() + "%"
            );
    }
}
```

### CarroService

```java
@Service
public class CarroService {

    private final CarroRepository carroRepository;
    private final MarcaRepository marcaRepository;

    public CarroService(CarroRepository carroRepository, MarcaRepository marcaRepository) {
        this.carroRepository = carroRepository;
        this.marcaRepository = marcaRepository;
    }

    @Transactional
    public CarroResponse criar(CarroRequest request) {
        var marca = marcaRepository.findById(request.marcaId())
            .orElseThrow(() -> new RecursoNaoEncontradoException("Marca", request.marcaId()));

        if (!marca.isAtivo()) {
            throw new RegraDeNegocioException("Não é possível adicionar carro a uma marca inativa");
        }

        var carro = new Carro(marca, request.modelo(), request.ano(),
                              request.preco(), request.cor(), request.quilometragem());
        return CarroResponse.from(carroRepository.save(carro));
    }

    @Transactional(readOnly = true)
    public CarroResponse buscarPorId(Long id) {
        return carroRepository.findByIdComMarca(id)
            .map(CarroResponse::from)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Carro", id));
    }

    @Transactional(readOnly = true)
    public Page<CarroResponse> buscar(
        Long marcaId, String modelo, Integer ano,
        BigDecimal precoMin, BigDecimal precoMax,
        StatusCarro status, Pageable pageable
    ) {
        var spec = Specification
            .where(CarroSpecs.comMarca(marcaId))
            .and(CarroSpecs.modeloContendo(modelo))
            .and(CarroSpecs.comAno(ano))
            .and(CarroSpecs.precoDe(precoMin))
            .and(CarroSpecs.precoAte(precoMax))
            .and(CarroSpecs.comStatus(status != null ? status : StatusCarro.DISPONIVEL));

        return carroRepository.findAll(spec, pageable).map(CarroResponse::from);
    }

    @Transactional
    public CarroResponse atualizar(Long id, CarroRequest request) {
        var carro = carroRepository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Carro", id));

        if (carro.getStatus() == StatusCarro.VENDIDO) {
            throw new RegraDeNegocioException("Não é possível atualizar um carro já vendido");
        }

        carro.atualizar(request.modelo(), request.ano(), request.preco(),
                        request.cor(), request.quilometragem());
        return CarroResponse.from(carro);
    }

    @Transactional
    public void deletar(Long id) {
        var carro = carroRepository.findById(id)
            .orElseThrow(() -> new RecursoNaoEncontradoException("Carro", id));

        if (carro.getStatus() == StatusCarro.VENDIDO) {
            throw new RegraDeNegocioException("Não é possível remover um carro vendido");
        }

        carroRepository.delete(carro);
    }
}
```

### CarroController com todos os filtros

```java
@RestController
@RequestMapping("/api/v1/carros")
@Tag(name = "Carros", description = "Catálogo de veículos")
public class CarroController {

    private final CarroService service;

    public CarroController(CarroService service) { this.service = service; }

    @Operation(summary = "Buscar catálogo com filtros")
    @GetMapping
    public ResponseEntity<Page<CarroResponse>> buscar(
        @RequestParam(required = false) Long marcaId,
        @RequestParam(required = false) String modelo,
        @RequestParam(required = false) Integer ano,
        @RequestParam(required = false) BigDecimal precoMin,
        @RequestParam(required = false) BigDecimal precoMax,
        @RequestParam(required = false) StatusCarro status,
        @PageableDefault(size = 20, sort = "preco") Pageable pageable
    ) {
        return ResponseEntity.ok(
            service.buscar(marcaId, modelo, ano, precoMin, precoMax, status, pageable)
        );
    }

    @GetMapping("/{id}")
    public ResponseEntity<CarroResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @PostMapping
    @SecurityRequirement(name = "JWT")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<CarroResponse> criar(@RequestBody @Valid CarroRequest request) {
        var response = service.criar(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(response.id()).toUri();
        return ResponseEntity.created(location).body(response);
    }

    @PutMapping("/{id}")
    @SecurityRequirement(name = "JWT")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<CarroResponse> atualizar(
        @PathVariable Long id, @RequestBody @Valid CarroRequest request
    ) {
        return ResponseEntity.ok(service.atualizar(id, request));
    }

    @DeleteMapping("/{id}")
    @SecurityRequirement(name = "JWT")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        service.deletar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## GlobalExceptionHandler completo do projeto

```java
package com.concessionaria.api.exception;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.validation.ConstraintViolationException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.*;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;

import java.util.Comparator;
import java.util.List;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNaoEncontrado(RecursoNaoEncontradoException ex, HttpServletRequest req) {
        log.warn("404 {} - {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(404, "Não encontrado", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(RecursoJaExisteException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleJaExiste(RecursoJaExisteException ex, HttpServletRequest req) {
        log.warn("409 {} - {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(409, "Conflito", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(RegraDeNegocioException.class)
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ErrorResponse handleRegraDeNegocio(RegraDeNegocioException ex, HttpServletRequest req) {
        log.warn("422 {} - {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(422, "Regra de negócio", ex.getMessage(), req.getRequestURI());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex, HttpServletRequest req) {
        var erros = ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> new CampoErro(fe.getField(), fe.getRejectedValue(), fe.getDefaultMessage()))
            .sorted(Comparator.comparing(CampoErro::campo))
            .toList();
        return ErrorResponse.ofValidation(400, req.getRequestURI(), erros);
    }

    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleConstraint(ConstraintViolationException ex, HttpServletRequest req) {
        var erros = ex.getConstraintViolations().stream()
            .map(cv -> {
                String path = cv.getPropertyPath().toString();
                String campo = path.contains(".") ? path.substring(path.lastIndexOf('.') + 1) : path;
                return new CampoErro(campo, cv.getInvalidValue(), cv.getMessage());
            }).toList();
        return ErrorResponse.ofValidation(400, req.getRequestURI(), erros);
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleTypeMismatch(MethodArgumentTypeMismatchException ex, HttpServletRequest req) {
        String msg = String.format("Parâmetro '%s' com valor '%s' é inválido", ex.getName(), ex.getValue());
        return ErrorResponse.of(400, "Parâmetro inválido", msg, req.getRequestURI());
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDataIntegrity(DataIntegrityViolationException ex, HttpServletRequest req) {
        log.error("409 Integridade {}: {}", req.getRequestURI(), ex.getMessage());
        return ErrorResponse.of(409, "Conflito de dados",
            "Operação viola restrição de integridade dos dados", req.getRequestURI());
    }

    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleAccessDenied(HttpServletRequest req) {
        return ErrorResponse.of(403, "Acesso negado",
            "Você não tem permissão para esta operação", req.getRequestURI());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenerico(Exception ex, HttpServletRequest req) {
        log.error("500 Erro inesperado em {}: {}", req.getRequestURI(), ex.getMessage(), ex);
        return ErrorResponse.of(500, "Erro interno",
            "Ocorreu um erro inesperado. Tente novamente mais tarde.", req.getRequestURI());
    }
}
```

---

## SwaggerConfig do projeto

```java
package com.concessionaria.api.config;

import io.swagger.v3.oas.models.*;
import io.swagger.v3.oas.models.info.*;
import io.swagger.v3.oas.models.security.*;
import org.springframework.context.annotation.*;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Concessionária API")
                .description("API REST para gestão completa de concessionária de veículos")
                .version("v1.0.0")
                .contact(new Contact()
                    .name("Equipe de Desenvolvimento")
                    .email("dev@concessionaria.com")))
            .addSecurityItem(new SecurityRequirement().addList("JWT"))
            .components(new Components()
                .addSecuritySchemes("JWT", new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")
                    .description("Faça login em /api/v1/auth/login e cole o token aqui")));
    }
}
```

---

## Testando a API completa

```bash
# 1. Registrar usuário
curl -X POST http://localhost:8080/api/v1/auth/registro \
  -H "Content-Type: application/json" \
  -d '{"nome":"Felipe","email":"felipe@ex.com","senha":"Senha@123"}'

# 2. Login
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@concessionaria.com","senha":"Admin@123"}'

TOKEN="eyJhbGci..."

# 3. Criar marca
curl -X POST http://localhost:8080/api/v1/marcas \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"nome":"Toyota","paisOrigem":"Japão"}'

# 4. Criar carro
curl -X POST http://localhost:8080/api/v1/carros \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"marcaId":1,"modelo":"Corolla","ano":2023,"preco":120000,"cor":"Prata","quilometragem":0}'

# 5. Buscar catálogo com filtros
curl "http://localhost:8080/api/v1/carros?precoMax=150000&ano=2023&page=0&size=10"

# 6. Realizar venda
curl -X POST http://localhost:8080/api/v1/vendas \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"carroId":1,"clienteId":1,"valorVenda":118000,"formaPagamento":"FINANCIAMENTO"}'
```

---

## Próximas notas
- [[54 - Logs com SLF4J]] — logging profissional
- [[55 - Profiles dev e prod]] — configuração por ambiente
- [[56 - Docker para Spring Boot]] — containerização
- [[57 - Deploy]] — colocando em produção
