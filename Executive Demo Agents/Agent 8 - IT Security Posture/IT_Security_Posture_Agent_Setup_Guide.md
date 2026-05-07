> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 8: IT Service & Security Posture Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that gives CIO/CISO executives a conversational view of IT service health, incident trends, project status, and security posture.

---

## Use Case

Manulife HK's CIO and CISO need quick situational awareness: *"How many P1 incidents this month?", "What's the cloud migration status?", "Are there any open security vulnerabilities?"*. This agent:

- **Summarises IT service health** and incident trends
- **Answers questions** about ongoing IT projects and initiatives
- **Provides security posture overview** (vulnerabilities, compliance, threat status)
- **Surfaces vendor risk assessments** and contract deadlines

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CIO / CISO / CEO)                                     │
│  "How many P1 incidents did we have this month?"             │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 IT Service & Security Posture Agent   │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 IT_Status_Report_Apr2026.docx        │               │
│  │  📄 Security_Posture_Report.docx         │               │
│  │  📊 SharePoint List: ITIncidents          │               │
│  │  📊 SharePoint List: ITProjects           │               │
│  │  📊 SharePoint List: SecurityMetrics      │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. IT Service Health                     │               │
│  │  2. Incident Summary                      │               │
│  │  3. Project Status                        │               │
│  │  4. Security Posture                      │               │
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

### 1.1 Create SharePoint List: `ITIncidents`

1. **New** → **List** → name it `ITIncidents`
2. Add columns:

| Column Name | Type |
|---|---|
| IncidentID | Single line of text *(rename "Title")* |
| Priority | Choice: `P1 - Critical`, `P2 - High`, `P3 - Medium`, `P4 - Low` |
| Category | Choice: `Infrastructure`, `Application`, `Security`, `Network`, `Database`, `Cloud`, `End User` |
| AffectedSystem | Single line of text |
| DateReported | Date |
| DateResolved | Date |
| Status | Choice: `Open`, `In Progress`, `Resolved`, `Closed` |
| MTTR_Hours | Number (1 decimal) |
| RootCause | Multiple lines of text |
| Impact | Multiple lines of text |
| AssignedTeam | Single line of text |

3. Add sample data:

| IncidentID | Priority | Category | AffectedSystem | DateReported | DateResolved | Status | MTTR_Hours | RootCause | Impact | AssignedTeam |
|---|---|---|---|---|---|---|---|---|---|---|
| INC-2026-0401 | P1 - Critical | Application | Policy Administration System | 2026-04-02 | 2026-04-02 | Closed | 3.5 | Memory leak in batch processing module triggered by quarterly batch run. Hotfix deployed. | Policy issuance delayed 3 hours. 45 policies affected. No data loss. | Application Support |
| INC-2026-0412 | P1 - Critical | Infrastructure | Primary Data Centre UPS | 2026-04-12 | 2026-04-12 | Closed | 1.2 | UPS unit 3 failed during routine power test. Backup power activated within 15 seconds. | 15-second disruption to non-critical systems. Core systems on redundant power unaffected. | Infrastructure Ops |
| INC-2026-0418 | P2 - High | Security | VPN Gateway | 2026-04-18 | 2026-04-19 | Closed | 8.0 | Anomalous traffic pattern detected on VPN. Investigation confirmed false positive triggered by legitimate bulk data transfer from compliance team. | VPN access restricted for 2 hours during investigation. 120 remote staff affected. | Cybersecurity |
| INC-2026-0422 | P2 - High | Cloud | Azure AD Connect | 2026-04-22 | 2026-04-22 | Closed | 2.5 | Azure AD Connect sync failure due to expired service account password. Password updated and sync restored. | New employee provisioning delayed for 4 hours. 8 new joiners affected. | Cloud Operations |
| INC-2026-0425 | P3 - Medium | Application | Customer Portal | 2026-04-25 | 2026-04-26 | Closed | 12.0 | Slow response times on customer portal during peak hours. Database query optimisation required. | Degraded performance for 2,000+ customers. No outage. | Application Support |
| INC-2026-0428 | P2 - High | Network | Branch Office - Kwun Tong | 2026-04-28 | 2026-04-29 | Closed | 6.0 | ISP circuit failure at Kwun Tong branch. Failover to backup circuit took 45 minutes (expected: 5 minutes). Failover configuration error identified and corrected. | Kwun Tong branch offline for 45 minutes. 50 staff switched to mobile hotspots. | Network Operations |
| INC-2026-0501 | P3 - Medium | End User | Email System | 2026-05-01 | 2026-05-01 | Closed | 1.5 | Outlook calendar sync issue affecting 30 users. Resolved by clearing cache and re-syncing. | 30 users unable to see calendar appointments for 90 minutes. | End User Support |
| INC-2026-0503 | P1 - Critical | Database | Claims Database | 2026-05-03 | — | In Progress | — | Claims database experiencing intermittent connection drops. Preliminary investigation suggests storage I/O bottleneck. DBA team investigating. | Claims processing slowed by 40%. Manual workaround in place. | Database Admin |
| INC-2026-0505 | P4 - Low | Application | Internal HR Portal | 2026-05-05 | — | Open | — | Leave balance display showing incorrect carry-forward amounts for 15% of staff. Data issue from year-end processing. | HR fielding manual inquiries. No financial impact. | Application Support |

