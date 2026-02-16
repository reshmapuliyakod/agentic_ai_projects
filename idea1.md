

# Multi-Modal Adaptive RAG — EU Market Regulations

## Why EU Regulations is a Perfect Domain for This

```
1. Documents are PDFs (official journals, directives)
2. Heavy cross-referencing between regulations
3. Mix of structured data (tables, annexes) and dense legal text
4. Real enterprise demand (compliance teams pay for this)
5. Publicly available documents (no data access issues)
```

---

## Pick One Regulation as Entry Point

### **MiFID II (Markets in Financial Instruments Directive)**

```
Why MiFID II:
- Central to EU financial regulation
- Links to EVERYTHING
- Mix of text + tables + annexes
- Multiple document types (directive, regulation, technical standards)
- Compliance teams struggle with it daily
```

---

## The Regulation Web (MiFID II as Hub)

```
                         ┌──────────────┐
                         │   MAR        │ (Market Abuse Regulation)
                         │  596/2014    │
                         └──────┬───────┘
                                │ insider dealing, market manipulation
                                │
         ┌──────────────┐       │       ┌──────────────────┐
         │  EMIR        │       │       │  SFDR            │
         │  648/2012    ├───────┤───────┤  2019/2088       │
         │ (derivatives)│       │       │ (sustainability) │
         └──────────────┘       │       └──────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │      MiFID II         │
                    │    2014/65/EU         │
                    │    (Directive)        │
                    │         +             │
                    │      MiFIR            │
                    │    600/2014           │
                    │    (Regulation)       │
                    └───────────┬───────────┘
                                │
         ┌──────────────┐       │       ┌──────────────────┐
         │  PRIIPs      │       │       │  UCITS / AIFMD   │
         │  1286/2014   ├───────┤───────┤  (Fund mgmt)     │
         │ (retail inv) │       │       │                  │
         └──────────────┘       │       └──────────────────┘
                                │
                         ┌──────┴───────┐
                         │  CRD/CRR     │
                         │  (Capital    │
                         │  Requirements│
                         └──────────────┘

         + DORA (Digital Operational Resilience)
         + AML Directives (6AMLD)
         + Benchmark Regulation (BMR)
         + CSDR (Settlement)
```

---

## Document Corpus

```python
corpus = {
    # Core
    "MiFID II Directive": "2014/65/EU",
    "MiFIR": "Regulation 600/2014",
    
    # Delegated/Implementing acts (the real detail)
    "MiFID II Delegated Reg": "2017/565",    # organizational requirements
    "RTS 1": "2017/587",                      # transparency equities
    "RTS 2": "2017/583",                      # transparency non-equities
    "RTS 25": "2017/580",                     # order record keeping
    
    # Linked regulations
    "MAR": "596/2014",
    "EMIR": "648/2012",
    "SFDR": "2019/2088",
    "PRIIPs": "1286/2014",
    "DORA": "2022/2554",
    "BMR": "2016/1011",
    
    # ESMA Guidelines & Q&As
    "ESMA MiFID II Q&A": "esma35-43-349",
    "ESMA Guidelines on suitability": "esma35-43-3172",
}

# Source: EUR-Lex (https://eur-lex.europa.eu) — all freely available as PDF/HTML
```

---

## System Architecture (Regulation-Specific)

