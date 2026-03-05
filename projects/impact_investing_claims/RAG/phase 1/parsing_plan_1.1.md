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