### 1.2 Create SharePoint List: `ITProjects`

1. **New** → **List** → name it `ITProjects`
2. Add columns:

| Column Name | Type |
|---|---|
| ProjectName | Single line of text *(rename "Title")* |
| Owner | Single line of text |
| Status | Choice: `On Track`, `Minor Delay`, `At Risk`, `Completed`, `On Hold` |
| Phase | Choice: `Planning`, `In Progress`, `Testing`, `Deployment`, `Post-Go-Live`, `Completed` |
| TargetDate | Date |
| Budget_M | Number (1 decimal) |
| Spent_M | Number (1 decimal) |
| Progress | Number |
| KeyRisks | Multiple lines of text |
| NextMilestone | Single line of text |
| NextMilestoneDate | Date |

3. Add sample data:

| ProjectName | Owner | Status | Phase | TargetDate | Budget_M | Spent_M | Progress | KeyRisks | NextMilestone | NextMilestoneDate |
|---|---|---|---|---|---|---|---|---|---|---|
| Cloud Migration Program | CIO | On Track | In Progress | 2026-12-31 | 25.0 | 14.5 | 72 | Legacy system dependencies delaying 3 workloads. Vendor support for SAP migration needs escalation. | Migrate CRM to Azure | 2026-06-15 |
| Digital Self-Service Portal Phase 2 | CIO | On Track | Testing | 2026-05-31 | 12.0 | 9.5 | 88 | UAT completion pending claims team availability. Performance testing shows 2-second response under load. | UAT Sign-off | 2026-05-15 |
| AI-Powered Underwriting | COO | Minor Delay | Testing | 2026-09-30 | 8.0 | 5.2 | 65 | Model validation by actuarial delayed 3 weeks. Regulatory approval for auto-decisions pending IA feedback. | Actuarial model sign-off | 2026-06-30 |
| Cybersecurity Enhancement | CISO | Minor Delay | In Progress | 2026-12-31 | 18.0 | 7.2 | 40 | Vendor contract for SIEM upgrade delayed 3 weeks. Zero trust pilot expanding slower than planned. | SIEM deployment | 2026-07-31 |
| ILAS Voice Recording System | CIO | On Track | Deployment | 2026-06-30 | 3.5 | 2.8 | 78 | Hardware delivery for 3 remaining branches delayed. Training completion at 85%. | Full branch rollout | 2026-06-15 |
| Data Warehouse Modernisation | CIO | At Risk | In Progress | 2026-09-30 | 6.0 | 3.8 | 45 | Data quality issues in legacy systems requiring significant cleansing. Resource conflict with cloud migration team. | Data cleansing completion | 2026-07-15 |
| Employee Experience Platform | CHRO | On Track | Planning | 2027-03-31 | 4.0 | 0.5 | 15 | Vendor shortlisting in progress. Integration with existing HRIS needs scoping. | Vendor selection | 2026-06-30 |
| Open API Platform | CIO | On Track | Planning | 2027-06-30 | 5.0 | 0.8 | 10 | Regulatory framework still under consultation. Security architecture design pending CISO review. | Security design review | 2026-08-31 |

