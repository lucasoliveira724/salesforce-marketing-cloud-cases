<div align="center">

# 🛒 Abandoned Cart Automation

### Salesforce Marketing Cloud Technical Case

**Automação • Elegibilidade • Controle de Reentrada • Personalização**

Solução para processamento de eventos de abandono de carrinho, controle de elegibilidade e prevenção de reentrada utilizando **Salesforce Marketing Cloud Engagement**.

![Salesforce](https://img.shields.io/badge/Salesforce-Marketing%20Cloud-0176D3?style=for-the-badge&logo=salesforce&logoColor=white)
![Journey Builder](https://img.shields.io/badge/Journey%20Builder-Customer%20Journey-00A1E0?style=for-the-badge)
![Automation Studio](https://img.shields.io/badge/Automation%20Studio-Automation-6C63FF?style=for-the-badge)
![SQL](https://img.shields.io/badge/SQL-Data%20Processing-4479A1?style=for-the-badge)
![AMPscript](https://img.shields.io/badge/AMPscript-Personalization-FF6B6B?style=for-the-badge)

</div>

---

# 📌 Visão Geral

Este case apresenta uma arquitetura de **Carrinho Abandonado** desenvolvida utilizando Salesforce Marketing Cloud Engagement.

O objetivo da solução é identificar eventos de abandono, validar a elegibilidade do cliente, controlar sua reentrada e disponibilizar os dados necessários para uma jornada de comunicação personalizada.

Um dos principais desafios da solução está no controle de recorrência.

Um mesmo cliente pode gerar diferentes eventos de abandono para um conjunto de produtos semelhante. Utilizar somente o identificador do carrinho ou do evento não seria suficiente para controlar a reentrada.

Por isso, a arquitetura utiliza uma combinação entre:

**Cliente + conjunto de produtos + janela de controle**

para determinar se uma nova entrada deve ser permitida.

---

# 🎯 Problema

Considere o seguinte cenário:

```text
Cliente
   ↓
Adiciona produtos ao carrinho
   ↓
Não conclui a compra
   ↓
Evento de abandono
   ↓
Marketing Cloud
   ↓
Jornada de recuperação
```

Até aqui, temos um fluxo tradicional de carrinho abandonado.

O problema surge quando o cliente abandona novamente os mesmos produtos.

```text
Primeiro abandono

CustomerID: 10001
Products: A,B,C
EventID: CART-001

        ↓

Segundo abandono

CustomerID: 10001
Products: A,B,C
EventID: CART-002
```

Os eventos possuem identificadores diferentes.

Porém, do ponto de vista da estratégia de comunicação, representam o **mesmo contexto de abandono**.

Se o controle fosse realizado somente utilizando `EventID`, o cliente poderia entrar novamente na jornada e receber comunicações repetidas.

---

# 💡 Estratégia da Solução

A solução cria uma camada de controle entre o evento recebido e a entrada no Journey Builder.

```text
Evento de Abandono
        │
        ▼
AbandonedCart_Staging
        │
        ▼
Automation Studio
        │
        ▼
Normalização dos Dados
        │
        ▼
Validação de Elegibilidade
        │
        ├───────────────┐
        │               │
   Elegível          Bloqueado
        │               │
        ▼               ▼
Journey Entry       History
        │
        ▼
Journey Builder
        │
        ▼
Personalização
        │
        ▼
Comunicação
```

A jornada recebe apenas registros previamente considerados elegíveis.

Dessa forma, a responsabilidade pelo controle não fica concentrada no Journey Builder.

---

# 🏗️ Arquitetura

A solução utiliza quatro estruturas principais de dados.

```text
┌──────────────────────────────┐
│ AbandonedCart_Staging        │
│                              │
│ Eventos recebidos            │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│ Automation Studio            │
│                              │
│ Processamento e validações   │
└───────────────┬──────────────┘
                │
       ┌────────┴────────┐
       │                 │
       ▼                 ▼
┌──────────────┐  ┌───────────────────┐
│ JourneyEntry │  │ Control           │
│              │  │                   │
│ Elegíveis    │  │ Reentrada         │
└──────┬───────┘  └───────────────────┘
       │
       ▼
┌──────────────────────────────┐
│ Journey Builder              │
└───────────────┬──────────────┘
                │
                ▼
        Comunicação

                +

┌──────────────────────────────┐
│ History                      │
│                              │
│ Aprovados / Bloqueados       │
└──────────────────────────────┘
```

---

# 🗄️ Modelo de Dados

Para fins deste case, os nomes das estruturas foram simplificados e anonimizados.

## 1. AbandonedCart_Staging

Responsável por receber os eventos de abandono.

Exemplo conceitual:

| Campo | Descrição |
|---|---|
| `CustomerID` | Identificador do cliente |
| `EmailAddress` | Endereço de e-mail |
| `EventID` | Identificador do evento/carrinho |
| `ProductIDs` | Produtos presentes no abandono |
| `ProductsJSON` | Informações dos produtos |
| `EventDate` | Data do evento |
| `Processed` | Status de processamento |

---

## 2. AbandonedCart_JourneyEntry

Contém apenas clientes aprovados para entrada na jornada.

| Campo | Descrição |
|---|---|
| `EntryKey` | Chave única da entrada |
| `CustomerID` | Cliente |
| `EmailAddress` | E-mail |
| `ProductIDs` | Produtos |
| `ProductsJSON` | Dados para personalização |
| `EntryDate` | Data de entrada |

---

## 3. AbandonedCart_Control

Responsável pelo controle de reentrada.

| Campo | Descrição |
|---|---|
| `ControlKey` | Chave de controle |
| `CustomerID` | Cliente |
| `ProductIDs` | Conjunto de produtos |
| `FirstEntryDate` | Primeira entrada |
| `LastEntryDate` | Última entrada |
| `ExpirationDate` | Expiração do controle |
| `Status` | Situação do controle |

---

## 4. AbandonedCart_History

Mantém o histórico das decisões tomadas pelo processamento.

| Campo | Descrição |
|---|---|
| `HistoryID` | Identificador do histórico |
| `CustomerID` | Cliente |
| `EventID` | Evento |
| `ProductIDs` | Produtos |
| `Action` | Aprovado ou Bloqueado |
| `ActionDate` | Data da decisão |
| `Observation` | Informação adicional |

---

# 🔑 Estratégia de Chaves

A definição das chaves é uma parte importante da arquitetura.

## EntryKey

Identifica uma entrada específica na jornada.

Conceitualmente:

```text
CustomerID + ProductIDs + EventID
```

Exemplo:

```text
10001_A-B-C_CART-001
```

---

## ControlKey

Representa a combinação utilizada para controlar recorrência.

```text
CustomerID + ProductIDs
```

Exemplo:

```text
10001_A-B-C
```

Assim, eventos diferentes podem ser reconhecidos como pertencentes ao mesmo contexto de abandono.

---

# 🧠 Normalização dos Produtos

Para que a comparação funcione corretamente, os produtos precisam possuir representação consistente.

Por exemplo:

```text
A,B,C
```

e:

```text
C,A,B
```

podem representar exatamente o mesmo conjunto de produtos.

Por isso, antes da geração da chave de controle, os identificadores podem ser normalizados e ordenados.

Resultado:

```text
A,B,C
```

Essa etapa evita que alterações apenas na ordem dos produtos sejam interpretadas como um novo contexto de abandono.

---

# 🔐 Controle de Reentrada

A regra conceitual é:

```text
CustomerID
      +
ProductIDs normalizados
      ↓
Existe controle ativo?
```

### Não

```text
Aprovar entrada
      ↓
JourneyEntry
      ↓
Criar/atualizar Control
      ↓
Registrar History = APPROVED
```

### Sim

```text
Bloquear entrada
      ↓
Não enviar para JourneyEntry
      ↓
Registrar History = BLOCKED
```

---

# ⏱️ Janela de Controle

A arquitetura permite definir uma janela durante a qual uma combinação cliente + produtos permanece bloqueada.

Exemplo conceitual:

```text
Primeira entrada
25/09

        ↓

Control ativo

        ↓

ExpirationDate
02/10
```

Durante esse período:

```text
CustomerID + ProductIDs
```

não poderá gerar uma nova entrada.

Após a expiração, o cliente volta a ser elegível.

> O período apresentado neste case é apenas ilustrativo e pode ser configurado conforme a estratégia de negócio.

---

# ⚙️ Automation Studio

O processamento é realizado de forma automatizada utilizando **Automation Studio**.

Fluxo conceitual:

```text
STEP 1
Identificar eventos elegíveis

        ↓

STEP 2
Atualizar controle

        ↓

STEP 3
Registrar aprovados

        ↓

STEP 4
Registrar bloqueados

        ↓

STEP 5
Marcar eventos processados
```

A separação em etapas facilita:

- manutenção;
- troubleshooting;
- auditoria;
- evolução das regras.

---

# 🔎 Elegibilidade

A lógica de elegibilidade verifica se já existe um controle ativo para aquela combinação.

Exemplo simplificado:

```sql
SELECT
    S.CustomerID,
    S.EmailAddress,
    S.EventID,
    S.ProductIDs,
    S.ProductsJSON

FROM AbandonedCart_Staging S

WHERE S.Processed = 0

AND NOT EXISTS (

    SELECT 1

    FROM AbandonedCart_Control C

    WHERE C.CustomerID = S.CustomerID
      AND C.ProductIDs = S.ProductIDs
      AND C.Status = 'ACTIVE'
      AND C.ExpirationDate > GETDATE()

)
```

> O SQL acima é demonstrativo e foi simplificado para fins de documentação. Ele não representa código proprietário de nenhum ambiente real.

---

# 🚦 Tratamento de Eventos Simultâneos

Outro cenário relevante ocorre quando múltiplos eventos semelhantes estão aguardando processamento.

Exemplo:

```text
CustomerID  ProductIDs  EventID
10001       A,B,C       CART-001
10001       A,B,C       CART-002
10001       A,B,C       CART-003
```

Sem tratamento adicional, todos poderiam ser considerados elegíveis no mesmo ciclo.

Uma estratégia é utilizar uma função de janela para selecionar apenas um evento por combinação.

Exemplo conceitual:

```sql
ROW_NUMBER() OVER (
    PARTITION BY CustomerID, ProductIDs
    ORDER BY EventDate
)
```

Somente:

```text
ROW_NUMBER = 1
```

segue para processamento.

Isso evita entradas duplicadas durante uma mesma execução.

---

# 📜 Histórico

Uma Data Extension específica registra as decisões.

Exemplo:

```text
CustomerID | EventID  | Products | Action
-----------|----------|----------|---------
10001      | CART-001 | A,B,C    | APPROVED
10001      | CART-002 | A,B,C    | BLOCKED
10002      | CART-003 | D,E      | APPROVED
```

Essa estrutura ajuda em:

- auditoria;
- troubleshooting;
- análise de comportamento;
- validação das regras;
- monitoramento da automação.

---

# 🚀 Journey Builder

Somente registros aprovados chegam à Data Extension utilizada como fonte da jornada.

```text
AbandonedCart_JourneyEntry
            │
            ▼
       Journey Builder
            │
            ▼
       Comunicação
```

Isso mantém o Journey Builder focado na **orquestração da comunicação**, enquanto regras mais complexas de dados e elegibilidade permanecem na camada de processamento.

---

# 🎨 Personalização

Os dados de produtos podem ser enviados para a Journey Entry utilizando uma estrutura JSON.

Exemplo fictício:

```json
[
  {
    "id": "PRODUCT-001",
    "name": "Produto A",
    "price": 199.90,
    "image": "https://example.com/product-a.jpg"
  },
  {
    "id": "PRODUCT-002",
    "name": "Produto B",
    "price": 89.90,
    "image": "https://example.com/product-b.jpg"
  }
]
```

Esse conteúdo pode ser utilizado pelo **AMPscript** para construir dinamicamente a vitrine apresentada no e-mail.

Conceitualmente:

```text
ProductsJSON
      ↓
AMPscript
      ↓
Parse dos produtos
      ↓
HTML dinâmico
      ↓
Vitrine personalizada
```

A quantidade de produtos apresentada pode ser controlada pelo próprio bloco de conteúdo.

---

# 🔄 Fluxo Completo

```text
Sistema de Origem
        │
        ▼
Evento de Carrinho Abandonado
        │
        ▼
AbandonedCart_Staging
        │
        ▼
Automation Studio
        │
        ├── Normalização
        │
        ├── Deduplicação
        │
        ├── Elegibilidade
        │
        └── Controle de Reentrada
        │
        ▼
┌───────────────────────────┐
│ Cliente elegível?         │
└─────────────┬─────────────┘
              │
       ┌──────┴──────┐
       │             │
      SIM           NÃO
       │             │
       ▼             ▼
JourneyEntry       History
       │          BLOCKED
       ▼
Control
       │
       ▼
History
APPROVED
       │
       ▼
Journey Builder
       │
       ▼
AMPscript
       │
       ▼
Conteúdo personalizado
       │
       ▼
Comunicação
```

---

# 🧩 Separação de Responsabilidades

Uma das principais decisões da arquitetura foi separar as responsabilidades.

### Automation Studio

Responsável por:

- processamento;
- normalização;
- deduplicação;
- elegibilidade;
- controle;
- histórico.

### Journey Builder

Responsável por:

- orquestração;
- fluxo de comunicação;
- tempos de espera;
- decisões relacionadas à jornada.

### Content Builder / AMPscript

Responsável por:

- personalização;
- interpretação dos dados de produtos;
- construção do conteúdo dinâmico.

Essa separação reduz o acoplamento e facilita manutenção e troubleshooting.

---

# 🛠️ Tecnologias Utilizadas

### Salesforce Marketing Cloud Engagement

`Journey Builder`

`Automation Studio`

`Email Studio`

`Content Builder`

`Data Extensions`

### Desenvolvimento

`SQL`

`AMPscript`

`SSJS`

`HTML`

### Dados

`JSON`

`Data Modeling`

`Data Processing`

`Deduplication`

---

# 💡 Principais Decisões Técnicas

### Não controlar reentrada somente pelo EventID

Eventos diferentes podem representar o mesmo abandono.

Por isso, o controle considera cliente + produtos.

---

### Normalizar produtos antes da comparação

A ordem dos produtos não deve gerar uma nova combinação quando o conjunto permanece o mesmo.

---

### Realizar elegibilidade antes do Journey Builder

A jornada recebe apenas contatos previamente aprovados.

---

### Manter histórico separado

A existência de uma estrutura de histórico facilita auditoria e investigação de problemas.

---

### Separar controle da entrada da jornada

A Data Extension de controle representa estado.

A Journey Entry representa eventos aprovados.

Misturar essas responsabilidades aumentaria a complexidade da solução.

---

# 📈 Benefícios da Arquitetura

A solução permite:

- reduzir comunicações repetidas;
- controlar reentrada de maneira determinística;
- manter histórico das decisões;
- separar regras de dados da orquestração;
- facilitar troubleshooting;
- permitir evolução das regras de negócio;
- personalizar comunicações com informações dos produtos.

---

# 🧠 Aprendizados

Este case demonstra como uma jornada aparentemente simples pode exigir uma camada adicional de arquitetura de dados.

O principal aprendizado é que:

> **O identificador técnico de um evento nem sempre representa o contexto de negócio que precisa ser controlado.**

Modelar corretamente esse contexto permite construir jornadas mais previsíveis, auditáveis e fáceis de manter.

---

# 🔒 Confidencialidade

Este case foi criado a partir de experiências profissionais reais, porém todo o conteúdo foi **anonimizado e adaptado exclusivamente para fins de portfólio**.

Foram removidos ou modificados:

- nomes de clientes;
- nomes de campanhas;
- nomes de Data Extensions;
- identificadores;
- dados pessoais;
- regras proprietárias;
- estruturas internas;
- códigos específicos do ambiente.

Os exemplos de dados, SQL, JSON e arquitetura apresentados nesta documentação possuem finalidade exclusivamente demonstrativa.

---

<div align="center">

### ☁️ Salesforce Marketing Cloud Technical Cases

[← Voltar para os cases](../../README.md)

</div>
