# 👋 Welcome Journey & Dynamic Product Personalization

> Case técnico de uma jornada automatizada de boas-vindas com personalização dinâmica de produtos utilizando Salesforce Marketing Cloud Engagement.

---

## 📌 Visão Geral

Este case apresenta uma solução de **Jornada de Boas-Vindas** desenvolvida no Salesforce Marketing Cloud Engagement, combinando automação, dados e personalização de conteúdo.

A solução utiliza informações do cliente e dados de produtos para selecionar recomendações relevantes e construir dinamicamente uma vitrine personalizada dentro do e-mail.

O fluxo combina recursos como:

- Automation Studio
- Journey Builder
- Data Extensions
- SQL
- Content Builder
- AMPscript
- JSON
- conteúdo dinâmico

O objetivo não é apenas realizar um disparo de boas-vindas, mas utilizar os dados disponíveis para entregar uma comunicação personalizada desde o início do relacionamento com o cliente.

---

## 🎯 Objetivo

Criar uma jornada automatizada capaz de:

- identificar clientes elegíveis para uma comunicação de boas-vindas;
- preparar os dados necessários para entrada na jornada;
- selecionar produtos relevantes para cada cliente;
- organizar e priorizar as recomendações;
- disponibilizar os produtos para personalização do e-mail;
- montar dinamicamente uma vitrine com diferentes quantidades de produtos;
- manter a preparação dos dados separada da camada de apresentação.

A arquitetura foi desenhada para que o **Journey Builder seja responsável pela orquestração**, enquanto o processamento e preparação dos dados acontecem antes da comunicação.

---

## 🏗️ Arquitetura da Solução

Fluxo conceitual:

```text
Customer Data
      │
      ▼
Data Extensions
      │
      ▼
Automation Studio
      │
      ├── Eligibility
      ├── Data Preparation
      ├── Product Selection
      └── Product Ranking
      │
      ▼
Journey Entry Data Extension
      │
      ▼
Journey Builder
      │
      ▼
Email Activity
      │
      ▼
Content Builder
      │
      ▼
AMPscript
      │
      ├── Customer Data
      ├── Product Data
      └── Dynamic Product Rendering
      │
      ▼
Personalized Welcome Email
```

Essa separação reduz a quantidade de lógica executada durante a renderização do e-mail e facilita manutenção, testes e evolução da solução.

---

## 🔄 Fluxo da Jornada

De forma simplificada, o processamento segue estas etapas:

```text
Novo cliente
     │
     ▼
Validação de elegibilidade
     │
     ▼
Preparação dos dados
     │
     ▼
Seleção dos produtos
     │
     ▼
Ranking das recomendações
     │
     ▼
Data Extension de entrada
     │
     ▼
Journey Builder
     │
     ▼
E-mail de boas-vindas
     │
     ▼
Vitrine personalizada
```

O cliente entra na jornada somente depois que os dados necessários para a comunicação estão preparados.

---

## 🗃️ Estrutura de Dados

A solução utiliza Data Extensions com responsabilidades distintas.

Os nomes apresentados neste case são genéricos e foram adaptados para fins de documentação.

### Customer Data

Responsável pelos dados utilizados para identificação e personalização do cliente.

Exemplo conceitual:

| Campo | Descrição |
|---|---|
| CustomerID | Identificador do cliente |
| EmailAddress | Endereço de e-mail |
| FirstName | Primeiro nome |
| EntryDate | Data de entrada |
| Segment | Segmento ou classificação |

---

### Product Ranking

Armazena os produtos disponíveis ou recomendados para cada cliente.

Exemplo:

| Campo | Descrição |
|---|---|
| CustomerID | Identificador do cliente |
| ProductID | Identificador do produto |
| ProductName | Nome do produto |
| ProductImage | URL da imagem |
| ProductURL | URL de destino |
| Price | Valor do produto |
| Ranking | Prioridade da recomendação |

O campo de ranking permite controlar quais produtos devem aparecer primeiro na comunicação.

---

### Journey Entry

Contém os clientes preparados para entrada no Journey Builder.

Exemplo:

| Campo | Descrição |
|---|---|
| CustomerID | Identificador do cliente |
| EmailAddress | Endereço de e-mail |
| FirstName | Nome utilizado na personalização |
| EntryDate | Data de entrada |
| RecommendationKey | Referência utilizada na recuperação dos produtos |

A Data Extension de entrada contém somente os dados necessários para a jornada e para localizar as informações complementares da personalização.

---

## ⚙️ Automation Studio

O Automation Studio é utilizado como camada de preparação dos dados.

Antes da entrada no Journey Builder, a automação pode executar processos como:

1. identificar novos clientes;
2. validar critérios de elegibilidade;
3. recuperar produtos disponíveis;
4. relacionar produtos ao cliente;
5. aplicar regras de priorização;
6. gerar o ranking das recomendações;
7. preparar a Data Extension de entrada.

