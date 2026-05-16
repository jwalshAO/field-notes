---
title: "Field Notes (was Field Intel Webapp)"
cockpit: true
domain: "Field"
description: "Mobile-first PWA for reps to capture field intelligence via Q (Claude chat). Notes flow into Supabase; the Brain extracts structured facts every 5 minutes, autonomously; facts surface on every CRM entity page (surgeon, hospital, manufacturer, competitor) in the OSO dashboard."
next_action: "Wipe synthetic data when ready (one-line query in §5), then optionally proceed to Brain v2 (cross-entity synthesis) and/or the Surgeon Calendar skill. Rep rollout is no longer gated by infra — capture, brain, and surfacing are all live."
rollout_target: "V4.0.7 LIVE. Brain v1 LIVE. OSO routes for surgeon/hospital/manufacturer/competitor LIVE. Rep rollout ready pending synthetic-data wipe."
build_machine: "Either — repo on GitHub, Vercel auto-deploys on push. Supabase MCP handles all backend work."
---

# Field Notes — Living Project Document
Last updated: 2026-05-16 | Session #7 (Brain v1 LIVE + autonomous 5-min cron sweep + draft persistence + rebrand + new orange icon + OSO Brain Facts surfacing across all entity types)

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
**Full capture → brain → surfacing pipeline LIVE.** V4.0.7 (Field Notes PWA) + Brain v1 + OSO Brain Facts panels on `/surgeon`, `/hospital`, `/manufacturer`, `/competitor`. As of session 7 close: 96 notes processed, 170 facts extracted, 0 pending — the cron caught up the entire backlog. Not yet rolled out to reps; gate is now just the synthetic-data wipe (John may keep the fictional data for further validation, or wipe immediately). No infra blockers remain for rep rollout.

---

## 3. What Exists (Artifacts & Files)

### Field Notes PWA (`jwalshAO/field-notes` → Vercel)
- `index.html` — V4.0.7: chat with Q + form fallback + confirm-and-edit + Done + Unfinished N drafts banner with Resume/Discard
- `manifest.json` — short_name `Field Notes`
- `sw.js` — cache `field-intel-v4-7`
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
- `field_notes` — main capture. Columns: `status` (`draft`/`submitted`/`deleted`), `conversation` JSONB (Q transcript), `draft_updated_at`, `brain_processed_at`, `is_test`, `deleted_at`/`deleted_by_rep_id`, plus FKs to surgeon/location/manufacturer/competitor
- `derived_facts` — Brain output. 8 `fact_kind` values, multi-entity anchoring, `confidence`, `observation_date`, `source_field_note_id`, `source_kind` (future-proof for orders/emails), `superseded_by`, `is_test`

**Edge Functions**
- `field-notes-chat` v3 — Q chat (Whisper + Claude Haiku 4.5 + tool use)
- `field-notes-process` v2 — Brain. `{note_id: N}` single or `{limit: N}` sweep. Catalog-aware (loads manufacturer/competitor lists). Service-role writes to `derived_facts` + `brain_processed_at`.

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

---

## 5. Active Work Items

**Pending (in priority order):**
- [ ] **Synthetic-data wipe before rep rollout** — `DELETE FROM derived_facts; DELETE FROM field_notes WHERE is_test = true; UPDATE field_notes SET brain_processed_at = NULL;` Then let the cron re-process the 13 real notes.
- [ ] **Anon RLS tighten** — `anon_field_notes_read` currently `qual = true`; flip to `qual = false` so reads require sign-in.
- [ ] **`npm install`** at `/Users/johnwalsh/Codex/AO/Projects/surgeon-dashboard` — local node_modules broken; needed before next local `next dev`.
- [ ] **Rep rollout comms** — already-installed reps must delete the home-screen icon and re-add it to pick up the new orange icon (iOS caches launch icon).
- [ ] **Brain v2 — cross-entity synthesis** — scheduled job that reads `derived_facts` and emits higher-order observations (schedule patterns, conversion likelihood, complication trends). Also writes summaries to Interaction Log sections of CRM markdown pages on the Mac Mini.
- [ ] **Surgeon Calendar skill** — predictive "where will Bozentka be Thursday" from `schedule_observation` facts + order ship-to data + calendar invites. Separate project, consumes Brain output.
- [ ] **Fact supersession logic** — schema has `superseded_by` column but Brain v1 doesn't use it. Brain v2 will need a rule.
- [ ] **Markdown sync** — Brain v2 writes summaries back into Surgeon/Hospital/Manufacturer `.md` Interaction Log sections via scheduled task on Mac Mini.

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
- 2026-05-16 — Session 7: **Capture + Brain + Surfacing all LIVE.**
  - V4.0.4: `is_test` flag added; feed filters it out
  - V4.0.5: `status` lifecycle + `conversation` JSONB + `draft_updated_at`; drafts banner with Resume/Discard; Submit UPDATEs draft row
  - V4.0.6: rebrand "Field Intel" → "Field Notes" everywhere user-visible; splash blurb tightened
  - V4.0.7: new orange notebook+pen icon, edge-to-edge fill
  - Brain v1: created `derived_facts` (8 fact_kinds, multi-entity anchoring, provenance, supersession-ready); added `brain_processed_at` to field_notes; deployed `field-notes-process` Edge Function v2 (catalog-aware); `pg_cron` sweep every 5 min. Decided cloud over Mac Mini for continuous processing.
  - OSO Brain Facts surfacing: new `EntityFactsPanel` component + `/hospital/[id]`, `/manufacturer/[id]`, `/competitor/[id]` routes; Brain Facts tab added to `/surgeon/[id]`. 
  - 90 synthetic notes inserted (`is_test=true`) to exercise the pipeline; brain extracted 170 facts from 96 notes (90 synthetic + 6 of 13 real, including the Kahlon/Arthrex/First State series). Cron caught up the entire backlog within the session.
  - Strategic clarification: John's "Mac Mini processing" ask was actually a "continuous synthesis brain across field_notes + calendar + orders + everything" ask — cloud is the right answer. Sequence locked: status column → Brain v1 → OSO routes → Brain v2 → Surgeon Calendar skill.