```
User: "What are the pre-trade transparency requirements 
       for bonds under MiFID II, and how does EMIR affect 
       OTC bond derivatives reporting?"

    ↓

┌─────────────────────────────────┐
│        Query Analyzer           │
│                                 │
│  Entities: [pre-trade transparency, bonds, EMIR, OTC derivatives]
│  Regulations detected: [MiFID II/MiFIR, EMIR]
│  Query type: CROSS-REGULATION + STRUCTURED DATA
│  Needs: regulatory text + tables (thresholds) + linkage
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│        Strategy Router          │
│                                 │
│  1. Vector search → MiFIR text on pre-trade transparency
│  2. Table query  → RTS 2 threshold tables for bonds
│  3. Graph query  → MiFID II ←→ EMIR linkage
│  4. Vector search → EMIR text on OTC reporting
└────────┬───────┬────────┬───────┘
         ↓       ↓        ↓
     ┌───┴──┐ ┌──┴───┐ ┌──┴────┐
     │Vector│ │Table │ │Graph  │
     │Store │ │Store │ │Store  │
     │      │ │      │ │       │
     │MiFIR │ │RTS 2 │ │Reg    │
     │Art 8 │ │Annex │ │Links  │
     │Art 10│ │III   │ │       │
     └───┬──┘ └──┬───┘ └──┬────┘
         ↓       ↓        ↓
┌─────────────────────────────────┐
│        Result Fusion            │
│                                 │
│  Text: MiFIR Articles 8,10 on pre-trade transparency
│  Table: SSTI/LIS thresholds for bond subcategories  
│  Graph: MiFIR Art 26 → links to EMIR Art 9 (reporting)
│  Text: EMIR Art 9 on trade reporting obligations
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│        Answer + Citations       │
│                                 │
│  "Pre-trade transparency for bonds is governed by 
│   MiFIR Articles 8 and 10, with specific thresholds
│   defined in RTS 2 Annex III:
│   
│   [TABLE: Bond category thresholds]
│   
│   For OTC bond derivatives, EMIR Article 9 requires
│   reporting to trade repositories. The link between
│   MiFIR and EMIR is established through MiFIR Art 26
│   which cross-references EMIR reporting obligations..."
│   
│   Sources: MiFIR Art 8, 10, 26 | RTS 2 Annex III | EMIR Art 9
└─────────────────────────────────┘
```

---

## Ingestion Pipeline (The Hard Part)

### Step 1: Document Processing

```python
# For each regulation PDF:

def ingest_regulation(pdf_path, regulation_id):
    
    # 1. Extract text with structure preservation
    articles = extract_articles(pdf_path)        # Article-level chunking
    recitals = extract_recitals(pdf_path)        # Preamble/context
    annexes = extract_annexes(pdf_path)          # Often contain tables
    
    # 2. Extract tables
    tables = extract_tables(pdf_path)            # Your adaptive PDF parser
    for table in tables:
        store_in_duckdb(table, regulation_id)    # Structured store
    
    # 3. Extract cross-references
    references = extract_cross_references(articles)
    # "as referred to in Article 4(1)(15) of Directive 2014/65/EU"
    # → parse into: {source: "EMIR Art 9", target: "MiFID II Art 4(1)(15)"}
    
    # 4. Chunk and embed
    chunks = chunk_articles(articles, chunk_size=512, overlap=50)
    embeddings = embed(chunks)
    store_in_vector_db(chunks, embeddings, metadata={
        "regulation": regulation_id,
        "article": article_num,
        "type": "directive|regulation|RTS|ITS|guideline"
    })
    
    # 5. Build knowledge graph edges
    for ref in references:
        add_graph_edge(
            source=ref.source_article,
            target=ref.target_article,
            relationship=ref.type  # "amends", "refers_to", "repeals", "supplements"
        )
```

### Step 2: Cross-Reference Extraction (Key Differentiator)

```python
# EU regulations are FULL of cross-references

patterns = [
    r"Article \d+(\(\d+\))? of Regulation \(EU\) (\d+/\d+)",
    r"as referred to in Directive (\d+/\d+/EU)",
    r"in accordance with Article \d+ of \[regulation\]",
    r"pursuant to paragraph \d+ of Article \d+",
    r"Regulation \(EU\) No (\d+/\d+) is amended as follows",
]

# Example from MiFIR Article 26:
# "...shall report to trade repositories registered under 
#  Article 55 of Regulation (EU) No 648/2012..."
# 
# → Edge: MiFIR_Art26 --refers_to--> EMIR_Art55
```

### Step 3: Knowledge Graph

