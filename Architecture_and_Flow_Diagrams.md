> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Contoso Smart Workspace — Architecture & Flow Diagrams

> All diagrams for the multi-agent system with connectors. Use as a quick visual reference during setup, demos, or troubleshooting.

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Multi-Agent Orchestration Flow](#2-multi-agent-orchestration-flow)
3. [Agent 2: Policy Lookup Agent — Topic Flow](#3-agent-2-policy-lookup-agent--topic-flow)
4. [Agent 3: Product Advisor Agent — Topic Flow](#4-agent-3-product-advisor-agent--topic-flow)
5. [Agent 4: Scheduling Assistant — Power Automate Flows](#5-agent-4-scheduling-assistant--power-automate-flows)
6. [Agent 4: Scheduling Assistant — Topic Flows](#6-agent-4-scheduling-assistant--topic-flows)
7. [Slot-Filling Behavior (With vs Without)](#7-slot-filling-behavior-with-vs-without)
8. [Data Model Reference](#8-data-model-reference)

---

## 1. System Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                   USER (Financial Advisor)                    │
│                          │                                   │
│                          ▼                                   │
│         ┌────────────────────────────────┐                   │
│         │   🤖 Primary Agent             │                   │
│         │   "Contoso Advisor Hub"        │                   │
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

### Agent Roles

| Agent | Role | Connectors / Knowledge |
|-------|------|----------------------|
| **Contoso Advisor Hub** | Orchestrator — routes to specialists | SharePoint (Knowledge: ProductGuides library) |
| **Policy Lookup Agent** | Queries client data | SharePoint (Data: ClientPolicies list) via Power Automate |
| **Product Advisor Agent** | Answers product questions, recommends plans | SharePoint (Knowledge: ProductGuides library) |
| **Scheduling Assistant** | Books meetings, sends emails, logs interactions | Outlook (Calendar + Email), Dataverse (InteractionLog) via Power Automate |

---

## 2. Multi-Agent Orchestration Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  USER MESSAGE                                                   │
│  "I need to check the policy for David Chan"                    │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  CONTOSO ADVISOR HUB (Generative Orchestration)                 │
│                                                                 │
│  1. Analyze intent from user message                            │
│  2. Match intent to sub-agent descriptions:                     │
│     • "policy lookup" → Policy Lookup Agent                     │
│     • "product recommendation" → Product Advisor Agent          │
│     • "schedule meeting / send email" → Scheduling Assistant    │
│  3. Extract entities: ClientName = "David Chan"                 │
│  4. Transfer to matched agent with pre-filled variables         │
└─────────┬──────────────────┬──────────────────┬─────────────────┘
          │                  │                  │
          ▼                  ▼                  ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
   │ Policy Lookup │  │ Product      │  │ Scheduling       │
   │ Agent         │  │ Advisor      │  │ Assistant        │
   │               │  │ Agent        │  │                  │
   │ Slot-filled:  │  │ Slot-filled: │  │ Slot-filled:     │
   │ • ClientName  │  │ • ClientAge  │  │ • ClientName     │
   │ • SearchMethod│  │ • Concern    │  │ • ClientEmail    │
   │               │  │ • PreExist   │  │ (NOT Purpose —   │
   │               │  │ • Budget     │  │  choice variable)│
   └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘
          │                  │                   │
          ▼                  ▼                   ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
   │ SharePoint   │  │ SharePoint   │  │ Outlook +        │
   │ List Query   │  │ Knowledge    │  │ Dataverse        │
   │ (Get Items)  │  │ (Gen. AI)    │  │ (Events, Email,  │
   │              │  │              │  │  Log Rows)       │
   └──────────────┘  └──────────────┘  └──────────────────┘
```

---

## 3. Agent 2: Policy Lookup Agent — Topic Flow

### "Lookup by Client Name" Topic

```
[Trigger: "Find client" / "Look up policy" / "Client information" / ...]
    │
    ▼
[Message] "Sure! Let me search for the client. How would you like to search?"
    │
    ▼
[Ask] "Search by:" → Multiple choice: Client Name / Policy Number
    → Topic.SearchMethod
    │
    ▼
[Condition] Topic.SearchMethod = ?
    │                                        │
    ▼ "Client Name"                          ▼ "Policy Number"
    │                                        │
[Ask] "What is the client's name?"     [Ask] "What is the policy number?"
  → Topic.ClientName                     → Topic.PolicyNumber1
    │                                        │
[Set] Global.SearchFieldValue          [Set] Global.SearchFieldValue
  = "Title"                              = "PolicyNumber"
    │                                        │
[Call Action: Search Client             [Call Action: Search Client
  Policies Flow]                          Policies Flow]
  SearchField ← Global.SearchFieldValue   SearchField ← Global.SearchFieldValue
  SearchValue ← Topic.ClientName          SearchValue ← Topic.PolicyNumber1
    │                                        │
[Condition] PolicyNumber                 [Condition] ReturnedClientName
  is not blank?                            is not blank?
    │           │                            │           │
    ▼ YES       ▼ NO                         ▼ YES       ▼ NO
    │           │                            │           │
[Message]    [Message]                   [Message]    [Message]
 Policy       "Not found"                Policy       "Not found"
 details      [End topic]                details      [End topic]
[End topic]                              [End topic]
```

### "Search Client Policies" Power Automate Flow

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER: When Copilot Studio calls a flow              │
│  Inputs:                                                │
│    • SearchField  (Text)  ← "Title" or "PolicyNumber"  │
│    • SearchValue  (Text)  ← "David Chan" or "CTO-..."  │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  COMPOSE: Build Filter Query                            │
│  Expression:                                            │
│    if SearchField = "PolicyNumber"                       │
│      → "PolicyNumber eq 'CTO-2025-00123'"               │
│    else                                                 │
│      → "Title eq 'David Chan'"                          │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  SHAREPOINT: Get items                                  │
│    Site:          [Your SharePoint site]                 │
│    List:          ClientPolicies                        │
│    Filter Query:  [Build Filter Query output]           │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  COMPOSE: Get First Result                              │
│    first(outputs('Get_items')?['body/value'])            │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Return value(s) to Copilot Studio              │
│  Outputs (10):                                          │
│    • ClientName      ← ['Title']                        │
│    • PolicyNumber    ← ['PolicyNumber']                  │
│    • PolicyType      ← ['PolicyType']['Value']           │
│    • CoverageAmount  ← string(['CoverageAmount'])        │
│    • PolicyStatus    ← ['PolicyStatus']['Value']         │
│    • StartDate       ← ['StartDate']                     │
│    • EndDate         ← ['EndDate']                       │
│    • ClientEmail     ← ['ClientEmail']                   │
│    • AssignedAdvisor ← ['AssignedAdvisor']               │
│    • Notes           ← ['Notes']                         │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Agent 3: Product Advisor Agent — Topic Flow

### "Product Recommendation" Topic

```
[Trigger: "Recommend a product" / "What should I suggest" / "Best product for" / ...]
    │
    ▼
[Message] "I'll help you find the right product for your client.
           Let me gather some details."
    │
    ▼
[Ask] "What is the client's age?"
   Identify: Number → Topic.ClientAge
    │
    ▼
[Ask] "What is the client's primary concern?"
   Identify: Multiple choice → Topic.PrimaryConcern
   Options:
     • Income protection for family
     • Medical expense coverage
     • Critical illness lump-sum benefit
     • Travel protection
     • Retirement planning
    │
    ▼
[Ask] "Does the client have any pre-existing medical conditions?"
   Identify: Multiple choice → Topic.PreExisting
   Options: Yes / No / Not sure
    │
    ▼
[Ask] "What is the client's approximate monthly budget for insurance?"
   Identify: Multiple choice → Topic.Budget
   Options: Under HKD 500 / HKD 500-1500 / HKD 1500-3000 / Above HKD 3000
    │
    ▼
[Generative Answer Node — using Knowledge]
   Prompt: "Based on the following client profile,
   recommend the most suitable Contoso product(s)
   with reasoning:
   - Age: {Topic.ClientAge}
   - Primary concern: {Topic.PrimaryConcern}
   - Pre-existing conditions: {Topic.PreExisting}
   - Budget: {Topic.Budget}
   Use the product knowledge base to provide
   accurate details."
    │
    ▼
[Message] "Would you like me to compare specific plans
           side by side, or would you like to schedule
           a meeting with this client to discuss?"
```

---

## 5. Agent 4: Scheduling Assistant — Power Automate Flows

### 5a. "Schedule Client Meeting" Flow

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER: When Copilot Studio calls a flow              │
│  Inputs:                                                │
│    • Subject       (Text)  ← "Contoso - Policy..."     │
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
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  OFFICE 365 OUTLOOK: Create event (V4)                  │
│    Calendar id:          Calendar                       │
│    Subject:              [Subject]                      │
│    Start time:           [StartDateTime]                │
│    End time:             [Calculate End Time]           │
│    Time zone:            UTC+08:00 Hong Kong            │
│    Required attendees:   [AttendeeEmail]                │
│    Body:                 [Body]                         │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Confirmation (Text) = "Meeting created         │
│          successfully"                                  │
└─────────────────────────────────────────────────────────┘
```

### 5b. "Send Follow-Up Email" Flow

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER: When Copilot Studio calls a flow              │
│  Inputs:                                                │
│    • ToEmail    (Text)  ← "david.chan@email.com"        │
│    • Subject    (Text)  ← "Follow-up from Contoso..."  │
│    • EmailBody  (Text)  ← "Dear David Chan,..."        │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  OFFICE 365 OUTLOOK: Send an email (V2)                 │
│    To:       [ToEmail]                                  │
│    Subject:  [Subject]                                  │
│    Body:     [EmailBody]                                │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Confirmation (Text) = "Email sent              │
│          successfully"                                  │
└─────────────────────────────────────────────────────────┘
```

### 5c. "Log Interaction to Dataverse" Flow

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
│    ClientName:       [ClientName]                       │
│    InteractionType:  [Mapped Choice Value]              │
│    Summary:          [Summary]                          │
│    FollowUpRequired: Yes                                │
│    FollowUpDate:     [FollowUpDate]                     │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│  RETURN: Confirmation (Text) = "Interaction logged"     │
└─────────────────────────────────────────────────────────┘
```

---

## 6. Agent 4: Scheduling Assistant — Topic Flows

### 6a. "Schedule Client Meeting" Topic

```
[Trigger: "Schedule a meeting" / "Book an appointment" / ...]
    │
    ▼
[Message] "Let's get that meeting scheduled! I'll need a few details."
    │
    ▼
[Ask] "What is the client's full name?"
   Identify: User's entire response → Topic.ClientName (string)
    │
    ▼
[Ask] "What is the client's email address?"
   Identify: Email → Topic.ClientEmail (string)
    │
    ▼
[Ask] "What date and time would you like to schedule the meeting?"
   Identify: Date and time → Topic.MeetingDateTime (datetime)
    │
    ▼
[Ask] "How long should the meeting be?"
   Identify: Multiple choice (30 min / 45 min / 1 hour) → Topic.Duration (choice)
    │
    ▼
[Ask] "What is the purpose of this meeting?"
   Identify: Multiple choice (5 options) → Topic.MeetingPurpose (choice)
    │
    ▼
[Message] "Here's the meeting summary: ..."  (all variables via {x})
    │
    ▼
[Ask] "Confirm?" → Yes, create it / No → Topic.Confirmation (choice)
    │
    ▼
[Condition] Topic.Confirmation = "Yes, create it"
    │                                    │
    ▼ TRUE                               ▼ ELSE
    │                                    │
[Set Variable]                     [Message] "No problem!"
  MeetingSubject = Concatenate(    [End current topic]
    "Contoso - ",
    Text(Topic.MeetingPurpose),
    " with ", Topic.ClientName)
    │
    ▼
[Set Variable]
  MeetingBody = Concatenate(
    "Dear ", Topic.ClientName,
    ", This meeting has been
    scheduled to discuss: ",
    Text(Topic.MeetingPurpose),
    ". Best regards, Contoso
    Financial Advisory Team")
    │
    ▼
[Set Variable] MeetingDateTimeText = Text(Topic.MeetingDateTime)
    │
    ▼
[Set Variable] DurationText = Text(Topic.Duration)
    │
    ▼
[Call Action: Schedule Client Meeting Flow]
  Subject       ← Topic.MeetingSubject
  StartDateTime ← Topic.MeetingDateTimeText
  Duration      ← Topic.DurationText
  AttendeeEmail ← Topic.ClientEmail
  Body          ← Topic.MeetingBody
    │
    ▼
[Message] "✅ Meeting created! Invitation sent to {Topic.ClientEmail}."
    │
    ▼
[Ask] "Would you also like me to log this interaction?"
  → Yes / No → Topic.LogInteraction (choice)
    │
    ▼
[Condition] Topic.LogInteraction = "Yes"
    │                        │
    ▼ TRUE                   ▼ ELSE
    │                        │
[Call Action:            [End current topic]
  Log Interaction
  to Dataverse]
  ClientName ← Topic.ClientName
  InteractionType ← "Meeting"
  Summary ← Topic.MeetingPurpose
  FollowUpDate ← Topic.MeetingDateTime
    │
    ▼
[Message] "Interaction logged successfully! ✅"
    │
    ▼
[End current topic]
```

### 6b. "Send Follow-Up Email" Topic

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
  Topic.EmailSubject = Concatenate(
    "Follow-up from Contoso - ",
    Topic.ClientName)
    │
    ▼
[Prompt (AI Builder Custom Prompt)]
   Instructions: "Compose the body of a professional
     follow-up email from a Contoso financial advisor
     to their client. Client name: {ClientName}.
     The advisor's key points: {EmailContent}."
   Inputs:
     ClientName  ← Topic.ClientName
     EmailContent ← Topic.EmailContent
   Output: predictionOutput (record)
   Access text via: Topic.predictionOutput.text
    │
    ▼
[Message] "Here's the draft email:
   {Topic.predictionOutput.text}
   Confirm send?"
    │
    ▼
[Ask] "Confirm send?"
  → Yes, send it / Edit and resend / Cancel
  → Topic.EmailConfirm (choice)
    │
    ▼
[Condition] Topic.EmailConfirm = "Yes, send it"
    │                              │
    ▼ TRUE                         ▼ ELSE
    │                              │
[Call Action:                [Message] "No problem.
  Send Follow-Up Email]       The email was not sent."
  ToEmail   ← ClientEmail   [End current topic]
  Subject   ← EmailSubject
  EmailBody ← predictionOutput.text
    │
    ▼
[Message] "✅ Email sent to {Topic.ClientEmail}!"
    │
    ▼
[End current topic]
```

---

## 7. Slot-Filling Behavior (With vs Without)

### With Slot-Filling Enabled (Section 4.2 configured)

```
Advisor: "I need to check the policy for David Chan"
                      │
                      ▼
Hub Agent → Recognizes intent: policy lookup
  → Extracts: ClientName = "David Chan"
  → Transfers to Policy Lookup Agent
  → Pre-fills Topic.ClientName = "David Chan"
                      │
                      ▼
Policy Lookup Agent:
  → Topic.ClientName is already filled
  → SKIPS "What is the client's name?" question
  → Proceeds directly to SharePoint query
  → Returns policy details
```

### Without Slot-Filling (Section 4.2 skipped)

```
Advisor: "I need to check the policy for David Chan"
                      │
                      ▼
Hub Agent → Transfers to Policy Lookup Agent
  → (no pre-filled variables)
                      │
                      ▼
Policy Lookup Agent:
  → "How would you like to search?"       ← RE-ASKS
  → "What is the client's name?"           ← RE-ASKS
  → Then queries SharePoint
```

### Slot-Filling Rules

| Variable Type | Safe to Slot-Fill? | Notes |
|--------------|-------------------|-------|
| **String** (User's entire response, Email) | ✅ Yes | Works in all contexts |
| **Number** | ✅ Yes | Orchestrator extracts numeric values |
| **Choice** (Multiple choice options) — used in messages only | ⚠️ Caution | Displays via {x} but `Text()` may return empty in Concatenate |
| **Choice** — used in `Concatenate(Text(...))` | ❌ No | `Text()` returns empty when slot-filled with free text |
| **Datetime** | ✅ Yes | Orchestrator can parse natural language dates |

---

## 8. Data Model Reference

### SharePoint List: `ClientPolicies`

| Column Name | Type | Example |
|-------------|------|---------|
| ClientName (Title) | Single line of text | David Chan |
| ClientEmail | Single line of text | david.chan@email.com |
| PolicyNumber | Single line of text | CTO-2025-00123 |
| PolicyType | Choice | Life / Health / Travel / Critical Illness |
| CoverageAmount | Currency | 1,000,000 |
| PolicyStatus | Choice | Active / Pending / Expired / Under Review |
| StartDate | Date | 2024-01-15 |
| EndDate | Date | 2034-01-15 |
| AssignedAdvisor | Single line of text | Sarah Wong |
| LastInteraction | Date | 2025-12-01 |
| Notes | Multiple lines of text | Client interested in adding critical illness rider |

### Dataverse Table: `InteractionLog`

| Column | Type | Notes |
|--------|------|-------|
| ClientName | Text | |
| AdvisorName | Text | |
| InteractionDate | Date | |
| InteractionType | Choice | Call (890950000) / Meeting (890950001) / Email (890950002) / Chat (890950003) |
| Summary | Multiline Text | |
| FollowUpRequired | Yes/No | |
| FollowUpDate | Date | |

### SharePoint Document Library: `ProductGuides`

| Document | Used By |
|----------|---------|
| Contoso_Product_FAQ.docx | Hub Agent (Knowledge), Product Advisor Agent (Knowledge) |
| Underwriting_Guidelines.docx | Hub Agent (Knowledge), Product Advisor Agent (Knowledge) |
| Compliance_Policy.docx | Hub Agent (Knowledge) |
