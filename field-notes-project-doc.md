---
title: "Field Notes (was Field Intel Webapp)"
cockpit: true
domain: "Field"
description: "Mobile-first PWA for reps to capture field intelligence via Q (Claude chat). Notes flow into Supabase; the Brain extracts structured facts every 5 minutes, autonomously; facts surface on every CRM entity page (surgeon, hospital, manufacturer, competitor, contact) in the Neo CRM (formerly OSO) admin portal."
next_action: "**Session 19 (2026-05-30) — PWA touched again: V4.2.0.** Replaced the V4.1.9 tap-to-lock visibility row on the confirm screen with a proper **Private toggle switch** (off=shared default, on=private), moved up to sit **between Subject and Notes**. Form-fallback save now respects rep `default_notes_shared` (owner=private). John (rep id 6) already had `default_notes_shared=false` so his confirm screen opens Private-on. SW cache v4-30 → **v4-31** (reps reopen the PWA to pull it). Commit c6f341f. Same Private toggle was mirrored on the Neo CRM `QuickNoteModal` (see People hub doc). Awaiting Noah's first real-world notes. Next: watch Q recognition on Noah's names; consider mirroring the count-up/whimsical-loader polish into the PWA if desired."
rollout_target: "Field Notes PWA V4.2.0 — commit c6f341f on jwalshAO/field-notes (Vercel auto-deploy), SW cache field-intel-v4-31. Q v19 unchanged. Prior Session 18 Neo CRM commits still live (afeff2e, 5e44f1a, f5b6da8, cad2a0d)."
build_machine: "Either — repo on GitHub, Vercel auto-deploys on push. Supabase MCP handles all backend work."
---

# Field Notes — Living Project Document
Last updated: 2026-05-30 | Session #19 (PWA V4.2.0 — Private toggle switch on confirm screen, between Subject and Notes)

---

## 1. Project Identity
- **What it is:** Mobile-first PWA for Agility Ortho reps to capture field intel via Q (Claude chat) → structured `field_notes` rows → autonomous Brain extracts structured facts → surfaced on every CRM entity page in the Neo CRM admin portal.
- **Goal:** Reps tap an icon, sign in once, chat with Q (or form fallback). Notes flow into Supabase. The Brain digests them 24/7 into structured facts anchored to surgeons/locations/manufacturers/competitors/contacts. Those facts surface on entity dashboards so the whole team gets smarter every time anyone enters a note.
- **Owner:** John Walsh
- **Key people:** John (owner, is_admin=true), Nate/Nick/Noah/Pat (TMs), 6 S3 reps. **Noah = first rep on Field Notes (2026-05-24).**
- **Key systems/tools:** Supabase (`oso-tray-tracker`), Vercel (Field Notes PWA + Neo CRM portal), GitHub (`jwalshAO/field-notes`, `jwalshAO/neo-crm`), Anthropic Claude API, OpenAI Whisper
- **Quick start:** To continue this project, say: `"continue Field Notes webapp"`

---

## 2. Current Status
**Session #19 (2026-05-30) — PWA V4.2.0: Private toggle.** Per John, replaced the V4.1.9 tap-to-lock visibility row on the confirm screen with a proper **"Private" toggle switch** (off = shared/team default, on = private) and moved it **up, between Subject and Notes** so the visibility choice is the first thing after the subject. The form-fallback insert (rare no-chat path, previously had no control) now sets `is_shared` from the rep's `default_notes_shared` so a fallback note can't accidentally go public. John's rep (id 6) already had `default_notes_shared=false`, so his confirm screen opens with Private ON. SW cache bumped v4-30 → **v4-31** (reps must reopen the PWA once to pull V4.2.0). Commit `c6f341f`. The identical Private toggle was mirrored onto the Neo CRM `QuickNoteModal` (tracked in the People hub project doc). Everything else from Session 18 still live.

