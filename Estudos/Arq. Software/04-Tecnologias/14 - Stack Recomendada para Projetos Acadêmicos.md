# 14 — Stack Recomendada para Projetos Acadêmicos

tags: #tecnologias #stack
links: [[13 - Como Escolher Tecnologias]] | [[15 - Quando Adicionar Ferramentas]]

---

## Stack base (2-4 pessoas, 1-3 meses)

```mermaid
flowchart TD
    subgraph Frontend
        R[React ou Vue]
    end
    subgraph Backend
        N[Node.js + Express\nou Python + FastAPI]
    end
    subgraph Banco
        P[(PostgreSQL)]
    end
    subgraph Deploy
        D[Railway ou Render\ngratuito para estudantes]
    end

    R -->|HTTP / REST| N
    N --> P
    N --> D

    style Frontend fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style Backend fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style Banco fill:#FAEEDA,stroke:#854F0B,color:#633806
    style Deploy fill:#F1EFE8,stroke:#5F5E5A,color:#444441
```

---

## Por que essas escolhas

| Tecnologia | Motivo |
|---|---|
| **Node.js + Express** | JavaScript full-stack, grande comunidade, simples de iniciar |
| **Python + FastAPI** | Excelente para quem já conhece Python, docs automáticas com OpenAPI |
| **React** | Mercado amplo, muitos recursos de aprendizado |
| **PostgreSQL** | Relacional robusto, gratuito, suporta JSON quando necessário |
| **Railway / Render** | Deploy simples, tier gratuito generoso, sem cartão de crédito |

---

## Adições comuns por tipo de projeto

| Tipo de projeto | Adiciona |
|---|---|
| Chat / notificações em tempo real | Socket.io |
| Upload de arquivos | Multer + S3 (ou Cloudinary gratuito) |
| Autenticação | JWT + bcrypt (sem biblioteca externa de auth) |
| Cache de consultas | Redis (via Upstash — gratuito) |
| Envio de e-mails | Resend ou Nodemailer + Gmail SMTP |
