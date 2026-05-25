# 04 — O que é uma API REST

tags: #springboot #fundamentos #rest #api
links: [[03 - IoC e Injeção de Dependência]] | [[05 - HTTP Fundamentos]] | [[Estudos/Projetos/00-Maps/🗺️ Mapa Principal]]

---

## Definição

**API** (Application Programming Interface) é um conjunto de regras que define como sistemas se comunicam.

**REST** (Representational State Transfer) é um **estilo arquitetural** — um conjunto de restrições para criar APIs web usando HTTP de forma padronizada e previsível.

> REST não é um protocolo, não é uma biblioteca, não é um framework. É um **jeito de pensar** como organizar sua API.

---

## Por que REST domina o mercado

Antes do REST (e em paralelo), existiam outras abordagens:

| Abordagem | Problema |
|---|---|
| **SOAP** | Verboso, XML pesado, difícil de usar |
| **XML-RPC** | Limitado, sem padrão para recursos |
| **CORBA** | Extremamente complexo |
| **REST** | Simples, usa HTTP que todo mundo já conhece |

REST ganhou porque aproveita a infraestrutura HTTP que já existe (navegadores, proxies, CDNs, caches) e tem uma curva de aprendizado muito menor.

---

## Os 6 princípios REST (constraints)

Para uma API ser RESTful, deve seguir estas restrições:

### 1. Interface Uniforme
A API usa um conjunto consistente de regras. Os recursos são identificados por URLs e as operações pelos métodos HTTP.

```
GET  /clientes        → lista clientes
GET  /clientes/42     → busca cliente 42
POST /clientes        → cria um cliente
PUT  /clientes/42     → atualiza cliente 42
DELETE /clientes/42   → remove cliente 42
```

### 2. Cliente-Servidor
O cliente (frontend, mobile, outro serviço) e o servidor (sua API) são independentes. O cliente não sabe como o servidor armazena dados; o servidor não sabe como o cliente exibe os dados.

### 3. Sem Estado (Stateless)
Cada requisição contém **todas as informações** necessárias para ser processada. O servidor não mantém sessão.

```
// ❌ Com estado (não REST)
// Servidor lembra que o usuário fez login antes

// ✅ Stateless (REST)
// Cada requisição inclui o token de autenticação
GET /meu-perfil
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### 4. Cacheável
Respostas podem ser marcadas como cacheáveis ou não. O cliente pode armazenar respostas em cache, reduzindo requisições ao servidor.

### 5. Sistema em Camadas
O cliente não precisa saber se está falando com o servidor real, um load balancer, um cache ou um gateway. A API abstrai isso.

### 6. Code on Demand (opcional)
O servidor pode retornar código executável (JavaScript). Raramente usado.

---

## Recursos — o conceito central do REST

Em REST, tudo é um **recurso**. Um recurso é qualquer entidade que faz sentido no seu domínio:

- Um cliente: `/clientes/42`
- Uma lista de pedidos: `/pedidos`
- Os pedidos de um cliente: `/clientes/42/pedidos`
- Um produto específico: `/produtos/99`

### Regras de nomenclatura de recursos

```
✅ Usar substantivos no plural
   /clientes, /pedidos, /produtos

❌ Não usar verbos nas URLs
   /buscarCliente, /criarPedido, /deletarProduto

✅ Hierarquia representa relação
   /clientes/42/pedidos  (pedidos do cliente 42)

✅ Minúsculo com hífens para espaços
   /categorias-de-produto

❌ Não usar underscores ou camelCase
   /categorias_produto, /categoriasProduto
```

---

## REST vs RESTful vs HTTP API

É comum confundir os termos:

- **HTTP API**: qualquer API que usa HTTP. Não precisa seguir os princípios REST.
- **REST**: o conjunto de princípios definido por Roy Fielding.
- **RESTful**: adjetivo para uma API que segue os princípios REST.

Na prática, a maioria das APIs chamadas de "REST" são tecnicamente "HTTP APIs" que seguem **parte** dos princípios. Isso é aceitável — o mercado usa "REST API" e "RESTful API" de forma intercambiável para APIs HTTP bem projetadas.

---

## Exemplo de API mal vs bem projetada

```
❌ API mal projetada (sem seguir REST):
POST /api/buscarCliente?id=42
POST /api/criarNovoPedido
POST /api/atualizarStatusPedido?id=10&status=pago
GET  /api/deletarCliente?id=42

✅ API RESTful:
GET    /clientes/42
POST   /pedidos
PATCH  /pedidos/10  { "status": "pago" }
DELETE /clientes/42
```

A API RESTful é auto-documentável — o método HTTP já diz a intenção da operação.

---

## O papel do JSON

REST não define o formato dos dados. Mas na prática, **JSON domina** por ser:
- Leve e legível
- Nativo no JavaScript (frontend/mobile)
- Suportado por todas as linguagens

```json
// Resposta típica de GET /clientes/42
{
  "id": 42,
  "nome": "Felipe Santos",
  "email": "felipe@exemplo.com",
  "createdAt": "2024-01-15T10:30:00Z"
}

// Resposta de GET /clientes (lista paginada)
{
  "content": [
    { "id": 42, "nome": "Felipe Santos" },
    { "id": 43, "nome": "Maria Silva" }
  ],
  "totalElements": 150,
  "totalPages": 8,
  "number": 0,
  "size": 20
}
```

---

## Próximas notas
- [[05 - HTTP Fundamentos]] — os métodos, status codes e headers
- [[39 - Padrão REST Completo]] — boas práticas avançadas de REST
