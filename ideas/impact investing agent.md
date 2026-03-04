# Greenwashing Detection System: Feasibility & Learning Evaluation

## What You're Actually Building

```
User: "Does Blackrock's ESG fund actually reduce carbon exposure 
       compared to their standard S&P500 fund?"

System:
├── Extract: issuer = "Blackrock", fund_type = ["ESG", "S&P500"]
├── Intent: comparative carbon exposure analysis
├── Data needs: structured (holdings %, carbon metrics) + 
│              unstructured (fund prospectus claims)
├── Route: → carbon_metrics_table + blackrock_fund_docs_index
└── Return: "Fund claims 40% lower carbon but holds 23% energy stocks..."
```

---

## Feasibility Assessment

### Technical Complexity Map

```
LAYER 1: Data Ingestion                    [Medium Complexity]
├── ESG Reports (PDFs)                     → PDF parsing
├── Fund Holdings (CSV/API)                → Structured ingestion  
├── Regulatory Filings (SEC EDGAR)         → API + parsing
└── News/Controversies                     → Web scraping

LAYER 2: Storage Architecture              [Medium-High Complexity]
├── Vector DB (Pinecone/Weaviate)          → Unstructured docs
├── Relational DB (Postgres)               → Metrics, holdings
└── Issuer-namespaced indexing             → Key design challenge

LAYER 3: Agentic Router                    [High Complexity - Core Learning]
├── Entity extraction (issuer, fund)       → NER / LLM extraction
├── Intent classification                  → LLM reasoning
├── Query planning                         → Which stores to hit?
└── Multi-source synthesis                 → Answer generation

LAYER 4: Greenwashing Logic               [Domain Complexity]
├── Claim vs. metric comparison            → Gap analysis
├── Controversy cross-referencing          → Signal detection
└── Confidence scoring                     → How greenwashed?
```

---

## Learning Scope - Honest Breakdown

### What You Will Genuinely Master

```
CONCEPT 1: RAG Architecture (Deep)
─────────────────────────────────
Standard RAG:     Query → Embed → Search → Answer
What you build:   Query → Parse → Route → Multi-fetch → Synthesize

Gap you'll fill:  Most tutorials don't teach you WHY routing matters
                  You'll feel the pain of wrong routing = wrong answers
                  That pain = real learning

CONCEPT 2: Agentic Reasoning
─────────────────────────────
You'll implement:
  - Tool selection (which DB to query)
  - Multi-step planning (do I need both structured + unstructured?)
  - Self-correction (if structured returns null, fall back to docs)

This is LangGraph / LangChain agents done with PURPOSE
Not just following a tutorial

CONCEPT 3: Data Modeling for AI Systems
────────────────────────────────────────
How do you structure:
  issuer_id → [documents] → [metrics] → [claims]
  
So that retrieval is fast AND contextually correct?
This is a real engineering skill, not just ML

CONCEPT 4: Prompt Engineering at System Level
──────────────────────────────────────────────
Not "write a better prompt"
But: How do prompts change based on query type?
  
  Carbon query    → prompt includes metric context
  Controversy     → prompt includes news + filing context
  Comparative     → prompt structures side-by-side analysis
```

### Honest Difficulty Warnings

```
⚠️  DATA ACQUISITION
    ESG data is expensive (Bloomberg, MSCI)
    Workaround: Use public SEC filings + free ESG datasets
    Risk: Limited data = limited system capability
    
⚠️  GROUND TRUTH FOR GREENWASHING
    How do you know if your system is RIGHT?
    Greenwashing is often subjective / legal grey area
    You need a validation strategy from day 1
    
⚠️  SCOPE CREEP IS REAL
    "Just add one more issuer"
    "Just add controversy detection"
    "Just add time-series comparison"
    → System becomes unmaintainable
    
⚠️  AGENTIC SYSTEMS FAIL SILENTLY
    Router sends query to wrong index
    Returns confident wrong answer
    You need observability built in early
```

---

## Recommended Build Phases

### Phase 1: Prove the Core Loop (2-3 weeks)
```
Goal: ONE issuer, ONE question type works end-to-end

Data:
  - 1 company (e.g., Tesla or BP - both controversial)
  - Their sustainability report (PDF)
  - Their actual emissions data (structured CSV)

Build:
  [Question] 
      ↓
  [LLM extracts: issuer + intent]
      ↓
  [Simple if/else router: "metrics?" → postgres, "claims?" → vector]
      ↓
  [Fetch from correct store]
      ↓
  [LLM synthesizes answer]

Success metric: 
  "Does BP's 2023 report claim align with their Scope 1 emissions data?"
  → System gives a defensible answer with citations
```

