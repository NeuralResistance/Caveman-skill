# Agent Prompt Pack
**Data Exploration + Parser Creation Workflow**
*Use these prompts in order. One prompt per session. Do not skip steps.*

---

## How This Works — Plain English

There are three skills and four prompts. Think of it like this:

```
New repo arrives
      ↓
Prompt 1 — EXPLORE    "What is in this repo?"
      ↓
Prompt 2 — COMPARE    "Does it fit our schema or do we need to extend it?"
      ↓
(Team reviews. Agrees on any schema changes.)
      ↓
Prompt 3 — UPDATE     "Update the schema with the agreed changes."  ← only if needed
      ↓
Prompt 4 — PARSE      "Now build the parser."
```

The skills are the instruction manuals the agent reads before it does anything.
The prompts are what you type into Copilot to start each step.

---

## Prompt 1 — Explore a New Repo

**What it does:** Profiles the new repo. Counts file types. Explains what each file defines in plain English. Does not touch the schema or write any code.

**Type this into Copilot:**

> *"Read @skills/repo_exploration_skill.md and follow it exactly for the repo at [paste repo path here]. Do not build a parser. Do not change the schema. Produce only the profile report as described in the skill and save it to output/repo_profile_[repo name].md."*

**What you get back:** A markdown report with five sections — what the repo contains, what it defines, how it compares to the existing schema, any gaps, and open questions.

**What to do next:** Read the profile. If there are no gaps — go straight to Prompt 4. If there are gaps — go to Prompt 2.

---

## Prompt 2 — Detailed Schema Comparison

**What it does:** Takes the profile from Prompt 1 and produces a side-by-side comparison of what the new repo introduces versus what the existing schema already covers. Proposes specific additions if needed. Easy to read — no code.

**Type this into Copilot:**

> *"Read @skills/repo_exploration_skill.md, @src/lineage/data_lineage_schema.md, and @output/repo_profile_[repo name].md. Produce a detailed comparison table showing every concept found in the repo alongside the closest existing schema type — or a gap description if nothing matches. Then list every proposed schema addition from the profile as a numbered recommendation with a one-sentence plain English justification for each. Output to output/schema_comparison_[repo name].md."*

**What you get back:**

```
Concept in repo          | Existing schema type  | Match?  | Gap / Proposal
─────────────────────────|───────────────────────|─────────|──────────────────
Data classification rule | —                     | No gap  | Propose: CLASSIFICATION_RULE
Landing zone config      | SNOWFLAKE_RELATION    | ✓ Covered | —
Sensitivity label file   | —                     | No gap  | Propose: SENSITIVITY_LABEL
```

**What to do next:** Share this with the team. Agree which proposals to accept. If none — go to Prompt 4. If some — go to Prompt 3.

---

## Prompt 3 — Update the Schema (Only If Team Agreed)

**What it does:** Adds the agreed new types to the authoritative schema cleanly and consistently. Writes a change note so the history is tracked.

**Only run this after the team has reviewed and agreed on the proposals from Prompt 2.**

**Type this into Copilot:**

> *"Read @skills/schema_update_skill.md and @src/lineage/data_lineage_schema.md. Add the following agreed changes only — do not add anything else: [list the agreed entity types and edge types here]. Follow the skill exactly — add rows to the correct tables, add key format rules, and write a change note at the top of the schema. After updating, confirm the existing parser test suite still passes."*

**What you get back:** An updated `src/lineage/data_lineage_schema.md` with the new types added and a change history entry at the top.

**What to do next:** Confirm the tests pass. Then go to Prompt 4.

---

## Prompt 4 — Build the Parser

**What it does:** Builds the actual Python parser for the new repo. The caveman-style skill keeps instructions short so the agent uses fewer tokens and stays focused.

**Type this into Copilot:**

> *"Read @skills/parser_creation_skill.md, @src/lineage/data_lineage_schema.md, and @output/repo_profile_[repo name].md. Build parsers/[repo name]_parser.py following the skill exactly. Use only entity types and edge types that exist in the current schema. For every node and edge, cite the source file. Mark confidence as confirmed only when the file explicitly declares the connection — otherwise mark it inferred. After writing the parser, run it against the real repo and save the output to output/[repo name]_raw.json. Then run the validator and report the result."*

**What you get back:**
- `parsers/[repo name]_parser.py` — the new parser
- `output/[repo name]_raw.json` — the validated JSON payload
- A validation report showing all checks pass

---

## For New Developers — Start Here

You are adding a new repo to a lineage system that already has:
- 5 working parsers in `src/lineage/parsers/`
- An authoritative schema in `src/lineage/data_lineage_schema.md`
- Three skill files in `skills/` that tell Copilot exactly how to work

**Your job is simple:**
1. Run Prompt 1 on your repo
2. Read the profile that comes back
3. If there are gaps — run Prompt 2 and show the team
4. If the team agrees on schema changes — run Prompt 3
5. Run Prompt 4 to build the parser

**Do not:**
- Write a parser before running Prompt 1
- Change the schema without running Prompt 2 first
- Skip the validator at the end of Prompt 4

**If something is confusing:** Read the skill file for that step. The skill file is the full explanation. The prompt is just the trigger.

---

## Quick Reference — Which Skill Does What

| Skill file | What it does | Token cost | When to use |
|---|---|---|---|
| `repo_exploration_skill.md` | Profiles a repo, compares to schema | Medium — detailed | Every new repo, every time |
| `parser_creation_skill.md` | Builds a schema-compliant parser | Low — caveman style | After profile is done |
| `schema_update_skill.md` | Extends the schema safely | Low — short steps | Only when team agrees |
