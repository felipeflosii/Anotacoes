---
tags: [java, ano-5, open-source, comunidade, carreira, speaking, blog]
trimestre: T18
meses: 55-57
---

# T18 · Open Source e Liderança Técnica
### Meses 55–57 · Ano 5

---

## 🔵 Bloco 1 — Contribuindo para Open Source Java

### Como começar

```
1. Escolha um projeto que você já usa
   - Spring Framework, Spring Boot, Spring Security
   - Quarkus, Micronaut
   - Testcontainers
   - Resilience4j
   - Apache Kafka clients

2. Setup do ambiente de contribuição
   - Fork do repositório
   - Ler o CONTRIBUTING.md
   - Entrar no Discord/Slack do projeto
   - Entender o processo de PR e CI

3. Primeiro issue — comece pequeno
   - Label "good first issue" ou "help wanted"
   - Documentação ou testes (menor risco de rejeição)
   - Fix de bug pequeno com teste
   - Melhoria de mensagem de erro

4. Construir credibilidade
   - Code reviews de outros PRs
   - Responder issues no GitHub
   - Participar de discussões em design issues
```

### Projetos com "good first issue" para Java devs

| Projeto | Área | Linguagem principal |
|---------|------|-------------------|
| spring-projects/spring-boot | Framework | Java |
| testcontainers/testcontainers-java | Testes | Java |
| resilience4j/resilience4j | Resilience | Java |
| apache/kafka | Messaging | Java/Scala |
| quarkusio/quarkus | Framework | Java |
| langchain4j/langchain4j | AI | Java |

### Criar seu próprio projeto open source

```
Ideias que têm boa recepção:
- Spring Boot Starter para algo que não existe ainda
- Biblioteca utilitária para problema real que você teve
- Plugin Maven/Gradle para automação de pipeline
- Arquétipo/template de projeto Java com boas práticas
- Ferramenta CLI em Java para devs

Estrutura mínima de um bom projeto OSS:
├── README.md           (instalação, uso, exemplos)
├── CONTRIBUTING.md     (como contribuir)
├── CHANGELOG.md        (histórico de versões)
├── LICENSE             (MIT, Apache 2.0)
├── .github/
│   ├── workflows/ci.yml
│   └── ISSUE_TEMPLATE/
└── src/
    └── (código com testes > 80% cobertura)
```

---

## 🔵 Bloco 2 — Blog Técnico e Conteúdo

### Plataformas para devs Java brasileiros

| Plataforma | Audiência | Formato ideal |
|-----------|-----------|--------------|
| **Dev.to** | Global (EN) | Artigos técnicos curtos/médios |
| **Medium** | Global (EN/PT) | Artigos longos, tutoriais |
| **Hashnode** | Global (EN) | Blog próprio no seu domínio |
| **LinkedIn** | BR + Global | Posts curtos, artigos |
| **YouTube** | BR | Tutoriais, live coding |

### Tipos de conteúdo que performam bem

```
1. "X coisas que aprendi sobre Y" — fácil de ler, compartilhável
2. "Como resolvemos Z em produção" — case real, muito valorizado
3. Comparativos: "Kafka Streams vs Flink: quando usar cada um"
4. Desmistificando: "Virtual Threads explicados sem complicar"
5. Série longa: "De zero a microservices em Java — parte 1/N"
```

---

## 🔵 Bloco 3 — Speaking e Conferências

### Progressão de exposição pública

```
1. Meetups locais (50-100 pessoas)
   - Java User Group (JUG) da sua cidade
   - Spring Brazil, GDG local
   - Custo: zero, feedback imediato

2. Conferências nacionais (200-1000 pessoas)
   - TDC (The Developer's Conference) — maior do BR
   - QCon São Paulo
   - JavaOne BR
   - DevOpsDays

3. Conferências internacionais (500-5000 pessoas)
   - Devoxx (Europa — Bélgica, França, UK)
   - QCon London/NYC/SF
   - Spring I/O (Barcelona)
   - JavaOne (San Francisco)
   - KubeCon (global)
```

