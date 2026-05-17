---
title: "Field Notes (was Field Intel Webapp)"
cockpit: true
domain: "Field"
description: "Mobile-first PWA for reps to capture field intelligence via Q (Claude chat). Notes flow into Supabase; the Brain extracts structured facts every 5 minutes, autonomously; facts surface on every CRM entity page (surgeon, hospital, manufacturer, competitor) in the OSO dashboard."
next_action: "Build V4.1.0 contacts (full plan locked in §5/§7 — schema, RPC, Q v9, Brain v3, OSO /contact/[id], PWA wire-through, Todoist follow-up). Then resume self-test. Synthetic-data wipe + anon RLS tighten + rep rollout still gated after V4.1.0 ships."
rollout_target: "V4.0.12 LIVE (new logo + Cancel button + smart chips + 7-tag taxonomy + celebrations). Q Edge Function v8 deployed. Brain v1 LIVE. OSO entity routes LIVE. V4.1.0 (contacts) is the next ship and fully scoped — start there next session."
build_machine: "Either — repo on GitHub, Vercel auto-deploys on push. Supabase MCP handles all backend work."
---

# Field Notes — Living Project Document
Last updated: 2026-05-17 | Session #8 (V4.0.9 → V4.0.12: tag taxonomy rebuild → Interaction/Feedback broadening → Cancel button + no-keyboard-on-chip-tap → new pink/blue Q logo replacing orange icon. V4.1.0 contacts fully scoped, build deferred to next session.)

---

## 1. Project Identity
- **What it is:** Mobile-first PWA for Agility Ortho reps to capture field intel via Q (Claude chat) → structured `field_notes` rows → autonomous Brain extracts structured facts → surfaced on every CRM entity page in OSO dashboard.
- **Goal:** Reps tap an icon, sign in once, chat with Q (or form fallback). Notes flow into Supabase. The Brain digests them 24/7 into structured facts anchored to surgeons/locations/manufacturers/competitors. Those facts surface on entity dashboards so the whole team gets smarter every time anyone enters a note.
- **Owner:** John Walsh
- **Key people:** John (owner, is_admin=true), Nate/Nick/Noah/Pat (TMs), 6 S3 reps
- **Key systems/tools:** Supabase (`oso-tray-tracker`), Vercel (Field Notes PWA + OSO dashboard), GitHub (`jwalshAO/field-notes`, `jwalshAO/surgeon-dashboard`), Anthropic Claude API, OpenAI Whisper
- **Quick start:** To continue this project, say: `"continue Field Notes webapp"`

---

## 2. Current Status
**Full capture → brain → surfacing pipeline LIVE.** V4.0.12 (Field Notes PWA) + Brain v1 + OSO Brain Facts panels on `/surgeon`, `/hospital`, `/manufacturer`, `/competitor`. Session 8 polished the rep-facing layer: 7-tag taxonomy (Interaction / Hospital / Competition / Opportunity / Feedback / Schedule / Observation), smart starter chips that rewrite Q's opening question and pre-select the matching tag, Cancel button for misclicks, no auto-keyboard after chip tap (so Q's tailored question is readable), confetti/count celebrations, and a new pink-and-blue Q logo replacing the orange notebook icon. **V4.1.0 (contacts) is fully scoped but not built** — that's the next ship. John is in multi-day self-test on V4.0.12. Rep rollout still gated on V4.1.0 + synthetic-data wipe + anon RLS tighten + comms.

---

## 3. What Exists (Artifacts & Files)

