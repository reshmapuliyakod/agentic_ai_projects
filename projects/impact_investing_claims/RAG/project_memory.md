# Project Memory Document
## GreenLens: UoP Bond Greenwashing Detection System

---

## 1. Project Identity

```
PROJECT NAME:     GreenLens (working title)
DOMAIN:           Greenwashing detection in EU green bonds
FOCUS:            Use of Proceeds (UoP) bonds
JURISDICTION:     EU / Ireland
BUILDER:          Central Bank of Ireland analyst
PURPOSE:          Verify green bond claims against evidence
                  across the full document chain
DUAL OUTPUT:      Q&A Mode + Analysis Mode
END STATE:        Agentic application
```

---

## 2. The Core Problem

```
WHAT IS A UoP BOND?
  Issuer raises capital (e.g. €500M)
  Promises to spend ONLY on eligible green projects
  Must report on allocation and impact

WHERE GREENWASHING HIDES:
  Gap between what documents CLAIM and what data SHOWS
  Gap between documents in the same chain
  Vague eligibility criteria with no measurable thresholds
  Impact claims without methodology
  Macro business direction contradicting green bond claims

WHY THIS IS HARD:
  No single document tells the full story
  Greenwashing lives in GAPS BETWEEN documents
  Mix of structured data (tables, numbers) and
  unstructured text (claims, commitments, opinions)
  Documents reference each other across time
```

---

## 3. The Document Universe

### Document Chain by Stage
```
PRE-ISSUANCE
  Green Bond Framework / Green Financing Framework
  Second Party Opinion (SPO)

AT ISSUANCE
  Base Prospectus (UoP section only)
  Final Terms

POST-ISSUANCE
  Allocation Report
  Impact Report
  Green Bond Report
  Sustainability Report (targeted sections)
  Annual Report (targeted sections)
```

### Document Priority for System
```
MOST STRUCTURED / START HERE:
  Final Terms              → ISIN, amounts, dates, references
  Allocation Report        → Money trail, categories, amounts

MIXED STRUCTURED + UNSTRUCTURED:
  Impact Report            → Metrics + methodology claims
  Green Bond Framework     → Eligibility rules + commitments

UNSTRUCTURED / TARGETED EXTRACTION:
  Second Party Opinion     → Verdict, version reviewed, concerns
  Base Prospectus          → UoP section, legal obligations
  Green Bond Report        → Programme summary, reconciliation
  Sustainability Report    → Emissions data, taxonomy %, controversies
  Annual Report            → Capex breakdown, financial notes,
                             strategic mentions (NEVER full doc)
```

### Key Fields Per Document
```
Green Bond Framework:
  eligible_categories, exclusion_list, framework_version,
  taxonomy_alignment_claims, reporting_commitments,
  project_selection_process, proceeds_management_approach

Second Party Opinion:
  reviewer_name, review_date, framework_version_reviewed,
  overall_verdict, icma_alignment_per_pillar,
  concerns_noted, eu_taxonomy_assessment

Base Prospectus (UoP):
  eligible_categories_legal, framework_version_referenced,
  legal_obligations, esg_risk_factors, covenants

Final Terms:
  isin, issue_size, issue_date, maturity_date,
  uop_category_reference, prospectus_reference,
  reporting_obligations

Allocation Report:
  total_proceeds_raised, total_allocated, unallocated_amount,
  allocation_by_category (table), project_list,
  project_dates, verification_body, assurance_level

Impact Report:
  impact_metrics_by_category, methodology_per_metric,
  baseline_assumptions, reporting_boundary,
  external_verification, reporting_period

Green Bond Report:
  programme_total_issued, cumulative_allocation,
  cumulative_impact, framework_version_current,
  framework_changes_noted

Sustainability Report (targeted):
  scope_1_emissions, scope_2_emissions, scope_3_emissions,
  emissions_targets, emissions_trend,
  eu_taxonomy_alignment_pct, controversies,
  green_bond_section_reference

Annual Report (targeted):
  capex_breakdown (green vs fossil),
  green_bond_financial_notes, ring_fenced_account,
  climate_risk_disclosure, strategic_report_mentions,
  legal_proceedings_environmental
```

---

## 4. The Four Verification Chains

