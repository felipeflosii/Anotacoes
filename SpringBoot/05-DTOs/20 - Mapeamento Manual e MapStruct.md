# 20 — Mapeamento Manual e MapStruct

tags: #springboot #dto #mapeamento #mapstruct
links: [[18 - O que são DTOs e Por que Usar]] | [[19 - Request vs Response DTO]] | [[🗺️ Mapa Principal]]

---

## O problema do mapeamento

Para cada operação, você precisa converter entre entidade e DTO:

```
Entidade → DTO de Resposta   (ao retornar dados ao cliente)
DTO de Request → Entidade    (ao criar/atualizar no banco)
```

Existem duas abordagens: **mapeamento manual** (você escreve o código) e **MapStruct** (gera o código automaticamente).

---

## Mapeamento Manual — controle total

### Padrão com Factory Method no DTO (mais comum)

```java
// No DTO — método estático de conversão
public record ClienteResponse(Long id, String nome, String email, LocalDateTime criadoEm) {

    // Entidade → DTO
    public static ClienteResponse from(Cliente cliente) {
        return new ClienteResponse(
            cliente.getId(),
            cliente.getNome(),
            cliente.getEmail(),
            cliente.getCriadoEm()
        );
    }
}

// No DTO de Request — método de conversão para entidade
public record ClienteRequest(String nome, String email, String telefone) {

    // DTO → Entidade
    public Cliente toEntity() {
        return new Cliente(this.nome, this.email, this.telefone);
    }
}
```

```java
// Usando no Service:
@Transactional
public ClienteResponse criar(ClienteRequest request) {
    Cliente cliente = request.toEntity();        // DTO → Entidade
    Cliente salvo = repository.save(cliente);
    return ClienteResponse.from(salvo);          // Entidade → DTO
}

public ClienteResponse buscarPorId(Long id) {
    return repository.findById(id)
        .map(ClienteResponse::from)              // method reference — limpo
        .orElseThrow(() -> new RecursoNaoEncontradoException("Cliente", id));
}

public Page<ClienteResponse> listar(Pageable pageable) {
    return repository.findAll(pageable)
        .map(ClienteResponse::from);             // Page<Cliente> → Page<ClienteResponse>
}
```

### Padrão com Mapper class separada

```java
// Classe de mapeamento — separa a responsabilidade
@Component
public class ClienteMapper {

    // Entidade → Response DTO
    public ClienteResponse toResponse(Cliente cliente) {
        return new ClienteResponse(
            cliente.getId(),
            cliente.getNome(),
            cliente.getEmail(),
            cliente.getTelefone(),
            cliente.getCriadoEm()
        );
    }

    // Request DTO → Entidade
    public Cliente toEntity(ClienteRequest request) {
        return new Cliente(
            request.nome(),
            request.email(),
            request.telefone()
        );
    }

    // Atualiza campos da entidade com dados do DTO (para PUT)
    public void updateEntity(Cliente cliente, ClienteUpdateRequest request) {
        cliente.atualizar(request.nome(), request.telefone());
    }

    // Lista de entidades → lista de DTOs
    public List<ClienteResponse> toResponseList(List<Cliente> clientes) {
        return clientes.stream()
            .map(this::toResponse)
            .toList();
    }
}

// Usando no Service:
@Service
public class ClienteService {
    private final ClienteRepository repository;
    private final ClienteMapper mapper;

    // construtor...

    public ClienteResponse criar(ClienteRequest request) {
        Cliente cliente = mapper.toEntity(request);
        Cliente salvo = repository.save(cliente);
        return mapper.toResponse(salvo);
    }
}
```

**Vantagens do mapeamento manual:**
- Controle total — você vê exatamente o que está acontecendo
- Sem dependência extra
- Fácil de debugar
- Lógica customizada (campos calculados, condicionais) é simples

**Desvantagens:**
- Verboso — muito código repetitivo para muitas entidades
- Propenso a erros — esquecer de mapear um campo novo
- Manutenção: cada novo campo exige alterar todos os mappers

---

## MapStruct — geração automática

MapStruct é um **gerador de código** que gera implementações de mapeamento em tempo de compilação (não via reflexão — é eficiente quanto o código manual).

### Configuração

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <annotationProcessorPaths>
                    <!-- MapStruct deve vir antes do Lombok -->
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>1.5.5.Final</version>
                    </path>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                        <version>${lombok.version}</version>
                    </path>
                    <!-- Integração Lombok + MapStruct -->
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok-mapstruct-binding</artifactId>
                        <version>0.2.0</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Uso básico

