# GreenLens: Task Tracker Per Phase

---

## Phase 1: Basic RAG
### Goal: One document, one question type, end to end working

```
ENVIRONMENT SETUP
  [ ] Create project repository and folder structure
  [ ] Set up virtual environment
  [ ] Install core dependencies
  [ ] Set up environment variables (.env)
  [ ] Set up Weaviate locally (Docker)
  [ ] Set up Langfuse account and connect
  [ ] Verify all connections working

DATA PREPARATION
  [ ] Download EIB green bond document set (public)
  [ ] Identify and isolate Allocation Report
  [ ] Read Allocation Report manually
  [ ] Write 20 test questions manually
  [ ] Answer all 20 questions manually from document
  [ ] Save as golden dataset (ground truth)
  [ ] Note which questions are easy vs hard

DOCUMENT INGESTION
  [ ] Load PDF successfully
  [ ] Inspect raw text output (check quality)
  [ ] Identify any parsing issues (tables, encoding)
  [ ] Decide chunk size (document, experiment with 2-3 sizes)
  [ ] Decide chunk overlap
  [ ] Implement chunking
  [ ] Inspect sample chunks manually (read 10 randomly)
  [ ] Log chunk statistics (count, size distribution)
  [ ] Fix any empty or broken chunks

EMBEDDING AND STORAGE
  [ ] Connect to OpenAI embeddings
  [ ] Embed sample chunks (test before full run)
  [ ] Design Weaviate schema for chunks
  [ ] Ingest all chunks into Weaviate
  [ ] Verify chunk count in store matches expected
  [ ] Test basic similarity search manually

RETRIEVAL
  [ ] Implement basic vector search function
  [ ] Test retrieval on 5 sample questions
  [ ] Inspect retrieved chunks manually
  [ ] Tune K (number of chunks retrieved)
  [ ] Check if relevant chunks are being returned
  [ ] Log retrieval results for inspection

GENERATION
  [ ] Design basic prompt template
  [ ] Connect retrieval output to LLM
  [ ] Generate answers for sample questions
  [ ] Read answers manually, check against ground truth
  [ ] Adjust prompt if answers are poor quality

PIPELINE WIRING
  [ ] Connect ingestion → retrieval → generation
  [ ] Run full pipeline on one question end to end
  [ ] Run full pipeline on all 20 golden questions
  [ ] Log inputs and outputs for every question

EVALUATION
  [ ] Score all 20 answers against ground truth manually
  [ ] Record baseline faithfulness score
  [ ] Record baseline answer relevance score
  [ ] Record baseline context precision score
  [ ] Identify top 5 failure cases
  [ ] Document WHY each failure happened
  [ ] Retrieval failure or generation failure?

OBSERVABILITY
  [ ] Confirm Langfuse is capturing all traces
  [ ] Check trace for one full pipeline run
  [ ] Verify you can see: query, chunks, prompt, answer
  [ ] Identify what is missing from traces

PHASE 1 COMPLETION CRITERIA
  [ ] 20 questions answered end to end
  [ ] Baseline scores recorded
  [ ] Failure modes documented
  [ ] Can clearly explain where system breaks and why
```

---

## Phase 2: Hybrid Search
### Goal: Structured + unstructured retrieval working together

