---
title: "Field Notes (was Field Intel Webapp)"
cockpit: true
domain: "Field"
description: "Mobile-first PWA for reps to capture field intelligence via Q (Claude chat). Notes flow into Supabase; the Brain extracts structured facts every 5 minutes, autonomously; facts surface on every CRM entity page (surgeon, hospital, manufacturer, competitor, contact) in the Neo CRM (formerly OSO) admin portal."
next_action: "**LAUNCHED 2026-05-24, V4.1.7 nickname fixes shipped 2026-05-25.** Noah is the first rep. Real-world testing exposed the Tony↔Anthony nickname gap + the speaker-vs-subject grammar miss; both fixed in V4.1.6 (universal first-name nicknames table) + V4.1.7 (per-entity nicknames text[] column) + Q v19 (SPEAKER vs SUBJECT grammar section + ROLE-AUTHORITY check + trust-the-lookup rule). Tony Matteo flow verified end-to-end on John's phone: Q resolved Tony→Anthony Matteo (contact id=15) and correctly asked which surgeon he meant, then accepted gracefully when John declined to name one. Awaiting Noah's first real notes."
rollout_target: "V4.1.7 LIVE. Cumulative state through this session: (1) Field Notes ready for Noah: Q v19 with Mo Bunnell Give-to-Grow philosophy + SPEAKER vs SUBJECT grammar parsing + ROLE-AUTHORITY check + nickname-aware lookup; intro modal V4.4.x; SW cache v4-27. (2) Lookup is now nickname-aware: 230 universal first-name pairs (Tony↔Anthony, Jim↔James, Liz↔Elizabeth, Bob↔Robert, Bill↔William, Mike↔Michael, etc.) + per-entity nicknames text[] column on surgeons/contacts/candidates (Boz→Bozentka seeded). (3) Phonebook backed by 2,158 candidates anchored to: 731 hospitals + 86 manufacturers + 184 competitors; ghost dup Tony Matteo candidate dismissed. (4) /admin/candidates picker page: unified Contact Type filter + bulk Promote/Set Aside/Dismiss. (5) Whole portal rebranded surgeon-dashboard → Neo CRM. (6) Geminus playbook gained '2.5 drill + 3.5 screw (12mm)' checkbox. DEFERRED: streak counter, admin leaderboard, Brain v2 cross-entity synthesis, Surgeon Calendar skill, nicknames management UI."
build_machine: "Either — repo on GitHub, Vercel auto-deploys on push. Supabase MCP handles all backend work."
---