```python
# Neo4j or NetworkX

# Nodes
(MiFID_II, {type: "Directive", id: "2014/65/EU", topic: "investment services"})
(MiFIR, {type: "Regulation", id: "600/2014", topic: "markets"})
(EMIR, {type: "Regulation", id: "648/2012", topic: "derivatives"})
(RTS_2, {type: "Technical Standard", id: "2017/583", topic: "non-equity transparency"})

# Article-level nodes
(MiFIR_Art8, {regulation: "MiFIR", article: 8, topic: "pre-trade transparency"})
(MiFIR_Art26, {regulation: "MiFIR", article: 26, topic: "transaction reporting"})
(EMIR_Art9, {regulation: "EMIR", article: 9, topic: "reporting obligation"})

# Edges
(MiFIR_Art26) --[REFERS_TO]--> (EMIR_Art9)
(RTS_2) --[SUPPLEMENTS]--> (MiFIR_Art8)
(MiFID_II) --[AMENDED_BY]--> (MiFID_II_Quick_Fix)  # 2021 amendments
(DORA) --[AMENDS]--> (MiFID_II_Art16)               # ICT requirements
```

---

## Query Types This Handles

```
TYPE 1: Single regulation lookup
  "What is best execution under MiFID II?"
  → Vector search on MiFID II chunks → Article 27

TYPE 2: Threshold/data lookup  
  "What is the LIS threshold for sovereign bonds?"
  → Table query on RTS 2 Annex III → exact number

TYPE 3: Cross-regulation
  "How does DORA change MiFID II requirements?"
  → Graph traversal (DORA→MiFID II edges) + vector search both

TYPE 4: Compliance question
  "We're an investment firm trading OTC derivatives. 
   What reporting obligations do we have?"
  → Multi-hop: MiFID II (firm classification) → MiFIR (reporting) 
    → EMIR (derivative reporting) → table (thresholds)

TYPE 5: Comparison
  "How do pre-trade transparency rules differ between 
   equities (RTS 1) and bonds (RTS 2)?"
  → Parallel vector search on both + table comparison
```

---

## Self-Correction (Regulation-Specific)

```python
def verify_regulatory_answer(query, answer, sources):
    checks = {
        # 1. Are cited articles real?
        "citation_valid": verify_article_exists(answer.citations),
        
        # 2. Is the regulation still in force? (not repealed/amended)
        "current": check_regulation_status(answer.citations),
        
        # 3. Cross-reference consistency
        "consistent": check_no_contradictions(answer, sources),
        
        # 4. Did we miss a linked regulation?
        "complete": check_graph_for_missing_links(query.entities),
        #  → "You answered about MiFIR transparency but didn't 
        #     mention RTS 2 thresholds — re-routing to include table data"
    }
    return checks
```

---

## Practical Build Order

```
Week 1-2: Ingestion
  ├── Download 5-6 core regulation PDFs from EUR-Lex
  ├── Parse into articles (text chunks)
  ├── Extract tables from annexes
  ├── Store in ChromaDB + DuckDB
  └── Basic RAG working on single regulation

Week 3: Cross-References
  ├── Regex + LLM extraction of cross-references
  ├── Build knowledge graph (even NetworkX is fine)
  ├── Query: "What regulations link to MiFID II Art 27?"
  └── Graph traversal working

Week 4: Adaptive Routing
  ├── Query classifier (text vs table vs graph vs hybrid)
  ├── Route to correct retrieval
  ├── Test with 30 regulation queries
  └── Measure routing accuracy

Week 5: Multi-Hop + Fusion
  ├── Cross-regulation queries working
  ├── Result fusion from multiple sources
  ├── Self-correction loop
  └── Evaluation pipeline

Week 6: Polish
  ├── Citation formatting (Article X of Regulation Y)
  ├── Confidence scores
  ├── "Related regulations you should also check" feature
  └── Simple UI
```

---

## Tech Stack

```
PDF Parsing:     PyMuPDF + pdfplumber + camelot (tables)
Vector Store:    ChromaDB (start) → Qdrant (scale)
Table Store:     DuckDB
Knowledge Graph: NetworkX (start) → Neo4j (scale)
Embeddings:      OpenAI text-embedding-3-small (or BGE-large for local)
LLM:             GPT-4o (or Claude for long context — regulations are LONG)
Text-to-SQL:     LLM-generated with schema context
Framework:       LangGraph for agent orchestration
Eval:            RAGAS + custom regulatory accuracy metrics
```