### Field Notes PWA (`jwalshAO/field-notes` → Vercel)
- `index.html` — V4.0.12: chat with Q + form fallback + confirm-and-edit + Done + Cancel + Unfinished N drafts banner with Resume/Discard + smart starter chips (rewrite Q's question + preset tag, no auto-keyboard) + confetti/count celebrations + new logo references
- `manifest.json` — short_name `Field Notes`, icons reference `icon-192.png` and `icon-512.png`
- `sw.js` — cache `field-intel-v4-12`
- `icon-192.png` / `icon-512.png` / `apple-touch-icon.png` / `icon-source-1024.png` — new pink notebook page with blue Q badge (replaces old orange notebook)
- `favicon-16.png` / `favicon-32.png` — desktop tab favicons (newly added in V4.0.12)
- `icons/` — full source set (android-chrome, apple-touch-icon at 60/76/120/152/167/180, favicon, 1024 master, SVG)
- `icon-192.png` / `icon-512.png` / `apple-touch-icon.png` — orange notebook+pen, edge-to-edge (no whitespace, no built-in corners — iOS rounds on add-to-home-screen)
- `icon-source-1024.png` — archived master

### OSO Surgeon Dashboard (`jwalshAO/surgeon-dashboard` → Vercel)
- `components/EntityFactsPanel.tsx` — shared Brain Facts renderer (groups by fact_kind, confidence chips, cross-entity links)
- `app/surgeon/[id]/page.tsx` — existing; gained "Brain Facts" nav entry
- `app/hospital/[id]/page.tsx` — new; entity header + Brain Facts + Field Notes + Known Surgeons
- `app/manufacturer/[id]/page.tsx` — new; entity header + Brain Facts + Field Notes
- `app/competitor/[id]/page.tsx` — new; entity header + Brain Facts + Field Notes
- ⚠️ Local node_modules broken; Vercel builds are authoritative until `npm install` is rerun

### Supabase (`oso-tray-tracker` / `pchhtltxdcmvdcwnwaeg`)

**Tables**
- `field_notes` — main capture. Columns: `status` (`draft`/`submitted`/`deleted`), `conversation` JSONB (Q transcript), `draft_updated_at`, `brain_processed_at`, `is_test`, `deleted_at`/`deleted_by_rep_id`, plus FKs to surgeon/location/manufacturer/competitor. **V4.1.0 will add `contact_id` FK.**
- `derived_facts` — Brain output. 8 `fact_kind` values, multi-entity anchoring, `confidence`, `observation_date`, `source_field_note_id`, `source_kind` (future-proof for orders/emails), `superseded_by`, `is_test`. **V4.1.0 will add `contact_id` anchor.**
- `manufacturers` — partner manufacturers (SD, Orthocell, Virak, TYBR, Enovis/DJO, Simparo, Captiox, Insight). Contract-heavy schema (commission_rate, contract_start/end, po_email, etc.). Confirmed intentionally separate from `competitors` (lean: name/category/notes/active) — not folding into a single table with `is_partner`.
- **V4.1.0 will add `contacts`** — non-surgeon CRM people: materials managers, OR staff, schedulers, hospital admin, manufacturer reps. Schema: `id, name, role, location_id, status (needs_review/active/inactive), notes, created_at, created_via_field_note_id`. Q auto-creates rows on the fly when reps mention names not in any existing table.

**Edge Functions**
- `field-notes-chat` v8 — Q chat (Whisper + Claude Haiku 4.5 + tool use). System prompt includes the 7-tag taxonomy with renamed Interaction/Feedback (broader). Tool enum on `submit_note.tags` matches. **V4.1.0 will be v9** — adds contact lookup, `create_contact` tool, Todoist POST.
- `field-notes-process` v2 — Brain. `{note_id: N}` single or `{limit: N}` sweep. Catalog-aware (loads manufacturer/competitor lists). Service-role writes to `derived_facts` + `brain_processed_at`. **V4.1.0 will be v3** — anchors facts to `contact_id` when present.
- `neo-mobile-chat` v17 — separate Neo Mobile app (uses same Supabase project). Already integrates with Todoist via `TODOIST_API_TOKEN` Deno env var. Confirms the token is set as a Supabase secret — Field Notes V4.1.0 will reference the same env var. Todoist endpoint: `api.todoist.com/api/v1/tasks` (no project_id = Inbox).

**RPCs:** `fn_lookup_entity`, `fn_surgeons_at_location`, `fn_recent_notes_about` (SECURITY DEFINER)
**Extensions:** `pg_trgm`, `fuzzystrmatch`, `pg_cron`, `pg_net`
**Cron:** `field-notes-brain-sweep` — `*/5 * * * *`, posts to Edge Function with `{limit:10}`

---

## 4. Settled Decisions
- **Q persona** — Bond Q-Branch reference; crisp efficient-sidekick tone, no emojis/exclamation points
- **Whisper API** for voice (session 5)
- **Chat-as-interface** — mic + text both always visible; form is one-tap escape hatch
- **All data in `oso-tray-tracker`** — single source of truth
- **Status lifecycle** (`draft`/`submitted`/`deleted`) + persisted conversation JSONB — abandoned chats survive phone close
- **Brain location: cloud** (Supabase Edge + `pg_cron`), not Mac Mini. Cloud: one platform, instant trigger, no machine to maintain, resilient. Mac Mini still owns markdown-side workflows.
- **Brain fact taxonomy** — 8 fact_kinds, multi-entity anchoring per fact. Manufacturer/competitor IDs resolved via static catalog; surgeon/location IDs restricted to anchored set.
- **Provenance from day one** — every `derived_facts` row has `source_field_note_id` + `source_kind`. Editing a note clears `brain_processed_at` → reprocessing.
- **Icon: orange notebook+pen edge-to-edge** — iOS rounds corners on add-to-home-screen; source artwork lives at `icon-source-1024.png`.
- **Deploy workflow** — GitHub → Vercel auto-deploys on push. `git push origin main`.
- **Starter chips, not free text only** — empty chat needs a nudge; chips give reps a button to tap, also seed Q with note_type context.
- **Smart chips (Option A behavior)** — tapping a chip rewrites Q's opening question to a tailored one for that note type AND pre-selects the matching tag for submission. No extra Anthropic round-trip. (Other = focus input only, no tag.)
- **No auto-focus after question-chip tap** — would pop the keyboard and hide Q's tailored question on iOS. "Other" still focuses since there's no question to read.
- **Cancel button next to Done** — visible the moment a chip is tapped / message sent / mic hit / draft resumed. Confirms only if rep has typed user content. Deletes the persisted draft row if one exists.
- **Tag taxonomy v3 (7 tags)** — **Interaction, Hospital, Competition, Opportunity, Feedback, Schedule, Observation.** Broadened from v2 (Session 8 evolution): Surgeon Interaction → Interaction (covers Jeanie at Methodist, materials managers, OR staff, schedulers, manufacturer reps); Case Feedback → Feedback (broader than OR cases — product, training, billing, service feedback all fit). Tag describes the *kind* of intel; subject_text captures *who*. Hospital + Schedule remain new vs. the original taxonomy; Schedule pairs with Brain's `schedule_observation` fact_kind to feed Surgeon Calendar.
- **Tag distinction: Feedback vs Observation** — Feedback was SAID by someone; Observation is something the rep noticed.
- **Contacts are first-class (V4.1.0)** — non-surgeon recurring people (Jeanie at Methodist, materials managers, etc.) get a `contacts` table so the Brain can anchor facts to them, OSO can show a contact page, and `recent_notes_about` works per-person. Q auto-creates with `status='needs_review'` + Todoist follow-up task — zero capture-time friction.
- **Celebrate every 5th note, big burst on 25/50** — adoption mechanic borrowed from Todoist. canvas-confetti (~5KB), client-side count query filters out `is_test`.
- **Logo is pink notebook page with blue Q badge** (V4.0.12). Replaced the orange notebook+pen from V4.0.7. Source files in `icons/` folder (PNG sizes 16 → 1024 + SVG master). Reps need to delete + re-add the home-screen icon to pick up the new icon on iOS.

---

## 5. Active Work Items

**NEXT SHIP — V4.1.0 contacts (fully scoped, ready to build):**

The flow John wants:
1. Rep mentions someone like "Jeanne at Methodist" (or "Jeanie at Meth" — voice transcription fuzz)
2. Q calls `fn_lookup_entity` which now also searches `contacts` via trigram match
3. **High soft hit (one match, conf ≥ 0.7)** → Q confirms once: *"Jeanie at Methodist — is that her?"*
4. **Multiple soft hits** → Q lists them: *"Did you mean Jeanie or Jeannette? Or new contact?"*
5. **No match OR rep rejects all** → Q silently creates a contact row (`status='needs_review'`, location anchored), POSTs a Todoist task to John's Inbox with a link to the new contact page, and continues capture with `contact_id` linked
6. Brain reads field_note, sees `contact_id`, anchors facts to her
7. Todoist task description includes: `https://surgeon-dashboard-zeta.vercel.app/contact/{id}` so John can tap to review/complete the name+role

Build order (next session, V4.1.0):
- [ ] **Schema migration** — `CREATE TABLE contacts(...)`, `ALTER TABLE field_notes ADD COLUMN contact_id BIGINT REFERENCES contacts(id)`, same for `derived_facts`. Additive only, zero risk to existing data.
- [ ] **RPC update** — extend `fn_lookup_entity` to also search contacts (add `'contact'` to the kinds enum). Trigram match on `name`, optionally boosted when location_id matches the rep's current context.
- [ ] **OSO `/contact/[id]` route** — reuses existing `EntityFactsPanel` component; minimal new code. Add a "Known Contacts" section to `app/hospital/[id]/page.tsx` listing contacts at that location.
- [ ] **Q Edge Function v9** — system prompt teaches the soft-hit confirmation flow; new `create_contact` tool (service-role insert); after create, POST to Todoist; `submit_note` tool gains optional `contact_id`.
- [ ] **Brain Edge Function v3** — load anchored contact alongside surgeon/location; allow `contact_id` in `record_fact` (sanitize against the note's anchored contact_id).
- [ ] **Field Notes PWA** — thread `contact_id` through `persistDraft`, `resumeDraft`, `showConfirmModal`, `finalizeNote`. No new UI needed in chat mode (Q handles it); form mode might want a contact autocomplete later but not required for V4.1.0.
- [ ] **Project doc update** to "V4.1.0 LIVE" once tested.

Token source confirmed: `TODOIST_API_TOKEN` already set as Supabase secret (used by `neo-mobile-chat` v17). Reference via `Deno.env.get("TODOIST_API_TOKEN")`. Tasks go to Todoist Inbox (no project_id specified). Endpoint: `POST https://api.todoist.com/api/v1/tasks` with `Authorization: Bearer ${token}` and JSON body `{content, description, due_string?}`.

**Active — multi-day self-test (started 2026-05-17):**
- [ ] John tests V4.0.12 personally for a few days. Watching for: Q entity-match accuracy, fact usefulness on entity pages, draft-resume behavior after phone close, missing/over-firing fact_kinds, chip→question UX, Cancel button behavior. Ideally test the new logo on home screen (delete + re-add icon to force iOS refresh).

**Before rep rollout (in priority order):**
- [ ] **Synthetic-data wipe** — `DELETE FROM derived_facts; DELETE FROM field_notes WHERE is_test = true; UPDATE field_notes SET brain_processed_at = NULL;` Then let the cron re-process real notes.
- [ ] **Anon RLS tighten** — `anon_field_notes_read` currently `qual = true`; flip to `qual = false` so reads require sign-in.
- [ ] **`npm install`** at `/Users/johnwalsh/Codex/AO/Projects/surgeon-dashboard` — local node_modules broken; needed before next local `next dev`.
- [ ] **Rep rollout comms** — already-installed reps must delete the home-screen icon and re-add it to pick up V4.0.8 + new orange icon (iOS caches launch icon).

**Future scope (deferred):**
- [ ] **Per-rep streak counter** — daily streak indicator near the John dropdown in the header. Data is there, just needs the UI + a last-submitted-date query.
- [ ] **Admin leaderboard** — view of rep counts/streaks on the OSO dashboard side, John-only. Pairs with celebrations.
- [ ] **Brain v2 — cross-entity synthesis** — scheduled job that reads `derived_facts` and emits higher-order observations (schedule patterns, conversion likelihood, complication trends). Also writes summaries to Interaction Log sections of CRM markdown pages on the Mac Mini.
- [ ] **Surgeon Calendar skill** — predictive "where will Bozentka be Thursday" from `schedule_observation` facts + order ship-to data + calendar invites. Separate project, consumes Brain output.
- [ ] **Fact supersession logic** — schema has `superseded_by` column but Brain v1 doesn't use it. Brain v2 will need a rule.
- [ ] **Markdown sync** — Brain v2 writes summaries back into Surgeon/Hospital/Manufacturer/Contact `.md` Interaction Log sections via scheduled task on Mac Mini.
- [ ] **Contact autocomplete in form mode** — V4.1.0 ships chat-mode contact handling only. If reps want a contact subject picker in the V3 form fallback, add later.

**Useful queries**
- Brain backlog: `SELECT count(*) FROM field_notes WHERE status='submitted' AND brain_processed_at IS NULL;`
- Facts about a surgeon: `SELECT fact_kind, fact_text, confidence FROM derived_facts WHERE surgeon_id = X AND superseded_by IS NULL ORDER BY confidence DESC;`
- Manual brain trigger: POST to `https://pchhtltxdcmvdcwnwaeg.supabase.co/functions/v1/field-notes-process` with `{"note_id": N}` or `{"limit": N}`

---

## 6. Open Questions & Blockers
- **"Add new surgeon" inline in V4 chat** — still kicks to V3 modal. UX shape for v4.x TBD.
- **Photo capture in chat mode** — V4.3 future scope.
- **Q chat conversation cost vs draft persistence frequency** — every Q turn now writes a draft row. Fine at current scale; worth monitoring at 10× usage.

---

## 7. Constraints & Preferences
- Light/white backgrounds for all HTML
- Tags 11px, small/unobtrusive
- Terminal commands one-shot
- All files inside `/Users/johnwalsh/Codex`
- Git push only — no `gh` or `vercel` CLI
- `oso-tray-tracker` project ID: `pchhtltxdcmvdcwnwaeg`
- Field Notes repo: `jwalshAO/field-notes` → https://field-intel-webapp.vercel.app
- OSO dashboard repo: `jwalshAO/surgeon-dashboard` → https://surgeon-dashboard-zeta.vercel.app

---

## 8. Background Context
- Agility Ortho — upper extremity orthopedic distribution (Eastern PA, South NJ, Northern DE)
- 11 team members: John (owner), 4 TMs, 6 S3 reps
- `oso-tray-tracker` is the master CRM database
- The Field Notes app is pure INPUT and synthesis. The OSO entity pages are where reps SEE the output.
- John's vision: an always-on brain digesting field_notes + calendar + orders + emails → connections, complications, opportunity signals on every entity page

---

## 9. History Log
- 2026-04-02 — Sessions 1-3: V1-V3 build (schema, OTP, surgeon autocomplete, photo capture, multi-entity Subject, soft-delete, splash)
- 2026-05-06 — Session 4: V4 conversational capture defined; Q persona introduced; sequencing locked
- 2026-05-15 — Session 5: Whisper locked; chat-as-interface; cost model
- 2026-05-16 — Session 6: V4.0 shipped — RPCs, Edge Function, chat UI, fuzzy match, Whisper priming, V4.0→V4.0.3 iterations
- 2026-05-17 — Session 8: **V4.0.9 → V4.0.12 — adoption polish, tag rebuild, contacts scoped, new logo.**
  - **V4.0.8** (early session): canvas-confetti (~5KB CDN) + per-rep count toast on submit. Milestones: 1st note, every 5th, big burst on 25/50. Filters `is_test`. Starter chips on empty chat (v1 — generic seed messages, "Option B behavior").
  - **V4.0.9**: Tag taxonomy v2 (7 tags) — Surgeon Interaction, Hospital, Competition, Opportunity, Case Feedback, Schedule, Observation. Smart chips switched to **Option A behavior**: tapping a chip rewrites Q's opening bubble to a tailored question AND pre-selects the matching tag for submission. No extra Anthropic round-trip. `pendingChipTag` survives Q forgetting. Q Edge Function v7 deployed.
  - **V4.0.10**: Cancel button next to Done (visible after chip/type/mic/resume; confirms if user content; deletes draft row). No auto-focus after question-providing chip tap so keyboard doesn't hide Q's question.
  - **V4.0.11**: Tag taxonomy v3 (broader names) — **Surgeon Interaction → Interaction** (covers Jeanie at Methodist, materials managers, OR staff, schedulers, manufacturer reps); **Case Feedback → Feedback** (broader than OR cases — product, training, billing). Q Edge Function v8 deployed; system prompt clarifies non-surgeon subjects are valid freeform.
  - **V4.0.12**: New logo — **pink notebook page with blue Q badge** (replaces orange notebook+pen). Replaced `icon-192`, `icon-512`, `apple-touch-icon`, `icon-source-1024`. Added `favicon-16`/`favicon-32` with HTML head links. Full source set committed under `icons/` (android-chrome, apple-touch sizes 60→180, favicon, 1024 master, SVG).
  - **V4.1.0 fully scoped, build deferred**: Contacts table + `field_notes.contact_id` + `derived_facts.contact_id` + RPC extension + Q v9 (soft-hit confirm flow, `create_contact` tool, Todoist POST) + Brain v3 + OSO `/contact/[id]` route + Hospital page contacts section + PWA wire-through. `TODOIST_API_TOKEN` already set as Supabase secret (confirmed via `neo-mobile-chat` v17). See §5 for the full plan.
  - Schema discussion: confirmed `manufacturers` (partners, contract-heavy) and `competitors` (lean) are intentionally separate tables — not folding into one with `is_partner`.
  - Decisions deferred: leaderboard, streak counter, Brain v2, Surgeon Calendar skill, fact supersession logic.
  - SW cache progression: v4-7 → v4-8 → v4-9 → v4-10 → v4-11 → v4-12.
  - Edge Function progression: field-notes-chat v6 → v7 → v8.
  - **Quick-start for next session**: `"continue Field Notes webapp"` — and the next step is to build V4.1.0 contacts as scoped.
- 2026-05-16 — Session 7: **Capture + Brain + Surfacing all LIVE.**
  - V4.0.4: `is_test` flag added; feed filters it out
  - V4.0.5: `status` lifecycle + `conversation` JSONB + `draft_updated_at`; drafts banner with Resume/Discard; Submit UPDATEs draft row
  - V4.0.6: rebrand "Field Intel" → "Field Notes" everywhere user-visible; splash blurb tightened
  - V4.0.7: new orange notebook+pen icon, edge-to-edge fill
  - Brain v1: created `derived_facts` (8 fact_kinds, multi-entity anchoring, provenance, supersession-ready); added `brain_processed_at` to field_notes; deployed `field-notes-process` Edge Function v2 (catalog-aware); `pg_cron` sweep every 5 min. Decided cloud over Mac Mini for continuous processing.
  - OSO Brain Facts surfacing: new `EntityFactsPanel` component + `/hospital/[id]`, `/manufacturer/[id]`, `/competitor/[id]` routes; Brain Facts tab added to `/surgeon/[id]`. 
  - 90 synthetic notes inserted (`is_test=true`) to exercise the pipeline; brain extracted 170 facts from 96 notes (90 synthetic + 6 of 13 real, including the Kahlon/Arthrex/First State series). Cron caught up the entire backlog within the session.
  - Strategic clarification: John's "Mac Mini processing" ask was actually a "continuous synthesis brain across field_notes + calendar + orders + everything" ask — cloud is the right answer. Sequence locked: status column → Brain v1 → OSO routes → Brain v2 → Surgeon Calendar skill.
