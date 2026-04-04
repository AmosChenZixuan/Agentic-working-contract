# {{ project-name }} — Technical Design

> **Version:** 0.1
> **Last updated:** YYYY-MM-DD

---

## 1. What This Is

Brief description of the project — what it does, how it works at a high level, and what the primary user interface is. One paragraph.

## 2. Goals & Non-Goals

### Goals

- **Goal name**: Brief description of what this goal means in practice
- Keep goals concrete and limited — avoid vague aspirational language

### Non-Goals

- **Not** a description of what this is NOT — instead, state what the system intentionally excludes
- Examples: "Not a general-purpose X", "Not a cloud or multi-user service", "Does not do Y"

---

## 3. System Architecture

```
{{ ascii diagram showing major components and their relationships }}
```

Brief paragraph explaining how the pieces connect. Cover serving mode (dev vs. prod) if it's non-obvious.

---

## 4. Storage Layout

```
~/.{{ project-name }}/
├── config.json          {{ description }}
├── {{ name }}.db        {{ SQLite — what it stores }}
├── notes/               {{ what lives here, file naming convention }}
├── transcripts/         {{ what lives here, file naming convention }}
└── downloads/           {{ what lives here, cleanup policy }}
```

---

## 5. Database Schema

### `{{ table_name }}` — {{ purpose }}

| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `name` | TEXT | User-visible name |
| ... | ... | ... |

**Key invariants or constraints** about this table (e.g., "this table is the source of truth for X", "rows are immutable once Y").

### `{{ other_table }}` — {{ purpose }}

| Column | Type | Notes |
|---|---|---|
| ... | ... | ... |

---

## 6. Core Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| {{ e.g. "Note storage" }} | Files in `~/.{{ project }}/notes/` | Decouples lifecycle from DB; human-readable, portable |
| ... | ... | ... |

The rationale column is the most important — explain the tradeoff, not just the choice.

---

## 7. API Contracts

### `POST /api/{{ endpoint }}`

**Request:**
```json
{
  "field": "type — description"
}
```

**Response:** `200`
```json
{
  "field": "type — description"
}
```

**Errors:** `400` — description | `401` — description

### `GET /api/{{ endpoint }}`

...

---

## 8. Processing Pipeline / Data Flow

```
{{ stages described in order with inputs and outputs }}
```

Explain any non-obvious transformations, retry logic, or failure handling.

### Chunking / Segmentation Strategy

Describe the strategy (if applicable): how segments are merged, what size targets apply, what metadata is preserved and why.

### Note / Output Format

```markdown
{{ frontmatter and structure of the primary output file }}
```

---

## 9. Configuration Schema

```json
{
  "{{ section }}": {
    "{{ key }}": "{{ type }} — default, description }}"
  }
}
```

---

## 10. Open Questions

1. **{{ Question text }}** — Status: open | resolved | being investigated
2. **{{ Question text }}** — Status: open

---

## Appendix: Key Algorithms

### {{ algorithm name }}

{{ brief description of why this is non-obvious and how it works }}
