> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 6: HR & Talent Intelligence Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that gives HR and executive leadership conversational access to workforce analytics — headcount, attrition, succession planning, and employee engagement data.

---

## Use Case

Manulife HK's CHRO and senior leaders need quick answers about their people: *"What's our attrition rate?", "Who are the succession candidates for Head of Claims?", "How does our headcount compare to budget?"*. This agent:

- **Answers workforce analytics questions** (headcount, attrition, hiring pipeline)
- **Provides succession planning information** with readiness levels
- **Compares actual vs budgeted headcount** by department
- **Surfaces employee engagement trends** and highlights
- **Answers HR policy questions** from knowledge base

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CHRO / Head of HR / CEO)                              │
│  "What's the attrition rate for the tech team?"              │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 HR & Talent Intelligence Agent        │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 HR_Policies_Summary.docx             │               │
│  │  📄 Succession_Plan_2026.docx            │               │
│  │  📊 SharePoint List: WorkforceMetrics     │               │
│  │  📊 SharePoint List: SuccessionPipeline   │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Workforce Dashboard                   │               │
│  │  2. Attrition Analysis                    │               │
│  │  3. Succession Planning Q&A               │               │
│  │  4. HR Policy Q&A (Knowledge)             │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  Power Automate Flows      │                           │
│     └─────────────┬──────────────┘                           │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  SharePoint Lists          │                           │
│     └────────────────────────────┘                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint List: `WorkforceMetrics`

1. **New** → **List** → name it `WorkforceMetrics`
2. Add columns:

| Column Name | Type |
|---|---|
| Department | Single line of text *(rename "Title")* |
| Period | Single line of text |
| Headcount | Number |
| BudgetedHeadcount | Number |
| OpenPositions | Number |
| NewHires | Number |
| Departures | Number |
| VoluntaryAttrition | Number (1 decimal) |
| AvgTenure | Number (1 decimal) |
| EngagementScore | Number |
| TrainingCompletion | Number (1 decimal) |
| DiversityRatio | Number (1 decimal) |
| DepartmentHead | Single line of text |

3. Add sample data:

| Department | Period | Headcount | BudgetedHeadcount | OpenPositions | NewHires | Departures | VoluntaryAttrition | AvgTenure | EngagementScore | TrainingCompletion | DiversityRatio | DepartmentHead |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| IT & Digital | Q1 2026 | 420 | 450 | 30 | 45 | 18 | 12.5 | 3.2 | 74 | 88 | 42.0 | Michael Cheung |
| Operations | Q1 2026 | 580 | 570 | 8 | 22 | 15 | 7.8 | 5.1 | 79 | 92 | 55.0 | Sarah Lam |
| Finance | Q1 2026 | 180 | 185 | 5 | 8 | 3 | 5.2 | 6.3 | 82 | 95 | 48.0 | David Wong |
| Legal & Compliance | Q1 2026 | 95 | 100 | 5 | 6 | 2 | 6.1 | 5.8 | 80 | 98 | 52.0 | James Li |
| Human Resources | Q1 2026 | 65 | 65 | 0 | 3 | 1 | 4.5 | 4.9 | 85 | 100 | 62.0 | Karen Ho |
| Marketing | Q1 2026 | 120 | 125 | 5 | 10 | 4 | 9.2 | 3.5 | 76 | 90 | 58.0 | Grace Yip |
| Sales & Distribution | Q1 2026 | 280 | 275 | 12 | 35 | 20 | 11.8 | 3.8 | 72 | 82 | 38.0 | Raymond Kwok |
| Risk Management | Q1 2026 | 85 | 90 | 5 | 5 | 2 | 6.8 | 5.5 | 81 | 94 | 45.0 | Vivian Fong |
| Customer Service | Q1 2026 | 350 | 360 | 10 | 28 | 22 | 15.2 | 2.1 | 68 | 85 | 60.0 | Derek Ng |
| Actuarial | Q1 2026 | 45 | 45 | 0 | 2 | 1 | 6.5 | 7.2 | 83 | 96 | 40.0 | Samantha Yu |
| Claims | Q1 2026 | 280 | 285 | 5 | 15 | 12 | 10.5 | 3.4 | 71 | 87 | 53.0 | (Vacant - see succession) |
| Investment | Q1 2026 | 60 | 60 | 0 | 3 | 2 | 9.8 | 4.5 | 77 | 91 | 35.0 | Andrew Chow |

