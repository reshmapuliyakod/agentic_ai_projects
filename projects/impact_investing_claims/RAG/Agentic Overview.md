# From RAG to Agentic: The Evolution

## The Core Shift in Thinking

```
RAG SYSTEM                        AGENTIC SYSTEM
──────────────────────────────────────────────────────
Answers questions                 Pursues goals
Reactive                          Proactive
User drives every step            System drives its own steps
Fixed pipeline                    Dynamic planning
Retrieves then responds           Plans, acts, observes, replans
"Here is what I found"            "Here is what I did and why"
──────────────────────────────────────────────────────

The RAG system you're building IS the foundation.
Every phase adds a capability the agent will use as a tool.
```

---

## The Evolution Arc

```
PHASE 1-2          PHASE 3-4          PHASE 5-6          AGENTIC
Basic RAG          Intelligent RAG    Self-Corrective    Application
─────────          ───────────────    ───────────────    ───────────
Fixed pipeline     Dynamic routing    Self-evaluation    Goal pursuit
Single doc         Multi-doc          Contradiction      Multi-step
retrieval          reasoning          detection          planning
                                                         Tool use
                                                         Memory
                                                         Autonomy
    │                   │                   │                │
    └───────────────────┴───────────────────┴────────────────┘
                    SAME CORE CAPABILITY
                    Gets more autonomous at each stage
```

---

## What Makes Something "Agentic"

```
FOUR PROPERTIES OF AN AGENT

1. PLANNING
   Given a goal, breaks it into steps
   Decides what to do before doing it
   Not just reacting to input

2. TOOL USE
   Has a set of capabilities it can invoke
   Decides WHICH tool to use and WHEN
   Your RAG phases become tools

3. OBSERVATION + REPLANNING
   Looks at what it got back
   Decides if it's sufficient
   Changes plan if needed

4. MEMORY
   Remembers what it did earlier in the task
   Doesn't repeat itself
   Builds on previous findings
```

---

## Your RAG Phases Become Agent Tools

```
WHAT YOU BUILD IN RAG PHASES    WHAT IT BECOMES IN THE AGENT
────────────────────────────────────────────────────────────────

Phase 1: Basic Retriever        tool: search_document(doc, query)

Phase 2: Hybrid Search          tool: search_structured(query)
                                tool: search_unstructured(query)
                                tool: extract_metric(doc, field)

Phase 3: Query Decomposer       tool: decompose_question(question)
         Query Constructor      tool: build_sql_query(question)
         Query Translator       tool: translate_query(question)

Phase 4: Adaptive Router        tool: classify_question(question)
                                tool: select_strategy(question_type)

Phase 5: Self-Corrector         tool: verify_answer(answer, sources)
                                tool: detect_contradiction(doc1, doc2)
                                tool: assess_confidence(answer)

Phase 6: Router                 tool: identify_issuer(question)
                                tool: get_document_set(issuer)
                                tool: route_to_index(issuer, doc_type)

────────────────────────────────────────────────────────────────
AGENT ORCHESTRATES ALL OF THESE
Based on the goal, not a fixed pipeline
```

---

## The Agentic Architecture

```
                         USER GOAL
                    "Verify this bond" OR
                    "Answer this question" OR
                    "Monitor this issuer"
                              │
                              ▼
                    ┌─────────────────┐
                    │   AGENT CORE    │
                    │                 │
                    │   Planning      │
                    │   Memory        │
                    │   Reasoning     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        TOOL LAYER      MEMORY LAYER   KNOWLEDGE LAYER
        ──────────      ────────────   ───────────────
        search()        Short-term     Document Store
        extract()       (this task)    Vector Index
        compare()                      Structured DB
        verify()        Long-term      External APIs
        flag()          (across tasks) Web Search
        report()
        escalate()      Episodic
                        (what happened
                         last time with
                         this issuer)
```

---

## The Three Agentic Modes Your System Can Evolve Into

### Mode 1: Reactive Agent (Q&A Evolved)
```
TRIGGER: User asks a question

AGENT BEHAVIOUR:
  User: "Is the SPO version consistent with the framework?"

  Agent thinks:
    Step 1: I need the framework version
            → tool: search_document(framework, "version")
            → Got: "v2.1, updated March 2023"

    Step 2: I need the SPO version reviewed
            → tool: search_document(SPO, "framework version reviewed")
            → Got: "v2.0, reviewed January 2022"

    Step 3: Are these the same?
            → tool: compare(v2.1, v2.0)
            → Got: Different versions

    Step 4: Is this a problem?
            → tool: verify_answer(finding, domain_rules)
            → Got: "SPO should cover current framework version
                    This is a Severity 2 flag"

    Step 5: Is my answer complete?
            → tool: assess_confidence(answer)
            → Got: 0.91, sufficient

  Agent responds:
    "The SPO reviewed framework v2.0 but current
     framework is v2.1. This is a significant flag.
     The v2.1 changes have not been independently reviewed.
     [Source: SPO p.3, Framework cover page]"

DIFFERENCE FROM RAG:
  RAG would retrieve and hope the answer is in the chunks
  Agent PLANS the verification steps explicitly
```

