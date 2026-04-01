> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Contoso Smart Workspace — Complete Setup Guide

> A comprehensive step-by-step guide for participants to build the complete multi-agent system from scratch. Follow each phase in order.

---

## Use Case: "Contoso Smart Workspace" — Multi-Agent with Connectors

A comprehensive internal-facing system for Contoso employees (financial advisors, operations staff, managers) that uses **multiple specialized agents** orchestrated together, with **connectors** to SharePoint, Outlook, Dataverse, and more.

### Business Scenario

Contoso financial advisors need to:
1. Look up client policy information (stored in **SharePoint**)
2. Get product recommendations based on client profiles
3. Schedule follow-up meetings with clients (via **Outlook Connector**)
4. Log interaction summaries back to **Dataverse**
5. Escalate complex cases to the underwriting team

Instead of one monolithic agent, we build **specialized agents** that each handle a domain, orchestrated by a primary agent.

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                   USER (Financial Advisor)                    │
│                          │                                   │
│                          ▼                                   │
│         ┌────────────────────────────────┐                   │
│         │   🤖 Primary Agent             │                   │
│         │   "Contoso Advisor Hub"       │                   │
│         │   (Orchestrator)               │                   │
│         └────────┬───────┬───────┬───────┘                   │
│                  │       │       │                            │
│         ┌───────▼┐  ┌───▼────┐ ┌▼────────┐                  │
│         │ Agent 2 │  │Agent 3 │ │ Agent 4  │                 │
│         │ Policy  │  │Product │ │Scheduling│                 │
│         │ Lookup  │  │Advisor │ │Assistant │                 │
│         └───┬─────┘  └───┬────┘ └──┬───┬──┘                 │
│             │            │         │   │                     │
│     ┌───────▼──┐    ┌────▼───┐  ┌──▼───┘  ┌───────────┐     │
│     │SharePoint│    │SharePt │  │ Outlook  │ Dataverse │     │
│     │List:     │    │Library:│  │Connector │ Connector │     │
│     │ClientPol.│    │Product │  │(Calendar │ (Log      │     │
│     │(Data)    │    │Guides  │  │ + Email) │  Interact)│     │
│     └──────────┘    │(Knowl.)│  └──────────┘───────────┘     │
│                     └────────┘                               │
│                        ▲                                     │
│            Hub Agent also uses this as Knowledge             │
└──────────────────────────────────────────────────────────────┘
```

### Key Concepts Reference

| Concept | What It Is | How It Shows in This Setup |
|---------|-----------|--------------------------|
| **Multi-Agent** | Multiple specialized agents that can call each other | Advisor Hub orchestrates Policy Lookup, Product Advisor, and Scheduling agents |
| **Connectors** | Pre-built integrations to external services (Microsoft 365, third-party APIs) | SharePoint for documents, Outlook for calendar, Dataverse for CRM |
| **Actions** | Specific operations an agent can perform (call a connector, run a flow) | Create calendar event, query SharePoint list, log interaction to Dataverse |
| **Agent Transfer** | Primary agent hands off to a specialized sub-agent | "Look up my client's policy" → transfers to Policy Lookup Agent |
| **Generative Orchestration** | The primary agent decides which sub-agent to call based on intent | User says "schedule a meeting" → routes to Scheduling Assistant |
| **Topics** | Structured conversation paths for specific workflows | Policy lookup, product recommendation, meeting scheduling, follow-up email |
| **Variables** | Data collected, passed between agents, and used in connector calls | Client name, policy number, meeting date, email address |
| **Slot Filling (Topic Inputs)** | Variables marked as "Receive values from other topics" so the orchestrator can pre-fill them from the user's message | User says "check policy for David Chan" → sub-agent skips asking for client name |
| **Knowledge** | Documents the agent searches to generate answers | Product guides, underwriting guidelines, compliance policies |
| **Instructions** | Rules and personality for each agent | Each agent has its own specialized instructions |

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access
- [ ] Power Apps / Dataverse access
- [ ] Outlook (Exchange Online) enabled

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint List: `ClientPolicies`

1. Go to your **SharePoint site** → **New** → **List** → name it `ClientPolicies`
2. Add columns:

| Column Name | Type | Example Value |
|-------------|------|---------------|
| ClientName | Single line of text *(uses the default "Title" column — rename it)* | "David Chan" |
| ClientEmail | Single line of text | "david.chan@email.com" |
| PolicyNumber | Single line of text | "CTO-2025-00123" |
| PolicyType | Choice: `Life`, `Health`, `Travel`, `Critical Illness` | "Life" |
| CoverageAmount | Currency | 1,000,000 |
| PolicyStatus | Choice: `Active`, `Pending`, `Expired`, `Under Review` | "Active" |
| StartDate | Date | 2024-01-15 |
| EndDate | Date | 2034-01-15 |
| AssignedAdvisor | Single line of text | "Sarah Wong" |
| LastInteraction | Date | 2025-12-01 |
| Notes | Multiple lines of text | "Client interested in adding critical illness rider" |

3. Add sample data:

| ClientName | ClientEmail | PolicyNumber | PolicyType | CoverageAmount | PolicyStatus | StartDate | EndDate | AssignedAdvisor | LastInteraction | Notes |
|-----------|------------|-------------|-----------|---------------|-------------|-----------|---------|----------------|----------------|-------|
| David Chan | david.chan@email.com | CTO-2025-00123 | Life | 1,000,000 | Active | 2024-01-15 | 2034-01-15 | Sarah Wong | 2025-12-01 | Client interested in adding critical illness rider |
| Emily Lau | emily.lau@email.com | CTO-2025-00456 | Health | 500,000 | Active | 2025-03-01 | 2026-03-01 | Sarah Wong | 2026-01-15 | Renewed for second year; added spouse coverage |
| Michael Tam | michael.tam@email.com | CTO-2024-00789 | Critical Illness | 800,000 | Under Review | 2024-06-10 | 2044-06-10 | James Li | 2026-02-20 | Pending additional medical report from specialist |
| Karen Ho | karen.ho@email.com | CTO-2025-00234 | Travel | 200,000 | Active | 2025-11-01 | 2026-11-01 | Sarah Wong | 2025-11-01 | Annual multi-trip plan; travels frequently to Japan |
| Alex Yeung | alex.yeung@email.com | CTO-2023-00567 | Life | 2,000,000 | Expired | 2013-09-01 | 2023-09-01 | James Li | 2023-08-15 | Policy expired; client did not respond to renewal notices |

### 1.2 Create SharePoint Document Library: `ProductGuides`

1. **New** → **Document library** → name it `ProductGuides`
2. Upload these 3 documents (create them from the content in **Appendix B** at the end of this guide):
   - `Contoso_Product_FAQ.docx`
   - `Underwriting_Guidelines.docx`
   - `Compliance_Policy.docx`

> 💡 **Why SharePoint instead of direct upload?** This is more realistic for enterprise scenarios — documents are maintained in one place (SharePoint), and when the business team updates a document there, the agent's knowledge updates automatically. It also showcases SharePoint as both a **data connector** (the `ClientPolicies` list) and a **knowledge source** (the `ProductGuides` library).

### 1.3 Create Dataverse Table: `InteractionLog`

1. Open **Power Apps** → **Tables** → **New table** → name it `InteractionLog`
2. Add columns:

| Column | Type |
|--------|------|
| ClientName | Text |
| AdvisorName | Text |
| InteractionDate | Date |
| InteractionType | Choice: `Call`, `Meeting`, `Email`, `Chat` |
| Summary | Multiline Text |
| FollowUpRequired | Yes/No |
| FollowUpDate | Date |

3. Note the **numeric Choice values** for InteractionType (Power Apps → Tables → InteractionLog → Edit column → Choices). Default values are typically:
   - Call = 890,950,000
   - Meeting = 890,950,001
   - Email = 890,950,002
   - Chat = 890,950,003

> ⚠️ Your values may differ. Record them — you'll need them for the Power Automate flow.

---

## Phase 2: Build Agent 1 — Contoso Advisor Hub (Orchestrator)

This is the **main agent** that financial advisors interact with. It routes requests to specialized sub-agents.

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Contoso Advisor Hub"**
3. Description: *"Central hub for Contoso financial advisors. Routes to specialized agents for policy lookup, product advice, and scheduling."*

### 2.2 Set Instructions

Paste this into the Instructions field:

```
You are the Contoso Advisor Hub, an AI assistant built for Contoso financial advisors.

Your role is to help advisors work more efficiently by:
- Looking up client policy information
- Providing product recommendations and answering product questions
- Scheduling meetings and sending follow-up emails to clients
- Logging interaction summaries

You coordinate with specialized agents:
- For policy lookups and client data queries → transfer to the Policy Lookup Agent
- For product questions, comparisons, and recommendations → transfer to the Product Advisor Agent
- For scheduling meetings and sending emails → transfer to the Scheduling Assistant Agent

Rules:
- Always confirm the client name or policy number before performing any lookup.
- Protect client confidentiality — never share one client's data with another.
- When you are unsure which agent to use, ask the advisor to clarify their request.
- Use a professional but friendly tone suitable for internal staff.
- If the advisor asks about underwriting or compliance, answer from the knowledge sources provided.
```

### 2.3 Add Knowledge Sources (via SharePoint)

1. Go to **Knowledge** → **+ Add knowledge**
2. Select **"SharePoint"** as the source
3. Enter the URL of your SharePoint site
4. Select the **`ProductGuides`** document library
5. Choose the documents to index:
   - `Contoso_Product_FAQ.docx`
   - `Underwriting_Guidelines.docx`
   - `Compliance_Policy.docx`
6. Click **Add** and wait for indexing to complete

### 2.4 Enable Generative Orchestration

1. Go to **Settings** → **Generative AI** → Enable **Generative orchestration**
2. This allows the primary agent to dynamically decide which sub-agent to invoke based on user intent

---

## Phase 3: Build Agent 2 — Policy Lookup Agent (Sub-Agent + SharePoint Connector)

This agent queries the SharePoint `ClientPolicies` list to retrieve client information.

### 3.1 Create the Agent

1. **Create** → **New agent**
2. Name: **"Policy Lookup Agent"**
3. Description: *"Looks up client policy details from SharePoint for financial advisors."*

### 3.2 Set Instructions

Paste this into the Instructions field:

```
You are the Policy Lookup Agent. Your job is to help Contoso financial advisors find client policy information.

You can:
- Search for a client by name or policy number
- Return policy details including type, coverage, status, and dates
- Identify policies that are expiring soon or under review

