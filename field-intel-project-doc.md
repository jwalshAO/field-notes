# Field Intel Webapp — Living Project Document
Last updated: 2026-04-02 | Session #3

---

## 1. Project Identity
- **What it is:** A mobile-first PWA for Agility Ortho reps to capture field intelligence notes (surgeon visits, competitive intel, case feedback, etc.) that flow into the CRM
- **Goal:** Reps tap an icon on their phone, sign in once with an email code, and quickly enter notes tied to a surgeon from the real CRM database. Notes are stored in Supabase and available for downstream processing.
- **Owner:** John Walsh
- **Key people:** John (owner), Nate/Nick/Noah/Pat (TMs), S3 reps — all 11 team members are in the system
- **Key systems/tools:** Supabase (`oso-tray-tracker` project — the main Agility ops DB), Vercel (hosting), GitHub (`jwalshAO/field-intel-webapp`)
- **Quick start:** To continue this project, say: `"continue Field Intel webapp"`

---

## 2. Current Status
V2 is live on Vercel. V3 updates (session 3) ready to push:
- **Multi-entity Subject field:** Renamed from "Surgeon" to "Subject (optional)." Autocomplete now searches across 4 entity types: Surgeons (649), Hospitals (168), Manufacturers (8), Competitors (14). Each result shows a color-coded badge (Surgeon=green, Hospital=purple, Manufacturer=teal, Competitor=red). Notes can also be submitted with no subject at all — first tag used as card title.
- **Competitors table:** New `competitors` table with 14 seeded entries (Acumed, DePuy Synthes, Stryker, Arthrex, Axogen, Zimmer Biomet, TriMed, Avanti, Smith & Nephew, Integra, Medartis, Biederman, AM Surgical, Globus Medical). Reps can add more via the "Add new surgeon" flow.
- **Splash screen:** Blue gradient splash with app icon, "Field Intel / Agility Ortho" text, and loading spinner. Fades out once auth check completes.
- **New tag: Observation** — Tags now: Surgeon Interaction, Case Feedback, Competitive Intel, Product Interest, Issue / Complaint, Observation.
- **Soft-delete notes:** Reps see ✕ on own notes. Deleted notes vanish from rep feeds but Owner sees them faded with "Deleted by [name]" red tag.
- **Service worker cache bumped to v2.**
- **Auth session persistence confirmed working.**

**V2 features (already live):**
- Supabase Auth email OTP (8-digit code, persistent session)
- Surgeon autocomplete from 649 surgeons in the DB
- Photo upload (camera + gallery) to Supabase Storage
- Next Action field (optional)
- Role-based feed visibility (S3 own, TM territory, Owner all)
- "Field Notes" kraft notebook icon

---

## 3. What Exists (Artifacts & Files)
- `index.html` — Main app (V2: Supabase Auth OTP, surgeon autocomplete, photo upload, next action)
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
- **Surgeon autocomplete from real DB** — Subject field pulls from `surgeons` table. Search by last name, first name, primary hospital. Active/prospect badges. "Add new surgeon" creates real DB records. (Decided session 1, built session 2)
- **Photo upload: camera + gallery** — Optional photo per note. Stored in Supabase Storage. Tap to view full-screen in feed. (Decided session 2)
- **Next Action field** — Optional 2-line freeform textarea, after Notes. Displays as orange badge in feed. (Decided session 2)
- **"Surgeon Interaction" tag** — Renamed from "Surgeon Visit" for broader coverage. (Decided session 2)
- **No separate contacts tab** — Surgeons live in the CRM DB. Field Intel doesn't reinvent that. (Decided session 1)
- **Freeform notes body** — The note text is freeform. Claude parses it downstream. (Decided session 1)
- **Role-based visibility** — S3s see own notes, TMs see territory's notes (via territory_id), Owner sees all. (Decided session 1, implemented session 2)
- **Soft-delete** — Reps/TMs can delete their own notes (✕ button). Owner can delete any note. Deleted notes vanish from non-owner feeds but remain visible to Owner with faded styling + "Deleted by [name]" red tag. (Decided session 3)
- **Multi-entity Subject** — "Surgeon" field renamed to "Subject." Autocomplete searches surgeons, hospitals, manufacturers, and competitors with color-coded type badges. Notes can also have no subject at all (first tag used as title, fallback: "General Note"). (Decided session 3)
- **Competitors table** — 14 seeded competitors in a new `competitors` table. (Decided session 3)
- **"Field Intel" under icon on home screen** — manifest short_name handles this. (Decided session 1)
- **Text-only notebook icon** — "Field Notes" in bold on kraft paper with leather spine. No triquetra/symbol. Edge-to-edge fill, iOS does its own rounding. Generated by Gemini. (Decided session 2)
- **All data consolidated in oso-tray-tracker** — One Supabase project for everything. (Decided session 1)
- **Deploy workflow** — GitHub repo → Vercel auto-deploy on push. `git push origin main` only. (Decided session 1)

---

## 5. Active Work Items
- [ ] Push V3 to GitHub → Vercel auto-deploys (John has the command)
- [ ] Test on phone: splash screen, Observation tag, soft-delete, session persistence across close/reopen
- [ ] Register remaining 7 reps in Supabase Auth (happens automatically on first login)
- [ ] Retire/clean up `iwmilzfdrcuixpitrpkh` project (or just leave it)

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
- 2026-04-02 — Session 3: Renamed "Surgeon" field to "Subject" — now a multi-entity autocomplete searching surgeons (649), hospitals (168), manufacturers (8), and competitors (14) with color-coded type badges. Created `competitors` table and seeded 14 entries. Added `subject_type`, `location_id`, `manufacturer_id`, `competitor_id` columns to `field_notes`. Subject is fully optional — notes without a subject use first tag as card title. Added splash screen (blue gradient, icon, fade-out). Added "Observation" tag. Built soft-delete (reps can delete own notes, Owner sees all with "Deleted by" tag). Bumped SW cache to v2.