### Mode 2: Proactive Agent (Analysis Evolved)
```
TRIGGER: User says "Verify bond [ISIN]"
         OR scheduled trigger (new document uploaded)

AGENT BEHAVIOUR:
  Agent creates its own plan:

  PLAN:
    1. Get all documents for this issuer/bond
    2. Run money trail verification
    3. Run framework trail verification
    4. Run impact trail verification
    5. Run issuer integrity check
    6. Compile flags by severity
    7. Generate report
    8. Identify questions needing human review

  EXECUTES PLAN:
    Each step uses tools from your RAG phases
    Observes results at each step
    Adjusts if something is missing
    e.g. "Impact report not found →
          flag as missing →
          continue with available docs"

  HANDLES UNEXPECTED:
    "Allocation report references a project
     not mentioned in the framework →
     I need to check the base prospectus too →
     adds unplanned step to verify"

DIFFERENCE FROM ANALYSIS MODE RAG:
  RAG analysis runs fixed question agenda
  Agent adapts its plan based on what it finds
```

### Mode 3: Monitoring Agent (New Capability)
```
TRIGGER: Scheduled (weekly/monthly)
         OR new document detected in system

AGENT BEHAVIOUR:
  Runs autonomously without user prompt

  "New allocation report uploaded for Issuer X"
  Agent wakes up:

    Step 1: What changed since last report?
            → Compare new vs previous allocation
            → tool: compare_versions(report_2023, report_2024)

    Step 2: Are new projects still eligible?
            → Check new projects against framework
            → tool: verify_eligibility(new_projects, framework)

    Step 3: Did cumulative numbers change correctly?
            → Arithmetic check
            → tool: verify_arithmetic(cumulative_totals)

    Step 4: Any new flags vs last assessment?
            → Compare to stored previous assessment
            → tool: diff_assessments(previous, current)

    Step 5: Does this need human attention?
            → Severity assessment
            → If Severity 2+: alert analyst
            → If Severity 0-1: update record silently

  OUTPUT:
    Analyst gets alert only when something meaningful changed
    Not a report every time - only when action needed

THIS IS FULLY AGENTIC:
  No human triggered it
  System decided what to check
  System decided if human needs to know
```

---

## The Memory Architecture

```
This is what separates a real agent from a smart RAG system

SHORT-TERM MEMORY (within one task)
────────────────────────────────────
  What I've already retrieved this session
  What steps I've completed
  What I've already flagged
  Prevents: Retrieving same thing twice
            Contradicting itself mid-task

LONG-TERM MEMORY (across tasks)
────────────────────────────────────
  Previous assessments of this issuer
  Historical flag patterns
  "Last time I checked, framework was v2.0"
  "This issuer has had 3 Severity 2 flags in 2 years"
  Enables: Trend detection
           Pattern recognition across issuers
           "This issuer's reporting is getting worse"

EPISODIC MEMORY (what happened)
────────────────────────────────────
  Log of every agent action and finding
  Full audit trail
  "On [date], agent found X in document Y"
  Critical for: Regulatory context
                Explaining decisions
                Debugging agent behaviour

DOMAIN MEMORY (what rules apply)
────────────────────────────────────
  Verification rules
  Flag severity definitions
  ICMA principles
  EU Taxonomy criteria
  "What counts as greenwashing"
  This is your domain knowledge encoded
```

---

## The Escalation Framework

```
Agent doesn't just produce reports.
It knows when humans need to be involved.

AGENT HANDLES AUTONOMOUSLY
  Factual Q&A with high confidence
  Routine allocation checks (arithmetic)
  Document inventory (what's missing)
  Severity 0-1 flags (informational)
  Scheduled monitoring with no new flags

AGENT FLAGS FOR HUMAN REVIEW
  Severity 2 flags (significant)
  Confidence below threshold (< 0.70)
  Contradictions it cannot resolve
  Missing critical documents
  Questions outside its domain knowledge

AGENT ESCALATES IMMEDIATELY
  Severity 3 flags (critical)
  Potential regulatory breach detected
  Proceeds > raised (possible fraud signal)
  Funded projects on exclusion list
  These go to analyst with full evidence trail

HUMAN FEEDBACK GOES BACK TO AGENT
  Analyst resolves a flag → agent learns
  Analyst overrides a finding → agent notes it
  Analyst adds context → agent stores it
  This is how the agent gets smarter over time
```

