# 40 — Versionamento de API

tags: #springboot #rest #versionamento #api
links: [[39 - Padrão REST Completo]] | [[41 - Paginação e Ordenação]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Por que versionar

Sem versionamento, uma mudança na API quebra todos os clientes que já a usam (frontend, mobile, parceiros). Com versionamento, você evolui a API enquanto mantém versões antigas funcionando durante a transição.

---

## Estratégias de versionamento

### 1. Versionamento por URL ✅ — mais comum e recomendado

```
/api/v1/clientes
/api/v2/clientes
```

```java
// Controller v1
@RestController
@RequestMapping("/api/v1/clientes")
public class ClienteV1Controller {
    @GetMapping("/{id}")
    public ClienteV1Response buscar(@PathVariable Long id) { ... }
}

// Controller v2 — nova estrutura de resposta
@RestController
@RequestMapping("/api/v2/clientes")
public class ClienteV2Controller {
    @GetMapping("/{id}")
    public ClienteV2Response buscar(@PathVariable Long id) { ... }
}
```

**Vantagens:** simples, visível na URL, fácil de testar com curl/Postman
**Desvantagens:** "polui" a URL (puristas REST não gostam)

### 2. Versionamento por Header

```
GET /api/clientes/42
Accept: application/vnd.empresa.v2+json
```

```java
@GetMapping(value = "/{id}", produces = "application/vnd.empresa.v1+json")
public ClienteV1Response buscarV1(@PathVariable Long id) { ... }

@GetMapping(value = "/{id}", produces = "application/vnd.empresa.v2+json")
public ClienteV2Response buscarV2(@PathVariable Long id) { ... }
```

### 3. Versionamento por Query Param

```
GET /api/clientes/42?version=2
```

**Para projetos novos:** use versionamento por URL — é o mais simples e amplamente adotado.

---

## Estratégia de migração

```
v1 → lançamento inicial
v2 → nova versão com mudanças breaking
     v1 fica ativa por 6-12 meses (deprecated)
     clientes migram gradualmente
     v1 é desativada com aviso antecipado

Header de deprecação:
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: </api/v2/clientes>; rel="successor-version"
```

```java
// Marcando endpoint como deprecated
@GetMapping
@Deprecated
public ResponseEntity<List<ClienteV1Response>> listar() {
    return ResponseEntity.ok()
        .header("Deprecation", "true")
        .header("Sunset", "Sat, 01 Jan 2025 00:00:00 GMT")
        .header("Link", "</api/v2/clientes>; rel=\"successor-version\"")
        .body(service.listarV1());
}
```

---

## Próximas notas
- [[41 - Paginação e Ordenação]] — paginação completa
- [[42 - Filtros e Idempotência]]
