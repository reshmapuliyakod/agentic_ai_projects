# AI Parse → Chunking → Extraction: Revised Order

---

## The Revised Order

```
BEFORE                          AFTER (YOUR APPROACH)
──────────────────────────────────────────────────────
1. Raw text extraction          1. Raw text extraction
2. Document classification      2. Document classification
3. Section detection            3. ai_parse_document
4. Field extraction             4. Section-aware chunking
5. Chunking                     5. Field extraction FROM parsed results
6. Embedding                    6. Embedding
```

---

## The Key Question You Are Asking

```
Can ai_parse_document output be used for BOTH:
  chunking (structural output)
  field extraction (semantic output)

ANSWER: YES
  ai_parse_document returns document elements
  Each element has:
    type        (what kind of element is this?)
    content     (the actual text)
    metadata    (position, page, hierarchy)

  These elements are richer than raw text
  They already carry structure
  Chunking uses the structure
  Extraction uses the content + structure
```

---

## What ai_parse_document Returns

```
INPUT:  PDF (raw bytes or path)

OUTPUT: List of elements, each element is:
  {
    type:     "Title" / "NarrativeText" / "Table" /
              "ListItem" / "Header" / "Footer" /
              "PageBreak" / "Image"

    content:  the actual text of this element

    metadata: {
      page_number:    INT
      filename:       TEXT
      coordinates:    position on page
      parent_id:      for hierarchical elements
    }
  }

WHAT THIS GIVES YOU IMMEDIATELY:
  Tables are already separated from text
  Headers are already identified
  List items are already identified
  Page numbers are attached to every element
  No need for separate section marker detection
  Structure comes from the parser, not from rules
```

---

## Revised Flow in Detail

```
STEP 1: RAW TEXT + ELEMENTS
  ai_parse_document(pdf)
  Output: list of typed elements
  Store: bronze_parsed_elements
    element_id, doc_id, element_type,
    content, page_number, sequence_order,
    parent_id, coordinates

        ↓

STEP 2: DOCUMENT CLASSIFICATION
  Input:  first N elements WHERE element_type = Title
          OR NarrativeText from page 1-3
  Process: LLM classifies document_type + document_year
  Why elements not raw text:
    Titles are already extracted by parser
    Cleaner input to classifier
    Less noise

        ↓

STEP 3: SECTION ASSEMBLY
  Input:  all elements in sequence order
  Process:
    Group elements into sections using Headers and Titles
    Each section = Header/Title + all following elements
                   until next Header/Title of same level
    Assign section_type using document_type context
  Output: silver_parsed_sections
    section_id, doc_id, section_name, section_type,
    page_start, page_end,
    elements: [list of element_ids in this section]

  WHY THIS IS BETTER THAN RULE-BASED SECTION DETECTION:
    ai_parse_document already found the headers
    You are assembling not detecting
    Much more reliable

        ↓

STEP 4A: SECTION-AWARE CHUNKING
  Input:  silver_parsed_sections + bronze_parsed_elements
  Process:
    For each section:
      Get all NarrativeText elements in sequence
      Apply recursive chunking within section
      Merge small chunks
      Split large chunks
      Tables handled separately (not chunked as text)
  Output: gold_document_chunks

        ↓ (parallel with 4A)

STEP 4B: TABLE EXTRACTION
  Input:  bronze_parsed_elements WHERE type = Table
  Process:
    Each table element → parse into rows/columns
    Classify table type
    Store as structured data
  Output: gold_extracted_tables
  Why parallel: tables are already separated by parser
                no dependency on chunking

        ↓

STEP 5: FIELD EXTRACTION FROM PARSED RESULTS
  Input:  silver_parsed_sections (for targeted extraction)
          gold_extracted_tables (for numeric fields)
          bronze_parsed_elements (for specific element types)
  Process:
    LLM extracts fields using parsed structure as context
    NOT from raw text (much better quality)
    Targets specific section_types for specific fields
  Output: gold_extracted_fields, gold_extracted_metrics

        ↓

STEP 6: EMBEDDING
  Input:  gold_document_chunks WHERE status = pending
  Output: vectors → vector store
```

---

## Why Extraction After Chunking Works Better

```
EXTRACTION FROM RAW TEXT (old way)
  LLM reads 50 pages of mixed content
  Must find eligible categories buried in legal text
  High noise, lower accuracy
  Expensive (large context window)

EXTRACTION FROM PARSED + SECTIONED RESULTS (new way)
  LLM reads only the eligible_categories section
  Content is already clean (parser removed headers/footers)
  Tables already separated and structured
  Low noise, higher accuracy
  Cheaper (smaller targeted context)

EXAMPLE:
  Extracting total_allocated from allocation report

  OLD WAY:
    Send full allocation report text to LLM
    Ask: what is the total allocated amount?
    LLM must find it in 30 pages

  NEW WAY:
    silver_parsed_sections WHERE section_type = allocation_summary
    gold_extracted_tables WHERE table_type = allocation_table
    Send only these to LLM
    Ask: what is the total allocated amount?
    LLM reads 1 page + 1 table
    Much more reliable
```