### Como criar uma boa palestra técnica

```
Estrutura de 40 minutos:

1. Hook (3 min) — por que este problema importa?
   "Em 2025 tivemos 3 incidentes de produção por causa disso."

2. Contexto (5 min) — onde estávamos antes
   Mostre o problema concreto com exemplos reais

3. Jornada (25 min) — o que fizemos / aprendemos
   Live coding ou demos > slides
   Mostre os erros que cometeram também (mais valioso!)

4. Resultado (5 min) — o que mudou
   Métricas antes/depois se possível

5. Q&A (5 min)

Dicas:
- Demo ao vivo > slides genéricos
- Mostrar falhas e aprendizados > só os sucessos
- Complexidade adequada ao público
- Slides no GitHub após a palestra
```

---

## 🔵 Bloco 4 — Carreira Global (Mercado Internacional)

### Posições que contratam remote-first para Java Sênior/Staff

| Empresa | Tipo | Onde encontrar |
|---------|------|---------------|
| Remote-first startups (EU/US) | Contrato PJ | LinkedIn, Otta, Himalayas |
| Big Techs com escritório remoto | CLT/contractor | careers.google.com, amazon.jobs |
| Empresas unicórnio BR (Nubank, iFood, Mercado Livre) | CLT | Greenhouse/Lever delas |
| Consultorias globais (Thoughtworks, EPAM) | CLT | Site delas |
| Crypto/Web3 (se interesse) | Token + salário | Telegram de comunidades |

### Preparação para entrevistas Big Tech

```
Estrutura típica de processo (Google, Meta, Amazon):

1. Recruiter Screen (30 min) — fit cultural, expectativas salariais
2. Technical Phone Screen (45-60 min) — LeetCode Medium
3. Virtual Onsite (4-5 rounds x 45-60 min cada):
   - 2-3x Coding (LeetCode Medium/Hard)
   - 1x System Design (projetar sistema distribuído)
   - 1x Behavioral (STAR method)

Preparação para Coding:
- LeetCode: 150+ problemas, foco em:
  - Arrays, Strings, HashMap (fácil/médio)
  - Trees, Graphs (médio)
  - Dynamic Programming (médio/difícil)
  - Two pointers, Sliding window
  - Heaps, Tries

Preparação para System Design:
- "Designing Data-Intensive Applications" — Kleppmann (obrigatório)
- "System Design Interview" — Alex Xu (volumes 1 e 2)
- Praticar: design de URL shortener, Twitter, Uber, Netflix CDN

Preparação para Behavioral (Amazon Leadership Principles):
- 10 histórias STAR prontas cobrindo:
  - Conflito técnico com colega/gestor
  - Decisão técnica com impacto grande
  - Entrega sob pressão
  - Erro cometido e lição aprendida
  - Influência sem autoridade
```

---

## 🔵 Bloco 5 — Finanças para Dev Sênior/Staff

```
Estratégias de maximização de renda:

Brasil CLT Sênior/Staff:
- Negociar PLR (participação nos lucros) além do salário
- Stock options/RSUs em startups
- Benefícios: home office budget, equipamento, educação

Remoto Internacional (PJ/Contractor):
- Salário USD/EUR → repatriação via conta internacional
- MEI (até R$81k/ano) ou empresa LTDA
- Planejamento tributário com contador especializado em TI

Renda extra:
- Cursos online no seu nicho (Java + Spring + Cloud = mercado grande)
- Mentoria paga (Toptal, Plataforma própria)
- Consultoria pontual
- Palestras pagas em eventos corporativos
- Livro técnico (O'Reilly, Casa do Código)
```

---

## 🔗 Navegação

← [[T17 — Staff Engineer e Impacto Técnico]]  
→ [[Mapa de Certificações]]
