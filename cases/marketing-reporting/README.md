# 📊 Marketing Reporting & Data Consolidation

> Case técnico de consolidação e persistência histórica de métricas de campanhas utilizando Salesforce Marketing Cloud Engagement.

---

## 📌 Visão Geral

Este case apresenta uma solução desenvolvida para consolidar informações de performance de campanhas do **Salesforce Marketing Cloud Engagement** em uma estrutura persistente de dados.

O Marketing Cloud disponibiliza informações de tracking por meio de Data Views, permitindo consultar eventos como:

- envios;
- aberturas;
- cliques;
- bounces;
- unsubscribes.

Essas estruturas são extremamente úteis para análises operacionais, porém possuem retenção limitada.

A solução foi desenhada para coletar periodicamente essas informações e armazená-las em uma **Data Extension histórica**, criando uma base consolidada para consultas, relatórios e análises futuras.

---

## 🎯 Objetivo

Criar uma estrutura capaz de:

- capturar informações de envio;
- identificar o e-mail relacionado ao subscriber;
- consolidar aberturas;
- consolidar cliques;
- registrar primeiro e último evento;
- calcular quantidade de interações;
- identificar bounces;
- identificar unsubscribes;
- associar informações de campanha e jornada;
- preservar o histórico além da retenção das Data Views;
- disponibilizar uma base única para análises futuras.

O resultado é uma camada histórica de reporting construída dentro do próprio Marketing Cloud.

---

## 🧩 Problema

As informações de tracking ficam distribuídas entre diferentes Data Views.

Conceitualmente:

```text
_Sent
  │
  ├── envio
  │
_Open
  │
  ├── abertura
  │
_Click
  │
  ├── clique
  │
_Bounce
  │
  ├── bounce
  │
_Unsubscribe
  │
  └── unsubscribe
```

Além disso, outras estruturas podem ser necessárias para complementar informações como:

```text
Subscriber
Email
Job
Journey
Campaign
```

Consultar essas fontes diretamente sempre que um relatório é necessário aumenta a complexidade e limita análises históricas.

---

## 🏗️ Arquitetura da Solução

Fluxo conceitual:

```text
Salesforce Marketing Cloud
          │
          ├── _Sent
          ├── _Open
          ├── _Click
          ├── _Bounce
          ├── _Unsubscribe
          ├── _Subscribers
          └── Metadata
                 │
                 ▼
          Automation Studio
                 │
                 ▼
        SQL Query Activities
                 │
        ┌────────┼────────┐
        │        │        │
        ▼        ▼        ▼
      Send     Open     Click
        │        │        │
        ├────────┼────────┤
                 │
            Bounce
                 │
            Unsubscribe
                 │
                 ▼
       Consolidated Reporting
          Data Extension
                 │
                 ▼
        Reporting / Analytics
```

A automação executa cada etapa de enriquecimento de forma controlada.

---

## 🗃️ Data Extension Consolidada

A Data Extension final representa cada envio e suas respectivas métricas.

Estrutura conceitual:

| Campo | Descrição |
|---|---|
| ReportID | Identificador único do registro |
| JobID | Identificador do envio |
| BatchID | Identificador do batch |
| SubscriberKey | Identificador do contato |
| EmailAddress | Endereço de e-mail |
| SendDate | Data do envio |
| EmailName | Nome da comunicação |
| JourneyName | Jornada relacionada |
| Campaign | Campanha relacionada |
| Sent | Indicador de envio |
| Opened | Indicador de abertura |
| FirstOpenDate | Primeira abertura |
| LastOpenDate | Última abertura |
| OpenCount | Quantidade de aberturas |
| Clicked | Indicador de clique |
| FirstClickDate | Primeiro clique |
| LastClickDate | Último clique |
| ClickCount | Quantidade de cliques |
| Bounced | Indicador de bounce |
| BounceType | Tipo de bounce |
| BounceDate | Data do bounce |
| Unsubscribed | Indicador de unsubscribe |
| UnsubscribeDate | Data do unsubscribe |

Campos adicionais podem ser utilizados para classificar URLs, campanhas ou comportamentos específicos.

---

## 🔑 Identificação Única

Como um mesmo subscriber pode receber diferentes comunicações, utilizar apenas `SubscriberKey` não é suficiente para identificar um envio.

Uma chave conceitual pode combinar:

```text
JobID
+
BatchID
+
SubscriberKey
```

Exemplo:

```text
ReportID =
JobID_BatchID_SubscriberKey
```

Isso permite distinguir diferentes ocorrências de envio para o mesmo contato.

---

## ⚙️ Automation Studio

A solução utiliza uma sequência de Query Activities.

Fluxo simplificado:

