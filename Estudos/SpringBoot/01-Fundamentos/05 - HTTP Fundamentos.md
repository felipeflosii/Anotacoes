# 05 — HTTP Fundamentos

tags: #springboot #fundamentos #http #rest
links: [[04 - O que é uma API REST]] | [[06 - Criando Projeto com Spring Initializr]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Por que entender HTTP é obrigatório

Sua API Spring Boot fala HTTP. Cada requisição que chega tem um método, headers e às vezes um body. Cada resposta tem um status code e headers. Sem entender HTTP, você não consegue debugar problemas, não consegue projetar APIs corretas e não consegue configurar segurança.

---

## Anatomia de uma requisição HTTP

```
POST /api/v1/clientes HTTP/1.1
Host: api.meusite.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Accept: application/json

{
  "nome": "Felipe Santos",
  "email": "felipe@exemplo.com"
}
```

Partes:
1. **Linha de request:** método + URL + versão HTTP
2. **Headers:** metadados da requisição
3. **Linha em branco:** separa headers do body
4. **Body:** dados enviados (nem sempre presente)

---

## Métodos HTTP — o que cada um significa

| Método | Semântica | Body na request? | Idempotente? | Safe? |
|---|---|---|---|---|
| **GET** | Buscar/ler um recurso | Não | ✅ Sim | ✅ Sim |
| **POST** | Criar um novo recurso | Sim | ❌ Não | ❌ Não |
| **PUT** | Substituir um recurso completo | Sim | ✅ Sim | ❌ Não |
| **PATCH** | Atualizar campos específicos | Sim | ❌ Não* | ❌ Não |
| **DELETE** | Remover um recurso | Não | ✅ Sim | ❌ Não |
| **HEAD** | Como GET, mas sem body na resposta | Não | ✅ Sim | ✅ Sim |
| **OPTIONS** | Descobre métodos suportados | Não | ✅ Sim | ✅ Sim |

> **Idempotente:** chamar N vezes tem o mesmo efeito que chamar 1 vez.
> **Safe:** não altera o estado do servidor.

### Exemplos práticos por método

```
# GET — buscar
GET /clientes         → retorna lista de clientes
GET /clientes/42      → retorna cliente específico

# POST — criar (gera novo ID)
POST /clientes
Body: { "nome": "Felipe", "email": "felipe@ex.com" }
→ retorna 201 Created com o recurso criado

# PUT — substituir completo (envia todos os campos)
PUT /clientes/42
Body: { "nome": "Felipe Santos", "email": "novo@ex.com", "telefone": "11999..." }
→ retorna 200 OK com o recurso atualizado

# PATCH — atualizar parcial (envia só o que muda)
PATCH /clientes/42
Body: { "email": "novo@ex.com" }
→ retorna 200 OK com o recurso atualizado

# DELETE — remover
DELETE /clientes/42
→ retorna 204 No Content
```

---

## Status Codes — o que cada faixa significa

### 2xx — Sucesso

| Código | Nome | Quando usar |
|---|---|---|
| **200** | OK | GET, PUT, PATCH bem-sucedidos |
| **201** | Created | POST que cria um recurso |
| **204** | No Content | DELETE bem-sucedido, ou PUT sem retorno de body |
| **206** | Partial Content | Resposta paginada ou em partes |

### 3xx — Redirecionamento

| Código | Nome | Quando usar |
|---|---|---|
| **301** | Moved Permanently | URL movida definitivamente |
| **302** | Found | Redirecionamento temporário |
| **304** | Not Modified | Recurso não mudou (cache válido) |

### 4xx — Erro do cliente

| Código | Nome | Quando usar |
|---|---|---|
| **400** | Bad Request | Dados inválidos, JSON malformado |
| **401** | Unauthorized | Não autenticado (sem token ou token inválido) |
| **403** | Forbidden | Autenticado mas sem permissão |
| **404** | Not Found | Recurso não existe |
| **405** | Method Not Allowed | Método HTTP não suportado nessa rota |
| **409** | Conflict | Estado conflitante (e-mail já cadastrado) |
| **422** | Unprocessable Entity | Validação de negócio falhou |
| **429** | Too Many Requests | Rate limiting |

### 5xx — Erro do servidor

| Código | Nome | Quando usar |
|---|---|---|
| **500** | Internal Server Error | Erro inesperado no servidor |
| **502** | Bad Gateway | Erro em serviço upstream |
| **503** | Service Unavailable | Servidor sobrecarregado ou em manutenção |
| **504** | Gateway Timeout | Timeout em serviço upstream |

---

## Headers HTTP mais importantes para APIs

### Headers de requisição

```http
# Formato do body enviado
Content-Type: application/json

# Formatos que o cliente aceita na resposta
Accept: application/json

# Token de autenticação
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

# Idioma preferido
Accept-Language: pt-BR

# Para cache condicional
If-None-Match: "abc123"
If-Modified-Since: Wed, 21 Oct 2024 07:28:00 GMT
```

### Headers de resposta

```http
# Formato do body retornado
Content-Type: application/json; charset=UTF-8

# URL do recurso criado (POST 201)
Location: /api/v1/clientes/42

# Controle de cache
Cache-Control: no-cache
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT

# CORS (Cross-Origin Resource Sharing)
Access-Control-Allow-Origin: https://meusite.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

---

## CORS — o que é e por que sempre aparece

CORS (Cross-Origin Resource Sharing) é um mecanismo de segurança dos navegadores que bloqueia requisições de um domínio para outro por padrão.

```
Frontend: https://meusite.com
Backend:  https://api.meusite.com

O navegador vai bloquear a requisição porque os domínios são diferentes.
O backend precisa dizer explicitamente: "eu aceito requisições de meusite.com"
```

No Spring Boot:

```java
// Configuração global de CORS
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://meusite.com", "http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
                    .allowedHeaders("*")
                    .allowCredentials(true);
            }
        };
    }
}

// Ou por controller/método:
@RestController
@CrossOrigin(origins = "http://localhost:3000")
public class ClienteController { ... }
```

---

## Como o Spring Boot mapeia HTTP para Java

```
Requisição HTTP chegando:
───────────────────────────────────────────
POST /api/v1/clientes HTTP/1.1
Content-Type: application/json
Authorization: Bearer abc123

{ "nome": "Felipe", "email": "felipe@ex.com" }
───────────────────────────────────────────

Spring Boot:
1. Recebe no Dispatcher Servlet
2. Roteia para o @RestController correto (pelo @RequestMapping)
3. Converte o body JSON para o objeto Java (@RequestBody)
4. Executa o método handler
5. Converte o objeto Java retornado para JSON
6. Monta a resposta HTTP com o status correto
7. Retorna ao cliente
```

---

## Tabela de referência rápida — REST + HTTP

| Operação | Método | URL | Status sucesso | Status erro comum |
|---|---|---|---|---|
| Listar todos | GET | /recursos | 200 | 400, 401 |
| Buscar um | GET | /recursos/:id | 200 | 404, 401 |
| Criar | POST | /recursos | 201 | 400, 409 |
| Substituir | PUT | /recursos/:id | 200 | 400, 404 |
| Atualizar parcial | PATCH | /recursos/:id | 200 | 400, 404 |
| Deletar | DELETE | /recursos/:id | 204 | 404 |

---

## Próximas notas
- [[06 - Criando Projeto com Spring Initializr]] — criar o primeiro projeto
- [[15 - Métodos HTTP GET POST PUT DELETE]] — como implementar em Spring Boot
