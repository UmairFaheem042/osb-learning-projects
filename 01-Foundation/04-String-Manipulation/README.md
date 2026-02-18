# Project: String Manipulation

## 📋 Overview

An Oracle Service Bus (OSB) project that receives a string input and performs multiple string operations on it, returning a structured response. This project covers core OSB concepts — Proxy Services, Pipelines, XPath String Functions, Pipeline Variables, and Switch-Case patterns using If-Then — with a single WSDL operation exposed to the client.

**What it does:**  
Input: A string value  
Output: Transformed string results with category classification based on string length

| Operation       | Input                              | Output                                                                |
| --------------- | ---------------------------------- | --------------------------------------------------------------------- |
| `processString` | `StringRequest` with `inputString` | `StringResponse` with upperCase, lowerCase, length, trimmed, category |

---

## 🏗️ Folder Structure

```
SBStringManipulation/
│
├── Pipelines/
│   └── StringManipulationPP.pipeline     # Pipeline with string operation logic
│
├── ProxyServices/
│   └── StringManipulationPS.proxy              # Exposes the SOAP endpoint to clients
│
├── Schemas/
│   └── StringManipulation.xsd                   # Defines StringRequest and StringResponse elements
│
├── WSDLs/
│   └── StringManipulation.wsdl                  # Service contract with processString operation
│
├── pom.xml                                       # Maven build configuration
│
└── SBStringManipulation.jpr                      # JDeveloper project file
```

---

## 🚀 Implementation Steps

### Step 1: Create Service Bus Application and Project

1. Open **Oracle JDeveloper**
2. File → New → **Service Bus Application with Service Bus Project**
3. Application Name: `SBStringManipulationApp`
4. Project Name: `SBStringManipulation`
5. Click **Finish**

---

### Step 2: Create XSD Schema

1. Right-click on **Schemas** folder → New → **XML Schema**
2. File Name: `StringManipulation.xsd`
3. Open in **Source view** and define the following elements:
   - `StringRequest` — contains `inputString` (string)
   - `StringResponse` — contains `upperCase`, `lowerCase`, `stringLength`, `trimmed`, `concatenated`, `category`
4. Set `targetNamespace` to: `http://practice.learning.com/osb/stringmanipulation/schema`
5. Set `elementFormDefault="qualified"`

---

### Step 3: Create WSDL

1. Right-click on **WSDLs** folder → New → **WSDL Document**
2. File Name: `StringManipulation.wsdl`
3. Open in **Source view** and configure:
   - Import the XSD schema using `<xsd:import>`
   - Define 2 messages: `StringRequestMsg`, `StringResponseMsg`
   - Define `StringManipulationPortType` with 1 operation: `processString`
   - Add `document/literal` SOAP binding
   - Add `<service>` block with `soap:address`
4. Set `targetNamespace` to: `http://practice.learning.com/osb/stringmanipulation/wsdl`

---

### Step 4: Create Pipeline

1. Open the **Service Bus Project** canvas (double-click `SBStringManipulation.jpr`)
2. From **Component Palette** → drag **Pipeline** into the **Pipelines/Split Joins** swim lane
3. Configure the Pipeline wizard:
   - Service Name: `StringManipulationPP`
   - Service Type: **WSDL Based Service** → browse and select `StringManipulation.wsdl`
   - Port: `StringManipulationPort`
   - **Uncheck** `Expose as Proxy Service` — we create the Proxy Service manually in Step 7
4. Click **Finish**

---

### Step 5: Declare Pipeline Variables

Before designing the pipeline logic, declare shared pipeline variables so they are accessible across all stages.

1. In the pipeline canvas, locate the **Variables** section (usually accessible via pipeline properties or right-clicking the pipeline header)
2. Add the following variables (name only — OSB infers type at runtime):

| Variable Name     | Inferred Type at Runtime |
| ----------------- | ------------------------ |
| `varUpperCase`    | string                   |
| `varLowerCase`    | string                   |
| `varLength`       | integer                  |
| `varTrimmed`      | string                   |
| `varConcatenated` | string                   |

> **Note:** OSB pipeline variables are loosely typed. You only define the name — the type is determined at runtime based on the value assigned. These variables appear alongside built-in context variables like `body`, `header`, `fault` in the Assign action dropdown.

---

### Step 6: Design Pipeline Logic

1. Double-click `StringManipulationPP` to open the Pipeline editor

#### 6.1 Add Stage to Request Pipeline — String Operations

1. Drag a **Stage** node into the **Request Pipeline**
2. Name it: `Stage_StringOperations`
3. Double-click to open it and add **5 Assign actions** (from Message Processing):

