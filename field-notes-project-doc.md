---
title: "Field Notes (was Field Intel Webapp)"
cockpit: true
domain: "Field"
description: "Mobile-first PWA for reps to capture field intelligence (surgeon visits, competitive intel, case feedback) tied to CRM entities. V4 reframes capture as a conversational, voice-first interaction with Claude that produces structured field_notes rows. Notes flow into Supabase and feed the Field Notes Processing pipeline."
next_action: "Decide synthetic-data smoke-test approach (Option A/B/C from session 6 — recommend B: 10 notes via Q live + 90 via SQL) and run it. Add `is_test BOOLEAN` column to field_notes first."
rollout_target: "V4.0.3 LIVE in production. Hold rep rollout until synthetic-data smoke test passes and Field Notes Processing Phase 1 is also live."
build_machine: "Either — repo on GitHub, Vercel auto-deploys on push. Built session 6 on the laptop."
---

# Field Notes — Living Project Document
Last updated: 2026-05-16 | Session #6 (V4.0 BUILT AND SHIPPED — V4.0.3 live in production. RPCs + Edge Function + chat UI + fuzzy match + Whisper priming + Done button. Next: synthetic-data smoke test.)

---

## 1. Project Identity
- **What it is:** A mobile-first PWA for Agility Ortho reps to capture field intelligence notes (surgeon visits, competitive intel, case feedback, etc.) that flow into the CRM
- **Goal:** Reps tap an icon on their phone, sign in once with an email code, and quickly enter notes tied to a surgeon, hospital, competitor, or manufacturer from the real CRM database. Notes are stored in Supabase and available for downstream processing.
- **Owner:** John Walsh
- **Key people:** John (owner), Nate/Nick/Noah/Pat (TMs), S3 reps — all 11 team members are in the system
- **Key systems/tools:** Supabase (`oso-tray-tracker` project — the main Agility ops DB), Vercel (hosting), GitHub (`jwalshAO/field-intel-webapp`)
- **Quick start:** To continue this project, say: `"continue Field Intel webapp"`

---

## 2. Current Status
**V4.0.3 is LIVE in production at https://field-intel-webapp.vercel.app.** Built end-to-end on the laptop session 6. Not yet rolled out to reps — holding until synthetic-data smoke test passes and Field Notes Processing Phase 1 is live (so reps see "make a note → fact appears" loop from day one).

**Default rep experience:** Chat with **Q** (efficient sidekick persona, Bond Q-Branch reference). Tap mic, talk, auto-stops at 1.5s of silence, Whisper transcribes (with CRM proper-noun priming), Claude Haiku 4.5 handles the conversation with tool use against the live CRM. Tool calls surface as visible system bubbles. "Done ✓" green pill below the composer short-circuits to confirm-and-edit at any time. V3 form survives as one-tap "Use form ▸" escape hatch.

**Original V3 status (now superseded but preserved for reference):**

**Core flow:** Rep opens app → splash screen (swipe up to dismiss, motivational blurb) → sign in once via email OTP → land on New Note form → pick a Subject (surgeon/hospital/manufacturer/competitor or freeform), select tags, write notes, optionally snap a photo → submit → view in Notes Feed.

**Key features:**
- **Auth:** Supabase email OTP, 8-digit code, persistent session (sign in once)
- **Multi-entity Subject:** Autocomplete searches surgeons (649), hospitals (168), manufacturers (8), competitors (14) with color-coded type badges. Optional — can leave blank for general intel. Freeform text also accepted.
- **Tags:** Surgeon Interaction, Case Feedback, Competitive Intel, Product Interest, Issue / Complaint, Observation
- **Notes:** Freeform textarea with guiding placeholder
- **Next Action:** Optional 3-line field with actionable placeholder text
- **Photo:** Camera icon button beside date field — tap for camera/gallery. Preview appears inline with ✕ to remove.
- **Soft-delete:** Reps see ✕ on own notes → removed from their feed. Owner sees all deleted notes faded with "Deleted by [name]" red tag.
- **Role-based visibility:** S3s see own notes, TMs see territory, Owner sees all
- **Splash screen:** Swipe-up-to-dismiss with subtle bouncing chevron. Blurb: "Every note you capture here makes the whole team sharper. The more we know, the better we sell."
- **Competitors table:** 14 seeded (Acumed, DePuy Synthes, Stryker, Arthrex, Axogen, Zimmer Biomet, TriMed, Avanti, S&N, Integra, Medartis, Biederman, AM Surgical, Globus Medical)
- **UI:** Clean minimal form — date + camera on one row, no unnecessary labels, tags at 11px