```text
Automation Start
      │
      ▼
1. Process Sends
      │
      ▼
2. Resolve Subscriber Data
      │
      ▼
3. Update Email Information
      │
      ▼
4. Process Opens
      │
      ▼
5. Process Clicks
      │
      ▼
6. Process Bounces
      │
      ▼
7. Process Unsubscribes
      │
      ▼
Reporting Data Extension
```

Cada etapa possui uma responsabilidade específica.

Isso facilita diagnóstico, manutenção e evolução do processo.

---

## 📤 Etapa 1 — Envios

A primeira etapa utiliza os eventos de envio como base para criação dos registros.

Conceitualmente:

```sql
SELECT
    SubscriberKey,
    JobID,
    BatchID,
    EventDate AS SendDate
FROM _Sent
```

A partir dessas informações é criada a chave única do relatório.

Exemplo conceitual:

```sql
ReportID =
    JobID
    + '_'
    + BatchID
    + '_'
    + SubscriberKey
```

O envio funciona como registro principal sobre o qual os demais eventos serão posteriormente associados.

---

## 👤 Etapa 2 — Identificação do Subscriber

Nem sempre todas as informações necessárias estão presentes diretamente no evento de envio.

Por isso, uma etapa complementar pode recuperar dados do subscriber.

Fluxo:

```text
Reporting DE
     │
     ▼
SubscriberKey
     │
     ▼
_Subscribers
     │
     ▼
EmailAddress
```

Em cenários com grande volume, uma estrutura auxiliar pode ser utilizada para processar somente os registros que ainda precisam de enriquecimento.

---

## ✉️ Etapa 3 — Enriquecimento

Após localizar as informações complementares, os registros do relatório são atualizados.

Exemplo:

```text
ReportID
SubscriberKey
EmailAddress
EmailName
JourneyName
Campaign
```

Essa etapa evita repetir operações desnecessárias nas consultas posteriores.

---

## 👁️ Etapa 4 — Aberturas

Os eventos de abertura são consolidados para gerar métricas como:

```text
Opened
FirstOpenDate
LastOpenDate
OpenCount
```

Conceitualmente:

```sql
SELECT
    JobID,
    SubscriberKey,
    MIN(EventDate) AS FirstOpenDate,
    MAX(EventDate) AS LastOpenDate,
    COUNT(*) AS OpenCount
FROM _Open
GROUP BY
    JobID,
    SubscriberKey
```

O código é ilustrativo e foi simplificado para documentação pública.

---

## 🖱️ Etapa 5 — Cliques

Os cliques são processados de forma semelhante.

Métricas consolidadas:

```text
Clicked
FirstClickDate
LastClickDate
ClickCount
```

Exemplo conceitual:

```sql
SELECT
    JobID,
    SubscriberKey,
    MIN(EventDate) AS FirstClickDate,
    MAX(EventDate) AS LastClickDate,
    COUNT(*) AS ClickCount
FROM _Click
GROUP BY
    JobID,
    SubscriberKey
```

Além da contagem geral, URLs podem ser classificadas para identificar comportamentos específicos.

---

## 🔗 Classificação de Cliques

Em alguns cenários, determinados links possuem significado específico para análise.

Exemplo conceitual:

```text
URL contém /product/
      │
      └── ProductInterest = 1

URL contém /education/
      │
      └── EducationInterest = 1

URL contém /contact/
      │
      └── ContactInterest = 1
```

Isso transforma eventos técnicos de clique em indicadores mais úteis para análise de comportamento.

---

## ⚠️ Etapa 6 — Bounces

Os eventos de bounce são associados ao envio correspondente.

Informações consolidadas podem incluir:

```text
Bounced
BounceType
BounceDate
```

Fluxo:

```text
_Bounce
   │
   ▼
JobID + SubscriberKey
   │
   ▼
Reporting DE
```

Além de indicar que houve falha, armazenar o tipo de bounce permite análises posteriores sobre qualidade e entregabilidade da base.

---

## 🚫 Etapa 7 — Unsubscribes

Os eventos de unsubscribe também são incorporados à estrutura histórica.

Campos:

```text
Unsubscribed
UnsubscribeDate
```

Isso permite relacionar o cancelamento da inscrição com o envio ou contexto em que o evento ocorreu.

---

## 🔄 Estratégia Incremental

Um ponto importante da arquitetura é evitar reconstruir todo o histórico em cada execução.

O fluxo pode trabalhar de forma incremental:

```text
Novos envios
     │
     ▼
INSERT
     │
     ▼
Registros existentes
     │
     ├── Opens → UPDATE
     ├── Clicks → UPDATE
     ├── Bounce → UPDATE
     └── Unsubscribe → UPDATE
```

Essa abordagem reduz processamento desnecessário conforme o histórico cresce.

---

## 🕒 Persistência Histórica

