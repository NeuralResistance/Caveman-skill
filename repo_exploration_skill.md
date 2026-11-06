# Repo Exploration Skill
**Use this skill whenever a new repository is being added to the lineage system.**
**Goal: understand what is in the repo, compare it to the authoritative schema, and produce a gap report before any parser is built.**

---

## When to Use This Skill

- A new repo has been added to the workspace
- You are unsure whether a repo introduces new node types or edge types
- Another developer needs to understand what a repo contains before writing a parser
- The schema needs to be reviewed before extending the system

---

## Inputs Required

| Input | Where to find it |
|---|---|
| New repo path | Provided by the developer |
| Authoritative schema | `src/lineage/data_lineage_schema.md` |
| Existing parsers | `src/lineage/parsers/` — read these to understand the established pattern |
| Existing profiles | `output/repo_profile_*.md` — read these to understand what has already been explored |

---

## Steps — Follow in Order, Do Not Skip

### Step 1 — Count and categorise what is in the repo

Walk the repo. For every file type found, record:
- File extension
- Count of files with that extension
- What those files appear to define (one plain-English sentence per type)

Do not read every file. Sample 2–3 files per type to understand the pattern.

Output format:
```
| File type | Count | What it defines |
|---|---|---|
| .sql | 47 | dbt model transformations |
| .yml | 23 | dbt source declarations and column metadata |
```

---

### Step 2 — Identify what concepts the repo introduces

For each file type, answer:
- What real-world thing does this file describe? (a job, a table, a rule, a channel?)
- Does it define how data moves, how data is shaped, or how data is governed?
- Does it reference other systems — Snowflake tables, S3 paths, Airflow DAGs?

Write one paragraph per file type. Plain English. No code.

---

### Step 3 — Compare against the authoritative schema

Read `src/lineage/data_lineage_schema.md`.

For each concept found in Step 2, check:
- Is there already an entity type in the schema that covers this? If yes — which one?
- Is there already an edge type that covers how this connects to other things? If yes — which one?
- If no match exists — describe the gap in one sentence

Format your comparison as a table:

```
| Concept found in repo | Covered by existing schema? | Existing type (if yes) | Gap description (if no) |
|---|---|---|---|
| Data classification rule | No | — | Defines sensitivity label per field — no CLASSIFICATION node type exists |
| Snowflake table DDL | Yes | SNOWFLAKE_RELATION | Covered by existing type |
```

---

### Step 4 — Propose schema extensions (only if gaps exist)

For each gap identified in Step 3, write a proposed extension:

```
## Proposed new entity type: CLASSIFICATION_RULE
- platform: governance
- layer: governance
- description: A rule that assigns a sensitivity label to a field or table
- key format: classification:<rule_name>
- github_file: path to the file that defines the rule
- Justification: kb-cdp-tenant-protect-downstream defines sensitivity labels per field
  that are referenced downstream. No existing entity type covers this concept.

## Proposed new edge type: GOVERNS
- source: CLASSIFICATION_RULE → target: SNOWFLAKE_RELATION or DBT_MODEL
- description: A classification rule governs the sensitivity of a target object
- Justification: Needed to link classification rules to the objects they apply to
```

Write one block per proposed addition. Keep it short. Do not propose something the existing schema already covers under a different name.

---

### Step 5 — Write the profile report

Output to `output/repo_profile_<repo_name>.md` with these sections:

1. **What this repo contains** — file type table from Step 1
2. **What it defines in plain English** — paragraphs from Step 2
3. **Comparison against schema** — table from Step 3
4. **Proposed schema extensions** — blocks from Step 4 (or "None — existing schema covers all concepts found")
5. **Open questions** — anything ambiguous that a developer or SME needs to clarify

---

## Rules

- **Never invent** — only report what you actually found in the files
- **Label everything** — CONFIRMED if you read the file, INFERRED if you matched a pattern
- **Do not modify the schema** — the profile is a proposal only. Schema changes happen in a separate step after review
- **Do not build a parser** — this skill is exploration only. Parser creation uses a separate skill
- **One repo per run** — do not profile two repos in one skill execution
- **Keep it readable** — the output will be read by developers who did not write this code. Plain English throughout

---

## Comparison Cheat Sheet — Existing Entity Types

Use this when checking Step 3. If a concept fits one of these, it is already covered:

| If the repo contains... | Use this entity type |
|---|---|
| A legacy BASIC extract program | `PROGRAM` |
| A MultiValue source file (ACCOUNT, TRAN) | `MV_FILE` |
| A pipe-delimited output file | `EXTRACT_FILE` |
| An HVR channel config | `HVR_CHANNEL` |
| An HVR named location | `HVR_LOCATION` |
| An S3 bucket path | `S3_PATH` |
| A product directory | `CDP_PRODUCT` |
| An Airflow DAG Python file | `AIRFLOW_DAG` |
| An Airflow task or operator | `AIRFLOW_TASK` |
| A dbt source declaration (.yml) | `DBT_SOURCE` |
| A dbt SQL model | `DBT_MODEL` |
| A physical Snowflake table or view | `SNOWFLAKE_RELATION` |
| A schemachange migration script | `SCHEMACHANGE_MIGRATION` |

If the concept does not appear in this table — it is a gap. Propose a new type in Step 4.