### 1.3 Create SharePoint List: `SecurityMetrics`

1. **New** → **List** → name it `SecurityMetrics`
2. Add columns:

| Column Name | Type |
|---|---|
| MetricName | Single line of text *(rename "Title")* |
| Period | Single line of text |
| CurrentValue | Single line of text |
| Target | Single line of text |
| Status | Choice: `Green`, `Amber`, `Red` |
| Category | Choice: `Vulnerability`, `Access`, `Compliance`, `Threat`, `Awareness`, `Vendor` |
| Details | Multiple lines of text |
| Owner | Single line of text |

3. Add sample data:

| MetricName | Period | CurrentValue | Target | Status | Category | Details | Owner |
|---|---|---|---|---|---|---|---|
| Critical Vulnerabilities | Apr 2026 | 2 open | 0 | Red | Vulnerability | 2 critical CVEs in legacy middleware (WebLogic). Patches available but require maintenance window. Scheduled for May 10. | Vulnerability Mgmt |
| High Vulnerabilities | Apr 2026 | 12 open | <10 | Amber | Vulnerability | 12 high-severity vulnerabilities across 8 systems. 5 being remediated, 7 in assessment. | Vulnerability Mgmt |
| Patch Compliance Rate | Apr 2026 | 94% | 98% | Amber | Vulnerability | 6% of servers not patched within 30-day SLA. Primarily legacy systems awaiting maintenance windows. | Vulnerability Mgmt |
| Phishing Simulation Click Rate | Apr 2026 | 4.2% | <5% | Green | Awareness | April campaign: 4.2% click rate (industry avg 8-12%). Improvement from 6.1% in Jan 2026. | Security Awareness |
| Security Training Completion | Apr 2026 | 91% | 95% | Amber | Awareness | Annual security awareness training. 9% incomplete — mostly new joiners within grace period. | Security Awareness |
| MFA Adoption | Apr 2026 | 99.2% | 100% | Green | Access | 0.8% accounts without MFA — legacy service accounts being migrated. | Identity & Access |
| Privileged Access Reviews | Apr 2026 | Completed | Quarterly | Green | Access | Q1 review completed. 23 accounts recertified, 5 revoked, 2 flagged for review. | Identity & Access |
| P1 Security Incidents | Apr 2026 | 0 | 0 | Green | Threat | Zero P1 security incidents in April. VPN anomaly (INC-0418) was false positive. | SOC |
| Average Threat Detection Time | Apr 2026 | 12 min | <15 min | Green | Threat | SIEM detecting threats within 12 minutes on average. Target met consistently. | SOC |
| SOC Alert Volume | Apr 2026 | 3,450 | N/A | Green | Threat | 3,450 alerts processed. 12 escalated for investigation. 0 confirmed incidents. False positive rate: 99.7%. | SOC |
| DLP Violations | Apr 2026 | 8 | <15 | Green | Compliance | 8 DLP policy violations. All unintentional (misdirected emails). Staff counselled. | Data Protection |
| Vendor Risk Assessments | Apr 2026 | 85% complete | 100% | Amber | Vendor | 15% of critical vendors pending annual reassessment. 3 vendors flagged for elevated risk. | Vendor Risk |
| Cloud Security Score | Apr 2026 | 82/100 | 85/100 | Amber | Compliance | Microsoft Secure Score 82/100. Key gaps: conditional access policies (3 pending), storage encryption (2 accounts). | Cloud Security |
| Backup & DR Test | Apr 2026 | Last: Mar 2026 | Quarterly | Green | Compliance | Q1 DR test successful. RTO: 4 hours (target: 6 hours). RPO: 15 minutes (target: 1 hour). | Infrastructure |