### 1.2 Create SharePoint List: `SuccessionPipeline`

1. **New** → **List** → name it `SuccessionPipeline`
2. Add columns:

| Column Name | Type |
|---|---|
| TargetRole | Single line of text *(rename "Title")* |
| CurrentIncumbent | Single line of text |
| IncumbentStatus | Choice: `Active`, `Retiring`, `Departing`, `Vacant` |
| CandidateName | Single line of text |
| CandidateCurrentRole | Single line of text |
| ReadinessLevel | Choice: `Ready Now`, `Ready in 1 Year`, `Ready in 2-3 Years`, `Development Needed` |
| StrengthAreas | Multiple lines of text |
| DevelopmentAreas | Multiple lines of text |
| DevelopmentPlan | Multiple lines of text |
| RiskIfNotFilled | Choice: `Critical`, `High`, `Medium`, `Low` |

3. Add sample data:

| TargetRole | CurrentIncumbent | IncumbentStatus | CandidateName | CandidateCurrentRole | ReadinessLevel | StrengthAreas | DevelopmentAreas | DevelopmentPlan | RiskIfNotFilled |
|---|---|---|---|---|---|---|---|---|---|
| Head of Claims | (Vacant) | Vacant | Emily Tsang | Deputy Head of Claims | Ready Now | 15 years claims experience, strong team leadership, excellent stakeholder management | Strategic planning, board-level presentation | Executive coaching program, board presentation workshop Q2 2026 | Critical |
| Head of Claims | (Vacant) | Vacant | Daniel Mok | Senior Claims Manager | Ready in 1 Year | Deep technical expertise, process improvement track record | People management at scale, cross-functional collaboration | Leadership development program, rotation to Operations for 6 months | Critical |
| CFO | David Wong | Active | Michelle Tam | VP Finance | Ready in 2-3 Years | Strong financial acumen, regulatory reporting expertise | M&A experience, investor relations, regional exposure | Regional finance rotation, M&A project lead assignment | High |
| COO | Sarah Lam | Active | Kevin Ng | Head of Operations Excellence | Ready in 1 Year | Operational transformation, digital adoption champion | P&L ownership, regulatory interaction | Assign P&L for Digital channel, IA liaison for operations audits | High |
| CIO | Michael Cheung | Active | Jennifer Wu | Head of IT Architecture | Ready in 1 Year | Cloud migration leader, cybersecurity expertise | Vendor management, budget ownership, business partnership | CIO shadow program, lead next vendor RFP, present to Board | High |
| Head of Distribution | Raymond Kwok | Retiring | Angela Fung | District Director - Kowloon East | Ready Now | Top district performance, agency recruitment excellence, MDRT culture | Head office politics, cross-functional alignment | Executive leadership program, rotate through product and marketing | Critical |
| Head of Distribution | Raymond Kwok | Retiring | Patrick Lam | District Director - NT West | Ready in 1 Year | Strong persistency record, advisor development focus | Scale — only managed NT West, digital channel understanding | Assign acting Head of Distribution for Q3, digital strategy workshop | Critical |
| CHRO | Karen Ho | Active | Diana Lee | Head of Talent & OD | Ready in 2-3 Years | Talent program design, engagement survey champion | Compensation & benefits, labor relations, regional HR | C&B project lead, regional HR summit participation | Medium |
| CMO | Grace Yip | Active | Chris Wong | Head of Digital Marketing | Ready in 1 Year | Digital transformation, data-driven marketing, brand campaigns | Traditional channels, agency marketing support, budget management | Marketing strategy project lead, full channel ownership for H2 2026 | Medium |
| Head of Risk | Vivian Fong | Active | Timothy Chan | Senior Risk Manager | Ready in 2-3 Years | Risk modelling, ERM framework design | Board communication, regulatory engagement | Executive communication course, IA inspection lead | Medium |

