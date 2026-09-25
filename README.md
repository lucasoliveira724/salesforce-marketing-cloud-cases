<div align="center">

# ☁️ Salesforce Marketing Cloud
## Technical Cases

**Automação • Dados • Jornadas • Personalização**

Cases técnicos e arquiteturas desenvolvidos a partir de experiências práticas com **Salesforce Marketing Cloud Engagement**.

[![Salesforce](https://img.shields.io/badge/Salesforce-Marketing%20Cloud-0176D3?style=for-the-badge&logo=salesforce&logoColor=white)](https://www.salesforce.com/marketing/)
![SQL](https://img.shields.io/badge/SQL-Data-4479A1?style=for-the-badge)
![AMPscript](https://img.shields.io/badge/AMPscript-Personalization-6C63FF?style=for-the-badge)
![SSJS](https://img.shields.io/badge/SSJS-Automation-F7DF1E?style=for-the-badge)

</div>

---

## 📌 Sobre este repositório

Este repositório reúne **cases técnicos de Salesforce Marketing Cloud Engagement**, documentando soluções envolvendo automação de marketing, jornadas de clientes, processamento de dados, personalização e comunicação multicanal.

O objetivo é apresentar não apenas as tecnologias utilizadas, mas principalmente:

- o problema a ser resolvido;
- a arquitetura da solução;
- as regras e decisões técnicas;
- o fluxo de dados;
- os recursos do Marketing Cloud utilizados;
- desafios encontrados;
- aprendizados e padrões reutilizáveis.

Os cases são apresentados como **estudos técnicos de arquitetura e implementação**, e não como reprodução dos ambientes originais.

---

## 🔒 Confidencialidade

> **Os cases apresentados neste repositório são baseados em experiências profissionais reais, porém foram anonimizados e adaptados exclusivamente para fins de portfólio.**
>
> Nomes de clientes, dados, identificadores, regras proprietárias, estruturas internas e outras informações confidenciais foram removidos ou substituídos por exemplos fictícios.

Diagramas, exemplos de dados e trechos técnicos apresentados aqui podem ter sido simplificados ou reconstruídos para demonstrar os conceitos utilizados sem expor ambientes ou informações de clientes.

---

# 🧩 Technical Cases

## 01 · 🛒 Carrinho Abandonado

### Abandoned Cart Automation

Solução para processamento de eventos de abandono de carrinho, controle de elegibilidade e prevenção de reentrada do mesmo cliente para o mesmo conjunto de produtos dentro de uma janela determinada.

**Principais conceitos**

`Journey Builder` `Automation Studio` `SQL` `Data Extensions` `AMPscript`

**Destaques técnicos**

- processamento automatizado de eventos;
- identificação de produtos;
- controle de elegibilidade;
- prevenção de reentrada;
- histórico de processamento;
- personalização da comunicação.

📁 **Case:** `cases/abandoned-cart`

---

## 02 · 👋 Jornada de Boas-Vindas

### Welcome Journey & Product Personalization

Jornada de boas-vindas com conteúdo personalizado e recomendação dinâmica de produtos utilizando dados processados no Marketing Cloud.

**Principais conceitos**

`Journey Builder` `Automation Studio` `Content Builder` `AMPscript` `SQL`

**Destaques técnicos**

- preparação dos dados;
- ranking de produtos;
- conteúdo dinâmico;
- personalização de e-mail;
- construção de vitrine de produtos;
- integração entre dados e conteúdo.

📁 **Case:** `cases/welcome-journey`

---

## 03 · 🔐 Gestão de Consentimento Multicanal

### Omnichannel Consent Management

Estrutura para consolidação e processamento de informações de **opt-in e opt-out** provenientes de diferentes canais de comunicação.

**Principais conceitos**

`SQL` `Data Views` `Email` `SMS` `Push` `WhatsApp`

**Destaques técnicos**

- consolidação de consentimentos;
- múltiplas fontes de dados;
- normalização de status;
- processamento automatizado;
- visão centralizada por cliente;
- preparação para integração com sistemas externos.

📁 **Case:** `cases/consent-management`

---

## 04 · 📊 Marketing Reporting

### Marketing Data & Reporting Pipeline

Pipeline de dados criado para consolidar e persistir métricas de campanhas e jornadas, permitindo análises além das limitações de retenção das Data Views.

**Principais conceitos**

`Automation Studio` `SQL` `Data Views` `Data Extensions`

**Destaques técnicos**

- histórico de envios;
- aberturas;
- cliques;
- bounces;
- unsubscribes;
- consolidação de métricas;
- persistência histórica;
- automação do processamento.

📁 **Case:** `cases/marketing-reporting`

---

# ⚙️ Tecnologias & Recursos

### Salesforce Marketing Cloud

`Journey Builder`  
`Automation Studio`  
`Email Studio`  
`Content Builder`  
`MobileConnect`  
`Data Extensions`  
`Data Views`

### Desenvolvimento

`SQL`  
`AMPscript`  
`SSJS`  
`HTML`  
`CSS`  
`JavaScript`

### Arquitetura & Dados

`Data Processing`  
`Marketing Automation`  
`Customer Journeys`  
`Personalization`  
`Data Modeling`  
`API Integration`

---

# 🏗️ Estrutura do repositório

```text
salesforce-marketing-cloud-cases/
│
├── README.md
│
├── cases/
│   ├── abandoned-cart/
│   │   └── README.md
│   │
│   ├── welcome-journey/
│   │   └── README.md
│   │
│   ├── consent-management/
│   │   └── README.md
│   │
│   └── marketing-reporting/
│       └── README.md
│
└── assets/
    ├── diagrams/
    └── images/
```

Cada case possui sua própria documentação técnica, permitindo detalhar arquitetura, fluxo, decisões e tecnologias sem transformar o README principal em documentação infinita.

---

# 🎯 Objetivo

Este repositório faz parte do meu portfólio profissional e tem como objetivo demonstrar experiência prática na construção de soluções utilizando **Salesforce Marketing Cloud Engagement**, especialmente em cenários que envolvem:

- automação de marketing;
- jornadas de clientes;
- processamento e modelagem de dados;
- personalização;
- comunicação multicanal;
- integração entre dados, regras de negócio e comunicação.

---

<div align="center">

## 👨‍💻 Lucas Oliveira

**Salesforce Marketing Cloud Consultant & Developer**

Salesforce • Development • Automation & AI

[![GitHub](https://img.shields.io/badge/GitHub-lucasoliveira724-181717?style=for-the-badge&logo=github)](https://github.com/lucasoliveira724)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Lucas%20Oliveira-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/lucas-oliveira-7948b494/)

</div>
