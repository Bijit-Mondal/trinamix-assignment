# Supply Chain Governance Assistant

## Public Chatbot URL

**URL:** https://cloud.flowiseai.com/chatbot/6c40db9d-8ebc-443a-bfb1-b84cfd9285bb

---

## Architecture Overview

The chatbot uses a hybrid Retrieval-Augmented Generation (RAG) and SQL architecture.

### Components

* Policy Document Knowledge Base (PDF)
* Supplier Performance Database (PostgreSQL on Neon)
* Retriever Tool for policy lookup
* SQL Database Chain for supplier analytics
* Tool Agent for orchestration
* Anthropic Claude Sonnet 4.6 as the primary LLM

### Query Routing

* Policy, compliance, audit, SLA and governance questions → Retriever Tool
* Supplier analytics, aggregations, filtering and calculations → SQL Database Chain
* Questions requiring both policy and supplier data → Tool Agent combines results from both sources

---

## Models Used

### LLM

* Model: `Claude Sonnet 4.6`
* Temperature: `0.1`

### Embeddings

* Embedding Model: `text-embedding-ada-002`
* Vector Store: `upstash vector`

---

## Chunking Experiments

### Configuration 1

| Setting          | Value                             |
| ---------------- | --------------------------------- |
| Chunk Size       | 1000                               |
| Chunk Overlap    | 100                                |
| Splitter         | Recursive Character Text Splitter |
| Chunks Generated | `19`                     |


---

# Validation Questions & Answers

## Q1

### Question

Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

### Answer

Supplier_ID 	Supplier_Name 	Region 	Country 	Product_Category 	Contract_Tier 	Risk_Level 	Active_Disruptions 	OTD_Rate_Pct 	Defect_Rate_Pct 	Compliance_Score 	Sustainability_Score
SUP-004 	Yangtze Fiber Materials 	APAC 	China 	Industrial Textiles 	Tier-2 	Medium 	Geopolitical tension flag 	90.4 	1.9 	79 	53
SUP-008 	Shengda Pack Industries 	APAC 	China 	Packaging Materials 	Tier-2 	High 	Export license risk 	85.3 	2.03 	78 	62
SUP-010 	Bohai Electronics 	APAC 	China 	Electronic Components 	Tier-3 	High 	Typhoon season disruption 	63.0 	2.63 	64 	30
SUP-011 	Dravex Components India 	APAC 	India 	Mechanical Components 	Tier-3 	High 	Typhoon season disruption 	80.6 	2.98 	74 	63
SUP-017 	Sahyadri Alloy Tech 	APAC 	India 	Specialty Alloys 	Tier-3 	High 	Export license risk 	65.0 	2.73 	52 	58
SUP-018 	Deltaforge Vietnam 	APAC 	Vietnam 	Mechanical Components 	Tier-3 	Medium 	Port congestion risk 	73.1 	1.91 	62 	33
SUP-020 	MeKong Pack Co 	APAC 	Vietnam 	Packaging Materials 	Tier-3 	Medium 	Geopolitical tension flag 	83.0 	1.43 	52 	45
📊 Key Observations
Metric 	Value
Total Tier-3 Suppliers with Disruptions 	Multiple across APAC
Dominant Region 	APAC (China, India, Vietnam)
Most Common Disruption 	Geopolitical tension flag, Typhoon season disruption
High Risk Suppliers 	SUP-010, SUP-011, SUP-017
⚠️ Risk Highlights

    🔴 SUP-010 (Bohai Electronics) — Lowest OTD at 63% with High Risk + Typhoon disruption
    🔴 SUP-017 (Sahyadri Alloy Tech) — Low compliance (52) + Export license risk
    🔴 SUP-011 (Dravex Components India) — Highest defect rate (2.98%) + High Risk
    🟡 SUP-018 (Deltaforge Vietnam) — Port congestion risk with low sustainability score (33)

    Recommendation: Prioritize contingency sourcing or activate alternate suppliers (Alt_Supplier_ID) for High-Risk Tier-3 suppliers, especially those with compounding disruption flags and low OTD rates.## ✅ Tier-3 Suppliers with Active Disruption Flags & Policy Response Levels

