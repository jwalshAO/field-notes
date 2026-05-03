# Field Intel Webapp — Living Project Document
Last updated: 2026-04-02 | Session #3

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
V3 ready to push (NOT YET DEPLOYED). All DB migrations are live. Full feature set:

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
- `index.html` — Main app (V3: multi-entity Subject, splash screen, soft-delete, streamlined form)
- `manifest.json` — PWA manifest (short_name: "Field Intel")
- `sw.js` — Service worker for offline/caching
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

## 5. Active Work Items
- [ ] Push V3 to GitHub → Vercel auto-deploys (John has the command — see below)
- [ ] Test on phone: swipe-up splash, multi-entity Subject, camera button, soft-delete, session persistence
- [ ] Register remaining 7 reps in Supabase Auth (auto on first login)
- [ ] Decide: downstream processing — how/when does Claude parse field notes into CRM actions?
- [ ] Retire/clean up `iwmilzfdrcuixpitrpkh` project (or just leave it)

**Deploy command:**
```
cd /Users/johnwalsh/Codex/"Field Intel Webapp" && git add index.html sw.js field-intel-project-doc.md && git commit -m "V3: multi-entity Subject, splash swipe-up, soft-delete, cleaner form" && git push origin main
```

---

## 6. Open Questions & Blockers
- **"Add new surgeon" flow** — Currently requires first name + last name. Hospital is optional. May want to add more fields later.
- **Downstream processing** — How/when does Claude parse field notes into structured CRM actions? Not yet defined.
- **oso-tray-tracker naming** — John wants to rename this project eventually. No action needed now.

---

## 7. Constraints & Preferences
- Light/white backgrounds for all HTML — never dark themes
- Tags should be small/unobtrusive (11px)
- Terminal commands in one shot, not multiple
- All files saved to Codex (`/Users/johnwalsh/Codex`)
- Git push workflow only — no `gh` or `vercel` CLI
- `oso-tray-tracker` project ID: `pchhtltxdcmvdcwnwaeg`
- GitHub repo: `jwalshAO/field-intel-webapp`
- Vercel URL: https://field-intel-webapp.vercel.app

---

## 8. Background Context
- Agility Ortho is an upper extremity orthopedic distribution company (Eastern PA, South NJ, Northern DE)
- 11 team members: John (owner), 4 TMs (Nate, Nick, Noah, Pat), 6 S3 reps
- The `oso-tray-tracker` Supabase project is the single operational database — surgeons, locations, reps, trays, sales, revenue
- The `reps` table has all 11 people with emails, roles, territory_ids
- `surgeons` table has rich data: names, NPIs, specialties, status, primary_hospital, funnel_stage
- 4 of 11 reps have Supabase Auth accounts; the rest will auto-create on first OTP login
- The Field Intel app's job is pure INPUT — make it dead simple for reps to capture intel. The CRM handles the rest.

---

## 9. History Log
- 2026-04-02 — Session 1: Designed and built V1 of Field Intel webapp. Created Supabase schema (wrong project initially), built PWA frontend, deployed to Vercel via GitHub. Discovered surgeon DB exists in oso-tray-tracker. Decided to consolidate there and rebuild Subject field as surgeon autocomplete. Custom notebook icon finalized. Multiple UI tweaks (tags, placeholder). Changes staged locally but NOT yet pushed to GitHub.
- 2026-04-02 — Session 2: Major V2 rewrite. Switched to correct Supabase project (oso-tray-tracker). Created `field_notes` table with FK links to reps and surgeons. Added RLS policies. Implemented Supabase Auth email OTP (persistent sessions). Built surgeon autocomplete from 649-row surgeons table. Added photo capture/upload (camera + gallery) with Supabase Storage bucket. Added "Next Action" freeform field. Renamed "Surgeon Visit" tag to "Surgeon Interaction". Removed Contacts tab. Iterated on icon 4 times with Gemini — landed on text-only "Field Notes" on kraft notebook. All changes ready to push but NOT yet deployed.
- 2026-04-02 — Session 3: Big feature + UX session. (1) Multi-entity Subject: renamed "Surgeon" to "Subject", autocomplete now searches surgeons/hospitals/manufacturers/competitors with color-coded badges. Created `competitors` table (14 seeded). Added `subject_type`, `location_id`, `manufacturer_id`, `competitor_id` columns. (2) Splash screen: swipe-up-to-dismiss with motivational blurb and subtle bouncing chevron. (3) Soft-delete: reps can ✕ their notes, Owner sees all with "Deleted by [name]" red tag. Added `deleted_at`, `deleted_by`, `deleted_by_rep_id` columns. (4) Form UX cleanup: date + camera button share one row (no labels), Subject label dropped "(optional)", removed "+ Add new surgeon" from dropdown (just freeform), photo section replaced with inline camera icon, Next Action expanded to 3 rows with new placeholder, Notes placeholder updated. (5) Added "Observation" tag. (6) Bumped SW cache to v2. All DB migrations applied. Code ready to push but NOT yet deployed.