```java
// Interface de mapeamento — MapStruct gera a implementação
@Mapper(componentModel = "spring")  // integra com Spring IoC
public interface ClienteMapper {

    // Entidade → Response DTO
    // MapStruct mapeia campos com mesmo nome automaticamente
    ClienteResponse toResponse(Cliente cliente);

    // Request DTO → Entidade
    @Mapping(target = "id", ignore = true)          // ignora o id (gerado pelo banco)
    @Mapping(target = "criadoEm", ignore = true)    // gerado na entidade
    @Mapping(target = "ativo", constant = "true")   // valor fixo
    Cliente toEntity(ClienteRequest request);

    // Atualiza entidade existente com dados do DTO
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "email", ignore = true)    // email não pode ser alterado
    @Mapping(target = "criadoEm", ignore = true)
    void updateEntity(@MappingTarget Cliente cliente, ClienteUpdateRequest request);

    // Lista
    List<ClienteResponse> toResponseList(List<Cliente> clientes);
}
```

### Mapeamentos com nomes diferentes

```java
@Mapper(componentModel = "spring")
public interface ProdutoMapper {

    // Campos com nomes diferentes entre entidade e DTO
    @Mapping(source = "categoria.nome", target = "nomeCategoria")
    @Mapping(source = "preco", target = "precoVenda")
    @Mapping(source = "estoque", target = "quantidadeEmEstoque")
    ProdutoResponse toResponse(Produto produto);

    // Campo calculado — usando expressão
    @Mapping(target = "margemLucro",
             expression = "java(produto.getPreco().subtract(produto.getCusto()))")
    ProdutoAdminResponse toAdminResponse(Produto produto);
}
```

### Mapeamento com relacionamentos

```java
@Mapper(componentModel = "spring", uses = {CategoriaMapper.class})
// uses = outras interfaces de mapper para reutilizar
public interface ProdutoMapper {

    // MapStruct usa CategoriaMapper para mapear o relacionamento
    ProdutoResponse toResponse(Produto produto);
}

@Mapper(componentModel = "spring")
public interface CategoriaMapper {
    CategoriaResponse toResponse(Categoria categoria);
}
```

### O código gerado pelo MapStruct

```java
// MapStruct gera automaticamente algo assim em target/generated-sources:
@Component
public class ClienteMapperImpl implements ClienteMapper {

    @Override
    public ClienteResponse toResponse(Cliente cliente) {
        if (cliente == null) return null;
        return new ClienteResponse(
            cliente.getId(),
            cliente.getNome(),
            cliente.getEmail(),
            cliente.getCriadoEm()
        );
    }

    @Override
    public Cliente toEntity(ClienteRequest request) {
        if (request == null) return null;
        Cliente cliente = new Cliente();
        cliente.setNome(request.nome());
        cliente.setEmail(request.email());
        cliente.setTelefone(request.telefone());
        cliente.setAtivo(true);
        return cliente;
    }
}
```

---

## Comparação: Manual vs MapStruct

| Critério | Manual (Factory Method) | MapStruct |
|---|---|---|
| **Verbosidade** | Alto (muito código) | Baixo (só a interface) |
| **Performance** | Excelente | Excelente (compile-time) |
| **Controle** | Total | Alto (com anotações) |
| **Curva de aprendizado** | Nenhuma | Média |
| **Manutenção** | Alta (alterar todos os mappers) | Baixa (campo novo → mapeia automático) |
| **Erros comuns** | Esquecer campos | Configuração errada das anotações |
| **Debug** | Simples | Requer ver o código gerado |

---

## Recomendação por cenário

```
Projeto pequeno / aprendizado:
  → Mapeamento manual com factory method no DTO
  → Simples, visível, sem dependência extra

Projeto médio com muitas entidades:
  → MapStruct — reduz muito o código repetitivo

Campos com lógica complexa:
  → Manual para esses campos específicos
  → Combine com MapStruct onde é simples

Regra geral do mercado:
  → Empresas com muitas entidades usam MapStruct
  → Startups pequenas costumam usar manual ou Jackson direto
```

---

## Próximas notas
- [[21 - JPA e Hibernate Fundamentos]] — persistência de dados
- [[29 - Bean Validation]] — validações nos DTOs de entrada