---

## 3. What Exists (Artifacts & Files)
- `index.html` — Main app (V4.0.3: chat with Q + V3 form escape hatch + confirm-and-edit modal + Done button)
- `manifest.json` — PWA manifest (short_name: "Field Intel")
- `sw.js` — Service worker (CACHE_NAME `field-intel-v4-3`)

### Supabase Edge Function: `field-notes-chat` (v3, ACTIVE)
- Handles two inputs: `audio_base64` (→ Whisper → text) or `text` directly
- Runs Claude Haiku 4.5 tool-use loop (up to 6 iterations)
- Whisper primed with proper-noun list: 9 manufacturers, 14 competitors, top ~25 hospitals, common procedures
- Anthropic prompt caching on system prompt + tools (5min TTL)
- Auth: `verify_jwt: true` — only signed-in reps can call
- Secrets: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY` (stored in Supabase via CLI session 6)

### Supabase RPC functions (SECURITY DEFINER, granted to authenticated + anon)
- `fn_lookup_entity(query, kinds[])` — fuzzy entity search across surgeons/locations/manufacturers/competitors using `pg_trgm` trigram similarity + `dmetaphone` phonetic match. Catches misspellings like "Lankanon" → Lankenau, "Tiber" → TYBR, "Petrocelli" → Petrucelli.
- `fn_surgeons_at_location(location_id)` — surgeons known to operate at a location (via `surgeon_locations` junction).
- `fn_recent_notes_about(entity_kind, entity_id, days)` — recent field notes about an entity to avoid redundant questions.

### Extensions enabled session 6
- `pg_trgm`, `fuzzystrmatch` (for the fuzzy lookup above)

### RLS fixes session 6
- Added `auth_manufacturers_read` and `anon_manufacturers_read` SELECT policies on `manufacturers` (it had RLS enabled but only a service_role policy — broke the Notes Feed embed query the moment a V4 note set `manufacturer_id`).

### Legacy V3 artifacts (still in repo)
- `vercel.json` — Vercel deployment config
- `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — "Field Notes" text-only kraft notebook icon
- `Home Screen Icon.png` — Source icon v1 (triquetra, with spiral)
- `Home Screen Icon 2.png` — Source icon v2 (triquetra, no spiral, rounded corners)
- `Home Screen Iconn 3.png` — Source icon v3 (triquetra, no spiral, square edges) — typo in filename
- `Home Screen Icon 4.png` — Source icon v4 (text only, "Field Notes" large) — CURRENT
- `Gemini_Generated_Image_o16shio16shio16s.png` — Original source icon (gitignored, 9MB)
- `field-intel-project-doc.md` — This file

### Supabase: `oso-tray-tracker` (pchhtltxdcmvdcwnwaeg) — THE MAIN DB
- `surgeons` — 649 rows (first_name, last_name, credential, NPI, specialty, status, primary_hospital, funnel_stage)
- `locations` — 168 rows (hospitals/facilities with territory, rep assignments)
- `reps` — 11 rows (id, name, email, role, rep_type, territory_id, active)
- `manufacturers` — 8 rows
- `field_notes` — (rep_id FK→reps, subject_type, surgeon_id FK→surgeons, location_id FK→locations, manufacturer_id FK→manufacturers, competitor_id FK→competitors, subject_text, note_body, tags[], note_date, next_action, photo_url, created_at, deleted_at, deleted_by, deleted_by_rep_id FK→reps)
- `competitors` — 14 rows (name, category, active)
- `auth.users` — 4 registered (John, Nick, Nate, Derek); others auto-create on first OTP
- Storage bucket: `field-intel-photos` (public read, authenticated upload)

### Supabase: `jwalsh@agilityortho.com's Project` (iwmilzfdrcuixpitrpkh) — RETIRED
- No longer referenced by any app. Can be deleted or left (no cost).

---

