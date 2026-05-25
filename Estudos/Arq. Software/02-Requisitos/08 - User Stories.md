# 08 — User Stories

tags: #requisitos #agile
links: [[07 - Requisitos Funcionais e Não-Funcionais]] | [[09 - Priorização com MoSCoW]] | [[Template - User Story]]

---

## O que é

User Story é uma forma de escrever requisitos **na perspectiva do usuário**, com foco no valor que a funcionalidade entrega — não nos detalhes técnicos de implementação.

---

## Formato padrão

```
Como [tipo de usuário],
quero [ação que deseja realizar],
para que [benefício / objetivo].
```

---

## Critério de aceitação (obrigatório)

Toda user story precisa de critérios de aceitação — são os testes que definem quando a story está "pronta".

```
Dado que [contexto inicial],
quando [ação do usuário],
então [resultado esperado].
```

---

## Exemplos completos

### Cadastro de usuário
```
Como visitante do site,
quero me cadastrar com e-mail e senha,
para que eu possa acessar as funcionalidades da plataforma.

Critérios de aceitação:
- Dado que estou na página de cadastro,
  quando preencho e-mail válido e senha com mínimo 8 caracteres,
  então recebo um e-mail de confirmação e sou redirecionado para o dashboard.

- Dado que tento me cadastrar com um e-mail já existente,
  quando submeto o formulário,
  então vejo a mensagem "E-mail já cadastrado" e o formulário não é enviado.
```

### Reset de senha
```
Como usuário cadastrado que esqueceu a senha,
quero receber um link de redefinição por e-mail,
para que eu possa recuperar o acesso à minha conta.

Critérios de aceitação:
- Dado que informo um e-mail cadastrado,
  quando clico em "Esqueci minha senha",
  então recebo um e-mail com link válido por 1 hora.

- Dado que o link expirou,
  quando tento acessá-lo,
  então vejo a mensagem "Link expirado. Solicite um novo."
```

---

## O que NÃO é uma boa user story

❌ "Implementar autenticação JWT" — isso é uma tarefa técnica, não uma story
❌ "O sistema deve ter um banco de dados" — isso é infraestrutura
❌ "Fazer o login funcionar" — vago, sem critério de aceitação

---

## Próximas notas
- [[09 - Priorização com MoSCoW]]
- [[Template - User Story]]
