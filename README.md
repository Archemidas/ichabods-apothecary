# Ichabod's Apothecary

> Plant medicine and the patient art of remembering.

A certified medicinal herbalist program, a small apothecary shop, and a community of students and growers. Built as an Interactive MVP — every system is simulated against the IMVP v1.0 schema, ready to be wired to real backends.

```
ichabods-imvp.json  ─▶  Demo Controls Subsystem  ─▶  ichabods-demo.html
                              │
                              ▼
                    [ Knowledge Store v2 ]
                    [ Live Class Portal ]
                    [ Shop · 60/40 vendor mix ]
                    [ Cohort + Archive ]
                    [ Instructor Knowledge View ]
```

## What's in this repo

| Path | Description |
|---|---|
| [`ichabods-demo.html`](./ichabods-demo.html) | Single-file interactive demo. Alpine.js v3, no build step. |
| [`ichabods-imvp.json`](./ichabods-imvp.json) | IMVP v1.0 spec governing the demo. Round-trippable. |
| [`PROMPTS.md`](./PROMPTS.md) | Development prompts indexing every page, pane, and subsystem that's mocked or incomplete — what to build next, scoped per implementer. |
| [`docs/`](./docs/) | Reference materials and the v2 Knowledge Store design notes. |

## The site features

**Active and demoable today:**
- Home page (vintage apothecary signboard with corner ornaments, jar row, quote rail)
- Shop (27 products, 60% Red Fawn / 40% Ichabod's, filterable, vintage-label cards)
- Course portal (live video, scrub bar with DVR, picture-in-picture, casting, fullscreen)
- **Live class with on-screen transcript** — every line clickable to annotate
- **Cursive annotations** in Pinyon Script attached to specific transcript lines
- **Knowledge Store v2** — search, auto-tagging, version history, free-form notes
- Lesson archive with canvas thumbnails
- Sign In / Admin Login
- Admin portal with rank-based knowledge store access (cohort view + per-student filter)
- Six-subsystem Demo Controls FAB

**Forthcoming (see [PROMPTS.md](./PROMPTS.md)):**
- Persistent backend for the Knowledge Store
- Real magic-link email auth
- Live STT for the transcript
- Real video stream from the studio
- Stripe checkout for the shop
- Materia Medica community field guide
- Calendar, community directory, account settings

## The Demo Controls Subsystem

Bottom-right `⚡` floating panel. Same six subsystems as the canonical IMVP Forge pattern:

1. **State Navigator** — page + tier selector
2. **Identity Switcher** — Admin / Instructor one-click logins
3. **Event Injector** — `+ Reaction`, `+ Hand`, `+ Chat`, `+ Transcript`, `+ Cohort Note`
4. **Activity Simulator** — six background loops (transcript emission, reactions, hands, viewer flux, chat, cohort notes)
5. **Reset & Snapshot** — full reset; two scenarios (`First Class`, `Instructor View`)
6. **Metadata Surface** — transcript source, video provider, shop vendor mix, auth strategies

## Knowledge Store v2 — what the doc asked for

The v2 update from the project Google Doc specified:

> Rank-based access (admin and student levels) for managing uploaded content. Automated parsing, ingestion, and archiving. Self-categorizing organization across multiple parameters. Searchable access. Recursive learning capabilities that track knowledge evolution over time, including documented shifts in interpretation.

Implemented in this demo:

- **Rank-based access** — student sees own; instructor sees their cohort; admin sees all (mocked; see [PROMPTS.md §1](./PROMPTS.md#1-knowledge-store--persistent-backend) for the backend)
- **Auto-categorizing** — six-tag keyword dictionary (preparation, identification, dosage, patience, safety, tradition) runs over each saved note. See [PROMPTS.md §2](./PROMPTS.md#2-auto-categorization-engine) for the embedding upgrade.
- **Searchable** — toolbar search box matches across note text, transcript quote, title, tags, session
- **Recursive learning / evolution** — every revision pushes the prior version into a `history` array with its original timestamp; UI surfaces an "Evolved · N revs" badge and a collapsible evolution trail
- **Multi-parameter filter** — tag chips + free search compose; clear button resets both
- **Free-form notes** — modal composer for notes not attached to any transcript line (with title, body in cursive Pinyon Script, comma-separated tags)

## The aesthetic

Victorian Gilded Age / Art Nouveau apothecary label. Direct references:
- Vintage apothecary labels with quadruple inset gilded frame
- Pinyon Script for ornamental subtitles ("crafted by moonlight & ancient wisdom")
- Cinzel inscriptional for titles
- Cormorant Garamond for body
- Caveat for handwritten annotations
- Ember red `#8F3818` · Gilded brown `#A37A28` · Dusty rose `#9B5C5A` · Sage forest `#3A4E2E`
- Hand-drawn corner SVG ornaments on every major frame
- Botanical jars row (Yarrow, Mugwort, Valerian) with Latin binomials

## Running it

It's one HTML file. Open it in a browser. There is no build step.

```bash
# Local preview
python3 -m http.server 8000
# then open http://localhost:8000/ichabods-demo.html
```

## License

MIT. See [LICENSE](./LICENSE).

---

Built as a derivative of the [IMVP Forge](https://github.com/Archemidas/imvp-forge) framework, using its canonical Demo Controls subsystem.