**Field Notes PWA: LIVE with Noah, V4.2.0 (was V4.1.9) + Q v19 + SW cache v4-31. Field Notes _surfacing_ (Neo CRM portal): four shipped today.** Session 18 started as housekeeping but pivoted hard when John reported that Paul Sibley's Geminus distal radius prefs "didn't save" after two attempts. Postgres logs showed the save (RPC) returned 200 and the row was intact — three GET `/rest/v1/surgeon_preferences` calls were 400ing. Root cause: the `reps` table has a single `name` column, not `first_name`/`last_name`; every embedded `reps(first_name, last_name)` PostgREST select was failing and the components silently fell back to empty. Same bug in 6 files, ~21 sites — including the "Assigned Rep(s)" panel on every surgeon page. Fixed in afeff2e. From there John kept building: a new Covering Notes panel at the top of Surgical Preferences (general OR style — "likes to teach," "moves fast," etc. — 4 checkboxes + Other notes, stored in new `surgeons.or_style` JSONB); a click-to-expand detail popup so procedure cards stay compact in the list but open into a 2xl modal with categorized Geminus sections; and a `yours` personal-notes counter as the new first cell on the Field Notes portal tile (impersonation-aware via targetRepId). Field Notes PWA itself untouched this session — awaiting Noah's first real notes.

---

## 3. What Exists (Artifacts & Files)

### Field Notes PWA (`jwalshAO/field-notes` → Vercel) — unchanged this session
- `index.html` V4.2.0 — chat with Q + form fallback + confirm-and-edit + Done + Cancel + Unfinished N drafts banner + smart starter chips + confetti + Team tab (owner-only) + first-time rep intro modal + rep context passed to Q + lock/share toggle per note in the feed + **V4.2.0 Private toggle SWITCH on the confirm screen, between Subject and Notes** (replaced the V4.1.9 tap-to-lock row; `confirmVisSwitch`/`confirmVisKnob`/`confirmVisSub` + rewritten `renderConfirmVisibility()`; `confirmIsShared` state). Form-fallback `submitNote` insert now sets `is_shared` from `currentUser.default_notes_shared`.
- `manifest.json` — short_name `Field Notes`
- `sw.js` — cache `field-intel-v4-31`
- `icons/` — pink notebook + blue Q badge

### Neo CRM portal (`jwalshAO/neo-crm` → Vercel) — Session 18 changes
- `components/SurgeonPreferencesPanel.tsx` — **Covering Notes** section at top (amber border, 4 checkboxes + Other notes); click-to-expand cards (whole card clickable, "View details →" affordance); new `ViewModal` with categorized Geminus layout; new `GeminusDetail` component (sections: Plate Setup / Plate Placement / Distal Fixation / Other, empty sections hidden); flat `GeminusSummary` retained for the compact list view; `direct_to_35_screw` chip finally rendered. Fixed `reps(name)` join.
- `app/page.tsx` — Field Notes tile: 3-col → 4-col stat grid; new `yours` (indigo) cell leads, computed from `weekRows.filter(n => n.rep_id === targetRepId).length`.
- `app/surgeon/[id]/page.tsx`, `app/hospital/[id]/page.tsx`, `app/manufacturer/[id]/page.tsx`, `app/competitor/[id]/page.tsx`, `app/contact/[id]/page.tsx` — all 5 fixed `reps(first_name, last_name)` → `reps(name)` (interfaces, selects, display sites).

### Supabase (`oso-tray-tracker` / `pchhtltxdcmvdcwnwaeg`)

**New this session**
- `surgeons.or_style jsonb` (+ `or_style_updated_at`, `or_style_updated_by_rep_id`) — surgeon-level OR style ("Covering Notes"). Shape: `{ tags: text[], other_notes: text|null }`. Tag vocab v1: `likes_to_teach`, `moves_fast`, `less_said`, `likes_guidance`.
- `update_surgeon_or_style(p_surgeon_id integer, p_or_style jsonb)` SECURITY DEFINER RPC. Resolves rep from `auth.uid()` via reps.user_id. Auto-clears the column when both tags empty AND notes empty. GRANT EXECUTE TO authenticated.