Esse modelo evita transferir processamento desnecessário para o momento da renderização do e-mail.

---

## 🔎 Ranking de Produtos

Quando existem diversos produtos possíveis para um cliente, a solução precisa determinar quais devem aparecer na comunicação.

Uma estratégia conceitual utilizando SQL é:

```sql
SELECT
    CustomerID,
    ProductID,
    ProductName,
    ProductImage,
    ProductURL,
    Price,
    ROW_NUMBER() OVER (
        PARTITION BY CustomerID
        ORDER BY RecommendationScore DESC
    ) AS Ranking
FROM ProductRecommendations
```

O exemplo acima é apenas ilustrativo.

A regra real de priorização pode considerar diferentes critérios de negócio.

O resultado permite recuperar posteriormente, por exemplo:

```text
Produto 1 → Ranking 1
Produto 2 → Ranking 2
Produto 3 → Ranking 3
Produto 4 → Ranking 4
```

---

## 🧭 Journey Builder

Após a preparação dos dados, os contatos elegíveis entram no Journey Builder.

A jornada é responsável principalmente por:

- receber os contatos;
- controlar a sequência de comunicação;
- executar atividades de espera quando necessário;
- avaliar decisões;
- direcionar o cliente para diferentes caminhos;
- executar o envio do e-mail.

A lógica pesada de dados permanece fora da jornada.

Essa separação mantém a orquestração mais simples e facilita a manutenção.

---

## ✉️ Content Builder

O e-mail utiliza um bloco de conteúdo responsável pela montagem da vitrine personalizada.

O conteúdo visual é separado da lógica de seleção dos produtos.

De forma conceitual:

```text
Email
 │
 ├── Header
 │
 ├── Welcome Content
 │
 ├── Dynamic Product Block
 │      │
 │      ├── Product 1
 │      ├── Product 2
 │      ├── Product 3
 │      └── Product 4
 │
 └── Footer
```

O bloco pode ser reutilizado e evoluído sem necessidade de reconstruir toda a comunicação.

---

## 🧩 AMPscript

AMPscript é utilizado para recuperar e preparar os produtos que serão apresentados no e-mail.

Um fluxo simplificado seria:

```ampscript
%%[
SET @customerId = AttributeValue("CustomerID")

/*
  Recuperação dos produtos relacionados ao cliente.
  Código simplificado para documentação pública.
*/

SET @productCount = RowCount(@products)
]%%
```

Depois da recuperação dos dados, o conteúdo pode decidir qual estrutura visual utilizar.

---

## 🖼️ Vitrine Dinâmica

Um dos principais pontos da solução é permitir que o e-mail se adapte à quantidade de produtos disponíveis.

Exemplo:

```text
1 produto
┌──────────────────────┐
│      Produto 1       │
└──────────────────────┘
```

```text
2 produtos
┌──────────┬──────────┐
│ Produto 1│ Produto 2│
└──────────┴──────────┘
```

```text
3 produtos
┌───────┬───────┬───────┐
│ Prod 1│ Prod 2│ Prod 3│
└───────┴───────┴───────┘
```

```text
4 produtos
┌──────────┬──────────┐
│ Produto 1│ Produto 2│
├──────────┼──────────┤
│ Produto 3│ Produto 4│
└──────────┴──────────┘
```

Assim, o mesmo bloco pode atender diferentes cenários sem exigir uma comunicação separada para cada quantidade de produtos.

---

## 📦 Uso de JSON

Em determinados cenários, as informações dos produtos também podem ser disponibilizadas em formato JSON.

Exemplo fictício:

```json
[
  {
    "id": "PROD001",
    "name": "Produto A",
    "price": 149.90,
    "image": "https://example.com/product-a.jpg",
    "url": "https://example.com/product-a"
  },
  {
    "id": "PROD002",
    "name": "Produto B",
    "price": 219.90,
    "image": "https://example.com/product-b.jpg",
    "url": "https://example.com/product-b"
  }
]
```

Esse modelo permite transportar múltiplos produtos dentro de uma única estrutura e processá-los dinamicamente durante a personalização.

Quando necessário, o conteúdo pode transformar o JSON em uma coleção de linhas para percorrer os produtos durante a renderização.

---

## 🧠 Separação de Responsabilidades

Uma decisão importante da arquitetura é evitar concentrar toda a solução dentro do e-mail.

### Automation Studio

Responsável por:

- processamento de dados;
- elegibilidade;
- relacionamento cliente/produto;
- priorização;
- ranking;
- preparação da entrada.

### Journey Builder

Responsável por:

- orquestração;
- sequência da comunicação;
- decisões;
- esperas;
- envio.

### Content Builder + AMPscript

