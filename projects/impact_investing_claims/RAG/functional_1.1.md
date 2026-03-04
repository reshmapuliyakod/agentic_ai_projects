# Bridging Technical Expertise with UoP Bond Domain Knowledge

## First, Reframe Your Situation

```
You don't need to become a green bond expert.
You need to become someone who can build systems
that help experts do their job faster and more accurately.

That's a completely different bar.
And it plays to your technical strengths.
```

---

## What UoP Bonds Actually Are (The Minimum You Need)

```
UoP = Use of Proceeds

CORE IDEA (brutally simple):
  Issuer says:   "I'm raising €500M"
  Issuer promises: "I will ONLY spend this on green projects"
  Documents say: "Here's what counts as green, here's proof we did it"

GREENWASHING RISK:
  Did they actually spend it on what they promised?
  Does "green" in their definition actually mean green?
  Are the impact numbers real or inflated?

YOUR SYSTEM'S JOB:
  Read all the documents
  Check if the promises match the evidence
  Flag where they don't
```

### The Document Chain (This Is Everything)
```
BEFORE ISSUANCE                          AFTER ISSUANCE
─────────────────────────────────────    ──────────────────────────────
Base Prospectus (UoP section)            Allocation Report
  "Here's what we're allowed               "Here's where we actually
   to spend the money on"                   spent the money"
         ↓                                        ↓
Second Party Opinion                     Impact Report
  "Independent reviewer says               "Here's the environmental
   our framework is credible"               impact we achieved"
         ↓                                        ↓
Final Terms                              Sustainability Report
  "This specific bond,                     "Here's our overall
   this specific amount,                    ESG performance"
   these specific projects"
                                         Green Bond Report
                                           "Summary of the whole
                                            green bond programme"
```

### The Verification Questions Your System Should Answer
```
QUESTION 1: ELIGIBILITY
  Base Prospectus says: "Eligible projects include renewable energy"
  Allocation Report says: "We funded a gas pipeline"
  → Is gas pipeline renewable energy? FLAG.

QUESTION 2: ALLOCATION COMPLETENESS  
  Bond raised: €500M
  Allocation Report shows: €380M allocated
  → Where is the other €120M? FLAG.

QUESTION 3: IMPACT CREDIBILITY
  Impact Report claims: "Avoided 2M tonnes CO2"
  Methodology section: Not disclosed
  → How was this calculated? FLAG.

QUESTION 4: FRAMEWORK CONSISTENCY
  Final Terms reference: Framework version 2.1
  Second Party Opinion reviewed: Framework version 2.0
  → Was the updated framework independently reviewed? FLAG.

QUESTION 5: TEMPORAL CONSISTENCY
  Bond issued: 2021
  Projects funded: Started 2019
  → Were these genuinely funded by this bond or pre-existing? FLAG.
```

---

## The Knowledge Gap Map

```
WHAT YOU HAVE                        WHAT YOU NEED TO LEARN
─────────────────────────────────    ──────────────────────────────
RAG architecture                     Document relationships in UoP chain
Vector search                        What fields matter in each document
Query decomposition                  What questions regulators actually ask
Hybrid retrieval                     EU taxonomy eligibility criteria
Evaluation frameworks                What constitutes a red flag
LLM orchestration                    Market conventions vs requirements
Data modeling                        ICMA Green Bond Principles (basics)
─────────────────────────────────    ──────────────────────────────

GOOD NEWS:
The right side is learnable in 2-3 weeks of focused reading.
The left side took you years.
You're more prepared than you think.
```

---

## How to Build Domain Knowledge Without Being an Expert