**Tables/RPCs/Edge Functions otherwise unchanged from Session 17.** Cron `field-notes-brain-sweep` still `*/5`.

---

## 4. Settled Decisions
- **Q persona** — Bond Q-Branch reference; crisp efficient-sidekick tone, no emojis/exclamation points
- **Whisper API** for voice; **chat-as-interface** with form fallback
- **All data in `oso-tray-tracker`** — single source of truth
- **Brain location: cloud** (Supabase Edge + `pg_cron`), not Mac Mini
- **Brain fact taxonomy** — 8 fact_kinds, multi-entity anchoring per fact
- **Provenance from day one** — every `derived_facts` row has `source_field_note_id` + `source_kind`
- **Tag taxonomy** — 7 topic tags + 3 contact-type tags (Meeting/Demo, Tend, Give to Grow)
- **Contacts first-class (V4.1.0)**, **candidates as reference-only pool (V4.1.1)**, **nickname architecture two-tier (V4.1.6 + V4.1.7)**, **Q speaker-vs-subject grammar (V4.1.7)**, **lock/share visibility per note (V4.1.8 + V4.1.9)**
- **Covering Notes architecture (Session 18)** — surgeon-level OR style lives in `surgeons.or_style` JSONB, NOT in `surgeon_preferences` (which is per-procedure) and NOT in `surgeon_field_guide` (also procedure-scoped). One row per surgeon, controlled vocab + free text, edited via `update_surgeon_or_style` RPC.
- **Compact list cards + click-to-expand detail popup (Session 18)** — procedure preference cards stay one-line in the list; click opens a 2xl read-only modal with categorized sections. Edit button in modal footer hops to existing edit modal. Same pattern is the model for any future Geminus-style structured surface.

---

## 5. Active Work Items

**✅ Shipped this session:**
- afeff2e — reps schema fix (`reps(first_name, last_name)` → `reps(name)`) across 6 files / 21 sites
- 5e44f1a — Covering Notes section + missing 3.5-screw chip
- f5b6da8 — click-to-expand detail popup with categorized Geminus sections
- cad2a0d — `yours` stat on Field Notes portal tile

**Active — Noah's first real notes (in flight, unchanged from S17):**
- [ ] Watch for: Q recognition accuracy on his rep-specific names, fact usefulness on entity pages, any role-authority misses, any new nickname pairs that need adding.

**Housekeeping:**
- [ ] `npm install` at renamed surgeon-dashboard folder before next local `next dev`. Doesn't affect Vercel.
- [ ] Local folder rename `AO/Projects/surgeon-dashboard/` → `AO/Projects/neo-crm/` + update Codex doc references.

**Future scope (deferred):**
- [ ] **Field Guide frontend tabs** on Neo CRM `/surgeon/[id]` and `/hospital/[id]` — backend (V4.2.0 schema, RLS, upsert RPCs) shipped Session 17. Frontend deferred — John removed from active list at start of S18.
- [ ] **Optimize `yours` counter** — currently computed from 200-row cap of fetched noteRows. When team-wide volume exceeds ~7 notes/day for a week, swap to `head: true, count: 'exact'` request scoped by rep_id.
- [ ] **Nicknames management UI** — `/admin/surgeons/[id]` (and contact/candidate) edit field for the `nicknames text[]` arrays. Today John adds via SQL.
- [ ] **AO-Wide flag on notes** — Google Spaces `#ao-intel` channel for cross-territory broadcast.
- [ ] **SMS OTP sign-in (Twilio)** — folded into `AO/Projects/twilio-sms-program-project-doc.md`.
- [ ] **Per-rep streak counter** + **admin leaderboard** on Neo CRM
- [ ] **Brain v2** — cross-entity synthesis; fact supersession logic; markdown sync back to entity `.md` Interaction Logs via Mac Mini scheduled task
- [ ] **Surgeon Calendar skill** — predictive "where will Bozentka be Thursday"
- [ ] **Contact autocomplete in form mode**, **contact merge UI**, **editable role/anchor on contact page**
- [ ] **Widen the edit modal** to match the new 2xl detail popup breathing room
- [ ] **Add more Covering Notes vocab** as patterns emerge from Noah/Nate/Nick/Pat

