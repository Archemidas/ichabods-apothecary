# Ichabod's Apothecary — Development Prompts Index

> A catalog of pages, panes, and subsystems that the current MVP simulates but does not yet implement against real infrastructure. Each entry is a self-contained prompt that can be handed to a developer (human or LLM) to build that piece against the spec.

Use this as the source of truth when extending the codebase. Each prompt is intentionally written like a brief, not a story: scope, inputs, outputs, integrations, edge cases.

---

## How to read this document

- **Status** — `mocked` (simulated in demo), `partial` (some real wiring), `not-started` (no code), `forthcoming` (spec only)
- **Spec ref** — JSON path inside `ichabods-imvp.json` that governs the feature
- **Prompt** — what to give an implementer
- **Definition of done** — observable behaviors that mean the work is complete
- **Edge cases** — known gotchas to design for

When you implement a prompt, change its **Status** to `built` and add a link to the merged PR.

---

## 1. Knowledge Store · Persistent Backend

**Status:** mocked
**Spec ref:** `domain.ontology.entities[Annotation]`, `features[knowledge-store]`
**Owner:** TBD

**Prompt.**
> Build a persistent backend for the Knowledge Store. Today every annotation lives in Alpine state and dies on page reload. Replace that with a real store: per-student private namespace, instructor read access to their cohort, admin global read. Schema must support the four current annotation shapes (transcript-attached, free-form, edited-with-history, tagged). Recursive learning means we don't overwrite — when a note is revised, the prior version becomes a history entry with its own timestamp.
>
> Pick the store: Supabase (Postgres + RLS) is recommended because row-level security cleanly enforces the rank-based access. Firestore works too. Anything serverless is fine — WebHash hosting is static-only.

**Definition of done.**
- Schema migration with tables `annotations`, `annotation_versions`, `annotation_tags`, `users`
- Row-level security: student can read/write own rows; instructor can read cohort rows; admin can read all
- API client at `lib/knowledge-store.ts` with `create`, `revise`, `delete`, `search`, `listMine`, `listCohort`, `listAll`
- Existing demo annotation UI calls the API instead of mutating Alpine state
- Annotations survive a page reload
- Cohort view in Admin → Knowledge Store reads live from the same store

**Edge cases.**
- A note can have zero or many tags. Tags are global per user — `@willow.fern`'s "patience" tag is separate from `@sage.bramble`'s.
- Version history must preserve the original `savedAt` timestamp of every revision.
- When a transcript line is edited by an instructor (forthcoming feature), associated annotations should NOT be silently broken.
- Soft-delete only — don't lose history on `delete`. Mark `deleted_at`.

---

## 2. Auto-Categorization Engine

**Status:** partial (keyword dictionary in demo)
**Spec ref:** `features[knowledge-store].controls`, `domain.logic.rules[annotations-save-private]`
**Owner:** TBD

**Prompt.**
> The demo categorizes notes by matching the note text against a small keyword dictionary (`preparation`, `identification`, `dosage`, `patience`, `safety`, `tradition`). Replace this with a proper auto-categorization pipeline.
>
> Two layers: (a) deterministic — keep the keyword dictionary as a fast first pass, expand it with herbal vocabulary; (b) semantic — embed each note and the lesson it came from, then cluster against a fixed taxonomy maintained by the masthead. Use OpenAI embeddings or local sentence-transformers depending on cost budget.
>
> Categories must be editable: the masthead can add/remove top-level tags. Students can add personal tags that don't affect the taxonomy.

**Definition of done.**
- Notes get categorized within 2 seconds of saving
- Taxonomy is editable in admin (CRUD)
- Embedding similarity > 0.78 to a category centroid pins that tag automatically
- Students can override or add personal tags
- A note can carry both auto-tags and manual tags; UI distinguishes them visually

**Edge cases.**
- Very short notes (under 6 words) often get no auto-tag — that's OK, fall back to deterministic dictionary only.
- Don't auto-categorize the original transcript quote, only the user's note text.
- Tag taxonomy migration: when masthead renames a tag, propagate the change retroactively.

