# Evaluation Strategy: Across All Phases

## The Core Philosophy First

```
Most people build RAG systems and ask "does it work?"
That's the wrong question.

Right questions:
"Is it getting BETTER across phases?"
"Where exactly is it failing?"
"Is the failure in retrieval or generation?"

Evaluation is not a final step.
It's the instrument panel you fly the whole project with.
```

---

## The Fundamental Decomposition

```
RAG has exactly two places it can fail:

RETRIEVAL FAILURE                    GENERATION FAILURE
"I fetched the wrong stuff"          "I had the right stuff but 
                                      said something wrong"
        ↓                                      ↓
Did we get the right chunks?         Did we use those chunks correctly?

These need SEPARATE metrics.
Conflating them is the #1 evaluation mistake.
```

---

## The Evaluation Stack

```
LEVEL 4: Business Outcome          "Did we correctly flag greenwashing?"
              ↑ built on
LEVEL 3: Answer Quality            "Is the answer correct, grounded, complete?"
              ↑ built on
LEVEL 2: Retrieval Quality         "Did we fetch the right chunks?"
              ↑ built on
LEVEL 1: Data Quality              "Is what we ingested clean and complete?"

Most people only measure Level 3.
Then wonder why their system is unreliable.
```

---

## Level 1: Data Quality Checks

### What to Measure
```
Before any RAG evaluation, know your data:

INGESTION CHECKS
├── How many chunks per document?
├── Average chunk size (too small? too large?)
├── Empty or near-empty chunks (parsing failures)
├── Chunks that are pure numbers/tables (embed poorly)
└── Duplicate chunks (re-ingestion bugs)

WHY THIS MATTERS
If your data is bad, every other metric lies to you.
A retrieval score of 0.3 could mean:
  → Bad retrieval logic     (fixable in Phase 2-3)
  → Bad chunk quality       (fixable in ingestion)
These require completely different fixes.
```

### The Simple Audit
```
After every ingestion, answer:
1. Did all pages load? (check page count vs expected)
2. Are chunks meaningful? (random sample 10, read them)
3. Are tables preserved? (find a known table, check its chunk)
4. Any encoding issues? (look for garbled text)

This takes 10 minutes and saves hours of debugging.
```

---

## Level 2: Retrieval Evaluation

### The Core Metrics

```
THREE METRICS TELL THE WHOLE STORY:

─────────────────────────────────────────────────────
METRIC 1: CONTEXT PRECISION
─────────────────────────────────────────────────────
Of everything I retrieved, how much was actually relevant?

Retrieved 5 chunks → 3 were relevant → Precision = 3/5 = 0.6

High precision = low noise in what LLM sees
Low precision = LLM gets confused by irrelevant context

─────────────────────────────────────────────────────
METRIC 2: CONTEXT RECALL  
─────────────────────────────────────────────────────
Of all relevant chunks that EXIST, how many did I find?

5 relevant chunks exist in corpus
I retrieved 3 of them → Recall = 3/5 = 0.6

High recall = complete picture for LLM
Low recall = LLM answers from incomplete information

─────────────────────────────────────────────────────
METRIC 3: MEAN RECIPROCAL RANK (MRR)
─────────────────────────────────────────────────────
Was the BEST chunk ranked first?

Best chunk at position 1 → MRR = 1.0
Best chunk at position 2 → MRR = 0.5
Best chunk at position 3 → MRR = 0.33

Why it matters:
LLMs pay more attention to early context.
Right answer buried at position 5 = effectively lost.
─────────────────────────────────────────────────────
```

### The Precision-Recall Tradeoff
```
You control this with K (how many chunks to retrieve)

Retrieve K=3:   High precision, lower recall
                Less noise, but might miss key chunks

Retrieve K=10:  Lower precision, higher recall  
                More complete, but more noise

Your job per phase:
Find the K where precision × recall is maximized
for your specific question types
```

