## Implementing Option 3: Two-Table Architecture

### Step 1: Create the Two Tables

```sql
-- Table 1: All tables in pipe format (your existing approach)
CREATE OR REPLACE TABLE all_parsed_tables (
    file_id         STRING,
    element_id      STRING,
    page_id         STRING,
    pipe_content    STRING,        -- Revenue | Q1 | Q2 | Q3 ...
    classification  STRING,        -- COMPUTATIONAL / REFERENCE
    table_category  STRING,        -- revenue / costs / headcount / other
    processed_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);

-- Table 2: Normalized metrics (only for COMPUTATIONAL tables)
CREATE OR REPLACE TABLE computed_metrics (
    file_id         STRING,
    element_id      STRING,        -- FK back to all_parsed_tables
    table_category  STRING,        -- revenue / costs / headcount
    metric_name     STRING,        -- total_revenue / q1_costs
    metric_value    FLOAT,         -- 500000.00
    string_value    STRING,        -- fallback for non-numeric
    period          STRING,        -- Q1 2024 / FY2023
    unit            STRING,        -- USD / % / count
    extracted_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);
```

---

### Step 2: Parse and Populate Table 1

```sql
-- Step 2a: Parse document and store all tables in pipe format
INSERT INTO all_parsed_tables (file_id, element_id, page_id, pipe_content, classification, table_category)

WITH parsed AS (
    SELECT 
        fileid,
        element_id,
        page_id,
        content,
        type
    FROM TABLE(ai_parse_document('@my_stage/document.pdf'))
    WHERE LOWER(type) LIKE '%table%'
),

classified AS (
    SELECT 
        fileid,
        element_id,
        page_id,
        -- Convert to pipe format
        ai_generate(
            'Convert this table to pipe-delimited format.
             Rules:
             - One row per line
             - Columns separated by |
             - Keep all values as-is
             - No extra formatting
             Table: ' || content
        ) AS pipe_content,

        -- Classify table type
        ai_generate(
            'Classify this table. Reply with ONE word only.
             COMPUTATIONAL = numbers, calculations, totals, financial figures
             REFERENCE = codes, descriptions, static lookup data
             
             Table: ' || LEFT(content, 500)
        ) AS classification,

        -- Categorize if computational
        ai_generate(
            'What category is this table? Reply ONE word only.
             Options: revenue / costs / headcount / balance_sheet / other
             
             Table: ' || LEFT(content, 500)
        ) AS table_category

    FROM parsed
)

SELECT 
    fileid,
    element_id,
    page_id,
    pipe_content,
    UPPER(TRIM(classification)),
    LOWER(TRIM(table_category))
FROM classified;
```

---

### Step 3: Populate Table 2 (Only Computational)

```sql
-- Step 3: Extract structured metrics from computational tables only
INSERT INTO computed_metrics (
    file_id, element_id, table_category,
    metric_name, metric_value, string_value, period, unit
)

WITH computational_tables AS (
    SELECT 
        file_id,
        element_id,
        table_category,
        pipe_content
    FROM all_parsed_tables
    WHERE classification = 'COMPUTATIONAL'
),

extracted AS (
    SELECT
        file_id,
        element_id,
        table_category,
        -- AI extracts metrics as JSON array
        ai_generate(
            'Extract all metrics from this table as a JSON array.
             Each metric should be:
             {
               "metric_name": "descriptive_snake_case_name",
               "metric_value": numeric_value_or_null,
               "string_value": "original_string",
               "period": "time_period_or_null",
               "unit": "USD or % or count or null"
             }
             
             Return ONLY the JSON array, no explanation.
             Table: ' || pipe_content
        ) AS metrics_json
    FROM computational_tables
)

-- Parse JSON array into rows
SELECT
    file_id,
    element_id,
    table_category,
    m.value:metric_name::STRING   AS metric_name,
    m.value:metric_value::FLOAT   AS metric_value,
    m.value:string_value::STRING  AS string_value,
    m.value:period::STRING        AS period,
    m.value:unit::STRING          AS unit
FROM extracted,
LATERAL FLATTEN(input => TRY_PARSE_JSON(metrics_json)) m;
```

