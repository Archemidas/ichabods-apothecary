# Knowledge Store v2 — Design Notes

Sourced from the project's Google Doc (v2 updates).

## Goals

1. **Rank-based access** — admin / instructor / student tiers
2. **Automated parsing, ingestion, archiving** — when a note saves, derive metadata
3. **Self-categorizing** — across multiple parameters (lesson, tag, topic, evolution)
4. **Searchable** — full-text across note, title, tags, quote, session
5. **Recursive learning** — track shifts in interpretation; every revision becomes history
6. **Evolution tracking** — explicit UI for "this note has evolved N revisions"

## Demo implementation

See `ichabods-demo.html` for the working implementation. Key functions:

- `saveAnnotation(lineId)` — handles both new + revision (revisions push prior version into `history[]`)
- `saveFreeNote()` — standalone note not tied to a transcript line
- `autoCategorize(text)` — keyword dictionary; six categories: preparation, identification, dosage, patience, safety, tradition
- `filteredMyAnnotations` (computed) — applies search + tag filter
- `availableTags` (computed) — derives all unique tags across user's notes

## State shape

```js
{
  id: 'ann-1779360000000',
  lineId: 'tl-1779360001-3',     // null for free-form notes
  text: 'patience pays in years not minutes',
  by: '@willow.fern',
  tier: 'student',
  session: 'Unit 4 · Decoctions',
  savedAt: '9:14 AM',
  tags: ['patience', 'preparation'],
  title: null,                    // optional for free-form
  freeForm: false,
  history: [                      // recursive learning trail
    { text: 'first version', savedAt: '9:10 AM' },
    { text: 'second version', savedAt: '9:12 AM' }
  ]
}
```

## Backend prompt

See `PROMPTS.md §1` for the production migration prompt — schema, RLS, API client surface.