---

## What Makes This Portfolio-Worthy

```
✓ Domain expertise signal    — "I understand financial regulation"
✓ Technical depth            — multi-modal retrieval, KG, self-correction
✓ Real-world applicability   — compliance teams actually need this
✓ Measurable                 — accuracy on regulatory queries is testable
✓ Extensible                 — add more regulations, more jurisdictions
✓ Combines all your ideas    — adaptive PDF parsing + agentic RAG + curation
```

**Start with MiFID II + MiFIR + RTS 1 + RTS 2 + EMIR. Five documents. Prove it works. Then expand.**




# Prospectus Regulation (EU) 2017/1129 — Assessment

## Is It Simple?

**Relatively yes. Good starting point. Here's why:**

```
✓ Single core regulation (not split into Directive + Regulation like MiFID)
✓ Well-structured — clear articles, numbered sections
✓ Manageable size — 109 articles + annexes
✓ Practical use case — "Can I raise capital? What do I need to disclose?"
✓ Clear linked regulations (not a sprawling web like MiFID II)
✗ BUT — delegated acts and annexes contain dense tables/templates
✗ AND — cross-references to other regulations still exist
```

---

## The Document Set

```
CORE:
├── Prospectus Regulation (EU) 2017/1129          ← main regulation
│
DELEGATED ACTS (the real detail):
├── Delegated Reg (EU) 2019/980                   ← format/content/scrutiny
├── Delegated Reg (EU) 2019/979                   ← technical standards (filing/publication)
│
ANNEXES (where tables live):
├── Annexes to 2019/980                           ← disclosure templates
│   ├── Annex 1:  Registration doc — equity
│   ├── Annex 3:  Registration doc — debt (wholesale)
│   ├── Annex 6:  Registration doc — debt (retail)
│   ├── Annex 11: Securities note — equity
│   ├── Annex 14: Securities note — debt
│   ├── Annex 20: EU Growth prospectus
│   └── ... (29 annexes total)
│
LINKED REGULATIONS:
├── MAR 596/2014              ← inside information in prospectus
├── MiFID II 2014/65/EU       ← distribution, investor categorization
├── Transparency Directive     ← ongoing disclosure after listing
├── ECSPR 2020/1503           ← crowdfunding exemption
└── EU Listing Act 2024       ← RECENT amendments to Prospectus Reg
```

```
Total corpus: ~8-10 documents vs MiFID II's 30+
Much more contained.
```

---

## Why Prospectus Reg is Great for This Project

### Real Query Diversity

```
TEXT QUERIES (vector search):
  "What are the exemptions from publishing a prospectus?"
  → Article 1(4), 1(5) — exemption thresholds
  
  "What is the universal registration document?"
  → Articles 9-12

TABLE QUERIES (structured):
  "What disclosures are required for a retail debt prospectus?"
  → Annex 6 of 2019/980 — checklist table
  
  "What are the size thresholds for prospectus exemptions?"
  → Article 1(3) — €8M threshold, member state options table

CROSS-REGULATION QUERIES (graph):
  "If I'm issuing securities, what MiFID II obligations apply 
   to the distributors?"
  → Prospectus Reg → MiFID II product governance
  
  "How does MAR affect what I put in a prospectus?"
  → Prospectus Reg Art 6 → MAR Art 7 (inside information)

MULTI-HOP QUERIES:
  "I'm a small company wanting to list on an SME growth market. 
   What simplified prospectus can I use and what are the 
   ongoing obligations after listing?"
  → Prospectus Reg Art 15 (EU Growth) → Annex 20 (template) 
    → Transparency Directive (ongoing) → MiFID II (SME market def)
```

---

## How Agents Come Into the Picture

### This is where people get confused. Let me be concrete.

**An agent is NOT just an LLM call. An agent is an LLM that:**
```
1. Reasons about what to do
2. Picks tools/actions
3. Observes results
4. Decides next step
5. Can backtrack and retry
```

### Agent Architecture for Prospectus Reg

