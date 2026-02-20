# Project: Date Time Services

## 📋 Overview

An Oracle Service Bus (OSB) project that receives a date/time input and performs multiple date operations on it, returning a structured response. This project covers core OSB concepts — Proxy Services, Pipelines, XQuery Date Functions and format conversion logic — with a single WSDL operation exposed to the client.

**What it does:**  
Input: A date value with optional format and operation metadata  
Output: Transformed date/time results based on the requested operation

| Operation            | Input                                   | Output                                               |
| -------------------- | --------------------------------------- | ---------------------------------------------------- |
| `getCurrentDateTime` | `DateTimeRequest` with `operation` only | `DateTimeResponse` with `outputDate` and `timestamp` |

---

## 🏗️ Folder Structure

```
DTService/
│
├── Pipelines/
│   └── DTServicePP.pipeline          # Pipeline with date operation logic
│
├── ProxyServices/
│   └── DTServicePS.proxy             # Exposes the SOAP endpoint to clients
│
├── Schemas/
│   └── DTService.xsd                 # Defines DateTimeRequest, DateTimeResponse, DateTimeFault elements
│
├── WSDLs/
│   └── DTService.wsdl                # Service contract with getCurrentDateTime operation
│
├── pom.xml                           # Maven build configuration
│
└── DTService.jpr                     # JDeveloper project file
```

---

## 🚀 Implementation Steps

### Step 1: Create Service Bus Application and Project

1. Open **Oracle JDeveloper**
2. File → New → **Service Bus Application with Service Bus Project**
3. Application Name: `DTServiceApp`
4. Project Name: `DTService`
5. Click **Finish**

---

### Step 2: Create XSD Schema

1. Right-click on **Schemas** folder → New → **XML Schema**
2. File Name: `DTService.xsd`
3. Open in **Source view** and define the following elements:
   - `DateTimeRequest` — contains `operation` (required), `inputDate`, `inputFormat`, `outputFormat`, `daysToAdd` (all optional)
   - `DateTimeResponse` — contains `status`, `operation`, `inputDate`, `outputDate`, `timestamp`, `message`
   - `DateTimeFault` — contains `errorCode`, `errorMessage`, `operation`
4. Set `targetNamespace` to: `http://practice.learning.com/osb/DTService/schema`
5. Set `elementFormDefault="qualified"`

**Key design decisions:**

- `minOccurs="0"` on all optional fields — `GET_CURRENT` only needs `operation`
- Enumerations used for `OperationType`, `DateFormatType`, and `StatusType` to restrict valid values
- Separate `DateTimeFault` element for structured SOAP fault handling

---

### Step 3: Create WSDL

1. Right-click on **WSDLs** folder → New → **WSDL Document**
2. File Name: `DTService.wsdl`
3. Open in **Source view** and configure:
   - Import the XSD schema using `<xsd:import schemaLocation="../Schemas/DTService.xsd"/>`
   - Define 3 messages: `DateTimeRequestMsg`, `DateTimeResponseMsg`, `DateTimeFaultMsg`
   - Define `DateTimeServicesPortType` with 1 operation: `getCurrentDateTime`
   - Add `document/literal` SOAP binding with `soapAction="getCurrentDateTime"`
   - Add `<service>` block with `soap:address location="http://localhost:7101/DateTimeServices"`
4. Set `targetNamespace` to: `http://practice.learning.com/osb/DTService/wsdl`

> **Note:** Only 1 operation is defined in this version. With a single operation, the Proxy Service Operation Selector can use either `SOAP Body Type` or `SOAP Action Header` without conflict.

---

### Step 4: Create Pipeline and Proxy Service Together

1. Open the **Service Bus Project** canvas (double-click `DTService.jpr`)
2. From **Component Palette** → drag **Pipeline** into the **Pipelines/Split Joins** swim lane
3. Configure the Pipeline wizard:
   - Service Name: `DTServicePP`
   - Service Type: **WSDL Based Service** → browse and select `DTService.wsdl`
   - Port: `DateTimeServicesPort`
   - **Check** `Expose as Proxy Service` — OSB will auto-create the Proxy Service simultaneously
   - Proxy Service Name: `DTServicePS`
   - Transport: `HTTP`
   - Endpoint URI: `/DTService`
4. Click **Finish**

You will see:

- `DTServicePP` created in the **middle swim lane**
- `DTServicePS` auto-created in the **left swim lane**
- A **wire** automatically connecting Proxy → Pipeline

---

### Step 5: Configure Proxy Service Operation Selector

1. Double-click `DTServicePS` to open its configuration
2. Navigate to **Operation Selection** tab
3. Set Selector to: `SOAP Action Header`

> **Why:** If multiple operations share the same input message element, `SOAP Body Type` throws `OSB-395139`. `SOAP Action Header` uses the HTTP SOAPAction header to identify the operation — each operation's `soapAction` value in the WSDL binding acts as the discriminator.

---

### Step 6: Design Pipeline Logic

