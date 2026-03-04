# Document Processing Agent: Design

---

## The Core Idea

```
Instead of a fixed pipeline that runs steps 1-10 in sequence
an agent that THINKS about each document and decides
what to do with it

FIXED PIPELINE                    DOCUMENT PROCESSING AGENT
──────────────────────────────────────────────────────────────
Same steps for every document     Different strategy per document
You hardcode the logic            Agent decides the logic
Breaks on unexpected docs         Agent handles new doc types
No awareness of what it found     Agent reports what it did and why
──────────────────────────────────────────────────────────────
```

---

## What the Agent Does

```
GIVEN: A folder of H&M green bond documents

AGENT GOAL: 
  Process every document correctly
  Store everything in the right place
  Report what it found and what it flagged
  Be ready for RAG ingestion

AGENT DECISIONS PER DOCUMENT:
  What type is this document?
  What extraction strategy applies?
  Which sections do I need?
  Are there tables? Where do they go?
  What metadata should I capture?
  Did this succeed or fail?
  What should I flag?
```

---

## Agent Architecture

```
                    DOCUMENT PROCESSING AGENT
                              │
                    ┌─────────┴─────────┐
                    │   ORCHESTRATOR    │
                    │   Receives folder │
                    │   Plans work      │
                    │   Tracks progress │
                    │   Compiles report │
                    └─────────┬─────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
   TOOL: INVENTORY     TOOL: CLASSIFIER    TOOL: VALIDATOR
   Check what exists   Identify doc type   Cross-check results
   Flag missing docs   Assign strategy     Flag inconsistencies
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
   TOOL: EXTRACTOR     TOOL: PARSER        TOOL: STORER
   Pull text           Apply strategy      Vector store
   Find sections       Parse tables        PostgreSQL
   Capture metadata    Clean content       Document registry
```

---

## The Orchestrator Logic

```
STEP 1: INVENTORY
  Scan input folder
  List all files found
  Check against expected document list
  Flag any missing critical documents
  Report: found N of expected M documents

STEP 2: CLASSIFY EACH DOCUMENT
  For each file:
    Try filename-based classification first
    If ambiguous: read first 2 pages, ask LLM
    Assign document_type
    Assign processing_strategy
    Log classification decision + confidence

STEP 3: PROCESS EACH DOCUMENT
  For each classified document:
    Call appropriate extraction strategy
    Extract text (full or targeted sections)
    Extract tables (if any)
    Extract metadata fields
    Log what was extracted

STEP 4: STORE EVERYTHING
  Text → chunk → embed → vector store
  Tables → validate → PostgreSQL
  Metadata → document registry
  Log storage confirmation

STEP 5: VALIDATE
  Run cross-checks across stored data
  Flag inconsistencies
  Generate processing report

STEP 6: REPORT
  What was processed
  What was flagged
  What failed
  What needs human review
  Ready for RAG? YES/NO
```

---

## The Classification Tool

```
INPUT:  filename + first 2 pages of text
OUTPUT: document_type + confidence + strategy

CLASSIFICATION LOGIC (in order)

LEVEL 1: FILENAME RULES (fast, no LLM)
  Contains "framework"          → green_bond_framework
  Contains "SPO" or "second party" → second_party_opinion
  Contains "final terms"        → final_terms
  Contains "prospectus"         → base_prospectus
  Contains "allocation"         → allocation_report
  Contains "impact"             → impact_report
  Contains "green bond report"  → green_bond_report
  Contains "sustainability"     → sustainability_report
  Contains "annual report"      → annual_report
  Confidence: HIGH if match found

LEVEL 2: CONTENT RULES (if filename ambiguous)
  Search first 2 pages for marker phrases:
  "use of proceeds" + "eligible categories" → green_bond_framework
  "second party opinion" + "sustainalytics" → second_party_opinion
  "final terms" + "ISIN"                    → final_terms
  "allocation of proceeds" + table present  → allocation_report
  Confidence: MEDIUM if match found

LEVEL 3: LLM CLASSIFICATION (if still ambiguous)
  Send first 2 pages to LLM
  Ask: what type of document is this?
  Map response to known document types
  Confidence: LOW, flag for human confirmation

IF UNCLASSIFIED:
  Move to /unclassified/ folder
  Flag in report
  Do not process further
  Human must classify manually
```

---

## The Processing Strategy Per Document Type

