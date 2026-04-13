> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Copilot Studio — Topics & Workflows Deep Dive

> A detailed guide on Topics and Workflows in Copilot Studio, with practical example scenarios for legal and compliance teams.
> **Audience**: Legal team members, compliance officers, business stakeholders, and hackathon participants

---

## Table of Contents

1. [What Are Topics?](#1-what-are-topics)
2. [Topic Anatomy — Triggers, Nodes, and Branches](#2-topic-anatomy--triggers-nodes-and-branches)
3. [What Are Workflows?](#3-what-are-workflows)
4. [Topics vs Workflows — When to Use Which](#4-topics-vs-workflows--when-to-use-which)
5. [Example Scenario 1: Contract Approval Routing Workflow](#5-example-scenario-1-contract-approval-routing-workflow)
6. [Example Scenario 2: Legal Compliance Check Topic](#6-example-scenario-2-legal-compliance-check-topic)
7. [Example Scenario 3: NDA Request & Generation Workflow](#7-example-scenario-3-nda-request--generation-workflow)
8. [Example Scenario 4: Regulatory Inquiry Triage Topic](#8-example-scenario-4-regulatory-inquiry-triage-topic)
9. [Configuring Workflows — Step-by-Step Walkthrough](#9-configuring-workflows--step-by-step-walkthrough)
10. [Governance & Audit Considerations for Legal Teams](#10-governance--audit-considerations-for-legal-teams)
11. [Key Gotchas & Best Practices](#11-key-gotchas--best-practices)

---

## 1. What Are Topics?

**Topics** are the building blocks of a Copilot Studio agent's conversational ability. Think of a Topic as a **structured conversation script** — it defines how the agent responds to a specific type of user request.

### Core Concept

| Aspect | Description |
|--------|-------------|
| **Definition** | A Topic is a discrete conversation path that handles one user intent (e.g., "check contract status", "request NDA") |
| **Trigger** | Each Topic has **trigger phrases** — the words or sentences that activate it (e.g., "I need a contract reviewed") |
| **Flow** | Inside a Topic, you build a flow of **nodes**: messages, questions, conditions, actions, and branches |
| **Scope** | A Topic belongs to a single agent. One agent can have many Topics |

### Analogy for Legal Teams

> Imagine a law firm's intake process. When a new client calls, the receptionist follows different scripts depending on the request:
> - "I need a contract reviewed" → follow the **Contract Review Intake** script
> - "I want to file a complaint" → follow the **Complaint Filing** script
> - "What are your fees?" → follow the **Fee Inquiry** script
>
> Each script is a **Topic**. The receptionist (the agent) decides which script to follow based on what the caller says (the trigger phrases).

### Types of Topics

| Type | Description | Example |
|------|-------------|---------|
| **Custom Topics** | Topics you build from scratch for your business processes | "Request Contract Review", "Check Compliance Status" |
| **System Topics** | Pre-built Topics that handle common interactions | Greeting, Goodbye, Escalate, Fallback |
| **Generative Topics** | AI-generated responses using Knowledge sources (no manual scripting needed) | "What does our data protection policy say about retention?" |

### How Topics Get Triggered

```
User types: "I need to get an NDA for a new vendor"

Agent evaluates:
  ├─ Topic A: "Contract Review" — trigger phrases: review contract, check contract...  ❌ No match
  ├─ Topic B: "NDA Request" — trigger phrases: need an NDA, request NDA, new NDA...    ✅ Match!
  └─ Topic C: "Fee Inquiry" — trigger phrases: how much, cost, fees...                 ❌ No match

→ Topic B is activated → Agent follows the NDA Request conversation flow
```

> 💡 **With Generative Orchestration enabled**, the agent uses AI to determine the best match rather than simple keyword matching. This means trigger phrases serve as guidance rather than strict pattern matching.

---

## 2. Topic Anatomy — Triggers, Nodes, and Branches

A Topic is composed of **nodes** — each node is a step in the conversation. Here is the full palette of node types:

### Node Types

| Node Type | Icon | What It Does | Example |
|-----------|------|-------------|---------|
| **Trigger** | ⚡ | Defines what phrases start the topic | "Request a contract", "I need legal review" |
| **Send a message** | 💬 | Displays text to the user | "I'll help you with that contract request." |
| **Ask a question** | ❓ | Asks the user a question and saves their response to a variable | "What type of contract is this?" → saves to `Topic.ContractType` |
| **Add a condition** | 🔀 | Creates branches based on variable values | If `Topic.ContractType` = "NDA" → go left; else → go right |
| **Variable management** | 📦 | Set, clear, or transform variables using formulas | `Topic.FullReference = Concatenate(Topic.ContractType, "-", Topic.ClientName)` |
| **Call an action** | ⚙️ | Runs a Power Automate flow or AI Builder prompt | Call "Create Approval Request" flow |
| **Topic management** | 🔄 | Redirect to another topic, end current topic, or transfer to agent | End current topic, or redirect to "Log Interaction" topic |
| **Generative answers** | 🤖 | Uses AI + Knowledge to generate a dynamic answer | "Based on our compliance policy, the retention period is..." |

### Visual Example — Simple Topic Structure

```
[Trigger] "I need a contract reviewed"
    │
    ▼
[Message] "I can help with that! Let me collect some details."
    │
    ▼
[Question] "What type of contract?"
   → Multiple choice: NDA / Service Agreement / Employment / Vendor
   → Saves to: Topic.ContractType
    │
    ▼
[Question] "Who is the other party?"
   → User's entire response
   → Saves to: Topic.OtherParty
    │
    ▼
[Question] "What is the contract value (if applicable)?"
   → Number
   → Saves to: Topic.ContractValue
    │
    ▼
[Condition] Topic.ContractValue > 500,000?
    │                    │
    ▼ YES                ▼ NO
[Message]            [Message]
"High-value           "Standard review.
contract — routing    I'll submit this
to senior counsel."   for review."
    │                    │
    ▼                    ▼
[Action: Route to    [Action: Create
 Senior Review]       Standard Review]
    │                    │
    ▼                    ▼
[Message]            [Message]
"Submitted to        "Review request
senior counsel.       submitted.
Reference: {ref}"    Reference: {ref}"
    │                    │
    ▼                    ▼
[End topic]          [End topic]
```

### Key Points About Nodes

- **Every branch must end** with "End current topic" or a redirect — otherwise the conversation may behave unpredictably
- **Variables are created inline** as you build nodes (via the "Create new" option in the "Save response as" dropdown) — there is no separate "add variable" button
- **Conditions** create parallel branches — you can chain multiple conditions for complex routing logic
- **"Call an action"** is the bridge to external systems — it invokes Power Automate flows that connect to SharePoint, Outlook, Dataverse, or any API

---

## 3. What Are Workflows?

In Copilot Studio, **"Workflow"** refers to the end-to-end process that combines **Topics**, **Power Automate flows**, **Conditions**, and **Agent transfers** into a complete business process automation.

### Workflow = Topic + Actions + External Systems

| Component | Role in the Workflow |
|-----------|---------------------|
| **Topic** | The conversation layer — collects input, displays output, guides the user |
| **Power Automate Flow** | The action layer — connects to external systems (SharePoint, Outlook, Dataverse, APIs) |
| **Conditions** | The decision layer — routes the conversation based on data or user choices |
| **Agent Transfer** | The delegation layer — hands off to a specialist agent when needed |
| **Knowledge + Generative Answers** | The intelligence layer — provides AI-generated responses from documents |

### Workflow Analogy for Legal Teams

> Think of a workflow like a legal case management process:
>
> 1. **Intake** (Topic) → Client describes the matter, paralegal collects key details
> 2. **Conflict check** (Power Automate Flow) → System queries the CRM for existing client relationships
> 3. **Triage** (Conditions) → If high-value or high-risk → route to partner; if standard → assign to associate
> 4. **Specialist handoff** (Agent Transfer) → Complex IP matter → transfer to IP specialist team
> 5. **Document generation** (Power Automate Flow) → Auto-generate engagement letter from template
> 6. **Follow-up** (Topic) → Confirm next steps with the client
>
> Each step uses a different capability, but together they form a complete **workflow**.

### How Workflows Differ from Simple Topics

| | Simple Topic | Workflow |
|--|-------------|----------|
| **Scope** | Answers a single question or collects one piece of info | Handles an entire business process end-to-end |
| **External systems** | None | Connects to SharePoint, Outlook, Dataverse, APIs via Power Automate |
| **Decision logic** | Simple or no branching | Multiple condition branches based on data |
| **Number of agents** | Single agent | May span multiple agents via orchestration |
| **Typical complexity** | 3-5 nodes | 10-30+ nodes across topics and flows |
| **Example** | "What are our office hours?" | "Submit, route, and track a contract review request end-to-end" |

---

## 4. Topics vs Workflows — When to Use Which

| Scenario | Use a Simple Topic | Use a Workflow |
|----------|-------------------|---------------|
| Answer a FAQ from documents | ✅ | |
| Collect one or two pieces of info and display a message | ✅ | |
| Route requests based on contract value or risk level | | ✅ |
| Create a calendar event and send confirmation email | | ✅ |
| Look up records in SharePoint, then branch based on status | | ✅ |
| Auto-generate a document and send for approval | | ✅ |
| Log an audit trail to Dataverse | | ✅ |
| Escalate high-risk items to a senior team member | | ✅ |

> 💡 **Rule of thumb**: If the agent only needs to talk and answer, use a **Topic**. If the agent needs to talk AND do something in an external system, you need a **Workflow** (Topic + Power Automate flow).

---

## 5. Example Scenario 1: Contract Approval Routing Workflow

### Business Context

The legal team receives contract approval requests. Contracts under HKD 500,000 go through standard review. Contracts above HKD 500,000 require senior counsel approval. All requests must be logged.

### Workflow Design

```
┌────────────────────────────────────────────────────────────────┐
│  TOPIC: "Contract Approval Request"                            │
│                                                                │
│  [Trigger] "I need contract approval" / "Submit contract       │
│            for review" / "Approve a contract"                  │
│      │                                                         │
│      ▼                                                         │
│  [Message] "I'll help you submit a contract for approval.      │
│            Let me gather the details."                         │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What is the contract title?"                      │
│     Identify: User's entire response → Topic.ContractTitle     │
│      │                                                         │
│      ▼                                                         │
│  [Question] "Who is the counterparty?"                         │
│     Identify: User's entire response → Topic.Counterparty      │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What type of contract?"                           │
│     Multiple choice:                                           │
│       • NDA / Non-Disclosure Agreement                         │
│       • Service Agreement                                      │
│       • Vendor Agreement                                       │
│       • Employment Contract                                    │
│       • Licensing Agreement                                    │
│     → Topic.ContractType                                       │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What is the total contract value (HKD)?"          │
│     Identify: Number → Topic.ContractValue                     │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What is the target completion date?"              │
│     Identify: Date and time → Topic.TargetDate                 │
│      │                                                         │
│      ▼                                                         │
│  [Question] "Any additional notes or special terms to flag?"   │
│     Identify: User's entire response → Topic.Notes             │
│      │                                                         │
│      ▼                                                         │
│  [Message] "Here's your contract submission summary:           │
│     📄 Title: {Topic.ContractTitle}                            │
│     🏢 Counterparty: {Topic.Counterparty}                     │
│     📋 Type: {Topic.ContractType}                              │
│     💰 Value: HKD {Topic.ContractValue}                        │
│     📅 Target Date: {Topic.TargetDate}                         │
│     📝 Notes: {Topic.Notes}                                    │
│                                                                │
│     Shall I submit this for approval?"                         │
│      │                                                         │
│      ▼                                                         │
│  [Question] "Confirm submission?"                              │
│     Multiple choice: Yes, submit / No, let me edit             │
│     → Topic.SubmitConfirm                                      │
│      │                                                         │
│      ▼                                                         │
│  [Condition] Topic.SubmitConfirm = "Yes, submit"               │
│      │                          │                              │
│      ▼ TRUE                     ▼ ELSE                         │
│  [Condition]               [Message] "No problem.              │
│  Topic.ContractValue        Let me know when                   │
│  > 500000?                  you're ready."                     │
│      │           │         [End topic]                          │
│      ▼ YES       ▼ NO                                          │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐                            │
│  │HIGH VALUE    │  │STANDARD      │                            │
│  │PATH          │  │PATH          │                            │
│  └──────┬───────┘  └──────┬───────┘                            │
│         │                 │                                    │
│         ▼                 ▼                                    │
│  [Call Action:       [Call Action:                              │
│   "Route to Senior    "Create Standard                         │
│    Counsel Review"]    Review Request"]                        │
│         │                 │                                    │
│         ▼                 ▼                                    │
│  [Call Action:       [Call Action:                              │
│   "Log to              "Log to                                 │
│    Dataverse"]          Dataverse"]                            │
│         │                 │                                    │
│         ▼                 ▼                                    │
│  [Message]           [Message]                                 │
│  "⚠️ High-value       "✅ Submitted for                       │
│   contract. Routed     standard review.                        │
│   to senior counsel.   Ref: {Topic.RefNumber}                  │
│   Ref: {Topic.RefNum}  Expected turnaround:                    │
│   You'll receive an    3-5 business days."                     │
│   email within 24h."                                           │
│         │                 │                                    │
│         ▼                 ▼                                    │
│  [End topic]         [End topic]                               │
└────────────────────────────────────────────────────────────────┘
```

### Power Automate Flows Required

**Flow 1: "Route to Senior Counsel Review"**

| Step | Action | Configuration |
|------|--------|---------------|
| 1 | Trigger: When Copilot Studio calls a flow | Inputs: `ContractTitle` (Text), `Counterparty` (Text), `ContractType` (Text), `ContractValue` (Text), `TargetDate` (Text), `Notes` (Text) |
| 2 | **Office 365 Outlook — Send an email (V2)** | **To**: `senior.counsel@contoso.com`; **Subject**: `Concatenate("⚠️ HIGH VALUE Contract Review: ", triggerBody()?['text'])` — *(ContractTitle)*; **Body**: Formatted with all contract details, value, counterparty, and notes |
| 3 | **Dataverse — Add a new row** | Table: `ContractReviews`; Map all fields; Set `Priority` = "High"; Set `Status` = "Pending Senior Review" |
| 4 | Return value(s) to Copilot Studio | Output: `RefNumber` (Text) = row ID from Dataverse |

**Flow 2: "Create Standard Review Request"**

| Step | Action | Configuration |
|------|--------|---------------|
| 1 | Trigger: When Copilot Studio calls a flow | Same inputs as above |
| 2 | **Dataverse — Add a new row** | Table: `ContractReviews`; Map all fields; Set `Priority` = "Standard"; Set `Status` = "Pending Review" |
| 3 | **Office 365 Outlook — Send an email (V2)** | **To**: `legal.review@contoso.com`; **Subject**: `Concatenate("Contract Review Request: ", triggerBody()?['text'])`; **Body**: Formatted details |
| 4 | Return value(s) to Copilot Studio | Output: `RefNumber` (Text) = row ID from Dataverse |

### Why This Matters for Legal

- **Audit trail**: Every submission is logged to Dataverse with timestamp, requester, and all details
- **Consistent routing**: The HKD 500,000 threshold is enforced by the system — no human judgment variance
- **Email notifications**: Senior counsel and the review team are notified automatically
- **Accountability**: Requesters get a reference number for tracking

---

## 6. Example Scenario 2: Legal Compliance Check Topic

### Business Context

Employees frequently ask the legal team whether certain business activities comply with internal policy. Rather than waiting for a lawyer's response, an agent provides instant guidance from the compliance knowledge base.

### Topic Design

This example uses **Knowledge + Generative Answers** — no Power Automate flow needed.

```
[Trigger] "Is this compliant?" / "Compliance check" /
          "Does this follow policy?" / "Legal question" /
          "Am I allowed to..."
    │
    ▼
[Message] "I can help check that against our compliance
          policies. Please describe the activity or
          situation you'd like me to review."
    │
    ▼
[Question] "Describe the activity or situation:"
   Identify: User's entire response
   → Topic.ComplianceQuery
    │
    ▼
[Question] "Which area does this relate to?"
   Multiple choice:
     • Data Protection & Privacy
     • Anti-Money Laundering (AML)
     • Client Communication
     • Sales Practices
     • Third-Party Sharing
     • Gifts & Entertainment
   → Topic.ComplianceArea
    │
    ▼
[Generative Answers Node]
   Activity input: {Topic.ComplianceQuery}
   Knowledge sources: Compliance_Policy.docx,
                      Data_Protection_Policy.docx,
                      AML_Guidelines.docx
    │
    ▼
[Message] "⚠️ Disclaimer: This guidance is based on our
          internal policy documents and is not legal
          advice. For binding determinations, please
          consult the legal team directly at
          legal@contoso.com."
    │
    ▼
[Question] "Was this helpful?"
   Multiple choice:
     • Yes, that answers my question
     • I need to speak with a lawyer
   → Topic.Satisfaction
    │
    ▼
[Condition] Topic.Satisfaction = "I need to speak with a lawyer"
    │                    │
    ▼ TRUE               ▼ ELSE
[Message]            [Message]
"I'll connect         "Glad I could
you with the          help! Remember,
legal team.           for specific
Sending a request     cases, always
now..."               consult legal."
    │                    │
    ▼                    ▼
[Action: Send          [End topic]
 Escalation Email
 to Legal]
    │
    ▼
[Message] "A member of
the legal team will
contact you within
24 hours. Ref: {ref}"
    │
    ▼
[End topic]
```

### Knowledge Sources Required

Upload these documents to SharePoint and connect them as Knowledge:

| Document | Contents |
|----------|----------|
| `Compliance_Policy.docx` | Company compliance guidelines — sales practices, communication rules, cool-off periods |
| `Data_Protection_Policy.docx` | GDPR/PDPO compliance, data retention, client consent requirements |
| `AML_Guidelines.docx` | Anti-money laundering thresholds, suspicious transaction reporting, KYC requirements |

### Key Design Decisions

| Decision | Reasoning |
|----------|-----------|
| **Disclaimer message after every answer** | Legal requirement — AI-generated guidance must be clearly marked as non-binding |
| **Escalation path always available** | Users must never feel trapped — they can always reach a human lawyer |
| **Knowledge-based answers, not hard-coded** | When policies change, update the SharePoint document — the agent's answers update automatically |
| **Compliance area selection** | Helps scope the generative answer and makes the query more specific for better AI results |

---

## 7. Example Scenario 3: NDA Request & Generation Workflow

### Business Context

Business units frequently request NDAs for new vendor relationships, partnerships, or client engagements. The legal team wants to automate the standard NDA process while maintaining oversight for non-standard requests.

### Workflow Design

```
┌────────────────────────────────────────────────────────────────┐
│  TOPIC: "NDA Request"                                          │
│                                                                │
│  [Trigger] "I need an NDA" / "Request NDA" / "New NDA" /      │
│            "Non-disclosure agreement"                          │
│      │                                                         │
│      ▼                                                         │
│  [Message] "I'll help you get an NDA set up. Let me            │
│            collect the required information."                  │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What type of NDA do you need?"                    │
│     Multiple choice:                                           │
│       • Mutual NDA (both parties share confidential info)      │
│       • One-Way NDA (only one party discloses)                 │
│     → Topic.NDAType                                            │
│      │                                                         │
│      ▼                                                         │
│  [Question] "Who is the other party? (company name)"           │
│     Identify: User's entire response → Topic.OtherParty        │
│      │                                                         │
│      ▼                                                         │
│  [Question] "What is the purpose of the NDA?"                  │
│     Multiple choice:                                           │
│       • Vendor evaluation                                      │
│       • Partnership discussion                                 │
│       • Client engagement                                      │
│       • M&A due diligence                                      │
│       • Technology licensing                                   │
│       • Other                                                  │
│     → Topic.NDAPurpose                                         │
│      │                                                         │
│      ▼                                                         │
│  [Condition] Topic.NDAPurpose = "M&A due diligence"            │
│      │                          │                              │
│      ▼ TRUE                     ▼ ELSE (all other purposes)    │
│                                                                │
│  [Message]                 [Question] "How long should         │
│  "⚠️ M&A NDAs require      the NDA be effective?"             │
│   senior legal review       Multiple choice:                   │
│   and cannot be auto-        • 1 year                          │
│   generated. I'm             • 2 years                         │
│   routing this to            • 3 years                         │
│   the M&A team."             • 5 years                         │
│      │                      → Topic.NDADuration                │
│      ▼                          │                              │
│  [Action: Route to              ▼                              │
│   M&A Legal Team]          [Question] "Contact email for       │
│      │                      the other party's signatory?"      │
│      ▼                      Identify: Email                    │
│  [Message]                  → Topic.SignatoryEmail              │
│  "Routed to M&A                │                              │
│   legal. They'll                ▼                              │
│   contact you                                                  │
│   within 48 hours."        [Message] Summary of request        │
│      │                          │                              │
│      ▼                          ▼                              │
│  [End topic]               [Question] "Confirm and submit?"    │
│                               → Topic.Confirm                  │
│                                 │                              │
│                                 ▼ YES                          │
│                            [Action: "Generate NDA              │
│                             from Template" flow]               │
│                                 │                              │
│                                 ▼                              │
│                            [Action: "Send NDA for              │
│                             Signature" flow]                   │
│                                 │                              │
│                                 ▼                              │
│                            [Action: "Log NDA Request           │
│                             to Dataverse" flow]                │
│                                 │                              │
│                                 ▼                              │
│                            [Message]                           │
│                            "✅ NDA generated and sent           │
│                             to {Topic.SignatoryEmail}           │
│                             for signature.                     │
│                             Ref: {Topic.RefNumber}             │
│                             Legal team CC'd."                  │
│                                 │                              │
│                                 ▼                              │
│                            [End topic]                         │
└────────────────────────────────────────────────────────────────┘
```

### Power Automate Flows Required

**Flow 1: "Generate NDA from Template"**

| Step | Action | Details |
|------|--------|---------|
| 1 | Trigger | Inputs: `NDAType`, `OtherParty`, `NDAPurpose`, `NDADuration`, `SignatoryEmail` |
| 2 | **SharePoint — Get file content** | Get the appropriate NDA template (Mutual or One-Way) from a `LegalTemplates` library |
| 3 | **Word Online — Populate a template** | Fill in placeholders: `{OtherParty}`, `{EffectiveDate}`, `{Duration}`, `{Purpose}` |
| 4 | **SharePoint — Create file** | Save the generated document to a `GeneratedNDAs` library |
| 5 | Return value(s) | `DocumentURL` (Text), `RefNumber` (Text) |

**Flow 2: "Send NDA for Signature"**

| Step | Action | Details |
|------|--------|---------|
| 1 | Trigger | Inputs: `SignatoryEmail`, `DocumentURL`, `OtherParty` |
| 2 | **Office 365 Outlook — Send an email (V2)** | To: `{SignatoryEmail}`; CC: `legal@contoso.com`; Attach the generated document; Professional cover letter |
| 3 | Return value(s) | `Confirmation` (Text) |

**Flow 3: "Log NDA Request to Dataverse"**

| Step | Action | Details |
|------|--------|---------|
| 1 | Trigger | Inputs: `NDAType`, `OtherParty`, `NDAPurpose`, `NDADuration`, `SignatoryEmail`, `DocumentURL` |
| 2 | **Dataverse — Add a new row** | Table: `NDARequests`; Map all fields; Status = "Sent for Signature" |
| 3 | Return value(s) | `RefNumber` (Text) |

### Legal Governance Built Into the Workflow

| Governance Point | How It's Implemented |
|-----------------|---------------------|
| **M&A NDAs require special handling** | Condition node routes M&A requests directly to the M&A legal team — no auto-generation |
| **All NDAs logged** | Every request is recorded in Dataverse with full details, timestamp, and requestor |
| **Legal team always CC'd** | The signature email CC's legal@contoso.com for every NDA sent |
| **Template-based generation** | NDAs are generated from approved templates in SharePoint — no free-form drafting by AI |
| **Audit trail** | Dataverse log + SharePoint document version history provide a complete audit chain |

---

## 8. Example Scenario 4: Regulatory Inquiry Triage Topic

### Business Context

When external regulators contact the company with inquiries, it's critical to route them correctly and respond within mandated timeframes. The agent performs initial triage and ensures the right team is notified immediately.

### Topic Design

```
[Trigger] "Regulatory inquiry" / "Regulator question" /
          "SFC inquiry" / "HKMA request" / "Compliance request
          from regulator" / "Government inquiry"
    │
    ▼
[Message] "⚠️ Regulatory inquiries are time-sensitive.
          I'll help route this immediately.
          All information will be logged for compliance."
    │
    ▼
[Question] "Which regulatory body is this from?"
   Multiple choice:
     • SFC (Securities and Futures Commission)
     • HKMA (Hong Kong Monetary Authority)
     • IA (Insurance Authority)
     • PCPD (Privacy Commissioner)
     • Other Government Body
   → Topic.Regulator
    │
    ▼
[Question] "What is the nature of the inquiry?"
   Multiple choice:
     • Information request
     • Investigation / Examination
     • Routine inspection
     • Complaint follow-up
     • Policy/license review
   → Topic.InquiryType
    │
    ▼
[Question] "What is the response deadline (if specified)?"
   Identify: Date and time → Topic.Deadline
    │
    ▼
[Question] "Brief description of the inquiry:"
   Identify: User's entire response → Topic.Description
    │
    ▼
[Question] "Your name and contact number for follow-up:"
   Identify: User's entire response → Topic.ContactInfo
    │
    ▼
[Condition] Topic.InquiryType = "Investigation / Examination"
    │                          │
    ▼ TRUE                     ▼ ELSE
                              
[Message]                 [Action: "Route
"🚨 INVESTIGATION          Standard Regulatory
ALERT — This will be       Inquiry" flow]
escalated to General           │
Counsel and the Chief          ▼
Compliance Officer         [Message]
immediately."              "✅ Inquiry logged and
    │                       routed to the
    ▼                       compliance team.
[Action: "Escalate          Ref: {Topic.RefNumber}"
 Investigation to              │
 General Counsel"              ▼
 flow]                     [End topic]
    │
    ▼
[Message]
"🚨 Escalated to General
Counsel and CCO.
They will contact you
within 1 hour.
Ref: {Topic.RefNumber}

DO NOT discuss this
inquiry with anyone
outside the legal team."
    │
    ▼
[End topic]
```

### Why This Design

| Design Decision | Legal Rationale |
|----------------|-----------------|
| **Investigation vs standard routing** | Investigations require immediate C-suite notification per most regulatory frameworks |
| **Deadline capture** | Regulatory deadlines are legally binding — capturing them ensures nothing is missed |
| **"Do not discuss" warning** | Standard legal hold / privilege protection for ongoing investigations |
| **All inquiries logged** | Regulatory response tracking is a compliance requirement |
| **Named contact capture** | Ensures there's always a responsible person for follow-up |

---

## 9. Configuring Workflows — Step-by-Step Walkthrough

This section walks through how to configure a complete workflow in Copilot Studio, using the **Contract Approval Routing** example from Scenario 1.

### Step 1: Create the Dataverse Table

Before building the agent, set up the data store.

1. Open **Power Apps** → **Tables** → **New table**
2. Name: `ContractReviews`
3. Add columns:

| Column | Type | Notes |
|--------|------|-------|
| ContractTitle | Text | |
| Counterparty | Text | |
| ContractType | Choice: `NDA`, `Service Agreement`, `Vendor Agreement`, `Employment`, `Licensing` | |
| ContractValue | Currency | |
| TargetDate | Date | |
| Notes | Multiline Text | |
| Priority | Choice: `Standard`, `High` | |
| Status | Choice: `Pending Review`, `Pending Senior Review`, `In Progress`, `Approved`, `Rejected` | |
| SubmittedBy | Text | |
| SubmittedDate | Date | Auto-set via flow |

### Step 2: Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Legal Services Assistant"**
3. Instructions:

```
You are the Legal Services Assistant for Contoso. You help employees with:
- Submitting contracts for legal review and approval
- Requesting NDAs and other standard legal documents
- Checking compliance with internal policies
- Routing regulatory inquiries to the appropriate team

Rules:
- Always collect complete information before submitting any request.
- Include a disclaimer that AI responses are not legal advice.
- For urgent matters, provide escalation contacts.
- Log all interactions for audit purposes.
- Be professional, precise, and clear.
- Protect confidentiality — never share details of one matter with another requester.
```

### Step 3: Build the Power Automate Flows

> 💡 **Design approach**: We use **two separate flows** — one for high-value contracts (senior counsel) and one for standard contracts. The **Topic** handles the routing logic via a condition node. This matches the architecture in Scenario 1 (Section 5) and is cleaner for maintainability — if the senior counsel process changes, you only edit that one flow.

#### Flow 1: "Route to Senior Counsel Review"

1. Open the agent → **Topics** → create a new topic (or open an existing one)
2. Click **+** → **"Call an action"** → **"Create a flow"**
3. Build the flow:

**Flow name**: `Route to Senior Counsel Review`

| Step | Action | Configuration |
|------|--------|---------------|
| 1 | **Trigger: When Copilot Studio calls a flow** | Add **six** text inputs: `ContractTitle` (Text), `Counterparty` (Text), `ContractType` (Text), `ContractValue` (Text), `TargetDate` (Text), `Notes` (Text) |
| 2 | **Compose** — "Map ContractType to Choice Value" | Click **Expression** tab (fx) → enter: `if(equals(triggerBody()?['text_2'],'NDA'),839560000,if(equals(triggerBody()?['text_2'],'Service Agreement'),839560001,if(equals(triggerBody()?['text_2'],'Vendor Agreement'),839560002,if(equals(triggerBody()?['text_2'],'Employment'),839560003,if(equals(triggerBody()?['text_2'],'Licensing'),839560004,839560000)))))` |
| 3 | **Dataverse — Add a new row** | **Table**: `ContractReviews` |
|   | | **ContractTitle**: Dynamic content → `ContractTitle` from trigger |
|   | | **Counterparty**: Dynamic content → `Counterparty` from trigger |
|   | | **ContractType**: Click **"Enter custom value"** → Dynamic content → **Outputs** from "Map ContractType to Choice Value" |
|   | | **ContractValue**: Expression tab → `float(triggerBody()?['text_3'])` |
|   | | **TargetDate**: Dynamic content → `TargetDate` from trigger |
|   | | **Notes**: Dynamic content → `Notes` from trigger |
|   | | **Priority**: Select **"High"** from the dropdown (hard-coded — this flow is only called for high-value contracts) |
|   | | **Status**: Select **"Pending Senior Review"** from the dropdown |
|   | | **SubmittedDate**: Expression tab → `utcNow()` |
| 4 | **Office 365 Outlook — Send an email (V2)** | **To**: `senior.counsel@contoso.com` |
|   | | **Subject**: Expression tab → `concat('⚠️ HIGH VALUE Contract Review: ', triggerBody()?['text'])` |
|   | | **Body**: Dynamic content → build with all trigger inputs: ContractTitle, Counterparty, ContractType, ContractValue, TargetDate, Notes |
| 5 | **Return value(s) to Copilot Studio** | Add output: `RefNumber` (Text) = Expression tab → `outputs('Add_a_new_row')?['body/crd49_contractreviewid']` |

4. **Save** the flow → return to Copilot Studio

> 💡 **How to find your Dataverse publisher prefix** (needed for the Return step expression):
>
> **Method 1 — From the flow run output (easiest):**
> 1. Save and test-run the flow in Power Automate (click **Test** → **Manually** → enter sample inputs → **Run flow**)
> 2. After the run completes, click the **"Add a new row"** step to expand it
> 3. Look at the **Outputs** → **Body** section — you'll see all the column names with their internal prefixes, e.g., `crd49_contracttitle`, `crd49_contractreviewid`
> 4. Note the prefix (e.g., `crd49_`) — this is your publisher prefix
> 5. Use this in the Return expression: `outputs('Add_a_new_row')?['body/crd49_contractreviewid']`
>
> **Method 2 — From Power Apps:**
> 1. Go to **Power Apps** → **Solutions** in the left navigation
> 2. Find and click the **solution** that contains your `ContractReviews` table (or the **Default Solution**)
> 3. Click the **Publisher** name shown under the solution details (or go to **Settings** ⚙️ → **Publishers**)
> 4. The **Prefix** field shows the publisher prefix (e.g., `cr0xx`)
>
> **Method 3 — From the Dataverse table directly:**
> 1. Go to **Power Apps** → **Tables** → **ContractReviews**
> 2. Click the **Properties** tab (or **Schema name** in the table details)
> 3. The schema name will show as something like `crd49_ContractReview` — the part before the underscore is your prefix
>
> ⚠️ **Common prefixes**: The default publisher prefix for new environments is often `crXXX` (e.g., `crd49`, `crc50`). Custom solutions will have the prefix you chose when creating the publisher. Your environment uses `crd49`.

> ⚠️ **Trigger input internal names** for this flow (6 inputs):
>
> | Display Name | Internal Name |
> |-------------|---------------|
> | ContractTitle | `text` |
> | Counterparty | `text_1` |
> | ContractType | `text_2` |
> | ContractValue | `text_3` |
> | TargetDate | `text_4` |
> | Notes | `text_5` |

#### Flow 2: "Create Standard Review Request"

1. Click **+** → **"Call an action"** → **"Create a flow"**
2. Build the flow:

**Flow name**: `Create Standard Review Request`

| Step | Action | Configuration |
|------|--------|---------------|
| 1 | **Trigger: When Copilot Studio calls a flow** | Same **six** text inputs as Flow 1: `ContractTitle`, `Counterparty`, `ContractType`, `ContractValue`, `TargetDate`, `Notes` |
| 2 | **Compose** — "Map ContractType to Choice Value" | Same expression as Flow 1: `if(equals(triggerBody()?['text_2'],'NDA'),839560000,if(equals(triggerBody()?['text_2'],'Service Agreement'),839560001,if(equals(triggerBody()?['text_2'],'Vendor Agreement'),839560002,if(equals(triggerBody()?['text_2'],'Employment'),839560003,if(equals(triggerBody()?['text_2'],'Licensing'),839560004,839560000)))))` |
| 3 | **Dataverse — Add a new row** | Same configuration as Flow 1, **except**: |
|   | | **Priority**: Select **"Standard"** from the dropdown |
|   | | **Status**: Select **"Pending Review"** from the dropdown |
| 4 | **Office 365 Outlook — Send an email (V2)** | **To**: `legal.review@contoso.com` *(general review team, not senior counsel)* |
|   | | **Subject**: Expression tab → `concat('Contract Review Request: ', triggerBody()?['text'])` |
|   | | **Body**: Dynamic content → all trigger inputs |
| 5 | **Return value(s) to Copilot Studio** | Add output: `RefNumber` (Text) = row ID expression (same as Flow 1) |

3. **Save** the flow

> 💡 **Why two flows instead of one?** Each flow has a single responsibility — this makes them easier to test, debug, and modify independently. If the senior counsel notification process changes (e.g., adding Teams notification), you only edit Flow 1. The Topic handles the routing decision.

### Step 4: Build the Topic — Detailed Instructions

#### Step 4.1: Create the Topic

1. Open the **Legal Services Assistant** agent in Copilot Studio
2. Go to **Topics** in the left navigation
3. Click **+ Add a topic** → **From blank**
4. Click the topic name at the top and rename it to: **`Contract Approval Request`**

#### Step 4.2: Add Trigger Phrases

1. Click the **Trigger** node at the top of the canvas
2. In the **Trigger phrases** panel on the right, add these phrases one by one (press Enter after each):
   - `Submit contract for review`
   - `Contract approval`
   - `I need a contract reviewed`
   - `Approve a contract`
   - `New contract review request`

#### Step 4.3: Add Welcome Message

1. Click **+** below the Trigger node → select **"Send a message"**
2. Type: `I'll help you submit a contract for approval. Let me gather the details.`

#### Step 4.4: Ask for Contract Title

1. Click **+** → **"Ask a question"**
2. Question text: `What is the contract title?`
3. **Identify**: Select **"User's entire response"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `ContractTitle`
   - Creates `Topic.ContractTitle` (string)

#### Step 4.5: Ask for Counterparty

1. Click **+** → **"Ask a question"**
2. Question text: `Who is the counterparty (the other party in the contract)?`
3. **Identify**: Select **"User's entire response"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `Counterparty`
   - Creates `Topic.Counterparty` (string)

#### Step 4.6: Ask for Contract Type

1. Click **+** → **"Ask a question"**
2. Question text: `What type of contract is this?`
3. **Identify**: Select **"Multiple choice options"**
4. Under **Options for user**, add five choices:
   - `NDA`
   - `Service Agreement`
   - `Vendor Agreement`
   - `Employment Contract`
   - `Licensing Agreement`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `ContractType`
   - Creates `Topic.ContractType` (**choice** type — auto-set, cannot be changed)

#### Step 4.7: Ask for Contract Value

1. Click **+** → **"Ask a question"**
2. Question text: `What is the total contract value in HKD?`
3. **Identify**: Select **"Number"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `ContractValue`
   - Creates `Topic.ContractValue` (**number** type — auto-set)

#### Step 4.8: Ask for Target Date

1. Click **+** → **"Ask a question"**
2. Question text: `What is the target completion date for the review? (e.g., April 30, 2026)`
3. **Identify**: Select **"Date and time"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `TargetDate`
   - Creates `Topic.TargetDate` (**datetime** type — auto-set, cannot be changed)

#### Step 4.9: Ask for Notes

1. Click **+** → **"Ask a question"**
2. Question text: `Any additional notes or special terms to flag?`
3. **Identify**: Select **"User's entire response"**
4. **Save response as**: Click the dropdown → **"Create new"** → name it `Notes`
   - Creates `Topic.Notes` (string)

#### Step 4.10: Show Confirmation Summary

1. Click **+** → **"Send a message"**
2. Type the following (use **{x}** button to insert each variable):

   ```
   Here's your contract submission summary:
   📄 Title: {Topic.ContractTitle}
   🏢 Counterparty: {Topic.Counterparty}
   📋 Type: {Topic.ContractType}
   💰 Value: HKD {Topic.ContractValue}
   📅 Target Date: {Topic.TargetDate}
   📝 Notes: {Topic.Notes}

   Shall I submit this for approval?
   ```

#### Step 4.11: Ask for Submission Confirmation

1. Click **+** → **"Ask a question"**
2. Question text: `Confirm submission?`
3. **Identify**: Select **"Multiple choice options"**
4. Options:
   - `Yes, submit`
   - `No, let me edit`
5. **Save response as**: Click the dropdown → **"Create new"** → name it `SubmitConfirm`

#### Step 4.12: Add First Condition — Submission Confirmation

1. Click **+** → **"Add a condition"** → **"Branch based on a condition"**
2. Configure:
   - **Select a variable**: `Topic.SubmitConfirm`
   - **Operator**: `is equal to`
   - **Value**: `Yes, submit`

3. Under the **Else** branch (right side):
   - Click **+** → **"Send a message"** → Type: `No problem. Let me know when you're ready to resubmit.`
   - Click **+** → **"Topic management"** → **"End current topic"**

#### Step 4.13: Add Second Condition — Contract Value Threshold

Under the **True** branch (left side of the submission confirmation):

1. Click **+** → **"Add a condition"** → **"Branch based on a condition"**
2. Configure:
   - **Select a variable**: `Topic.ContractValue`
   - **Operator**: `is greater than`
   - **Value**: `500000`

   > 💡 This creates two branches: **True** (left — high value) and **Else** (right — standard).

#### Step 4.14: Build the HIGH VALUE Path (True Branch — Left Side)

Under the **True** branch of the value condition (ContractValue > 500,000):

**14a: Convert ContractType (choice) to string**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `ContractTypeText`
3. **To value**: Switch to **Formula** mode (fx), enter:
   ```text
   Text(Topic.ContractType)
   ```

**14b: Convert TargetDate (datetime) to string**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `TargetDateText`
3. **To value**: Formula mode (fx), enter:
   ```text
   Text(Topic.TargetDate)
   ```

**14c: Convert ContractValue (number) to string**

1. Click **+** → **"Variable management"** → **"Set a variable value"**
2. **Set variable**: Click the dropdown → **"Create new"** → name it `ContractValueText`
3. **To value**: Formula mode (fx), enter:
   ```text
   Text(Topic.ContractValue)
   ```

> ⚠️ **Why convert all non-string variables?** The Power Automate flows expect all inputs as **Text**. Passing `number`, `datetime`, or `choice` variables directly to a flow's Text input will cause type mismatch errors. Always create string versions using `Text()`.

**14d: Call the "Route to Senior Counsel Review" flow**

1. Click **+** → **"Call an action"**
2. Select **"Route to Senior Counsel Review"** from the list
3. Map the inputs:

   | Flow Input | Set To |
   |-----------|--------|
   | **ContractTitle** | `Topic.ContractTitle` (already string) |
   | **Counterparty** | `Topic.Counterparty` (already string) |
   | **ContractType** | `Topic.ContractTypeText` (converted string) |
   | **ContractValue** | `Topic.ContractValueText` (converted string) |
   | **TargetDate** | `Topic.TargetDateText` (converted string) |
   | **Notes** | `Topic.Notes` (already string) |

4. **For the flow output**: Map `RefNumber` → click **"Create new"** → name it `RefNumber`

**14e: Show high-value confirmation message**

1. Click **+** → **"Send a message"**
2. Type: `⚠️ High-value contract (above HKD 500,000). Routed to senior counsel for review. Reference: ` then click **{x}** → select `Topic.RefNumber` → then type: `. You'll receive an email confirmation within 24 hours.`

**14f: End the branch**

1. Click **+** → **"Topic management"** → **"End current topic"**

#### Step 4.15: Build the STANDARD Path (Else Branch — Right Side)

Under the **Else** branch of the value condition (ContractValue ≤ 500,000):

**15a: Convert variables (reuse or create new)**

> ⚠️ The `Topic.ContractTypeText`, `Topic.TargetDateText`, and `Topic.ContractValueText` variables were created in Step 4.14 on the **True** branch. However, variables on one condition branch are **not guaranteed to be populated** on the other branch. You must add the same "Set variable value" nodes on this branch too.

1. Add three **"Set a variable value"** nodes — same as Steps 14a, 14b, 14c:
   - `Topic.ContractTypeText` = `Text(Topic.ContractType)`
   - `Topic.TargetDateText` = `Text(Topic.TargetDate)`
   - `Topic.ContractValueText` = `Text(Topic.ContractValue)`

   > 💡 Since the variables already exist (created in Step 4.14), you don't need to click "Create new" — just select them from the dropdown.

**15b: Call the "Create Standard Review Request" flow**

1. Click **+** → **"Call an action"**
2. Select **"Create Standard Review Request"** from the list
3. Map the inputs — same mapping as Step 14d:

   | Flow Input | Set To |
   |-----------|--------|
   | **ContractTitle** | `Topic.ContractTitle` |
   | **Counterparty** | `Topic.Counterparty` |
   | **ContractType** | `Topic.ContractTypeText` |
   | **ContractValue** | `Topic.ContractValueText` |
   | **TargetDate** | `Topic.TargetDateText` |
   | **Notes** | `Topic.Notes` |

4. **For the flow output**: Map `RefNumber` → select existing `Topic.RefNumber`

**15c: Show standard confirmation message**

1. Click **+** → **"Send a message"**
2. Type: `✅ Contract submitted for standard review. Reference: ` then click **{x}** → select `Topic.RefNumber` → then type: `. Expected turnaround: 3-5 business days.`

**15d: End the branch**

1. Click **+** → **"Topic management"** → **"End current topic"**

#### Complete Topic Flow — Visual Reference

```
[Trigger: "Submit contract for review" / "Contract approval" / ...]
    │
    ▼
[Message] "I'll help you submit a contract for approval..."
    │
    ▼
[Ask] "What is the contract title?"
   Identify: User's entire response → Topic.ContractTitle (string)
    │
    ▼
[Ask] "Who is the counterparty?"
   Identify: User's entire response → Topic.Counterparty (string)
    │
    ▼
[Ask] "What type of contract?"
   Identify: Multiple choice (5 options) → Topic.ContractType (choice)
    │
    ▼
[Ask] "Total contract value in HKD?"
   Identify: Number → Topic.ContractValue (number)
    │
    ▼
[Ask] "Target completion date?"
   Identify: Date and time → Topic.TargetDate (datetime)
    │
    ▼
[Ask] "Additional notes?"
   Identify: User's entire response → Topic.Notes (string)
    │
    ▼
[Message] Confirmation summary with all variables
    │
    ▼
[Ask] "Confirm submission?" → Yes, submit / No, let me edit
   → Topic.SubmitConfirm (choice)
    │
    ▼
[Condition] Topic.SubmitConfirm = "Yes, submit"
    │                              │
    ▼ TRUE                         ▼ ELSE
[Condition]                   [Message] "No problem."
Topic.ContractValue > 500000  [End current topic]
    │                │
    ▼ TRUE           ▼ ELSE
                    
── HIGH VALUE ──    ── STANDARD ──

[Set Variable]      [Set Variable]
  ContractTypeText    ContractTypeText
  = Text(Type)        = Text(Type)

[Set Variable]      [Set Variable]
  TargetDateText      TargetDateText
  = Text(Date)        = Text(Date)

[Set Variable]      [Set Variable]
  ContractValueText   ContractValueText
  = Text(Value)       = Text(Value)

[Call Action:       [Call Action:
 "Route to Senior    "Create Standard
  Counsel Review"]    Review Request"]
  Map all inputs      Map all inputs

[Message]           [Message]
"⚠️ High-value.     "✅ Standard review.
 Routed to senior    Ref: {RefNumber}
 counsel.            Turnaround: 3-5
 Ref: {RefNumber}"   business days."

[End topic]         [End topic]
```

#### Variables Summary

| Variable | Type | Created In | Used In |
|----------|------|-----------|---------|
| `Topic.ContractTitle` | string | Step 4.4 | Summary message, both flows |
| `Topic.Counterparty` | string | Step 4.5 | Summary message, both flows |
| `Topic.ContractType` | **choice** | Step 4.6 (auto-set by Multiple choice) | Summary message. Convert with `Text()` before passing to flow |
| `Topic.ContractValue` | **number** | Step 4.7 (auto-set by Number entity) | Condition (> 500000), summary message. Convert with `Text()` before passing to flow |
| `Topic.TargetDate` | **datetime** | Step 4.8 (auto-set by Date and time entity) | Summary message. Convert with `Text()` before passing to flow |
| `Topic.Notes` | string | Step 4.9 | Summary message, both flows |
| `Topic.SubmitConfirm` | **choice** | Step 4.11 (auto-set) | Condition branch |
| `Topic.ContractTypeText` | string | Step 4.14a / 4.15a | Flow input (ContractType) |
| `Topic.ContractValueText` | string | Step 4.14c / 4.15a | Flow input (ContractValue) |
| `Topic.TargetDateText` | string | Step 4.14b / 4.15a | Flow input (TargetDate) |
| `Topic.RefNumber` | string | Step 4.14d (flow output mapping) | Confirmation messages on both branches |

#### Troubleshooting This Topic

| Issue | Solution |
|-------|----------|
| **Flow input shows type error (DateTime, Number, or Choice)** | You must convert non-string variables first. Use "Set variable value" with `Text()` formula, then pass the converted string variable to the flow |
| **Condition on ContractValue doesn't work** | Ensure the variable is **number** type (set by the "Number" Identify entity). If you used "User's entire response", the variable is string and comparisons won't work correctly |
| **Both branches call the same flow by accident** | Verify each "Call an action" node: left branch should show "Route to Senior Counsel Review", right branch should show "Create Standard Review Request" |
| **RefNumber is blank in the confirmation message** | Check the flow in Power Automate → Run history. The Dataverse row ID expression in the Return step may need adjustment for your table's publisher prefix |
| **"Set variable value" nodes on the Else branch show errors** | You may need to re-select the variables (they were created on the True branch). Click the "Set variable" dropdown → select the existing variable (do not create new) |
| **Contract value of exactly 500,000 goes to high-value path** | Check the condition operator: it should be **"is greater than"** (not "is greater than or equal to"). 500,000 exactly should route to Standard |

### Step 5: Configure Variable Inputs for Multi-Agent (Optional)

If this agent is a sub-agent of a Hub:

1. Open the Topic → click **Variables** in the top toolbar
2. Enable **"Receive values from other topics"** for:
   - `Topic.ContractTitle` (string — safe)
   - `Topic.Counterparty` (string — safe)
   - `Topic.ContractValue` (number — safe)
3. Do **NOT** enable it for choice variables used in `Concatenate(Text(...))` formulas

### Step 6: Test the Workflow

| Test Case | Input | Expected Result |
|-----------|-------|-----------------|
| **Standard contract** | Value: 200,000 | Logged with Priority = Standard; email to legal.review@ |
| **High-value contract** | Value: 1,500,000 | Logged with Priority = High; email to senior.counsel@ |
| **Cancel flow** | "No, let me edit" at confirmation | Topic ends gracefully; nothing logged |
| **Edge case** | Value: 500,000 (exactly at threshold) | Should route to Standard (condition is > 500,000, not >=) |

---

## 10. Governance & Audit Considerations for Legal Teams

### Built-In Compliance Features

| Feature | How It Supports Legal/Compliance |
|---------|--------------------------------|
| **Dataverse logging** | Every interaction, request, and decision is stored in structured tables with timestamps |
| **Power Automate run history** | Every flow execution is logged — who triggered it, what inputs were sent, what the outcome was |
| **Copilot Studio analytics** | Session transcripts, topic usage, resolution rates — all available for review |
| **SharePoint version history** | Document templates and knowledge sources maintain full version history |
| **Conditional routing** | Systematic, unbiased routing based on defined thresholds — eliminates human judgment variance |
| **Disclaimer enforcement** | The agent always adds compliance disclaimers before providing guidance |
| **Escalation paths** | Every Topic can include a path to a human expert — users are never trapped in an AI loop |

### Data Protection Considerations

| Consideration | Recommendation |
|--------------|----------------|
| **Data residency** | Copilot Studio data is stored in the same region as your Microsoft 365 tenant. Verify this meets your regulatory requirements. |
| **Data retention** | Configure Dataverse table retention policies to align with your legal hold and document retention policies |
| **Access control** | Use Dataverse security roles to restrict who can view contract data, regulatory inquiry logs, etc. |
| **User authentication** | Copilot Studio agents can require Microsoft Entra ID authentication — ensuring only authorized employees can access the system |
| **Conversation transcripts** | Session transcripts are available in Copilot Studio analytics. Define a retention policy for these records. |
| **AI-generated content** | Always include disclaimers. Generative answers from Knowledge sources should reference the source document for verifiability. |

### Audit Trail Architecture

```
┌─────────────────────────────────────────────────────┐
│  AUDIT TRAIL                                        │
│                                                     │
│  Layer 1: Copilot Studio Analytics                  │
│  ├─ Session transcripts (full conversation logs)    │
│  ├─ Topic trigger counts and resolution rates       │
│  └─ User satisfaction metrics                       │
│                                                     │
│  Layer 2: Power Automate Run History                │
│  ├─ Every flow execution (inputs + outputs)         │
│  ├─ Success/failure logs with error details         │
│  └─ Connector authentication logs                   │
│                                                     │
│  Layer 3: Dataverse Tables                          │
│  ├─ ContractReviews (all submissions + status)      │
│  ├─ NDARequests (all NDA lifecycle data)            │
│  ├─ RegulatoryInquiries (all inquiry logs)          │
│  └─ InteractionLog (all agent interactions)         │
│                                                     │
│  Layer 4: SharePoint                                │
│  ├─ Document version history (templates, NDAs)      │
│  └─ Knowledge source change tracking                │
│                                                     │
│  Layer 5: Microsoft 365 Audit Log                   │
│  ├─ Outlook email send/receive logs                 │
│  ├─ Calendar event creation logs                    │
│  └─ SharePoint access logs                          │
└─────────────────────────────────────────────────────┘
```

---

## 11. Key Gotchas & Best Practices

### Topics — Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Trigger phrases overlap between topics | Make trigger phrases specific; use different keywords for each topic |
| Forgetting to end branches | Always add "End current topic" at the end of every branch |
| Hard-coding text in message nodes that should be dynamic | Use variables via the {x} button — update once, changes everywhere |
| Topic becomes too complex (>30 nodes) | Split into multiple topics and use "Redirect to another topic" to chain them |
| Testing with stale data | Always click the refresh/reset icon in the test panel to start a clean session |

### Workflows — Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Flow not appearing in "Call an action" | Ensure the flow uses the "When Copilot Studio calls a flow" trigger; refresh the page |
| Choice variables cause errors in formulas | Wrap with `Text()` in `Concatenate()`; use Compose steps in the flow to map text to numeric values |
| Connector auth expires | Pre-authenticate all connectors before demos/production use; test flows in Power Automate first |
| Passing datetime/choice to flow Text inputs | Create intermediate string variables with `Text()` conversion before the "Call an action" node |
| Flow returns success but data is wrong | Check the flow run history in Power Automate — inspect each step's inputs/outputs |

### Legal-Specific Best Practices

| Practice | Why |
|----------|-----|
| **Always include AI disclaimers** | Any AI-generated content must be labeled as informational, not legal advice |
| **Require confirmation before external actions** | Never auto-send emails or auto-create records without user confirmation |
| **Log everything to Dataverse** | Create a complete audit trail that can be queried, reported on, and retained |
| **Use approved templates, not AI generation, for legal documents** | Word Online "Populate a template" is more reliable and legally defensible than AI text generation for contracts |
| **Build in escalation at every decision point** | Users must always have a path to a human lawyer |
| **Version-control your knowledge sources** | Store compliance documents in SharePoint with version history enabled — you need to know what guidance was active on any given date |
| **Test with edge cases** | Boundary values (e.g., exactly HKD 500,000), empty inputs, unexpected date formats |
| **Review AI-generated compliance guidance regularly** | Even though answers come from your documents, the AI's interpretation should be spot-checked periodically |

---

## Quick Reference Card

| Term | Definition |
|------|-----------|
| **Topic** | A structured conversation path in Copilot Studio — handles one user intent with triggers, nodes, and branches |
| **Workflow** | An end-to-end business process combining Topics + Power Automate flows + conditions + agent transfers |
| **Trigger Phrase** | Words/sentences that activate a Topic (e.g., "I need an NDA") |
| **Node** | A single step in a Topic — message, question, condition, action, or variable management |
| **Variable** | Data collected during a conversation — created inline via "Ask a question" or "Set variable value" nodes |
| **Power Automate Flow** | A cloud flow that connects the agent to external systems (SharePoint, Outlook, Dataverse) |
| **Condition** | A branching node that routes the conversation based on variable values |
| **Generative Answers** | AI-generated responses using Knowledge sources (documents in SharePoint or uploaded files) |
| **Slot Filling** | The orchestrator pre-fills Topic Input variables from the user's message, skipping redundant questions |
| **Agent Transfer** | The orchestrator routes the conversation to a specialized sub-agent |
| **Escalation** | A path in the Topic that connects the user to a human expert |
| **Audit Trail** | The combined logs from Copilot Studio analytics, Power Automate run history, and Dataverse tables |