Double-click `DTServicePP` to open the Pipeline editor.

#### 6.1 Add Stage to Request Pipeline — Log Incoming Request

1. Drag a **Stage** node into the **Request Pipeline**
2. Name it: `Stage_LogRequest`
3. Double-click to open it and add a **Log** action (from Reporting)
4. Configure the Log action:

| Field    | Value                        |
| -------- | ---------------------------- |
| Severity | `Info`                       |
| Summary  | `DTService Request Received` |

**Log Expression:**

```xquery
fn:concat(
    "Operation: ",
    data($body/*:DateTimeRequest/*:operation),
    " | InputDate: ",
    data($body/*:DateTimeRequest/*:inputDate),
    " | Timestamp: ",
    xs:string(fn:current-dateTime())
)
```

#### 6.2 Add Operational Branch — Route by Operation

1. Drag an **Operational Branch** node onto the **main pipeline canvas** — outside and below the main Pipeline Pair node
2. OSB will auto-populate the branches from the WSDL operations:
   - Branch: `getCurrentDateTime`

> **Note:** Operational Branch is a **node-level component** — it cannot be placed inside a Stage. It must sit on the main pipeline canvas between Pipeline Pair nodes.

#### 6.3 Configure Branch — getCurrentDateTime

1. Inside the `getCurrentDateTime` branch, drag a **Pipeline Pair** node
2. Inside its **Request Pipeline**, drag a **Stage**
3. Name it: `Stage_GetCurrentDateTime`
4. Double-click to open it and add an **Assign** action (from Message Processing)
5. Configure the Assign action:

| Field    | Value  |
| -------- | ------ |
| Variable | `body` |

**Add to Namespace Table:**

| Prefix | URI                                                 |
| ------ | --------------------------------------------------- |
| `tns`  | `http://practice.learning.com/osb/DTService/schema` |

**Expression:**

```xquery
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    <tns:DateTimeResponse
        xmlns:tns="http://practice.learning.com/osb/DTService/schema">
        <tns:status>SUCCESS</tns:status>
        <tns:operation>GET_CURRENT</tns:operation>
        <tns:inputDate>{data($body/*:DateTimeRequest/*:inputDate)}</tns:inputDate>
        <tns:outputDate>
            {fn:format-date(fn:current-date(), "[Y0001]-[M01]-[D01]")}
        </tns:outputDate>
        <tns:timestamp>
            {fn:format-dateTime(fn:current-dateTime(), "[Y0001]-[M01]-[D01]T[H01]:[m01]:[s01]")}
        </tns:timestamp>
        <tns:message>Current date and time retrieved successfully</tns:message>
    </tns:DateTimeResponse>
</soap-env:Body>
```

#### 6.4 Add Error Handler

1. Drag an **Error Handler** onto the **main pipeline canvas** — outside all Pipeline Pair nodes
2. Inside the Error Handler, drag a **Stage**
3. Inside the Stage, drag an **Assign** action
4. Configure the Assign action:

| Field    | Value   |
| -------- | ------- |
| Variable | `fault` |

**Add to Namespace Table:**

| Prefix | URI                                                 |
| ------ | --------------------------------------------------- |
| `tns`  | `http://practice.learning.com/osb/DTService/schema` |
| `ctx`  | `http://www.bea.com/wli/sb/context`                 |

**Expression:**

```xquery
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    <tns:DateTimeFault
        xmlns:tns="http://practice.learning.com/osb/DTService/schema">
        <tns:errorCode>{fn:string($fault/ctx:errorCode)}</tns:errorCode>
        <tns:errorMessage>{fn:string($fault/ctx:reason)}</tns:errorMessage>
        <tns:operation>
            {data($body/*:Body/*:DateTimeRequest/*:operation)}
        </tns:operation>
    </tns:DateTimeFault>
</soap-env:Body>
```

5. After the Assign, drag a **Reply** action (from Flow Control)
6. Set Reply option to: `Failure`

---

### Step 7: Deploy the Project

1. **Start WebLogic Server** (if not running):
   - Right-click on **IntegratedWebLogicServer** in Application Server Navigator
   - Select **Start Server Instance**
   - Wait until you see `Server started in RUNNING mode`
2. **Deploy the Project:**
   - Right-click on **DTService** project
   - Select **Deploy → DTService**
   - Select **Deploy to Application Server**
   - Choose server: `IntegratedWebLogicServer`
   - Click **Finish**
3. **Confirm deployment:**
   - Watch the console output for: `Deployment finished`
   - No validation errors should appear

---

## 🧪 Testing

### Test via Service Bus Console

1. Open browser and navigate to: **http://localhost:7101/sbconsole**
2. **Login** with WebLogic credentials
3. Navigate to:
   - **Project Explorer** → **DTService** → **Proxy Services** → **DTServicePS**
4. Click **Launch Test Console** button
5. Set **SOAPAction** header to: `getCurrentDateTime`

---

#### ✅ Test 1 — GET_CURRENT (minimal request)