```
DOMAIN PREPARATION
  [ ] Read Impact Report from same EIB bond manually
  [ ] Identify all tables in Impact Report
  [ ] Identify all claim statements in Impact Report
  [ ] Add 10 new test questions (metric-focused)
  [ ] Add 10 new test questions (claim-focused)
  [ ] Answer all new questions manually (extend golden dataset)

DATA MODELING
  [ ] Design PostgreSQL schema for structured metrics
  [ ] Decide which fields go in DB vs vector store
  [ ] Document the decision and reasoning
  [ ] Set up PostgreSQL locally

STRUCTURED DATA INGESTION
  [ ] Extract tables from Allocation Report manually first
  [ ] Build table extraction pipeline
  [ ] Validate extracted data against source document
  [ ] Load structured data into PostgreSQL
  [ ] Verify data integrity (row counts, spot checks)
  [ ] Extract tables from Impact Report
  [ ] Load Impact Report structured data

UNSTRUCTURED DATA INGESTION
  [ ] Ingest Impact Report text into Weaviate
  [ ] Tag chunks with document type metadata
  [ ] Verify Impact Report chunks are retrievable
  [ ] Check table text chunks separately

KEYWORD SEARCH (BM25)
  [ ] Enable BM25 indexing in Weaviate
  [ ] Test BM25 search on sample queries
  [ ] Compare BM25 results vs vector results on same query
  [ ] Document when BM25 wins vs vector search

HYBRID SEARCH IMPLEMENTATION
  [ ] Implement combined BM25 + vector search
  [ ] Implement Reciprocal Rank Fusion (RRF) for merging
  [ ] Test hybrid search on numerical questions
  [ ] Test hybrid search on claim questions
  [ ] Compare hybrid vs vector-only on golden dataset

SQL QUERY LAYER
  [ ] Implement basic text to SQL for metric questions
  [ ] Test on allocation amount questions
  [ ] Test on category breakdown questions
  [ ] Validate SQL outputs against known values

RESULT MERGING
  [ ] Implement logic to merge vector + SQL results
  [ ] Test merged results on mixed questions
  [ ] Ensure LLM receives both structured and unstructured context
  [ ] Check prompt handles both types of context well

EVALUATION
  [ ] Run full golden dataset (now 40 questions)
  [ ] Compare Phase 2 vs Phase 1 scores
  [ ] Specifically check: did numerical questions improve?
  [ ] Specifically check: did claim questions improve?
  [ ] Document any regressions
  [ ] Identify new failure modes

PHASE 2 COMPLETION CRITERIA
  [ ] Numerical questions clearly better than Phase 1
  [ ] Claim questions retrieving both text and numbers
  [ ] Scores recorded and compared to Phase 1 baseline
  [ ] Can explain when hybrid beats vector-only and why
```

---

## Phase 3: Query Intelligence
### Goal: System understands and transforms what user is really asking

```
DOMAIN PREPARATION
  [ ] Add Green Bond Framework document to system
  [ ] Read Framework manually
  [ ] Write 10 cross-document test questions
  [ ] Write 10 complex multi-part test questions
  [ ] Answer all manually (extend golden dataset)
  [ ] Label each question by type:
      factual / comparative / verification / explanatory

QUERY TRANSLATION
  [ ] Implement query rewriting prompt
  [ ] Test on 10 sample questions
  [ ] Compare retrieval quality before vs after rewriting
  [ ] Implement HyDE (hypothetical document embedding)
  [ ] Test HyDE on questions where direct search fails
  [ ] Compare HyDE vs standard embedding on hard questions
  [ ] Document when translation helps vs hurts

QUERY DECOMPOSITION
  [ ] Implement question decomposition prompt
  [ ] Test decomposition on 5 complex questions
  [ ] Manually verify sub-questions are correct
  [ ] Implement multi-query retrieval (retrieve per sub-question)
  [ ] Implement answer synthesis across sub-answers
  [ ] Test end to end on cross-document questions
  [ ] Check: does decomposition improve completeness?

QUERY CONSTRUCTION (Text to SQL)
  [ ] Extend SQL query builder from Phase 2
  [ ] Handle more complex metric queries
  [ ] Handle year-over-year comparison queries
  [ ] Handle aggregation queries (totals, percentages)
  [ ] Validate all generated SQL before execution
  [ ] Add SQL error handling and fallback

QUESTION TYPE CLASSIFIER
  [ ] Build classifier: factual / comparative /
      verification / explanatory
  [ ] Test classifier on golden dataset questions
  [ ] Check classification accuracy manually
  [ ] Route each type to appropriate query strategy

CROSS-DOCUMENT RETRIEVAL
  [ ] Implement retrieval across multiple documents
  [ ] Test: Framework eligibility + Allocation categories
  [ ] Test: SPO version + Framework version comparison
  [ ] Ensure metadata filtering works by document type
  [ ] Check issuer filtering still works correctly

EVALUATION
  [ ] Run full golden dataset (now 60 questions)
  [ ] Specifically measure completeness score
  [ ] Compare complex question performance vs Phase 2
  [ ] Measure latency increase from decomposition
  [ ] Document cost per query increase
  [ ] Identify over-decomposition cases (simple → complex)

PHASE 3 COMPLETION CRITERIA
  [ ] Complex cross-document questions answerable
  [ ] Completeness score measurably improved
  [ ] Question type classification working
  [ ] Latency and cost tradeoff documented
```