---

## 3. Live Transcription · Real STT

**Status:** mocked (scripted line emission)
**Spec ref:** `features[transcript]`, `simulation.events[transcript-line]`
**Owner:** TBD

**Prompt.**
> Replace the scripted transcript emission with real speech-to-text. The class video is streamed from the studio rig — extract audio, send to STT, push transcript lines to all viewers in real time over a websocket. Lines should carry: text, speaker label (best-effort speaker diarization), timestamp aligned to video playhead.
>
> Use OpenAI Realtime API, Deepgram, or AssemblyAI. Deepgram has the best latency-to-cost ratio for live class scenarios.

**Definition of done.**
- Lines arrive within 2 seconds of being spoken
- Speaker labels appear (instructor vs student) with at least 80% accuracy
- Transcript persists per-session and is replayable from the archive
- Each line carries a stable `lineId` so annotations stay anchored across reconnects

**Edge cases.**
- Studio mic vs student mic — diarization needs separate channels if possible.
- Reconnect: when a viewer drops and rejoins, replay the rolling buffer so they catch up.
- Latin botanical names get transcribed badly — maintain a hot-words list (`Achillea`, `Artemisia`, etc.).

---

## 4. Live Video · Studio Stream

**Status:** mocked (canvas simulation)
**Spec ref:** `features[video-player].providers`
**Owner:** TBD

**Prompt.**
> The demo uses an animated canvas. Replace with a real video stream. Theta Edge Node is the existing planned provider but the Theta iframe is CSP-blocked in sandboxed environments — keep the canvas as a fallback. Wire up RTMP ingest from the studio's capture rig (OBS or hardware encoder) to Theta. Player on the client receives via Theta's HLS/DASH URL.
>
> Picture-in-Picture, casting, fullscreen, scrubbing (DVR window), and the IPTV channel switcher are already mocked in the player UI — those need to wire to the real stream.

**Definition of done.**
- Studio can go live with `<2s` glass-to-glass latency for primary stream
- Players support pause + DVR scrub back up to 2 hours
- Picture-in-Picture overlay can host a secondary YouTube/Vimeo embed without breaking the primary
- Fullscreen API works in non-sandboxed deployments; CSS fallback used otherwise

**Edge cases.**
- Encoder loses network mid-class — auto-reconnect, preserve archive recording.
- Multi-camera switching: keep the spec but defer real implementation.
- Cast to Chromecast/AirPlay — `RemotePlayback` API on Chrome only.

---

## 5. Shop · Real Inventory + Checkout