**Request:**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header/>
    <soapenv:Body>
        <sch:DateTimeRequest xmlns:sch="http://practice.learning.com/osb/DTService/schema">
            <sch:operation>GET_CURRENT</sch:operation>
        </sch:DateTimeRequest>
    </soapenv:Body>
</soapenv:Envelope>
```

**Expected Response:**

```xml
<tns:DateTimeResponse>
    <tns:status>SUCCESS</tns:status>
    <tns:operation>GET_CURRENT</tns:operation>
    <tns:inputDate/>
    <tns:outputDate>2026-02-19</tns:outputDate>
    <tns:timestamp>2026-02-19T17:52:24</tns:timestamp>
    <tns:message>Current date and time retrieved successfully</tns:message>
</tns:DateTimeResponse>
```

---

#### ✅ Test 2 — GET_CURRENT (with optional inputDate)

**Request:**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header/>
    <soapenv:Body>
        <sch:DateTimeRequest xmlns:sch="http://practice.learning.com/osb/DTService/schema">
            <sch:operation>GET_CURRENT</sch:operation>
            <sch:inputDate>19-02-2026</sch:inputDate>
            <sch:inputFormat>DD/MM/YYYY</sch:inputFormat>
            <sch:outputFormat>MM-DD-YYYY</sch:outputFormat>
        </sch:DateTimeRequest>
    </soapenv:Body>
</soapenv:Envelope>
```

**Expected Response:**

- `inputDate` = `19-02-2026` (echoed back from request)
- `outputDate` = current system date (input is ignored for GET_CURRENT)
- `timestamp` = current system timestamp

> 💡 This test verifies the **Canonical Message Model** pattern — optional fields are passed by the caller but gracefully ignored by the service when not applicable to the operation.

---

## 🐛 Challenges and Issues

**Issue 1:** `OSB-395139: Two or more operations expect the same incoming message`

**Error:**

```
[OSB-395139] Two or more operations expect the same incoming message,
you must use a selector different than message body
```

**Root Cause:** When multiple WSDL operations share the same input message element (e.g., all operations use `DateTimeRequest`), OSB cannot distinguish which operation to invoke by inspecting the SOAP body alone.

**Solution:** In the Proxy Service **Operation Selection**, change the selector from `SOAP Body Type` to **`SOAP Action Header`**. OSB will then use the HTTP SOAPAction header to identify the operation. Each operation in the WSDL binding already has a unique `soapAction` value that acts as the discriminator.

| Selector Option      | When to Use                                              |
| -------------------- | -------------------------------------------------------- |
| `SOAP Body Type`     | Each operation has a unique root XML element in the body |
| `SOAP Action Header` | Multiple operations share the same input message element |
| `SOAP Header`        | Operation identity embedded in SOAP header               |
| `WS-Addressing`      | WS-Addressing `Action` header is used                    |
| `Transport Header`   | Custom HTTP header identifies the operation              |

---

**Issue 2:** `OSB-382040: Value must be an instance of {http://schemas.xmlsoap.org/soap/envelope/}Body`

**Error:**

```
OSB-382040: Failed to set the value of context variable "body".
Value must be an instance of {http://schemas.xmlsoap.org/soap/envelope/}Body.
```

**Root Cause:** The `$body` context variable in OSB must always contain a proper `<soap:Body>` wrapper element. Assigning just the payload element (e.g., `<tns:DateTimeResponse>`) directly without the wrapper violates this contract.

**Solution:** Always wrap the response element inside `<soap-env:Body>` when assigning to `$body`:

```xquery
<soap-env:Body xmlns:soap-env="http://schemas.xmlsoap.org/soap/envelope/">
    <tns:DateTimeResponse ...>
        ...
    </tns:DateTimeResponse>
</soap-env:Body>
```

---

**Issue 3:** Nested element returned instead of text value

**Symptom:** Response contained nested element with namespace prefix instead of plain text value:

```xml
<tns:inputDate>
    <sch:inputDate xmlns:sch="...">19-02-2026</sch:inputDate>
</tns:inputDate>
```

**Root Cause:** Using `{$body/*:DateTimeRequest/*:inputDate}` in a construction expression returns the **entire element node** including its tags — not just the text content.

**Solution:** Wrap with `data()` to extract the text value only:

```xquery
{data($body/*:DateTimeRequest/*:inputDate)}
```

---

## 🪜 Enhancements for This Project

- Add `convertDateFormat` operation to convert dates between formats: `YYYY-MM-DD`, `DD/MM/YYYY`, `MM-DD-YYYY`, `YYYY/MM/DD` using `fn:substring()` based parsing
- Add `addDaysToDate` operation using `xs:date()` and `xs:dayTimeDuration()` for native XQuery date arithmetic
- Add a **Validate** node before the Operational Branch to enforce XSD schema compliance on incoming requests
- Add a **Log** node in each operation branch to trace which branch was invoked
- Use `fn:format-date()` with `DD-MMM-YYYY` picture string to support month abbreviation output format
