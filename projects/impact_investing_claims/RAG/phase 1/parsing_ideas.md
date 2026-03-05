# Document Parsing + Section-Aware Chunking

---

## The Two-Step Core Idea

```
STEP 1: AI PARSE (Databricks AI Functions)
  PDF → structured sections + extracted fields
  LLM understands what each part of the document means
  Output: sections with labels, extracted fields per doc type

STEP 2: SECTION-AWARE CHUNKING
  Sections → clean meaningful chunks
  Respects section boundaries
  Dynamically adjusts size
  Merges small, splits large
  Output: chunks ready for embedding
```

---

## Step 1: AI Parse Using Databricks AI Functions

### What Databricks AI Functions Give You
```
ai_parse()          extracts structured content from documents
ai_classify()       classifies text into categories
ai_extract()        pulls specific fields from text
ai_summarize()      summarizes sections

ALL RUN AS SQL FUNCTIONS
  SELECT ai_extract(text, schema) FROM documents
  Runs at scale on Delta tables
  No separate API calls to manage
  Results stored directly in Delta tables
```

### The Parse Flow
```
RAW PDF TEXT (per page, stored in Delta)
        ↓
ai_parse() call
  Input:  raw text + document_type + extraction schema
  Output: structured JSON per document
        ↓
TWO OUTPUTS IN PARALLEL:

  OUTPUT A: SECTIONS
    List of sections found in document
    Each section has:
      section_name
      section_type
      page_start
      page_end
      raw_text

  OUTPUT B: EXTRACTED FIELDS
    Document-type-specific fields
    Stored as structured columns
    Ready for cross-checks and verification
```

### What Gets Extracted Per Document Type
```
green_bond_framework
  eligible_categories     ARRAY
  exclusion_list          ARRAY
  framework_version       STRING
  proceeds_management     STRING
  reporting_commitments   STRING

second_party_opinion
  reviewer_name           STRING
  review_date             DATE
  framework_version_reviewed STRING
  overall_rating          STRING
  icma_alignment          JSON
  concerns                ARRAY

final_terms
  isin                    STRING
  issue_size_eur          NUMERIC
  issue_date              DATE
  maturity_date           DATE
  uop_categories          ARRAY

allocation_report
  total_proceeds          NUMERIC
  total_allocated         NUMERIC
  unallocated_amount      NUMERIC
  allocation_by_category  JSON
  verification_body       STRING
  assurance_level         STRING

impact_report
  metrics                 JSON
  methodology_disclosed   BOOLEAN
  reporting_boundary      STRING
  verification_body       STRING

sustainability_report
  scope_1                 NUMERIC
  scope_2                 NUMERIC
  scope_3                 NUMERIC
  reporting_year          INT
  taxonomy_alignment_pct  NUMERIC

annual_report
  green_capex             NUMERIC
  fossil_capex            NUMERIC
  total_capex             NUMERIC
  green_bond_mentioned    BOOLEAN
  climate_risk_rating     STRING
```

### The Delta Tables After AI Parse
```
bronze_raw_pages
  doc_id, issuer_id, isin, document_type,
  document_year, page_number, raw_text

silver_parsed_sections
  section_id, doc_id, issuer_id, isin,
  document_type, document_year,
  section_name, section_type,
  page_start, page_end, raw_text

silver_extracted_fields
  field_id, doc_id, issuer_id, isin,
  document_type, document_year,
  field_name, field_value, field_type,
  confidence
```

---

## Step 2: Section-Aware Flexible Chunking

### The Core Logic
```
FOR EACH SECTION in silver_parsed_sections:

  1. Get section_type
  2. Look up size config for that section_type
  3. Apply recursive chunking within section
  4. Check each chunk size:
       too small → merge with next chunk in same section
       too large → split further at sentence boundary
       just right → keep as is
  5. Attach full metadata to every chunk
  6. Write to gold_document_chunks
```

### Section Type Size Configuration
```
SECTION TYPE                MIN    TARGET    MAX
─────────────────────────────────────────────────
eligible_categories         200    400       600
exclusion_list              150    300       500
impact_metrics              150    300       500
methodology                 300    600       900
allocation_summary          200    400       600
legal_obligations           300    500       700
proceeds_management         200    400       600
reporting_commitments       200    400       600
spo_assessment              300    500       700
narrative_overview          400    700      1000
general_text                300    500       700
─────────────────────────────────────────────────
ALL SIZES IN CHARACTERS
```

### The Recursive Chunking Logic
```
GIVEN: section_text, section_type

STEP 1: INITIAL SPLIT
  Try splitting at paragraph boundaries (\n\n)
  If no paragraphs: split at sentence boundaries (. )
  If no sentences: split at word boundaries ( )
  This is the recursive part:
    Try largest natural boundary first
    Fall back to smaller boundaries
    Never split mid-word

STEP 2: SIZE CHECK EACH CHUNK
  Get MIN, TARGET, MAX for this section_type

  FOR EACH CHUNK:
    IF char_count < MIN:
      → MERGE candidate
        look at next chunk in same section
        if next chunk exists:
          merge current + next
          re-check merged size against MAX
          if merged > MAX: accept oversized, flag it
          if merged <= MAX: keep merged chunk
        if no next chunk:
          merge with previous chunk
          if no previous: keep as-is, flag as undersized

    IF char_count > MAX:
      → SPLIT candidate
        find sentence boundaries within chunk
        split at sentence boundary closest to TARGET
        re-check both resulting chunks against MIN
        if either < MIN: revert split, keep oversized, flag it

    IF MIN <= char_count <= MAX:
      → KEEP as-is

STEP 3: FINAL PASS
  Any remaining oversized chunks: flag but keep
  Any remaining undersized chunks: flag but keep
  Never discard a chunk
  Always flag anomalies
```