```
┌─────────────────────────────────────────────────┐
│                 ORCHESTRATOR AGENT               │
│                                                  │
│  "I received a query. Let me think about         │
│   what I need to do step by step."               │
│                                                  │
│  Tools available:                                │
│   - vector_search(query, regulation_filter)      │
│   - table_lookup(regulation, annex, criteria)    │
│   - graph_traverse(start_node, relationship)     │
│   - text_to_sql(natural_language_query)          │
│   - verify_answer(answer, sources)               │
│   - ask_user_clarification(question)             │
└──────────────────────┬──────────────────────────┘
                       │
                       ↓
              Agent Reasoning Trace
```

### Concrete Agent Trace Example

```
USER: "We're a company with €50M market cap wanting to raise 
       €5M through a public offer of shares in France. 
       What type of prospectus do we need?"

AGENT THINKING:
─────────────────────────────────────────────────
Step 1: Parse the query
  - Entity: company, €50M market cap, €5M raise, shares, France, public offer
  - This is a CLASSIFICATION question — which prospectus regime applies
  - I need to check: exemptions first, then prospectus type

Step 2: Check if prospectus is even needed
  ACTION: vector_search("prospectus exemption thresholds public offer")
  RESULT: Article 1(3) — offers < €8M MAY be exempt per member state
          Article 3(2) — offers €1M-€8M, member states can set rules
  
  OBSERVATION: €5M is under €8M. But this depends on France's 
  national implementation. Let me check.

Step 3: Check France-specific threshold
  ACTION: vector_search("France member state prospectus exemption threshold")
  RESULT: Found — France (AMF) sets threshold at €8M. 
          So €5M offer COULD be exempt from full prospectus.
  
  OBSERVATION: But wait — user said "public offer." Even if exempt 
  from prospectus, there may be a simplified disclosure. 
  Also need to check if they want EU passport.

Step 4: Check if they need EU Growth prospectus
  ACTION: vector_search("EU Growth prospectus SME eligibility criteria")
  RESULT: Article 15 — available for SMEs on SME growth markets
          AND for offers not exceeding €20M
  
  ACTION: table_lookup("2019/980", "Annex 20", "EU Growth requirements")
  RESULT: [Table of simplified disclosure items]
  
  OBSERVATION: €50M market cap + €5M raise = eligible for EU Growth 
  prospectus if listing on SME growth market. But if listing on 
  regulated market, different rules.

Step 5: Need clarification
  ACTION: ask_user_clarification(
    "Are you planning to list on a regulated market or an SME growth 
     market? And do you need to passport the offer to other EU states?"
  )
  
  [If user says: "regulated market, France only"]

Step 6: Determine prospectus type
  ACTION: vector_search("prospectus type regulated market equity")
  RESULT: Full prospectus required for regulated market admission
          Article 6 — content requirements
          Must use Annex 1 (registration) + Annex 11 (securities note)
  
  ACTION: table_lookup("2019/980", "Annex 1", "all items")
  RESULT: [Full disclosure checklist — 20+ items]

Step 7: Check linked obligations
  ACTION: graph_traverse("Prospectus_Reg_Art6", "REFERS_TO")
  RESULT: 
    → MAR Art 7: inside information must be consistent
    → MiFID II Art 16(3): product governance for distributors
    → Transparency Directive: ongoing reporting after admission

Step 8: Verify and compile answer
  ACTION: verify_answer(compiled_answer, all_sources)
  RESULT: ✓ Citations valid, ✓ Thresholds current, 
          ⚠ Check: EU Listing Act 2024 may have amended thresholds
  
  ACTION: vector_search("EU Listing Act 2024 prospectus amendments")
  RESULT: Yes — Listing Act raised some thresholds. Include caveat.
─────────────────────────────────────────────────

FINAL ANSWER:
  [Structured response with articles, tables, cross-references, 
   and flag about recent Listing Act amendments]
```

---

## Agent vs No-Agent — Why It Matters Here

```
WITHOUT AGENT (naive RAG):
  Query → embed → retrieve top 5 chunks → generate answer
  
  Problem: Retrieves Article 1 (exemptions) but misses that 
  France has specific thresholds. Doesn't check EU Growth option.
  Doesn't flag Listing Act amendments. Gives incomplete answer.

WITH AGENT:
  Query → reason → search → observe → "not enough" → search again 
  → "need table" → get table → "need cross-ref" → traverse graph 
  → "is this current?" → verify → compile
  
  Result: Complete, sourced, cross-referenced answer.
```

