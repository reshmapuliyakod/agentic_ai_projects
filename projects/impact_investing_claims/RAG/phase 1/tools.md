# Tools to Define: Document Processing Agent

---

## The Precise Answer

```
9 TOOLS
3 UTILITIES (shared helpers, not agent tools)
```

---

## The 9 Agent Tools

```
TOOL 1: scan_document_folder
  Input:  folder path
  Output: list of files found, list of expected docs missing
  Does:   inventory check, flags missing critical documents

TOOL 2: classify_document
  Input:  file path + first 2 pages text
  Output: document_type, processing_strategy, confidence
  Does:   filename rules → content rules → LLM fallback

TOOL 3: extract_text
  Input:  file path + document_type + strategy
  Output: pages with text, page numbers, extraction quality flag
  Does:   full extraction OR targeted section extraction
          depending on strategy assigned by classifier

TOOL 4: detect_sections
  Input:  extracted pages + document_type
  Output: list of sections with name, type, page range, text
  Does:   heading detection → document-type markers →
          LLM fallback, builds section tree

TOOL 5: chunk_sections
  Input:  list of sections + document_type
  Output: list of chunks with full metadata
  Does:   section-aware chunking, merge if too small,
          split if too large, attach all metadata

TOOL 6: extract_tables
  Input:  extracted pages + document_type
  Output: list of tables with type, page, CSV path, DataFrame
  Does:   table detection, classification, validation,
          saves CSV to file system

TOOL 7: extract_metadata
  Input:  extracted text + document_type
  Output: structured metadata dictionary
  Does:   regex → keyword match → LLM fallback
          for all type-specific fields

TOOL 8: store_document
  Input:  chunks + tables + metadata
  Output: storage confirmation, any failures
  Does:   chunks → Weaviate (embed + store)
          tables → PostgreSQL
          metadata → document registry
          all in one transaction-aware call

TOOL 9: validate_and_flag
  Input:  issuer_id (runs across all stored docs for issuer)
  Output: validation report, list of flags with severity
  Does:   all 5 cross-checks
          amount reconciliation
          framework version match
          category eligibility
          document completeness
          date logic
```

---

## The 3 Shared Utilities

```
Not agent tools - called internally by tools above

UTILITY 1: pdf_reader
  Called by: extract_text
  Does:      PyMuPDF first, pdfplumber fallback,
             returns raw text per page

UTILITY 2: section_size_config
  Called by: chunk_sections
  Does:      returns min/target/max chunk size
             for a given section_type
             single config object, easy to tune

UTILITY 3: generate_chunk_id
  Called by: chunk_sections
  Does:      deterministic ID generation
             issuer + doc_type + section_type + page + index
```

---

## The Tool Execution Order

```
scan_document_folder        ← always first
       ↓
classify_document           ← once per file
       ↓
extract_text                ← once per file
       ↓
detect_sections             ← once per file
       ↓
chunk_sections              ← once per file
extract_tables              ← once per file (parallel with chunking)
extract_metadata            ← once per file (parallel with chunking)
       ↓
store_document              ← once per file
       ↓
validate_and_flag           ← once per issuer (after all files stored)
```

---

## What the Orchestrator Does

```
NOT a tool itself
The agent brain that:
  Calls tools in order
  Handles tool failures (continue vs stop)
  Passes outputs between tools
  Tracks progress per file
  Compiles final report
```

---

## Summary

```
TOTAL TOOLS:        9
TOTAL UTILITIES:    3
ORCHESTRATOR:       1 (agent brain, not a tool)

TOOLS PER PHASE OF PROCESSING:
  Discovery:        1  (scan_document_folder)
  Classification:   1  (classify_document)
  Extraction:       2  (extract_text, detect_sections)
  Transformation:   3  (chunk_sections, extract_tables,
                        extract_metadata)
  Storage:          1  (store_document)
  Validation:       1  (validate_and_flag)
```