### 1.3 Create Knowledge Documents

#### `HR_Policies_Summary.docx`

```
MANULIFE HONG KONG — HR POLICIES SUMMARY

1. LEAVE POLICIES
- Annual Leave: 15-25 days based on grade level (15 days for new joiners, 
  +1 day per year of service, max 25 days)
- Sick Leave: 14 days paid per year (accumulative up to 120 days)
- Maternity Leave: 14 weeks paid (statutory requirement)
- Paternity Leave: 5 days paid
- Compassionate Leave: 3-5 days depending on relationship
- Marriage Leave: 5 days
- Study Leave: Up to 5 days per year for approved qualifications
- Sabbatical: Available after 10 years of service — up to 3 months unpaid

2. PERFORMANCE MANAGEMENT
- Annual performance review cycle: January — March
- Mid-year check-in: July
- Rating scale: 1 (Below Expectations) to 5 (Exceptional)
- Forced ranking applied at department level
- PIP (Performance Improvement Plan): Triggered by rating of 1 or 2 for 
  two consecutive periods — duration 3 months

3. COMPENSATION & BENEFITS
- Annual salary review: April (effective date)
- Bonus: 0-4 months based on company and individual performance
- MPF employer contribution: 5% (above statutory minimum for senior grades)
- Medical insurance: Group plan covering employee + dependents
- Life insurance: 3x annual salary
- Dental: HKD 5,000 per year per employee
- Flexible benefits: HKD 8,000 annual wellness allowance

4. LEARNING & DEVELOPMENT
- Mandatory training hours: 40 hours per year
- Company-sponsored education: Up to HKD 30,000 per year for approved 
  programs
- Internal mobility: Employees can apply for internal positions after 
  12 months in current role
- Mentoring program: Available for high-potential employees (Grade E and above)

5. DIVERSITY & INCLUSION
- Target: 45% female representation in leadership (Grade D and above) by 2027
- Current: 42% (as of Q1 2026)
- ERG (Employee Resource Groups): 5 active groups
- Pay equity review: Conducted annually, last review December 2025

6. RESIGNATION & EXIT
- Notice period: 1 month (standard), 3 months (Grade D and above)
- Exit interview: Mandatory, conducted by HR within last 2 weeks
- Garden leave: At discretion of management for sensitive roles
- Non-compete: 6 months for Grade D and above (Hong Kong jurisdiction)
```

#### `Succession_Plan_2026.docx`

```
MANULIFE HONG KONG — SUCCESSION PLANNING FRAMEWORK 2026

1. PURPOSE
Ensure leadership continuity and minimise business disruption by 
maintaining a pipeline of ready-now and developing successors for 
all critical leadership roles.

2. CRITICAL ROLES (IMMEDIATE ATTENTION)
- Head of Claims: VACANT — needs immediate appointment. Two internal 
  candidates identified. Emily Tsang (Deputy Head) is Ready Now. 
  External search also initiated as backup.
- Head of Distribution: Raymond Kwok retiring end of 2026. Angela Fung 
  and Patrick Lam are lead candidates. Transition plan needed by Q3.

3. READINESS DEFINITIONS
- Ready Now: Can step into the role within 0-6 months with minimal 
  onboarding. Has the experience, skills, and leadership capability.
- Ready in 1 Year: Strong candidate who needs targeted development in 
  1-2 specific areas. Could serve as acting/interim.
- Ready in 2-3 Years: High-potential talent with significant development 
  remaining. Good long-term pipeline candidate.
- Development Needed: Has potential but requires substantial development 
  across multiple dimensions.

4. SUCCESSION HEALTH METRICS
- Roles with Ready Now successor: 40% (target: 60%)
- Roles with at least one successor identified: 90% (target: 100%)
- Average successors per critical role: 1.8 (target: 2.0)
- Diversity of succession pipeline: 35% female (target: 45%)

5. KEY ACTIONS FOR 2026
- Q2: Appoint Head of Claims (internal or external)
- Q2: Finalise distribution leadership transition plan
- Q3: Launch executive development program for "Ready in 1 Year" candidates
- Q4: Board succession review — present updated pipeline to Board HR Committee
```