---

## Phase 4: Adaptive RAG
### Goal: System chooses its own retrieval strategy per question

```
DOMAIN PREPARATION
  [ ] Add SPO document to system
  [ ] Add Base Prospectus (UoP section) to system
  [ ] Write 10 simple questions (should use simple retrieval)
  [ ] Write 10 complex questions (should use full pipeline)
  [ ] Label expected strategy for each question
  [ ] Extend golden dataset with strategy labels

LANGGRAPH SETUP
  [ ] Install and configure LangGraph
  [ ] Understand graph nodes and edges conceptually
  [ ] Design pipeline as a graph (draw it first)
  [ ] Implement basic graph with two nodes
  [ ] Test basic graph execution

COMPLEXITY CLASSIFIER
  [ ] Define complexity levels:
      simple / analytical / comparative / verification
  [ ] Implement LLM-based complexity classifier
  [ ] Test on 20 sample questions
  [ ] Measure classification accuracy vs manual labels
  [ ] Tune classifier prompt

STRATEGY DEFINITIONS
  [ ] Define simple strategy:
      direct vector search → answer
  [ ] Define analytical strategy:
      decompose → multi-retrieve → synthesise
  [ ] Define comparative strategy:
      parallel retrieve → merge → compare → answer
  [ ] Define verification strategy:
      retrieve claim → retrieve evidence →
      compare → flag if gap

ADAPTIVE PIPELINE
  [ ] Implement strategy router in LangGraph
  [ ] Connect classifier output to strategy selection
  [ ] Implement each strategy as graph branch
  [ ] Test each branch independently
  [ ] Test full routing end to end
  [ ] Verify simple questions stay on simple path
  [ ] Verify complex questions use full pipeline

COST AND LATENCY TRACKING
  [ ] Add token counting per query
  [ ] Add latency measurement per query
  [ ] Compare cost: simple vs complex strategy
  [ ] Verify simple questions are cheaper than Phase 3
  [ ] Document cost per question type

FALLBACK LOGIC
  [ ] Define fallback if strategy fails
  [ ] Implement fallback to simpler strategy
  [ ] Test fallback triggers correctly
  [ ] Log all fallback events

EVALUATION
  [ ] Run full golden dataset
  [ ] Measure strategy selection accuracy
  [ ] Compare overall scores vs Phase 3
  [ ] Measure cost reduction for simple questions
  [ ] Identify misclassification patterns
  [ ] Document edge cases

PHASE 4 COMPLETION CRITERIA
  [ ] Router correctly classifies question complexity
  [ ] Simple questions use simple path (cost efficient)
  [ ] Complex questions use full pipeline
  [ ] Cost per query lower than Phase 3 on average
  [ ] LangGraph traces visible in Langfuse
```

---

## Phase 5: Self-Corrective RAG
### Goal: System evaluates and fixes its own answers

