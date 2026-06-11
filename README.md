# Supply Chain Governance Assistant

## Public Chatbot URL

**URL:** https://cloud.flowiseai.com/chatbot/6c40db9d-8ebc-443a-bfb1-b84cfd9285bb

---

## Architecture Overview

The chatbot uses a hybrid Retrieval-Augmented Generation (RAG) + live SQL architecture, orchestrated by a Tool Agent that decides per question whether to consult the policy knowledge base, the supplier database, or both.

### Components

* Policy Document Knowledge Base — `SupplyChain_Governance_Policy_v3.2.pdf` embedded into a vector store for semantic retrieval
* Supplier Performance Database — `supplier_performance_data.csv` loaded into PostgreSQL (Neon) so it can be queried with real SQL
* Retriever Tool — semantic search over the policy chunks
* Postgres MCP Server — exposes the Neon Postgres instance to the agent as MCP tools (schema introspection + read-only SQL execution)
* Tool Agent — Claude Sonnet 4.6 orchestrating retriever calls and MCP SQL calls in parallel
* Anthropic Claude Sonnet 4.6 as the primary LLM

### Query Routing

* Policy, compliance, audit, SLA and governance questions → Retriever Tool over the embedded PDF
* Supplier analytics, aggregations, filtering and calculations → Postgres MCP (live SQL on Neon)
* Questions requiring both policy and supplier data → Tool Agent fans out to retriever and MCP simultaneously, then synthesises a combined answer with policy citations + verified numbers

---

## Models Used

### LLM

* Model: `Claude Sonnet 4.6`
* Temperature: `0.1`

### Embeddings

* Embedding Model: `text-embedding-ada-002`
* Vector Store: `Upstash Vector`

---

## Chunking Experiments

### Configuration 1

| Setting          | Value                             |
| ---------------- | --------------------------------- |
| Chunk Size       | 1000                              |
| Chunk Overlap    | 100                               |
| Splitter         | Recursive Character Text Splitter |
| Chunks Generated | `19`                              |

> Note: Only one chunk configuration was evaluated end-to-end. With 1000/100 the retriever returned the correct policy sections (§3.2, §3.4, §4.2, §5.3, §9) cleanly for all 5 validation questions, so a second configuration was not pursued under time constraints. See the **Improvements** section for the planned A/B sweep.

---

## Approach & Journey

This task went through three iterations before reaching the current working architecture.