### How to Get Ground Truth for Retrieval
```
The chicken-and-egg problem:
"I need to know which chunks are relevant
 to evaluate retrieval,
 but finding relevant chunks IS the retrieval problem"

Solution: Build a small golden dataset manually

PROCESS:
1. Write 20-30 test questions about BP's sustainability report
2. Manually find the chunks that answer each question
3. Record: question → [chunk_id_1, chunk_id_2, ...]
4. This is your ground truth
5. Run retrieval, compare to ground truth

Yes, this is manual work.
No, there's no shortcut for Phase 1.
Later phases use LLMs to help generate this.
```

---

## Level 3: Answer Quality Evaluation

### The Four Dimensions
```
Every answer should be scored on four axes:

DIMENSION 1: FAITHFULNESS
"Did the answer come from the retrieved context?"
"Or did the LLM make something up?"

Score: What % of answer claims are supported by retrieved chunks?
Target: > 0.85 (some inference is ok, fabrication is not)

DIMENSION 2: ANSWER RELEVANCE  
"Does the answer actually address the question?"

Score: Semantic similarity between question and answer
Target: > 0.75

DIMENSION 3: CONTEXT RELEVANCE
"Were the retrieved chunks relevant to the question?"
(This overlaps with retrieval eval - that's intentional)

Score: What % of retrieved chunks were needed for the answer?
Target: > 0.70

DIMENSION 4: COMPLETENESS (domain-specific)
"Did the answer cover all aspects of the question?"

For greenwashing specifically:
  Did it address the CLAIM?
  Did it address the EVIDENCE?
  Did it address the GAP between them?
  
Score: 0/1/2/3 aspects covered
Target: All 3 for analytical questions
```

### The Evaluation Matrix Per Phase
```
PHASE 1 (Basic RAG)
Focus on:    Faithfulness + Answer Relevance
Skip for now: Completeness (system can't do it yet)
Target:      Faithfulness > 0.80, Relevance > 0.70
Baseline:    Whatever you get is your baseline

PHASE 2 (Hybrid Search)
New metric:  Did hybrid beat vector-only on metric questions?
Expect:      Precision improves on factual/numerical questions
Watch for:   Recall might drop (hybrid can be more selective)

PHASE 3 (Query Intelligence)
New metric:  Sub-question coverage
             Did decomposition cover all aspects of complex questions?
Expect:      Completeness score improves significantly
Watch for:   Latency increases (more LLM calls)

PHASE 4 (Adaptive RAG)
New metric:  Strategy accuracy
             Did the router choose the right strategy?
             Simple question → did it use simple retrieval?
             Complex question → did it decompose?
Expect:      Cost per query drops (simple questions stay simple)
Watch for:   Router errors on edge cases

PHASE 5 (Self-Corrective RAG)
New metric:  Hallucination rate before vs after correction
             How often does self-correction actually change the answer?
Expect:      Faithfulness jumps significantly
Watch for:   Over-correction (system becomes too uncertain)
```

---

## Level 4: Greenwashing-Specific Evaluation

### The Domain Metric
```
This is what makes your project unique.

GREENWASHING DETECTION SCORE
For each issuer + claim pair:

CLAIM:    "BP committed to net zero by 2050"
EVIDENCE: Scope 1 emissions increased 3% in 2023
          Fossil fuel capex increased 15% in 2023
          Renewable investment = 4% of total capex

SYSTEM OUTPUT:
  Claim detected:        ✓
  Supporting evidence:   1 item found
  Contradicting evidence: 3 items found
  Gap score:             0.78 (high greenwashing risk)
  Confidence:            0.82

GROUND TRUTH (you manually labeled):
  Actual gap score:      High risk
  
EVALUATION:
  Did system detect the claim?           Yes/No
  Did system find contradicting evidence? Yes/No  
  Was risk level correct?                Yes/No
  Was confidence calibrated?             (predicted 0.82, was it right?)
```

### Building Your Greenwashing Test Set
```
START WITH 15-20 LABELED EXAMPLES:

CLEAR GREENWASHING (5 examples)
  Claims that are demonstrably contradicted by data
  These should be easy for your system to catch

LEGITIMATE CLAIMS (5 examples)  
  Claims that are actually supported by data
  System should NOT flag these

AMBIGUOUS CASES (5 examples)
  Claims that are partially supported
  These reveal how nuanced your system is

MISSING DATA CASES (5 examples)
  Claims where you can't verify either way
  System should say "insufficient evidence" not guess

This test set is your north star for the entire project.
Every phase should improve scores on this set.
```