```
DOMAIN PREPARATION
  [ ] Add Sustainability Report (targeted sections) to system
  [ ] Add Annual Report (targeted sections) to system
  [ ] Write 10 questions designed to trigger contradictions
      between documents
  [ ] Write 10 questions where answer is not in documents
      (system should say insufficient evidence)
  [ ] Manually identify expected contradictions
  [ ] Extend golden dataset with expected corrections

RETRIEVAL GRADER
  [ ] Implement chunk relevance grader (LLM as judge)
  [ ] Define relevance scoring rubric
  [ ] Test grader on 20 retrieved chunk sets
  [ ] Validate grader decisions manually
  [ ] Set relevance threshold for proceeding vs re-retrieving
  [ ] Implement re-retrieval with rewritten query if threshold not met

ANSWER GRADER
  [ ] Implement faithfulness checker
      (is answer supported by chunks?)
  [ ] Implement relevance checker
      (does answer address the question?)
  [ ] Test both checkers on sample answers
  [ ] Validate checker decisions manually
  [ ] Define thresholds for pass / retry / escalate

HALLUCINATION DETECTOR
  [ ] Implement claim extraction from answer
  [ ] Implement claim verification against source chunks
  [ ] Test on answers known to contain hallucinations
  [ ] Measure false positive rate (flags correct answers)
  [ ] Measure false negative rate (misses hallucinations)
  [ ] Tune sensitivity

CONTRADICTION DETECTOR
  [ ] Implement cross-document contradiction check
  [ ] Test: Impact Report claim vs Sustainability Report data
  [ ] Test: Allocation Report vs Framework eligibility
  [ ] Test: SPO version vs Framework version
  [ ] Define contradiction severity levels
  [ ] Implement contradiction flagging in output

INSUFFICIENT EVIDENCE HANDLER
  [ ] Implement detection when answer cannot be supported
  [ ] System should say "insufficient evidence" not guess
  [ ] Test on questions about missing documents
  [ ] Test on questions outside document scope
  [ ] Verify system does not hallucinate when evidence missing

CORRECTION LOOP
  [ ] Implement retry logic with query rewrite
  [ ] Set maximum retry limit (prevent infinite loops)
  [ ] Implement web search fallback for missing information
  [ ] Log all correction events
  [ ] Track correction rate (how often does it trigger?)

CONFIDENCE SCORING
  [ ] Implement confidence score per answer
  [ ] Calibrate: does confidence correlate with accuracy?
  [ ] Test confidence on easy vs hard questions
  [ ] Test confidence on contradictory evidence cases
  [ ] Define confidence thresholds for auto vs human review

ESCALATION LOGIC
  [ ] Define escalation triggers
  [ ] Implement escalation flag in output
  [ ] Test escalation on Severity 2+ flag scenarios
  [ ] Verify high confidence wrong answers are caught

EVALUATION
  [ ] Run full golden dataset
  [ ] Measure faithfulness before vs after correction
  [ ] Measure hallucination rate before vs after
  [ ] Measure correction trigger rate
  [ ] Check for over-correction (correct answers changed)
  [ ] Measure confidence calibration
  [ ] Document contradiction detection accuracy

PHASE 5 COMPLETION CRITERIA
  [ ] Faithfulness measurably higher than Phase 4
  [ ] Hallucination rate measurably lower
  [ ] Contradictions between documents detected and flagged
  [ ] System says insufficient evidence when appropriate
  [ ] Confidence scores are calibrated
  [ ] Escalation triggers correctly on serious flags
```

---

## Phase 6: Agentic Router
### Goal: Multi-issuer, full document chain, system plans its own approach