```
CHAIN 1: MONEY TRAIL
  Did they spend it where they promised?
  Final Terms → Allocation Report → Framework → Annual Report
  Key checks:
    Raised amount = Allocated amount?
    All categories eligible under framework?
    Projects post-date bond issuance?
    Unallocated proceeds explained and properly held?

CHAIN 2: FRAMEWORK TRAIL
  Is the independent review still valid?
  Framework → SPO → Base Prospectus → Final Terms
  Key checks:
    SPO reviewed same version as current framework?
    Prospectus references current framework?
    SPO concerns addressed by issuer?
    Framework updated without new SPO?

CHAIN 3: IMPACT TRAIL
  Did the spending achieve real environmental impact?
  Allocation Report → Impact Report → Sustainability Report
  Key checks:
    Impact report covers same projects as allocation?
    Methodology disclosed per metric?
    Impact pro-rated to bond's share of project?
    Macro emissions trend consistent with micro impact claims?

CHAIN 4: ISSUER INTEGRITY TRAIL
  Is the business going in the same direction as the bond claims?
  Annual Report → Sustainability Report → Green Bond Claims
  Key checks:
    Capex breakdown: green vs fossil direction?
    Scope 1/2/3 trend since issuance?
    Green bond mentioned in strategic report?
    Environmental legal proceedings?
    EU Taxonomy alignment % vs claims?
```

---

## 5. The Flag Severity Framework

```
SEVERITY 3: CRITICAL
  Allocated > Raised
  Funded projects on exclusion list
  Allocation Report never published (>2 years)
  No ring-fencing in financial notes
  Projects predate bond by >3 years unexplained

SEVERITY 2: SIGNIFICANT
  SPO reviewed different framework version
  Impact methodology not disclosed
  Macro emissions up while impact claims positive
  Capex <10% green
  Cumulative reports don't reconcile
  No external verification of allocation or impact
  Reporting delayed >12 months

SEVERITY 1: MODERATE
  Vague eligibility criteria
  Impact not pro-rated to bond share
  Scope 3 not disclosed
  SPO >2 years old, not renewed
  Aggregate allocation only, no project detail
  Qualitative claims dominate

SEVERITY 0: INFORMATIONAL
  Minor delays <6 months
  Framework updated (positive if SPO updated)
  Partial allocation year 1 (normal)
  Metrics evolved year over year
```

---

## 6. The Two System Modes

```
Q&A MODE
  Trigger:    User asks a question
  Driver:     User sets the agenda
  Output:     Direct answer + source + confidence
  Speed:      Fast (seconds)
  Use case:   Exploration, specific lookups,
              verification of specific claims

  Question types:
    Factual lookup    → Single doc, single answer
    Comparative       → Same field across docs or time
    Verification      → User suspects something, wants check
    Explanatory       → User wants to understand something

ANALYSIS MODE
  Trigger:    User requests full verification OR
              scheduled OR new document uploaded
  Driver:     System runs predefined question agenda
  Output:     Structured verification report
  Speed:      Thorough (minutes)
  Use case:   Complete bond assessment,
              regulatory review preparation

  Internal question agenda:
    30 predefined questions across 4 verification chains
    System asks and answers its own questions
    Adapts plan based on what it finds

INTERACTION BETWEEN MODES:
  Analysis → Q&A: drill down on specific flags
  Q&A → Analysis: question reveals need for full check
  Analysis → suggests follow-up Q&A questions
```

---

## 7. The Phased Build Plan

```
PHASE 1: Basic RAG
  What:     Single document, single question type
  Document: Allocation Report (most structured, start here)
  Goal:     Factual Q&A working end to end
  Learns:   Chunking, embedding, vector search,
            retrieval quality fundamentals

PHASE 2: Hybrid Search
  What:     Structured + unstructured retrieval
  Add:      Impact Report (has both metrics and claims)
  Goal:     Get the number AND the methodology claim
  Learns:   BM25 + vector, RRF merging,
            when each retrieval type wins

PHASE 3: Query Intelligence
  What:     Transform vague questions into precise retrieval
  Add:      Framework + cross-document questions
  Techniques:
    Query Translation   → rewrite for better search
    Query Decomposition → break complex into sub-questions
    Query Construction  → natural language to SQL
  Learns:   LLM as query planner, multi-query retrieval

PHASE 4: Adaptive RAG
  What:     System chooses its own retrieval strategy
  Goal:     Simple questions stay simple,
            complex questions get full treatment
  Learns:   LangGraph branching, cost vs quality tradeoffs,
            query complexity classification

PHASE 5: Self-Corrective RAG
  What:     System evaluates and fixes its own answers
  Techniques:
    CRAG pattern        → grade chunks, rewrite if bad
    Hallucination check → answer grounded in sources?
    Contradiction detect→ two docs disagree → flag
  Learns:   LLM-as-judge, feedback loops,
            confidence calibration

PHASE 6: Agentic Router
  What:     Multi-issuer, full document chain,
            system plans its own approach
  Add:      Issuer namespacing, full tool orchestration
  Learns:   Agent architecture, tool design,
            multi-source synthesis

POST PHASE 6: Full Agentic Application
  Monitoring agent (triggered by new documents)
  Multi-agent architecture (specialist agents)
  Memory layer (short, long, episodic, domain)
  Escalation framework (auto vs human review)
  Feedback loop (analyst input improves system)
```