Upload both to a `HRDocs` document library on SharePoint.

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"HR & Talent Intelligence Agent"**
3. Description: *"Provides HR and executive leadership with conversational access to workforce analytics, succession planning, and HR policy information for Manulife HK."*

### 2.2 Set Instructions

```
You are the HR & Talent Intelligence Agent for Manulife Hong Kong. You help the CHRO, department heads, and senior executives access workforce data and HR information.

Your role:
- Answer workforce analytics questions: headcount, attrition, hiring pipeline, engagement
- Provide succession planning information including candidate readiness and development plans
- Compare workforce metrics across departments
- Answer HR policy questions from the knowledge base
- Highlight areas of concern: high attrition, low engagement, vacant critical roles, unfilled positions

Rules:
- Present workforce data in structured tables with department, metric, and comparison to budget/target
- For succession planning, always include: target role, candidates, readiness level, and development needs
- Flag departments with voluntary attrition above 10% as requiring attention
- Flag departments with engagement score below 70 as requiring intervention
- When discussing individual employee data, remind the user that this information is confidential
- NEVER disclose compensation details of specific individuals
- Present attrition data with context: industry benchmark for HK insurance is ~10-12%
- Include headcount variance (actual vs budget) when discussing staffing
- Use a professional, data-driven tone
- If asked about something outside the data, direct to the HR team
```

### 2.3 Add Knowledge Sources

Upload or connect:
- `HR_Policies_Summary.docx`
- `Succession_Plan_2026.docx`

---

## Phase 3: Create Power Automate Flows

### 3.1 Flow: "Get Workforce Metrics"

1. **Trigger input**: `Department` (Text)
2. **SharePoint — Get items**:
   - List: `WorkforceMetrics`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Title eq ''', triggerBody()?['text'], ''''))
     ```
3. **Select**:
   - Dept: `item()?['Title']`
   - Headcount: `item()?['Headcount']`
   - Budget: `item()?['BudgetedHeadcount']`
   - Open: `item()?['OpenPositions']`
   - Hires: `item()?['NewHires']`
   - Departures: `item()?['Departures']`
   - Attrition: `item()?['VoluntaryAttrition']`
   - Engagement: `item()?['EngagementScore']`
   - Training: `item()?['TrainingCompletion']`
   - Head: `item()?['DepartmentHead']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `WorkforceData` (Text)
6. Name: `Get Workforce Metrics` → **Save**

### 3.2 Flow: "Get Succession Pipeline"

1. **Trigger input**: `TargetRole` (Text)
2. **SharePoint — Get items**:
   - List: `SuccessionPipeline`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Title eq ''', triggerBody()?['text'], ''''))
     ```
3. **Select**:
   - Role: `item()?['Title']`
   - Incumbent: `item()?['CurrentIncumbent']`
   - IncumbentStatus: `item()?['IncumbentStatus']?['Value']`
   - Candidate: `item()?['CandidateName']`
   - CurrentRole: `item()?['CandidateCurrentRole']`
   - Readiness: `item()?['ReadinessLevel']?['Value']`
   - Strengths: `item()?['StrengthAreas']`
   - DevAreas: `item()?['DevelopmentAreas']`
   - DevPlan: `item()?['DevelopmentPlan']`
   - Risk: `item()?['RiskIfNotFilled']?['Value']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `SuccessionData` (Text)
6. Name: `Get Succession Pipeline` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Workforce Dashboard"