| Assign # | Variable          | Expression                                                       |
| -------- | ----------------- | ---------------------------------------------------------------- |
| 1        | `varUpperCase`    | `fn:upper-case($body/*:StringRequest/*:inputString)`             |
| 2        | `varLowerCase`    | `fn:lower-case($body/*:StringRequest/*:inputString)`             |
| 3        | `varLength`       | `fn:string-length($body/*:StringRequest/*:inputString)`          |
| 4        | `varTrimmed`      | `fn:normalize-space($body/*:StringRequest/*:inputString)`        |
| 5        | `varConcatenated` | `fn:concat($body/*:StringRequest/*:inputString, ' - processed')` |

#### 6.2 Add Stage to Request Pipeline — Switch Case

1. Drag a second **Stage** node into the **Request Pipeline** below the first
2. Name it: `Stage_SwitchCase`
3. Double-click to open it and drag an **If Then** action (from Flow Control)
4. Configure the branches:

**If condition** (Short String):

```xpath
fn:string-length($body/*:StringRequest/*:inputString) <= 5
```

- Add a **Log** action inside: `fn:concat("Short String - Length is: ", fn:string-length($body/*:StringRequest/*:inputString))`
- Severity: `Info`

**Else If condition** (Medium String):

```xpath
fn:string-length($body/*:StringRequest/*:inputString) <= 10
```

- Add a **Log** action inside: `fn:concat("Medium String - Length is: ", fn:string-length($body/*:StringRequest/*:inputString))`
- Severity: `Info`

**Else** (Long String):

- Add a **Log** action inside: `fn:concat("Long String - Length is: ", fn:string-length($body/*:StringRequest/*:inputString))`
- Severity: `Info`

#### 6.3 Add Stage to Response Pipeline — Build Response

1. Drag a **Stage** node into the **Response Pipeline**
2. Name it: `Stage_BuildResponse`
3. Double-click to open it and drag a **Replace** action (from Message Processing)
4. Configure the Replace action:

| Field          | Value               |
| -------------- | ------------------- |
| Variable       | `body`              |
| XPath          | `*[1]`              |
| Replace Option | Replace Entire Node |

**Expression:**

```xpath
<sch:StringResponse
    xmlns:sch="http://practice.learning.com/osb/stringmanipulation/schema">
    <sch:upperCase>{$varUpperCase}</sch:upperCase>
    <sch:lowerCase>{$varLowerCase}</sch:lowerCase>
    <sch:stringLength>{$varLength}</sch:stringLength>
    <sch:trimmed>{$varTrimmed}</sch:trimmed>
    <sch:concatenated>{$varConcatenated}</sch:concatenated>
    <sch:category>
        {
            if (fn:string-length($body/*:StringRequest/*:inputString) <= 5)
            then "Short String"
            else if (fn:string-length($body/*:StringRequest/*:inputString) <= 10)
            then "Medium String"
            else "Long String"
        }
    </sch:category>
</sch:StringResponse>
```

---

### Step 7: Create Proxy Service

1. From **Component Palette**, drag **Proxy Service** into the **Proxy Services** swim lane (left column)
2. Configure the wizard:
   - Service Name: `StringManipulationPS`
   - Service Type: **WSDL Based Service** → browse and select `StringManipulation.wsdl`
   - Port: `StringManipulationPort`
   - Transport: `HTTP`
3. Click **Finish**

---

### Step 8: Wire Proxy Service to Pipeline

1. On the main canvas, hover over `StringManipulationPS`
2. A **yellow connector arrow** appears on its right edge
3. Click and drag it across to `StringManipulationPP`
4. Release — a connection line appears linking both components

---

### Step 9: Deploy the Project

1. **Start WebLogic Server** (if not running):
   - Right-click on **IntegratedWebLogicServer** in Application Server Navigator
   - Select **Start Server Instance**
   - Wait until you see `Server started in RUNNING mode`
2. **Deploy the Project:**
   - Right-click on **SBStringManipulation** project
   - Select **Deploy → SBStringManipulation**
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
   - **Project Explorer** → **SBStringManipulation** → **Proxy Services** → **StringManipulationPS**
4. Click **Launch Test Console** button

---

#### ✅ Test 1 — Short String (length ≤ 5)

**Request:**

```xml
<soapenv:Body>
    <sch:StringRequest xmlns:sch="http://practice.learning.com/osb/stringmanipulation/schema">
        <sch:inputString>Hello</sch:inputString>
    </sch:StringRequest>
</soapenv:Body>
```

**Expected Response:**

```xml
<sch:StringResponse>
    <sch:upperCase>HELLO</sch:upperCase>
    <sch:lowerCase>hello</sch:lowerCase>
    <sch:stringLength>5</sch:stringLength>
    <sch:trimmed>Hello</sch:trimmed>
    <sch:concatenated>Hello - processed</sch:concatenated>
    <sch:category>Short String</sch:category>
</sch:StringResponse>
```

---

#### ✅ Test 2 — Medium String (length 6–10)

**Request:**

```xml
<soapenv:Body>
    <sch:StringRequest xmlns:sch="http://practice.learning.com/osb/stringmanipulation/schema">
        <sch:inputString>HelloWorld</sch:inputString>
    </sch:StringRequest>
</soapenv:Body>
```