```
DOMAIN PREPARATION
  [ ] Add second issuer document set
  [ ] Add Final Terms document type
  [ ] Add Green Bond Report document type
  [ ] Write 10 multi-issuer comparison questions
  [ ] Write 10 full verification chain questions
  [ ] Extend golden dataset to 100+ questions
  [ ] Label questions by issuer and document chain required

ISSUER NAMESPACING
  [ ] Design issuer namespace schema in Weaviate
  [ ] Implement issuer-specific collections or tenants
  [ ] Re-ingest all documents with issuer namespace
  [ ] Test retrieval filters by issuer
  [ ] Verify no cross-issuer contamination

ENTITY EXTRACTION
  [ ] Implement issuer name extraction from question
  [ ] Implement ISIN extraction from question
  [ ] Implement document type detection from question
  [ ] Test on 20 sample questions
  [ ] Handle ambiguous issuer references
  [ ] Handle questions with no clear issuer

DOCUMENT INVENTORY MANAGER
  [ ] Build document registry per issuer
      (what documents exist, what is missing)
  [ ] Implement missing document detection
  [ ] Implement document freshness check
      (is this document outdated?)
  [ ] Test inventory check on known complete set
  [ ] Test inventory check on incomplete set

FULL TOOL LIBRARY
  [ ] Wrap all Phase 1-5 capabilities as tools
  [ ] Define tool interfaces clearly
      (inputs, outputs, when to use)
  [ ] Test each tool independently
  [ ] Document tool selection logic

AGENT CORE
  [ ] Design agent planning prompt
  [ ] Implement goal decomposition
  [ ] Implement tool selection logic
  [ ] Implement observation and replanning
  [ ] Test agent on simple goal (one chain)
  [ ] Test agent on complex goal (full verification)
  [ ] Test agent on ambiguous goal (needs clarification)

VERIFICATION CHAIN ORCHESTRATION
  [ ] Implement Money Trail chain as agent workflow
  [ ] Implement Framework Trail chain as agent workflow
  [ ] Implement Impact Trail chain as agent workflow
  [ ] Implement Issuer Integrity chain as agent workflow
  [ ] Test each chain independently
  [ ] Test all chains together (full verification)

MEMORY IMPLEMENTATION
  [ ] Implement short-term memory (within task)
  [ ] Implement long-term memory (across tasks)
  [ ] Implement episodic memory (audit trail)
  [ ] Test memory persistence across sessions
  [ ] Test memory retrieval for returning issuers

Q&A MODE FINALISATION
  [ ] All question types working with full tool library
  [ ] Conversational context maintained within session
  [ ] Source citations on every answer
  [ ] Confidence score on every answer
  [ ] Escalation flag when needed

ANALYSIS MODE IMPLEMENTATION
  [ ] Implement 30-question verification agenda
  [ ] Implement adaptive agenda
      (skip questions if document missing)
  [ ] Implement flag register compilation
  [ ] Implement verification report generation
  [ ] Test on complete document set
  [ ] Test on incomplete document set
  [ ] Validate report against manual analysis

MODE SWITCHING
  [ ] Implement Q&A → Analysis trigger
  [ ] Implement Analysis → Q&A drill down
  [ ] Implement suggested follow-up questions
      from analysis output
  [ ] Test mode switching in conversation flow

FINAL OUTPUT
  [ ] Implement Bond Verification Summary format
  [ ] Include all four chain results
  [ ] Include flag register with severity
  [ ] Include document inventory
  [ ] Include overall risk rating
  [ ] Include confidence score
  [ ] Include human review recommendation
  [ ] Test output format on 3 different bonds

EVALUATION
  [ ] Run complete golden dataset (100+ questions)
  [ ] Measure end-to-end greenwashing detection accuracy
  [ ] Measure false positive rate
  [ ] Measure false negative rate
  [ ] Measure confidence calibration
  [ ] Compare all metrics across all phases
  [ ] Document system limitations honestly

PHASE 6 COMPLETION CRITERIA
  [ ] Multi-issuer routing working without contamination
  [ ] Full verification report generated end to end
  [ ] Q&A and Analysis modes both functional
  [ ] Mode switching works naturally
  [ ] All evaluation metrics recorded across all phases
  [ ] System knows what it does not know
  [ ] Escalation working correctly
```

---

## Cross-Phase Tasks (Run Throughout)