---

## How Parsed Results Feed Extraction

```
THREE SOURCES FROM ai_parse_document
USED FOR EXTRACTION:

SOURCE 1: SECTION TEXT
  silver_parsed_sections.raw_text
  Used for: text-based fields
  e.g. framework_version, reviewer_name,
       reporting_commitments, overall_rating

SOURCE 2: EXTRACTED TABLES
  gold_extracted_tables
  Used for: numeric fields
  e.g. total_allocated, allocation_by_category,
       scope_1_emissions, impact_metrics

SOURCE 3: LIST ITEMS
  bronze_parsed_elements WHERE type = ListItem
  AND section_type = eligible_categories
  Used for: list-based fields
  e.g. eligible_categories[], exclusion_list[]
  Lists are already parsed as individual items
  No need to ask LLM to parse a list from prose
```

---

## The Dependency Tree: Revised

```
PDF
 │
 └──→ ai_parse_document → bronze_parsed_elements
           │
           └──→ Document Classification
                     │
                     └──→ Section Assembly → silver_parsed_sections
                               │
                    ┌──────────┴──────────┐
                    │                     │
              STEP 4A                STEP 4B
              Chunking               Table Extraction
              gold_document_chunks   gold_extracted_tables
                    │                     │
                    └──────────┬──────────┘
                               │
                         STEP 5: Field Extraction
                         Uses sections + tables + list items
                         gold_extracted_fields
                               │
                         STEP 6: Embedding
                         gold_document_chunks → vectors
```

---

## What Changes in the Delta Tables

```
BRONZE
  bronze_parsed_elements        ← NEW (replaces bronze_raw_pages)
    element_id
    doc_id
    issuer_id
    element_type                (Title/NarrativeText/Table/ListItem)
    content
    page_number
    sequence_order
    parent_id

SILVER
  silver_parsed_sections        ← SAME but now assembled from elements
    section_id                      not detected from raw text
    doc_id
    section_name
    section_type
    page_start
    page_end
    element_ids                 ARRAY of element_ids in this section

GOLD
  gold_document_chunks          ← SAME
  gold_extracted_tables         ← NOW COMES FROM PARSER not pdfplumber
  gold_extracted_fields         ← NOW USES SECTIONS + TABLES as input
  gold_extracted_metrics        ← SAME
```

---

## The Practical Benefit

```
ai_parse_document does the hard structural work
  Separates tables from text
  Identifies headers and titles
  Preserves reading order
  Tags element types

YOU BUILD ON TOP OF THAT STRUCTURE
  Section assembly uses identified headers
  Chunking uses clean NarrativeText elements
  Table extraction uses pre-identified Table elements
  Field extraction targets specific sections

RESULT:
  Less rule-writing
  Less regex
  Less prompt engineering for structure
  More reliable outputs
  Cleaner chunks
  Better field extraction
```

---

## Summary: The Revised Order

```
1. ai_parse_document
   PDF → typed elements (bronze)

2. Document classification
   Elements → document_type + year

3. Section assembly
   Elements + document_type → sections (silver)

4A. Section-aware chunking     ┐ parallel
4B. Table extraction           ┘

5. Field extraction
   Sections + tables + list items → structured fields (gold)

6. Embedding
   Chunks → vectors

KEY INSIGHT:
  ai_parse_document output is the foundation for BOTH
  chunking (uses structure)
  extraction (uses content + structure)
  You are not doing two separate parses
  One parse, two uses
```
# Table Storage: Your Approach is Right

---

## Your Instinct is Correct

```
NOT EVERY TABLE NEEDS TO BE STORED AS STRUCTURED DATA
MOST TABLES JUST NEED TO BE RETRIEVABLE AS TEXT

The overhead of parsing every table into
rows/columns/typed fields is not worth it
for tables that will only ever be read, not computed on
```

---

## Two Types of Tables in These Documents

```
TYPE 1: COMPUTATION TABLES
  You will run arithmetic or comparisons on these
  Examples:
    Allocation breakdown (amounts, percentages)
    Emissions by year (scope 1, 2, 3)
    Impact metrics (values, units)
    Capex breakdown (green vs fossil)

  These need structured storage
  You will query: WHERE category = 'Renewable Energy'
  You will compute: SUM(amount), year over year change
  You will compare: allocated vs raised

TYPE 2: REFERENCE TABLES
  You will read these but not compute on them
  Examples:
    Eligibility criteria table
    ICMA alignment table in SPO
    Project list table
    SDG mapping table
    Assurance scope table

  These do NOT need structured storage
  Just need to be retrievable as text
  Pipe-concatenated string is perfect
```