**Expected Response:**

```xml
<sch:StringResponse>
    <sch:upperCase>HELLOWORLD</sch:upperCase>
    <sch:lowerCase>helloworld</sch:lowerCase>
    <sch:stringLength>10</sch:stringLength>
    <sch:trimmed>HelloWorld</sch:trimmed>
    <sch:concatenated>HelloWorld - processed</sch:concatenated>
    <sch:category>Medium String</sch:category>
</sch:StringResponse>
```

---

#### ✅ Test 3 — Long String (length > 10)

**Request:**

```xml
<soapenv:Body>
    <sch:StringRequest xmlns:sch="http://practice.learning.com/osb/stringmanipulation/schema">
        <sch:inputString>Hello Umair Faheem</sch:inputString>
    </sch:StringRequest>
</soapenv:Body>
```

**Expected Response:**

```xml
<sch:StringResponse>
    <sch:upperCase>HELLO UMAIR FAHEEM</sch:upperCase>
    <sch:lowerCase>hello umair faheem</sch:lowerCase>
    <sch:stringLength>18</sch:stringLength>
    <sch:trimmed>Hello Umair Faheem</sch:trimmed>
    <sch:concatenated>Hello Umair Faheem - processed</sch:concatenated>
    <sch:category>Long String</sch:category>
</sch:StringResponse>
```

---

#### ✅ Test 4 — String with extra whitespace (tests normalize-space)

**Request:**

```xml
<soapenv:Body>
    <sch:StringRequest xmlns:sch="http://practice.learning.com/osb/stringmanipulation/schema">
        <sch:inputString>  hi  </sch:inputString>
    </sch:StringRequest>
</soapenv:Body>
```

**Expected Response:**

- `trimmed` = `hi` (leading/trailing spaces removed)
- `stringLength` = `6` (original length including spaces)
- `category` = `Short String`

> 💡 This test verifies that `fn:normalize-space()` correctly strips surrounding whitespace while `fn:string-length()` captures the original raw length.

---

## 🐛 Challenges and Issues

**Issue 1:** `OSB-395105: The TokenIterator does not correspond to a single XmlObject value`

**Error:**

```
OSB Assign action failed updating variable "body": [OSB-395105]
The TokenIterator does not correspond to a single XmlObject value
```

**Root Cause:** The namespace prefix used in XPath expressions (`inp1:`, `sch:`) was not declared in the Expression Editor's Namespace Bindings table, so OSB could not resolve the element path and returned multiple or zero nodes.

**Solution 1 — Declare namespace in Expression Editor:**  
In the Expression Builder, scroll to the **Namespace Bindings** section and add:

- Prefix: `sch`
- URI: `http://practice.learning.com/osb/stringmanipulation/schema`

Then use `sch:StringRequest/sch:inputString` in the expression.

**Solution 2 — Use wildcard namespace syntax (Recommended):**  
Use `*:localname` to match elements regardless of namespace prefix:

```xpath
fn:upper-case($body/*:StringRequest/*:inputString)
```

This is the most resilient approach as it does not depend on prefix declarations.

---

**Issue 2:** Assign action had no visible variable name field

**Root Cause:** OSB's built-in Assign action only targets **existing context variables** (`body`, `header`, `fault`, etc.). Custom variables like `varUpperCase` must be **declared first** at the pipeline level before they appear in the Assign dropdown.

**Solution:** Declare all custom variables in the pipeline's **Variables** section first. Once declared, they appear alongside `body`, `header`, and `fault` in the Assign action's variable selector.

---

**Issue 3:** `StringResponse` nested inside `StringRequest` in response body

**Root Cause:** The Replace action's XPath was set to `$body/*[1]` instead of `*[1]`. Since the Variable field was already set to `body`, using `$body/*[1]` in the XPath caused double-referencing, and OSB replaced the content **inside** `StringRequest` rather than replacing the entire node.

**Solution:** Set the XPath to `*[1]` (relative to the `body` variable, not absolute). This correctly targets and replaces the first child element of `$body` — which is `StringRequest` — with the new `StringResponse` element.

| Field    | Wrong        | Correct |
| -------- | ------------ | ------- |
| Variable | `body`       | `body`  |
| XPath    | `$body/*[1]` | `*[1]`  |

---

**Issue 4:** Routing Table caused cycle warning or duplicate proxy service

**Root Cause:** Routing Table in OSB is designed to route to **external Business Services**. Using it without a target service, or routing back to the same pipeline/proxy, causes cycle detection warnings or unexpected behavior.

**Solution:** Replace the Routing Table with an **If-Then action** inside a Stage for internal conditional processing. Use Routing Table only when routing to different external backend services based on conditions.

---

## 🪜 Enhancements for This Project

- Add a **Validate** node before `Stage_StringOperations` to enforce XSD schema compliance on incoming requests
- Add an **Error Handler** on the Pipeline Pair to catch and return a structured SOAP Fault for invalid input