---

## The Agentic Evolution Roadmap

```
WHERE YOU ARE          WHAT YOU ADD           WHAT YOU GET
────────────────────────────────────────────────────────────

Phase 1-2              Nothing yet            Smart document search
Basic RAG              Just building tools    

Phase 3-4              Chain tools together   Agent can plan
Query Intelligence     Simple sequences       2-3 step reasoning
Adaptive RAG           

Phase 5                Add observation loop   Agent can self-correct
Self-Corrective        "Was that right?"      Knows when to retry

Phase 6                Add memory             Agent remembers context
Agentic Router         Add goal framing       Agent pursues objectives
                       LangGraph workflows    

Post Phase 6           Add monitoring         Agent runs autonomously
Agentic Application    Add escalation         Agent decides who to tell
                       Add feedback loop      Agent learns from humans
                       Add multi-agent        Agents check each other
────────────────────────────────────────────────────────────
```

---

## Multi-Agent Extension

```
As complexity grows, one agent isn't enough.
Your system can evolve into specialised agents.

ORCHESTRATOR AGENT
  Receives goal
  Breaks into sub-tasks
  Assigns to specialist agents
  Compiles final output
        │
        ├──→ RETRIEVAL AGENT
        │    Specialist in finding information
        │    Knows which index, which document type
        │    Optimises search strategies
        │
        ├──→ VERIFICATION AGENT
        │    Specialist in cross-document checking
        │    Runs verification chains
        │    Applies domain rules
        │
        ├──→ ANALYSIS AGENT
        │    Specialist in interpretation
        │    Generates findings and flags
        │    Applies severity framework
        │
        └──→ REPORTING AGENT
             Specialist in output generation
             Formats for different audiences
             Analyst report vs executive summary
             vs regulatory submission format

WHY MULTI-AGENT?
  Each agent can be optimised for its task
  Agents can verify each other's work
  Parallel execution (faster)
  Easier to debug (know which agent failed)
  Easier to improve (update one agent at a time)
```

---

## What This Looks Like End to End

```
FULLY AGENTIC SCENARIO (Future State)

Monday 9am: New document detected
  → Monitoring agent wakes up
  → "New Impact Report for Issuer X uploaded"

9:01am: Orchestrator plans
  → "Run full verification update"
  → Assigns tasks to specialist agents

9:02am: Agents work in parallel
  → Retrieval agent: fetches all current documents
  → Verification agent: runs money + framework trails
  → Analysis agent: compares to previous assessment

9:04am: Agents complete, orchestrator compiles
  → 2 new flags found since last assessment
  → 1 Severity 2: Impact methodology changed, not disclosed
  → 1 Severity 1: Reporting period shifted by 3 months

9:05am: Escalation decision
  → Severity 2 flag → analyst notification
  → Severity 1 flag → logged, no immediate action

9:05am: Analyst receives alert
  → Summary of what changed
  → Evidence trail
  → Suggested questions to investigate further
  → One-click to enter Q&A mode for this bond

9:15am: Analyst asks follow-up questions
  → Q&A agent answers from evidence already gathered
  → No new retrieval needed (short-term memory)

9:20am: Analyst marks Severity 2 as confirmed flag
  → Feedback stored in long-term memory
  → Agent updates issuer risk profile
  → Regulatory log updated automatically
```

---

## The Honest Assessment

```
WHAT YOU'RE BUILDING IS GENUINELY AGENTIC CAPABLE

Your domain is perfect for agents because:
  Clear goals (verify this bond)
  Verifiable outcomes (flag is right or wrong)
  Structured decision rules (severity framework)
  Natural tool decomposition (each doc type = tool)
  Real escalation need (regulatory context)
  Monitoring use case (documents update regularly)

THE PATH IS CLEAR:
  Build RAG phases as tools
  Connect tools with LangGraph
  Add memory layer
  Add goal framing
  Add monitoring triggers
  Add escalation logic

YOU DON'T NEED TO PLAN ALL OF THIS NOW
  Phase 1 is still just:
  One document + one question + one answer
  
  But every decision you make in Phase 1
  should not block the agentic future
  
  That's the only thing to keep in mind now.
```

---

**The RAG system you're building is not a stepping stone to something else.**

**It IS the agent, at an early stage of its evolution.**

**Every phase makes it more autonomous, more capable, more trustworthy.**

**Want to now lock in the Phase 1 scope and start building?**