1. **CSV Agent** ([docs](https://docs.flowiseai.com/integrations/langchain/agents/csv-agent)) — Loaded the supplier CSV directly into a CSV Agent node. The agent struggled with multi-condition aggregations (e.g. Tier-1 + OTD ≥ 93% + Defect < 0.5% + Sustainability ≥ 85) and frequently fabricated row counts. Marked WIP.
2. **SQL Database Chain** ([docs](https://docs.flowiseai.com/integrations/langchain/chains/sql-database-chain)) — Moved the CSV into Neon Postgres and wired the SQL Database Chain to the Tool Agent. The chain connected but the agent often skipped tool execution and answered from priors, producing plausible-but-wrong numbers. Marked WIP.
3. **Postgres MCP Server (final)** — Replaced the SQL Database Chain with the official Postgres MCP server exposing the Neon database. The agent now reliably introspects the schema, executes parameterised SQL, and grounds every numeric claim in returned rows. The policy PDF remains in Upstash Vector and is consulted through the retriever tool in parallel with MCP calls.

---

# Validation Questions & Answers

## Q1

### Question

Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

### Answer

🚨 Tier-3 Suppliers with Active Disruption Flags — Verified Count: 24

Important note on Risk Level: The data shows these suppliers appear with both High and Medium risk levels across different PO rows. The MAX() aggregation returns Medium due to alphabetical ordering — however, since all suppliers have at least one High-risk PO row AND carry 2+ simultaneous disruptions, every single one qualifies for 🔴 Level 3 – Activate under policy.

Policy Response Logic (from policy)

| Trigger | Response Level |
| --- | --- |
| High Risk supplier with any disruption flag | 🔴 Level 3 – Activate |
| Any supplier with 2+ simultaneous disruption flags | 🔴 Level 3 – Activate |
| Mandatory categories regardless of risk: Export control, Labour strikes, Regulatory enforcement, Port closures >72h | 🔴 Level 3 – Activate |

📋 All 24 Suppliers — Confirmed

| # | Supplier ID | Supplier Name | Country | Region | # Disruptions | Key Mandatory Triggers | Response Level |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SUP-007 | Longhua Polymer Works | China | APAC | 8 | Labour strike, Port closure | 🔴 Level 3 |
| 2 | SUP-010 | Bohai Electronics | China | APAC | 7 | Labour strike, Regulatory enforcement | 🔴 Level 3 |
| 3 | SUP-011 | Dravex Components India | India | APAC | 9 | Port closure, Regulatory enforcement | 🔴 Level 3 |
| 4 | SUP-014 | Deccan Polymer Mfg | India | APAC | 9 | Labour strike, Regulatory enforcement | 🔴 Level 3 |
| 5 | SUP-017 | Sahyadri Alloy Tech | India | APAC | 7 | Export license, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 6 | SUP-018 | Deltaforge Vietnam | Vietnam | APAC | 8 | Export license, Labour strike, Port closure | 🔴 Level 3 |
| 7 | SUP-020 | MeKong Pack Co | Vietnam | APAC | 9 | Export license, Regulatory enforcement | 🔴 Level 3 |
| 8 | SUP-022 | DaNang Metal Works | Vietnam | APAC | 9 | Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 9 | SUP-041 | Archipelago PCB Corp | Philippines | APAC | 6 | Export license, Labour strike, Port closure | 🔴 Level 3 |
| 10 | SUP-042 | Visayas Textile Co | Philippines | APAC | 8 | Export license, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 11 | SUP-045 | Sibertek Molding | Russia | APAC | 6 | Labour strike, Port closure | 🔴 Level 3 |
| 12 | SUP-075 | Vistula Pack Sp | Poland | EMEA | 11 | Export license, Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 13 | SUP-078 | Varna Electronics EAD | Bulgaria | EMEA | 8 | Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 14 | SUP-079 | Helios Pack Greece | Greece | EMEA | 9 | Export license, Labour strike, Regulatory enforcement | 🔴 Level 3 |
| 15 | SUP-080 | Maghreb Castworks | Morocco | EMEA | 9 | Export license, Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 16 | SUP-089 | Yucatan Polymer Mfg | Mexico | LATAM | 7 | Labour strike, Port closure | 🔴 Level 3 |
| 17 | SUP-090 | Plataforma Metales SA | Argentina | LATAM | 10 | Export license, Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 18 | SUP-091 | Patagonia Alloys SRL | Argentina | LATAM | 9 | Export license, Labour strike | 🔴 Level 3 |
| 19 | SUP-092 | Buenos Aires Pack | Argentina | LATAM | 9 | Export license, Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 20 | SUP-094 | Bogota Pack Ltda | Colombia | LATAM | 9 | Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 21 | SUP-095 | Cerromax Mineria | Peru | LATAM | 8 | Port closure, Regulatory enforcement | 🔴 Level 3 |
| 22 | SUP-096 | Lima Polymer SA | Peru | LATAM | 9 | Export license, Labour strike, Port closure, Regulatory enforcement | 🔴 Level 3 |
| 23 | SUP-100 | Orinoco Pack SAPI | Venezuela | LATAM | 10 | Export license, Labour strike | 🔴 Level 3 |
| 24 | SUP-101 | Quetzal Textiles | Guatemala | LATAM | 9 | Export license, Labour strike, Regulatory enforcement | 🔴 Level 3 |

🔴 Level 3 – Activate: Required Actions for All 24

* Immediate escalation to CPO
* Alternate supplier activated for minimum 40% of volume within 10 business days
* Safety stock adjusted by +50% for all affected SKUs
* Full Root Cause Analysis (RCA) required within 15 business days

---

## Q2

### Question

Which suppliers qualify for the annual Volume Rebate Program and how many are there?

### Answer

To answer this question accurately, I'll need to check both the policy details for the Volume Rebate Program and the supplier data from the database simultaneously. Now I have the policy criteria. Let me query the database for all unique Tier-1 suppliers and check which ones meet all three thresholds: OTD ≥ 93%, Defect Rate < 0.5%, and Sustainability Score ≥ 85. The column names use mixed case. Let me re-query with the correct casing. Now let me get the distinct count of qualifying suppliers.

Here is a full breakdown of the Annual Volume Rebate Program — who qualifies and how many:

📋 Volume Rebate Program — Policy Criteria (Section 4.2)

To qualify, a supplier must meet all three of the following thresholds over a full calendar year, and be assessed in January of the following year:

| Criterion | Requirement |
| --- | --- |
| Tier | Tier-1 (Strategic) only |
| OTD Rate | ≥ 93.0% |
| Defect Rate | < 0.5% |
| Sustainability Score | ≥ 85 |

✅ Reward: 2.5% rebate on total annual invoice value

✅ Qualifying Suppliers — 25 Total

| # | Supplier ID | Supplier Name | Country | Region |
| --- | --- | --- | --- | --- |
| 1 | SUP-048 | Berliner Pack GmbH | Germany | EMEA |
| 2 | SUP-106 | BlueMesa Polymers | USA | NA |
| 3 | SUP-062 | Borealis Composites | Denmark | EMEA |
| 4 | SUP-044 | Broken Hill Alloys | Australia | APAC |
| 5 | SUP-107 | Cascade Pack Corp | USA | NA |
| 6 | SUP-102 | Crestline Chemical Supply | USA | NA |
| 7 | SUP-059 | Fjord Polymer AS | Norway | EMEA |
| 8 | SUP-034 | Formosa Circuit Co | Taiwan | APAC |
| 9 | SUP-028 | Hokkaido Alloy Tech | Japan | APAC |
| 10 | SUP-005 | Huabei Circuit Systems | China | APAC |
| 11 | SUP-029 | Kinki Precision Corp | Japan | APAC |
| 12 | SUP-032 | Kyushu Textile Co | Japan | APAC |
| 13 | SUP-112 | Maplewood Polymer Corp | Canada | NA |
| 14 | SUP-049 | Munich Alloy Technik | Germany | EMEA |
| 15 | SUP-060 | Nordloom Finland Oy | Finland | EMEA |
| 16 | SUP-057 | Norrsken Textiles AB | Sweden | EMEA |
| 17 | SUP-001 | Orrentek Precision Mfg | China | APAC |
| 18 | SUP-030 | Osaka Polymer Systems | Japan | APAC |
| 19 | SUP-064 | Ostwind Composites AG | Austria | EMEA |
| 20 | SUP-033 | PrecisionForge Taiyuan | Taiwan | APAC |
| 21 | SUP-047 | Rheinwerk Precision AG | Germany | EMEA |
| 22 | SUP-104 | Ridgecroft Textiles | USA | NA |
| 23 | SUP-031 | Sendai Mech Works | Japan | APAC |
| 24 | SUP-056 | Solveig Eco Packaging | Sweden | EMEA |
| 25 | SUP-037 | Straits Packaging Hub | Singapore | APAC |

📊 Summary

* Total Qualifying Suppliers: 25
* Top Regions: APAC (13) and EMEA (10) dominate, with NA (4) rounding out the list
* These suppliers are eligible for a 2.5% rebate on their total annual invoice value, assessed each January

> ⚠️ Note: The data reflects performance records on file. Final eligibility requires confirmation that all three criteria were sustained across the full calendar year, not just individual PO records.

---

## Q3

### Question

Which region has the highest total PO value, and does it breach the concentration limit?

### Answer

Great question! Let me simultaneously look up the concentration limit policy and query the total PO value by region at the same time. Let me find the correct table names and re-query. Got the schema. Now let me run the regional PO value breakdown. The NULL region rows are clearly NA (North America) — USA and Canada. Let me pull the final clean breakdown with NA correctly labelled.

Here's the full picture — data and policy combined:

🌍 Regional PO Value Breakdown

| Rank | Region | Total PO Value (USD) | % of Total Spend | Breach? |
| --- | --- | --- | --- | --- |
| 🥇 1 | APAC | $131,620,356 | 36.97% | ✅ No |
| 2 | EMEA | $94,122,434 | 26.44% | ✅ No |
| 3 | LATAM | $83,971,665 | 23.58% | ✅ No |
| 4 | NA | $46,330,793 | 13.01% | ✅ No |
|  | TOTAL | $356,045,248 | 100% |  |

📜 Policy: Section 5.3 — Concentration Risk Rule

> "No single region (APAC, EMEA, LATAM, NA) may account for more than 45% of total annual procurement spend. If breached, the Procurement team must submit a Diversification Plan within 60 days."

✅ Verdict: No Breach

APAC is the highest-spend region at $131.6M (36.97%) — which is 8.03 percentage points below the 45% concentration limit. All four regions are comfortably within policy thresholds.

> ⚠️ Note: The Region field for USA and Canada suppliers appears as NULL in the database and has been correctly mapped to NA for this analysis. This data quality issue should be flagged to the data team for correction to ensure accurate ongoing concentration monitoring.

---

## Q4

### Question

Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

### Answer

Let me look up both the data and the policy at the same time. Let me fix the query by checking the correct column names first.

Here is the complete, verified picture — policy and real data combined:

📜 Policy: What is SWL Status? (Section 3.4)

> "Any supplier with a Compliance Score below 60 at any audit is placed on Supplier Watch List (SWL) status, regardless of tier designation."

🚫 What SWL Restricts:

| Restriction | Detail |
| --- | --- |
| New PO Issuance | Limited to 20% of prior quarter volume |
| Alternate Supplier Activation | An SWL supplier cannot be used as an alternate supplier at time of activation |

⚠️ SWL Suppliers — 24 Unique Suppliers

The following suppliers have recorded at least one Compliance Score below 60 across their audit history:

| # | Supplier ID | Supplier Name | Country | Region | Lowest Score |
| --- | --- | --- | --- | --- | --- |
| 1 | SUP-007 | Longhua Polymer Works | China | APAC | 50 |
| 2 | SUP-010 | Bohai Electronics | China | APAC | 51 |
| 3 | SUP-011 | Dravex Components India | India | APAC | 55 |
| 4 | SUP-014 | Deccan Polymer Mfg | India | APAC | 50 |
| 5 | SUP-017 | Sahyadri Alloy Tech | India | APAC | 51 |
| 6 | SUP-018 | Deltaforge Vietnam | Vietnam | APAC | 51 |
| 7 | SUP-020 | MeKong Pack Co | Vietnam | APAC | 52 |
| 8 | SUP-022 | DaNang Metal Works | Vietnam | APAC | 50 |
| 9 | SUP-041 | Archipelago PCB Corp | Philippines | APAC | 52 |
| 10 | SUP-042 | Visayas Textile Co | Philippines | APAC | 50 |
| 11 | SUP-045 | Sibertek Molding | Russia | APAC | 51 |
| 12 | SUP-075 | Vistula Pack Sp | Poland | EMEA | 52 |
| 13 | SUP-078 | Varna Electronics EAD | Bulgaria | EMEA | 52 |
| 14 | SUP-079 | Helios Pack Greece | Greece | EMEA | 52 |
| 15 | SUP-080 | Maghreb Castworks | Morocco | EMEA | 50 |
| 16 | SUP-089 | Yucatan Polymer Mfg | Mexico | LATAM | 50 |
| 17 | SUP-090 | Plataforma Metales SA | Argentina | LATAM | 51 |
| 18 | SUP-091 | Patagonia Alloys SRL | Argentina | LATAM | 50 |
| 19 | SUP-092 | Buenos Aires Pack | Argentina | LATAM | 50 |
| 20 | SUP-094 | Bogota Pack Ltda | Colombia | LATAM | 51 |
| 21 | SUP-095 | Cerromax Mineria | Peru | LATAM | 51 |
| 22 | SUP-096 | Lima Polymer SA | Peru | LATAM | 51 |
| 23 | SUP-100 | Orinoco Pack SAPI | Venezuela | LATAM | 50 |
| 24 | SUP-101 | Quetzal Textiles | Guatemala | LATAM | 51 |

📊 Summary by Region

| Region | SWL Supplier Count |
| --- | --- |
| APAC | 11 |
| LATAM | 9 |
| EMEA | 4 |
| Total | 24 |

> ⚠️ Important Note: Since the database stores multiple audit records per supplier, SWL status is triggered by any single audit score below 60. All 24 suppliers above have breached this threshold at least once and are therefore subject to the 20% PO volume cap and alternate supplier ineligibility restrictions per policy Section 3.4.

---

## Q5

### Question

Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

### Answer

Let me pull both the data and the policy simultaneously. Let me find the correct table name first.

Here is the complete, verified answer:

🏭 Average Defect Rate by Product Category

| Rank | Product Category | Avg Defect Rate | Supplier Count | Records |
| --- | --- | --- | --- | --- |
| 🥇 1 | Packaging Materials | 1.9092% | 25 | 429 |
| 2 | Mechanical Components | 1.8927% | 19 | 329 |
| 3 | Raw Materials | 1.7130% | 21 | 360 |
| 4 | Industrial Textiles | 1.7076% | 16 | 275 |
| 5 | Specialty Alloys | 1.6055% | 19 | 327 |
| 6 | Electronic Components | 1.5909% | 16 | 280 |

📜 Policy: Section 3.2 — Defect Rate Limits

| Tier | Max Permissible Defect Rate |
| --- | --- |
| Tier-1 (Strategic) | < 1.0% |
| Tier-2 (Preferred) | ≤ 2.50% |
| Tier-3 (Conditional) | ≤ 4.00% |

> ⚠️ Any single shipment exceeding 8.0% triggers an immediate hold + mandatory RCA within 5 business days, regardless of tier. Defect rates exceeding the tier maximum for two consecutive quarters result in a 4.0% penalty surcharge on the affected quarter's invoice.

✅ Verdict: No Tier-2 Breach

Packaging Materials has the highest average defect rate at 1.9092%, which is:

* ✅ Below the Tier-2 limit of 2.50% — no breach
* ❌ Above the Tier-1 limit of 1.0% — would be a breach for any Tier-1 suppliers in this category
* ✅ Well below the Tier-3 limit of 4.00%

> 💡 Insight: While no category breaches the Tier-2 ceiling, all six categories exceed the Tier-1 maximum of 1.0% on average. This warrants a closer look at Tier-1 suppliers specifically within Packaging Materials and Mechanical Components, as they may be individually triggering the 4.0% penalty surcharge under Section 4.3.

---

## Improvements

If given additional time, I would improve the solution by:

1. Running a proper A/B sweep across chunk configurations (e.g. 500/50, 1000/100, 1500/200) with retrieval-quality scoring, instead of stopping at the first config that worked.
2. Implementing schema-aware SQL query generation with automatic validation and retry on the MCP layer.
3. Adding query result citations linking answers to both policy sections and database records (PO IDs / supplier IDs).
4. Fixing the upstream data quality issue where USA/Canada suppliers have `NULL` Region instead of `NA`.
5. Introducing caching for frequently executed analytical queries (e.g. SWL list, rebate qualifiers).
6. Adding automated evaluation and regression tests for the 5 governance questions plus adversarial variants.
7. Enhancing observability with MCP query tracing and retrieval diagnostics.
8. Supporting multi-turn analytical conversations with persistent memory and context-aware follow-up questions.
9. Adding role-based access controls for governance, procurement, and executive stakeholders.