## 4. Settled Decisions
- **Supabase Auth email OTP** — Rep enters email → gets 8-digit code → enters code → session persists indefinitely. Same pattern as Tray Tracker. (Decided session 2)
- **Multi-entity Subject autocomplete** — Searches surgeons, hospitals, manufacturers, competitors with color-coded badges. Freeform text also accepted. Subject not labeled "optional" (to encourage use) but technically not required. (Decided session 3, replaces surgeon-only autocomplete from sessions 1-2)
- **Photo: camera icon button** — Camera SVG button beside date field (half-width each). Tap for camera/gallery prompt. Green badge when photo attached. Preview inline with ✕ remove. Stored in Supabase Storage. (Redesigned session 3 from full-width photo section)
- **Next Action field** — Optional 3-line freeform textarea. Placeholder: Setup next "touch", email link/video, text dates to next lab, get dates for a demo, setup lunch, etc... Displays as orange badge in feed. (Decided session 2, updated session 3)
- **"Surgeon Interaction" tag** — Renamed from "Surgeon Visit" for broader coverage. (Decided session 2)
- **No separate contacts tab** — Surgeons live in the CRM DB. Field Intel doesn't reinvent that. (Decided session 1)
- **Freeform notes body** — The note text is freeform. Claude parses it downstream. (Decided session 1)
- **Role-based visibility** — S3s see own notes, TMs see territory's notes (via territory_id), Owner sees all. (Decided session 1, implemented session 2)
- **Soft-delete** — Reps/TMs can delete their own notes (✕ button). Owner can delete any note. Deleted notes vanish from non-owner feeds but remain visible to Owner with faded styling + "Deleted by [name]" red tag. (Decided session 3)
- **Multi-entity Subject** — "Surgeon" field renamed to "Subject." Autocomplete searches surgeons, hospitals, manufacturers, and competitors with color-coded type badges. Notes can also have no subject at all (first tag used as title, fallback: "General Note"). (Decided session 3)
- **Competitors table** — 14 seeded competitors in a new `competitors` table. (Decided session 3)
- **Splash screen: swipe-up to dismiss** — Blue gradient, app icon, title, motivational blurb, subtle bouncing chevron. Stays until user swipes up or taps. Also works as tap-to-dismiss. Waits for auth check before becoming dismissable. (Decided session 3)
- **Minimal form layout** — Date and camera share one row (half-width each). No "Date" label. Subject label says "Subject" (not "optional"). Tags label says "Tags (optional)". No "+ Add new surgeon" in dropdown — just freeform "Use as-is" option. (Decided session 3)
- **"Field Intel" under icon on home screen** — manifest short_name handles this. (Decided session 1)
- **Text-only notebook icon** — "Field Notes" in bold on kraft paper with leather spine. No triquetra/symbol. Edge-to-edge fill, iOS does its own rounding. Generated by Gemini. (Decided session 2)
- **All data consolidated in oso-tray-tracker** — One Supabase project for everything. (Decided session 1)
- **Deploy workflow** — GitHub repo → Vercel auto-deploy on push. `git push origin main` only. (Decided session 1)

---

## 5. V4 Plan — Conversational Capture (rollout target)

V4 is the version reps will see at rollout. V3 was never shown to the field; we skip the V3 deploy as a standalone event and go straight to V4, which includes V3's form as an alternate "Type" mode.

### 5.1 Concept
Rep opens the app → on the New Note screen taps a **mic / chat button** (default-prominent) → starts talking (or typing) → Claude responds in chat, asking clarifying follow-ups to surface more facts → multi-turn dialogue collapses into a single structured `field_notes` row at the end → rep gets a brief confirm-and-edit screen → submit.

Example flow:
> **Rep:** Dr. Jones is at Methodist today.
> **Claude:** Got it — Aaron Jones at Methodist Lankenau, or Wendy Jones at Methodist KOP?
> **Rep:** Aaron, Lankenau.
> **Claude:** Recording that. What types of cases is he doing there today?
> **Rep:** Distal radius. He's using Acumed.
> **Claude:** Thanks. Any other surgeons operating there today, or other intel from the visit?
> **Rep:** No, just him. That's it.
> **Claude:** Confirming: visit to Aaron Jones at Methodist Lankenau, distal radius cases, using Acumed. Tags: Surgeon Interaction, Competitive Intel, Surgeon Using. Submit?