**Status:** mocked (60% Red Fawn / 40% Ichabod's catalog hardcoded)
**Spec ref:** `features[shop]`, `domain.ontology.entities[Product]`
**Owner:** TBD

**Prompt.**
> Build the shop against real inventory. Red Fawn products should sync from their Shopify or whatever they're on — scrape if no API. Ichabod's own products live in our store (Shopify, Stripe Products, or a custom Postgres table). Maintain the 60/40 visual ratio in the merchandising layer, not the database.
>
> Checkout: Stripe (cards) + Stripe Onramp (fiat-to-crypto) for parity with the VOS deployment story. Address validation via the Google Maps Places API. Tax + shipping rules per-region. Order confirmation email via Resend or similar.

**Definition of done.**
- Catalog auto-refreshes from Red Fawn nightly
- Cart persists across pageloads (localStorage minimum, account-tied better)
- Real Stripe checkout works in test mode end-to-end
- Order confirmation email arrives within 30s
- Admin can mark items as featured / on the shelf / temporarily withdrawn

**Edge cases.**
- Red Fawn might be out-of-stock on items we surface — show a "Currently Steeping" badge instead of letting them buy.
- Wholesale 1lb herbal blend ($64) needs a special "trade order" flow with VAT exemption.
- Postpartum kit ($49) — bundled product, needs separate fulfillment handling.
- Vendor splits: when a Red Fawn product sells, Stripe Connect should route the share to Red Fawn's account.

---

## 6. Course Enrollment & Cohort Management

**Status:** not-started (UI shells exist; no real enrollment)
**Spec ref:** `roles[student]`, `features[course]`
**Owner:** TBD

**Prompt.**
> Build the enrollment flow. Students apply to a cohort. Admins review applications and approve into a cohort number (currently "Cohort III" in the demo). Each cohort has start/end dates, a weekly schedule, and a max capacity. Once enrolled, a student gets:
> - Access to the live class portal during scheduled sessions
> - Their cohort's chat channel
> - The full archive of their cohort's sessions
> - Their personal knowledge store (already covered in prompt #1)
>
> Payment: $X per cohort (TBD, suggest $480 for a 12-month program billed monthly). Stripe Subscriptions. Defer scholarship/sliding-scale workflow as a separate prompt.

**Definition of done.**
- Application form lives at `/course/apply` with name, email, motivation, prior experience
- Admin sees applications in a queue, can approve into a named cohort
- Approved students get a magic link to set their password and complete onboarding
- Stripe subscription created on approval
- Calendar of upcoming sessions visible to enrolled students

**Edge cases.**
- Audit mode (free tier) must still work — auditors see read-only class but cannot annotate or use chat send.
- Mid-cohort joiners — pro-rate, give access to recorded prior sessions.
- Refund window — 14 days from cohort start, then non-refundable.

---

## 7. Instructor Dashboard · Beyond Demo

**Status:** mocked (analytics + roster shells exist)
**Spec ref:** `features[admin-portal]`
**Owner:** TBD

**Prompt.**
> The Admin portal currently shows a mocked cohort roster, mocked analytics, and a working live preview of the stream. Build it out:
> - Real reaction & engagement metrics per student
> - Cohort-wide knowledge-store analytics: most-annotated transcript line, most-tagged concept, average note length over time
> - "Who's quiet" report — students who haven't raised hand, posted in chat, or annotated in N sessions
> - Direct message a student
> - Mark attendance (auto-tracked, manually correctable)
> - Cohort grading rubrics for the certification path

**Definition of done.**
- Every metric is fed by real events, not mocks
- "Who's quiet" report runs over a configurable window (default: last 3 sessions)
- DM uses the same Matrix infrastructure as cohort chat, routed to a private room
- Certification rubric supports criteria, weights, and per-student scoring with comments

**Edge cases.**
- A student who's actively annotating but never talking in chat should NOT show as "quiet" — annotation activity is the highest-value engagement signal.
- DMs from instructor must be unmistakable — different badge color than peer chat.

---

## 8. Materia Medica · Community Field Guide

**Status:** forthcoming (faded card on home page)
**Spec ref:** `features` (new — add `materia-medica` kind)
**Owner:** TBD

**Prompt.**
> A community-edited field guide for herbs. Each entry has: common name, Latin binomial, drawing or photograph, preparation methods, traditional uses, safety notes, references, contributors. Edits go through review — masthead approves before publication, like Wikipedia with editorial gating. Each entry can link out to the products in our shop and to relevant lesson archives.
>
> Built on Sanity, Strapi, or a Notion-like custom CMS. Search must work across common + Latin names + tags.

**Definition of done.**
- 30+ entries seeded by first cohort graduation
- Contributors are credited per-entry
- Each entry has a stable URL suitable for citing in students' knowledge stores
- Cross-links between Materia Medica → Shop → Archive work both ways

**Edge cases.**
- Conflicting traditions — entry must accommodate "Western herbalist says X / TCM says Y" without picking sides.
- Safety claims need citation requirements (medical disclaimer modal at first read).
- Photographs need licensing — track contributor + license per image.

---

## 9. Magic-Link Email Auth

**Status:** mocked (any email "works" in demo)
**Spec ref:** `auth.primary`
**Owner:** TBD

**Prompt.**
> Replace the mocked sign-in with real email magic-link auth. Resend or Postmark for email delivery. Token TTL 15 minutes, one-time use, revoked on sign-out. Magic link should deep-link to where the user was trying to go.

**Definition of done.**
- Email arrives within 10s
- Link works once, expires correctly
- Deep-linking: clicking a link to `/portal` puts the signed-in user in `/portal`
- Suspicious activity (multiple sign-ins from new IPs in short window) flagged to user

**Edge cases.**
- User clicks link in a different browser than they requested it from — fine, common, allow it.
- User signs in on phone but session lives on a desktop — same account, two devices, both stay active.

---

## 10. Hosting & Deployment

**Status:** undecided (Wix considered in the v2 doc; WebHash already used for VOS)
**Spec ref:** outside the spec — operational
**Owner:** TBD

**Prompt.**
> Decide the hosting strategy. Constraints:
> - Most of this site is static-feasible (home, shop browse, archive list, signed-out states).
> - Knowledge Store, live class, chat, payment all need backends.
>
> Options:
> 1. **Wix** — easy admin, mediocre developer ergonomics. The v2 doc asks specifically about feasibility — answer: shop and home work fine on Wix, but knowledge store + live transcription + RLS need a separate API service (Cloudflare Workers or similar). Wix Velo gives some scripting but not enough.
> 2. **WebHash Pro + serverless** — IPFS static hosting (as VOS used), with Cloudflare Workers for the auth + knowledge-store API. Cleanest separation. Decentralized story for Web3 marketing.
> 3. **Vercel + Supabase** — most boring, fastest to ship. Recommended unless decentralization is a hard requirement.

**Recommendation.** Option 3 unless there's a strategic reason for IPFS. Option 2 if so.

---

## 11. Wix Compatibility Audit

**Status:** open question from v2 doc
**Spec ref:** outside the spec — operational
**Owner:** TBD

**Prompt.**
> The v2 doc asks: "What additional resources or capabilities would be necessary to implement this vision using Wix as the hosting platform?"
>
> Concrete answer:
> 1. **Wix Velo** for any custom logic (annotation save, knowledge search, free notes).
> 2. **Wix Stores** for the shop side — handles products, cart, checkout, fulfillment.
> 3. **External API** required for: live transcription, real video stream ingest, embedding-based auto-categorization. These cannot run inside Wix; need Cloudflare Workers, Vercel functions, or a small Node service.
> 4. **Custom database collections** in Wix CMS for: cohorts, applications, knowledge store entries, version history.
> 5. **Wix Forms** for cohort applications.
> 6. **Wix Members** for student accounts.
>
> Verdict: feasible, but the Knowledge Store v2 with embedding-based categorization is the binding constraint. If that's a must-have, the auxiliary API service is required regardless of host. If categorization stays keyword-only, the whole stack can sit inside Wix + Velo.

---

## 12. Pages not yet visible in the demo

These exist in the spec or in user requests but have no UI implementation yet:

- `/community` — circle directory of cohort members (gated to enrolled)
- `/calendar` — schedule of upcoming sessions, ical export
- `/about` — Ichabod's history, lineage, contributors
- `/contact` — apothecary visit hours, mail-in remedy requests
- `/legal` — disclaimers, herbal-medicine advisories, refund policy
- `/account` — student preferences, subscription management, knowledge-store export

Each of these is a small lift. Suggested prompt for the developer:

> Build page `/<route>` using the existing Cinzel + Cormorant + Pinyon Script type system. Apply the framed-label treatment (corner SVGs + quadruple inset box-shadow) for major content blocks. Use the same color tokens (`--accent`, `--gilded`, `--ember`, `--rose`, `--forest`). Match the voice from the spec's `aesthetic.voice` — slow, instructive, archaic.

---

## Maintenance

When adding a new prompt:

1. Increment the section number
2. Tag with **Status**, **Spec ref**, **Owner**
3. Include a clear **Definition of done** — observable, not aspirational
4. Document at least 2 edge cases
5. Update the spec JSON if a new entity, tier, or feature is introduced

Last updated: 2026-05-21
