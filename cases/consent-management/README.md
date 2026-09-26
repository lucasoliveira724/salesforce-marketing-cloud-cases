# 🔐 Omnichannel Consent Management

> Case técnico de consolidação de opt-in e opt-out de múltiplos canais utilizando Salesforce Marketing Cloud Engagement.

---

## 📌 Visão Geral

Este case apresenta uma solução para **centralização e consolidação do status de consentimento de clientes em diferentes canais de comunicação** dentro do Salesforce Marketing Cloud Engagement.

Em uma arquitetura multicanal, o status de comunicação de um cliente pode estar distribuído entre diferentes Data Views, estruturas internas da plataforma e Data Extensions.

A solução foi desenvolvida para consolidar essas informações em uma estrutura única, permitindo identificar o status de comunicação do cliente para canais como:

- Email
- SMS
- Push
- WhatsApp

O resultado é uma visão centralizada que pode ser utilizada por processos internos, integrações e sistemas externos.

---

## 🎯 Objetivo

Criar uma estrutura capaz de:

- consultar os status de comunicação existentes no Marketing Cloud;
- identificar opt-in e opt-out por canal;
- consolidar essas informações por cliente;
- registrar datas relevantes de opt-out;
- normalizar diferentes estruturas de dados;
- disponibilizar o resultado em uma Data Extension única;
- preparar os dados para consumo por integrações externas.

A proposta é transformar informações distribuídas pela plataforma em uma visão consistente de consentimento.

---

## 🧩 Desafio

Cada canal possui características próprias para armazenamento e consulta de informações.

Conceitualmente:

```text
Email
  │
  └── Subscriber Status / Unsubscribe

SMS
  │
  └── Mobile Subscription

Push
  │
  └── Push Address / Subscriber

WhatsApp
  │
  └── Messaging / Data Extensions

        ↓

Estruturas diferentes
        ↓
Normalização necessária
        ↓
Visão única por cliente
```

Isso significa que não existe necessariamente uma única origem capaz de representar todo o relacionamento de comunicação do cliente.

---

## 🏗️ Arquitetura da Solução

Fluxo conceitual:

```text
Salesforce Marketing Cloud
        │
        ├── Email
        │     ├── Subscribers
        │     └── Business Unit Unsubscribes
        │
        ├── SMS
        │     └── SMS Subscription
        │
        ├── Push
        │     ├── Push Address
        │     └── Subscribers
        │
        └── WhatsApp
              └── Messaging Data Extensions
                    │
                    ▼
             Automation Studio
                    │
             SQL Processing
                    │
             Data Normalization
                    │
                    ▼
        Unified Consent Data Extension
                    │
                    ▼
           Integration / API Layer
```

O Automation Studio funciona como camada de consolidação e normalização dos diferentes canais.

---

## 🗃️ Modelo Consolidado

A solução utiliza uma Data Extension central para representar o status multicanal do cliente.

Estrutura conceitual:

| Campo | Descrição |
|---|---|
| CustomerID | Identificador único do cliente |
| EmailAddress | Endereço de e-mail |
| DocumentID | Identificador de negócio |
| EmailOptIn | Status de comunicação por e-mail |
| EmailOptOutDate | Data do opt-out de e-mail |
| SMSOptIn | Status de comunicação por SMS |
| SMSOptOutDate | Data do opt-out de SMS |
| WhatsAppOptIn | Status de comunicação por WhatsApp |
| WhatsAppOptOutDate | Data do opt-out de WhatsApp |
| PushOptIn | Status de comunicação por Push |
| PushOptOutDate | Data do opt-out de Push |
| UpdatedAt | Data da última atualização |
| IntegrationStatus | Status de processamento para integração |

Os nomes apresentados são genéricos e foram adaptados para documentação pública.

---

## ✉️ Email

O status de e-mail pode depender de informações existentes nas estruturas de Subscribers e dos registros de unsubscribe.

Exemplo conceitual:

```text
Subscribers
      +
Business Unit Unsubscribes
      │
      ▼
Determinação do status
      │
      ▼
EmailOptIn
EmailOptOutDate
```

A lógica precisa considerar corretamente o escopo da conta e das Business Units envolvidas.

