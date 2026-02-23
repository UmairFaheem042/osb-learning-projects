# Two-Service-Orchestration — Project Summary
### Oracle Service Bus 14c | JDeveloper | SOA Suite

---

## 1. Problem Statement

We needed to build a **single unified API endpoint** that internally:

1. Accepts a `customerId` from a caller
2. Calls a **Customer Service** to fetch customer details (name, email)
3. Uses the result to call an **Order Service** to fetch that customer's order details
4. Combines both responses into one **OrchestratedResponse** and returns it to the caller

This is a classic **Service Orchestration** pattern — one entry point, multiple backend calls, one combined response.

```
Caller ──→ [OrchestratorPS] ──→ [Pipeline] ──→ [CustomerBS] ──→ Customer Backend
                                           └──→ [OrderBS]    ──→ Order Backend
                ←── OrchestratedResponse ──────────────────────────────────────
```

---

## 2. Why These Components?

### Why 2 Business Services?

| Business Service | Purpose |
|---|---|
| `CustomerBS` | Represents the Customer backend — OSB uses it to call `localhost:8011/customer` |
| `OrderBS` | Represents the Order backend — OSB uses it to call `localhost:8012/order` |

> A **Business Service** in OSB is an outbound connector. Every backend system you want to call needs its own Business Service. We have 2 backends → 2 Business Services.

### Why 1 Proxy Service?

The **Proxy Service** (`OrchestratorPS`) is the **single inbound entry point** exposed to the caller. There is only one because:
- The caller makes **one request** and expects **one response**
- All the complexity of calling multiple backends is hidden inside OSB
- The caller doesn't know or care about `CustomerBS` or `OrderBS`

### Why 1 Pipeline?

The **Pipeline** (`OrchestratorPL`) contains all the orchestration logic. One pipeline is sufficient because:
- We have one linear flow: receive → call service A → call service B → combine → reply
- The pipeline sits between the Proxy Service and the Business Services
- It's the "brain" of the orchestration

---

## 3. Architecture Overview

```
┌──────────────────┬──────────────────────────┬─────────────────┐
│  Proxy Services  │       Pipelines          │ Business Svcs   │
│                  │                          │                 │
│ [OrchestratorPS] →→→ [OrchestratorPL]      │  [CustomerBS]   │
│                  │         │                │                 │
│                  │   [Pipeline Pair]        │  [OrderBS]      │
│                  │    ├─ RequestPipeline    │                 │
│                  │    │   └─ CallServicesStage               │
│                  │    └─ ResponsePipeline   │                 │
└──────────────────┴──────────────────────────┴─────────────────┘
```

---

## 4. Files Created

### XSD Schemas
| File | Namespace | Purpose |
|---|---|---|
| `CustomerRequest.xsd` | `.../schema/customer` | Input to Customer Service |
| `CustomerResponse.xsd` | `.../schema/customer` | Output from Customer Service |
| `OrderRequest.xsd` | `.../schema/order` | Input to Order Service |
| `OrderResponse.xsd` | `.../schema/order` | Output from Order Service |
| `OrchestratedResponse.xsd` | `.../schema/orchestrated` | Final combined response |

### WSDLs
| File | Purpose |
|---|---|
| `CustomerService.wsdl` | Contract for Customer backend (`getCustomer` operation) |
| `OrderService.wsdl` | Contract for Order backend (`getOrder` operation) |
| `OrchestratorService.wsdl` | Contract exposed to callers (`orchestrate` operation) |

---

## 5. Pipeline Internals — What Each Component Did

### Pipeline Pair
Splits processing into two lanes:
- **Request Pipeline** — where inbound message is processed and backends are called
- **Response Pipeline** — where the final response is processed before returning to caller

### Stage (`CallServicesStage`)
A logical container inside the Request Pipeline that groups all our processing steps together.

---

### The 3 Assign Nodes — Sequential Data Flow

```
$body (inbound request)
    ↓
[Assign 1] → reads customerId from $body → stores CustomerRequest XML into customerRequest variable
    ↓
[ServiceCallout 1] → sends customerRequest to CustomerBS → stores response into customerResponse
    ↓
[Assign 2] → reads customerId from $customerResponse → stores OrderRequest XML into orderRequest variable
    ↓
[ServiceCallout 2] → sends orderRequest to OrderBS → stores response into orderResponse
    ↓
[Assign 3] → reads from both $customerResponse and $orderResponse → stores OrchestratedResponse into $body
    ↓
$body returned to caller
```

#### Assign 1 — Build Customer Request
- **Reads from:** `$body` (the inbound SOAP body containing `customerId`)
- **Builds:** A `CustomerRequest` XML element
- **Stores into:** `customerRequest` variable
- **Purpose:** Prepare the request payload for `CustomerBS`

```xquery
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    <cust:CustomerRequest xmlns:cust="...schema/customer">
        <cust:customerId>
            {$body//*[local-name()='customerId']}
        </cust:customerId>
    </cust:CustomerRequest>
</soap-env:Body>
```

#### Assign 2 — Build Order Request
- **Reads from:** `$customerResponse` (what CustomerBS returned)
- **Builds:** An `OrderRequest` XML element using the `customerId` from the customer response
- **Stores into:** `orderRequest` variable
- **Purpose:** This is the **sequential dependency** — we use the output of call 1 as input to call 2