Responsáveis por:

- personalização;
- recuperação das informações necessárias;
- formatação;
- renderização dinâmica da vitrine.

Essa divisão torna a solução mais fácil de testar, manter e evoluir.

---

## 🛡️ Tratamento de Cenários

Uma comunicação dinâmica também precisa considerar situações em que os dados disponíveis não são exatamente os esperados.

Exemplos:

```text
Nenhum produto
      │
      └── Conteúdo alternativo / vitrine não exibida

1 produto
      │
      └── Layout individual

2 produtos
      │
      └── Layout de duas colunas

3 produtos
      │
      └── Layout adaptado

4 ou mais produtos
      │
      └── Exibição limitada aos produtos priorizados
```

Isso evita que a estrutura visual do e-mail dependa de uma quantidade fixa de recomendações.

---

## 🧪 Validação

Antes da ativação da comunicação, alguns pontos importantes precisam ser validados:

- elegibilidade do cliente;
- disponibilidade dos produtos;
- ordenação do ranking;
- URLs das imagens;
- links dos produtos;
- valores e formatação;
- quantidade de produtos;
- renderização para diferentes cenários;
- comportamento responsivo do e-mail;
- fallback quando dados estiverem ausentes.

Preview e testes com diferentes perfis ajudam a validar as variações antes da ativação.

---

## ⚠️ Desafios Técnicos

### Quantidade variável de produtos

O conteúdo não poderia depender de uma quantidade fixa de recomendações.

**Abordagem:** criação de uma estrutura dinâmica capaz de adaptar o layout conforme o número de produtos recuperados.

---

### Priorização das recomendações

Um cliente poderia possuir mais produtos elegíveis do que o espaço disponível no e-mail.

**Abordagem:** preparação e ranking dos produtos antes da renderização, limitando a comunicação às recomendações prioritárias.

---

### Complexidade no momento do envio

Executar toda a lógica de negócio durante a renderização aumentaria a complexidade do conteúdo.

**Abordagem:** mover processamento, elegibilidade e priorização para o Automation Studio, deixando o AMPscript focado principalmente na personalização e apresentação.

---

### Dados estruturados

Produtos podem possuir múltiplos atributos, como nome, imagem, preço e URL.

**Abordagem:** utilização de estruturas tabulares ou JSON conforme a necessidade do fluxo, mantendo a camada de apresentação desacoplada da origem dos dados.

---

## 💡 Principais Aprendizados

Este case reforça alguns princípios importantes em soluções de personalização no Marketing Cloud:

### Preparar antes de personalizar

Quanto mais informação puder ser organizada antes do envio, menor será a complexidade durante a renderização.

### Separar dados de apresentação

Regras de elegibilidade e priorização não precisam estar misturadas ao HTML do e-mail.

### Projetar para variação

Conteúdo dinâmico deve considerar desde o início que a quantidade e a disponibilidade dos dados podem mudar.

### Reutilizar componentes

Blocos reutilizáveis no Content Builder reduzem duplicação e facilitam evolução da comunicação.

### Personalização vai além do nome

Uma jornada personalizada pode utilizar comportamento, produtos, ranking e contexto para modificar efetivamente o conteúdo entregue ao cliente.

---

## 🧰 Tecnologias Utilizadas

- Salesforce Marketing Cloud Engagement
- Automation Studio
- Journey Builder
- Content Builder
- Data Extensions
- SQL
- AMPscript
- HTML
- CSS
- JSON

---

## 🔐 Confidencialidade

Este case é baseado em uma experiência profissional real e foi **adaptado e anonimizado para fins de portfólio**.

Nomes de clientes, Data Extensions, campos, dados, regras proprietárias, identificadores e trechos específicos da implementação original foram removidos ou substituídos por exemplos genéricos.

Os exemplos apresentados têm finalidade exclusivamente técnica e demonstrativa.

---

## 👨‍💻 Competências Demonstradas

Este projeto demonstra experiência em:

- arquitetura de soluções no Salesforce Marketing Cloud;
- automação e processamento de dados;
- SQL para preparação e priorização de informações;
- Journey Builder;
- modelagem com Data Extensions;
- desenvolvimento de conteúdo dinâmico;
- AMPscript;
- personalização baseada em produtos;
- tratamento de JSON;
- desenvolvimento de e-mails dinâmicos;
- separação de responsabilidades entre dados, orquestração e apresentação.

---

## 🔗 Outros Cases

Este projeto faz parte da coleção:

**Salesforce Marketing Cloud Cases**

Outros estudos incluem:

- Abandoned Cart Automation
- Omnichannel Consent Management
- Marketing Reporting & Data Consolidation

---

> Este repositório apresenta estudos técnicos baseados em experiências profissionais reais, com informações sensíveis e proprietárias removidas ou adaptadas.