---

## Your Proposed Storage Approach

```
FOR TYPE 2 TABLES (reference, read-only):

  doc_id      | table_id  | table_header        | table_content
  ────────────────────────────────────────────────────────────────
  uuid-001    | tbl_001   | Eligible Categories | Renewable Energy |
              |           |                     | Solar, Wind, Hydro |
              |           |                     | Min capacity 1MW ||
              |           |                     | Green Buildings |
              |           |                     | NZEB standard ||

  PIPE SYMBOL USAGE:
    Single pipe  |   separates cells within a row
    Double pipe  ||  separates rows
    Clean, readable, searchable
    LLM can parse this easily when retrieved
    No complex JSON, no nested structures
```

---

## What to Store Per Table

```
COLUMNS FOR ALL TABLES (both types)

  table_id          UUID
  doc_id            UUID FK → documents
  issuer_id         TEXT
  document_type     TEXT
  document_year     INT
  page_number       INT
  table_sequence    INT     (1st table, 2nd table on page)
  table_header      TEXT    (text above table = likely title)
  table_content     TEXT    (pipe-concatenated)
  table_type        TEXT    (computation / reference)
  row_count         INT
  col_count         INT
  needs_structured  BOOL    (does this need Type 1 treatment?)
```

---

## Decision: Which Tables Get Structured Treatment

```
RULE:
  If you will ever run SQL on the values → structured
  If you will only ever retrieve and read → pipe text

STRUCTURED (needs_structured = TRUE)
  allocation_by_category    amounts + percentages
  emissions_by_year         scope 1/2/3 numeric values
  impact_metrics            metric values + units
  capex_breakdown           green/fossil amounts
  bond_terms_summary        issue size, dates

PIPE TEXT (needs_structured = FALSE)
  eligibility_criteria      read only, no computation
  icma_alignment            read only, verdict text
  project_list              read only, descriptive
  sdg_mapping               read only, reference
  assurance_scope           read only, reference
  any_unknown_table         default to pipe text
                            can always upgrade later
```

---

## How This Reduces Overhead

```
BEFORE (store everything structured)
  Every table → detect headers → type columns →
  validate → create schema → insert rows
  Fails on merged cells, irregular tables
  Maintenance nightmare for 50+ tables across documents

AFTER (your approach)
  Every table → pipe concatenate → store one row
  Simple, never fails, always retrievable
  For the 5-6 tables that need computation:
    ALSO parse into structured metrics tables
    These are known, predictable tables
    Worth the extra effort

RESULT:
  One simple table stores ALL tables as text
  Five structured tables store COMPUTATION tables
  No overhead for reference tables
  LLM can read pipe text naturally
```

---

## How Retrieval Uses This

```
Q&A: "What are the eligible categories for H&M?"

RETRIEVAL:
  Query gold_document_tables
  WHERE issuer_id = 'hm'
  AND document_type = 'green_bond_framework'
  AND table_header LIKE '%eligible%'

  Returns:
  "Eligible Categories | Criteria | Threshold ||
   Renewable Energy | Solar Wind Hydro | Min 1MW ||
   Green Buildings | NZEB certified | EPC A rating ||
   Clean Transport | Electric vehicles | Zero direct emissions"

  LLM reads this naturally
  No structured parsing needed
  Answer is accurate and sourced
```

---

## The Table Storage in Context of Full Pipeline

```
ai_parse_document
  identifies Table elements
        ↓
FOR EACH TABLE ELEMENT:
  Extract table_header (text above table)
  Concatenate cells with pipe separator
  Store in gold_document_tables (one row per table)
  Classify: computation or reference?
  IF computation:
    ALSO parse into structured metrics table
    (allocations, emissions, impact_metrics etc.)
        ↓
gold_document_tables     ← ALL tables, pipe text
gold_extracted_metrics   ← COMPUTATION tables only, structured
```

---

## Summary

```
YOUR UNDERSTANDING IS CORRECT

Store ALL tables as:
  doc_id + table_id + table_header + pipe_concatenated_content
  One row per table
  Simple, always works, LLM-readable

For the small number of computation tables:
  ALSO store as structured rows in metrics tables
  Only 5-6 table types need this treatment
  Known in advance, worth the extra parsing

BENEFIT:
  No complex table parsing for 80% of tables
  Full structured querying for the 20% that need it
  Single retrieval pattern for all table content
  Reduced storage overhead
  Reduced pipeline complexity
```