Um dos principais objetivos da solução é desacoplar o histórico analítico da retenção das Data Views.

```text
Data Views
Retenção limitada
      │
      ▼
Processamento periódico
      │
      ▼
Reporting Data Extension
      │
      ▼
Histórico persistente
```

Com isso, informações coletadas anteriormente continuam disponíveis mesmo depois que o evento deixa de existir na Data View original.

---

## 📈 Possibilidades de Análise

A estrutura consolidada permite análises como:

- volume de envios;
- taxa de abertura;
- taxa de clique;
- engajamento por comunicação;
- performance por campanha;
- performance por jornada;
- quantidade média de interações;
- incidência de bounce;
- unsubscribe após comunicações;
- interesse por categorias de links;
- evolução histórica das campanhas.

A Data Extension também pode servir como origem para ferramentas externas de analytics ou processos adicionais de integração.

---

## ⚠️ Desafios Técnicos

### Dados distribuídos

As métricas estão armazenadas em diferentes Data Views.

**Abordagem:** consolidar os eventos em uma estrutura central utilizando processos independentes.

---

### Retenção das Data Views

Eventos antigos deixam de estar disponíveis após o período de retenção da plataforma.

**Abordagem:** persistir periodicamente os eventos relevantes em uma Data Extension histórica.

---

### Duplicidade

Um subscriber pode possuir diversos eventos relacionados ao mesmo envio.

**Abordagem:** utilizar uma chave de relatório capaz de identificar corretamente cada ocorrência de envio e agregar eventos posteriores.

---

### Volume de dados

Consultas históricas podem crescer significativamente ao longo do tempo.

**Abordagem:** utilizar processamento incremental e evitar reconstruções completas desnecessárias.

---

### Eventos múltiplos

Um contato pode abrir ou clicar diversas vezes.

**Abordagem:** armazenar indicadores agregados como primeira ocorrência, última ocorrência e quantidade total.

---

## 🧠 Separação de Responsabilidades

A solução mantém funções diferentes em camadas distintas.

### Data Views

Origem dos eventos operacionais.

### Automation Studio

Orquestração das consultas e atualização periódica.

### SQL

Transformação, agregação e consolidação dos eventos.

### Reporting Data Extension

Persistência histórica e estrutura analítica.

### Analytics / Integration

Consumo das informações consolidadas.

Essa separação facilita manutenção e permite evoluir cada camada independentemente.

---

## 💡 Principais Aprendizados

### Tracking operacional não substitui histórico analítico

Data Views são excelentes fontes operacionais, mas não devem ser tratadas automaticamente como armazenamento histórico permanente.

### Definir a granularidade é fundamental

Antes de construir o relatório é necessário definir o que cada linha representa.

Neste modelo:

```text
1 linha ≈ 1 ocorrência de envio para um subscriber
```

### A chave precisa refletir essa granularidade

Combinar informações como `JobID`, `BatchID` e `SubscriberKey` evita colisões entre diferentes ocorrências.

### Processamento incremental melhora escalabilidade

Quanto maior o histórico, menos sentido faz reconstruir tudo a cada execução.

### Eventos precisam ser transformados em informação

Aberturas e cliques isolados são eventos técnicos.

Quando consolidados em indicadores, datas e categorias, tornam-se dados úteis para análise.

---

## 🧰 Tecnologias Utilizadas

- Salesforce Marketing Cloud Engagement
- Automation Studio
- SQL Query Activities
- Data Extensions
- Marketing Cloud Data Views
- Email Studio
- Journey Builder
- SQL

---

## 🔐 Confidencialidade

Este case é baseado em uma experiência profissional real e foi **adaptado e anonimizado para fins de portfólio**.

Nomes de clientes, Data Extensions, campanhas, jornadas, URLs, identificadores, dados pessoais e regras proprietárias foram removidos ou substituídos por exemplos genéricos.

As consultas apresentadas são conceituais e não representam código proprietário utilizado em ambiente produtivo.

---

## 👨‍💻 Competências Demonstradas

Este projeto demonstra experiência em:

- Salesforce Marketing Cloud Engagement;
- Automation Studio;
- SQL;
- Marketing Cloud Data Views;
- modelagem de Data Extensions;
- processamento incremental;
- consolidação de dados;
- tracking de campanhas;
- persistência histórica;
- tratamento de grandes volumes;
- modelagem para reporting;
- preparação de dados para analytics.

---

## 🔗 Outros Cases

Este projeto faz parte da coleção:

**Salesforce Marketing Cloud Cases**

Outros estudos incluem:

- Abandoned Cart Automation
- Welcome Journey & Dynamic Product Personalization
- Omnichannel Consent Management

---

> Este repositório apresenta estudos técnicos baseados em experiências profissionais reais, com informações sensíveis e proprietárias removidas ou adaptadas.