### Chunk Metadata
```
EVERY CHUNK CARRIES:

IDENTITY
  chunk_id          {issuer_id}_{doc_type}_{section_type}_{page}_{idx}
  doc_id            FK to documents
  section_id        FK to silver_parsed_sections
  issuer_id
  isin

DOCUMENT CONTEXT
  document_type
  document_year
  framework_version (where applicable)
  source_file

SECTION CONTEXT
  section_name      "Eligible Project Categories"
  section_type      "eligible_categories"
  page_number

CHUNK PROPERTIES
  chunk_index       position within section
  total_chunks_in_section
  char_count
  was_merged        BOOL
  was_split         BOOL
  is_oversized      BOOL
  is_undersized     BOOL

RETRIEVAL HELPERS
  embedding_status  pending / complete / failed
```

### The Delta Table After Chunking
```
gold_document_chunks
  chunk_id          TEXT PRIMARY KEY
  doc_id            UUID
  section_id        UUID
  issuer_id         TEXT
  isin              TEXT
  document_type     TEXT
  document_year     INT
  section_name      TEXT
  section_type      TEXT
  page_number       INT
  chunk_index       INT
  total_in_section  INT
  text              TEXT
  char_count        INT
  was_merged        BOOL
  was_split         BOOL
  is_oversized      BOOL
  is_undersized     BOOL
  embedding_status  TEXT
  created_at        TIMESTAMP
```

---

## The Medallion Architecture

```
BRONZE                    SILVER                    GOLD
──────────────────────────────────────────────────────────
Raw ingestion             Parsed + structured       Ready for use

bronze_raw_pages          silver_parsed_sections    gold_document_chunks
  raw PDF text per page     sections with labels      chunked, metadata-rich
                            page ranges               embedding-ready

                          silver_extracted_fields   gold_extracted_metrics
                            structured fields         verified, typed
                            per document type         cross-check ready
──────────────────────────────────────────────────────────

BRONZE → SILVER:  AI parse (Databricks ai_parse)
SILVER → GOLD:    Section-aware chunking + field validation
```

---

## Why This Works Well in Databricks

```
BRONZE TABLE
  Load raw PDF text using Databricks document loaders
  One row per page
  Partitioned by issuer_id, document_type

SILVER TRANSFORMATION (SQL + AI Functions)
  SELECT
    doc_id,
    issuer_id,
    document_type,
    ai_extract(
      raw_text,
      schema_for_document_type    ← dynamic per doc type
    ) AS extracted_fields,
    ai_classify(
      raw_text,
      section_types_list
    ) AS section_type
  FROM bronze_raw_pages

GOLD TRANSFORMATION (Python UDF or notebook)
  Apply chunking logic as Python UDF
  Runs distributed across cluster
  Writes directly to gold_document_chunks
  Partitioned by issuer_id for fast retrieval

EVERYTHING IS A DELTA TABLE
  Full history (time travel)
  Re-run any stage without losing previous results
  Audit trail built in
  Incremental processing (only new rows)
```

---

## The Section-to-Chunk Flow End to End

```
silver_parsed_sections
  section: "Eligible Project Categories"
  section_type: eligible_categories
  raw_text: 1,847 characters
  page_start: 3, page_end: 5
        ↓
CHUNKING APPLIED
  size config: MIN=200, TARGET=400, MAX=600
  initial split at \n\n → 5 raw chunks:
    chunk 0: 412 chars  ✓ keep
    chunk 1: 380 chars  ✓ keep
    chunk 2: 167 chars  ✗ too small → merge with chunk 3
    chunk 3: 203 chars  → merged: 370 chars ✓ keep
    chunk 4: 685 chars  ✗ too large → split at sentence
             → 341 chars ✓ keep
             → 344 chars ✓ keep
  final: 5 clean chunks from original 5 raw chunks
        ↓
gold_document_chunks
  5 rows written
  All carry section_type = eligible_categories
  All carry issuer_id, doc_id, section_id
  embedding_status = pending
```

---

## What Comes After Chunking

```
gold_document_chunks
  embedding_status = pending
        ↓
EMBEDDING STAGE (separate step)
  Read all pending chunks
  Batch embed using OpenAI or
  Databricks embedding model
  Write vectors to Vector Store (Weaviate / Databricks VS)
  Update embedding_status = complete
        ↓
READY FOR RAG
  Vector search on chunk text
  Metadata filters on section_type, issuer_id, document_type
  Structured queries on gold_extracted_metrics
```

---

**Bronze → Silver → Gold.**

**AI parse gives you structure and meaning.**

**Section-aware chunking gives you clean, retrievable segments.**

**Every chunk knows exactly where it came from and what it means.**

**Ready to define the embedding stage?**
