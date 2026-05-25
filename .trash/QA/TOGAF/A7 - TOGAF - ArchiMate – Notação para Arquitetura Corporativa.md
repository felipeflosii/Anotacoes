> **Disciplina:** Compliance, Quality Assurance & Tests  
> **Prof.:** Dr. Gustavo Molina  
> **Tags:** #archimate #togaf #arquitetura-corporativa #fiap  
> **Relacionado:** [[TOGAF - Arquitetura Corporativa & ADM]]

---

## Download e Instalação

Ferramenta: **Archi** (open source)  
Site: https://www.archimatetool.com/  
Instalação: Next → Next → Finish

---

## Elementos Chave

### Agrupamento por Domínio (cores)

|Cor|Domínio|Fase TOGAF|
|---|---|---|
|Amarelo|Motivação|Fase A|
|Amarelo|Negócio|Fase B|
|Azul|Aplicação|Fase C|
|Verde|Tecnologia|Fase D|

### Categorias (válido para Negócio, Aplicação e Tecnologia)

|Categoria|Analogia linguística|Descrição|
|---|---|---|
|**Estrutura Ativa**|Sujeito|Quem executa|
|**Comportamento**|Verbo|O que é feito|
|**Estrutura Passiva**|Objeto|Sobre o que se age|

> Os elementos de **Motivação** não se dividem nessas categorias.

---

## Elementos por Domínio

### Motivação (Fase A)

|Elemento|Definição|
|---|---|
|**Stakeholder**|Papel de indivíduo, equipe ou organização com interesse na arquitetura|
|**Driver (Direcionador)**|Condição interna/externa que motiva a organização a definir objetivos e implementar mudanças|
|**Assessment (Avaliação)**|Resultado de uma análise da situação relacionada a um Driver|
|**Goal (Objetivo)**|Declaração de alto nível de intenção ou resultado final desejado|
|**Principle (Princípio)**|Declaração de intenção que se aplica a qualquer sistema no contexto da arquitetura|
|**Requirement (Requisito)**|Declaração de necessidade que se aplica a um sistema específico|
|**Constraint (Limitação)**|Limitação na realização ou implementação de um elemento|

---

### Negócio (Fase B)

|Categoria|Elemento|Definição|
|---|---|---|
|Ativa|**Actor (Ator)**|Entidade capaz de executar ação ou comportamento|
|Ativa|**Role (Papel)**|Responsabilidade de executar um comportamento específico|
|Ativa|**Interface**|Ponto de acesso onde serviços são disponibilizados|
|Comportamento|**Process (Processo)**|Sequência de comportamentos que atingem um resultado|
|Comportamento|**Function (Função)**|Coleção de ações baseada em critério específico (recursos/competências)|
|Comportamento|**Event (Evento)**|Mudança de status relacionada ao negócio|
|Comportamento|**Service (Serviço)**|Expõe a funcionalidade de um Role para o ambiente; realizado por Processo ou Função|

---

### Aplicação (Fase C)

|Categoria|Elemento|Definição|
|---|---|---|
|Ativa|**Component (Componente)**|Encapsulamento de funcionalidade modular e substituível|
|Ativa|**Interface**|Ponto de acesso dos serviços da aplicação|
|Comportamento|**Function (Função)**|Comportamento automático executado por um componente|
|Comportamento|**Process (Processo)**|Sequência de comportamentos que atingem um resultado|
|Comportamento|**Event (Evento)**|Mudança de estado da aplicação|
|Comportamento|**Service (Serviço)**|Expõe ao ambiente um comportamento específico da aplicação|
|Passiva|**Data Object**|Conjunto de dados estruturados para processamento|

---

### Tecnologia (Fase D)

|Categoria|Elemento|Definição|
|---|---|---|
|Ativa|**Node (Nó)**|Recurso físico ou computacional que hospeda/interage com outros recursos|
|Ativa|**Device (Dispositivo)**|Recurso físico de TI onde software/artefatos são armazenados ou executados|
|Ativa|**Software**|Software que fornece ambiente de armazenagem, execução e uso|
|Ativa|**Network (Rede)**|Estruturas que conectam dispositivos/software para transmissão de dados|
|Comportamento|**Event (Evento)**|Mudança de estado|
|Passiva|**Artifact (Artefato)**|Informação usada/produzida no desenvolvimento ou operação de TI|
|Passiva|**Facility (Instalação)**|Ambiente ou estrutura física|

---

## Relacionamentos

|Relacionamento|Descrição|
|---|---|
|**Composição**|Um elemento consiste de um ou mais outros → _composto de / compõe_|
|**Agregação**|Um elemento combina um ou mais outros → _agrega / agregado em_|
|**Designação**|Alocação de responsabilidade, execução, armazenamento → _designado para / recebe designação_|
|**Realização**|Elemento tem papel crítico na criação/manutenção de elemento mais abstrato → _realiza / realizado por_|
|**Servidão**|Elemento fornece sua funcionalidade a outro → _serve / servido por_|
|**Acesso**|Elementos ativos/comportamento agem sobre elementos passivos → _acessa / acessado por_|
|**Influência**|Elemento afeta implementação/atingimento de elemento de Motivação → _influencia / influenciado por_|
|**Associação**|Relação não especificada ou não corretamente representada pelos outros tipos|

---

## Exemplo de Diagrama

![[Exemplo_Archi.JPG]]

> O diagrama acima ilustra um modelo de **Vendas** com três camadas:
> 
> - **Negócio (amarelo):** Role _Atendimento_, processo de vendas (Abordagem → Oferta → Fechamento), serviço de vendas
> - **Aplicação (azul):** Sistema CRM, Sistema ERP, serviços de gestão e registro
> - **Tecnologia (verde):** Server CRM (Java 8.1), Server Oracle, Server ERP (Windows)

---

## Referências

- https://pubs.opengroup.org/togaf-standard/index.html
- https://conexiam.com/pt/togaf-adm-phases-explained/
- https://arquiteturacorporativa.com.br/2010/09/frameworks-de-arquitetura-parte-2-togaf/
- https://www.youtube.com/watch?v=zOAtcyDzYBs
- https://archimatetool.gitbook.io/quick_guide