---

## 8. The Evaluation Strategy

### The Fundamental Rule
```
RAG fails in exactly two places:
  Retrieval failure: fetched wrong chunks
  Generation failure: had right chunks, said wrong thing
These need SEPARATE metrics.
Never conflate them.
```

### Four Evaluation Levels
```
LEVEL 1: Data Quality
  Chunk count, chunk size distribution,
  empty chunks, table preservation,
  encoding issues
  → Check before any other evaluation

LEVEL 2: Retrieval Quality
  Context Precision: of retrieved, how many relevant?
  Context Recall:    of all relevant, how many found?
  MRR:               was best chunk ranked first?
  → Needs golden dataset (manual ground truth)

LEVEL 3: Answer Quality
  Faithfulness:      answer supported by retrieved chunks?
  Answer Relevance:  answer addresses the question?
  Completeness:      all aspects covered?
  (Greenwashing specific: claim + evidence + gap all addressed?)

LEVEL 4: Domain Outcome
  Greenwashing detection accuracy
  False positive rate
  False negative rate
  Confidence calibration
  "When system says confident, is it right?"
```

### Golden Dataset
```
BUILD IN PHASE 1, USE ACROSS ALL PHASES:
  20-30 test questions about real documents
  Manual answers = ground truth
  Composition:
    5 clear greenwashing examples
    5 legitimate claim examples
    5 ambiguous cases
    5 missing data cases (system should say insufficient)

EVALUATION WORKFLOW PER PHASE:
  Before build: run test set, record baseline
  After build:  run test set, compare
  Track:        which metric improved?
                did anything regress?
```

### Metrics Target by Phase
```
PHASE 1 BASELINE:    Whatever you get (establish baseline)
PHASE 2 TARGET:      Precision improves on numerical questions
PHASE 3 TARGET:      Completeness score improves
PHASE 4 TARGET:      Cost per query drops for simple questions
PHASE 5 TARGET:      Faithfulness jumps, hallucination drops
PHASE 6 TARGET:      End-to-end greenwashing detection accuracy
```

---

## 9. The Agentic Evolution

### Agent Properties to Build Toward
```
PLANNING:      Given goal, breaks into steps
TOOL USE:      RAG phases become callable tools
OBSERVATION:   Looks at results, decides if sufficient
REPLANNING:    Changes approach based on findings
MEMORY:        Remembers within task and across tasks
ESCALATION:    Knows when humans need to be involved
```

### Memory Architecture
```
SHORT-TERM:   Within one task (prevents repetition)
LONG-TERM:    Across tasks (issuer history, patterns)
EPISODIC:     Audit trail of every action and finding
DOMAIN:       Encoded verification rules and flag criteria
```

### Three Agentic Modes
```
REACTIVE:     User triggers, agent plans steps to answer
PROACTIVE:    User triggers full analysis, agent runs agenda
              adapts plan based on findings
MONITORING:   System triggers on new document upload,
              runs autonomously, escalates only if needed
```

### Multi-Agent Future
```
ORCHESTRATOR  → breaks goal into sub-tasks
RETRIEVAL     → specialist in finding information
VERIFICATION  → specialist in cross-document checking
ANALYSIS      → specialist in interpretation and flagging
REPORTING     → specialist in output formatting
```

---

## 10. Tech Stack

```
ORCHESTRATION:    LangGraph
                  (explicit routing, visible agent decisions)

VECTOR STORE:     Weaviate (self-hosted)
                  (native multi-tenancy for issuer namespacing,
                   hybrid search built in,
                   free while learning)

STRUCTURED STORE: PostgreSQL + pgvector
                  (metrics, allocations, bond metadata)

LLM:              GPT-4o (complex reasoning, entity extraction)
                  GPT-3.5-turbo (simple routing, cheap calls)

EMBEDDINGS:       text-embedding-3-small
                  (cost effective, good quality)

DATA SOURCES:     SEC EDGAR / EU regulatory filings (free)
                  Real issuer documents (manual download initially)
                  EIB bonds recommended as starting point
                  (public, well-structured, good quality)

OBSERVABILITY:    Langfuse
                  (add in Phase 1, not Phase 10)
                  (see every routing and retrieval decision)

UI:               Streamlit
                  (fast, focus stays on backend learning)

EVALUATION:       RAGAS library
                  (automated scoring from Phase 2 onward)
```

---

## 11. Design Principles