Um cliente pode existir na base global de subscribers enquanto possui um contexto específico de unsubscribe em determinada unidade.

---

## 📱 SMS

Para SMS, a solução consulta informações relacionadas às assinaturas do MobileConnect.

Conceitualmente:

```text
SMS Subscription Log
        │
        ├── Mobile Number
        ├── Subscriber Key
        ├── Opt-In Status
        ├── Opt-Out Status
        ├── Opt-In Date
        └── Opt-Out Date
                │
                ▼
            SMSOptIn
        SMSOptOutDate
```

Como podem existir múltiplos registros relacionados ao mesmo contato, é necessário determinar qual informação representa o estado relevante para a consolidação.

---

## 🔔 Push

O canal Push possui uma estrutura diferente dos canais tradicionais.

A consolidação pode combinar informações relacionadas ao endereço Push com a identificação do subscriber.

Fluxo conceitual:

```text
Push Address
      +
Subscribers
      │
      ▼
Relacionamento por identificador
      │
      ▼
PushOptIn
PushOptOutDate
```

Essa etapa exige atenção especial ao relacionamento entre o identificador utilizado pelo dispositivo e o identificador do cliente.

---

## 💬 WhatsApp

Em determinados cenários, informações relacionadas ao WhatsApp podem estar disponíveis em Data Extensions utilizadas pelos processos de mensageria.

Conceitualmente:

```text
WhatsApp Send Data
        +
WhatsApp Status History
        │
        ▼
Identificação do cliente
        │
        ▼
Determinação do status
        │
        ▼
WhatsAppOptIn
WhatsAppOptOutDate
```

Uma característica importante é que contatos presentes nos processos de WhatsApp podem não possuir necessariamente o mesmo histórico ou representação utilizada em outros canais.

Por isso, o modelo não deve assumir que todos os clientes existem previamente em uma única estrutura global.

---

## ⚙️ Automation Studio

A consolidação é executada por uma automação responsável por processar periodicamente as diferentes origens.

Fluxo simplificado:

```text
Automation Start
      │
      ├── Email Processing
      │
      ├── SMS Processing
      │
      ├── Push Processing
      │
      ├── WhatsApp Processing
      │
      ▼
Data Normalization
      │
      ▼
Unified Consent Data Extension
      │
      ▼
Integration Status
```

Separar o processamento por canal facilita manutenção e diagnóstico.

Se uma origem sofrer alterações, a lógica daquele canal pode ser ajustada sem reconstruir toda a solução.

---

## 🧮 Normalização dos Status

Como diferentes origens podem representar status de formas distintas, a solução utiliza uma camada de normalização.

Conceitualmente:

```text
Status original
      │
      ▼
Regra específica do canal
      │
      ▼
Status padronizado
      │
      ├── 1 = Opt-In
      └── 0 = Opt-Out
```

Também é importante definir o comportamento quando não existe informação suficiente para determinado canal.

Dependendo da regra de negócio, ausência de informação não deve ser automaticamente interpretada como consentimento.

---

## 🧹 Tratamento de Valores Nulos

Durante a consolidação podem existir clientes sem informação para determinados canais.

Exemplo:

```text
Customer A

Email       → Opt-In
SMS         → Opt-In
WhatsApp    → Sem informação
Push        → Sem informação
```

Uma etapa específica de saneamento pode ser utilizada para garantir que a estrutura final siga o padrão esperado pelo processo consumidor.

Essa regra deve ser definida de acordo com a política de consentimento e os requisitos da integração.

---

## 🔄 Preparação para Integração

Além dos status por canal, a estrutura pode possuir um campo responsável por controlar o processamento por sistemas externos.

Exemplo:

```text
IntegrationStatus
```

Possíveis estados:

```text
Pending
Processed
Error
```

Fluxo:

```text
Marketing Cloud
      │
      ▼
Consent Consolidation
      │
      ▼
IntegrationStatus = Pending
      │
      ▼
External Integration
      │
      ▼
IntegrationStatus = Processed
```

Isso permite separar a geração da informação do processo responsável por consumi-la.

---

## 🔍 Exemplo Conceitual

Após a consolidação:

