> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Contoso Smart Workspace — Copilot Studio Multi-Agent Hackathon

A hands-on hackathon kit for building a **multi-agent AI system** using **Microsoft Copilot Studio**. Participants learn to create specialized agents that orchestrate together, connect to enterprise data sources, and automate real business workflows — all without writing code.

---

## Business Scenario

Contoso financial advisors need a unified AI assistant that can:

1. **Look up** client policy information from SharePoint
2. **Recommend** insurance products based on client profiles
3. **Schedule** follow-up meetings via Outlook
4. **Send** AI-drafted follow-up emails
5. **Log** interaction summaries to Dataverse

Instead of one monolithic bot, we build **four specialized agents** orchestrated by a primary hub agent.

---

## Architecture

```
                    ┌──────────────────────────┐
                    │   Financial Advisor       │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │  Contoso Advisor Hub      │
                    │  (Orchestrator Agent)     │
                    └──┬─────────┬──────────┬──┘
                       │         │          │
            ┌──────────▼┐  ┌────▼─────┐  ┌─▼────────────┐
            │ Policy     │  │ Product  │  │ Scheduling   │
            │ Lookup     │  │ Advisor  │  │ Assistant    │
            │ Agent      │  │ Agent    │  │ Agent        │
            └──┬─────────┘  └────┬─────┘  └─┬──────┬────┘
               │                 │           │      │
         ┌─────▼─────┐   ┌──────▼────┐  ┌───▼──┐ ┌─▼────────┐
         │ SharePoint │   │ SharePoint│  │Outlook│ │ Dataverse│
         │ List:      │   │ Library:  │  │Email &│ │ Interact.│
         │ ClientPol. │   │ Product   │  │Cal.   │ │ Log      │
         └────────────┘   │ Guides    │  └───────┘ └──────────┘
                          └───────────┘
```

| Agent | Role | Connectors |
|-------|------|------------|
| **Contoso Advisor Hub** | Orchestrator — routes user intent to specialists | SharePoint (Knowledge) |
| **Policy Lookup Agent** | Queries client policy data | SharePoint List via Power Automate |
| **Product Advisor Agent** | Product Q&A, comparisons, and recommendations | SharePoint Document Library (Knowledge) |
| **Scheduling Assistant** | Books meetings, sends emails, logs interactions | Outlook + Dataverse via Power Automate |

---

## Key Copilot Studio Concepts Covered

| Concept | Description |
|---------|-------------|
| **Multi-Agent Orchestration** | Hub agent routes to specialist sub-agents based on intent |
| **Generative Orchestration** | AI-driven intent matching — no manual routing rules needed |
| **Connectors & Actions** | Power Automate flows call SharePoint, Outlook, and Dataverse |
| **Topics & Variables** | Structured conversation flows with data collection |
| **Slot Filling** | Orchestrator pre-fills sub-agent variables from the user's message |
| **Knowledge (RAG)** | Agents answer from uploaded product documents |
| **AI Builder Prompts** | Custom prompt generates professional follow-up emails |

---

## Repository Contents

### Guides

| File | Description |
|------|-------------|
| [End_to_End_Setup_Guide.md](End_to_End_Setup_Guide.md) | Step-by-step build instructions for all 4 agents, flows, and data |
| [Architecture_and_Flow_Diagrams.md](Architecture_and_Flow_Diagrams.md) | Visual diagrams for all agent flows, Power Automate flows, and data models |

### Sample Knowledge Documents

| File | Purpose |
|------|---------|
| Contoso_Product_FAQ.docx | Product FAQ — upload to SharePoint `ProductGuides` library |
| Underwriting_Guidelines.docx | Underwriting rules — upload to SharePoint `ProductGuides` library |
| Compliance_Policy.docx | Compliance policy — upload to SharePoint `ProductGuides` library |

---

## Prerequisites

- Microsoft 365 tenant with **Copilot Studio** license
- SharePoint Online access
- Power Automate access
- Power Apps / Dataverse access
- Outlook (Exchange Online) enabled

---

## Quick Start

1. **Read** [End_to_End_Setup_Guide.md](End_to_End_Setup_Guide.md) — follow the 8 phases in order
2. **Phase 1** — Create SharePoint list, document library, and Dataverse table
3. **Phase 2–5** — Build the 4 agents with their topics, flows, and knowledge
4. **Phase 6** — Connect sub-agents to the hub and enable slot filling
5. **Phase 7** — Test each agent individually, then test end-to-end orchestration
6. **Phase 8** — Publish all agents

---

## Common Gotchas

| Issue | Solution |
|-------|----------|
| SharePoint "ClientName" has internal name `Title` | Use `Title` in filter queries |
| SharePoint Get Items returns an array | Use `first()` in a Compose step |
| Choice columns return objects, not text | Use `?['Value']` to extract the display text |
| `choice` / `datetime` variables in `Concatenate()` | Wrap with `Text()` |
| Flow output name conflicts with topic variable | Create a separate variable (e.g., `FlowConfirmation`) |
| `MeetingPurpose` breaks when slot-filled | Do NOT enable "Receive values from other topics" on it |
| Connector auth expired before demo | Re-authenticate flows in Power Automate before presenting |

---

## Workshop Agenda

| Block | Duration | Content |
|-------|----------|---------|
| Sharing Session | ~60–75 min | Concepts, platform overview, governance |
| Live Demo | ~25 min | End-to-end multi-agent walkthrough |
| Hackathon | Flexible | Participants build their own agents |

---

## License

This project is provided as hackathon/workshop material by Microsoft. All rights reserved.