**Trigger phrases**: `Workforce overview`, `Headcount summary`, `How many staff do we have`, `Staffing levels`, `HR dashboard`, `People metrics`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "Which department? Or 'All' for the full company." → Multiple choice: All / IT & Digital / Operations / Finance / Sales & Distribution / Customer Service / Claims / (other departments) | `Topic.Department` |
| 2 | Call Action: **Get Workforce Metrics** (Department ← Topic.Department) | `Topic.WorkforceData` |
| 3 | **Generative answers** — prompt: `Present the following workforce data as a department comparison table. Highlight: (1) total headcount vs budget (over/under), (2) departments with attrition >10%, (3) departments with engagement <70, (4) departments with most open positions. If showing all departments, provide a company total summary first. Data: {Topic.WorkforceData}` | — |
| 4 | End topic | — |

### 4.2 Topic: "Attrition Analysis"

**Trigger phrases**: `Attrition rate`, `Who's leaving`, `Turnover`, `Which department has highest attrition`, `Retention`, `Staff leaving`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get Workforce Metrics** (Department = "All") | `Topic.AllData` |
| 2 | **Generative answers** — prompt: `Analyse the following workforce data for attrition patterns. Present: (1) Company-wide voluntary attrition rate (weighted average), (2) Departments ranked by attrition — highest to lowest, (3) Flag departments above the 10% industry benchmark, (4) Correlation between engagement score and attrition, (5) Recommendations for high-attrition departments. Data: {Topic.AllData}` | — |
| 3 | End topic | — |

### 4.3 Topic: "Succession Planning Q&A"

**Trigger phrases**: `Succession plan`, `Who can replace`, `Succession candidates`, `Who's next in line for`, `Critical role vacancies`, `Leadership pipeline`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "Which role are you interested in? Or 'All' for the full pipeline." → User's entire response | `Topic.TargetRole` |
| 2 | Call Action: **Get Succession Pipeline** (TargetRole ← Topic.TargetRole) | `Topic.SuccessionData` |
| 3 | **Generative answers** — prompt: `Present the following succession pipeline data. For each target role, show: role, incumbent status, candidates with readiness level, key development areas, and risk if not filled. Group by risk level (Critical first). If showing all roles, provide a summary: total critical roles, roles with Ready Now successor, pipeline health score. Data: {Topic.SuccessionData}` | — |
| 4 | Message: "⚠️ *Succession data is confidential. Please handle appropriately.*" | — |
| 5 | End topic | — |

### 4.4 Topic: "HR Policy Q&A"

This uses generative answers from knowledge only — no flow needed.

**Trigger phrases**: `HR policy`, `Leave policy`, `How many days annual leave`, `Performance review`, `Benefits`, `Notice period`, `Salary review`

1. **Generative answers** node linked to `HR_Policies_Summary.docx`
2. End topic

---

## Phase 5: Testing

| Test Query | Expected Response |
|---|---|
| "What's the overall headcount?" | Company summary: ~2,560 staff across 12 departments |
| "Which department has the highest attrition?" | "Customer Service at 15.2% — well above industry benchmark" |
| "Who are the succession candidates for Head of Claims?" | Emily Tsang (Ready Now) and Daniel Mok (Ready in 1 Year) with development plans |
| "What's the engagement score for IT?" | "74 — slightly above company average but room for improvement" |
| "How many annual leave days do I get?" | "15-25 days based on grade, +1 per year of service" |
| "Which roles are vacant and critical?" | Head of Claims — vacant, Critical risk |

### Demo Script (8 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "Give me a workforce overview" | "The CHRO needs people data at their fingertips" |
| 1:30 | Show department comparison table | "Instant view of headcount vs budget, attrition, engagement" |
| 2:30 | "Which department has the highest attrition?" | "Identify problem areas before they become crises" |
| 3:30 | Show attrition analysis with recommendations | "Customer Service at 15.2% — flagged immediately" |
| 4:30 | "Who can replace the Head of Claims?" | "Succession planning questions answered instantly" |
| 5:30 | Show succession candidates with readiness | "Two candidates identified with specific development plans" |
| 6:30 | "What's our annual leave policy?" | "HR policies accessible too — one agent for all HR questions" |
| 7:30 | Show policy answer | "Reduces routine HR queries to the HR team by 40-60%" |