**Useful queries**
- Backlog: `SELECT count(*) FROM field_notes WHERE status='submitted' AND brain_processed_at IS NULL;`
- OR style for a surgeon: `SELECT or_style, or_style_updated_at FROM surgeons WHERE id = X;`
- Update Covering Note tag vocab: edit `OR_STYLE_OPTIONS` array in `components/SurgeonPreferencesPanel.tsx` (no DB enum constraint — keys are validated client-side only).

---

## 6. Open Questions & Blockers
- **"Add new surgeon" inline in V4 chat** — still kicks to V3 modal. UX shape for v4.x TBD.
- **Photo capture in chat mode** — V4.3 future scope.
- **Q chat conversation cost vs draft persistence frequency** — every Q turn writes a draft row. Monitor at 10× usage.
- **Ambiguous single-token nicknames** — "Boz" returns Bozentka (0.95) AND Bozdogan (0.90). Q disambiguates; reassess if Bozdogan causes friction.
- **Covering Notes tag vocab** — locked to 4 in v1. Watch which patterns reps describe in free-text "Other notes" — those become candidates for promotion to checkboxes.

---

## 7. Constraints & Preferences
- Light/white backgrounds for all HTML
- Tags 11px, small/unobtrusive
- Terminal commands one-shot
- All files inside `/Users/johnwalsh/Codex`
- Git push only — no `gh` or `vercel` CLI
- `oso-tray-tracker` project ID: `pchhtltxdcmvdcwnwaeg`
- Field Notes repo: `jwalshAO/field-notes` → https://field-intel-webapp.vercel.app
- Neo CRM portal repo: `jwalshAO/neo-crm` → https://portal.agilityortho.com
- **Google Apps Script links → Google Chrome, NOT Safari.**
- **IDN caution** — `@jefferson.edu`, `@mlhs.org`, `@uphs.upenn.edu` etc. span multiple facilities.
- **reps table has a single `name` column** — NOT first_name/last_name. (Hard-won this session.)

---

## 8. Background Context
- Agility Ortho — upper extremity orthopedic distribution (Eastern PA, South NJ, Northern DE)
- 11 team members: John (owner), 4 TMs, 6 S3 reps
- `oso-tray-tracker` is the master CRM database
- The Field Notes app is pure INPUT and synthesis. The Neo CRM entity pages are where reps SEE the output.
- **Brand map (2026-05-24):** Neo CRM = the system (admin portal + Supabase backend); Field Notes + Tray Tracker = rep-facing tools that write into Neo CRM. OSO is just Tray Tracker's bear logo identity.
- **Surgeon Preferences vs Field Guide vs Covering Notes** — three distinct surfaces: (1) `surgeon_preferences` = per-procedure structured (Geminus playbook checkboxes + technique notes). (2) `surgeon_field_guide` (Session 17 backend) = per-procedure categorical free-text (positioning, fluoro, etc.) — frontend pending. (3) `surgeons.or_style` (Session 18) = general OR style, NOT procedure-scoped — what a covering rep needs regardless of what's being done. Visually all three render on the same "Surgical Preferences" tab eventually, with Covering Notes at the top.

---