```
DOCUMENTATION
  [ ] Update README after each phase
  [ ] Document every design decision and why
  [ ] Document every failure and how it was fixed
  [ ] Keep evaluation scores table updated

EVALUATION CONTINUITY
  [ ] Never delete golden dataset questions
  [ ] Always run full golden dataset at phase end
  [ ] Always compare to previous phase baseline
  [ ] Track all four metric levels per phase

OBSERVABILITY
  [ ] Langfuse traces active from Phase 1
  [ ] Review traces after every major change
  [ ] Flag any unexpected routing decisions
  [ ] Keep trace logs for debugging reference

DOMAIN KNOWLEDGE
  [ ] Read ICMA Green Bond Principles before Phase 1
  [ ] Read EU Green Bond Standard before Phase 3
  [ ] Talk to CBI colleague before Phase 1
  [ ] Validate system outputs with domain expert
      at end of Phase 3 and Phase 6
```

---

## Progress Tracker
```
PHASE          STATUS        BASELINE SCORE    COMPLETION DATE
──────────────────────────────────────────────────────────────
Phase 1        [ ] Not started    -                -
Phase 2        [ ] Not started    -                -
Phase 3        [ ] Not started    -                -
Phase 4        [ ] Not started    -                -
Phase 5        [ ] Not started    -                -
Phase 6        [ ] Not started    -                -
──────────────────────────────────────────────────────────────

METRICS TO TRACK ACROSS ALL PHASES

RETRIEVAL
  Context Precision     [ ] [ ] [ ] [ ] [ ] [ ]
  Context Recall        [ ] [ ] [ ] [ ] [ ] [ ]
  MRR                   [ ] [ ] [ ] [ ] [ ] [ ]

GENERATION
  Faithfulness          [ ] [ ] [ ] [ ] [ ] [ ]
  Answer Relevance      [ ] [ ] [ ] [ ] [ ] [ ]
  Completeness          [ ] [ ] [ ] [ ] [ ] [ ]

DOMAIN
  Detection Accuracy    [ ] [ ] [ ] [ ] [ ] [ ]
  False Positive Rate   [ ] [ ] [ ] [ ] [ ] [ ]
  Confidence Calibration[ ] [ ] [ ] [ ] [ ] [ ]

SYSTEM
  Avg Latency           [ ] [ ] [ ] [ ] [ ] [ ]
  Avg Cost Per Query    [ ] [ ] [ ] [ ] [ ] [ ]
  Correction Rate       [ ] [ ] [ ] [ ] [ ] [ ]
```

---

## Post Phase 6: Agentic Extension Tasks

```
MONITORING AGENT
  [ ] Define document change detection triggers
  [ ] Implement new document upload detection
  [ ] Implement scheduled verification runs
  [ ] Implement diff against previous assessment
  [ ] Implement change significance classifier
  [ ] Test: new allocation report triggers correct checks
  [ ] Test: minor change does not trigger alert
  [ ] Test: significant change triggers analyst alert

MULTI-AGENT ARCHITECTURE
  [ ] Design agent specialisation boundaries
  [ ] Implement Orchestrator Agent
  [ ] Implement Retrieval Agent
  [ ] Implement Verification Agent
  [ ] Implement Analysis Agent
  [ ] Implement Reporting Agent
  [ ] Test agent communication
  [ ] Test parallel agent execution
  [ ] Test orchestrator handles agent failures

MEMORY LAYER EXTENSION
  [ ] Implement issuer risk profile (long-term memory)
  [ ] Implement flag history per issuer
  [ ] Implement pattern detection across issuers
      "Three issuers with same methodology gap"
  [ ] Implement analyst feedback storage
  [ ] Test memory retrieval accuracy
  [ ] Test memory does not contaminate across issuers

ESCALATION FRAMEWORK
  [ ] Define escalation matrix formally
      Severity 0-1 → log silently
      Severity 2   → analyst notification
      Severity 3   → immediate escalation
  [ ] Implement notification system
  [ ] Implement escalation audit trail
  [ ] Test all escalation paths
  [ ] Verify Severity 3 never goes unnoticed

FEEDBACK LOOP
  [ ] Implement analyst flag confirmation interface
  [ ] Implement analyst override with reason capture
  [ ] Implement feedback storage
  [ ] Implement system improvement from feedback
  [ ] Track: how often does analyst agree with system?
  [ ] Track: what types of flags get overridden most?
  [ ] Use override patterns to improve classifier

UI DEVELOPMENT
  [ ] Design Q&A interface (Streamlit)
  [ ] Design Analysis mode trigger and output view
  [ ] Design document upload interface
  [ ] Design flag register view
  [ ] Design issuer dashboard
      (all bonds, all flags, risk trend)
  [ ] Design cross-issuer comparison view
  [ ] Add confidence indicators to all outputs
  [ ] Add source citation viewer
      (click source → see exact document section)
  [ ] Test UI with non-technical user
  [ ] Iterate based on feedback

REGULATORY OUTPUT
  [ ] Define regulatory report format
  [ ] Implement export to structured format
      (PDF, JSON, structured report)
  [ ] Implement full audit trail export
  [ ] Implement evidence package per flag
      (flag + source + contradicting source + reasoning)
  [ ] Test regulatory output completeness
  [ ] Validate with CBI colleague
```

