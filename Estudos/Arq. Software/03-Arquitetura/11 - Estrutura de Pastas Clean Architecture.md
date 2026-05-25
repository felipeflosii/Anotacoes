# 11 — Estrutura de Pastas Clean Architecture

tags: #arquitetura #clean #código
links: [[03 - Clean Architecture]] | [[12 - Regra de Dependência]]

---

## Estrutura completa

```
src/
├── domain/                    ← Núcleo — zero dependências externas
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   ├── repositories/          ← Interfaces (contratos)
│   │   ├── IUserRepository.ts
│   │   └── IOrderRepository.ts
│   └── services/              ← Regras de negócio puras
│       └── PricingService.ts
│
├── application/               ← Casos de uso — usa o domínio
│   └── use-cases/
│       ├── user/
│       │   ├── CreateUser.ts
│       │   └── AuthenticateUser.ts
│       └── order/
│           └── PlaceOrder.ts
│
├── infrastructure/            ← Implementações concretas
│   ├── database/
│   │   ├── UserRepositoryPG.ts     ← implementa IUserRepository
│   │   └── OrderRepositoryPG.ts
│   ├── http/
│   │   ├── routes/
│   │   │   └── userRoutes.ts
│   │   └── controllers/
│   │       └── UserController.ts
│   ├── email/
│   │   └── SendgridEmailService.ts
│   └── config/
│       └── database.ts
│
├── shared/                    ← Utilitários sem regra de negócio
│   ├── errors/
│   │   └── AppError.ts
│   └── utils/
│       └── dateUtils.ts
│
└── main.ts                    ← Ponto de entrada — monta as dependências
```

---

## Regra de imports

```
✅ domain   → não importa nada de fora do domain
✅ application → importa só de domain
✅ infrastructure → importa de application e domain
❌ domain → infrastructure  (NUNCA)
❌ domain → application     (NUNCA)
```

---

## Como revisar um PR violando a regra

Se você ver isso no código, rejeite o PR:

```typescript
// ❌ VIOLAÇÃO — entidade importando do banco
import { db } from '../../infrastructure/database'

class User {
  async save() {
    await db.query('INSERT INTO users ...')  // domínio não deve saber do banco
  }
}
```

```typescript
// ✅ CORRETO — entidade pura, repositório na infraestrutura
class User {
  constructor(private email: string, private name: string) {}
  validate() { return this.email.includes('@') }
}
```

---

## Próximas notas
- [[12 - Regra de Dependência]]
- [[19 - Contrato de API]]