### 1.4 Create Knowledge Documents

#### `IT_Status_Report_Apr2026.docx`

```
MANULIFE HONG KONG — IT MONTHLY STATUS REPORT
APRIL 2026

EXECUTIVE SUMMARY
April was a stable month for IT operations with two P1 incidents, both 
resolved within SLA. Cloud migration reached 72% completion. Digital 
portal Phase 2 is in UAT with May 31 launch on track. Cybersecurity 
posture remains strong with zero confirmed security incidents.

SERVICE HEALTH DASHBOARD

| Service | Availability | SLA | Status |
|---------|-------------|-----|--------|
| Policy Admin System | 99.92% | 99.9% | Green |
| Claims Processing | 99.85% | 99.9% | Amber (INC-0503 ongoing) |
| Customer Portal | 99.95% | 99.5% | Green |
| Email & Collaboration | 100% | 99.9% | Green |
| Network (HQ) | 99.99% | 99.9% | Green |
| Network (Branches) | 99.80% | 99.5% | Green |
| VPN / Remote Access | 99.90% | 99.5% | Green |

INCIDENT SUMMARY
- Total incidents: 9
- P1 (Critical): 2 — both resolved
- P2 (High): 3 — all resolved
- P3 (Medium): 2 — all resolved
- P4 (Low): 2 — 1 open
- Average MTTR (P1): 2.35 hours (SLA: 4 hours)
- Average MTTR (P2): 5.5 hours (SLA: 8 hours)

CHANGE MANAGEMENT
- Changes deployed: 28
- Emergency changes: 2 (both related to P1 incidents)
- Failed changes: 0
- Change success rate: 100%

IT BUDGET STATUS
- Annual budget: HKD 85M
- YTD spend: HKD 28.5M (33.5% — on track for 33.3% at end of April)
- Key overspend areas: None
- Underspend: Training budget (12% below plan — scheduling issue, catching up in Q2)

TEAM METRICS
- IT headcount: 420
- Open positions: 30 (7.1% vacancy rate)
- Staff satisfaction: 74/100
- Training hours completed: 4,200 (target: 4,800)
```

#### `Security_Posture_Report.docx`

