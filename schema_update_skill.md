# Schema Update Skill
**Use this only after a repo profile has been reviewed and a gap has been confirmed by the team.**
**Never update the schema mid-parser. Schema changes happen separately.**

---

## When This Skill Is Used

A new repo was profiled using `repo_exploration_skill.md`.
The profile identified a gap — a concept with no matching entity or edge type.
The team has reviewed the gap and agreed the schema should be extended.

This skill adds the new type cleanly and consistently.

---

## Steps

### Step 1 — Read the current schema

Read `src/lineage/data_lineage_schema.md` in full before making any change.
Confirm the proposed new type does not already exist under a different name.

### Step 2 — Add the new entity type (if needed)

Add a new row to the Entity Types table. Fill in every column:

| entity_type | platform | layer | description | github_file |
|---|---|---|---|---|
| `NEW_TYPE` | which system | which layer | one plain sentence | path pattern or null |

Add the key format rule to the Key Format Rules table.

### Step 3 — Add the new edge type (if needed)

Add a new row to the Edge Types table. Fill in every column:

| edge_type | source | target | parser | description |
|---|---|---|---|---|
| `new_edge` | source entity_type | target entity_type | which parser will emit it | one sentence |

### Step 4 — Update the status table

At the bottom of the schema file, update the repo status table to show the new repo as profiled.

### Step 5 — Write a change note

At the top of the schema file, add one line under a `## Change History` section:

```
- 2026-11-06 · Added CLASSIFICATION_RULE entity type and GOVERNS edge type
  for kb-cdp-tenant-protect-downstream · proposed by [your name] · reviewed by [reviewer]
```

---

## Rules

- One change per skill run. Do not add multiple new types in one go.
- The team must agree before this skill runs. Never update unilaterally.
- All existing parsers continue to work after the update — do not rename existing types.
- After updating, run all existing tests to confirm nothing broke.