# Field Notes — Living Project Document
Last updated: 2026-05-25 | Session #17 (V4.1.6 + V4.1.7 + Q v19 — nickname fixes after first real-world Tony Matteo miss; verified end-to-end on John's phone)

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
**LIVE with Noah; V4.1.7 nickname fixes shipped 2026-05-25 from first real-world miss.** John tested with "Tony Matteo told me he would buy a Geminus tray" on 2026-05-25 — Q v18 failed in three ways: didn't recognize Tony as a nickname for Anthony (a real contact id=15 OR inventory clerk at Methodist); had a duplicate "Tony Matteo" ghost candidate (id=1165) from the iPhone vCard load that the contacts-vs-candidates dedup couldn't catch (different first-name form); and Q treated Tony as the surgeon-buyer instead of recognizing the grammar (Tony was the source, "he" was an unnamed surgeon, and an OR inventory clerk doesn't authorize purchases). Three fixes shipped same-session: ghost candidate dismissed; new `name_nicknames` table seeded with 230 universal first-name pairs; new `nicknames text[]` column on surgeons/contacts/candidates for entity-specific shortenings (Bozentka seeded with `{boz}`); `fn_lookup_entity` rebuilt twice (V4.1.6 + V4.1.7) to score both nickname sources; Q v19 deployed with new SPEAKER vs SUBJECT grammar section + ROLE-AUTHORITY check + "trust the lookup" rule. John re-tested on his phone — Q correctly resolved Tony→Anthony Matteo, asked "which surgeon did Tony mean?", and accepted gracefully when John declined to name one (friction rules held). Cumulative state through this session: 24 real field_notes (post-wipe), 0 synthetic, anon RLS still dropped, SW cache v4-27.

---

## 3. What Exists (Artifacts & Files)

### Field Notes PWA (`jwalshAO/field-notes` → Vercel)
- `index.html` — V4.1.0: chat with Q + form fallback + confirm-and-edit + Done + Cancel + Unfinished N drafts banner with Resume/Discard + smart starter chips + confetti/count celebrations + Team tab (owner-only) + V4.4.0 first-time rep explainer modal with reopenable "?" button + V4.1.0 rep_id/rep_name passed to Q + contact_id threaded through draft/submit + "✓ Added X to contacts" system bubble when Q auto-creates
- `manifest.json` — short_name `Field Notes`
- `sw.js` — cache `field-intel-v4-27`
- `icon-*.png` / favicon-*.png — pink notebook + blue Q badge (V4.0.12)
- `icons/` — full source set (android-chrome, apple-touch-icon at 60/76/120/152/167/180, favicon, 1024 master, SVG)

### Neo CRM portal (`jwalshAO/neo-crm` → Vercel, formerly `surgeon-dashboard`)
- `components/EntityFactsPanel.tsx` — shared Brain Facts renderer (groups by fact_kind, confidence chips, cross-entity links); supports `entityKind='contact'`
- `app/surgeon/[id]/page.tsx` — gained "Brain Facts" nav entry
- `app/hospital/[id]/page.tsx` — entity header + Brain Facts + Field Notes + Known Surgeons + Contacts tab
- `app/manufacturer/[id]/page.tsx` — entity header + Brain Facts + Field Notes
- `app/competitor/[id]/page.tsx` — entity header + Brain Facts + Field Notes
- `app/contact/[id]/page.tsx` — contact header (name, role, status chip, anchor links) + Mark Active / Dismiss / Re-Activate actions + notes-append textarea + Intel tab + Field Notes tab
- `app/admin/candidates/page.tsx` — picker page: unified Contact Type filter + bulk Promote/Set Aside/Dismiss

### Supabase (`oso-tray-tracker` / `pchhtltxdcmvdcwnwaeg`)

**Tables**
- `field_notes` — main capture. Status lifecycle + conversation JSONB + brain_processed_at + is_test + soft-delete + `contact_id` FK.
- `derived_facts` — Brain output. 8 fact_kinds, multi-entity anchoring, confidence, observation_date, source provenance, supersession-ready, `contact_id` FK.
- `manufacturers` — partner manufacturers, contract-heavy schema. Confirmed intentionally separate from `competitors`.
- `competitors` — lean schema (name/category/notes/active).
- `contacts` — 28-column schema for non-surgeon people. Anchors: location_id/surgeon_id/manufacturer_id/competitor_id (≥1 required). Status: needs_review/active/inactive. RLS blocks direct writes; SECURITY DEFINER RPCs only. **V4.1.7: + `nicknames text[]` column.**
- `contact_candidates` — phonebook reference pool. 2,160 seeded from John's iPhone vCard. No anchor requirement. Searched alongside contacts via fn_lookup_entity; promoted via RPC to become real contacts. **V4.1.7: + `nicknames text[]` column.** (ghost dup id=1165 dismissed 2026-05-25.)
- `surgeons` — pre-existing surgeon master table. **V4.1.7: + `nicknames text[]` column (Bozentka seeded with `{boz}`).**
- **`name_nicknames` — NEW V4.1.6.** Universal first-name pair table (formal/casual). 230 pairs seeded: Tony↔Anthony, Jim↔James, Liz↔Elizabeth, Bob↔Robert, Bill↔William, Mike↔Michael, Kate↔Katherine, Jeanne↔Jeannie, Sue↔Susan, etc. Symmetric — lookups check both directions. GRANTed to anon/authenticated/service_role.

**Edge Functions**
- `field-notes-chat` **v19** — Q chat (Whisper + Claude Haiku 4.5 + tool use). V4.1.7: new SPEAKER vs SUBJECT grammar section teaching that "X told me he…" / "per X…" / "X said Y" patterns mean X is the SOURCE (contact), not the subject; ROLE-AUTHORITY CHECK teaching that inventory clerks/MMs/schedulers report, surgeons decide; "trust the lookup — don't dismiss high-confidence hits because the matched role doesn't fit your inferred subject" rule; lookup_entity tool description updated to mention nickname-awareness. V4.1.1: promote_candidate tool + kind='candidate' soft-hit flow.
- `field-notes-process` v7 — Brain. V4.1.0: loads anchored contact, accepts contact_id, role-authority weighting.
- `neo-mobile-chat` v17 — separate Neo Mobile app, same Supabase project.

**RPCs**
- **`fn_lookup_entity` V4.1.7** — Adds per-entity nicknames branch on top of V4.1.6 universal first-name nicknames. Scoring summary:
  - Per-entity nickname match (surgeons): 0.95
  - Per-entity nickname match (contacts): 0.93
  - Per-entity nickname match (candidates): 0.88
  - Universal first-name nickname + last-name match (contacts/surgeons): 0.90
  - Universal first-name nickname + last-name match (candidates): 0.85
  - Plus prior trigram, phonetic (dmetaphone, V4.1.2 boosted), prefix, location-anchor +0.15 boost (contacts/candidates only)
- `fn_surgeons_at_location`, `fn_recent_notes_about` (SECURITY DEFINER reads)
- Write RPCs: `upsert_contact`, `update_contact_status`, `dispose_contact_replacement`, `promote_contact_candidate`, `dismiss_contact_candidate`, `set_aside_contact_candidate`, `unset_aside_contact_candidate`

**Extensions:** `pg_trgm`, `fuzzystrmatch`, `pg_cron`, `pg_net`
**Cron:** `field-notes-brain-sweep` — `*/5 * * * *`, posts to Edge Function with `{limit:10}`

---

## 4. Settled Decisions
- **Q persona** — Bond Q-Branch reference; crisp efficient-sidekick tone, no emojis/exclamation points
- **Whisper API** for voice
- **Chat-as-interface** — mic + text both always visible; form is one-tap escape hatch
- **All data in `oso-tray-tracker`** — single source of truth
- **Status lifecycle** (`draft`/`submitted`/`deleted`) + persisted conversation JSONB — abandoned chats survive phone close
- **Brain location: cloud** (Supabase Edge + `pg_cron`), not Mac Mini
- **Brain fact taxonomy** — 8 fact_kinds, multi-entity anchoring per fact
- **Provenance from day one** — every `derived_facts` row has `source_field_note_id` + `source_kind`
- **Deploy workflow** — GitHub → Vercel auto-deploys on push
- **Tag taxonomy** — 7 topic tags (Interaction, Hospital, Competition, Opportunity, Feedback, Schedule, Observation) + 3 contact-type tags (Meeting/Demo, Tend, Give to Grow). Multi-tag freely.
- **Contacts are first-class (V4.1.0)** — non-surgeon recurring people get a `contacts` table; Brain can anchor facts to them; OSO contact page; auto-create with `status='needs_review'` + Todoist task
- **Candidates as reference-only pool (V4.1.1)** — phonebook rows separate from contacts; promote on confirmation
- **Nickname architecture (V4.1.6 + V4.1.7)** — two-tier: (a) `name_nicknames` table for universal first-name pairs (Tony↔Anthony) shared across all entities; (b) `nicknames text[]` column for entity-specific shortenings (Boz→Bozentka). Both compose cleanly in fn_lookup_entity.
- **Q speaker-vs-subject grammar (V4.1.7)** — When a rep says "X told me he…" / "per X…" / "X said Y," X is the source/contact, not the subject. Cross-check with role authority (clerks report, surgeons decide). Q asks ONE clarifying question, then submits.
- **Logo: pink notebook + blue Q badge** (V4.0.12)

---

## 5. Active Work Items

**✅ V4.1.6 universal nicknames — SHIPPED 2026-05-25.**
**✅ V4.1.7 per-entity nicknames — SHIPPED 2026-05-25.**
**✅ Q v19 grammar/role-authority/trust-the-lookup — SHIPPED 2026-05-25.**
**✅ Tony Matteo end-to-end verification — DONE 2026-05-25.** John tested on phone; Q correctly resolved Tony→Anthony Matteo contact id=15, asked which surgeon, accepted gracefully when John declined to name one.

**Active — Noah's first real notes (in flight):**
- [ ] Watch Noah's notes for: Q recognition accuracy on his rep-specific names/locations, fact usefulness on entity pages, any role-authority misses, any new nickname pairs that need adding to either the universal table or per-entity column.

**Side issues, non-blocking:**
- [ ] 45 phonebook candidates didn't load (2,160/2,205) — likely SQL parser truncation on rows with very long multi-line NOTE fields. Easy to backfill later via re-running `emit_candidates_sql.py` with ON CONFLICT guards.
- [ ] Jamina has a similar ghost (contacts.id=2 + candidates.id=2004). Not blocking — kind='contact' wins by 0.05 — but a broader sweep for contacts↔candidates dups would tidy. Run periodically as part of CRM hygiene.
- [ ] `npm install` at the renamed `/Users/johnwalsh/Codex/AO/Projects/surgeon-dashboard` folder. Needed before next local `next dev`. Doesn't affect Vercel.
- [ ] Local folder rename `AO/Projects/surgeon-dashboard/` → `AO/Projects/neo-crm/` + update Codex doc references.

**Future scope (deferred):**
- [ ] **Nicknames management UI** — small `/admin/surgeons/[id]` (and contact/candidate) edit field to add/remove entries in the `nicknames text[]` array. Today John adds via SQL. Easy follow-up when more nicknames pile up.
- [ ] **AO-Wide flag on notes** — Google Spaces `#ao-intel` channel for cross-territory broadcast notes. Decided at build time but Google Spaces is the path of least resistance (already in Workspace, webhook-friendly).
- [ ] **SMS OTP sign-in (Twilio)** — folded into broader Twilio SMS Program (see `AO/Projects/twilio-sms-program-project-doc.md`).
- [ ] **Per-rep streak counter** — daily streak indicator near the John dropdown in header
- [ ] **Admin leaderboard** — view of rep counts/streaks on the Neo CRM portal, John-only
- [ ] **Brain v2 — cross-entity synthesis** — scheduled job that reads `derived_facts` and emits higher-order observations
- [ ] **Surgeon Calendar skill** — predictive "where will Bozentka be Thursday"
- [ ] **Fact supersession logic** — `superseded_by` column exists, Brain v1 doesn't use it
- [ ] **Markdown sync** — Brain v2 writes summaries back into Surgeon/Hospital/Manufacturer/Contact `.md` Interaction Log sections via Mac Mini scheduled task
- [ ] **Contact autocomplete in form mode** — V4.1.0 ships chat-mode only
- [ ] **Contact merge UI** — `dispose_contact_replacement` RPC exists but no UI surface yet
- [ ] **Editable role/anchor on contact page** — V4.1.0 ships read-only role/anchor display. `update_contact_fields` RPC + edit UI when painful.

**Useful queries**
- Brain backlog: `SELECT count(*) FROM field_notes WHERE status='submitted' AND brain_processed_at IS NULL;`
- Facts about a surgeon: `SELECT fact_kind, fact_text, confidence FROM derived_facts WHERE surgeon_id = X AND superseded_by IS NULL ORDER BY confidence DESC;`
- Manual brain trigger: POST to `https://pchhtltxdcmvdcwnwaeg.supabase.co/functions/v1/field-notes-process` with `{"note_id": N}` or `{"limit": N}`
- Add per-entity nickname: `UPDATE surgeons SET nicknames = ARRAY['petro'] WHERE last_name = 'Petrocelli';` (same pattern for contacts/contact_candidates)
- Add universal first-name pair: `INSERT INTO name_nicknames (formal, casual) VALUES ('formal_name', 'casual_name');` (lowercase, symmetric — store one direction, lookup checks both)

---

## 6. Open Questions & Blockers
- **"Add new surgeon" inline in V4 chat** — still kicks to V3 modal. UX shape for v4.x TBD.
- **Photo capture in chat mode** — V4.3 future scope.
- **Q chat conversation cost vs draft persistence frequency** — every Q turn writes a draft row. Fine at current scale; monitor at 10× usage.
- **Ambiguous single-token nicknames** — "Boz" alone returns both Bozentka (0.95, via nickname) AND Bozdogan (0.90, via substring). Q's disambiguation handles it. If a rep uses Boz a lot, watch whether Bozdogan ever causes friction; if so, consider boosting per-entity nickname score above substring scores.

---

## 7. Constraints & Preferences
- Light/white backgrounds for all HTML
- Tags 11px, small/unobtrusive
- Terminal commands one-shot
- All files inside `/Users/johnwalsh/Codex`
- Git push only — no `gh` or `vercel` CLI
- `oso-tray-tracker` project ID: `pchhtltxdcmvdcwnwaeg`
- Field Notes repo: `jwalshAO/field-notes` → https://field-intel-webapp.vercel.app
- Neo CRM portal repo: `jwalshAO/neo-crm` → https://portal.agilityortho.com (also surgeon-dashboard-zeta.vercel.app, surgeon-dashboard-agility-ortho.vercel.app)
- **Google Apps Script links → Google Chrome, NOT Safari.**
- **IDN caution** — `@jefferson.edu`, `@mlhs.org`, `@uphs.upenn.edu` etc. span multiple facilities; don't infer location from the domain.

---

## 8. Background Context
- Agility Ortho — upper extremity orthopedic distribution (Eastern PA, South NJ, Northern DE)
- 11 team members: John (owner), 4 TMs, 6 S3 reps
- `oso-tray-tracker` is the master CRM database
- The Field Notes app is pure INPUT and synthesis. The Neo CRM entity pages are where reps SEE the output.
- John's vision: an always-on brain digesting field_notes + calendar + orders + emails → connections, complications, opportunity signals on every entity page
- **Brand map (2026-05-24):** Neo CRM = the system (admin portal + Supabase backend); Field Notes + Tray Tracker = rep-facing tools with their own brands that write into Neo CRM. OSO (Spanish for bear) is just Tray Tracker's logo identity, not a separate product.

---

## 9. History Log
- **2026-05-25 — Session 17: V4.1.6 + V4.1.7 + Q v19 — first real-world miss → nickname fixes.**
  - **The miss:** John tested "Tony Matteo told me he would buy a Geminus tray." Q v18 failed three ways: (1) didn't recognize Tony Matteo (Anthony, contact id=15, OR inventory clerk at Methodist) because Anthony↔Tony nickname gap escaped trigram + dmetaphone; (2) the V4.1.1 vCard load had created a duplicate "Tony Matteo" candidate (id=1165) since the dedup couldn't catch the different-first-name form; (3) Q treated Tony as the buyer despite Tony being an OR inventory clerk who couldn't buy trays — grammar parsing failure ("X told me he would buy Y" → X is source, "he" is unknown surgeon).
  - **V4.1.6 universal first-name nicknames** — migration `name_nicknames_v4_1_6` created the `name_nicknames(formal text, casual text)` table with PRIMARY KEY (formal, casual), indexed both columns, GRANTed SELECT to anon/authenticated/service_role. Seeded 230 pairs covering the most common American/English first-name nicknames (Anthony↔Tony, Robert↔Bob/Rob/Bobby, Richard↔Rick/Dick/Rich, William↔Will/Bill/Billy, James↔Jim/Jimmy/Jamie, John↔Jack/Johnny, Michael↔Mike/Mickey, Christopher↔Chris, Charles↔Charlie/Chuck, Thomas↔Tom/Tommy, Daniel↔Dan/Danny, Joseph↔Joe/Joey, Edward↔Ed/Eddie/Ted/Teddy/Ned, David↔Dave, Steven/Stephen↔Steve/Stevie, Andrew↔Andy/Drew, Matthew↔Matt/Matty, Patrick↔Pat/Patty, Timothy↔Tim/Timmy, Frederick↔Fred/Freddie/Fritz, Theodore↔Ted/Teddy/Theo, Benjamin↔Ben/Benny, Nicholas↔Nick/Nicky, Alexander↔Alex/Al/Xander/Sasha, Samuel↔Sam/Sammy, Lawrence↔Larry, Vincent↔Vince/Vinny, Henry↔Hank/Harry, Raymond↔Ray, Francis↔Frank/Frankie/Fran, Joshua↔Josh, Jacob↔Jake, Ronald↔Ron/Ronny, Eugene↔Gene, Walter↔Walt, Howard↔Howie, Russell↔Russ, Gerald↔Jerry/Gerry, Jeffrey↔Jeff, Bradley↔Brad, Douglas↔Doug, Gregory↔Greg, Bernard↔Bernie, Stanley↔Stan, Norman↔Norm, Philip↔Phil, Peter↔Pete, Wesley↔Wes, Maxwell↔Max, Leonard↔Leo/Lenny/Len, Manuel↔Manny, Dennis↔Denny, Curtis↔Curt, Augustus↔Gus, Zachary↔Zach, Jonathan↔Jon/Jonny, Arthur↔Art/Artie, plus female counterparts: Elizabeth↔Liz/Beth/Betty/Betsy/Lizzy/Eliza, Margaret↔Maggie/Meg/Peggy/Marge/Greta, Katherine/Catherine↔Kate/Katie/Kathy/Cathy/Kat, Patricia↔Pat/Patty/Tricia/Trish, Barbara↔Barb/Barbie, Jennifer↔Jen/Jenn/Jenny, Susan↔Sue/Susie/Suzie, Christine/Christina↔Chris/Tina/Christy/Chrissy, Deborah↔Debbie/Deb, Stephanie↔Steph, Pamela↔Pam, Cynthia↔Cindy, Kimberly↔Kim/Kimmy, Caroline/Carolyn↔Carrie, Sandra↔Sandy, Michelle↔Shelly/Shell, Laura↔Laurie, Sarah↔Sara/Sally, Jessica↔Jess/Jessie, Jacqueline↔Jackie, Theresa/Teresa↔Terri/Tess/Tracy, Virginia↔Ginny/Ginger, Frances↔Fran/Frannie, Rebecca↔Becky/Becca, Rachel↔Rae, Janet/Janice↔Jan, Diana↔Di, Constance↔Connie, Eleanor↔Ellie/Nora, Antonia↔Toni, Charlotte↔Charlie/Char/Lottie, Dorothy↔Dot/Dottie, Florence↔Flo, Genevieve↔Gen, Isabella↔Bella/Izzy, Madeleine/Madeline↔Maddie, Melissa↔Mel/Missy, Nicole↔Nikki, Olivia↔Liv, Penelope↔Penny, Samantha↔Sam/Sammy, Veronica↔Ronnie, Victoria↔Vicky/Tori, Roberta↔Bobbie, Jeanne↔Jeannie/Jeanie, Andrea↔Andie, Alexandra↔Alex/Sasha). Migration `fn_lookup_entity_v4_1_6_nickname_aware` rebuilt the function with a new branch on surgeons/contacts/candidates that fires when query first-token nickname-matches entity first-token AND entity last-token appears in query — scores 0.90 (contacts/surgeons) or 0.85 (candidates). "Tony Matteo" → Anthony Matteo contact id=15 at 0.90 verified, clear 0.15 gap over noise. "tony" alone correctly does NOT false-positive into Anthony (no last-name context = no nickname expansion).
  - **V4.1.7 per-entity nicknames** — John flagged "Boz is also a nickname for Bozentka." That's an entity-specific surname shortening, not a universal first-name pair. Migration `entity_nicknames_v4_1_7` added `nicknames text[] DEFAULT ARRAY[]::text[]` columns to surgeons, contacts, contact_candidates. Bozentka (id=1009) seeded with `{boz}`. fn_lookup_entity rebuilt to score against per-entity nicknames (0.95 surgeons / 0.93 contacts / 0.88 candidates) on top of V4.1.6 universal pairs. "Boz" → David Bozentka at 0.95 verified (clear 0.05 gap over Bozdogan substring match at 0.90).
  - **Ghost candidate dismissed** — `contact_candidates.id=1165` "Tony Matteo" UPDATEd with `dismissed_at=now(), dismissed_by_rep_id=6, dismissed_reason='Duplicate of contacts.id=15 (Anthony Matteo, OR inventory clerk at Methodist). Tony is nickname for Anthony.'` Real curated contact stays as the only Tony.
  - **Q v19 deployed** with three new prompt elements: (1) **SPEAKER vs SUBJECT — READ THE GRAMMAR** section teaching that "X told me he…" / "per X…" / "X said Y" patterns mean X is the SOURCE (contact), and the actual subject is whoever the pronoun/clause refers to; four worked examples including the exact Tony Matteo case. (2) **ROLE-AUTHORITY CHECK** — inventory clerks/MMs/schedulers REPORT, surgeons DECIDE; manufacturer reps speak for company not surgeon adoption; PAs/fellows speak for attending. When role mismatch is clear, ask ONE clarifying question, then submit. (3) **"Trust the lookup"** rule added to TOOL USE — high-confidence lookup hits whose role doesn't fit your inferred subject are usually SOURCE matches, not wrong matches. lookup_entity tool description updated to advertise nickname-awareness and the same warning. Edge Function counter 18→19.
  - **End-to-end verification** — John re-tested "Tony Matteo told me he would buy a Geminus tray" on his phone. Q resolved Tony→Anthony Matteo, asked "which surgeon did Tony mean?", and accepted gracefully when John declined to name one (friction rules held — no blocking).
  - **Known minor residual:** Jamina has a similar contact-vs-candidate dup (contacts.id=2 + candidates.id=2004). Contact wins by 0.05 so not blocking. Logged as a low-priority hygiene sweep.
- 2026-05-24 — Session 16 (END-OF-DAY MEGA-SESSION): **Launch prep + Neo CRM rebrand + V4.1.4-5 + intro overhaul.** Synthetic data wipe (92 test field_notes + 195 derived_facts), anon RLS dropped. V4.1.4 manufacturer + competitor anchoring (86 candidates → mfrs, 184 → competitors, 40 PA/NJ facilities added). V4.1.5 cleanup: cleared 1,030 low-confidence auto-anchors, phonetic boost contacts 0.7→0.8 / candidates 0.65→0.75, 9 email-domain backfills, picker UI unified Contact Type filter, chunked .range() fix for PostgREST 1000-row cap. Neo CRM rebrand: GitHub repo + Vercel project + page title + package.json + UI back-links. Intro modal V4.4.x with Mo Bunnell Give-to-Grow philosophy popup. Geminus playbook gained 2.5/3.5 screw checkbox. Q v17→v18. SW cache v4-22→v4-27.
- 2026-05-24 — Session 15: **V4.1.3 /admin/candidates picker page** for bulk-promoting phonebook candidates. Trainee filter (3 states), set-aside lifecycle (distinct from dismiss), bulk action bar, lifecycle chips, pagination. Migration `contact_candidates_v4_1_3_set_aside` added set_aside_at/by/reason columns + RPCs. Commit `c2b6398`.
- 2026-05-24 — Session 14: **V4.1.2 phonetic match boost + email-domain backfill.** Triggered by "What if I said Jeannie?" Real-world test exposed Jeannie→Jeanne scoring 0.65 below confirm threshold. Migration `fn_lookup_entity_v4_1_2_boost_phonetic`: dmetaphone score boost (contacts 0.7→0.8, candidates 0.65→0.75); 9 NULL-location candidates anchored via single-facility domains; IDN domains deliberately NOT included; Jeanne Romagnole manually anchored to Methodist (id=52). After: "jeannie" + Methodist anchor → Jeanne Romagnole at 0.90, clear 0.10 gap above Jenn McNally at 0.80.
- 2026-05-24 — Session 13: **V4.1.1 SHIPPED — phonebook candidates as a reference-only pool.** New `contact_candidates` table — same lookup surface as contacts, invisible everywhere else. Rows become real contacts ONLY on rep confirmation. RLS read-only for authenticated; writes via promote_contact_candidate + dismiss_contact_candidate RPCs. fn_lookup_entity dropped + recreated to search candidates too (default kinds expanded to all 6, candidate scores capped -0.05 vs contacts). Q v17 gained promote_candidate tool + CONTACTS & CANDIDATES section. Bulk-loaded 2,160/2,205 candidates from John's iPhone vCard.
- 2026-05-24 — Session 12: **V4.1.0 SHIPPED end-to-end — contacts integration.** Added contact_id FK to field_notes + derived_facts. fn_lookup_entity extended to kind='contact' with optional anchor_location_id +0.15 boost. Q v16 with soft-hit confirm flow + create_contact tool + Todoist auto-task. Brain v7 with role-authority weighting. OSO frontend: EntityFactsPanel supports contact, new /contact/[id] page, hospital Contacts tab.
- 2026-05-24 — Session 11: **V4.3.0 + V4.4.0 — 3 new contact-type chips + first-time rep explainer modal.** Meeting/Demo, Tend, Give to Get added (later renamed Give to Grow). Rep-usage-ordered chip rail. First-time intro modal with reopenable "?" button.
- 2026-05-23 — Session 10: **V4.2.0 Field Guide capture — built then reverted same session.** Reference data (surgical preferences, hospital info) belongs in CRM-direct entry, NOT Field Notes chat. Full revert. Insight preserved: intel↔reference distinction is real.
- 2026-05-20 — Session 9: **V4.0.13 → V4.0.19 — adoption polish chain.** Daily-3 celebration, Team tab (owner-only), Q v9 products list, V4.0.16 resume-path fix, V4.0.15 PostgREST relationship disambiguation, V4.0.17 RLS UPDATE/DELETE policies (silent draft-flip bug). New RPC `fn_rep_submission_stats()` admin-guarded.
- 2026-05-17 — Session 9 (earlier): V4.1.0 contacts build paused on discovery of pre-existing 19-row contacts table (Order/PO Automation seed). CRM contacts redesign split into its own project.
- 2026-05-17 — Session 8: **V4.0.9 → V4.0.12 — adoption polish, tag rebuild, contacts scoped, new logo.** Tag taxonomy v3 (Interaction broadened from Surgeon Interaction, Feedback broadened from Case Feedback), smart chips Option A behavior, Cancel button, new pink notebook + blue Q badge logo.
- 2026-05-16 — Session 7: **Capture + Brain + Surfacing all LIVE.** V4.0.4-7 polish. Brain v1 with derived_facts (8 fact_kinds, multi-entity anchoring). pg_cron 5-min sweep. OSO entity pages with EntityFactsPanel. 90 synthetic notes inserted to exercise pipeline.
- 2026-05-15 — Session 5: Whisper locked; chat-as-interface; cost model
- 2026-05-06 — Session 4: V4 conversational capture defined; Q persona introduced
- 2026-04-02 — Sessions 1-3: V1-V3 build (schema, OTP, surgeon autocomplete, photo capture, multi-entity Subject, soft-delete, splash)
