# Phased RAG Learning Roadmap: Greenwashing Detection

## The Core Idea First

```
Every phase answers ONE question:
"What was wrong with the previous phase, and how do we fix it?"

That's the learning loop.
```

---

## Phase 1: Naive RAG — Get Something Working

### What It Is
```
Question → Embed → Search Vector Store → Retrieve Chunks → LLM → Answer
```

### The Mental Model
```
You have a book (BP Sustainability Report)
You highlight important paragraphs (chunks)
You search by meaning similarity (vector search)
LLM reads the highlights and answers

That's it. Nothing more.
```

### What You Learn
```
- How chunking decisions affect answer quality
- What embeddings actually represent
- How vector similarity search works
- Where naive RAG breaks badly
```

### Where It Breaks (This Motivates Phase 2)
```
User: "What was BP's Scope 1 emissions in 2022 vs 2021?"

Naive RAG returns:
→ Paragraphs TALKING ABOUT emissions
→ NOT the actual numbers from the data table

Why? Tables don't embed well as text.
Numbers lose meaning when treated as prose.
```

---

## Phase 2: Hybrid Search — Structured Meets Unstructured

### What Changes
```
Before:  Question → Vector Search Only → Chunks
After:   Question → Vector Search  ─┐
                  → Keyword Search ─┤→ Merge & Rerank → Chunks
                  → SQL Query      ─┘
```

### The Mental Model
```
Two types of information in ESG documents:

CLAIMS (unstructured)          METRICS (structured)
"We are committed to           Scope 1: 45.2 MT CO2
 net zero by 2050"             Scope 2: 12.1 MT CO2
→ Vector search finds this     → SQL/keyword finds this

Greenwashing detection NEEDS BOTH.
The gap between claims and metrics IS the greenwashing signal.
```

### What You Learn
```
- BM25 keyword search vs semantic vector search
- When each approach wins
- Reciprocal Rank Fusion (how to merge two result lists)
- Storing structured data alongside unstructured
- The retrieval problem is really a data modeling problem
```

### Where It Breaks (This Motivates Phase 3)
```
User: "Is BP actually reducing emissions or just claiming to?"

This is NOT one question. It's three:
1. What are BP's emission reduction claims?
2. What do the actual metrics show year over year?
3. What's the gap between 1 and 2?

A single retrieval call can't answer this well.
```

---

## Phase 3: Query Intelligence — Understanding What's Really Being Asked

### Three Techniques, One Goal

```
Goal: Transform a vague user question into precise retrieval operations
```

#### 3A: Query Translation
```
What it is:
User question is often ambiguous or poorly worded
Rewrite it into a better search query before retrieval

Example:
User asks:  "Is BP greenwashing?"
Translated: "BP carbon emission reduction targets vs actual performance"
            "BP fossil fuel investment vs renewable investment ratio"
            "BP sustainability claims regulatory violations"

Why it matters:
The words users use ≠ the words in documents
Translation bridges that gap
```

#### 3B: Query Decomposition
```
What it is:
Break one complex question into multiple focused sub-questions
Answer each independently, then synthesize

Example:
"Is BP greenwashing on climate?"
        ↓
Sub-question 1: "What climate commitments has BP made?"
Sub-question 2: "What are BP's actual emission trends 2019-2023?"
Sub-question 3: "How much is BP investing in fossil fuels vs renewables?"
Sub-question 4: "Has BP faced regulatory action on climate claims?"
        ↓
Synthesize all answers → Final verdict

Why it matters:
Complex questions need multiple retrieval passes
One query = one perspective = incomplete answer
```

#### 3C: Query Construction
```
What it is:
Convert natural language into structured queries
for your structured data store

Example:
User asks: "Show me emission trends"
Constructed: SELECT year, scope1, scope2, scope3 
             FROM bp_metrics 
             WHERE metric_type = 'emissions'
             ORDER BY year DESC

Why it matters:
Some answers live in databases not documents
LLM needs to generate the right query, not just search
```

### What You Learn
```
- LLMs as query planners, not just answer generators
- Step-back prompting (zoom out before zooming in)
- HyDE (generate hypothetical answer, use it as search query)
- Text-to-SQL fundamentals
- Multi-query retrieval patterns
```

### Where It Breaks (This Motivates Phase 4)
```
Not every question needs all this machinery.

"What page is the emissions table on?" → Simple retrieval is fine
"Is this entire fund greenwashing?"    → Needs full decomposition

Using heavy machinery for simple questions = slow + expensive
System needs to decide its own strategy
```

---

## Phase 4: Adaptive RAG — System Chooses Its Own Strategy

### What Changes
```
Before:  Every question → same retrieval pipeline
After:   Question → Classify complexity → Choose strategy → Execute
```

### The Mental Model
```
                    ┌─ Simple ──→ Direct retrieval
Question → Router ──┤
                    ├─ Analytical → Decompose → Multi-retrieval  
                    │
                    └─ Comparative → Parallel retrieval → Merge
```