```
DOCUMENT TYPE          STRATEGY
──────────────────────────────────────────────────────────────

green_bond_framework   FULL EXTRACTION
                       All pages
                       Look for tables (eligibility criteria)
                       Extract: version, eligible categories,
                                exclusions, reporting commitments

second_party_opinion   FULL EXTRACTION
                       All pages
                       Extract: reviewer, date, rating,
                                framework version reviewed,
                                concerns noted

final_terms            FULL EXTRACTION
                       Short document, take everything
                       Extract: ISIN, amount, dates,
                                UoP reference

base_prospectus        TARGETED EXTRACTION
                       Find UoP section only
                       Markers: "use of proceeds"
                       Extract until next major section heading
                       Flag: how many pages extracted vs total

allocation_report      FULL EXTRACTION + TABLE PRIORITY
                       All pages
                       Table extraction is primary goal
                       Extract: allocation table,
                                proceeds summary,
                                verification details

impact_report          FULL EXTRACTION + TABLE PRIORITY
                       All pages
                       Tables: impact metrics
                       Text: methodology claims
                       Extract: metrics, methodology,
                                verification details

green_bond_report      FULL EXTRACTION
                       All pages
                       Extract: programme totals,
                                framework version current,
                                cumulative allocation

sustainability_report  TARGETED EXTRACTION
                       Target sections only:
                         emissions / climate
                         green bond reference
                         EU taxonomy
                       Skip: social, governance,
                             supply chain, GRI index
                       Flag: pages extracted vs total

annual_report          TARGETED EXTRACTION
                       Target sections only:
                         strategic report (green mentions)
                         capital expenditure table
                         green bond financial notes
                         climate risk section
                       Skip: everything else
                       Flag: pages extracted vs total
──────────────────────────────────────────────────────────────
```

---

## The Table Extraction Tool

```
INPUT:  extracted pages for a document
OUTPUT: list of structured tables with metadata

FOR EACH PAGE
  Detect if table present (pdfplumber)
  If table found:
    Extract as rows x columns
    Capture page number
    Capture text above table (likely the title)
    Capture text below table (likely footnotes)
    Convert to DataFrame
    Validate (has headers? numeric columns numeric?)

TABLE IDENTIFICATION
  Classify each table by content:
    Has "category" + "amount" columns → allocation_table
    Has "metric" + "value" + "unit"   → impact_metrics_table
    Has "scope" + emissions numbers   → emissions_table
    Has "capex" or "investment"       → capex_table
    Has "ISIN" + financial terms      → bond_terms_table
    Unclassified                      → generic_table_{n}

TABLE STORAGE
  Save each table as CSV:
    {issuer}_{doc_type}_{table_type}_{page}.csv
    hm_allocation_report_allocation_table_p4.csv
    hm_impact_report_impact_metrics_table_p7.csv

  Also load to PostgreSQL:
    Table name maps to schema table
    allocation_table → allocations table
    emissions_table  → emissions table
    Unclassified     → raw_tables table

  Always keep CSV as backup
  Always store source_doc + page_number with table
```

---

## The Metadata Extraction Tool

```
INPUT:  document text + document type
OUTPUT: structured metadata dictionary

UNIVERSAL FIELDS (every document)
  issuer_name:        rule-based (filename or first page)
  issuer_id:          normalised lowercase no spaces
  document_type:      from classifier
  document_date:      regex for dates on cover page
  page_count:         from PDF reader
  file_path:          local path
  extraction_date:    today

TYPE-SPECIFIC FIELDS

  final_terms:
    isin:             regex (XS + 10 digits)
    issue_size:       regex (€ + number + M/B)
    issue_date:       regex
    maturity_date:    regex

  green_bond_framework:
    framework_version: regex (v + number) or LLM
    eligible_categories: LLM extraction from text
    has_exclusion_list: boolean (keyword search)

  second_party_opinion:
    reviewer_name:    keyword match (Sustainalytics etc)
    review_date:      regex
    framework_version_reviewed: regex or LLM
    overall_rating:   keyword match (Dark/Medium/Light Green)

  allocation_report:
    reporting_year:   regex
    total_proceeds:   regex or LLM
    total_allocated:  regex or LLM
    verification_body: keyword match

EXTRACTION APPROACH
  Try regex first (fast, reliable for structured fields)
  Try keyword match second (for categorical fields)
  Use LLM last (for fields needing interpretation)
  Flag any field that could not be extracted
  Never guess - leave null and flag
```

---

## The Validation Tool