## 9. History Log
- **2026-05-30 — Session 19: PWA V4.2.0 — Private toggle.** Replaced the V4.1.9 tap-to-lock visibility row on the confirm screen with a real "Private" toggle switch (off=shared default, on=private), moved up to between Subject and Notes. Form-fallback `submitNote` now respects rep `default_notes_shared` (owner=private; John id 6 already false). SW cache v4-30 → v4-31. Commit c6f341f. Mirrored the same Private toggle onto Neo CRM `QuickNoteModal` (People hub doc). Part of a broader Neo CRM portal polish session (surgeon hub, hover glow ring, count-up, whimsical loaders, colored CRM tile) tracked in the People hub project doc.
- **2026-05-25 — Session 18: Neo CRM portal session.** Sibley Geminus "didn't save" → reps schema regression diagnosed via API 400 logs (`reps(first_name, last_name)` doesn't exist — single `name` column). Fixed across 6 files / 21 sites (afeff2e). Built Covering Notes panel at top of Surgical Preferences: amber-bordered section, 4 controlled checkboxes (Likes to teach / Moves fast / Less said / Likes guidance) + free-text Other; backed by new `surgeons.or_style jsonb` column + `update_surgeon_or_style` SECURITY DEFINER RPC; auto-clears on empty (5e44f1a). Also added missing `direct_to_35_screw` chip to GeminusSummary (was being saved since Session 16 but never rendered). Pivoted on UX feedback ("this isn't enough" → "maybe each one is a pop up"): kept compact list cards but made the whole card clickable, opening a 2xl ViewModal with new GeminusDetail component that groups chips by category (Plate Setup / Plate Placement / Distal Fixation / Other) with per-section italic notes (f5b6da8). Closed with `yours` stat on Field Notes portal tile — Field Notes tile sub-stat grid 3-col → 4-col, indigo "yours" leads, computed from existing noteRows fetch, impersonation-aware via targetRepId (cad2a0d). Field Notes PWA untouched.
- 2026-05-25 — Session 17 (extended): V4.1.8 lock/share visibility shipped end-to-end + V4.1.9 visibility-at-capture toggle. Intro modal copy overhaul ("the team dossier" + Q-Branch origin sentence). Field Guide V4.2.0 schema rebuilt (`surgeon_field_guide` + `location_field_guide` tables + RLS + upsert RPCs; frontend deferred). SW cache v4-27 → v4-30.
- 2026-05-25 — Session 17: V4.1.6 universal nicknames (230 first-name pairs) + V4.1.7 per-entity nicknames (`nicknames text[]` on surgeons/contacts/candidates) + Q v19 SPEAKER vs SUBJECT grammar + ROLE-AUTHORITY check + trust-the-lookup rule. Tony Matteo end-to-end verified.
- 2026-05-24 — Session 16: Launch prep + Neo CRM rebrand + V4.1.4-5 + intro overhaul. Synthetic data wipe. V4.1.4 mfr + competitor anchoring. V4.1.5 cleanup. Neo CRM rebrand (repo + Vercel + UI). Intro modal Mo Bunnell Give-to-Grow popup. Geminus 2.5/3.5 screw checkbox. Q v17→v18.
- 2026-05-24 — Session 15: V4.1.3 /admin/candidates picker page with set-aside lifecycle.
- 2026-05-24 — Session 14: V4.1.2 phonetic match boost + email-domain backfill (Jeannie→Jeanne fix).
- 2026-05-24 — Session 13: V4.1.1 SHIPPED — phonebook candidates as reference-only pool (2,160 from John's iPhone vCard).
- 2026-05-24 — Session 12: V4.1.0 SHIPPED end-to-end — contacts integration.
- 2026-05-24 — Session 11: V4.3.0 + V4.4.0 — 3 new contact-type chips + first-time rep explainer modal.
- 2026-05-23 — Session 10: V4.2.0 Field Guide capture — built then reverted same session.
- 2026-05-20 — Session 9: V4.0.13 → V4.0.19 adoption polish chain.
- 2026-05-17 — Session 8: V4.0.9 → V4.0.12 — adoption polish, tag rebuild, contacts scoped, new logo.
- 2026-05-16 — Session 7: Capture + Brain + Surfacing all LIVE. Brain v1 with derived_facts.
- 2026-05-15 — Session 5: Whisper locked; chat-as-interface; cost model
- 2026-05-06 — Session 4: V4 conversational capture defined; Q persona introduced
- 2026-04-02 — Sessions 1-3: V1-V3 build