```
MANULIFE HONG KONG — CYBERSECURITY POSTURE REPORT
APRIL 2026

1. THREAT LANDSCAPE SUMMARY

The Hong Kong financial sector continues to face elevated cyber threats:
- Ransomware targeting financial services increased 23% globally in Q1 2026
- Phishing campaigns specifically targeting insurance industry advisors
- Supply chain attacks via third-party software vendors
- AI-generated deepfake attempts in social engineering attacks

Manulife HK has NOT been directly targeted in any confirmed attack in Q1/Q2 2026.

2. SECURITY POSTURE SCORECARD

| Area | Score | Rating | Trend |
|------|-------|--------|-------|
| Vulnerability Management | 7/10 | Amber | Improving |
| Identity & Access | 9/10 | Green | Stable |
| Threat Detection | 9/10 | Green | Improving |
| Data Protection | 8/10 | Green | Stable |
| Security Awareness | 8/10 | Green | Improving |
| Cloud Security | 7.5/10 | Amber | Improving |
| Vendor Risk | 7/10 | Amber | Stable |
| Incident Response | 9/10 | Green | Stable |
| Overall | 8.1/10 | Green | Improving |

3. KEY RISKS

Risk 1: Legacy middleware vulnerabilities (WebLogic)
- 2 critical CVEs remain unpatched
- Maintenance window scheduled May 10
- Temporary compensating controls in place (WAF rules, network segmentation)
- Risk owner: CIO

Risk 2: Vendor risk management gaps
- 15% of critical vendors pending annual reassessment
- 3 vendors flagged for elevated risk (data handling practices)
- Action: Complete reassessments by June 30
- Risk owner: CISO

Risk 3: Cloud security configuration gaps
- Microsoft Secure Score 82/100 (target: 85)
- Gaps in conditional access policies and storage encryption
- Action: Remediation plan in progress — target 85+ by Q3
- Risk owner: Cloud Security Lead

4. SECURITY INVESTMENT STATUS

| Initiative | Budget | Spent | Status |
|-----------|--------|-------|--------|
| Zero Trust Architecture | HKD 8M | HKD 3.2M | In Progress |
| SIEM Upgrade (Sentinel) | HKD 4M | HKD 1.5M | Delayed 3 weeks |
| Employee Security Training | HKD 1.5M | HKD 0.8M | On Track |
| Penetration Testing (annual) | HKD 0.8M | HKD 0.8M | Completed |
| Cloud Security Posture Mgmt | HKD 2M | HKD 1.2M | On Track |
| Total Cyber Program | HKD 18M | HKD 7.2M | Minor Delay |

5. COMPLIANCE STATUS

| Framework | Status | Last Audit | Next Review |
|-----------|--------|------------|-------------|
| ISO 27001 | Certified | Dec 2025 | Dec 2026 |
| SOC 2 Type II | Compliant | Mar 2026 | Mar 2027 |
| IA Technology Risk Guidelines | Compliant | Jan 2026 | Jul 2026 |
| PDPO (Data Privacy) | Compliant | Feb 2026 | Aug 2026 |
| PCI DSS | Compliant | Nov 2025 | Nov 2026 |

6. RECOMMENDATIONS FOR EXECUTIVE ATTENTION

1. Approve maintenance window for WebLogic patching (May 10) — brief service disruption expected
2. Escalate vendor risk reassessments — 3 vendors pose data handling concerns
3. Increase cloud security investment by HKD 0.5M to close Secure Score gap
4. Review SIEM upgrade timeline — vendor delay may impact threat detection capability
5. Approve CISO's request for additional 2 SOC analysts for 24/7 coverage
```

Upload both documents to a `ITDocs` library on SharePoint.

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"IT Service & Security Posture Agent"**
3. Description: *"Provides CIO and CISO with conversational access to IT service health, incident trends, project status, and cybersecurity posture for Manulife HK."*

### 2.2 Set Instructions

```
You are the IT Service & Security Posture Agent for Manulife Hong Kong. You help the CIO, CISO, and executive leadership understand IT operations and security status.

Your role:
- Provide IT service health summaries including system availability and SLA performance
- Summarise incident trends by priority, category, and resolution time
- Report on IT project status, budgets, and timelines
- Present cybersecurity posture including vulnerability status, threat detection, and compliance
- Surface vendor risk assessments and contract deadlines
- Highlight items requiring executive attention or decisions

Rules:
- Always present data with RAG (Red/Amber/Green) status indicators where applicable
- For incidents, always include: total count by priority, MTTR vs SLA, open incidents
- For projects, always include: status, progress %, budget vs spent, key risks, next milestone
- For security, always include: overall score, open vulnerabilities, threat status, compliance
- Flag any P1 incidents that are still open or exceeded MTTR SLA
- Flag any projects that are "At Risk" or have budget overruns
- Flag any security metrics in Red status
- Use technical terminology appropriate for CIO/CISO audience
- Do not disclose specific vulnerability details (CVE numbers) outside IT leadership conversations
- If asked about remediation timelines, always include the risk of delay
- Present budget data in HKD millions
```

### 2.3 Add Knowledge Sources

Upload or connect:
- `IT_Status_Report_Apr2026.docx`
- `Security_Posture_Report.docx`

