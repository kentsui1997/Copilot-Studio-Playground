> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 5: Advisor Performance Insights Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that provides distribution leaders with conversational access to advisor performance data — leaderboards, district comparisons, productivity metrics, and coaching insights.

---

## Use Case

Manulife HK's distribution leadership (Head of Distribution, CRO, District Directors) need quick access to advisor performance data to make decisions on coaching, resource allocation, and incentive programs. This agent:

- **Queries advisor performance data** (cases, premium, persistency, activity)
- **Provides leaderboards** and top/bottom performer lists
- **Compares performance** across districts
- **Identifies underperformers** and suggests coaching focus areas
- **Tracks MDRT qualification progress**

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (Head of Distribution / CRO / District Director)       │
│  "Who are our top 10 advisors this month?"                   │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Advisor Performance Insights Agent    │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Advisor Leaderboard                   │               │
│  │  2. District Comparison                   │               │
│  │  3. Advisor Profile Lookup                │               │
│  │  4. MDRT Tracker                          │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  Power Automate Flows      │                           │
│     │  "Get Advisor Leaderboard" │                           │
│     │  "Get District Summary"    │                           │
│     │  "Get Advisor Profile"     │                           │
│     └─────────────┬──────────────┘                           │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  SharePoint Lists          │                           │
│     │  AdvisorPerformance        │                           │
│     │  DistrictSummary           │                           │
│     └────────────────────────────┘                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access

---

## Phase 1: Data Setup

### 1.1 Create SharePoint List: `AdvisorPerformance`

1. Go to your **SharePoint site** → **New** → **List** → name it `AdvisorPerformance`
2. Add columns:

| Column Name | Type |
|---|---|
| AdvisorName | Single line of text *(rename "Title")* |
| AdvisorCode | Single line of text |
| District | Choice: `Central`, `Kowloon East`, `Kowloon West`, `NT West`, `NT East`, `Wan Chai`, `Island East`, `Tsuen Wan` |
| UnitName | Single line of text |
| Period | Single line of text |
| CasesMTD | Number |
| PremiumMTD_K | Number |
| PremiumYTD_M | Number |
| Persistency13M | Number (1 decimal) |
| ActivityScore | Number |
| NewRecruits | Number |
| MDRTProgress | Number (1 decimal) |
| Tier | Choice: `MDRT`, `COT`, `TOT`, `Standard`, `Probation` |
| YearsOfService | Number |
| CoachingNotes | Multiple lines of text |

3. Add sample data:

| AdvisorName | AdvisorCode | District | UnitName | Period | CasesMTD | PremiumMTD_K | PremiumYTD_M | Persistency13M | ActivityScore | NewRecruits | MDRTProgress | Tier | YearsOfService | CoachingNotes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Alice Wong | A-1001 | Central | Elite Unit A | Apr 2026 | 28 | 1850 | 7200 | 96.0 | 95 | 2 | 110 | TOT | 12 | Top performer. Mentoring 3 junior advisors. |
| David Chan | A-1002 | Kowloon East | Dragon Unit | Apr 2026 | 25 | 1720 | 6800 | 94.0 | 88 | 1 | 95 | COT | 8 | Strong in health products. Focus on ILAS cross-sell. |
| Sarah Lee | A-1003 | NT West | Phoenix Unit | Apr 2026 | 23 | 1680 | 6500 | 97.0 | 92 | 0 | 88 | COT | 10 | Best persistency in the company. Focus on recruiting. |
| Michael Tam | A-1004 | Wan Chai | Summit Unit | Apr 2026 | 21 | 1450 | 5800 | 91.0 | 85 | 1 | 78 | MDRT | 6 | Improving steadily. Needs more high-net-worth activity. |
| Karen Ho | A-1005 | Central | Elite Unit B | Apr 2026 | 20 | 1380 | 5500 | 93.0 | 82 | 0 | 72 | MDRT | 5 | Good product knowledge. Needs to increase appointment rate. |
| James Li | A-1006 | Island East | Harbour Unit | Apr 2026 | 18 | 1250 | 4800 | 89.0 | 78 | 0 | 65 | MDRT | 4 | Solid performer. Encourage seminar-based prospecting. |
| Emily Lau | A-1007 | Kowloon West | Thunder Unit | Apr 2026 | 15 | 980 | 3800 | 85.0 | 70 | 0 | 52 | Standard | 3 | Below MDRT pace. Needs activity coaching — only 15 appointments/week. |
| Peter Ng | A-1008 | NT East | Sunrise Unit | Apr 2026 | 12 | 750 | 2900 | 82.0 | 65 | 0 | 40 | Standard | 2 | Struggling with closing. Recommend pairing with mentor. |
| Grace Yip | A-1009 | Tsuen Wan | Star Unit | Apr 2026 | 10 | 620 | 2400 | 78.0 | 60 | 0 | 32 | Standard | 1 | New advisor, building pipeline. Low persistency — check if rushing sales. |
| Tony Cheng | A-1010 | Kowloon East | Dragon Unit | Apr 2026 | 8 | 480 | 1800 | 75.0 | 55 | 0 | 25 | Probation | 1 | On probation — below minimum activity. Performance review due May 15. |