---

## The Evaluation Infrastructure

### What to Build (Not When to Build It)
```
BUILD IN PHASE 1:
  Simple scoring script
  Manual test set (20 questions)
  CSV log of: question, retrieved chunks, answer, scores
  
  Why now? You need a baseline before you can show improvement.

ADD IN PHASE 2:
  Automated scoring (RAGAS library does this well)
  Comparison view: Phase 1 vs Phase 2 scores
  Per-question-type breakdown

ADD IN PHASE 3:
  Trace logging (which sub-questions were generated?)
  Langfuse or LangSmith integration
  You need to SEE the decomposition to debug it

ADD IN PHASE 4:
  Strategy decision logging
  Cost per query tracking
  Latency per strategy type

ADD IN PHASE 5:
  Correction rate (how often did self-correction trigger?)
  Before/after correction score comparison
  False positive rate (corrected things that were fine)
```

### The Observability Principle
```
For every query, you should be able to answer:

WHAT HAPPENED?
  What query was sent to retrieval?
  What chunks came back?
  What score did each chunk get?
  What prompt was sent to LLM?
  What answer came back?
  What was the self-correction decision?

WHY DID IT FAIL? (when it does)
  Was it a retrieval failure? (wrong chunks)
  Was it a generation failure? (right chunks, wrong answer)
  Was it a data failure? (answer doesn't exist in corpus)
  Was it a query failure? (question was ambiguous)

Without this, debugging is guessing.
```

---

## The Evaluation Workflow Per Phase

```
BEFORE BUILDING EACH PHASE:
  Run current test set → record baseline scores
  Identify the specific failure mode you're fixing
  Hypothesize: "Phase N should improve metric X by Y"

AFTER BUILDING EACH PHASE:
  Run same test set → compare to baseline
  Did the target metric improve?
  Did any other metric regress? (common - fixing one thing breaks another)
  Update test set with new edge cases you discovered

THIS CREATES:
  A learning log, not just a codebase
  Evidence that each phase added value
  A debugging trail when things go wrong
```

---

## The Honest Metrics Dashboard

```
What your dashboard should show at Phase 6:

RETRIEVAL
  Context Precision:    Phase1=0.52 → Phase6=0.81  ↑
  Context Recall:       Phase1=0.61 → Phase6=0.79  ↑
  MRR:                  Phase1=0.44 → Phase6=0.76  ↑

GENERATION  
  Faithfulness:         Phase1=0.71 → Phase6=0.91  ↑
  Answer Relevance:     Phase1=0.68 → Phase6=0.84  ↑
  Completeness:         Phase1=0.31 → Phase6=0.78  ↑

DOMAIN
  Greenwashing Detection Accuracy:    0.82
  False Positive Rate:                0.11
  Insufficient Evidence Rate:         0.14  ← system knows what it doesn't know

SYSTEM
  Avg Latency:          Phase1=1.2s → Phase6=3.8s  (acceptable tradeoff)
  Avg Cost per Query:   Phase1=$0.002 → Phase6=$0.011
  Self-correction Rate: 23% of queries triggered correction
```

---

## The Single Most Important Evaluation Insight

```
CALIBRATION > ACCURACY

A system that says:
  "I'm 90% confident BP is greenwashing" and is right 90% of the time
  
is MORE VALUABLE than a system that says:
  "I'm 90% confident" and is right 70% of the time

For greenwashing detection specifically:
  Overconfident wrong answers have legal/reputational consequences
  
Your final evaluation question:
  "When my system says it's confident, is it actually right?"
  
That's calibration.
That's what separates a demo from a trustworthy system.
```

---

**The test set you build in Phase 1 is the most important artifact in this entire project. Spend more time on it than you think you should.**

Want me to design the specific test questions and labeling schema for the Phase 1 golden dataset?