---

**One row per table. Pipe-separated. Simple.**

**Only parse to structured when you need to compute.**

**Ready for Step 5: Field Extraction?**
# Revised Document Processing Flow

---

## The Workflow

```
INPUT
  issuer_id + isin (provided)
  PDF documents in folder
        │
        ▼
STEP 1: ai_parse_document
  PDF → typed elements
    Title
    NarrativeText
    Table
    ListItem
    Header
    Footer
  Output → bronze_parsed_elements
  One row per element
  Carries: element_type, content, page_number, sequence_order
        │
        ▼
STEP 2: DOCUMENT CLASSIFICATION
  Input:  Title + NarrativeText elements from page 1-3
  LLM assigns:
    document_type     (one of 9 standard types)
    document_year     (coverage year)
    confidence
  Output → documents table
        │
        ▼
STEP 3: SECTION ASSEMBLY
  Input:  all elements in sequence_order + document_type
  Group elements under their nearest Header/Title
  Assign section_type using document_type context
  Output → silver_parsed_sections
    section_id, section_name, section_type
    page_start, page_end
    element_ids[]
        │
        ▼
        ├─────────────────────────┐
        │                         │
        ▼                         ▼
STEP 4A: CHUNKING            STEP 4B: TABLE EXTRACTION
  Input:                       Input:
    silver_parsed_sections       bronze_parsed_elements
    NarrativeText +              WHERE type = Table
    ListItem elements
    per section                Process:
                                 Extract table_header
  Process:                       (text above table)
    Per section:                 Concatenate cells
      recursive split            with pipe separator
      at \n\n → \n → .           Classify table_type:
      check size vs               computation?
      section_type config         reference?
      merge if < MIN
      split if > MAX           Output →
                               gold_document_tables
  Output →                       One row per table
  gold_document_chunks           pipe-concatenated

        │                         │
        └─────────────────────────┘
                    │
                    ▼
STEP 5: FIELD EXTRACTION
  Input:
    silver_parsed_sections      (targeted section text)
    gold_document_tables        (pipe text tables)
    ListItem elements           (pre-parsed lists)

  Process per document_type:
    Target specific section_types
    LLM extracts fields using
    section text + relevant tables
    as context

  Two outputs:

  OUTPUT A: gold_extracted_fields
    All fields as key-value pairs
    doc_id, issuer_id, field_name,
    field_value, confidence

  OUTPUT B: gold_extracted_metrics
    Computation tables parsed to structured rows
    Only for tables WHERE needs_structured = TRUE
    allocations, emissions,
    impact_metrics, capex_breakdown,
    bond_terms, framework_details,
    spo_details
        │
        ▼
STEP 6: EMBEDDING
  Input:  gold_document_chunks
          WHERE embedding_status = pending
  Batch embed chunk text
  Write vectors → Vector Store
  Update embedding_status = complete
        │
        ▼
STEP 7: VALIDATION + FLAGS
  Input:  gold_extracted_metrics
          gold_extracted_fields
          documents table
  Run cross-checks:
    Amount reconciliation
    Framework version match
    Category eligibility
    Document completeness
    Date logic
  Output → flags table
           document_inventory
        │
        ▼
OUTPUT
  gold_document_chunks      → Vector Store (RAG ready)
  gold_document_tables      → Queryable table text
  gold_extracted_metrics    → Structured for computation
  gold_extracted_fields     → Key-value field store
  flags                     → Verification findings
  document_inventory        → What exists per issuer
```

---

## The Delta Table Lineage

```
BRONZE                SILVER                    GOLD
──────────────────────────────────────────────────────────────
bronze_               silver_                   gold_
parsed_elements       parsed_sections           document_chunks
                                                document_tables
                                                extracted_fields
                                                extracted_metrics
──────────────────────────────────────────────────────────────
Raw parse output      Structured sections       Ready for use
One row per element   One row per section       RAG + queries
```

---

## Parallel Steps

```
CAN RUN IN PARALLEL:     4A + 4B (both depend on Step 3 only)
MUST RUN IN SEQUENCE:    1 → 2 → 3 → 4 → 5 → 6 → 7
ACROSS DOCUMENTS:        Each document fully parallel
```

---

## What Each Step Depends On

```
STEP 1    depends on    PDF file
STEP 2    depends on    STEP 1
STEP 3    depends on    STEP 2
STEP 4A   depends on    STEP 3
STEP 4B   depends on    STEP 1 (Table elements already in bronze)
STEP 5    depends on    STEP 3 + STEP 4B
STEP 6    depends on    STEP 4A
STEP 7    depends on    STEP 5 + STEP 6
```