| CustomerID | Email | SMS | WhatsApp | Push |
|---|---:|---:|---:|---:|
| C001 | 1 | 1 | 0 | 1 |
| C002 | 1 | 0 | 1 | 0 |
| C003 | 0 | 1 | 1 | 1 |

O exemplo utiliza dados fictícios apenas para demonstrar o modelo.

---

## ⚠️ Desafios Técnicos

### Diferentes fontes por canal

Cada canal possui sua própria estrutura de armazenamento.

**Abordagem:** criação de processos independentes de extração e normalização antes da consolidação.

---

### Identificação do cliente

Nem todas as fontes utilizam necessariamente o mesmo identificador.

**Abordagem:** estabelecer uma chave de cliente comum sempre que possível e realizar os relacionamentos necessários durante o processamento.

---

### Múltiplos registros

Um mesmo cliente pode possuir diversos eventos ou registros de assinatura.

**Abordagem:** aplicar regras de agregação e seleção para determinar o estado relevante.

---

### Contatos fora da estrutura tradicional de Subscribers

Alguns canais podem possuir contatos que não aparecem da mesma maneira nas estruturas utilizadas por Email.

**Abordagem:** não utilizar uma única Data View como universo absoluto da solução.

---

### Valores ausentes

A ausência de um registro não representa necessariamente opt-in ou opt-out.

**Abordagem:** definir explicitamente as regras de normalização e saneamento de acordo com os requisitos do negócio.

---

## 🛡️ Consentimento e Preferências

Uma distinção importante em arquiteturas desse tipo é separar:

```text
Preferência de comunicação
            ≠
Registro técnico de envio
            ≠
Consentimento de negócio
```

Uma plataforma de marketing pode armazenar estados técnicos necessários para comunicação, enquanto sistemas corporativos podem possuir regras adicionais de consentimento e preferência.

Por isso, a arquitetura deve definir claramente qual sistema representa a fonte oficial de cada informação.

---

## 💡 Principais Aprendizados

### Multicanal exige normalização

Email, SMS, Push e WhatsApp possuem estruturas diferentes. Uma visão consolidada precisa abstrair essas diferenças.

### Não assumir uma única população

Utilizar apenas uma fonte como universo de clientes pode excluir contatos existentes em outros canais.

### Separar extração de consolidação

Processar cada canal individualmente torna a solução mais simples de manter.

### Ausência de informação precisa ser tratada explicitamente

`NULL`, opt-out e opt-in representam situações diferentes e não devem ser confundidos sem uma regra de negócio clara.

### Integração precisa de controle de estado

Campos de processamento permitem identificar registros pendentes, processados ou com erro sem misturar essa responsabilidade com a captura do consentimento.

---

## 🧰 Tecnologias Utilizadas

- Salesforce Marketing Cloud Engagement
- Automation Studio
- Data Extensions
- SQL
- Marketing Cloud Data Views
- Email Studio
- MobileConnect
- MobilePush
- WhatsApp / Messaging Data
- Integrações externas

---

## 🔐 Confidencialidade

Este case é baseado em uma experiência profissional real e foi **adaptado e anonimizado para fins de portfólio**.

Nomes de clientes, Data Extensions, campos, identificadores, regras proprietárias, dados pessoais e detalhes específicos da implementação original foram removidos ou substituídos por estruturas genéricas.

Nenhum dado real de cliente é apresentado neste documento.

---

## 👨‍💻 Competências Demonstradas

Este projeto demonstra experiência em:

- arquitetura de dados no Salesforce Marketing Cloud;
- Automation Studio;
- SQL;
- Marketing Cloud Data Views;
- modelagem de Data Extensions;
- Email;
- SMS;
- Push;
- WhatsApp;
- consolidação multicanal;
- normalização de dados;
- tratamento de consentimento;
- preparação de dados para integrações externas.

---

## 🔗 Outros Cases

Este projeto faz parte da coleção:

**Salesforce Marketing Cloud Cases**

Outros estudos incluem:

- Abandoned Cart Automation
- Welcome Journey & Dynamic Product Personalization
- Marketing Reporting & Data Consolidation

---

> Este repositório apresenta estudos técnicos baseados em experiências profissionais reais, com informações sensíveis e proprietárias removidas ou adaptadas.