```
RUNS AFTER ALL DOCUMENTS PROCESSED

CHECK 1: DOCUMENT INVENTORY
  Expected document types: list all 9
  Found document types: what was classified
  Missing: flag each missing type with severity
    Critical missing: Framework, SPO, Allocation Report
    Important missing: Impact Report, Final Terms
    Nice to have: Green Bond Report, Annual Report sections

CHECK 2: AMOUNT RECONCILIATION
  Source 1: Final Terms → issue_size
  Source 2: Allocation Report → total_proceeds
  Action: Compare, flag if difference > 1%

CHECK 3: FRAMEWORK VERSION MATCH
  Source 1: Framework → framework_version
  Source 2: SPO → framework_version_reviewed
  Source 3: Base Prospectus → framework_version_referenced
  Action: All three should match, flag any mismatch

CHECK 4: CATEGORY ELIGIBILITY
  Source 1: Framework → eligible_categories list
  Source 2: Allocation Report → funded_categories list
  Action: Every funded category must be in eligible list
  Flag: Any funded category NOT in eligible list

CHECK 5: DATE LOGIC
  Source 1: Final Terms → issue_date
  Source 2: Allocation Report → project_dates (if available)
  Action: Projects should post-date bond issuance
  Flag: Any project dated before bond issuance

OUTPUT: validation_report.json
  Per check: PASS / FAIL / INSUFFICIENT_DATA
  Per flag: severity + description + sources
```

---

## The Agent Report

```
DOCUMENT PROCESSING REPORT
────────────────────────────────────────────────
Issuer:     H&M Group
Run date:   [date]
────────────────────────────────────────────────
DOCUMENT INVENTORY
  green_bond_framework    ✓ Found  | 24 pages | v1.0 | 2020
  second_party_opinion    ✓ Found  | 18 pages | Sustainalytics
  base_prospectus         ✓ Found  | 8 pages extracted of 340
  final_terms             ✓ Found  | 12 pages | XS2235371480
  allocation_report       ✓ Found  | 31 pages
  impact_report           ✗ Missing | CRITICAL FLAG
  green_bond_report       ✓ Found  | 28 pages
  sustainability_report   ✓ Found  | 14 pages extracted of 180
  annual_report           ✓ Found  | 9 pages extracted of 310

EXTRACTION SUMMARY
  Total pages processed:    144
  Total chunks created:     387
  Total tables extracted:   12
  Tables loaded to DB:      10
  Tables failed:            2  ← flagged for review
  Metadata fields captured: 47 of 52 expected

VALIDATION RESULTS
  Amount reconciliation:    PASS
  Framework version match:  FAIL ← SPO reviewed v1.0,
                                    framework now v1.1
  Category eligibility:     PASS
  Date logic:               INSUFFICIENT DATA
  Document completeness:    FAIL ← Impact report missing

FLAGS GENERATED
  [CRITICAL]   Impact Report not found
  [SIGNIFICANT] SPO reviewed v1.0, current framework v1.1
  [MODERATE]   2 tables failed extraction, manual review needed

READY FOR RAG INGESTION: YES (with flags noted)
────────────────────────────────────────────────
```

---

## Why This Should Be an Agent Not a Pipeline

```
PIPELINE BREAKS WHEN:                AGENT HANDLES WHEN:
────────────────────────────────────────────────────────
Document type is ambiguous           Classifies using multiple signals
New document type appears            Flags as unclassified, continues
Table extraction fails               Notes failure, continues with text
Section markers not found            Tries alternative markers, flags
Metadata field missing               Leaves null, flags, continues
Two documents contradict             Notes contradiction, flags both
Document is scanned PDF              Detects, flags for OCR treatment
────────────────────────────────────────────────────────

A pipeline stops or crashes on unexpected input.
An agent handles it, notes it, and continues.
That resilience matters when documents vary
across issuers, years, and formats.

THIS AGENT IS ALSO YOUR FIRST REAL AGENTIC COMPONENT.
You learn agent patterns on a contained, verifiable problem
before applying them to the harder retrieval and analysis problems.
```

---

## How This Fits the Learning Arc

```
DOCUMENT PROCESSING AGENT    →    LATER PHASES
────────────────────────────────────────────────
Classification tool          →    Query classifier (Phase 4)
Strategy selection           →    Retrieval strategy selection (Phase 4)
Validation checks            →    Self-correction checks (Phase 5)
Processing report            →    Verification report (Phase 6)
Handles unexpected input     →    Agent replanning (Phase 6)

You are not building throwaway code.
You are building the first agent in your system
and learning the patterns you will use throughout.
```

---

**Build this agent before Phase 1 RAG.**

**When it works, your data is clean, structured, validated, and ready.**

**Phase 1 then becomes purely about retrieval quality, not data quality.**