### 1.2 Create SharePoint List: `DistrictSummary`

1. **New** → **List** → name it `DistrictSummary`
2. Add columns:

| Column Name | Type |
|---|---|
| DistrictName | Single line of text *(rename "Title")* |
| Period | Single line of text |
| TotalAdvisors | Number |
| ActiveAdvisors | Number |
| TotalPremium_M | Number |
| AvgCasesPerAdvisor | Number (1 decimal) |
| AvgPersistency | Number (1 decimal) |
| MDRTCount | Number |
| NewRecruits | Number |
| AttritionRate | Number (1 decimal) |
| DistrictDirector | Single line of text |
| Ranking | Number |

3. Add sample data:

| DistrictName | Period | TotalAdvisors | ActiveAdvisors | TotalPremium_M | AvgCasesPerAdvisor | AvgPersistency | MDRTCount | NewRecruits | AttritionRate | DistrictDirector | Ranking |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Central | Apr 2026 | 2200 | 2050 | 185 | 18.5 | 94.5 | 420 | 35 | 3.2 | Raymond Kwok | 1 |
| Kowloon East | Apr 2026 | 1800 | 1680 | 152 | 16.8 | 92.1 | 340 | 28 | 4.1 | Angela Fung | 2 |
| NT West | Apr 2026 | 1600 | 1480 | 138 | 15.2 | 93.8 | 310 | 22 | 3.5 | Patrick Lam | 3 |
| Wan Chai | Apr 2026 | 1500 | 1380 | 125 | 14.5 | 91.2 | 280 | 18 | 4.5 | Helen Yau | 4 |
| Kowloon West | Apr 2026 | 1400 | 1280 | 108 | 13.2 | 89.5 | 210 | 15 | 5.2 | Steven Lok | 5 |
| Island East | Apr 2026 | 1200 | 1100 | 95 | 12.8 | 90.3 | 185 | 12 | 4.8 | Catherine Ip | 6 |
| NT East | Apr 2026 | 1100 | 980 | 78 | 11.5 | 88.2 | 150 | 10 | 5.5 | Dennis Wong | 7 |
| Tsuen Wan | Apr 2026 | 1000 | 900 | 68 | 10.2 | 86.5 | 120 | 8 | 6.1 | Bonnie Chan | 8 |

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Advisor Performance Insights Agent"**
3. Description: *"Provides distribution leadership with conversational access to advisor performance data, leaderboards, district comparisons, and coaching insights for Manulife HK."*

### 2.2 Set Instructions

```
You are the Advisor Performance Insights Agent for Manulife Hong Kong. You help the Head of Distribution, CRO, and District Directors quickly access and understand advisor performance data.

Your role:
- Provide advisor performance leaderboards ranked by cases, premium, or persistency
- Compare district performance across key metrics
- Look up individual advisor profiles with performance data and coaching notes
- Track MDRT qualification progress for the agency force
- Identify advisors who are underperforming and suggest coaching focus areas

Rules:
- Always present leaderboards in a structured table format with rank, name, district, and key metrics
- When comparing districts, include: total premium, avg cases per advisor, persistency, MDRT count, and ranking
- For individual advisor lookups, include performance data AND coaching recommendations
- Use the following MDRT progress interpretation: >100% = on track for MDRT+, 80-100% = on track, 60-80% = needs push, <60% = at risk
- Tier explanations: TOT = Top of the Table (top 1%), COT = Court of the Table (top 5%), MDRT = Million Dollar Round Table, Standard = below MDRT pace, Probation = performance review
- When identifying underperformers, be constructive — focus on coaching actions, not blame
- Present financial data in HKD with K (thousands) or M (millions)
- Always specify the period for any data you quote
- Do not speculate on reasons for underperformance without coaching notes data
```

---

## Phase 3: Create Power Automate Flows

