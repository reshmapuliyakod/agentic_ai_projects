# Two Modes: Analysis vs Q&A

## The Core Distinction

```
These are fundamentally different tasks
requiring different system designs

Q&A MODE                          ANALYSIS MODE
────────────────────────────────────────────────────────
User has a specific question      User wants a complete picture
Answer is focused and direct      Output is structured and comprehensive
Single retrieval cycle            Multiple retrieval chains
Conversational                    Report-like
Fast (seconds)                    Slow (minutes)
User drives the inquiry           System drives the inquiry
"What is the allocation amount?"  "Verify this bond end to end"
────────────────────────────────────────────────────────
```

---

## The Two Modes in Context

```
SAME UNDERLYING SYSTEM
SAME DOCUMENTS
SAME RETRIEVAL INFRASTRUCTURE
         │
         ├──→ Q&A MODE
         │    User asks, system answers
         │    One question at a time
         │
         └──→ ANALYSIS MODE
              System asks its OWN questions
              Runs verification chains
              Produces structured report
```

---

## Mode 1: Q&A

### What It Looks Like
```
User: "What categories were funded in the 2023 allocation?"

System:
  → Understands question
  → Retrieves from Allocation Report
  → Returns direct answer with source citation

Response:
  "The 2023 allocation covered three categories:
   Renewable Energy (€300M, 60%),
   Green Buildings (€120M, 24%),
   Clean Transport (€80M, 16%).
   Source: Allocation Report 2023, Table 2, Page 8"
```

### The Question Types in This Domain
```
TYPE 1: FACTUAL LOOKUP
  Single document, single answer
  "What is the ISIN of this bond?"
  "Who provided the Second Party Opinion?"
  "What is the framework version?"
  → Simple retrieval, Phase 1 handles this

TYPE 2: COMPARATIVE
  Same field across documents or time
  "How did 2022 allocation compare to 2023?"
  "Does the SPO version match the framework version?"
  "Did Scope 1 emissions go up or down since bond issuance?"
  → Needs multi-document retrieval, Phase 3

TYPE 3: VERIFICATION
  User suspects something, wants to check
  "Were all funded projects eligible under the framework?"
  "Did they publish the impact report on time?"
  "Is the CO2 methodology disclosed?"
  → Needs cross-document reasoning, Phase 4-5

TYPE 4: EXPLANATORY
  User wants to understand something
  "Why is the unallocated amount so high?"
  "What does Dark Green rating from CICERO mean?"
  "What is DNSH and does this bond address it?"
  → Needs retrieval + domain knowledge, Phase 3
```

### Q&A Conversation Flow
```
User
 │
 ├── Simple factual    →  Direct retrieval → Answer + Source
 │
 ├── Comparative       →  Multi-doc retrieval → Side by side → Answer
 │
 ├── Verification      →  Decompose → Retrieve each part →
 │                        Compare → Answer with flag if needed
 │
 └── Explanatory       →  Retrieve context → LLM explains →
                          Answer with domain context

All responses include:
  - Direct answer
  - Source document + section + page
  - Confidence level
  - "Want me to dig deeper?" prompt
```

---

## Mode 2: Analysis

### What It Looks Like
```
User: "Run a full verification on [Issuer] [ISIN]"

System runs its own internal Q&A:
  → "What was raised?" → checks Final Terms
  → "What was allocated?" → checks Allocation Report
  → "Do amounts match?" → compares
  → "Are categories eligible?" → checks Framework
  → "Was impact reported?" → checks Impact Report
  → "Is methodology disclosed?" → checks Impact Report
  → "Do macro emissions align?" → checks Sustainability Report
  → "What does capex show?" → checks Annual Report
  ... and so on through all verification chains

Output: Structured verification report
```

### The Internal Question Set the System Runs
```
ANALYSIS MODE = System running Q&A on itself
                with a predefined question agenda

MONEY TRAIL QUESTIONS
  Q1:  What is the total proceeds amount? [Final Terms]
  Q2:  What is the total allocated amount? [Allocation Report]
  Q3:  Do Q1 and Q2 match? If not, is gap explained?
  Q4:  What categories received allocation? [Allocation Report]
  Q5:  Are all categories in Q4 eligible? [Framework]
  Q6:  Do any funded projects predate the bond? [Allocation Report]
  Q7:  Is unallocated amount explained? [Allocation Report]
  Q8:  Where are unallocated proceeds held? [Annual Report]

FRAMEWORK TRAIL QUESTIONS
  Q9:  What framework version is in use? [Framework]
  Q10: What framework version did SPO review? [SPO]
  Q11: Do Q9 and Q10 match?
  Q12: What is the SPO overall verdict? [SPO]
  Q13: Are there concerns noted in SPO? [SPO]
  Q14: What framework version does prospectus reference? [Prospectus]
  Q15: Do Q9 and Q14 match?

IMPACT TRAIL QUESTIONS
  Q16: Was impact report published? [Impact Report]
  Q17: How long after issuance? [Impact Report + Final Terms]
  Q18: What impact metrics are claimed? [Impact Report]
  Q19: Is methodology disclosed per metric? [Impact Report]
  Q20: What baseline was used for CO2 calculations? [Impact Report]
  Q21: Was impact externally verified? [Impact Report]
  Q22: Are impact claims pro-rated to bond share? [Impact Report]

ISSUER INTEGRITY QUESTIONS
  Q23: What is Scope 1/2/3 trend since issuance? [Sust. Report]
  Q24: What % of capex is green vs fossil? [Annual Report]
  Q25: Is green bond mentioned in strategic report? [Annual Report]
  Q26: Any environmental controversies or legal proceedings?
       [Annual Report + Sustainability Report]
  Q27: What is EU Taxonomy alignment %? [Sustainability Report]

CONSISTENCY QUESTIONS
  Q28: Do cumulative numbers in Green Bond Report
       reconcile with individual reports?
  Q29: Does impact report cover same projects as allocation report?
  Q30: Are emissions trends in sustainability report
       consistent with impact claims?
```