### 5.2 Why V4 is the rollout target
- **Reps haven't seen V3** — no UX migration cost to skip ahead
- **Lower cognitive load on the rep** — no form to scan, no 14-tag taxonomy to navigate; Claude does the structuring
- **Drives completeness** — Claude prompts for the next fact when the rep would otherwise stop typing ("any other surgeons there today?")
- **Voice fits the moment** — most field-note moments are mobile (between accounts, walking to car); a mic button beats a keyboard
- **Auto tag selection** — the 14-tag taxonomy stops being a UI problem and becomes a Claude inference problem
- **Aligns with the Bunnell methodology axis** — the tax is on capture today; conversational capture removes friction for higher-value tags (Give to Get, Opportunity, Surgeon Preference)

### 5.3 Architectural decisions
- **Claude API required** — conversational mode is real-time on the rep's phone. Anthropic API called via a thin Supabase Edge Function proxy that keeps the key off the device.
- **Voice runtime: Whisper API (locked)** — better medical-vocabulary accuracy (surgeon names, "distal radius", "Acumed", "Methodist Lankenau"), predictable behavior across all devices/browsers/PWA standalone mode, removes iOS Safari Web Speech API risk entirely. Cost ~$0.006/min, negligible. OpenAI API key lives in Supabase secrets next to the Anthropic key. (Decided session 5)
- **Assistant identity: Q (locked session 5)** — The conversational assistant inside Field Notes is named **Q**, a nod to James Bond's Q Branch (the tech sidekick supplying field operatives). Single letter = ultimate "efficient sidekick" branding; the rep is the field agent, Q is the helper back at HQ. Universal recognition, zero training cost. Q is a Field Notes-specific persona — distinct from "Neo," the broader Codex assistant.
- **Q's tone: efficient sidekick (Option A, locked session 5)** — Crisp, brief, no fluff. "Got it. Anyone else there today?" not "Interesting — Aaron's been busy!" Q asks one question at a time, never lectures, never moralizes, gets out of the way. Matches Jarvis/Q archetype.
- **Chat is the interface; voice and text are both inputs to it (Option B, locked session 5)** — Rep opens New Note → sees a chat window with a mic button AND a text input bar at the bottom, both always available. Tap mic to talk, or type, or mix the two mid-note. Claude asks the same follow-ups regardless of how the rep is feeding it information. The V3 form survives as a one-tap escape hatch ("Use form ▸") for the rep who doesn't want to converse — not as a primary mode. This means voice failure ≠ losing the conversational value; rep just keeps typing in the same chat. The previously planned "Talk / Type toggle" framing is dropped.
- **CRM-aware tool use during the conversation:**
  - `lookup_entity(query, kinds=[surgeon|location|manufacturer|competitor])` — autocomplete-style entity match with disambiguation
  - `surgeons_at_location_today(location_id, date)` — "anyone else there today?" flows
  - `recent_notes_about(entity_id, days=30)` — Claude avoids asking what's already known
  - `propose_tags(conversation)` — final-step tag suggestion using the 14-tag taxonomy
- **Entity disambiguation in-flow** — when "Jones" matches multiple surgeons, Claude asks the rep to pick, not a dropdown
- **End-of-conversation confirm screen** — parsed subject, tags, prose body, next action (if any) — rep can edit any field, then Submit. Submit creates the `field_notes` row, which the Field Notes Processing pipeline then extracts from.
- **Prompt caching** — the static portion (system prompt, tag taxonomy, fact schema, tool definitions) cached for 5 min; only the conversation deltas vary per call. Materially cuts cost given conversation length.
- **Anthropic API key + workspace** — create a dedicated "Field Notes" workspace at console.anthropic.com (shared with Field Notes Processing for billing visibility). Key lives in Supabase secrets, never in the client bundle.

### 5.4 Build sequencing (within V4)
- **V4.0 — Conversational MVP:** mic button entry point on New Note; voice-in via Web Speech API; chat UI; `lookup_entity` and `surgeons_at_location_today` tools wired; end-of-conversation confirm screen; creates a `field_notes` row identical in shape to V3 output.
- **V4.1 — Tag auto-suggestion:** Claude proposes tags from the 14-tag taxonomy at the confirm step; rep can adjust via the tag bottom-sheet (which is now the manual-override surface, not the primary input).
- **V4.2 — Multi-entity output:** one conversation can produce multiple `field_notes` rows when the rep covered multiple surgeons in a single capture session.
- **V4.3 — Photo mid-conversation:** "send me the photo when you can" → camera button appears inline; photo attached to the resulting note.
- **V4.4 — Offline queueing:** if signal drops mid-conversation, local state preserves the transcript; sync resumes when online.