**The agent's value is in MULTI-STEP REASONING, not just retrieval.**

---

## Agent Implementation (Practical)

```python
# Using LangGraph (or build your own state machine)

from langgraph.graph import StateGraph

class RegState(TypedDict):
    query: str
    intent: str
    entities: list
    regulations_involved: list
    retrieved_text: list
    retrieved_tables: list
    graph_results: list
    answer: str
    confidence: float
    needs_clarification: bool
    retry_count: int

def query_analyzer(state: RegState) -> RegState:
    """LLM analyzes query, extracts entities, determines regulations"""
    prompt = f"""
    Analyze this regulatory query:
    {state['query']}
    
    Extract:
    1. Key entities (amounts, jurisdictions, instrument types)
    2. Which EU regulations are relevant
    3. What type of information is needed (text/table/cross-reference)
    4. Is this answerable or do we need user clarification
    """
    # LLM call
    ...

def route_decision(state: RegState) -> str:
    """Decide next action based on current state"""
    if state['needs_clarification']:
        return "ask_user"
    if not state['retrieved_text']:
        return "vector_search"
    if state['intent'] == 'threshold_lookup' and not state['retrieved_tables']:
        return "table_search"
    if len(state['regulations_involved']) > 1 and not state['graph_results']:
        return "graph_search"
    if state['answer'] and state['confidence'] < 0.7:
        return "verify_and_retry"
    return "generate_answer"

def vector_search(state: RegState) -> RegState:
    """Search vector store with regulation filter"""
    ...

def table_search(state: RegState) -> RegState:
    """Query DuckDB for structured data"""
    ...

def graph_search(state: RegState) -> RegState:
    """Traverse regulation knowledge graph"""
    ...

def verify(state: RegState) -> RegState:
    """Check answer quality, citations, completeness"""
    ...

# Build graph
workflow = StateGraph(RegState)
workflow.add_node("analyze", query_analyzer)
workflow.add_node("vector_search", vector_search)
workflow.add_node("table_search", table_search)
workflow.add_node("graph_search", graph_search)
workflow.add_node("generate", generate_answer)
workflow.add_node("verify", verify)

workflow.add_conditional_edges("analyze", route_decision)
workflow.add_conditional_edges("verify", route_decision)  # retry loop
```

---

## Build Order (Prospectus Reg Specific)

```
Week 1: Ingest Core Documents
  ├── Download Prospectus Reg + 2019/980 + key annexes from EUR-Lex
  ├── Parse into article-level chunks
  ├── Extract annex tables (disclosure checklists)
  ├── Vector store + DuckDB loaded
  └── Basic RAG: "What does Article 1 say?" works

Week 2: Cross-References + Graph
  ├── Extract all "Article X of Regulation Y" references
  ├── Build knowledge graph
  ├── Add MAR, MiFID II, Transparency Directive (just referenced articles)
  └── Graph query: "What links to Prospectus Reg Art 6?" works

Week 3: Agent Logic
  ├── Query analyzer (classify intent, extract entities)
  ├── Router (vector vs table vs graph vs hybrid)
  ├── Multi-step reasoning chain
  └── Test: complex queries trigger multi-step correctly

Week 4: Self-Correction + Polish
  ├── Verification step
  ├── Retry with different strategy
  ├── Clarification requests when query is ambiguous
  ├── Citation formatting
  └── 30-query evaluation set

Week 5: Expand
  ├── Add EU Listing Act amendments
  ├── Add ESMA Q&As on prospectus
  ├── Member state threshold variations
  └── Demo-ready UI
```

---

## Verdict