---

## Phase 3: Create Power Automate Flows

### 3.1 Flow: "Get IT Incidents"

1. **Trigger input**: `PriorityFilter` (Text)
2. **SharePoint — Get items**:
   - List: `ITIncidents`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Priority eq ''', triggerBody()?['text'], ''''))
     ```
   - Sort By: `DateReported`
   - Sort Order: Descending
3. **Select**:
   - ID: `item()?['Title']`
   - Priority: `item()?['Priority']?['Value']`
   - Category: `item()?['Category']?['Value']`
   - System: `item()?['AffectedSystem']`
   - Reported: `item()?['DateReported']`
   - Status: `item()?['Status']?['Value']`
   - MTTR: `item()?['MTTR_Hours']`
   - Impact: `item()?['Impact']`
   - Team: `item()?['AssignedTeam']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `IncidentData` (Text)
6. Name: `Get IT Incidents` → **Save**

### 3.2 Flow: "Get IT Projects"

1. **Trigger input**: `StatusFilter` (Text)
2. **SharePoint — Get items**:
   - List: `ITProjects`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Status eq ''', triggerBody()?['text'], ''''))
     ```
   - Sort By: `Status`
3. **Select**:
   - Project: `item()?['Title']`
   - Owner: `item()?['Owner']`
   - Status: `item()?['Status']?['Value']`
   - Progress: `item()?['Progress']`
   - Budget: `item()?['Budget_M']`
   - Spent: `item()?['Spent_M']`
   - Target: `item()?['TargetDate']`
   - Risks: `item()?['KeyRisks']`
   - NextMilestone: `item()?['NextMilestone']`
   - MilestoneDate: `item()?['NextMilestoneDate']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `ProjectData` (Text)
6. Name: `Get IT Projects` → **Save**

### 3.3 Flow: "Get Security Metrics"

1. **Trigger input**: `Category` (Text)
2. **SharePoint — Get items**:
   - List: `SecurityMetrics`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Category eq ''', triggerBody()?['text'], ''''))
     ```
3. **Select**:
   - Metric: `item()?['Title']`
   - Value: `item()?['CurrentValue']`
   - Target: `item()?['Target']`
   - Status: `item()?['Status']?['Value']`
   - Category: `item()?['Category']?['Value']`
   - Details: `item()?['Details']`
   - Owner: `item()?['Owner']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `SecurityData` (Text)
6. Name: `Get Security Metrics` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "IT Service Health"

**Trigger phrases**: `IT service health`, `System status`, `How are our systems`, `Service availability`, `IT dashboard`, `Any outages`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get IT Incidents** (PriorityFilter = "All") | `Topic.IncidentData` |
| 2 | **Generative answers** — prompt: `Using the IT status report knowledge and incident data, present an IT service health summary: (1) Overall service availability status, (2) Incident summary: total by priority, avg MTTR vs SLA, (3) Any open/unresolved incidents requiring attention, (4) Change management success rate, (5) Key concerns and recommendations. Incident data: {Topic.IncidentData}` | — |
| 3 | Ask: "Would you like to see: Project status / Security posture / Specific incident details" → Multiple choice | `Topic.NextAction` |
| 4 | Condition routing or End topic | — |

### 4.2 Topic: "Incident Summary"

**Trigger phrases**: `Incidents this month`, `P1 incidents`, `How many incidents`, `Open incidents`, `Incident report`, `What broke`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "Which priority level?" → Multiple choice: All / P1 - Critical / P2 - High / P3 - Medium | `Topic.Priority` |
| 2 | Call Action: **Get IT Incidents** (PriorityFilter ← Topic.Priority) | `Topic.IncidentData` |
| 3 | **Generative answers** — prompt: `Present the following incident data as a summary table with columns: ID, Priority, System, Date, Status, MTTR, Impact. Then highlight: (1) Any open/in-progress incidents, (2) Incidents that exceeded MTTR SLA (P1: 4 hours, P2: 8 hours), (3) Root cause patterns, (4) Recommendations. Data: {Topic.IncidentData}` | — |
| 4 | End topic | — |