---

### Step 4: Querying Both Tables

```sql
-- Query 1: LLM reads pipe content naturally (Table 1)
SELECT 
    file_id,
    table_category,
    pipe_content
FROM all_parsed_tables
WHERE file_id = 'doc1'
  AND classification = 'COMPUTATIONAL';

-- Query 2: SQL aggregations on metrics (Table 2)
SELECT 
    file_id,
    table_category,
    metric_name,
    SUM(metric_value) AS total,
    period
FROM computed_metrics
WHERE metric_name LIKE '%revenue%'
GROUP BY file_id, table_category, metric_name, period
ORDER BY total DESC;

-- Query 3: Join both tables for full context
SELECT 
    a.file_id,
    a.table_category,
    a.pipe_content,           -- full table for LLM
    c.metric_name,
    c.metric_value,
    c.period
FROM all_parsed_tables a
JOIN computed_metrics c
    ON  a.file_id    = c.file_id
    AND a.element_id = c.element_id
WHERE a.file_id = 'doc1';

-- Query 4: Compare same metric across documents
SELECT 
    file_id,
    metric_value,
    period
FROM computed_metrics
WHERE metric_name = 'total_revenue'
ORDER BY file_id, period;
```

---

### Step 5: Full Pipeline as a Stored Procedure

```sql
CREATE OR REPLACE PROCEDURE process_document(file_id STRING, stage_path STRING)
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN

    -- Step 1: Parse and store all tables
    INSERT INTO all_parsed_tables (file_id, element_id, page_id, pipe_content, classification, table_category)
    WITH parsed AS (
        SELECT fileid, element_id, page_id, content
        FROM TABLE(ai_parse_document(:stage_path))
        WHERE LOWER(type) LIKE '%table%'
    )
    SELECT
        fileid,
        element_id,
        page_id,
        ai_generate('Convert to pipe-delimited format. One row per line, | between columns. Table: ' || content),
        UPPER(TRIM(ai_generate('Reply COMPUTATIONAL or REFERENCE only. Table: ' || LEFT(content,500)))),
        LOWER(TRIM(ai_generate('Reply one word: revenue/costs/headcount/balance_sheet/other. Table: ' || LEFT(content,500))))
    FROM parsed;

    -- Step 2: Extract metrics for computational tables only
    INSERT INTO computed_metrics (file_id, element_id, table_category, metric_name, metric_value, string_value, period, unit)
    WITH extracted AS (
        SELECT
            file_id, element_id, table_category,
            ai_generate(
                'Return JSON array of metrics: 
                 [{"metric_name":"","metric_value":0,"string_value":"","period":"","unit":""}]
                 Table: ' || pipe_content
            ) AS metrics_json
        FROM all_parsed_tables
        WHERE file_id   = :file_id
          AND classification = 'COMPUTATIONAL'
    )
    SELECT
        file_id, element_id, table_category,
        m.value:metric_name::STRING,
        m.value:metric_value::FLOAT,
        m.value:string_value::STRING,
        m.value:period::STRING,
        m.value:unit::STRING
    FROM extracted,
    LATERAL FLATTEN(input => TRY_PARSE_JSON(metrics_json)) m;

    RETURN 'Processed: ' || :file_id;
END;
$$;

-- Run for each document
CALL process_document('doc1', '@my_stage/doc1.pdf');
CALL process_document('doc2', '@my_stage/doc2.pdf');
```

---

### What You Get

```
all_parsed_tables          computed_metrics
─────────────────          ────────────────
doc1 | t1 | ref  | ...     doc1 | t2 | revenue | total_revenue | 500K
doc1 | t2 | comp | ...     doc1 | t2 | revenue | q1_revenue    | 100K
doc1 | t3 | ref  | ...     doc1 | t2 | revenue | q2_revenue    | 150K
doc2 | t1 | comp | ...     doc2 | t1 | costs   | total_costs   | 300K
doc2 | t2 | ref  | ...     doc2 | t1 | costs   | q1_costs      |  80K
...                        ...
100 docs, 500 tables  →    2 tables, scales forever ✅
```

Want me to add error handling or adjust the metric extraction prompt?