Here are the 4 Tier-3 suppliers with active disruption flags, mapped to their applicable policy response level:
🔴 Level 3 — ACTIVATE (High Risk)
Supplier ID 	Supplier Name 	Country 	Disruption Flag 	OTD % 	Defect % 	Compliance Score
SUP-010 	Bohai Electronics 	China 	Typhoon season disruption 	63.0% 	2.63% 	64
SUP-011 	Dravex Components India 	India 	Typhoon season disruption 	80.6% 	2.98% 	74
SUP-017 	Sahyadri Alloy Tech 	India 	Export license risk 	65.0% 	2.73% 	52

    Policy Action (Level 3):

        🚨 Immediate escalation to CPO
        Alternate supplier activated for minimum 40% of volume within 10 business days
        Safety stock adjusted by +50%
        Full Root Cause Analysis (RCA) required within 15 business days
        ⚠️ Note: SUP-017's "Export license risk" is a mandatory Level 3 trigger regardless of risk level, per policy Section 9

🟡 Level 2 — MANAGE (Medium Risk)
Supplier ID 	Supplier Name 	Country 	Disruption Flag 	OTD % 	Defect % 	Compliance Score
SUP-018 	Deltaforge Vietnam 	Vietnam 	Port congestion risk 	73.1% 	1.91% 	62
SUP-020 	MeKong Pack Co 	Vietnam 	Geopolitical tension flag 	83.0% 	1.43% 	52

    Policy Action (Level 2):

        📞 Bi-weekly escalation calls
        Safety stock adjusted by +30%
        Alternate supplier placed on 48-hour readiness notice

📌 Summary Table
Supplier 	Risk Level 	Disruption 	Response Level
Bohai Electronics 	🔴 High 	Typhoon season disruption 	Level 3 – Activate
Dravex Components India 	🔴 High 	Typhoon season disruption 	Level 3 – Activate
Sahyadri Alloy Tech 	🔴 High 	Export license risk 	Level 3 – Activate (mandatory)
Deltaforge Vietnam 	🟡 Medium 	Port congestion risk 	Level 2 – Manage
MeKong Pack Co 	🟡 Medium 	Geopolitical tension flag 	Level 2 – Manage

    💡 Bottom Line: 3 out of 5 disrupted Tier-3 suppliers require immediate CPO escalation (Level 3). All Tier-3 suppliers are mandated to have an alternate supplier on file — ensure those alternates are not in the same geopolitical risk region before activation.## Tier-3 Suppliers with Active Disruption Flags

Query Results
Supplier_ID 	Supplier_Name 	Region 	Country 	Product_Category 	Contract_Tier 	Risk_Level 	Active_Disruptions 	OTD_Rate_Pct 	Defect_Rate_Pct 	Compliance_Score 	Sustainability_Score
SUP-004 	Yangtze Fiber Materials 	APAC 	China 	Industrial Textiles 	Tier-2 	Medium 	Geopolitical tension flag 	90.4 	1.9 	79 	53
SUP-008 	Shengda Pack Industries 	APAC 	China 	Packaging Materials 	Tier-2 	High 	Export license risk 	85.3 	2.03 	78 	62
SUP-010 	Bohai Electronics 	APAC 	China 	Electronic Components 	Tier-3 	High 	Typhoon season disruption 	63.0 	2.63 	64 	30
SUP-011 	Dravex Components India 	APAC 	India 	Mechanical Components 	Tier-3 	High 	Typhoon season disruption 	80.6 	2.98 	74 	63
SUP-017 	Sahyadri Alloy Tech 	APAC 	India 	Specialty Alloys 	Tier-3 	High 	Export license risk 	65.0 	2.73 	52 	58
SUP-018 	Deltaforge Vietnam 	APAC 	Vietnam 	Mechanical Components 	Tier-3 	Medium 	Port congestion risk 	73.1 	1.91 	62 	33
SUP-020 	MeKong Pack Co 	APAC 	Vietnam 	Packaging Materials 	Tier-3 	Medium 	Geopolitical tension flag 	83.0 	1.43 	52 	45
📊 Key Observations
Metric 	Value
Total Tier-3 Suppliers with Disruptions 	Multiple across APAC
Dominant Region 	APAC (China, India, Vietnam)
Most Common Disruption 	Geopolitical tension flag, Typhoon season disruption
High Risk Suppliers 	SUP-010, SUP-011, SUP-017
⚠️ Risk Highlights
🔴 SUP-010 (Bohai Electronics) — Lowest OTD at 63% with High Risk + Typhoon disruption
🔴 SUP-017 (Sahyadri Alloy Tech) — Low compliance (52) + Export license risk
🔴 SUP-011 (Dravex Components India) — Highest defect rate (2.98%) + High Risk
🟡 SUP-018 (Deltaforge Vietnam) — Port congestion risk with low sustainability score (33)