```
Prospectus Regulation:
  ✓ Contained scope (8-10 docs vs 30+ for MiFID II)
  ✓ Rich enough for multi-modal (text + tables + cross-refs)
  ✓ Clear agent use case (multi-step reasoning for "do I need a prospectus?")
  ✓ Real-world value (issuers and law firms actually need this)
  ✓ Natural expansion path (add MiFID II, MAR later)
  ✓ Good first regulation to prove the system works

Start here. Nail it. Then expand to MiFID II.
```
# Value for Supervisors (NCAs like AMF, BaFin, ESMA)

## Their Pain Today

```
Supervisors manually review prospectuses against regulation.
Hundreds of filings. Dense documents. Cross-checking against 
multiple regulations. Slow, error-prone, inconsistent.
```

## How This Tool Helps

### 1. **Completeness Check**
```
"Does this prospectus cover all required disclosures 
 under Annex 1 of 2019/980?"

Agent: Checks filing against disclosure checklist table
→ "Missing items: risk factors related to ESG (item 3.4), 
   working capital statement (item 11.1)"
```

### 2. **Cross-Reference Validation**
```
"Does this prospectus comply with MAR requirements 
 on inside information?"

Agent: Traverses graph → MAR Art 7 → checks prospectus content
→ Flags inconsistencies
```

### 3. **Threshold Monitoring**
```
"This issuer claims exemption under Article 1(3). 
 Is the offer size actually below the threshold?"

Agent: Table lookup → member state threshold → validates claim
```

### 4. **Consistency Across Filings**
```
"How does this prospectus compare to similar equity 
 prospectuses approved last quarter?"

Agent: Cross-filing comparison → flags outliers in disclosure quality
```

### 5. **Regulatory Change Impact**
```
"The Listing Act just amended Article 14. 
 Which approved prospectuses are affected?"

Agent: Graph traversal → identifies all filings referencing Art 14
→ Flags ones needing review
```

### 6. **Speed**
```
What takes a reviewer 2-3 days of manual cross-checking 
→ preliminary assessment in minutes
Supervisor still makes final call, but starts from a structured gap analysis
```

---

## One Line Summary

**Turns supervisors from manual document reviewers into exception-based decision makers — the tool flags issues, they make judgments.**
Interms 
<img width="891" height="19384" alt="image" src="https://github.com/user-attachments/assets/a3264789-331c-4e81-a40e-c59ea5cd581f" />


# What You Need to Know — Honest Skill Map

## Core Skills (Non-Negotiable)

### 1. Python (Intermediate+)
```
You need:
  - Comfortable with classes, async, error handling
  - File I/O, data manipulation
  - API consumption (REST)
  - Package management, project structure

Don't need:
  - Advanced CS, algorithms
  - Web frameworks (not yet)
```

### 2. PDF Parsing
```
Learn:
  - PyMuPDF (fitz) — text extraction, metadata
  - pdfplumber — table extraction
  - camelot — complex tables

How deep:
  - 2-3 days experimenting with real regulation PDFs
  - Understand why different tools fail on different PDFs
  - Handle: multi-column, headers/footers, annexes
```

### 3. Embeddings & Vector Search
```
Learn:
  - What embeddings are (conceptually)
  - Chunking strategies (fixed, semantic, article-level)
  - Similarity search (cosine similarity)
  - One vector DB: ChromaDB (simplest to start)

Resources:
  - OpenAI embeddings docs (1 hour)
  - ChromaDB quickstart (1 hour)
  - Experiment: embed 20 articles, query them, see what works
```

### 4. Basic RAG Pipeline
```
Learn:
  - Ingest → Chunk → Embed → Store → Retrieve → Generate
  - Prompt engineering for answer generation
  - Context window management
  - Citation/source tracking

How:
  - Build a naive RAG on one regulation first
  - Use LangChain OR build from scratch (I'd recommend scratch first)
  - 3-4 days to get basic version working
```

### 5. LLM API Usage
```
Learn:
  - OpenAI API (chat completions, function calling)
  - OR Anthropic API (Claude)
  - Prompt engineering (system prompts, few-shot)
  - Structured output (JSON mode, tool calling)

Don't need (yet):
  - Fine-tuning
  - Running local models
  - Training anything
```

---

## Secondary Skills (Needed for Full System)