---

## Master Checklist Summary

```
FOUNDATION
  [ ] Memory document saved (this document)
  [ ] Task tracker saved
  [ ] Project repository created
  [ ] Domain reading completed
  [ ] Expert conversation completed
  [ ] Golden dataset started

PHASE GATES (must complete before moving to next phase)
  [ ] Phase 1: Baseline scores recorded,
               failure modes documented
  [ ] Phase 2: Hybrid beats vector-only on numerical questions,
               scores compared to Phase 1
  [ ] Phase 3: Complex questions answerable,
               completeness improved
  [ ] Phase 4: Router classifies correctly,
               cost per query reduced
  [ ] Phase 5: Faithfulness improved,
               contradictions detected,
               confidence calibrated
  [ ] Phase 6: Full verification report generated,
               both modes working,
               multi-issuer working

AGENTIC EXTENSION GATES
  [ ] Monitoring agent running autonomously
  [ ] Multi-agent architecture tested
  [ ] Feedback loop operational
  [ ] UI usable by non-technical analyst
  [ ] Regulatory output validated

PROJECT COMPLETE WHEN
  [ ] Given any EIB or EU issuer green bond ISIN
      system can produce a credible verification report
      with evidence trail, flag register, and
      confidence-calibrated risk rating
      without human intervention for routine cases
      with correct escalation for complex cases
```

---

## Quick Reference Card

```
WHEN YOU RETURN TO THIS PROJECT
────────────────────────────────────────────────────────

STEP 1: Read the Memory Document (15 mins)
STEP 2: Check Progress Tracker (5 mins)
STEP 3: Find current phase, find first unchecked task
STEP 4: Before any build, run golden dataset, record score
STEP 5: Build, test, evaluate, document
STEP 6: Check phase gate criteria before moving on

IF SOMETHING IS BROKEN
  Is it a retrieval failure or generation failure?
  Check Langfuse traces first
  Check data quality second
  Check prompt third
  Check chunking last

IF SCORES ARE NOT IMPROVING
  Check if golden dataset questions are fair
  Check if failure is consistent or random
  Isolate: which question type is failing?
  Go back one phase, re-examine assumptions

IF DOMAIN IS UNCLEAR
  Re-read relevant section of memory document
  Check document field definitions
  Check verification chain definitions
  Check flag severity framework
  Talk to CBI colleague

NORTH STAR
  Given a bond ISIN
  System produces a credible verification report
  Analyst can trust it enough to act on it
  System knows what it does not know
  Every finding has a source
────────────────────────────────────────────────────────
```

---

**This is your complete project reference.**

**Memory Document + Task Tracker = everything you need to pick this up at any point.**

**When ready: Phase 1, Task 1, Environment Setup.**