### 4.3 Topic: "Project Status"

**Trigger phrases**: `IT project status`, `Cloud migration status`, `Project update`, `What projects are delayed`, `IT initiatives`, `Digital portal status`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "Which projects?" → Multiple choice: All / On Track / Minor Delay / At Risk | `Topic.StatusFilter` |
| 2 | Call Action: **Get IT Projects** (StatusFilter ← Topic.StatusFilter) | `Topic.ProjectData` |
| 3 | **Generative answers** — prompt: `Present the following IT project data as a status dashboard table with columns: Project, Owner, Status, Progress %, Budget/Spent, Target Date, Next Milestone. Then highlight: (1) Projects at risk or delayed, (2) Budget variances, (3) Key risks that need executive attention, (4) Upcoming milestones in the next 30 days (today is May 2026). Data: {Topic.ProjectData}` | — |
| 4 | End topic | — |

### 4.4 Topic: "Security Posture"

**Trigger phrases**: `Security posture`, `Cybersecurity status`, `Vulnerabilities`, `Are we secure`, `Security score`, `Threat status`, `CISO report`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get Security Metrics** (Category = "All") | `Topic.SecurityData` |
| 2 | **Generative answers** — prompt: `Using the security posture report knowledge and the following metrics data, present a comprehensive security posture summary: (1) Overall security score and trend, (2) RAG dashboard of all security metrics, (3) Items in Red or Amber status requiring attention, (4) Threat landscape summary, (5) Compliance status, (6) Top 3 recommendations for executive action. Metric data: {Topic.SecurityData}` | — |
| 3 | End topic | — |

---

## Phase 5: Testing

| Test Query | Expected Response |
|---|---|
| "How many P1 incidents this month?" | "2 P1 incidents in April — both resolved. 1 P1 currently open (INC-0503, Claims Database)" |
| "What's the cloud migration status?" | "72% complete, on track for year-end. Next milestone: CRM to Azure by June 15" |
| "Are there any open security vulnerabilities?" | "2 critical CVEs (WebLogic) — patches scheduled May 10. 12 high-severity in remediation" |
| "What's our security score?" | "Overall 8.1/10 (Green, Improving). Secure Score 82/100 — target 85" |
| "Which projects are delayed?" | "AI Underwriting (3 weeks) and Cybersecurity Enhancement (vendor delay)" |
| "Any incidents still open?" | "INC-0503 (Claims Database, P1) and INC-0505 (HR Portal, P4)" |

### Demo Script (7 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "How are our IT systems?" | "The CIO needs a real-time view of IT health without reading a 20-page report" |
| 1:30 | Show service health summary | "99.9%+ availability across core systems, 2 P1s resolved within SLA" |
| 2:30 | "What's the status of the cloud migration?" | "Project status in a single question" |
| 3:30 | Show project dashboard | "72% complete, on track. Data warehouse at risk — needs attention" |
| 4:30 | "What's our security posture?" | "For the CISO — instant security dashboard" |
| 5:30 | Show security scorecard | "Overall Green at 8.1/10. Two critical CVEs flagged with remediation date" |
| 6:30 | Show items needing executive action | "Clear action items: approve patching window, escalate vendor risk, add SOC analysts" |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Incident data not filtering by priority | Priority choice values contain spaces/dashes | Ensure filter matches exact choice text: `P1 - Critical` |
| Security metrics showing wrong category | Category filter mismatch | Use exact SharePoint choice values in topic options |
| Project budget calculations wrong | Number formatting | Ensure Budget_M and Spent_M are Number type in SharePoint |
| Open incidents not highlighted | AI not interpreting "Open"/"In Progress" status | Add explicit instruction in prompt: "flag items where Status is Open or In Progress" |