### Phase 2: Multi-Issuer + Real Routing (3-4 weeks)
```
Goal: 5 issuers, router makes intelligent decisions

Add:
  - Issuer-namespaced vector indices
  - Structured metrics table with issuer_id foreign key
  - LLM-based router (not if/else)
  - Fallback logic when one store returns nothing

Key learning moment:
  Router needs to handle ambiguity
  "Is Vanguard ESG fund greenwashing?" 
  → Issuer = Vanguard, but WHICH fund?
  → System must ask for clarification OR search broadly
```

### Phase 3: Greenwashing Intelligence (3-4 weeks)
```
Goal: System doesn't just retrieve, it ANALYZES

Add:
  - Claim extraction from documents
  - Metric extraction from tables  
  - Gap scoring: |claimed_value - actual_value| / actual_value
  - Controversy signals from news index
  - Confidence level in greenwashing assessment

Output format:
  {
    "issuer": "BP",
    "claim": "Net zero by 2050",
    "supporting_evidence": [...],
    "contradicting_evidence": [...],
    "greenwashing_risk_score": 0.73,
    "reasoning": "..."
  }
```

### Phase 4: Scale + Observability (2-3 weeks)
```
Goal: System is trustworthy, not just functional

Add:
  - LangSmith / Langfuse tracing (see every routing decision)
  - Evaluation dataset (20 questions with known answers)
  - Automated accuracy scoring
  - Simple UI (Streamlit is fine)

This phase teaches you:
  How to know if your AI system is getting WORSE over time
  Critical skill almost no tutorial covers
```

---

## Tech Stack Recommendation

```
COMPONENT              CHOICE              WHY
─────────────────────────────────────────────────────
Orchestration          LangGraph           Explicit routing logic
                                           You SEE the agent decisions
                                           
Vector Store           Weaviate            Native multi-tenancy
                       (self-hosted)       = perfect for issuer namespacing
                       
Structured Store       PostgreSQL          You know SQL
                       + pgvector          Hybrid search later
                       
LLM                    GPT-4o              Reliable for entity extraction
                       + GPT-3.5-turbo     Cheaper for simple routing calls
                       
Embeddings             text-embedding-3    Cost effective, good quality
                       -small              
                       
Data Sources           SEC EDGAR API       Free, legitimate, structured
                       + PDF reports       Manual download initially
                       
Observability          Langfuse            Free tier, excellent tracing
(add early)            
                       
UI                     Streamlit           Fast, focus on backend learning
```

---

## The Real Learning Value Proposition

```
WHAT MOST PEOPLE BUILD:          WHAT YOU'LL BUILD:
──────────────────────────────────────────────────────
Single-index RAG                 Multi-index routed RAG
"Chat with PDF"                  "Analyze claims vs. evidence"
No domain logic                  Greenwashing gap analysis
No observability                 Full decision tracing
Tutorial following               Problem-driven architecture

SKILLS THAT TRANSFER:
├── Any domain with structured + unstructured data mix
│   (healthcare, legal, finance, compliance)
├── Agentic routing patterns
│   (reusable in any multi-source system)
├── Evaluation methodology
│   (how to measure if RAG system is correct)
└── Data modeling for AI
    (often the hardest, least taught skill)
```

---

## My Honest Recommendation

```
BUILD IT. But with these constraints:

✅ Start with 2 issuers maximum in Phase 1
✅ Define "greenwashing" operationally before writing code
   Example: "Claim in doc says X, metric table shows Y, gap > 20% = flag"
✅ Build observability in Week 1, not Week 10
✅ Set a hard scope boundary: 
   "V1 covers only carbon/emissions claims"
   Not all ESG dimensions at once

The domain is genuinely interesting, legally relevant,
and has enough public data to build something real.

The routing architecture problem is non-trivial enough
that solving it will teach you things tutorials won't.

This is a portfolio project that demonstrates:
  - Systems thinking
  - AI engineering (not just AI usage)
  - Domain application
  
That combination is rare and valuable.
```

---

**Biggest risk isn't technical - it's defining done.** 

Greenwashing detection could be a PhD thesis or a weekend project depending on scope. Pin down your V1 definition before anything else, and the rest becomes an engineering problem you can solve systematically.

Want me to go deeper on any specific phase or the data architecture design?