### Analysis Output Structure
```
VERIFICATION REPORT
─────────────────────────────────────────────
HEADER
  Issuer, ISIN, Amount, Date, Framework version

DOCUMENT INVENTORY
  Which docs were found / missing / outdated

VERIFICATION CHAINS
  Money Trail:      PASS / FLAG / FAIL + evidence
  Framework Trail:  PASS / FLAG / FAIL + evidence
  Impact Trail:     PASS / FLAG / FAIL + evidence
  Issuer Integrity: PASS / FLAG / FAIL + evidence

FLAGS REGISTER
  Each flag: severity + description + source + contradicting source

OVERALL RATING
  Risk level + confidence + recommendation

EVIDENCE TRAIL
  Every finding linked to source document + section + page
─────────────────────────────────────────────
```

---

## How the Two Modes Relate

```
                    SHARED INFRASTRUCTURE
                    ─────────────────────
                    Document Store
                    Retrieval Engine
                    Domain Logic
                    Verification Rules
                         │
          ┌──────────────┴──────────────┐
          │                             │
       Q&A MODE                   ANALYSIS MODE
       ──────────                 ─────────────
       User provides              System provides
       the questions              the questions
                                  (predefined agenda)
          │                             │
       Conversational             Report output
       Interactive                Batch process
       Fast                       Thorough
       Exploratory                Systematic
          │                             │
          └──────────────┬──────────────┘
                         │
                 SAME ANSWER ENGINE
                 Same retrieval
                 Same LLM
                 Same evaluation
```

---

## The Interaction Between Modes

```
These modes are not isolated.
They feed each other naturally.

SCENARIO 1: Analysis first, Q&A to drill down
  User runs full analysis
  Report flags: "Impact methodology not disclosed"
  User switches to Q&A:
    "What exactly did they say about CO2 calculation?"
    "Has this issuer ever disclosed methodology before?"
    "What does PCAF require for this asset class?"

SCENARIO 2: Q&A reveals need for full analysis
  User asks: "Is the allocation complete for this bond?"
  System answers: "87% allocated, €65M unexplained"
  User: "Run a full verification on this bond"
  System switches to Analysis mode

SCENARIO 3: Analysis generates Q&A suggestions
  Analysis report includes:
  "Suggested follow-up questions:
   - Why was the SPO not renewed after framework update?
   - What happened to the €65M unallocated proceeds?
   - Why did Scope 1 emissions increase in the same year
     CO2 avoidance was claimed?"
  User clicks a question → Q&A mode answers it
```

---

## How This Maps to Your Learning Phases

```
PHASE 1: Basic RAG
  Supports:   Q&A only
  Question types: Factual lookup from single document
  "What is the total allocation amount?"

PHASE 2: Hybrid Search
  Supports:   Q&A with structured + unstructured
  Question types: Factual + some comparative
  "What categories were funded and what did the
   framework say about each?"

PHASE 3: Query Intelligence
  Supports:   Q&A with complex questions
              First steps toward Analysis mode
  Question types: All Q&A types
  "Were all funded categories eligible?"
  → System decomposes this automatically

PHASE 4: Adaptive RAG
  Supports:   Full Q&A mode
              Analysis mode skeleton
  System runs predefined question agenda
  on a single verification chain (money trail only)

PHASE 5: Self-Corrective RAG
  Supports:   Reliable Q&A
              Analysis mode with contradiction detection
  System catches when two documents disagree
  Flags instead of hallucinating a resolution

PHASE 6: Full System
  Supports:   Both modes completely
              Mode switching
              Analysis → Q&A drill down
              Q&A → trigger full analysis
```

---

## The Design Principle

```
Q&A MODE:       User is the analyst, system is the research assistant
                System answers what it's asked

ANALYSIS MODE:  System is the analyst, domain rules are the agenda
                System asks and answers its own questions
                Human reviews the output

Both modes require the same underlying capability:
  Retrieve the right information
  From the right document
  Answer accurately with evidence
  Know when it doesn't know

The difference is WHO sets the agenda.
```

---

**Q&A mode is what you build phase by phase.**

**Analysis mode emerges naturally when you string those Q&A capabilities together with a predefined verification agenda.**

**You don't build Analysis mode separately. You build Q&A well, then orchestrate it.**

Want to now define the Phase 1 build scope precisely - one document, one question type, end to end?