### The Four Sources You Actually Need
```
SOURCE 1: ICMA Green Bond Principles (2 hours)
  What it is: The global standard for green bonds
  What to extract: 
    - 4 core components (Use of Proceeds, Process for 
      Project Evaluation, Management of Proceeds, Reporting)
    - These 4 components = your evaluation framework
  Where: icmagroup.org (free PDF)

SOURCE 2: EU Green Bond Standard (3 hours)
  What it is: EU's stricter version, now regulation
  What to extract:
    - EU Taxonomy alignment requirements
    - What "Do No Significant Harm" means practically
    - Disclosure requirements
  Why it matters: CBI operates in EU jurisdiction
  Where: EUR-Lex (free)

SOURCE 3: Read 3 Actual Bond Documents (4 hours)
  Pick any large EU issuer: KfW, EIB, or an Irish bank
  Read in this order:
    1. Base Prospectus UoP section (30 mins)
    2. Second Party Opinion (30 mins)  
    3. Allocation Report (30 mins)
    4. Impact Report (30 mins)
  Goal: Not to understand everything
        To see the STRUCTURE and LANGUAGE patterns
  
SOURCE 4: Talk to One Domain Expert (1 hour)
  You work at CBI. Someone there knows this deeply.
  Ask them ONE question:
  "What are the top 5 things you look for when 
   reviewing a green bond for greenwashing risk?"
  That answer is worth more than 10 hours of reading.
```

### The Expert Interview Framework
```
When you talk to that domain expert, ask:

ABOUT THE DOCUMENTS:
  "Which document do you trust most? Least?"
  "Which document is most often misleading?"
  "What's missing from these documents that you wish was there?"

ABOUT THE WORKFLOW:
  "Walk me through how you review a bond. What order?"
  "Where do you spend most of your time?"
  "What takes longest that a computer could help with?"

ABOUT RED FLAGS:
  "Give me 3 examples of clear greenwashing you've seen"
  "Give me 3 examples of legitimate green claims"
  "What's the hardest case to judge?"

ABOUT YOUR SYSTEM:
  "If I could answer one question automatically, what would be most useful?"

Their answers become your test cases.
Their red flags become your evaluation criteria.
Their workflow becomes your system design.
```

---

## Mapping Domain Knowledge to Your Technical Phases

```
PHASE 1: Basic RAG
  Domain need:    Answer simple factual questions from one document
  Start with:     Allocation Report (most structured, clearest facts)
  Test questions: "What is the total amount allocated?"
                  "What project categories received funding?"
                  "What percentage went to renewable energy?"
  Why this doc:   Tables + clear numbers = easy to verify your system

PHASE 2: Hybrid Search
  Domain need:    Extract metrics AND claims from same document
  Add:            Impact Report (has both prose claims and number tables)
  Test questions: "What CO2 reduction was claimed and what methodology
                   was used to calculate it?"
  Why this matters: Claim is in prose, number is in table
                    Hybrid search is motivated by real domain need

PHASE 3: Query Intelligence  
  Domain need:    Answer cross-document questions
  Add:            Base Prospectus + Allocation Report together
  Test questions: "Were all funded projects eligible under the framework?"
  Why complex:    Eligibility criteria in Prospectus
                  Actual projects in Allocation Report
                  Requires decomposition to answer

PHASE 4: Adaptive RAG
  Domain need:    Some questions are simple lookups, some are analysis
  Test questions: Simple: "What is the bond ISIN?"
                  Complex: "Is this bond aligned with EU Taxonomy?"
  System learns:  ISIN question → direct retrieval
                  Taxonomy question → multi-document analysis

PHASE 5: Self-Corrective RAG
  Domain need:    Regulatory context requires high confidence answers
  Critical here:  CBI context means wrong answers have consequences
  System learns:  "I found allocation data but no methodology disclosure
                   → answer is incomplete → flag for human review"

PHASE 6: Agentic Router
  Domain need:    Multiple issuers, full document chain per issuer
  Full system:    Issuer → Route to their document set → 
                  Cross-document verification → 
                  Greenwashing risk score
```

---

## The Domain-Technical Translation Layer