### Decision Logic
```
SIMPLE FACTUAL          ANALYTICAL              COMPARATIVE
"What is BP's           "Is BP reducing         "How does BP compare
 2023 Scope 1?"          emissions?"             to Shell on climate?"
      ↓                       ↓                        ↓
Single vector           Decompose into          Parallel retrieval
search                  sub-questions           from both issuers
                        + SQL for metrics       + side-by-side synthesis
```

### What You Learn
```
- Query complexity classification
- Conditional pipeline execution
- LangGraph for branching workflows (this is where it gets real)
- Cost vs quality tradeoffs in retrieval
- The system is now making decisions, not just executing them
```

### Where It Breaks (This Motivates Phase 5)
```
System retrieves confidently but retrieves WRONG things.

Example:
User: "What is BP's net zero commitment?"
System retrieves: Shell's net zero page (wrong issuer)
LLM answers confidently with wrong information

Or:
Retrieved chunks don't actually contain enough to answer
LLM hallucinates the gap

System needs to CHECK its own work.
```

---

## Phase 5: Self-Corrective RAG — System Evaluates Its Own Answers

### What Changes
```
Before:  Retrieve → Generate → Return answer
After:   Retrieve → Grade retrieval → 
         If bad: re-retrieve or web search →
         Generate → Grade answer →
         If bad: regenerate or flag →
         Return answer with confidence
```

### The Mental Model
```
Three checkpoints:

CHECKPOINT 1: Is what I retrieved relevant?
  Retrieved chunks about BP's history
  Question was about BP's emissions
  → Relevance score low → try different query

CHECKPOINT 2: Is my answer grounded in what I retrieved?
  Answer says "BP committed to 2040 net zero"
  Retrieved chunks say "2050 net zero"
  → Hallucination detected → regenerate

CHECKPOINT 3: Does my answer actually address the question?
  Question: "Is BP greenwashing?"
  Answer: "BP has a sustainability report"
  → Doesn't answer → flag as insufficient
```

### The CRAG Pattern (Corrective RAG)
```
Retrieve
   ↓
Grade each chunk: Relevant / Irrelevant / Ambiguous
   ↓
If mostly irrelevant → rewrite query → retrieve again
If ambiguous → supplement with web search
If relevant → proceed to generation
   ↓
Generate answer
   ↓
Grade answer against source chunks
   ↓
If hallucination detected → regenerate with stricter prompt
If answer is vague → decompose and retry
   ↓
Return answer + confidence score + sources
```

### What You Learn
```
- LLM-as-judge pattern (using LLM to evaluate LLM output)
- Hallucination detection strategies
- Feedback loops in AI pipelines
- When to escalate vs when to answer
- This is where RAG becomes trustworthy, not just functional
```

---

## Phase 6: Agentic Router — Full System

### What Changes
```
Before:  One issuer, one document type, fixed pipeline
After:   Any issuer, any question, system plans its own approach
```

### The Full Picture
```
User Question
      ↓
[Entity Extraction]     → Who are we talking about? (BP, Shell, Vanguard ESG?)
      ↓
[Intent Classification] → What do they really want?
      ↓
[Strategy Selection]    → Which Phase 1-5 approach fits?
      ↓
[Issuer-Specific Index] → Route to correct data namespace
      ↓
[Multi-Source Retrieval]→ Docs + Metrics + News + Filings
      ↓
[Self-Correction Loop]  → Is this answer good enough?
      ↓
[Greenwashing Analysis] → Claims vs Evidence gap scoring
      ↓
Structured Answer with confidence + citations
```

---

## The Learning Progression Summarized

```
PHASE    CORE CONCEPT              KEY INSIGHT YOU GAIN
─────────────────────────────────────────────────────────────
1        Basic RAG                 Retrieval quality = answer quality
                                   Chunking is a real design decision

2        Hybrid Search             Different data needs different retrieval
                                   Structured + unstructured must coexist

3        Query Intelligence        User questions are rarely retrieval-ready
                                   The query IS the problem to solve

4        Adaptive RAG              Not all questions need the same treatment
                                   Systems should be cost-aware

5        Self-Corrective RAG       Confidence ≠ correctness
                                   Systems must validate themselves

6        Agentic Router            Orchestration is the real engineering
                                   All previous phases become tools
─────────────────────────────────────────────────────────────
```

---

## What Makes This Domain Perfect for Learning

```
GREENWASHING DETECTION FORCES YOU TO:

Handle both numbers and prose      → Motivates hybrid search naturally
Deal with vague user questions     → Motivates query intelligence naturally
Compare claims vs evidence         → Motivates multi-step retrieval naturally
Be accurate (legal implications)   → Motivates self-correction naturally
Scale to many issuers              → Motivates routing naturally

The domain teaches the architecture.
You're not adding features for the sake of it.
Each phase solves a real problem you felt in the previous phase.
```

---

**Start Phase 1 this week. Break it intentionally. That frustration is the curriculum.**

Want me to design the evaluation strategy that runs across all phases?