```xquery
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    <ord:OrderRequest xmlns:ord="...schema/order">
        <ord:customerId>
            {$customerResponse//*[local-name()='customerId']}
        </ord:customerId>
    </ord:OrderRequest>
</soap-env:Body>
```

#### Assign 3 — Build Final Orchestrated Response
- **Reads from:** Both `$customerResponse` AND `$orderResponse`
- **Builds:** A combined `OrchestratedResponse` XML element with all fields
- **Stores into:** `body` (OSB's special context variable)
- **Purpose:** Merge both backend responses into one unified response for the caller

---

### The 2 Service Callout Nodes

| | ServiceCallout 1 | ServiceCallout 2 |
|---|---|---|
| **Service** | `CustomerBS` | `OrderBS` |
| **Operation** | `getCustomer` | `getOrder` |
| **RequestAction (reads from)** | `customerRequest` | `orderRequest` |
| **ResponseAction (writes into)** | `customerResponse` | `orderResponse` |

> **Service Callout vs Routing:**
> - **Service Callout** = call a backend mid-pipeline and **continue** execution
> - **Routing** = forward message to a backend and **end** pipeline
>
> We used Service Callout because we needed to make **2 calls** and combine results — execution must continue after each call.

### Log Node (Response Pipeline)
- Logs the final `$body` content at `Info` severity
- Executes naturally as the response flows back to the caller
- Useful for debugging and confirming the orchestrated response

---

## 6. Shared Variables

4 custom variables were created in the pipeline's shared variable section:

| Variable | Written by | Read by |
|---|---|---|
| `customerRequest` | Assign 1 | ServiceCallout 1 RequestAction |
| `customerResponse` | ServiceCallout 1 ResponseAction | Assign 2, Assign 3 |
| `orderRequest` | Assign 2 | ServiceCallout 2 RequestAction |
| `orderResponse` | ServiceCallout 2 ResponseAction | Assign 3 |

---

## 7. Errors Encountered and Solutions

### Error 1 — OSB-382040: Failed to set value of context variable "body"

**Error message:**
```
Value must be an instance of {http://schemas.xmlsoap.org/soap/envelope/}Body
```

**Root Cause:**
Assign 3 was assigning a plain XML element to `$body`. OSB's `$body` is a **special context variable** that must always contain a proper SOAP Body wrapper element.

**Solution:**
Wrap all content assigned to `$body` (and any variable used in ServiceCallout RequestAction) inside a SOAP Body envelope:
```xml
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    ... your XML content here ...
</soap-env:Body>
```

---

### Error 2 — OSB-380002: Connection Refused on localhost:8011

**Error message:**
```
Tried all: '2' addresses, but could not connect over HTTP to server: 'localhost', port: '8011'
```

**Root Cause:**
The pipeline was correctly built and executing, but no actual services were running on ports `8011` and `8012`. The Business Services had nowhere to connect to.

**Solution:**
Created **SoapUI Mock Services** to simulate the backends:
- `CustomerMockService` running on port `8011` returning a static `CustomerResponse`
- `OrderMockService` running on port `8012` returning a static `OrderResponse`

---

### Error 3 — Nested Elements in Response

**Symptom:**
```xml
<orch:customerId>
    <cust:customerId xmlns:cust="...">C001</cust:customerId>
</orch:customerId>
```
Instead of:
```xml
<orch:customerId>C001</orch:customerId>
```

**Root Cause:**
Using `{$customerResponse//*[local-name()='customerId']}` returns the **entire element including its tags**, not just the text value.

**Solution:**
Wrap with `fn:data()` to extract only the typed text value:
```xquery
{fn:data($customerResponse//*[local-name()='customerId'])}
```

---

## 8. Key Concepts Covered

| Concept | Where Applied |
|---|---|
| **Service Orchestration** | Overall project pattern — one proxy, multiple backends, one response |
| **Sequential Service Calls** | Assign 2 uses output of Callout 1 as input to Callout 2 |
| **Service Callout vs Routing** | Service Callout used to call mid-pipeline without ending flow |
| **$body context variable** | Reading inbound request, writing final SOAP response |
| **SOAP Body wrapper requirement** | Any variable assigned to $body or used in RequestAction needs wrapper |
| **Shared Pipeline Variables** | Custom variables scoped across the entire pipeline |
| **fn:data() vs node extraction** | fn:data() extracts text value only, without element tags |
| **Pipeline Pair** | Separates request processing from response processing |
| **SoapUI Mock Services** | Simulating backends for testing without real services |

---

## 9. Final Working Flow

```
1. Caller sends SOAP request with customerId C001
         ↓
2. OrchestratorPS receives request on /orchestrator
         ↓
3. OrchestratorPL Request Pipeline begins
         ↓
4. Assign 1: extracts customerId from $body → builds customerRequest
         ↓
5. ServiceCallout 1: sends customerRequest → CustomerBS → mock returns John Doe
         ↓
6. Assign 2: extracts customerId from customerResponse → builds orderRequest
         ↓
7. ServiceCallout 2: sends orderRequest → OrderBS → mock returns Laptop / 1500.00
         ↓
8. Assign 3: merges customerResponse + orderResponse → writes OrchestratedResponse into $body
         ↓
9. Response Pipeline: Log node logs final $body
         ↓
10. OrchestratorPS returns OrchestratedResponse to caller
```