### 5.5 Open design questions
- ~~**Voice as the only first-class input, or co-equal with text?**~~ **RESOLVED session 5:** co-equal. Mic and text input both always visible in the chat. V3 form is a separate escape hatch, not the text mode.
- **Auto-submit vs. confirm screen?** Leaning confirm screen for v0; auto-submit might come later once we trust the parse.
- **When is the conversation "done"?** Heuristic: rep says "that's it" / "done" / "submit" / hits a Submit button manually. Don't rely on long pauses (rep might be driving).
- **Do TMs or John ever review the conversation transcript, or only the final note?** Default: final note only. Transcript stored for debugging, not surfaced.
- **Can the rep edit a submitted note's parsed structure later, or only the prose?** Leaning prose-only for v0 — edits re-trigger the Processing extractor anyway.

### 5.6 Cost model (per-note + monthly)
Per-note all-in (Whisper + Claude with prompt caching):
- **Whisper:** ~60–120 sec of audio across all turns → ~$0.012/note
- **Claude (Sonnet 4.6, with prompt caching):** static system prompt + tools + tag taxonomy cached; only conversation deltas pay full price. ~4-turn conversation with 1-2 tool calls → ~$0.03–0.05/note
- **All-in: ~$0.04–0.06 per note** (call it ~$0.05/note as planning number)

Monthly projections (assuming ~22 working days, 10 active reps + John):
- **Light usage** (5 notes/day per rep, ~1,100 notes/mo): **~$55/mo**
- **Moderate usage** (10 notes/day per rep, ~2,200 notes/mo): **~$110/mo**
- **Heavy usage** (15 notes/day per rep, ~3,300 notes/mo): **~$165/mo**

Sunk costs not incremental to this project: Supabase Pro (~$25/mo, already paying), Vercel (~$0–20/mo, already paying).

Risks that could blow the budget: reps having long rambling conversations (5+ min of audio per note), heavy tool use per conversation (>3 lookups), retries on bad transcriptions. Mitigation: log input/output tokens per call from day one so we can see actual cost after the first 100 real notes and tune.

Field Notes Processing (downstream Claude calls to extract structured facts from `field_notes` rows) is a **separate cost line** — not included in the numbers above.

### 5.7 Relationship to Field Notes Processing
V4 and Field Notes Processing share **one interface: the `field_notes` table.** V4 produces rich, structured prose rows; Processing extracts facts from them. The two projects build in parallel:
- Processing Phase 1 (schema) and V4.0 (conversational MVP) can run concurrently on the Mini
- Rollout to reps waits until at least Processing Phase 1 + V4.0 are in place, so reps see "I made a note → it appears as a fact on the surgeon profile" almost immediately
- The "tag bottom-sheet" side-track from Processing's plan moves into V4.1 as the manual-override surface for auto-suggested tags

---

## 6. Active Work Items