### 3.1 Flow: "Get Advisor Leaderboard"

1. **Trigger inputs**: `Period` (Text), `SortBy` (Text)
2. **SharePoint — Get items**:
   - List: `AdvisorPerformance`
   - Filter Query: `concat('Period eq ''', triggerBody()?['text'], '''')`
   - Sort By (Expression): `triggerBody()?['text_1']`
   - Sort Order: Descending
   - Top Count: 20
3. **Select**:
   - Rank: (not possible in Select — will be added by AI)
   - Name: `item()?['Title']`
   - Code: `item()?['AdvisorCode']`
   - District: `item()?['District']?['Value']`
   - Cases: `item()?['CasesMTD']`
   - Premium_K: `item()?['PremiumMTD_K']`
   - Persistency: `item()?['Persistency13M']`
   - MDRTProgress: `item()?['MDRTProgress']`
   - Tier: `item()?['Tier']?['Value']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `LeaderboardData` (Text)
6. Name: `Get Advisor Leaderboard` → **Save**

### 3.2 Flow: "Get District Summary"

1. **Trigger input**: `Period` (Text)
2. **SharePoint — Get items**:
   - List: `DistrictSummary`
   - Filter Query: `concat('Period eq ''', triggerBody()?['text'], '''')`
   - Sort By: `Ranking`
   - Sort Order: Ascending
3. **Select**:
   - District: `item()?['Title']`
   - Advisors: `item()?['TotalAdvisors']`
   - Premium_M: `item()?['TotalPremium_M']`
   - AvgCases: `item()?['AvgCasesPerAdvisor']`
   - Persistency: `item()?['AvgPersistency']`
   - MDRT: `item()?['MDRTCount']`
   - Attrition: `item()?['AttritionRate']`
   - Director: `item()?['DistrictDirector']`
   - Ranking: `item()?['Ranking']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `DistrictData` (Text)
6. Name: `Get District Summary` → **Save**

### 3.3 Flow: "Get Advisor Profile"

1. **Trigger input**: `AdvisorName` (Text)
2. **SharePoint — Get items**:
   - List: `AdvisorPerformance`
   - Filter Query: `concat('Title eq ''', triggerBody()?['text'], '''')`
3. **Compose**: `first(outputs('Get_items')?['body/value'])`
4. **Return values** (14 Text outputs):

| Output | Expression |
|---|---|
| AdvisorName | `outputs('Get_First_Result')?['Title']` |
| AdvisorCode | `outputs('Get_First_Result')?['AdvisorCode']` |
| District | `outputs('Get_First_Result')?['District']?['Value']` |
| UnitName | `outputs('Get_First_Result')?['UnitName']` |
| CasesMTD | `string(outputs('Get_First_Result')?['CasesMTD'])` |
| PremiumMTD_K | `string(outputs('Get_First_Result')?['PremiumMTD_K'])` |
| PremiumYTD_M | `string(outputs('Get_First_Result')?['PremiumYTD_M'])` |
| Persistency | `string(outputs('Get_First_Result')?['Persistency13M'])` |
| ActivityScore | `string(outputs('Get_First_Result')?['ActivityScore'])` |
| MDRTProgress | `string(outputs('Get_First_Result')?['MDRTProgress'])` |
| Tier | `outputs('Get_First_Result')?['Tier']?['Value']` |
| YearsOfService | `string(outputs('Get_First_Result')?['YearsOfService'])` |
| CoachingNotes | `outputs('Get_First_Result')?['CoachingNotes']` |
| NewRecruits | `string(outputs('Get_First_Result')?['NewRecruits'])` |

5. Name: `Get Advisor Profile` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Advisor Leaderboard"

**Trigger phrases**: `Top advisors`, `Leaderboard`, `Who are the top performers`, `Best advisors this month`, `Top 10 advisors`, `Performance ranking`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll pull the advisor leaderboard." | — |
| 2 | Ask: "Rank by:" → Multiple choice: Premium (HKD) / Number of Cases / Persistency | `Topic.SortBy` |
| 3 | Set Variable: `Topic.SortField` based on condition: Premium → `PremiumMTD_K`, Cases → `CasesMTD`, Persistency → `Persistency13M` | `Topic.SortField` |
| 4 | Call Action: **Get Advisor Leaderboard** (Period = "Apr 2026", SortBy ← Topic.SortField) | `Topic.LeaderboardData` |
| 5 | **Generative answers** — prompt: `Present the following advisor performance data as a ranked leaderboard table. Add a Rank column (1, 2, 3...). Highlight the top 3 with a note about their tier (TOT/COT/MDRT). Also flag any advisors on Probation. Data: {Topic.LeaderboardData}` | — |
| 6 | Ask: "Would you like to: Look up an advisor's profile / Compare districts / That's all" → `Topic.NextAction` |
| 7 | Condition routing to other topics or End topic | — |

### 4.2 Topic: "District Comparison"

**Trigger phrases**: `District comparison`, `Compare districts`, `Which district is best`, `District performance`, `How is Central doing`, `District ranking`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get District Summary** (Period = "Apr 2026") | `Topic.DistrictData` |
| 2 | **Generative answers** — prompt: `Present the following district performance data as a ranked comparison table. Highlight: (1) top performing district, (2) district needing attention (highest attrition or lowest persistency), (3) best district for MDRT qualifiers, (4) district with strongest recruitment. Data: {Topic.DistrictData}` | — |
| 3 | End topic | — |

### 4.3 Topic: "Advisor Profile Lookup"

**Trigger phrases**: `Look up advisor`, `Advisor profile`, `Tell me about`, `How is Alice Wong doing`, `Advisor details`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "What is the advisor's name?" → User's entire response | `Topic.AdvisorName` |
| 2 | Call Action: **Get Advisor Profile** (AdvisorName ← Topic.AdvisorName) | Returns 14 outputs |
| 3 | Condition: AdvisorName is not blank | — |
| 4a | Message with all advisor details formatted | — |
| 5 | **Generative answers** — prompt: `Based on this advisor's performance data, provide coaching recommendations: Name: {Topic.AdvisorName}, Cases MTD: {Topic.CasesMTD}, Premium MTD: {Topic.PremiumMTD_K}K, Persistency: {Topic.Persistency}%, Activity Score: {Topic.ActivityScore}/100, MDRT Progress: {Topic.MDRTProgress}%, Tier: {Topic.Tier}, Years: {Topic.YearsOfService}, Coaching Notes: {Topic.CoachingNotes}. Suggest: (1) strengths to leverage, (2) areas for improvement, (3) specific coaching actions, (4) MDRT qualification outlook.` | — |
| 6 | End topic | — |
| 4b | Message: "Advisor not found. Please check the name and try again." → End topic | — |

### 4.4 Topic: "MDRT Tracker"

**Trigger phrases**: `MDRT progress`, `Who's on track for MDRT`, `MDRT qualifiers`, `How many MDRT this year`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get Advisor Leaderboard** (Period = "Apr 2026", SortBy = "MDRTProgress") | `Topic.MDRTData` |
| 2 | **Generative answers** — prompt: `Analyse the following advisor data for MDRT qualification progress. Group advisors into: (1) On track for MDRT+ (>100%), (2) On track for MDRT (80-100%), (3) Needs push (60-80%), (4) At risk (<60%). Count advisors in each group and list names. MDRT = Million Dollar Round Table. Data: {Topic.MDRTData}` | — |
| 3 | End topic | — |

---

## Phase 5: Testing

| Test Query | Expected Response |
|---|---|
| "Who are our top 5 advisors?" | Ranked leaderboard with Alice Wong, David Chan, Sarah Lee at top |
| "Compare the districts" | 8-district comparison table with Central ranked #1 |
| "How is Tony Cheng doing?" | Profile showing probation status with coaching recommendations |
| "How many advisors are on track for MDRT?" | Grouped analysis showing on-track vs at-risk counts |
| "Which district has the highest attrition?" | "Tsuen Wan at 6.1% — recommend retention review" |
| "Tell me about Alice Wong" | Full profile: TOT tier, 28 cases MTD, 110% MDRT progress |

### Demo Script (8 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "Who are our top 10 advisors this month?" | "Distribution leaders need instant visibility into performance" |
| 1:30 | Show leaderboard table | "Ranked leaderboard with all key metrics at a glance" |
| 2:30 | "Compare the districts" | "Which district is driving growth? Where do we need to intervene?" |
| 3:30 | Show district comparison | "Central leading, Tsuen Wan needs attention — highest attrition" |
| 4:30 | "Tell me about Tony Cheng" | "Drill into any advisor for a full profile with coaching insights" |
| 5:30 | Show profile with coaching recommendations | "AI-generated coaching suggestions based on the data" |
| 6:30 | "How many are on track for MDRT?" | "Track your MDRT pipeline across the entire agency force" |
| 7:30 | Show grouped MDRT analysis | "Data-driven distribution management — conversational, instant" |