```
1. ISSUER-AWARE FROM DAY 1
   Even Phase 1 with one issuer should use issuer_id
   Zero refactoring needed when scaling to multi-issuer

2. OBSERVABILITY FROM DAY 1
   Langfuse traces from Phase 1
   Debugging bad retrieval starts with knowing what happened

3. EVALUATION BEFORE IMPROVEMENT
   Run test set before each phase build
   Establish baseline before claiming improvement

4. SEPARATION OF CONCERNS
   Retrieval failure ≠ Generation failure
   Measure them separately, fix them separately

5. TRUST HIERARCHY
   Annual Report (audited) > Allocation Report (verified)
   > Impact Report (often unverified) > Sustainability Report
   When docs contradict, weight by trust level

6. CALIBRATION OVER ACCURACY
   System that says 90% confident and is right 90% of time
   is more valuable than system that says 90% and is right 70%
   Especially critical in regulatory context

7. DOMAIN DRIVES ARCHITECTURE
   Each phase solves a problem felt in previous phase
   Not adding features for the sake of it
   Greenwashing domain naturally motivates each RAG technique

8. TWO MODES, ONE INFRASTRUCTURE
   Q&A and Analysis share same retrieval and storage
   Difference is WHO sets the agenda (user vs system)
   Analysis mode = orchestrated Q&A with predefined agenda

9. AGENT IS NOT A REWRITE
   RAG phases become agent tools
   Evolution not replacement
   Every phase decision should not block agentic future

10. HUMAN IN THE LOOP
    System handles routine, escalates meaningful
    Analyst feedback improves system over time
    Especially for Severity 2+ flags
```

---

## 12. Recommended Starting Point

```
FIRST DOCUMENT:   EIB (European Investment Bank) green bond
                  Allocation Report
                  Reason: Public, well-structured,
                          clear tables, verifiable numbers

FIRST QUESTIONS:
  "What is the total amount allocated?"
  "What project categories received funding?"
  "What percentage went to renewable energy?"
  "Is there any unallocated amount?"

FIRST EVALUATION:
  Answer these manually first
  Those answers = your ground truth
  Phase 1 success = system matches your manual answers

DOMAIN READING (before coding):
  ICMA Green Bond Principles (2 hours, free PDF)
  EU Green Bond Standard summary (2 hours, EUR-Lex)
  Read one full EIB document set (4 hours)
  Talk to one CBI colleague who reviews these (1 hour)
```

---

## 13. The North Star Output

```
BOND VERIFICATION SUMMARY
─────────────────────────────────────────────
Issuer:          [Name]
ISIN:            [ISIN]
Issue Date:      [Date]
Amount:          [€M]
Framework:       [Version]
─────────────────────────────────────────────
VERIFICATION STATUS
  Money Trail:         ✓ PASS / ⚠ FLAG / ✗ FAIL
  Impact Trail:        ✓ PASS / ⚠ FLAG / ✗ FAIL
  Framework Trail:     ✓ PASS / ⚠ FLAG / ✗ FAIL
  Issuer Integrity:    ✓ PASS / ⚠ FLAG / ✗ FAIL
─────────────────────────────────────────────
FLAGS IDENTIFIED
  [CRITICAL]    Description → Source → Contradicting source
  [SIGNIFICANT] Description → Source → Contradicting source
  [MODERATE]    Description → Source → Contradicting source
─────────────────────────────────────────────
DOCUMENTS REVIEWED
  ✓ Green Bond Framework v2.1 (2022)
  ✓ SPO - Sustainalytics (2022) - reviewed v2.0 ⚠
  ✓ Base Prospectus (2021)
  ✓ Final Terms (2023)
  ✓ Allocation Report (2023)
  ✗ Impact Report - NOT PUBLISHED ⚠
  ✓ Green Bond Report (2023)
  ✓ Sustainability Report - targeted (2023)
  ✓ Annual Report - targeted (2023)
─────────────────────────────────────────────
OVERALL RISK RATING:    MEDIUM-HIGH
CONFIDENCE:             0.74
HUMAN REVIEW NEEDED:    YES
─────────────────────────────────────────────
```

---

## 14. What This Project Demonstrates

```
TECHNICAL SKILLS:
  RAG architecture (deep, not tutorial-level)
  Hybrid search implementation
  Agentic system design
  LLM orchestration with LangGraph
  Evaluation methodology for AI systems
  Multi-source data modeling

DOMAIN APPLICATION:
  EU green bond market
  Greenwashing detection methodology
  Cross-document verification
  Regulatory context awareness

RARE COMBINATION:
  Most RAG builders: generic domain, no evaluation criteria
  You: specific domain, verifiable ground truth,
       real stakes, real documents, real experts available

  That combination is what separates
  a demo from a trustworthy system.
```

---

**Save this document. Every future session starts here.**

**Next step when ready: Phase 1 scope lock and build start.**