### Build This as a Reference Document for Yourself
```
DOCUMENT          KEY FIELDS TO EXTRACT        TECHNICAL CHALLENGE
──────────────────────────────────────────────────────────────────
Base Prospectus   Eligible categories          Long document, UoP section
(UoP section)     Exclusion criteria           buried in 200+ pages
                  Framework version            → Needs section detection

Second Party      Reviewer name                Opinion vs fact distinction
Opinion           Review date                  → Metadata extraction
                  Framework version reviewed   → Version matching logic
                  Alignment verdict            

Final Terms       ISIN                         Highly structured
                  Issue size                   → Good for SQL storage
                  Settlement date              → Not vector search
                  UoP reference to prospectus  

Allocation        Total proceeds               Tables dominant
Report            Amount allocated             → Table extraction critical
                  Project categories           → Hybrid search needed
                  Unallocated amount           → Numerical reasoning

Impact Report     Impact metrics               Methodology claims
                  Methodology                  → Claim vs evidence gap
                  Reporting period             → Temporal reasoning
                  Baseline assumptions         

Green Bond        Programme summary            Aggregates other docs
Report            Issuances to date            → Cross-reference checker
                  Cumulative allocation        

Sustainability    Broader ESG context          Very long, low signal/noise
Report            Scope 1/2/3 emissions        → Targeted retrieval needed
                  ESG targets                  → Risk of irrelevant chunks
──────────────────────────────────────────────────────────────────
```

---

## Your Biggest Advantage You're Not Seeing

```
MOST PEOPLE BUILDING RAG SYSTEMS:
  Pick a generic domain (customer service, HR docs)
  Have no real evaluation criteria
  Can't tell if answers are actually correct
  Build demos, not systems

YOU HAVE:
  A domain with REAL documents (not synthetic data)
  A domain with VERIFIABLE ground truth
    (allocation amounts are either right or wrong)
  A domain with REAL STAKES (regulatory context)
  Access to REAL EXPERTS (your colleagues)
  A domain where the FAILURE MODE IS CLEAR
    (greenwashing = claim without evidence)

This makes your evaluation strategy honest.
This makes your system actually useful.
This is rare.
```

---

## Practical First Steps This Week

```
MONDAY (2 hours)
  Download one complete bond document set
  Recommended: Any EIB (European Investment Bank) green bond
  They're public, well-structured, good quality
  Get: Allocation Report + Impact Report + Framework

TUESDAY (2 hours)
  Read ICMA Green Bond Principles summary (not full document)
  Write down in your own words:
    "What are the 4 things a green bond must demonstrate?"
  This becomes your evaluation rubric

WEDNESDAY (1 hour)
  Talk to one colleague who reviews these documents
  Ask the 4 question categories above
  Record their answers

THURSDAY (2 hours)
  Build your Phase 1 test set
  Write 20 questions about the EIB documents you downloaded
  Answer them manually by reading the documents
  These manual answers = your ground truth

FRIDAY (rest of time)
  Start Phase 1 build
  You now have:
    Real documents ✓
    Domain understanding ✓  
    Test questions ✓
    Ground truth answers ✓
    A colleague who can validate outputs ✓
```

---

## The Mental Model That Ties Everything Together

```
Think of your system as a JUNIOR ANALYST
who is very good at reading and remembering documents
but needs guidance on what to look for.

YOUR JOB as the builder:
  Teach it what questions to ask        (query design)
  Teach it where to look                (routing + retrieval)
  Teach it what counts as evidence      (domain logic)
  Teach it when to be uncertain         (self-correction)
  Teach it how to explain its reasoning (generation)

The domain knowledge tells you WHAT to look for.
The technical knowledge tells you HOW to find it.

You need both.
You're building both.
That's the project.
```

---

**The single most valuable thing you can do this week is get three real UoP bond document sets and read them with one question in mind: "What would I need to verify to know if this bond is legitimate?"**

**That question will design your system better than any architecture diagram.**

Want me to design the specific data model for how these seven document types relate to each other in your system?
