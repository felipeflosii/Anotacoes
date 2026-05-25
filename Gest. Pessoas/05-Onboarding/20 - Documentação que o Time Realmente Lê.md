# 20 — Documentação que o Time Realmente Lê

tags: #onboarding #documentação #processo
links: [[19 - Plano de Onboarding em 30 Dias]] | [[08 - Bus Factor e Conhecimento Compartilhado]]

---

## O problema da documentação

A maioria da documentação de projetos:
- Está desatualizada
- É longa demais para alguém ler voluntariamente
- Descreve *o que* o código faz — que já está no código
- Não descreve *por que* as decisões foram tomadas

---

## Os três documentos que realmente importam

### 1. README — a porta de entrada

O README precisa responder uma única pergunta: **"Como faço esse projeto rodar na minha máquina agora?"**

```markdown
## Como rodar localmente

### Pré-requisitos
- Node 20+
- Docker

### Passos
1. Clone o repositório
2. Copie `.env.example` para `.env` e preencha os valores
3. `docker compose up -d` (sobe banco e Redis)
4. `npm install`
5. `npm run dev`

Acesse: http://localhost:3000
```

**Teste de qualidade:** dê o README para alguém que nunca viu o projeto. Se travar em algum passo, o README está incompleto.

---

### 2. ADRs — o porquê das decisões

Veja [[Template - Decisão de Arquitetura]]. Uma ADR por decisão importante.

O valor aparece 6 meses depois, quando alguém pergunta: "Por que usamos MongoDB aqui em vez de PostgreSQL?"

---

### 3. Guia de contribuição — como o time trabalha

```markdown
## Como contribuir

### Branches
- `main` — produção
- `develop` — base para novas features
- `feature/nome-da-feature` — seu trabalho

### Commits
Use Conventional Commits:
- `feat: adiciona tela de login`
- `fix: corrige validação de e-mail`
- `docs: atualiza README`

### Pull Requests
1. Abra PR para `develop` (nunca para `main` diretamente)
2. Preencha o template de PR
3. Aguarde revisão de pelo menos 1 pessoa
4. Não faça merge do seu próprio PR
```

---

## Documentação que você NÃO precisa escrever

- Comentários explicando *o que* o código faz (escreva código legível)
- Diagramas de sequência de cada endpoint (só para fluxos complexos)
- Manual de usuário detalhado antes de ter usuário real

---

## Como manter a documentação atualizada

**Regra:** quem muda o comportamento, atualiza a documentação. Não é tarefa separada — é parte do PR.

Adicione ao checklist de PR:
- [ ] O README ainda está correto?
- [ ] A decisão tomada precisa de uma ADR?
- [ ] O guia de contribuição foi afetado?

---

## Próximas notas
- [[Template - Plano de Onboarding]]
- [[🗺️ Mapa Principal]]