**Done session 6:**
- [x] V4.0 build (laptop, not Mini — machine doesn't matter, Vercel auto-deploys on push)
- [x] Anthropic + OpenAI keys provisioned and stored in Supabase secrets
- [x] Voice runtime decided: Whisper (session 5) + Whisper proper-noun prompt (session 6)
- [x] Prompt caching wired on system prompt + tools
- [x] Three RPCs deployed + smoke-tested
- [x] Edge Function `field-notes-chat` deployed (v1 → v3)
- [x] Frontend chat UI with mic/text composer, tool-call surfacing, confirm-and-edit modal, Done button
- [x] Friction fixes: fuzzy entity matching (pg_trgm + dmetaphone), Whisper priming, no-reject prompt rules
- [x] Notes Feed RLS bug fixed (manufacturers table)
- [x] FX Shoulder added to manufacturers (id 9)

**Active / pending (in priority order):**
- [ ] **Synthetic data smoke test** — generate ~100 test field_notes (real CRM IDs, `is_test=true` flag) → validate feed at volume, role-based visibility, eventual Processing extractor input. **Pending decision: Option A (all SQL) / B (10 via Q + 90 SQL) / C (all via Q live). Pick B unless told otherwise.** First step: add `is_test BOOLEAN DEFAULT false` column to `field_notes` + hide test rows from default feed query.
- [ ] Test on phone with John actually using it for a day or two: tone calibration, mic latency feel, transcription accuracy on real-world audio
- [ ] Settle remaining open §5.5 questions (most resolved session 5; "add new surgeon inline" and "photo in chat mode" still open)
- [ ] 14-tag taxonomy decision (currently shipping with 6 V3 tags)
- [ ] Field Notes Processing Phase 1 (separate project — runs on Mac Mini, extracts structured facts from `field_notes` rows). MUST be live before V4 rollout to reps.
- [ ] Rollout to reps once Processing Phase 1 + V4 are both stable

**Deploy command:**
```
cd "/Users/johnwalsh/Codex/Projects/Field Notes" && git add . && git commit -m "V4: ..." && git push origin main
```

**Useful Supabase MCP queries for next session:**
- Edge Function logs: service `edge-function`, filter for `field-notes-chat`
- Postgres logs: service `postgres`, look for RPC call patterns
- Field notes table inspection: `SELECT id, rep_id, subject_text, subject_type, tags, created_at, is_test FROM field_notes ORDER BY created_at DESC LIMIT 20;`

---

## 7. Open Questions & Blockers
- **V4 design questions** — see §5.5 (voice vs. text primacy, auto-submit vs. confirm, "done" heuristic, transcript retention, edit scope)
- **"Add new surgeon" flow inside V4** — Today's flow is a separate modal (first/last/hospital). In V4 conversational mode, Claude should propose "I don't have an Aaron Jones at Lankenau in the system — add him?" inline. Open: does "add" happen in the conversation or kick to the existing modal?
- **Photo capture in voice flow** — V4.3, but worth deciding the UX shape (mid-conversation camera prompt vs. attach-at-confirm)
- **oso-tray-tracker naming** — John wants to rename this project eventually. No action needed now.

---

## 8. Constraints & Preferences
- Light/white backgrounds for all HTML — never dark themes
- Tags should be small/unobtrusive (11px)
- Terminal commands in one shot, not multiple
- All files saved to Codex (`/Users/johnwalsh/Codex`)
- Git push workflow only — no `gh` or `vercel` CLI
- `oso-tray-tracker` project ID: `pchhtltxdcmvdcwnwaeg`
- GitHub repo: `jwalshAO/field-intel-webapp`
- Vercel URL: https://field-intel-webapp.vercel.app

---

## 9. Background Context
- Agility Ortho is an upper extremity orthopedic distribution company (Eastern PA, South NJ, Northern DE)
- 11 team members: John (owner), 4 TMs (Nate, Nick, Noah, Pat), 6 S3 reps
- The `oso-tray-tracker` Supabase project is the single operational database — surgeons, locations, reps, trays, sales, revenue
- The `reps` table has all 11 people with emails, roles, territory_ids
- `surgeons` table has rich data: names, NPIs, specialties, status, primary_hospital, funnel_stage
- 4 of 11 reps have Supabase Auth accounts; the rest will auto-create on first OTP login
- The Field Intel app's job is pure INPUT — make it dead simple for reps to capture intel. The CRM handles the rest.

---

## 10. History Log
- 2026-04-02 — Session 1: Designed and built V1 of Field Intel webapp. Created Supabase schema (wrong project initially), built PWA frontend, deployed to Vercel via GitHub. Discovered surgeon DB exists in oso-tray-tracker. Decided to consolidate there and rebuild Subject field as surgeon autocomplete. Custom notebook icon finalized. Multiple UI tweaks (tags, placeholder). Changes staged locally but NOT yet pushed to GitHub.
- 2026-04-02 — Session 2: Major V2 rewrite. Switched to correct Supabase project (oso-tray-tracker). Created `field_notes` table with FK links to reps and surgeons. Added RLS policies. Implemented Supabase Auth email OTP (persistent sessions). Built surgeon autocomplete from 649-row surgeons table. Added photo capture/upload (camera + gallery) with Supabase Storage bucket. Added "Next Action" freeform field. Renamed "Surgeon Visit" tag to "Surgeon Interaction". Removed Contacts tab. Iterated on icon 4 times with Gemini — landed on text-only "Field Notes" on kraft notebook. All changes ready to push but NOT yet deployed.
- 2026-04-02 — Session 3: Big feature + UX session. (1) Multi-entity Subject: renamed "Surgeon" to "Subject", autocomplete now searches surgeons/hospitals/manufacturers/competitors with color-coded badges. Created `competitors` table (14 seeded). Added `subject_type`, `location_id`, `manufacturer_id`, `competitor_id` columns. (2) Splash screen: swipe-up-to-dismiss with motivational blurb and subtle bouncing chevron. (3) Soft-delete: reps can ✕ their notes, Owner sees all with "Deleted by [name]" red tag. Added `deleted_at`, `deleted_by`, `deleted_by_rep_id` columns. (4) Form UX cleanup: date + camera button share one row (no labels), Subject label dropped "(optional)", removed "+ Add new surgeon" from dropdown (just freeform), photo section replaced with inline camera icon, Next Action expanded to 3 rows with new placeholder, Notes placeholder updated. (5) Added "Observation" tag. (6) Bumped SW cache to v2. All DB migrations applied. Code ready to push but NOT yet deployed.
- 2026-05-16 — Session 6: **V4.0 BUILT AND SHIPPED.** End-to-end build day on the laptop — V4.0.3 is now live in production at https://field-intel-webapp.vercel.app.

  **Q is named and live.** Assistant persona is **Q** (Bond Q-Branch reference — efficient sidekick, brief, supplies the field operative). Tone: "Got it. Anything else?" Crisp, no fluff, no exclamation points, no emojis. Single-letter name was the unlock — universal recognition, zero training cost, matches the rep-as-field-agent metaphor.

  **Provisioning:** Created "Field Notes" workspace in Anthropic console + API key. Created Field Notes project in OpenAI + Whisper API key. Both keys stored as Supabase secrets via Supabase CLI (installed via brew session 6) — `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`. Spend limits set: Anthropic $200/mo workspace cap, OpenAI $50/mo project cap.

  **Built the backend:**
  - 3 SECURITY DEFINER RPCs in `add_field_notes_v4_rpcs` migration: `fn_lookup_entity`, `fn_surgeons_at_location`, `fn_recent_notes_about`. Smoke-tested with real CRM data.
  - Edge Function `field-notes-chat` v1 deployed: handles audio (Whisper) or text input, runs Claude tool-use loop, returns parsed note when Q calls `submit_note`.

  **Built the frontend (in `index.html`):**
  - Chat thread + mic-on-right composer + Done ✓ green pill below
  - Mic ↔ Send button toggle: empty input shows mic, typing shows send
  - MediaRecorder + Web Audio API silence detection (1.5s threshold, 60s max)
  - Audio → base64 → POST to Edge Function (no multipart)
  - Tool calls render as italic system bubbles ("🔍 Looking up Jones")
  - End-of-conversation confirm-and-edit modal with editable subject/tags/body/next_action
  - Submit writes to `field_notes` with V3-compatible shape
  - V3 form preserved as one-tap "Use form ▸" escape hatch, hidden by default

  **Iterated V4.0 → V4.0.3 in 4 deploys:**
  - V4.0: initial baseline (Sonnet 4.6, mic on left, no fuzzy match, no Whisper priming)
  - V4.0.1: Mic moved to right with send-button swap, silence auto-stop, tool calls visible, switched to Haiku 4.5 for speed, stronger tool-use prompt
  - V4.0.2: **The friction-killer release.** pg_trgm + fuzzystrmatch extensions enabled. `fn_lookup_entity` rewritten using `similarity()` + `dmetaphone()`. Whisper prompt added with all 9 manufacturers + 14 competitors + top 25 hospitals + common procedures. Q's system prompt rewritten with explicit FRICTION RULES (never reject a note, never argue "did you mean X?" on zero matches — capture freeform and move on; treat "submit/done/that's it" as a hard signal). Added Done ✓ button.
  - V4.0.3: Moved Done button from top bar to below the composer (green pill, prominent). Tightened chat-thread max-height so composer + Done always stay visible.

  **Live-verified fuzzy match on the exact words from John's first failed test:** "Lankanon" → Lankenau Medical Center (phonetic 0.7), "Tiber Health" → TYBR Health (phonetic 0.8), "Petrocelli" → Philip Petrucelli, MD (trigram 0.51). Every entity in his test note was real in the CRM — the system just hadn't been able to connect the dots before V4.0.2.

  **Notes Feed bug fixed:** Empty feed after first V4 submission. Root cause: `manufacturers` table had RLS enabled but only a `service_role` SELECT policy. When V4 produced a note with `manufacturer_id` set, PostgREST 401'd the entire embed query. Old V3 notes never set `manufacturer_id` so this latent bug went undetected. Added `auth_manufacturers_read` and `anon_manufacturers_read` SELECT policies — matches pattern on `locations`/`competitors`. All 12 historical notes load again.

  **CRM update:** Added **FX Shoulder** (id 9, short_name `FX`, focus `Shoulder`, status `active`). Whisper prompt updated to include it.

  **Model choice:** Switched from `claude-sonnet-4-6` to `claude-haiku-4-5` for chat turns. Meaningfully faster (~2x) for this structured dialogue, no observable quality loss in tool-use behavior, cheaper.

  **Strategic discussion (no code) — Neo Mobile concept defined.** Q is for capture (one observation, one field_note row). "Pull the last case Dr X did", "build a target list for Orthocell", "draft an email to Sue Lee" — those are open-ended CRM queries, NOT field notes. That work belongs to **Neo Mobile**, a separate PWA. Decision: build it as a separate project once Field Notes V4 is stable. Architecture: same Supabase auth, same Edge Function pattern (Whisper + Claude tool loop), same PWA shell — but broader system prompt, ~15-20 tools instead of 4, no confirm-and-edit screen (just rich chat answers). User base: John + 4 TMs (Nick, Nate, Noah, Pat). S3 reps don't need it. Estimated v0: ~half a day. New repo when work begins.

  **Open: synthetic data smoke test.** Proposed generating ~100 fictitious field_notes (real CRM IDs, `is_test=true` flag, spread across realistic rep assignments and dates) to stress-test the feed at volume, validate role-based visibility, exercise the fuzzy match across diverse inputs, and prepare a corpus for Field Notes Processing once that pipeline is live. Three options offered: A (all SQL), B (10 via Q live + 90 SQL — recommended), C (all via Q live). Decision pending — picked up at start of next session.

- 2026-05-15 — Session 5: Architecture review on the laptop. Three decisions locked: (1) **Whisper API as the voice runtime** (drop Web Speech API entirely + drop the iOS Safari smoke test) — better medical-vocabulary accuracy, predictable across all devices and PWA standalone mode, ~$0.006/min cost is negligible, eliminates the biggest unknown. (2) **Chat-as-interface (Option B)** — chat window with mic button AND text input bar both always available, V3 form survives as a one-tap "Use form ▸" escape hatch, not a primary mode. Means voice failure no longer means losing the conversational value — rep just keeps typing in the same chat. The previously planned "Talk / Type toggle" framing is dropped. (3) **Cost model added (§5.6)** — ~$0.05/note all-in, ~$55–165/month depending on usage. Also clarified that build machine doesn't matter (Vercel auto-deploys on push). Next step: provision Anthropic + OpenAI API keys, store in Supabase secrets, then build the Edge Function and V4.0 UI.
- 2026-05-06 — Session 4: V4 conversational capture defined and adopted as the rollout target. Reps have not seen V3; we skip its standalone deploy and go straight to V4, which keeps V3's form as the "Type" toggle inside a new mic-first flow. Architecture locked: Claude API required (real-time on phone), voice-first via Web Speech API with text fallback, CRM-aware tool use during conversation (`lookup_entity`, `surgeons_at_location_today`, `recent_notes_about`, `propose_tags`), end-of-conversation confirm-and-edit screen, prompt caching from day one. V4 build sequencing locked at V4.0–V4.4 (see §5.4). Build runs on the Mac Mini; rollout to reps waits until Field Notes Processing Phase 1 + V4.0 are both live so the loop "make a note → see a fact" is visible from day one. Tag bottom-sheet (originally a side-track in the Processing plan) folds into V4.1 as the manual-override surface for auto-suggested tags. Capture-side maintenance fixed this session: header user chip no longer signs out instantly; `field_notes` RLS policy `auth_field_notes_select` patched to allow `rep_id = current_rep_id()` so reps see their own notes regardless of subject (commit `a4ca4bb` on `jwalshAO/field-notes`; RLS change applied via `execute_sql` — backfill into a tracked migration when convenient). Cross-machine pattern adopted: project docs are the laptop ↔ Mini handoff, not Shuttle Folder files.