Recommendation: Prioritize contingency sourcing or activate alternate suppliers (Alt_Supplier_ID) for High-Risk Tier-3 suppliers, especially those with compounding disruption flags and low OTD rates.

---

## Q2

### Question

Which suppliers qualify for the annual Volume Rebate Program and how many are there?

### Answer

📦 Suppliers Qualifying for Annual Volume Rebate Program

Qualification Criteria: Annual_Volume_Units ≥ 1,000,000 units

🔍 Query Results Summary

Based on the data returned, here are the qualifying suppliers:
# 	Supplier ID 	Supplier Name 	Region 	Country 	Product Category 	Contract Tier 	Annual Volume (Units) 	OTD Rate % 	Compliance Score 	Risk Level
1 	SUP-006 	Dongfeng Castings Co 	APAC 	China 	Mechanical Components 	Tier-2 	1,535,740 	87.1% 	81 	Low
2 	SUP-009 	Tianhe Alloy Group 	APAC 	China 	Specialty Alloys 	Tier-2 	1,526,657 	88.1% 	86 	Medium
3 	SUP-016 	Konkan Textile Mills 	APAC 	India 	Industrial Textiles 	Tier-2 	1,778,964 	85.9% 	88 	Low
4 	SUP-015 	Narmada Pack Solutions 	APAC 	India 	Packaging Materials 	Tier-2 	1,793,443 	81.6% 	85 	Medium
5 	SUP-004 	Yangtze Fiber Materials 	APAC 	China 	Industrial Textiles 	Tier-2 	1,310,795 	90.4% 	79 	Medium
6 	SUP-002 	Zhenlong ElectroCo 	APAC 	China 	Electronic Components 	Tier-2 	979,796 	82.0% 	84 	Medium
7 	SUP-008 	Shengda Pack Industries 	APAC 	China 	Packaging Materials 	Tier-2 	1,134,977 	85.3% 	78 	High
📊 Key Insights
Metric 	Value
✅ Total Qualifying Suppliers 	6–7 suppliers (≥1M units)
🏆 Highest Volume 	Narmada Pack Solutions — 1,793,443 units
🌍 Top Region 	APAC (China & India dominant)
📋 Most Common Tier 	Tier-2
⚠️ High Risk Qualifiers 	SUP-008 (Shengda Pack) — needs monitoring
💡 Recommendations

Priority Rebate Candidates: SUP-015 & SUP-016 lead in volume — offer tiered rebate slabs (e.g., 2% at 1M, 3.5% at 1.5M+)
Risk Flag: SUP-008 qualifies by volume but carries High Risk — consider conditional rebate tied to compliance improvement
Near-Threshold Watch: SUP-002 at ~980K units is close — incentivize to cross the 1M threshold for program inclusion

---

At this moment I understand even if there is data in NeonDB PostgreSQL flowiseai is unable to call and fabricating those responses and I think this section is still WIP.
https://docs.flowiseai.com/integrations/langchain/chains/sql-database-chain.

I previously tried using CSV Agent and that too was WIP
https://docs.flowiseai.com/integrations/langchain/agents/csv-agent

So, either I am wrong or the documentation needs an update for my architecture to work out.

---

## Improvements

If given additional time, I would improve the solution by:

1. Implementing schema-aware SQL query generation with automatic validation and retry.
2. Adding query result citations linking answers to both policy sections and database records.
3. Introducing caching for frequently executed analytical queries.
4. Adding automated evaluation and regression tests for supply chain governance questions.
5. Enhancing observability with query tracing and retrieval diagnostics.
6. Supporting multi-turn analytical conversations with persistent memory and context-aware follow-up questions.
7. Adding role-based access controls for governance, procurement, and executive stakeholders.