Rules:
- Always confirm the search criteria with the advisor before querying.
- Present results in a clear, structured format.
- If multiple results are found, list them and ask the advisor to select.
- Never modify client data — you are read-only.
- If no results are found, suggest checking the spelling or trying a different search.
```

### 3.3 Create Power Automate Flow: "Search Client Policies"

In Copilot Studio, connector actions (like querying SharePoint) are done via **Power Automate cloud flows**. You create the flow first, then call it from within a Topic.

1. Open your **Policy Lookup Agent** in Copilot Studio
2. Go to **Topics** → open any topic (or create a new one)
3. In the topic flow, click the **+** (Add node) button → select **"Call an action"**
4. Click **"Create a flow"** — this opens **Power Automate** in a new tab
5. Build the flow:

   **Flow name**: `Search Client Policies`

   | Step | Action | Configuration |
   |------|--------|---------------|
   | 1 | **When Copilot Studio calls a flow** (trigger — auto-added) | Add **two** text inputs: `SearchField` (Text) and `SearchValue` (Text) |
   | 2 | **Compose** — "Build Filter Query" | Click **Expression** tab and enter: `if(equals(triggerBody()?['text'],'PolicyNumber'),concat('PolicyNumber eq ''',triggerBody()?['text_1'],''''),concat('Title eq ''',triggerBody()?['text_1'],''''))` — Note: `text` = SearchField, `text_1` = SearchValue (verify order in your trigger) |
   | 3 | **SharePoint — Get items** | **Site Address**: your SharePoint site URL |
   |   |  | **List Name**: `ClientPolicies` |
   |   |  | **Filter Query**: select the **Outputs** from "Build Filter Query" via Dynamic Content (do NOT type — select the blue tag) |
   | 4 | **Compose** — "Get First Result" | Click **Expression** tab: `first(outputs('Get_items')?['body/value'])` |
   | 5 | **Return value(s) to Copilot Studio** | Add outputs using Expression tab for each: |

   **Return value expressions** (click Expression tab for each output):

   | Output Name | Type | Expression |
   |------------|------|-----------|
   | ClientName | Text | `outputs('Get_First_Result')?['Title']` |
   | PolicyNumber | Text | `outputs('Get_First_Result')?['PolicyNumber']` |
   | PolicyType | Text | `outputs('Get_First_Result')?['PolicyType']?['Value']` |
   | CoverageAmount | Text | `string(outputs('Get_First_Result')?['CoverageAmount'])` |
   | PolicyStatus | Text | `outputs('Get_First_Result')?['PolicyStatus']?['Value']` |
   | StartDate | Text | `outputs('Get_First_Result')?['StartDate']` |
   | EndDate | Text | `outputs('Get_First_Result')?['EndDate']` |
   | ClientEmail | Text | `outputs('Get_First_Result')?['ClientEmail']` |
   | AssignedAdvisor | Text | `outputs('Get_First_Result')?['AssignedAdvisor']` |
   | Notes | Text | `outputs('Get_First_Result')?['Notes']` |

6. **Save** the flow → go back to Copilot Studio
7. The flow now appears as an available action when you click **+** → **"Call an action"** in any topic

> 💡 **Why Power Automate?** In Copilot Studio, connector actions (SharePoint, Outlook, Dataverse, etc.) are accessed through Power Automate cloud flows. The flow acts as a bridge between the agent and the connector. This is actually a strength — business users can build and test flows independently.

> ⚠️ **Key lessons learned during setup:**
> - SharePoint internal column names may differ from display names (e.g., the first column "ClientName" has internal name `Title`). Always verify by running a Get Items with no filter and checking raw output.
> - SharePoint Get Items returns an **array** — you must use `first()` in a Compose step to extract the first item before returning values.
> - Choice columns (PolicyType, PolicyStatus) return objects — use `?['Value']` to get the text.
> - Currency columns return numbers — wrap in `string()` since return type is Text.
> - Filter Query in Copilot Studio must use **dynamic content tags** (blue pills), not typed text with `+` signs.

### 3.4 Create Topic: "Lookup by Client Name"

**Trigger phrases:**
- "Find client"
- "Look up policy"
- "Search for client"
- "Client information"
- "Policy details"

**Variables used:**

| Variable | Type | Scope | Used In |
|----------|------|-------|---------|
| Topic.SearchMethod | Choice | Topic | Search method selection |
| Topic.ClientName | String | Topic | User input (name) & flow output (name from left path) |
| Topic.PolicyNumber | String | Topic | Flow output (policy number from left path) |
| Topic.PolicyNumber1 | String | Topic | User input (policy number from right path) |
| Topic.ReturnedClientName | String | Topic | Flow output (client name from right path) |
| Global.SearchFieldValue | String | Global | Passed to flow as SearchField ("Title" or "PolicyNumber") |
| Topic.PolicyType, CoverageAmount, etc. | String | Topic | Flow outputs (shared by both paths) |

**Topic Flow:**

```
[Trigger] →

[Message] "Sure! Let me search for the client. How would you like to search?"

→ [Ask a question] "Search by:"
   - Multiple choice: Client Name / Policy Number
   - Save to: Topic.SearchMethod

→ [Condition] If Topic.SearchMethod = "Client Name":

   → [Ask a question] "What is the client's name?"
      - Identify: User's entire response
      - Save to: Topic.ClientName

   → [Set variable] Global.SearchFieldValue = "Title"

   → [Call Action: Search Client Policies flow]
      - SearchField = Global.SearchFieldValue
      - SearchValue = Topic.ClientName
      - Save outputs to: Topic.ClientName, Topic.PolicyNumber, Topic.PolicyType,
        Topic.CoverageAmount, Topic.PolicyStatus, Topic.StartDate, Topic.EndDate,
        Topic.ClientEmail, Topic.AssignedAdvisor, Topic.Notes

   → [Condition] If Topic.PolicyNumber is not blank:
      → [Message]
         "Here's what I found for {Topic.ClientName}:
         📋 Policy Number: {Topic.PolicyNumber}
         📦 Policy Type: {Topic.PolicyType}
         💰 Coverage: {Topic.CoverageAmount}
         ✅ Status: {Topic.PolicyStatus}
         📅 Valid: {Topic.StartDate} to {Topic.EndDate}
         📧 Email: {Topic.ClientEmail}
         👤 Assigned Advisor: {Topic.AssignedAdvisor}
         📝 Notes: {Topic.Notes}"
      → [End current topic]

   → Else:
      → [Message] "I couldn't find any records for '{Topic.ClientName}'.
         Please check the spelling or try searching by policy number."
      → [End current topic]

→ [Condition] If Topic.SearchMethod = "Policy Number":

   → [Ask a question] "What is the policy number?"
      - Identify: User's entire response
      - Save to: Topic.PolicyNumber1

   → [Set variable] Global.SearchFieldValue = "PolicyNumber"

   → [Call Action: Search Client Policies flow]
      - SearchField = Global.SearchFieldValue
      - SearchValue = Topic.PolicyNumber1
      - Save outputs to: Topic.ReturnedClientName, Topic.PolicyNumber1, Topic.PolicyType,
        Topic.CoverageAmount, Topic.PolicyStatus, Topic.StartDate, Topic.EndDate,
        Topic.ClientEmail, Topic.AssignedAdvisor, Topic.Notes

   → [Condition] If Topic.ReturnedClientName is not blank:
      → [Message]
         "Here's what I found for policy {Topic.PolicyNumber1}:
         👤 Client Name: {Topic.ReturnedClientName}
         📦 Policy Type: {Topic.PolicyType}
         💰 Coverage: {Topic.CoverageAmount}
         ✅ Status: {Topic.PolicyStatus}
         📅 Valid: {Topic.StartDate} to {Topic.EndDate}
         📧 Email: {Topic.ClientEmail}
         👤 Assigned Advisor: {Topic.AssignedAdvisor}
         📝 Notes: {Topic.Notes}"
      → [End current topic]

   → Else:
      → [Message] "I couldn't find any records for policy number {Topic.PolicyNumber1}.
         Please check the number or try searching by client name."
      → [End current topic]
```

> 💡 **Concepts highlighted:**
> - **Variables** — Topic variables (`Topic.ClientName`, `Topic.PolicyNumber1`) reset each time the topic triggers. Global variables (`Global.SearchFieldValue`) persist across the session and are used to control which SharePoint column the flow filters by.
> - **Branching** — Each condition branch must end with "End current topic" to prevent fall-through.
> - **Power Automate integration** — The same flow is called from both branches with different SearchField values ("Title" vs "PolicyNumber"), demonstrating dynamic connector behavior.

**Save** the topic.

---

## Phase 4: Build Agent 3 — Product Advisor Agent (Sub-Agent + Knowledge)

This agent is the product expert — it answers questions and makes recommendations using uploaded product documents.

### 4.1 Create the Agent

1. **Create** → **New agent**
2. Name: **"Product Advisor Agent"**
3. Description: *"Provides product information, comparisons, and recommendations for Contoso financial advisors."*

### 4.2 Set Instructions

Paste this into the Instructions field:

```
You are the Product Advisor Agent. You help Contoso financial advisors understand products and make recommendations to their clients.

You can:
- Answer detailed questions about Life, Health, Travel, and Critical Illness insurance products
- Compare products side by side
- Recommend products based on a client's age, family situation, and needs
- Explain underwriting requirements and exclusions

Rules:
- Base all answers on the knowledge sources provided to you. Do not make up product details.
- When recommending products, always explain the reasoning.
- Present comparisons in a table format for easy reading.
- Always note that final product suitability is determined by the advisor and the client.
- Flag any underwriting concerns (e.g., pre-existing conditions, age limits).
```

### 4.3 Add Knowledge Sources (via SharePoint)

Reuse the same SharePoint Document Library as the knowledge source:

1. Go to **Knowledge** → **+ Add knowledge**
2. Select **"SharePoint"** as the source
3. Enter the URL of your SharePoint site
4. Select the **`ProductGuides`** document library
5. Choose:
   - `Contoso_Product_FAQ.docx`
   - `Underwriting_Guidelines.docx`
6. Click **Add** and wait for indexing to complete

> 💡 Both the Hub agent and the Product Advisor agent point to the same SharePoint library. This means updating a document in SharePoint once updates knowledge for all agents — single source of truth.

### 4.4 Create Topic: "Product Recommendation"

**Trigger phrases:**
- "Recommend a product"
- "What should I suggest"
- "Best product for"
- "Product recommendation"
- "What plan fits"

**Topic Flow:**

```
[Trigger] →

[Message] "I'll help you find the right product for your client. Let me gather some details."

→ [Ask a question] "What is the client's age?"
   - Identify: Number
   - Save to: Topic.ClientAge

→ [Ask a question] "What is the client's primary concern?"
   - Multiple choice:
     • Income protection for family
     • Medical expense coverage
     • Critical illness lump-sum benefit
     • Travel protection
     • Retirement planning
   - Save to: Topic.PrimaryConcern

→ [Ask a question] "Does the client have any pre-existing medical conditions?"
   - Multiple choice: Yes / No / Not sure
   - Save to: Topic.PreExisting

→ [Ask a question] "What is the client's approximate monthly budget for insurance?"
   - Multiple choice: Under HKD 500 / HKD 500-1500 / HKD 1500-3000 / Above HKD 3000
   - Save to: Topic.Budget

→ [Generative Answer Node — using Knowledge]
   Generate a recommendation based on:
   - Client age: {Topic.ClientAge}
   - Primary concern: {Topic.PrimaryConcern}
   - Pre-existing conditions: {Topic.PreExisting}
   - Budget: {Topic.Budget}

   Prompt: "Based on the following client profile, recommend the most suitable Contoso product(s) with reasoning:
   - Age: {Topic.ClientAge}
   - Primary concern: {Topic.PrimaryConcern}
   - Pre-existing conditions: {Topic.PreExisting}
   - Budget: {Topic.Budget}
   Use the product knowledge base to provide accurate details."

→ [Message] "Would you like me to compare specific plans side by side, or would you like to schedule a meeting with this client to discuss?"
```

> 💡 Link the SharePoint knowledge docs in the "Create generative answers" node: Click the node → right panel → **+ Add knowledge** → select your SharePoint docs.

> 💡 **Concepts**: Knowledge + Generative Answers, Variables driving AI responses, structured data collection

**Save** the topic.

### 4.5 Testing the Product Advisor Agent

After building the topic and configuring the "Create generative answers" node, test it in the **Test your agent** panel.

**Test 1 — Topic-based product recommendation:**

Type: `Recommend a product`

The agent will walk through each question:
- **Age**: `42`
- **Primary concern**: Select `Critical illness lump-sum benefit`
- **Pre-existing conditions**: Select `No`
- **Budget**: Select `HKD 1500-3000`

**Expected result** — the generative answers node should produce a structured recommendation referencing the product knowledge base with detailed coverage info, underwriting considerations, and plan tier comparisons.

**Test 2 — Direct knowledge question (no topic trigger):**

Type: `What types of life insurance does Contoso offer?`

→ Should answer from the FAQ document using the agent's generative mode (not the topic).

**Test 3 — Underwriting question:**

Type: `What are the age limits for critical illness coverage?`

→ Should answer from the Underwriting Guidelines document.

**Test 4 — Different client profile through the topic:**

Type: `Recommend a product`
- **Age**: `55`
- **Primary concern**: Select `Medical expense coverage`
- **Pre-existing conditions**: Select `Yes`
- **Budget**: Select `HKD 500-1500`

→ Should generate a different recommendation (likely Health insurance) and flag the 12-month pre-existing condition waiting period from the Underwriting Guidelines.

> 💡 **If Tests 2 and 3 work but the topic-based recommendation is weak**: The knowledge sources are connected at the agent level but may not be linked in the "Create generative answers" node. Click the node → right panel → **+ Add knowledge** → select your SharePoint docs.

---

## Phase 5: Build Agent 4 — Scheduling Assistant (Sub-Agent + Outlook & Dataverse Connectors)

This agent handles meeting scheduling and email follow-ups.

### 5.1 Create the Agent

1. **Create** → **New agent**
2. Name: **"Scheduling Assistant"**
3. Description: *"Schedules client meetings via Outlook and sends follow-up emails for Contoso advisors."*

### 5.2 Set Instructions

Paste this into the Instructions field:

```
You are the Scheduling Assistant for Contoso financial advisors.

You can:
- Schedule meetings with clients via Outlook calendar
- Send follow-up emails to clients after consultations
- Log interactions to the system

Rules:
- Always confirm the meeting details (date, time, attendees, subject) before creating the event.
- Default meeting duration is 30 minutes unless the advisor specifies otherwise.
- Include a professional Contoso email signature in all outgoing emails.
- Time zone is Hong Kong (UTC+8).
- Never send emails without the advisor's explicit confirmation.
```

### 5.3 Create Power Automate Flow: "Schedule Client Meeting"

#### Step A: Open Power Automate from Copilot Studio

1. Open the **Scheduling Assistant** agent in Copilot Studio
2. Go to **Topics** in the left navigation → click **+ Add a topic** → **From blank** (or open an existing topic)
3. In the topic canvas, click the **+** (Add node) button
4. Select **"Call an action"** from the menu
5. Click **"Create a flow"** — this opens **Power Automate** in a new browser tab with the Copilot Studio trigger pre-configured

#### Step B: Configure the Trigger — "When Copilot Studio calls a flow"

The trigger step is auto-added. You need to define the **input parameters** that Copilot Studio will send to this flow.

1. Click on the trigger step **"When Copilot Studio calls a flow"** (it may also say "When Power Virtual Agents calls a flow")
2. Click **"+ Add an input"** and add the following **five** inputs one by one:

   | Input # | Click "Add an input" → Select | Name it | Type |
   |---------|-------------------------------|---------|------|
   | 1 | **Text** | `Subject` | Text |
   | 2 | **Text** | `StartDateTime` | Text |
   | 3 | **Text** | `Duration` | Text |
   | 4 | **Text** | `AttendeeEmail` | Text |
   | 5 | **Text** | `Body` | Text |

   > ⚠️ **Important**: Name each input **exactly** as shown (case-sensitive). These names become the parameter labels when you call the flow from Copilot Studio.
   >
   > 💡 We use **Text** type for all inputs (including StartDateTime and Duration) because Copilot Studio passes values as strings. We will handle date/time formatting inside the flow.

#### Step C: Add a Compose Step — "Calculate End Time"

We need to calculate the meeting end time by adding the duration to the start time.

1. Click **"+ New step"** (or **+** between steps)
2. Search for **"Compose"** → select **"Compose"** (under Data Operations)
3. Rename this step to **`Calculate End Time`** (click the three dots **⋯** → **Rename**)
4. In the **Inputs** field, click on the **Expression** tab (fx) and enter:

   ```
   addMinutes(triggerBody()?['text_1'], if(equals(triggerBody()?['text_2'], '30 minutes'), 30, if(equals(triggerBody()?['text_2'], '45 minutes'), 45, 60)))
   ```

   > 📝 **How the expression works:**
   > - `triggerBody()?['text_1']` = the `StartDateTime` input (second input in the trigger)
   > - `triggerBody()?['text_2']` = the `Duration` input (third input in the trigger)
   > - The nested `if` checks the Duration string and adds the corresponding minutes
   > - Returns a new datetime string for the end time
   >
   > ⚠️ **Verify your input field names**: Power Automate names trigger inputs sequentially as `text` (1st), `text_1` (2nd), `text_2` (3rd), `text_3` (4th), `text_4` (5th). Click on the trigger step and hover over each input to confirm the internal names match. Your mapping should be:
   >
   > | Display Name | Internal Name |
   > |-------------|---------------|
   > | Subject | `text` |
   > | StartDateTime | `text_1` |
   > | Duration | `text_2` |
   > | AttendeeEmail | `text_3` |
   > | Body | `text_4` |

5. Click **OK** to save the expression

#### Step D: Add the Outlook Connector — "Create Event (V4)"

1. Click **"+ New step"**
2. Search for **"Office 365 Outlook"** → select **"Create event (V4)"**
3. If prompted, **sign in** to your Office 365 account to authorize the Outlook connection
4. Configure the fields as follows:

   | Field | How to Set It |
   |-------|---------------|
   | **Calendar id** | Select **"Calendar"** from the dropdown (this is your default calendar) |
   | **Subject** | Click in the field → switch to **Dynamic content** tab → select **`Subject`** (from the trigger — it shows as a blue tag) |
   | **Start time** | Click → **Dynamic content** → select **`StartDateTime`** (from the trigger) |
   | **End time** | Click → **Dynamic content** → select **`Outputs`** from the **"Calculate End Time"** Compose step |
   | **Time zone** | Select **`(UTC+08:00) Beijing, Chongqing, Hong Kong, Urumqi`** from the dropdown |
   | **Required attendees** | Click → **Dynamic content** → select **`AttendeeEmail`** (from the trigger) |
   | **Body** | Click → **Dynamic content** → select **`Body`** (from the trigger) |

   > 💡 **Tips:**
   > - For **Dynamic content**: Always click the blue tag from the panel — do NOT type the variable name manually.
   > - **Calendar id**: If you have multiple calendars, choose the one you want meetings to appear in. "Calendar" is the default.
   > - **Is HTML** (optional): You can expand **Advanced options** and set **Is HTML** to **Yes** if your Body contains HTML formatting.
   > - **Reminder** (optional): Expand **Advanced options** → set to `15` minutes if you want a default reminder.

#### Step E: Add the Return Step — "Return value(s) to Copilot Studio"

This step is usually auto-added at the bottom. If not, add it manually.

1. Scroll to the bottom of the flow — you should see **"Return value(s) to Power Virtual Agents"** (or "Return value(s) to Copilot Studio")
2. If it's missing: click **"+ New step"** → search for **"Return value(s) to Power Virtual Agents"** → select it
3. Click **"+ Add an output"** → select **Text**
4. Configure the output:

   | Field | Value |
   |-------|-------|
   | **Title** (output name) | `Confirmation` |
   | **Enter a value to respond** | Type: `Meeting created successfully` (plain text — not dynamic content) |

#### Step F: Name and Save the Flow

1. Click on the flow name at the top left (it may say "Untitled" or a generated name)
2. Rename it to: **`Schedule Client Meeting`**
3. Click **Save** in the top right corner
4. Wait for the "Your flow is ready" confirmation banner

#### Step G: Return to Copilot Studio and Link the Flow

1. Go back to your **Copilot Studio** browser tab
2. In the topic where you were building, click **+** → **"Call an action"**
3. Your newly saved flow **"Schedule Client Meeting"** should now appear in the list — click it
4. The action node will appear with the five input fields. You will map these to your topic variables when building the topic (covered in Section 5.7)

#### Complete Flow Summary (Visual Reference)

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER: When Copilot Studio calls a flow              │
│  Inputs:                                                │
│    • Subject       (Text)  ← "Contoso - Policy..."    │
│    • StartDateTime (Text)  ← "2026-03-25T14:00:00"     │
│    • Duration      (Text)  ← "30 minutes"              │
│    • AttendeeEmail (Text)  ← "david.chan@email.com"     │
│    • Body          (Text)  ← "Dear David Chan,..."     │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  COMPOSE: Calculate End Time                            │
│  Expression:                                            │
│    addMinutes(StartDateTime,                            │
│      if(Duration='30 minutes', 30,                      │
│        if(Duration='45 minutes', 45, 60)))              │
│  Output: End datetime string                            │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  OFFICE 365 OUTLOOK: Create event (V4)                  │
│    Calendar id:          Calendar                       │
│    Subject:              [Subject] (dynamic)            │
│    Start time:           [StartDateTime] (dynamic)      │
│    End time:             [Calculate End Time] (dynamic) │
│    Time zone:            UTC+08:00 Hong Kong            │
│    Required attendees:   [AttendeeEmail] (dynamic)      │
│    Body:                 [Body] (dynamic)               │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Return value(s) to Copilot Studio              │
│  Outputs:                                               │
│    • Confirmation (Text) = "Meeting created             │
│      successfully"                                      │
└─────────────────────────────────────────────────────────┘
```

#### Troubleshooting This Flow

| Issue | Solution |
|-------|----------|
| **"Calculate End Time" expression error** | Verify the internal input names (`text_1`, `text_2`) match your trigger order. Click the trigger step and hover over each input to see the internal name. |
| **Outlook connection error / "Forbidden"** | Click the "Create event" step → click the connection name → **Re-authenticate** with your Office 365 account. Ensure the account has Exchange Online / Outlook permissions. |
| **End time shows wrong value** | Make sure the `StartDateTime` value from Copilot Studio is in ISO 8601 format (e.g., `2026-03-25T14:00:00`). If the "Date and time" entity is used in the topic, it usually provides this format automatically. |
| **Calendar event created but attendee didn't receive invite** | Verify the email address format. Also check that your Office 365 account has permission to send meeting invites (some tenants restrict this). |
| **Flow not appearing in Copilot Studio** | Make sure the flow was saved successfully. Refresh the Copilot Studio page. The flow must use the "When Power Virtual Agents calls a flow" / "When Copilot Studio calls a flow" trigger to be visible. |
| **Duration not matching expected values** | The `if()` expression checks for exact strings `"30 minutes"`, `"45 minutes"`. Ensure the Multiple Choice options in your topic match these strings exactly. If they differ (e.g., `"30 mins"`), update the expression to match. |

> ⚠️ **Pre-authentication reminder**: Before testing, open this flow in Power Automate and do a **test run** to ensure the Outlook connection is active and authenticated. Connections can expire, especially in shared environments.

### 5.4 Create Power Automate Flow: "Send Follow-Up Email"

1. Click **+** → **"Call an action"** → **"Create a flow"**
2. Build the flow:

   **Flow name**: `Send Follow-Up Email`

   | Step | Action | Configuration |
   |------|--------|---------------|
   | 1 | **When Power Virtual Agents calls a flow** (trigger) | Add inputs: `ToEmail` (Text), `Subject` (Text), `EmailBody` (Text) |
   | 2 | **Office 365 Outlook — Send an email (V2)** | **To**: `ToEmail` input |
   |   |  | **Subject**: `Subject` input |
   |   |  | **Body**: `EmailBody` input |
   | 3 | **Return value(s) to Power Virtual Agents** | Output: `Confirmation` (Text) = "Email sent successfully" |

3. **Save** the flow

### 5.5 Create Power Automate Flow: "Log Interaction to Dataverse"

#### Step 1: Configure the Trigger

1. Click **+** → **"Call an action"** → **"Create a flow"**
2. Click on the trigger step **"When Copilot Studio calls a flow"**
3. Click **"+ Add an input"** and add the following **four** inputs:

   | Input # | Type | Name |
   |---------|------|------|
   | 1 | Text | `ClientName` |
   | 2 | Text | `InteractionType` |
   | 3 | Text | `Summary` |
   | 4 | Text | `FollowUpDate` |

#### Step 2: Add a Compose Step — "Map InteractionType to Choice Value"

> ⚠️ **Why this step is needed**: The `InteractionType` column in Dataverse is a **Choice** column, not a text column. Dataverse Choice fields require a **numeric value** (not the label text). You cannot directly pass a text string like `"Meeting"` — you must pass the corresponding integer value.
>
> The Choice values for `InteractionType` (visible in Power Apps → Tables → InteractionLog → Edit column → Choices) are typically:
>
> | Choice Label | Numeric Value |
> |-------------|---------------|
> | Call | 890,950,000 |
> | Meeting | 890,950,001 |
> | Email | 890,950,002 |
> | Chat | 890,950,003 |
>
> ⚠️ **Your values may differ!** Always verify the actual numeric values in your Dataverse table. Go to **Power Apps** → **Tables** → **InteractionLog** → click the **InteractionType** column → **Edit column** → scroll to **Choices** section to see the Label and Value pairs.

1. Click **"+ New step"**
2. Search for **"Compose"** → select **Compose** (under Data Operations)
3. Rename this step to **`Map InteractionType to Choice Value`** (click **⋯** → **Rename**)
4. In the **Inputs** field, click the **Expression** tab (fx) and enter:

   ```
   if(equals(triggerBody()?['text_1'],'Call'),890950000,if(equals(triggerBody()?['text_1'],'Meeting'),890950001,if(equals(triggerBody()?['text_1'],'Email'),890950002,if(equals(triggerBody()?['text_1'],'Chat'),890950003,890950001))))
   ```

   > 📝 **How the expression works:**
   > - `triggerBody()?['text_1']` = the `InteractionType` text input (second input in the trigger)
   > - The nested `if()` checks the text value and returns the matching numeric Choice value
   > - Default fallback is `890950001` (Meeting) if no match is found
   >
   > ⚠️ **Verify your input field name**: The internal name `text_1` corresponds to the second input. If your inputs are in a different order, adjust accordingly. Hover over the trigger inputs to confirm:
   >
   > | Display Name | Internal Name |
   > |-------------|---------------|
   > | ClientName | `text` |
   > | InteractionType | `text_1` |
   > | Summary | `text_2` |
   > | FollowUpDate | `text_3` |

5. Click **OK** to save the expression

#### Step 3: Add the Dataverse Connector — "Add a new row"

1. Click **"+ New step"**
2. Search for **"Microsoft Dataverse"** → select **"Add a new row"**
3. If prompted, sign in to authorize the Dataverse connection
4. Configure the fields:

   | Field | How to Set It |
   |-------|---------------|
   | **Table name** | Select **`InteractionLogs`** from the dropdown (it may appear as `InteractionLog` — select the one matching your table) |
   | **ClientName** | Click → **Dynamic content** → select **`ClientName`** from the trigger |
   | **InteractionType** | Click the field → click **"Enter custom value"** at the bottom of the dropdown → switch to **Dynamic content** tab → select **Outputs** from the **"Map InteractionType to Choice Value"** Compose step |
   | **Summary** | Click → **Dynamic content** → select **`Summary`** from the trigger |
   | **FollowUpRequired** | Select **Yes** from the dropdown |
   | **FollowUpDate** | Click → **Dynamic content** → select **`FollowUpDate`** from the trigger |

   > ⚠️ **Critical for InteractionType**: The Dataverse connector shows a dropdown with the choice labels (Call, Meeting, Email, Chat). **Do NOT select from this dropdown** — the labels won't work when the value comes dynamically at runtime. Instead:
   > 1. Click the **InteractionType** field
   > 2. You'll see the dropdown with Call / Chat / Email / Meeting options
   > 3. Click **"Enter custom value"** (the link at the bottom of the dropdown list)
   > 4. The field becomes a free-text input
   > 5. Click the field → open **Dynamic content** panel → select the **Outputs** from your **"Map InteractionType to Choice Value"** Compose step (it will appear as a blue tag)
   >
   > This passes the numeric value (e.g., `890950001`) which Dataverse correctly interprets as the Choice option.

   > 💡 **If you don't see all columns**: Click **"Show advanced options"** or check the **"Advanced parameters"** dropdown (e.g., "Showing 5 of 14") and click **"Show all"** to reveal hidden columns like FollowUpDate, FollowUpRequired, etc.

#### Step 4: Add the Return Step

1. Scroll to the bottom — the return step should be auto-added
2. Click **"+ Add an output"** → select **Text**
3. Set:
   - **Title**: `Confirmation`
   - **Value**: `Interaction logged`

#### Step 5: Name and Save

1. Rename the flow to: **`Log Interaction to Dataverse`**
2. Click **Save** → wait for confirmation

#### Complete Flow Summary (Visual Reference)

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER: When Copilot Studio calls a flow              │
│  Inputs:                                                │
│    • ClientName       (Text)  ← "David Chan"           │
│    • InteractionType  (Text)  ← "Meeting"              │
│    • Summary          (Text)  ← "Policy renewal..."    │
│    • FollowUpDate     (Text)  ← "2026-03-25"           │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  COMPOSE: Map InteractionType to Choice Value           │
│  Expression:                                            │
│    "Meeting" → 890950001                                │
│    "Call"    → 890950000                                │
│    "Email"   → 890950002                                │
│    "Chat"    → 890950003                                │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  DATAVERSE: Add a new row                               │
│    Table:            InteractionLog                     │
│    ClientName:       [ClientName] (dynamic)             │
│    InteractionType:  [Map Choice Value] (dynamic)       │
│                      → "Enter custom value" with        │
│                         numeric output from Compose     │
│    Summary:          [Summary] (dynamic)                │
│    FollowUpRequired: Yes                                │
│    FollowUpDate:     [FollowUpDate] (dynamic)           │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Return value(s) to Copilot Studio              │
│  Outputs:                                               │
│    • Confirmation (Text) = "Interaction logged"         │
└─────────────────────────────────────────────────────────┘
```

#### Troubleshooting This Flow

| Issue | Solution |
|-------|----------|
| **InteractionType shows dropdown instead of accepting dynamic value** | You must click **"Enter custom value"** at the bottom of the dropdown to switch to free-text mode, then use Dynamic content to insert the Compose output |
| **Error "Invalid value for Choice column"** | The numeric value doesn't match your Dataverse choices. Go to Power Apps → Tables → InteractionLog → Edit the InteractionType column → verify the exact numeric values under Choices, then update the Compose expression |
| **Dataverse "Add a new row" fails with 403** | Re-authenticate the Dataverse connection. Ensure your account has write access to the InteractionLog table |
| **FollowUpDate not saving correctly** | Ensure the date is passed in a recognized format (e.g., `2026-03-25` or ISO 8601). The Copilot Studio "Date and time" entity usually provides this automatically |
| **Columns not visible in the Dataverse step** | Click **"Show advanced options"** or expand the **"Advanced parameters"** dropdown (it may say "Showing 5 of 14") → click **"Show all"** |

### 5.6 Understanding How Variables Are Created in Copilot Studio

> ⚠️ **Important**: In Copilot Studio, you **cannot** pre-create variables from the Variables panel. The Variables panel (top toolbar) is only for **viewing and managing** existing variables — it does not have an "+ Add a variable" button.
>
> **Variables are created automatically** as you build your topic nodes:
>
> | How the Variable Gets Created | When It Happens |
> |-------------------------------|-----------------|
> | **"Ask a question"** node → **Save response as** → click **"Create new"** | When you set up a question node and choose where to save the user's answer |
> | **"Set a variable value"** node → **Set variable** → click **"Create new"** | When you add a variable management node and need a new variable to write to |
> | **"Call an action"** node → output mapping → click **"Create new"** | When mapping flow outputs to variables |
>
> **This means**: You will create variables **as you go** through the topic steps below. Each step tells you when and how to create the variable you need.
>
> 💡 **When creating a new variable**, always:
> 1. Click the **"Create new"** link in the variable picker dropdown
> 2. Enter the variable name (e.g., `ClientName` — Copilot Studio auto-prefixes with `Topic.`)
> 3. Scope: **Topic** (default)
>
> ⚠️ **Variable types are often auto-determined by the Identify entity** — you cannot override them:
>
> | Identify Entity | Auto-Assigned Variable Type | Can You Change It? |
> |----------------|---------------------------|-------------------|
> | User's entire response | **string** | N/A (already string) |
> | Email | **string** | N/A (already string) |
> | Date and time | **datetime** | ❌ No — locked |
> | Multiple choice options | **choice** | ❌ No — locked |
> | Number | **number** | ❌ No — locked |
>
> **This is normal.** All these types work correctly when passed to Power Automate Text inputs (auto-converted), displayed in messages via {x}, and used in conditions. For `Concatenate()` formulas, non-string types need the `Text()` wrapper.

### 5.7 Create Topic: "Schedule Client Meeting"

#### Step 1: Create the Topic

1. Open the **Scheduling Assistant** agent in Copilot Studio
2. Go to **Topics** in the left navigation
3. Click **+ Add a topic** → **From blank**
4. At the top of the canvas, click the topic name (e.g., "Untitled") and rename it to: **`Schedule Client Meeting`**

#### Step 2: Add Trigger Phrases

1. Click the **Trigger** node at the top of the canvas (it says "Phrases")
2. In the **Trigger phrases** panel on the right, add these phrases one by one (press Enter after each):
   - `Schedule a meeting`
   - `Book an appointment`
   - `Set up a call`
   - `Arrange meeting with client`
   - `Calendar`

#### Step 3: Add the Welcome Message

1. Click the **+** button below the Trigger node
2. Select **"Send a message"**
3. Type: `Let's get that meeting scheduled! I'll need a few details.`

#### Step 4: Ask for Client Name

1. Click **+** below the message node → select **"Ask a question"**
2. In the question text, type: `What is the client's full name?`
3. In the right panel, under **Identify**, click the dropdown and select **"User's entire response"**

   > ⚠️ **Why "User's entire response" instead of "Person name"?** The "Person name" entity in Copilot Studio can be unreliable — it may not recognize all name formats (e.g., Chinese names, names with hyphens). Using "User's entire response" captures exactly what the user types. For reliability, this is the better choice.

4. Under **Save response as**, click the variable dropdown → click **"Create new"** → name it `ClientName`
   - "User's entire response" entity creates a **string** type variable
   - This creates the variable **`Topic.ClientName`**

#### Step 5: Ask for Client Email

1. Click **+** → **"Ask a question"**
2. Question text: `What is the client's email address?`
3. **Identify**: Select **"Email"** from the dropdown

   > ✅ The "Email" entity works well — it validates the format and extracts a proper email address.

4. **Save response as**: Click the dropdown → **"Create new"** → name it `ClientEmail`

#### Step 6: Ask for Meeting Date/Time

1. Click **+** → **"Ask a question"**
2. Question text: `What date and time would you like to schedule the meeting? (e.g., March 25, 2026 at 2:00 PM)`
3. **Identify**: Select **"Date and time"** from the dropdown
4. **Save response as**: Click the dropdown → **"Create new"** → name it `MeetingDateTime`

   > ⚠️ **The variable type will be automatically set to `datetime`** — you cannot change this. This is by design.
   >
   > **This is fine!** Here's why:
   > - The `datetime` type still works when passed to Power Automate flow inputs that expect Text — Copilot Studio automatically converts it to an ISO 8601 string (e.g., `2026-03-25T14:00:00`)
   > - It also works when inserted into messages via the **{x}** button
   > - For Power Fx formulas that need it as text (e.g., in `Concatenate()`), wrap it with the `Text()` function: `Text(Topic.MeetingDateTime)`
   >
   > 💡 **About the "Date and time" entity:**
   > - It understands natural language like "next Tuesday at 2pm", "March 25 at 14:00", etc.
   > - Including an example in the question text helps users provide a parseable format

#### Step 7: Ask for Duration

1. Click **+** → **"Ask a question"**
2. Question text: `How long should the meeting be?`
3. **Identify**: Select **"Multiple choice options"**
4. Under **Options for user**, add three choices:
   - `30 minutes`
   - `45 minutes`
   - `1 hour`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `Duration`

   > ⚠️ **The variable type will be automatically set to `choice`** — you cannot change this to String. This is normal and works correctly.

   > ⚠️ **Make sure the choice text matches exactly** what the Power Automate flow expects. The "Calculate End Time" expression checks for `"30 minutes"`, `"45 minutes"`, and defaults to 60 for anything else (including `"1 hour"`). Keep these strings consistent.

#### Step 8: Ask for Meeting Purpose

1. Click **+** → **"Ask a question"**
2. Question text: `What is the purpose of this meeting?`
3. **Identify**: Select **"Multiple choice options"**
4. Under **Options for user**, add these choices:
   - `New policy consultation`
   - `Policy renewal discussion`
   - `Claim follow-up`
   - `Annual review`
   - `Other`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `MeetingPurpose`

   > ⚠️ **Important for multi-agent orchestration**: Do NOT enable "Receive values from other topics" on this variable (see Phase 7). Because it's a `choice` type, slot-filling from the orchestrator will cause `Text()` in `Concatenate()` formulas to return empty. The agent will always present these choice buttons — this ensures the meeting subject and body are built correctly.

#### Step 9: Show Confirmation Summary

1. Click **+** → **"Send a message"**
2. In the message box, type the following. To insert variables, click the **{x}** icon in the message toolbar and select each variable:

   ```
   Here's the meeting summary:
   👤 Client: {Topic.ClientName}
   📧 Email: {Topic.ClientEmail}
   📅 Date/Time: {Topic.MeetingDateTime}
   ⏱ Duration: {Topic.Duration}
   📋 Purpose: {Topic.MeetingPurpose}
   
   Shall I go ahead and create this calendar event?
   ```

   > 💡 **How to insert variables in messages:**
   > 1. Type the text normally
   > 2. When you reach a variable, click the **{x}** button in the message editor toolbar
   > 3. Select the variable from the dropdown (e.g., `Topic.ClientName`)
   > 4. It appears as a blue pill/tag in the message
   > 5. Continue typing after the variable

#### Step 10: Ask for Confirmation

1. Click **+** → **"Ask a question"**
2. Question text: `Confirm?`
3. **Identify**: Select **"Multiple choice options"**
4. Options:
   - `Yes, create it`
   - `No, let me change something`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `Confirmation`

#### Step 11: Add Condition Branch

1. Click **+** → **"Add a condition"** → **"Branch based on a condition"**
2. In the condition node, configure:
   - **Select a variable**: `Topic.Confirmation`
   - **Operator**: `is equal to`
   - **Value**: `Yes, create it`

   > 💡 This creates two branches: **True** (left) and **All Other Conditions** (right / Else). We'll build the "Yes" path on the left branch.

#### Step 12: Build Concatenated Values Using "Set Variable Value" Nodes

> ⚠️ **This is a key step.** Copilot Studio's "Call an action" input fields accept either a plain variable or a typed literal, but not inline concatenation with `{variables}`.
>
> **The solution**: Use **"Set variable value"** nodes with **Power Fx formulas** to build the concatenated strings BEFORE the "Call an action" node, then pass the pre-built variables to the flow.

**12a: Set the Meeting Subject**

1. Under the **True** branch (left side of the condition), click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `MeetingSubject`
3. **To value**: Click the **>** arrow or **fx** button to switch to **Formula** mode, then enter:

   ```
   Concatenate("Contoso - ", Text(Topic.MeetingPurpose), " with ", Topic.ClientName)
   ```

   > 📝 **Power Fx `Concatenate()` function** joins multiple strings together. This produces output like: `"Contoso - Policy renewal discussion with David Chan"`
   >
   > ⚠️ **Note the `Text()` wrapper on `Topic.MeetingPurpose`**: Since this variable is type `choice` (auto-set by "Multiple choice options" entity), you must wrap it with `Text()` for use in `Concatenate()`. Same rule applies to any `datetime` or `choice` variable used inside `Concatenate()`.
   >
   > ⚠️ **How to enter a formula:**
   > 1. Click the **"To value"** field
   > 2. Look for a **Formula** tab or **fx** icon — click it to switch from the variable picker to formula mode
   > 3. Type the `Concatenate(...)` expression
   > 4. Press Enter or click outside to confirm
   >
   > If you don't see a Formula option, you can alternatively:
   > - Use the **Expression** input: type `Concatenate("Contoso - ", Text(Topic.MeetingPurpose), " with ", Topic.ClientName)` directly

**12b: Set the Meeting Body**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `MeetingBody`
3. **To value** (Formula mode):

   ```text
   Concatenate("Dear ", Topic.ClientName, ", This meeting has been scheduled to discuss: ", Text(Topic.MeetingPurpose), ". Best regards, Contoso Financial Advisory Team")
   ```

   > 💡 **Want line breaks in the email body?** Use `Char(10)` for newlines in Power Fx:
   > ```text
   > Concatenate("Dear ", Topic.ClientName, ",", Char(10), Char(10), "This meeting has been scheduled to discuss: ", Text(Topic.MeetingPurpose), ".", Char(10), Char(10), "Best regards,", Char(10), "Contoso Financial Advisory Team")
   > ```

#### Step 13: Convert Non-String Variables for the Flow

> ⚠️ **Why this step is needed**: The Power Automate flow "Schedule Client Meeting" expects all inputs as **Text (String)**. However, `Topic.MeetingDateTime` is type `datetime` and `Topic.Duration` is type `choice`. Copilot Studio will show red error text like *"Input variable 'StartDateTime' is of incorrect type: DateTime"* if you map these directly.

**13a: Convert MeetingDateTime to String**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `MeetingDateTimeText`
3. **To value**: Switch to **Formula** mode (fx), then enter:
   ```text
   Text(Topic.MeetingDateTime)
   ```

**13b: Convert Duration to String**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `DurationText`
3. **To value**: Switch to **Formula** mode (fx), then enter:
   ```text
   Text(Topic.Duration)
   ```

#### Step 14: Call the "Schedule Client Meeting" Flow

1. Click **+** → **"Call an action"**
2. From the list, select **"Schedule Client Meeting"** (the Power Automate flow you created in Section 5.3)
3. The node will display the five input fields from the flow. Map each one:

   | Flow Input | Set To (click the field → select variable) | Why |
   |-----------|------|-----|
   | **Subject** | Select variable → **`Topic.MeetingSubject`** | Already a string (from Concatenate) |
   | **StartDateTime** | Select variable → **`Topic.MeetingDateTimeText`** | Converted from datetime to string in Step 13a |
   | **Duration** | Select variable → **`Topic.DurationText`** | Converted from choice to string in Step 13b |
   | **AttendeeEmail** | Select variable → **`Topic.ClientEmail`** | Already a string (from Email entity) |
   | **Body** | Select variable → **`Topic.MeetingBody`** | Already a string (from Concatenate) |

   > ⚠️ **Do NOT map `Topic.MeetingDateTime` or `Topic.Duration` directly** — they are `datetime` and `choice` types respectively, and the flow expects String. You will see red error messages if you do. Always use the converted `Topic.MeetingDateTimeText` and `Topic.DurationText` variables instead.

4. **For the flow Output**: The flow returns a `Confirmation` (string) output. **Do NOT let it map to `Topic.Confirmation`** — that variable is type `choice` (from Step 10), not string, and will cause an error.

   Instead:
   - Click on the output mapping for `Confirmation`
   - Click **"Create new"** → name it `FlowConfirmation`
   - This creates a new **string** variable `Topic.FlowConfirmation` to receive the flow's text output

#### Step 15: Show Success Message

1. Click **+** → **"Send a message"**
2. Type: `✅ Meeting created! A calendar invitation has been sent to ` then click **{x}** → select `Topic.ClientEmail` → then type `.`

   Full message: `✅ Meeting created! A calendar invitation has been sent to {Topic.ClientEmail}.`

#### Step 16: Ask About Logging the Interaction

1. Click **+** → **"Ask a question"**
2. Question text: `Would you also like me to log this interaction?`
3. **Identify**: Select **"Multiple choice options"**
4. Options:
   - `Yes`
   - `No`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `LogInteraction`

#### Step 17: Add Condition for Logging

1. Click **+** → **"Add a condition"** → **"Branch based on a condition"**
2. Configure:
   - **Select a variable**: `Topic.LogInteraction`
   - **Operator**: `is equal to`
   - **Value**: `Yes`

#### Step 18: Call the "Log Interaction to Dataverse" Flow

Under the **True** branch of the logging condition:

1. Click **+** → **"Call an action"**
2. Select **"Log Interaction to Dataverse"** (the Power Automate flow from Section 5.5)
3. Map the inputs:

   | Flow Input | Set To |
   |-----------|--------|
   | **ClientName** | Select variable → **`Topic.ClientName`** |
   | **InteractionType** | Type the literal text: `Meeting` |
   | **Summary** | *(see options below)* |
   | **FollowUpDate** | Select variable → **`Topic.MeetingDateTime`** |

   > ⚠️ **For InteractionType**: You can type the literal text `Meeting` directly — the Power Automate flow's Compose step will convert it to the numeric Dataverse Choice value.
   >
   > ⚠️ **For Summary**: You have two options:
   >
   > **Option A (Simple)**: Just pass **`Topic.MeetingPurpose`** directly — the purpose text alone is descriptive enough for a log entry.
   >
   > **Option B (Full string)**: Add another **"Set variable value"** node BEFORE this "Call an action" node:
   > 1. Create a new variable `Topic.LogSummary` (String)
   > 2. Use formula: `Concatenate(Text(Topic.MeetingPurpose), " scheduled for ", Text(Topic.MeetingDateTime))`
   > 3. Then map the Summary input to `Topic.LogSummary`

#### Step 19: Show Logging Confirmation

1. Under the "Call an action" node for logging, click **+** → **"Send a message"**
2. Type: `Interaction logged successfully! ✅`

#### Step 20: End All Branches

1. Under the **Else** branch of the logging condition (the "No" path) — click **+** → **"Topic management"** → **"End current topic"**
2. Also add **"End current topic"** at the bottom of the logging success path (after the "Interaction logged" message)
3. Under the **Else** branch of the main confirmation condition (the "No, let me change something" path) — click **+** → **"Send a message"** → Type: `No problem! Let's start over.` followed by **"Topic management"** → **"End current topic"**

> 💡 **Always close your branches** with "End current topic" or a redirect — otherwise the conversation may fall through to unexpected behavior.

#### Complete Topic Flow — Visual Reference

```
[Trigger: "Schedule a meeting" / "Book an appointment" / ...]
    │
    ▼
[Message] "Let's get that meeting scheduled! I'll need a few details."
    │
    ▼
[Ask] "What is the client's full name?"
   Identify: User's entire response → Topic.ClientName
    │
    ▼
[Ask] "What is the client's email address?"
   Identify: Email → Topic.ClientEmail
    │
    ▼
[Ask] "What date and time would you like to schedule the meeting?"
   Identify: Date and time → Topic.MeetingDateTime
    │
    ▼
[Ask] "How long should the meeting be?"
   Identify: Multiple choice (30 min / 45 min / 1 hour) → Topic.Duration
    │
    ▼
[Ask] "What is the purpose of this meeting?"
   Identify: Multiple choice (5 options) → Topic.MeetingPurpose
    │
    ▼
[Message] Summary with all variables inserted via {x}
    │
    ▼
[Ask] "Confirm?" → Multiple choice → Topic.Confirmation
    │
    ▼
[Condition] Topic.Confirmation = "Yes, create it"
    │                                    │
    ▼ TRUE                               ▼ ELSE
[Set Variable]                     [Message] "No problem!"
  Topic.MeetingSubject =           [End current topic]
  Concatenate("Contoso - ",
    Text(Topic.MeetingPurpose),
    " with ", Topic.ClientName)
    │
    ▼
[Set Variable]
  Topic.MeetingBody =
  Concatenate("Dear ",
    Topic.ClientName, ...)
    │
    ▼
[Set Variable]
  Topic.MeetingDateTimeText =
  Text(Topic.MeetingDateTime)
    │
    ▼
[Set Variable]
  Topic.DurationText =
  Text(Topic.Duration)
    │
    ▼
[Call Action: Schedule Client Meeting]
  Subject ← Topic.MeetingSubject
  StartDateTime ← Topic.MeetingDateTimeText
  Duration ← Topic.DurationText
  AttendeeEmail ← Topic.ClientEmail
  Body ← Topic.MeetingBody
    │
    ▼
[Message] "✅ Meeting created! Invitation sent to {Topic.ClientEmail}."
    │
    ▼
[Ask] "Log this interaction?" → Yes / No → Topic.LogInteraction
    │
    ▼
[Condition] Topic.LogInteraction = "Yes"
    │                        │
    ▼ TRUE                   ▼ ELSE
[Call Action:            [End current topic]
  Log Interaction
  to Dataverse]
  ClientName ← Topic.ClientName
  InteractionType ← "Meeting" (literal)
  Summary ← Topic.MeetingPurpose (or Topic.LogSummary)
  FollowUpDate ← Topic.MeetingDateTime
    │
    ▼
[Message] "Interaction logged successfully! ✅"
    │
    ▼
[End current topic]
```

#### Variables Summary

| Variable | Type | Created In | Used In |
|----------|------|-----------|---------|
| `Topic.ClientName` | String | Step 4: "Ask a question" → Save response as → Create new | Summary message, MeetingSubject formula, MeetingBody formula, Log flow |
| `Topic.ClientEmail` | String | Step 5: "Ask a question" → Save response as → Create new | Schedule flow, success message |
| `Topic.MeetingDateTime` | **datetime** | Step 6: "Ask a question" → Save response as → Create new (type auto-set by "Date and time" entity — cannot be changed) | Schedule flow, Log flow, summary message. Use `Text(Topic.MeetingDateTime)` in Power Fx `Concatenate()` formulas |
| `Topic.Duration` | **choice** | Step 7: "Ask a question" → Save response as → Create new (type auto-set by "Multiple choice options" entity — cannot be changed) | Schedule flow. Auto-converts to text when passed to Power Automate. Use `Text(Topic.Duration)` in `Concatenate()` |
| `Topic.MeetingPurpose` | **choice** | Step 8: "Ask a question" → Save response as → Create new (type auto-set) | MeetingSubject formula, MeetingBody formula, Log flow. Use `Text(Topic.MeetingPurpose)` in `Concatenate()`. Do NOT enable "Receive values from other topics" — choice variables break when slot-filled (see Phase 7). |
| `Topic.Confirmation` | **choice** | Step 10: "Ask a question" → Save response as → Create new (type auto-set) | Condition branch |
| `Topic.LogInteraction` | **choice** | Step 16: "Ask a question" → Save response as → Create new (type auto-set) | Condition branch |
| `Topic.MeetingSubject` | String | Step 12a: "Set variable value" → Set variable → Create new | Schedule flow (Subject input) |
| `Topic.MeetingBody` | String | Step 12b: "Set variable value" → Set variable → Create new | Schedule flow (Body input) |
| `Topic.MeetingDateTimeText` | String | Step 13a: "Set variable value" → Set variable → Create new. Formula: `Text(Topic.MeetingDateTime)` | Schedule flow (StartDateTime input) — converts datetime to string |
| `Topic.DurationText` | String | Step 13b: "Set variable value" → Set variable → Create new. Formula: `Text(Topic.Duration)` | Schedule flow (Duration input) — converts choice to string |
| `Topic.FlowConfirmation` | String | Step 14: "Call an action" → Output mapping → Create new | Receives the flow's text output. Prevents type mismatch with `Topic.Confirmation` (which is choice) |

#### Troubleshooting This Topic

| Issue | Solution |
|-------|----------|
| **"Call an action" input field doesn't accept `{variable}` syntax** | This is expected. You cannot type inline `{Topic.ClientName}` in action inputs. Use **"Set variable value"** with `Concatenate()` formula to build the string first, then pass the resulting variable to the action. |
| **Variable not appearing in the picker** | Variables are created via the **"Create new"** link inside node dropdowns: in "Ask a question" → **Save response as** → **Create new**, or in "Set a variable value" → **Set variable** → **Create new**. Follow the steps in order. |
| **"Date and time" entity fails to parse user input** | Add an example in the question text (e.g., "March 25, 2026 at 2:00 PM"). If it keeps failing, switch to **"User's entire response"** — this will create a String variable instead. |
| **`datetime` or `choice` variable causes error in `Concatenate()` formula** | Wrap any non-string variable with `Text()` inside `Concatenate()`. For example: `Text(Topic.MeetingDateTime)`, `Text(Topic.MeetingPurpose)`, `Text(Topic.Duration)`. |
| **Variable type locked to `datetime` or `choice` and can't be changed** | This is expected behavior. Each Identify entity forces a specific variable type. They still work correctly with Power Automate Text inputs (auto-converted), in messages via {x}, and in conditions. Only `Concatenate()` formulas require the `Text()` wrapper. |
| **Formula / Power Fx not available** | Go to **Settings** → **General** → ensure **Power Fx formulas** is turned on. If not available, use a workaround: pass individual variables to the Power Automate flow and do the string concatenation inside the flow using a Compose step. |
| **"Call an action" node doesn't show the flow** | Refresh the Copilot Studio page. Ensure the Power Automate flow was saved and uses the correct trigger ("When Copilot Studio calls a flow"). |
| **"Person name" entity misidentifies names** | Switch to **"User's entire response"** for more reliable name capture. |
| **Condition branch not working** | Verify the condition uses **"is equal to"** and the value matches the **exact** choice text (case-sensitive). |
| **Meeting subject/body missing purpose when called from Hub agent** | `Topic.MeetingPurpose` is a `choice` variable — when the orchestrator slot-fills it with free text, `Text()` returns empty. **Fix**: Do NOT enable "Receive values from other topics" on `Topic.MeetingPurpose` (see Phase 7). |

> 💡 **Power Fx Workaround**: If Power Fx formulas are not available in your environment, modify the **Power Automate flow** (Schedule Client Meeting) to accept `ClientName` and `MeetingPurpose` as separate inputs, and add Compose steps inside the flow to build the Subject and Body strings. Then in the topic, pass `Topic.ClientName` and `Topic.MeetingPurpose` directly to the flow instead of the concatenated variables.

#### Testing the Schedule Client Meeting Topic

**Pre-test checklist:**

- [ ] All nodes are connected (no orphan nodes floating on the canvas)
- [ ] No red error messages on any node (especially the "Call an action" node)
- [ ] The Power Automate flow "Schedule Client Meeting" is saved and published
- [ ] The Outlook connector in the flow is authenticated (test the flow separately in Power Automate first if unsure)
- [ ] Click **Save** in the top right of Copilot Studio before testing

**How to test:**

1. Click **Save** at the top right of Copilot Studio
2. Open the **Test your agent** panel on the right side (click the **Test** button in the top right corner if it's not visible)
3. If there's a previous conversation, click the **refresh/reset** icon (circular arrow) at the top of the test panel to start a fresh session

**Test 1 — Full happy path (with Outlook):**

Type: `Schedule a meeting`

| Step | Agent Asks | You Respond |
|------|-----------|-------------|
| Welcome | "Let's get that meeting scheduled!" | (automatic) |
| Client name | "What is the client's full name?" | Type: `David Chan` |
| Email | "What is the client's email address?" | Type: `david.chan@email.com` |
| Date/time | "What date and time..." | Type: `March 25, 2026 at 2:00 PM` |
| Duration | "How long should the meeting be?" | Select: `30 minutes` |
| Purpose | "What is the purpose..." | Select: `Policy renewal discussion` |
| Summary | Shows meeting summary with all details | Review the values |
| Confirm | "Confirm?" | Select: `Yes, create it` |

**Expected result after confirmation:**
- The agent should show: `✅ Meeting created! A calendar invitation has been sent to david.chan@email.com.`
- A calendar event should appear in your Outlook calendar
- The agent then asks: `Would you also like me to log this interaction?`

> ⚠️ **If the flow fails at runtime**: The agent may show an error or no response after "Yes, create it". Open **Power Automate** → go to **My flows** → click **Schedule Client Meeting** → check **Run history** for detailed error messages.

**Test 2 — Cancel path:**

Type: `Book an appointment`

Go through the questions with any values, then at the confirmation step:
- Select: `No, let me change something`

**Expected result**: The agent should show "No problem!" and end the topic (or restart).

**Test 3 — Test with logging:**

Repeat Test 1, but after the meeting is created:
- When asked "Would you also like me to log this interaction?" → Select: `Yes`

**Expected result**: The agent should show `Interaction logged successfully! ✅`

**Test 4 — Verify the Outlook calendar event (live environment):**

1. Open **Outlook** (web or desktop)
2. Go to your **Calendar**
3. Navigate to the date you specified (e.g., March 25, 2026)
4. You should see the calendar event with:
   - **Subject**: "Contoso - Policy renewal discussion with David Chan"
   - **Time**: 2:00 PM - 2:30 PM (HKT)
   - **Attendee**: david.chan@email.com
   - **Body**: The meeting body text from your Concatenate formula

**Test 5 — Verify the Dataverse log (if you tested logging):**

1. Open **Power Apps** → **Tables** → **InteractionLog**
2. Click **Data** tab
3. You should see a new row with:
   - **ClientName**: David Chan
   - **InteractionType**: Meeting
   - **FollowUpRequired**: Yes
   - **FollowUpDate**: The date you specified

**What to check if something goes wrong:**

| Symptom | What to Check |
|---------|--------------|
| Agent doesn't respond after "Yes, create it" | The Power Automate flow failed. Check flow run history in Power Automate. |
| Flow run shows "Forbidden" or "Unauthorized" | Re-authenticate the Outlook connection in the flow. Click the Outlook step → connection → sign in again. |
| Calendar event has wrong time | The datetime format may not include timezone. The flow's Time zone setting (UTC+8) should handle this. |
| Variables show as blank in the summary message | A previous node may have failed silently. Add a test message before the summary to print individual variables. |
| "Topic checker" shows warnings | Click the **Topic checker** icon in the top toolbar to see all warnings. Yellow warnings are usually non-blocking. Red errors must be fixed. |
| Agent triggers the wrong topic | Your trigger phrases may overlap with system topics. Make them more specific, or try typing the exact trigger phrase. |

> 💡 **Tip**: Use the **Topic checker** button (checkmark icon in the top toolbar, next to Variables) to scan for errors before testing.

### 5.8 Create Topic: "Send Follow-Up Email"

#### Step 1: Create the Topic

1. Open the **Scheduling Assistant** agent in Copilot Studio
2. Go to **Topics** → click **+ Add a topic** → **From blank**
3. Rename the topic to: **`Send Follow-Up Email`**

#### Step 2: Add Trigger Phrases

1. Click the **Trigger** node
2. Add these trigger phrases:
   - `Send a follow-up`
   - `Email the client`
   - `Send an email`
   - `Follow up with client`

#### Step 3: Ask for Client Email

1. Click **+** → **"Ask a question"**
2. Question text: `What is the client's email address?`
3. **Identify**: Select **"Email"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `ClientEmail`

#### Step 4: Ask for Client Name

1. Click **+** → **"Ask a question"**
2. Question text: `What is the client's name?`
3. **Identify**: Select **"User's entire response"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `ClientName`

#### Step 5: Ask for Email Content

1. Click **+** → **"Ask a question"**
2. Question text: `What would you like the email to say? (I'll format it professionally)`
3. **Identify**: Select **"User's entire response"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `EmailContent`

#### Step 6: Build the Email Subject (Set Variable)

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `EmailSubject`
3. **To value**: Switch to **Formula** mode (fx), then enter:

   ```text
   Concatenate("Follow-up from Contoso - ", Topic.ClientName)
   ```

#### Step 7: Generate the Email Body (AI Builder Custom Prompt)

> Instead of `Concatenate()` (which is too rigid), we use the **Prompt node** (AI Builder custom prompt). This gives AI-generated content with controllable output — the result is stored in a variable you can pass to the flow.

1. Click **+** → **"Call an action"** → scroll down to **"Create a prompt"**

   > This opens the **Custom prompt editor** dialog.

2. In the **Instructions** field, enter:

   ```
   Compose the body of a professional follow-up email from a Contoso financial advisor to their client. Client name: {ClientName}. The advisor's key points to include: {EmailContent}.
   ```

   > ⚠️ **Important**: The `{ClientName}` and `{EmailContent}` here are **prompt input placeholders** — NOT Copilot Studio Topic variables. You will create these inputs in the next step.

3. **Add Input variables** — click **"+ Add content"** at the bottom of the Instructions box, then click the **Input** section:
   - Click **Text** to add the first input → name it `ClientName`
   - Click **Text** again to add the second input → name it `EmailContent`

4. **Configure the Output**:
   - In the **Outputs** section at the bottom, you'll see `predictionOutput` (record type) — this is the default output variable auto-created by the Custom Prompt node
   - **Keep it as `record` type** — do NOT try to change it to string
   - The `predictionOutput` record contains a `.text` property that holds the actual generated text. You will access this as **`Topic.predictionOutput.text`** in subsequent nodes

   > 💡 **How the Prompt node outputs work:**
   > - `predictionOutput` = a **record** containing the AI-generated result
   > - To access the generated email text, use **`Topic.predictionOutput.text`**
   > - You do **NOT** need to create a separate `FormattedEmail` variable — just use the default `predictionOutput` variable
   >
   > ⚠️ **Common mistake**: Trying to map the output to a new string variable causes type errors. Just use `Topic.predictionOutput.text`.

5. Click **Save** to save the custom prompt

6. **Back on the topic canvas**, the Prompt node will appear. Map the inputs:

   | Prompt Input | Set To (select variable) |
   |-------------|--------------------------|
   | **ClientName** | Select → **`Topic.ClientName`** |
   | **EmailContent** | Select → **`Topic.EmailContent`** |

#### Step 8: Show the Draft Email for Review

1. Click **+** → **"Send a message"**
2. Type the following (use **{x}** to insert the variable):

   `Here's the draft email:` then press Enter, click **{x}** → select `Topic.predictionOutput.text`, then on a new line type: `Confirm send?`

#### Step 9: Ask for Send Confirmation

1. Click **+** → **"Ask a question"**
2. Question text: `Confirm send?`
3. **Identify**: Select **"Multiple choice options"**
4. Options:
   - `Yes, send it`
   - `Edit and resend`
   - `Cancel`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `EmailConfirm`

#### Step 10: Add Condition Branch

1. Click **+** → **"Add a condition"** → **"Branch based on a condition"**
2. Configure:
   - **Select a variable**: `Topic.EmailConfirm`
   - **Operator**: `is equal to`
   - **Value**: `Yes, send it`

#### Step 11: Call the "Send Follow-Up Email" Flow

Under the **True** branch:

1. Click **+** → **"Call an action"**
2. Select **"Send Follow-Up Email"** (the Power Automate flow from Section 5.4)
3. Map the inputs:

   | Flow Input | Set To | Why |
   |-----------|--------|-----|
   | **ToEmail** | Select variable → **`Topic.ClientEmail`** | Already a string (Email entity) |
   | **Subject** | Select variable → **`Topic.EmailSubject`** | Built with Concatenate in Step 6 |
   | **EmailBody** | Select variable → **`Topic.predictionOutput.text`** | AI-generated body from Step 7 |

4. **For the flow Output**: The flow returns `Confirmation` (string). Map it to a new variable:
   - Click **"Create new"** → name it `EmailFlowConfirmation`
   - This prevents type conflicts with other choice variables

#### Step 12: Show Success Message

1. Click **+** → **"Send a message"**
2. Type: `✅ Email sent to ` then click **{x}** → select `Topic.ClientEmail` → then type `!`

   Full message: `✅ Email sent to {Topic.ClientEmail}!`

#### Step 13: Handle the "Cancel" and "Edit" Branches

1. Under the **Else** branch of the condition:

   **Option A (Simple — recommended):**
   - Click **+** → **"Send a message"** → Type: `No problem. The email was not sent.`
   - Click **+** → **"Topic management"** → **"End current topic"**

   **Option B (Advanced — handle Edit separately):**
   - Add another condition: `Topic.EmailConfirm` is equal to `Edit and resend`
   - True branch → **"Topic management"** → **"Redirect to another topic"** → select this same topic (restarts the flow)
   - Else branch (Cancel) → Message: `Email cancelled.` → **"End current topic"**

2. Also add **"End current topic"** after the success message in Step 12.

#### Complete Topic Flow — Visual Reference

```
[Trigger: "Send a follow-up" / "Email the client" / ...]
    │
    ▼
[Ask] "What is the client's email address?"
   Identify: Email → Topic.ClientEmail (string)
    │
    ▼
[Ask] "What is the client's name?"
   Identify: User's entire response → Topic.ClientName (string)
    │
    ▼
[Ask] "What would you like the email to say?"
   Identify: User's entire response → Topic.EmailContent (string)
    │
    ▼
[Set Variable]
  Topic.EmailSubject =
  Concatenate("Follow-up from
    Contoso - ", Topic.ClientName)
    │
    ▼
[Prompt (AI Builder Custom Prompt)]
   Instructions: "Compose the body of a
          professional follow-up email.
          Client: {ClientName}.
          Key points: {EmailContent}."
   Inputs:
     ClientName ← Topic.ClientName
     EmailContent ← Topic.EmailContent
   Output: predictionOutput (record)
    │
    ▼
[Message] "Here's the draft email:
   {Topic.predictionOutput.text}
   Confirm send?"
    │
    ▼
[Ask] "Confirm send?" → Yes, send it / Edit and resend / Cancel
   → Topic.EmailConfirm (choice)
    │
    ▼
[Condition] Topic.EmailConfirm = "Yes, send it"
    │                              │
    ▼ TRUE                         ▼ ELSE
[Call Action:                [Message] "No problem.
  Send Follow-Up Email]        The email was not sent."
  ToEmail ← ClientEmail      [End current topic]
  Subject ← EmailSubject
  EmailBody ← predictionOutput.text
    │
    ▼
[Message] "✅ Email sent to {Topic.ClientEmail}!"
    │
    ▼
[End current topic]
```

#### Variables Summary

| Variable | Type | Created In | Used In |
|----------|------|-----------|---------|
| `Topic.ClientEmail` | string | Step 3: "Ask a question" → Email entity | Flow input (ToEmail), success message |
| `Topic.ClientName` | string | Step 4: "Ask a question" → User's entire response | Generative prompt, EmailSubject formula |
| `Topic.EmailContent` | string | Step 5: "Ask a question" → User's entire response | Generative prompt (key points) |
| `Topic.EmailSubject` | string | Step 6: "Set variable value" → Concatenate formula | Flow input (Subject) |
| `Topic.predictionOutput` | **record** | Step 7: Prompt node (AI Builder) → Auto-created default output | Use `Topic.predictionOutput.text` for email body |
| `Topic.EmailConfirm` | **choice** | Step 9: "Ask a question" → Multiple choice options (locked) | Condition branch |
| `Topic.EmailFlowConfirmation` | string | Step 11: "Call an action" → Output mapping → Create new | Receives flow output (prevents type conflict) |

#### Troubleshooting This Topic

| Issue | Solution |
|-------|----------|
| **Draft email is empty or generic** | The `Topic.EmailContent` variable may be empty. Add a test message before Step 6 to print `Topic.EmailContent` via {x} and verify it has a value. |
| **"Call an action" shows type error on Subject** | Use the pre-built `Topic.EmailSubject` string variable from the Concatenate formula (Step 6). |
| **"Call an action" shows type error on EmailBody** | Use `Topic.predictionOutput.text` from Step 7. |
| **Prompt node output is record type, not string** | This is correct and expected. Use `Topic.predictionOutput.text` to access the generated text string. |
| **Email not received by recipient** | Check Power Automate → "Send Follow-Up Email" flow → Run history. Common issues: Outlook connection expired, sender doesn't have send permissions, or email address format is invalid. |
| **`predictionOutput.text` is empty** | Check the Prompt node: ensure the Instructions field contains the prompt text with `{ClientName}` and `{EmailContent}` placeholders, the Text inputs are added and named correctly, and the output is the default `predictionOutput` (record type). |

#### Testing the Send Follow-Up Email Topic

**Pre-test checklist:**

- [ ] All nodes are connected (no orphan nodes)
- [ ] No red error messages on any node
- [ ] The Power Automate flow "Send Follow-Up Email" is saved and published
- [ ] The Outlook connector in the flow is authenticated
- [ ] Click **Save** before testing

**Test 1 — Full happy path:**

Type: `Send a follow-up email`

| Step | Agent Asks | You Respond |
|------|-----------|-------------|
| Email | "What is the client's email address?" | Type: `david.chan@email.com` |
| Name | "What is the client's name?" | Type: `David Chan` |
| Content | "What would you like the email to say?" | Type: `Thank him for our meeting today. Summarize that we discussed his life insurance renewal and I will prepare a renewal quote by next week.` |
| Draft | Shows AI-generated professional email | Review the draft |
| Confirm | "Confirm send?" | Select: `Yes, send it` |

**Expected result:**
- The AI generates a professional email (warm tone, includes the key points, Contoso sign-off)
- After confirmation: `✅ Email sent to david.chan@email.com!`

**Test 2 — Cancel path:**

Type: `Email the client`

Go through the questions, review the draft, then:
- Select: `Cancel`

**Expected result**: "No problem. The email was not sent."

**Test 3 — Verify the email in Outlook (live environment):**

1. Open **Outlook** (web or desktop)
2. Check **Sent Items** — you should see the email with:
   - **To**: david.chan@email.com
   - **Subject**: "Follow-up from Contoso - David Chan"
   - **Body**: The AI-generated professional email text

**Test 4 — Test with different content styles:**

| Test | EmailContent Input | What to Verify |
|------|-------------------|----------------|
| **Bullet points** | `- Discussed critical illness coverage - Client wants Gold tier - Follow up in 2 weeks` | AI should expand bullets into a flowing email |
| **Short note** | `Thanks for the call, will send proposal Monday` | AI should add professional padding and sign-off |
| **Detailed** | `We met today to review Emily's health insurance. She wants to add spouse coverage and increase her plan to Premier tier. I'll prepare the paperwork and send it by Friday.` | AI should structure it cleanly without losing details |

**Save** the topic.

---

## Phase 6: Connect Sub-Agents to the Hub

### 6.1 Register Sub-Agents

1. Open **"Contoso Advisor Hub"** (primary agent)
2. Go to **Settings** → **Generative AI** → ensure **Orchestration** is enabled
3. Go to **Agents** (in the left nav) → **+ Add agent**
4. Add each sub-agent:

| Agent | Description for Orchestrator |
|-------|------------------------------|
| Policy Lookup Agent | Use this agent when the advisor wants to find client policy details, search by client name or policy number, or check for expiring policies. |
| Product Advisor Agent | Use this agent when the advisor asks about product details, comparisons, recommendations, or underwriting guidelines. |
| Scheduling Assistant | Use this agent when the advisor wants to schedule a meeting, send an email, or log an interaction with a client. |

### 6.2 Enable Topic Input Variables (Critical — Prevents Re-Asking)

> ⚠️ **Without this step, sub-agents will re-ask every question** even when the user already provided the information in their original message to the Hub agent. This is because the sub-agent's topic variables are not configured to receive values from the orchestrator.
>
> **The fix**: Mark each topic's input variables as **"Receive values from other topics"** (i.e., make them Topic Inputs). This allows the generative orchestrator to extract entities from the user's original message and **slot-fill** them automatically — skipping "Ask a question" nodes whose answers are already known.

**For each sub-agent topic, do the following:**

#### Policy Lookup Agent — "Lookup by Client Name" Topic

1. Open the **Policy Lookup Agent** → **Topics** → open **"Lookup by Client Name"**
2. Click the **Variables** button in the top toolbar to open the Variables panel
3. For the variable **`Topic.ClientName`**:
   - Click on it to open its properties
   - Enable **"Receive values from other topics"** (toggle it ON)
   - This marks it as a **Topic Input** — the orchestrator can now fill it when transferring
4. For the variable **`Topic.SearchMethod`**:
   - Enable **"Receive values from other topics"** as well
   - The orchestrator may infer the search method from context
5. Click **Save**

> 💡 **How slot-filling works**: When the user types "I need to check the policy for David Chan" to the Hub agent, the orchestrator:
> 1. Identifies the intent → routes to Policy Lookup Agent
> 2. Sees that the "Lookup by Client Name" topic has `Topic.ClientName` as a Topic Input
> 3. Extracts "David Chan" from the original message → fills `Topic.ClientName` = "David Chan"
> 4. Since `Topic.ClientName` is already filled, the "Ask a question" node for client name is **skipped**
> 5. The topic proceeds directly to the flow call

#### Product Advisor Agent — "Product Recommendation" Topic

1. Open the **Product Advisor Agent** → **Topics** → open **"Product Recommendation"**
2. Click **Variables** in the top toolbar
3. Enable **"Receive values from other topics"** for these variables:
   - **`Topic.ClientAge`** — so the orchestrator can extract age (e.g., "age 42"). This is a **number** type — slot-filling works correctly.
   - **`Topic.PrimaryConcern`** — ⚠️ This is a **choice** variable. The orchestrator may or may not match free-text to a valid choice. If the match fails, the agent will still ask the user to select. Since this variable is only used in the generative answers prompt (not in `Concatenate(Text(...))`), slot-filling issues won't cause empty values in output.
   - **`Topic.PreExisting`** — choice variable, same caveat as above. Used in the generative prompt, not in Concatenate.
   - **`Topic.Budget`** — choice variable, same caveat as above. Used in the generative prompt, not in Concatenate.
4. Click **Save**

> 💡 These choice variables are **safe to slot-fill** in the Product Advisor because they are only used inside the Generative Answers prompt (which receives the raw value as text). They are NOT used in `Concatenate(Text(...))` formulas.

#### Scheduling Assistant — "Schedule Client Meeting" and "Send Follow-Up Email" Topics

1. Open the **Scheduling Assistant** → **Topics** → open **"Schedule Client Meeting"**
2. Enable **"Receive values from other topics"** for:
   - **`Topic.ClientName`** — string type, safe to slot-fill
   - **`Topic.ClientEmail`** — string type, safe to slot-fill
   - ⚠️ **Do NOT enable it for `Topic.MeetingPurpose`** — it is a `choice` variable used in `Concatenate(Text(...))` formulas. Slot-filling it with free text causes `Text()` to return empty, breaking the meeting subject and body. The agent will present choice buttons for purpose selection instead.
3. Click **Save**

> 💡 **Why not slot-fill `Topic.MeetingPurpose`?** It's a **choice** variable (from "Multiple choice options"). When the orchestrator fills it with free text like "discuss his policy renewal", `Text(Topic.MeetingPurpose)` returns empty in `Concatenate()` formulas — because the free text doesn't match a predefined choice label. The meeting subject and body would be missing the purpose.
>
> **The trade-off**: The user will be asked to select the meeting purpose from buttons even when they mentioned it in their message to the Hub. This adds one extra interaction but ensures the subject line and body are correct. All other variables (ClientName, ClientEmail) are still slot-filled.

4. Open **"Send Follow-Up Email"** topic
5. Enable **"Receive values from other topics"** for:
   - **`Topic.ClientEmail`**
   - **`Topic.ClientName`**
   - **`Topic.EmailContent`** (optional — may be inferred from the user's request)
6. Click **Save**

> ⚠️ **Important notes about slot-filling behavior:**
>
> | Behavior | Details |
> |----------|---------|
> | **Only Topic Input variables are slot-filled** | Variables without "Receive values from other topics" enabled will ALWAYS trigger their "Ask a question" nodes |
> | **⚠️ Choice variables + `Text()` = DANGER with slot-filling** | If a **choice** variable is slot-filled by the orchestrator with free text, `Text(variable)` in `Concatenate()` returns **empty string**. **Do NOT enable slot-filling on choice variables used in `Concatenate(Text(...))`**. |
> | **String variables are always safe to slot-fill** | Variables from "User's entire response" or "Email" entities are type `string` — slot-filling works perfectly |
> | **Variables the orchestrator can't extract are still asked** | If the user says "Schedule a meeting with David Chan" but doesn't mention a date, `Topic.MeetingDateTime` will still be asked |
> | **Variables filled by the orchestrator skip their "Ask a question" node** | The topic flow jumps past any question node whose variable is already populated |
> | **Global variables are NOT slot-filled** | Only Topic-scoped variables with "Receive values from other topics" enabled participate in slot-filling |

### 6.3 How Orchestration Works

When an advisor types a message into the Hub agent:
1. The orchestrator analyzes the intent
2. It matches the intent to the best sub-agent based on the descriptions
3. It extracts relevant entities from the user's message (names, numbers, dates, etc.)
4. It **transfers** the conversation to that sub-agent, **pre-filling Topic Input variables** with extracted values
5. The sub-agent skips questions for pre-filled variables and only asks for missing information
6. The sub-agent handles the request and can transfer back when done

**Example flow (with slot-filling enabled):**
```
Advisor: "I need to check the policy for David Chan"

Hub Agent → Recognizes intent: policy lookup
  → Extracts: ClientName = "David Chan"
  → Transfers to Policy Lookup Agent
  → Pre-fills Topic.ClientName = "David Chan"

Policy Lookup Agent:
  → Topic.ClientName is already filled → SKIPS "What is the client's name?" question
  → Proceeds directly to SharePoint query
  → Returns policy details
```

**Without slot-filling (what happens if you skip this section):**
```
Advisor: "I need to check the policy for David Chan"

Hub Agent → Transfers to Policy Lookup Agent (no pre-filled variables)

Policy Lookup Agent:
  → "How would you like to search?"      ← RE-ASKS (annoying!)
  → "What is the client's name?"          ← RE-ASKS (user already said it!)
  → Then queries SharePoint
```

> 💡 **This is the single most important step for a smooth multi-agent experience.** Without Topic Input variables, the multi-agent experience feels broken — users have to repeat themselves after every transfer. With it, the conversation flows naturally.

---

## Phase 7: Test

### 7.1 Test Each Sub-Agent Individually

Open each sub-agent's **Test your agent** panel and verify:

| Agent | Test Input | Expected |
|-------|-----------|----------|
| Policy Lookup | "Find client David Chan" | Returns policy CTO-2025-00123 details from SharePoint |
| Policy Lookup | "Search by policy number CTO-2025-00456" | Returns Emily Lau's health policy |
| Product Advisor | "Recommend a product" → age 42, critical illness, no pre-existing, HKD 1500-3000 | AI recommendation from knowledge base |
| Scheduling Assistant | "Schedule a meeting" → walk through all fields → confirm | Calendar event appears in Outlook |
| Scheduling Assistant | "Send a follow-up email" → provide details → confirm | Email appears in Outlook Sent Items |

### 7.2 Test End-to-End via Hub Agent

Open **Contoso Advisor Hub** → **Test your agent**:

1. Type: `I need to check the policy for David Chan`
   - Should transfer to Policy Lookup Agent
   - Should skip asking for client name (slot-filled)
   - Should return policy details from SharePoint

2. Type: `I have a new client, age 42, mainly concerned about critical illness protection, no pre-existing conditions, budget around HKD 1500/month`
   - Should transfer to Product Advisor Agent
   - Should extract variables from message
   - Should generate AI recommendation

3. Type: `Let's schedule a meeting with David Chan to discuss his policy renewal`
   - Should transfer to Scheduling Assistant
   - Should slot-fill ClientName
   - Should ask for remaining details (email, date, duration, purpose via buttons)
   - Confirm → calendar event created

4. Type: `Send a follow-up email to david.chan@email.com`
   - Should transfer to Scheduling Assistant
   - Should generate AI email draft
   - Confirm → email sent

### 7.3 Verify in External Systems

- [ ] **Outlook Calendar**: Event with correct subject, time, attendee
- [ ] **Outlook Sent Items**: Follow-up email with AI-generated body
- [ ] **Power Apps → InteractionLog table**: New row with correct data

---

## Phase 8: Publish

1. Open **Contoso Advisor Hub** → click **Publish** (top right)
2. Open each sub-agent → click **Publish**
3. All 4 agents must be published for orchestration to work

---

## Quick Reference: All Flows & Topics

| Agent | Topics | Power Automate Flows |
|-------|--------|---------------------|
| Contoso Advisor Hub | *(uses generative orchestration — no custom topics needed)* | — |
| Policy Lookup Agent | Lookup by Client Name | Search Client Policies |
| Product Advisor Agent | Product Recommendation | — |
| Scheduling Assistant | Schedule Client Meeting | Schedule Client Meeting, Log Interaction to Dataverse |
| Scheduling Assistant | Send Follow-Up Email | Send Follow-Up Email |

---

## Quick Reference: Key Gotchas

| Issue | Solution |
|-------|----------|
| SharePoint column "ClientName" has internal name `Title` | Use `Title` in filter queries, not `ClientName` |
| SharePoint Get Items returns array | Use `first()` in a Compose step before returning values |
| Choice columns return objects | Use `?['Value']` to get text; use `string()` for Currency |
| `choice`/`datetime` variables in `Concatenate()` | Wrap with `Text()`: e.g., `Text(Topic.MeetingPurpose)` |
| Flow output `Confirmation` conflicts with topic variable | Create a separate `FlowConfirmation` variable (string) |
| `predictionOutput` is record type | Access text via `Topic.predictionOutput.text` |
| MeetingPurpose slot-filling breaks subject/body | Do NOT enable "Receive values from other topics" on it |
| Connector auth expires before testing | Test all flows in Power Automate → re-authenticate before use |
| Sub-agent re-asks questions the user already answered | Enable **"Receive values from other topics"** on the sub-agent's topic variables (see Phase 6) |
| Meeting subject/body missing purpose after orchestrator transfer | `Topic.MeetingPurpose` is a `choice` variable — `Text()` returns empty when slot-filled. Do NOT enable slot-filling on it |
| Generative answers are inaccurate | Improve the uploaded documents — add more context and structure |

---

## Appendix A: Preparation Checklist

- [ ] Create SharePoint site with `ClientPolicies` list and sample data
- [ ] Upload product documents to SharePoint document library (`ProductGuides`)
- [ ] Create Dataverse `InteractionLog` table
- [ ] Build Agent 1: Contoso Advisor Hub (orchestrator)
- [ ] Build Agent 2: Policy Lookup Agent + SharePoint connector flow
- [ ] Build Agent 3: Product Advisor Agent + Knowledge sources
- [ ] Build Agent 4: Scheduling Assistant + Outlook/Dataverse connector flows
- [ ] Register sub-agents in the Hub agent
- [ ] Enable "Receive values from other topics" on key topic variables for slot-filling (Phase 6)
- [ ] Test each agent individually
- [ ] Test end-to-end multi-agent flow
- [ ] Pre-authenticate all connectors in Power Automate flows (SharePoint, Outlook, Dataverse)
- [ ] Publish all 4 agents

---

## Appendix B: Sample Support Documents

### Underwriting_Guidelines.docx

Copy the content below into a Word document and upload to the `ProductGuides` SharePoint library:

```
CONTOSO UNDERWRITING GUIDELINES — ADVISOR REFERENCE

GENERAL PRINCIPLES:
- All applications are subject to medical underwriting for coverage above HKD 2,000,000.
- Applicants aged 50+ require a basic health screening regardless of coverage amount.
- Smokers are rated separately and may have higher premiums.

LIFE INSURANCE:
- Age 18-45: Standard underwriting, no medical exam for coverage up to HKD 3,000,000.
- Age 46-55: Medical exam required for coverage above HKD 1,000,000.
- Age 56-65: Medical exam required for all applications. Maximum coverage HKD 5,000,000.

HEALTH INSURANCE:
- Pre-existing conditions: 12-month waiting period applies. Must be declared at application.
- Mental health coverage: Available on Premier plan only. 6-month waiting period.
- Maternity: Available as add-on rider. 10-month waiting period.

CRITICAL ILLNESS:
- Family history of cancer/heart disease: May require additional screening.
- Prior critical illness diagnosis: Generally not eligible for new critical illness coverage.
- Early-stage benefit: Available on Gold and Platinum tiers only.

RED FLAGS (REFER TO SENIOR UNDERWRITER):
- Applicant has been declined by another insurer
- Hazardous occupation (mining, offshore, aviation)
- Travel to high-risk regions in the past 12 months
- BMI above 35 or below 16
```

### Compliance_Policy.docx

Copy the content below into a Word document and upload to the `ProductGuides` SharePoint library:

```
CONTOSO COMPLIANCE GUIDELINES FOR FINANCIAL ADVISORS

DATA PROTECTION:
- Client personal data must not be shared outside of Contoso systems.
- All client conversations must be logged within 24 hours.
- Client consent is required before sharing data with third parties.

SALES PRACTICES:
- Advisors must complete a needs analysis before recommending any product.
- All product recommendations must be documented with reasoning.
- "Best interest" duty: recommend what suits the client, not what pays highest commission.
- Cool-off period: clients have 21 days to cancel a new policy for full refund.

COMMUNICATION:
- All client-facing materials must use approved Contoso templates.
- Social media posts about products must be pre-approved by compliance.
- Claims of guaranteed returns are strictly prohibited.
- WhatsApp/WeChat may be used for scheduling only, not for policy advice.

ANTI-MONEY LAUNDERING:
- Verify client identity for all new policies.
- Flag any single premium payment above HKD 500,000 for review.
- Report suspicious transaction patterns to compliance@contoso.com.
```

---

## Appendix C: Comprehensive Troubleshooting

| Issue | Solution |
|-------|----------|
| Connector authentication fails | Open the Power Automate flow → click the connector step → re-authenticate your connection |
| Sub-agent not being triggered | Check the description text in the agent registration — make it more specific |
| Sub-agent re-asks questions the user already answered | Enable **"Receive values from other topics"** on the sub-agent's topic variables (see Phase 6). Only Topic Input variables are slot-filled by the orchestrator. |
| Meeting subject/body missing purpose after orchestrator transfer | `Topic.MeetingPurpose` is a `choice` variable — `Text()` returns empty when slot-filled with free text. **Fix**: Do NOT enable "Receive values from other topics" on `Topic.MeetingPurpose` (Phase 6). |
| SharePoint Get Items returns nothing | Verify OData filter syntax in the flow and ensure column names match exactly (case-sensitive) |
| Power Automate flow not appearing in topic | Ensure the flow is saved and uses the "When Power Virtual Agents calls a flow" trigger |
| Variables not passing between agents | Use global variables or reinitialize in the sub-agent topic |
| Generative answers are inaccurate | Improve the uploaded documents — add more context and structure |
| Outlook event not appearing | Check the attendee email format and ensure the Outlook connection in the flow has calendar permissions |
| Flow errors at runtime | Open Power Automate → check the flow run history for detailed error messages |
| InteractionType shows dropdown instead of accepting dynamic value | Click **"Enter custom value"** at the bottom of the dropdown to switch to free-text mode |
| Error "Invalid value for Choice column" | Verify the exact numeric values in your Dataverse choices match your Compose expression |
| Dataverse "Add a new row" fails with 403 | Re-authenticate the Dataverse connection. Ensure your account has write access |
| "Call an action" input field doesn't accept `{variable}` syntax | Use **"Set variable value"** with `Concatenate()` formula to build the string first |
| Formula / Power Fx not available | Go to **Settings** → **General** → ensure **Power Fx formulas** is turned on |
| Variable type locked to `datetime` or `choice` | Expected behavior — use `Text()` wrapper in `Concatenate()` formulas |
| `predictionOutput.text` is empty | Verify the Prompt node has inputs mapped and the Instructions contain the correct placeholders |
| Agent doesn't respond after "Yes, create it" | The Power Automate flow failed. Check flow run history in Power Automate |
| "Topic checker" shows warnings | Click the **Topic checker** icon in the top toolbar. Yellow warnings are usually non-blocking. Red errors must be fixed. |
