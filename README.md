# Oracle Service Bus (OSB) Project Based Learning

![Projects Completed](https://img.shields.io/badge/Projects-4%2F25-blue)
![OSB Focus](https://img.shields.io/badge/Focus-OSB%20Development-green)
![Last Updated](https://img.shields.io/badge/Updated-February%202026-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## 🎯 Overview

This repository contains hands-on implementation of **25 progressive integration projects in OSB**.

While the code in this repository is OSB-focused, each project is also designed
from a **SOA Suite perspective** to understand:

- When to use OSB vs SOA
- Mediation vs Orchestration
- Pipeline vs BPEL
- Routing vs Process Flow
- Stateless vs Long-running processes

For every project, I analyze:

- ✅ OSB implementation (code available in this repo)
- 🧠 Equivalent SOA approach (conceptual comparison & architecture notes)
- 📊 Trade-offs and decision-making considerations

The goal is not just to learn tools — but to develop strong
**integration architecture thinking** using Oracle’s middleware stack.

---

## 📊 Progress Tracker

### Overall Progress: 4/25

| Level                     | Projects | Status         | Count |
| ------------------------- | -------- | -------------- | ----- |
| **Level 1: Foundation**   | 5        | 🔄 In Progress | 4/5   |
| **Level 2: Intermediate** | 5        | ⏳ Not Started | 0/5   |
| **Level 3: Advanced**     | 5        | ⏳ Not Started | 0/5   |
| **Level 4: Enterprise**   | 5        | ⏳ Not Started | 0/5   |
| **Level 5: Expert**       | 5        | ⏳ Not Started | 0/5   |

---

## 📚 Projects by Level

### 🟢 Level 1: Foundation (Beginner)

Master OSB and SOA basics through simple projects.

| #   | Project                                                          | OSB Status  | Topics Covered                                   |
| --- | ---------------------------------------------------------------- | ----------- | ------------------------------------------------ |
| 01  | [Hello User](01-Foundation/01-HelloUser/)                        | ✅ Complete | Proxy Services, Pipelines, Replace Action, XPath |
| 02  | [Calculator Service](01-Foundation/02-Calculator/)               | ✅ Complete | Conditional Logic, XPath Math, Error Messages    |
| 03  | [Temperature Converter](01-Foundation/03-Temperature-Converter/) | ✅ Complete | Multiple Transformations, XPath Functions        |
| 04  | [String Manipulation](01-Foundation/04-String-Manipulation/)     | ✅ Complete | String Functions, Switch-Case Patterns           |
| 05  | [Date Time Service](01-Foundation/05-Date-Time-Service/)         | 🔄 In Progress  | Date Functions, Format Conversions               |

**Learning Objectives:**

- OSB: Proxy Services, Pipelines, Message Flow, XPath basics
- SOA: BPEL basics, Assign activity, Receive/Reply, XSD/WSDL

---

### 🔵 Level 2: Intermediate

Service integration and orchestration patterns.

| #   | Project                                                        | OSB Status | Topics Covered                              |
| --- | -------------------------------------------------------------- | ---------- | ------------------------------------------- |
| 06  | [Two-Service Orchestration](02-Intermediate/06-orchestration/) | ⏳ Pending | Service Callout vs Invoke, Sequential Calls |
| 07  | [Parallel Invocation](02-Intermediate/07-parallel/)            | ⏳ Pending | Split-Join vs Flow, Aggregation             |
| 08  | [Content-Based Routing](02-Intermediate/08-routing/)           | ⏳ Pending | Route Node vs If/Else, Dynamic Routing      |
| 09  | [Data Transformation](02-Intermediate/09-transformation/)      | ⏳ Pending | XSLT vs Assign, Complex Mapping             |
| 10  | [Error Handling](02-Intermediate/10-error-handling/)           | ⏳ Pending | Error Pipelines vs Fault Handlers           |

**Learning Objectives:**

- OSB: Business Services, Service Callout, Split-Join, XSLT
- SOA: Partner Links, Invoke, Parallel Flow, Fault Handling

---

### 🟠 Level 3: Advanced

Real-world integration patterns.

| #   | Project                                           | OSB Status | Topics Covered                  |
| --- | ------------------------------------------------- | ---------- | ------------------------------- |
| 11  | [Async Order Processing](03-Advanced/11-async/)   | ⏳ Pending | Fire-and-Forget vs Async BPEL   |
| 12  | [Human Task Workflow](03-Advanced/12-human-task/) | ⏳ N/A     | SOA-only (Human Task Component) |
| 13  | [Database CRUD](03-Advanced/13-database/)         | ⏳ Pending | DB Adapter in OSB vs SOA        |
| 14  | [File Processing](03-Advanced/14-file/)           | ⏳ Pending | File Adapter, Batch Processing  |
| 15  | [REST API Integration](03-Advanced/15-rest/)      | ⏳ Pending | REST Adapter, JSON Handling     |

**Learning Objectives:**

- OSB: Asynchronous patterns, Adapters, JSON transformations
- SOA: Async BPEL, Human Tasks, Correlation, Adapters

---

### 🟣 Level 4: Enterprise Patterns

Distributed systems and advanced patterns.

| #   | Project                                                        | OSB Status | Topics Covered                               |
| --- | -------------------------------------------------------------- | ---------- | -------------------------------------------- |
| 16  | [Pub-Sub Messaging](04-Enterprise/16-pubsub/)                  | ⏳ Pending | JMS Topics, Message Broadcasting             |
| 17  | [Saga Pattern](04-Enterprise/17-saga/)                         | ⏳ Pending | Compensation Logic, Distributed Transactions |
| 18  | [API Gateway](04-Enterprise/18-api-gateway/)                   | ⏳ Pending | ⏳ N OSB-focused (Service Virtualization)    |
| 19  | [Event Sourcing](04-Enterprise/19-event-sourcing/)             | ⏳ Pending | Event Store, CQRS Pattern                    |
| 20  | [Microservices Orchestration](04-Enterprise/20-microservices/) | ⏳ Pending | Service Coordination, Resilience             |

**Learning Objectives:**

- OSB: API Management, Service Virtualization, Policy Enforcement
- SOA: Compensation, Saga Pattern, Complex Orchestration

---

### 🔴 Level 5: Expert Integration

Modern integration patterns and multi-technology.

| #   | Project                                            | Status     | Topics Covered                         |
| --- | -------------------------------------------------- | ---------- | -------------------------------------- |
| 21  | [B2B EDI Integration](05-Expert/21-b2b/)           | ⏳ Pending | EDI Translation, Trading Partners      |
| 22  | [Blockchain Integration](05-Expert/22-blockchain/) | ⏳ Pending | Smart Contracts, Immutable Ledger      |
| 23  | [AI/ML Integration](05-Expert/23-ai-ml/)           | ⏳ Pending | ML Model APIs, Prediction Services     |
| 24  | [Stream Processing](05-Expert/24-streaming/)       | ⏳ Pending | Kafka Integration, Real-time Analytics |
| 25  | [Multi-Cloud Hub](05-Expert/25-multi-cloud/)       | ⏳ Pending | AWS, Azure, GCP Integration            |

**Learning Objectives:**

- Advanced integration with modern technologies
- Cloud-native patterns
- Real-time processing

---

## 🛠️ Tech Stack

### Core Technologies

- **Oracle Service Bus (OSB)** 14c - Primary focus for mediation and routing
- **Oracle SOA Suite** 14c - For orchestration and workflow
- **Oracle JDeveloper** 14.1.2.0.0 - IDE for development
- **Oracle WebLogic Server** 14.1.2.0.0 - Application server

### Development Tools

- **SoapUI** - SOAP/REST service testing
- **Postman** - REST API testing
- **Git** - Version control
- **Maven** - Build automation

### Languages & Standards

- **BPEL** - Business Process Execution Language
- **XSLT** - XML transformations
- **XQuery** - XML query language
- **XPath** - XML navigation
- **WSDL** - Service descriptions
- **XSD** - Data structure definitions

---

## Which one to choose, OSB or SOA?

```
START
  │
  ├─ Does it take longer than 1 minute? ────────────────── YES ──▶ SOA
  │
  NO
  │
  ├─ Does a human need to approve/review something? ──────── YES ──▶ SOA
  │
  NO
  │
  ├─ Do you need to wait for another system (hours/days)? ── YES ──▶ SOA
  │
  NO
  │
  ├─ Do you need to remember what happened yesterday? ────── YES ──▶ SOA
  │
  NO
  │
  ├─ Is it just transformation/routing/validation? ───────── YES ──▶ OSB
  │
  └─ Is it processing 1000s of messages per minute? ──────── YES ──▶ OSB
```

---

## 🤝 Contributing

While this is primarily a personal learning journey, I welcome:

- 🐛 **Bug reports** - If you spot errors in my implementations
- 💡 **Suggestions** - Better approaches or patterns I should consider
- 📖 **Documentation improvements** - Clearer explanations
- ❓ **Questions** - Learning together through discussion

## 📝 License

This project is licensed under the **MIT License**

---

<div align="center">

### 🚀 Let's learn integration architecture together!

![Oracle](https://img.shields.io/badge/Oracle-Middleware-red?logo=oracle&logoColor=white)
![Primary Focus](https://img.shields.io/badge/Primary-OSB%2014c-blue)
![Parallel Learning](https://img.shields.io/badge/Also%20Exploring-SOA%20Suite-orange)
![Status](https://img.shields.io/badge/Status-Learning-green)

**Built with ❤️ and lots of ☕ by Umair Faheem**

</div>
