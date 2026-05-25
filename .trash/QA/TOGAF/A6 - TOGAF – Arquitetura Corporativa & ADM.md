> **Disciplina:** Compliance, Quality Assurance & Tests  
> **Prof.:** Dr. Gustavo Molina  
> **Tags:** #togaf #arquitetura-corporativa #adm #fiap

---

## 1. Arquitetura Corporativa

### Definições

- **Def. técnica:** Descrição formal de um sistema, ou planejamento detalhado no nível de componentes, para orientar sua execução.
- **Def. estrutural:** Estrutura dos componentes, seus inter-relacionamentos, e os princípios que regem sua concepção e evolução.
- **Def. estratégica:** Prática que orienta organizações a executar sua estratégia por meio de análises, planejamentos, implementações e mudanças em processos, informações e tecnologia — usando abordagem **holística** (organização como sistema integrado).
- **Def. abstrata:** Conjunto total de cruzamentos entre abstrações e perspectivas relevantes para descrever uma organização.
- **Def. resumida (prof.):** _"Conjunto de modelos necessários para representar uma organização."_

> 💡 Arquitetura não é só criar as representações — é **manter atualizadas**. Sem isso, perde o valor.

### Componentes de uma organização

```
Pessoas · Informação · Processos · Aplicação · Infraestrutura
         ↕                  ↕
    Estratégias         Projetos
```

### Propósito

AC é sobre entender o mundo real de forma que **possibilite discussões sobre mudanças** nesse mesmo mundo real.

**Investe-se em AC para aumentar a chance de sucesso em processos de mudança.**

Ela ajuda a responder:

- Mudar ou não mudar?
- O que mudar?
- Como mudar?
- No que **não** mexer?
- Como lidar com uma mudança malsucedida?

### Contexto — quando usar AC?

|Situação|Exemplo|
|---|---|
|Iniciativas de negócio|Novos serviços/produtos/receitas|
|Iniciativas tecnológicas|Aumento de eficiência, redução de custos|
|Fusões/aquisições|Integração de sistemas|
|Débito tecnológico|Modernização de legado|

> AC gerencia a **complexidade** quando uma mudança envolve múltiplos sistemas e interdependências, buscando equilíbrio entre **transformação** e **continuidade operacional**.

---

## 2. O Padrão TOGAF

**TOGAF** = _The Open Group Architecture Framework_

- Framework para **desenvolver, manter e usar** uma Arquitetura Corporativa.
- Pode ser usado livremente por qualquer empresa para uso interno.
- Desenvolvido e mantido pelo **The Open Group** (time: _Architecture Forum_).

---

## 3. TOGAF-ADM

**ADM** = _Architecture Development Method_

- É o **núcleo** do padrão TOGAF.
- Descreve um método para desenvolver e gerenciar o **ciclo de vida** de uma AC.
- **Não** é um modelo de processo linear.
- É um processo **cíclico, incremental e iterativo** dividido em fases.

### Visão Geral das Fases

```
         ┌─────────────────────┐
         │   Fase Preliminar   │
         └──────────┬──────────┘
                    ↓
         ┌──────────┴──────────┐
         │  A. Visão da Arq.   │
         └──────────┬──────────┘
           ┌────────┤
           ↓        ↓ (ciclo)
  B → C → D → E → F → G → H
           ↑__________________________|
```

### Detalhamento das Fases

#### Fase Preliminar

Prepara o ambiente para iniciar o ADM.

- Define: princípios, escopo, ferramentas, governança e papéis da arquitetura.

#### Fase A — Visão da Arquitetura

- Define a visão inicial, alinhando **objetivos de negócio** com a arquitetura proposta.
- Estabelece: escopo, objetivos, stakeholders e visão de alto nível da solução.

#### Fase B — Arquitetura de Negócio

- Desenha a **estrutura organizacional**, processos e funções de negócio.
- Alinha tudo com a estratégia da empresa.

#### Fase C — Arquitetura de Sistemas de Informação

- Define a arquitetura de **aplicativos e dados** que suportam os processos de negócio.

#### Fase D — Arquitetura de Tecnologia

- Especifica a **infraestrutura tecnológica** (hardware, redes, plataformas) necessária para suportar os sistemas.

#### Fase E — Oportunidades e Soluções

- Agrupa soluções e identifica **projetos para implementação**.

#### Fase F — Planejamento da Migração

- Define **cronograma, prioridades e dependências** para a transição.

#### Fase G — Governança da Implementação

- Garante que a execução **siga a arquitetura planejada**.

#### Fase H — Gerenciamento de Mudanças

- Monitora mudanças e ajusta a arquitetura para garantir **evolução contínua**.

---

## Resumo Rápido das Fases

|Fase|Nome|O que faz|
|---|---|---|
|Preliminar|Preparação|Define princípios, escopo, ferramentas e governança|
|A|Visão da Arquitetura|Alinha negócio + arquitetura; define escopo e stakeholders|
|B|Arquitetura de Negócio|Processos, funções e organização alinhados à estratégia|
|C|Sistemas de Informação|Dados e aplicativos que suportam o negócio|
|D|Tecnologia|Infraestrutura tecnológica necessária|
|E|Oportunidades e Soluções|Agrupa soluções e projetos|
|F|Planejamento da Migração|Cronograma, prioridades e dependências|
|G|Governança da Implementação|Execução conforme arquitetura|
|H|Gerenciamento de Mudanças|Monitora e ajusta para evolução contínua|

---

## Referências

- [[TOGAF - Resumo ADM]] — tabela de fases resumida
- Framework Zachman — pioneiro em arquitetura corporativa (citado nos slides)