### 6. SQL Basics
```
Why: Tables from PDFs go into DuckDB, queried via text-to-SQL

Learn:
  - SELECT, WHERE, JOIN, GROUP BY, aggregations
  - DuckDB Python API
  - 1-2 days if you don't know SQL
```

### 7. Knowledge Graphs (Basic)
```
Why: Cross-regulation references

Learn:
  - NetworkX basics (nodes, edges, traversal)
  - Graph concepts: nodes, relationships, properties
  - NOT Neo4j yet — NetworkX is enough to start

How deep:
  - 1 day tutorial + experimentation
  - Build a small graph: 5 regulations, 20 cross-references
```

### 8. LangGraph (Agent Framework)
```
Why: Orchestrates the multi-step agent logic

Learn:
  - State machines concept
  - LangGraph: nodes, edges, conditional routing
  - Tool calling pattern

Alternative:
  - Build your own with a while loop + LLM decisions
  - Honestly simpler for learning, LangGraph for production

Resources:
  - LangGraph docs + tutorials (2-3 days)
```

### 9. Regex (Intermediate)
```
Why: Extracting cross-references from legal text

"Article 4(1)(15) of Directive 2014/65/EU"
→ You need to parse this reliably

Learn:
  - Capture groups, non-greedy matching, alternation
  - 1 day focused practice
```

---

## You Do NOT Need (Common Misconceptions)

```
✗ Machine Learning / model training
✗ Deep NLP theory
✗ Kubernetes / cloud deployment (not yet)
✗ Frontend frameworks (CLI or Streamlit is fine)
✗ Fine-tuning LLMs
✗ Building embeddings from scratch
✗ Graph neural networks
✗ Advanced math
```

---

## Learning Path (Ordered)

```
Week 1: Foundations
├── Day 1-2: PDF parsing (PyMuPDF + pdfplumber)
│            Download Prospectus Reg, extract text + tables
├── Day 3-4: Embeddings + ChromaDB
│            Chunk articles, store, query
├── Day 5-7: Basic RAG pipeline
│            Query → retrieve → LLM answer
│            Working prototype on single regulation
│
Week 2: Structured Data
├── Day 1-2: DuckDB + extracted tables
│            Store annex tables, write SQL queries
├── Day 3-4: Text-to-SQL via LLM
│            Natural language → SQL → result
├── Day 5:   Combine vector + table retrieval
│            Router picks one based on query
│
Week 3: Graph + Cross-References
├── Day 1-2: Regex extraction of cross-references
├── Day 3-4: NetworkX graph, traversal queries
├── Day 5-7: Integrate graph as third retrieval mode
│
Week 4: Agent Logic
├── Day 1-3: LangGraph or custom agent loop
│            Multi-step reasoning, tool selection
├── Day 4-5: Self-correction / verification
├── Day 6-7: Evaluation on 30 test queries
│
Week 5: Polish + Expand
├── Add more regulations (MAR, MiFID II links)
├── Streamlit UI
├── Demo-ready
```

---

## Resources (Specific, No Fluff)

```
PDF Parsing:
  - pdfplumber docs + examples on GitHub
  - "Extracting tables from PDFs" — camelot docs

RAG:
  - "Building RAG from scratch" — any LangChain tutorial (then ditch LangChain)
  - RAGAS docs (for evaluation later)

Agents:
  - LangGraph official tutorials (langgraph.dev)
  - "Building Effective Agents" — Anthropic blog post (best one out there)

Vector DBs:
  - ChromaDB getting started (15 min)

EU Regulations:
  - EUR-Lex (eur-lex.europa.eu) — download actual PDFs
  - ESMA website — guidelines, Q&As
```

---

## Honest Assessment

```
If you know Python well:          4-5 weeks to working prototype
If you're learning Python too:    8-10 weeks
If you've built basic RAG before: 2-3 weeks to add adaptive + agents

Hardest parts (in order):
  1. PDF table extraction that actually works
  2. Getting the agent routing logic right
  3. Cross-reference extraction from legal text
  4. Evaluation — knowing if your system is actually good
```

**Start building on Day 1. Don't spend 3 weeks "learning" before touching code. Parse one PDF, embed it, query it. Then iterate.**
