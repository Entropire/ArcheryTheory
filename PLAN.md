# ArcheryTheory – Project Plan

## Goals

- A central, community-improved place for archery knowledge, starting with **tuning for Olympic recurve**.
- Users do everything on the website. This repo is only for development.
- Running costs as close to zero as possible.

## Tech stack

| Part | Choice | Why |
|---|---|---|
| Site framework | **Astro** + TypeScript | Content-heavy site, ships plain HTML by default, fast and free to host |
| Interactive parts | **React** components (Astro islands) | Wizard, setup forms, change-request form |
| Content | **MDX files in this repo** | Pages are edited by hand; git keeps history of every change |
| Hosting | **Cloudflare Pages** (free tier) | Unlimited bandwidth, automatic deploy on push |
| Database + login (phase 2) | **Supabase** (free tier) | Postgres + auth + row-level security, called from the browser, so no server to pay for |
| Spam protection (phase 2) | **Cloudflare Turnstile** (free) | Change requests are allowed without login, so they need bot protection |

> Note: Supabase's free tier pauses a project after ~1 week without activity. Fine during development; once the site has regular visitors it stays awake. Alternative if that becomes a problem: Cloudflare D1 + Workers (also free tier).

### Backend decision (decide before phase 2)
Phase 1 needs no backend, so this can wait. The two realistic options:

| | Supabase (no own backend) | Kotlin + Spring Boot backend |
|---|---|---|
| Cost | Free tier | Free only on Oracle Cloud "Always Free" VM; otherwise ~€5+/month |
| Maintenance | None (managed) | You run and update the server, database and backups |
| Familiarity | New | Already known |
| Logic location | Rules in the database (row-level security) | Rules in your own Kotlin code |

Current leaning: **Supabase**, because the backend logic needed (save setups, submit/moderate tickets) is simple CRUD and it keeps cost and maintenance at zero.

## Content model

Content lives in `src/content/`, separate from code, so editing it never requires touching components.

### Tuning steps (`src/content/steps/*.mdx`)
One file per tuning step (e.g. brace height, nocking point, centershot, plunger, bareshaft). Frontmatter:

```yaml
id: nocking-point
title: Nocking point
discipline: olympic-recurve
summary: Short one-liner
methods:                       # several ways to do the same step
  - id: bow-square
    level: beginner            # beginner | intermediate | pro
    equipment: [bow square]
  - id: bareshaft
    level: intermediate
    equipment: [2-3 unfletched arrows]
sources:                       # proof / research backing the content
  - title: ...
    url: ...
status: draft                  # draft | in-review | published
lastReviewed: 2026-10-20
```

Draft and in-review steps are visible in local development but are left out of the live site, so content can be written and reviewed before it goes live.

The body explains the theory and each method under its own heading (headings get stable anchors, which change requests point at, see below).

### Guides (`src/content/guides/*.yaml`)
The **order** of steps is not hard-coded. Each guide is a small file listing step ids per level:

```yaml
discipline: olympic-recurve
level: beginner
steps: [brace-height, tiller, nocking-point, centershot, plunger]
```

Reordering, adding or removing a step = editing one list. The beginner / intermediate / pro guides reuse the same step pages and show only the methods for that level (and below).

## Saved setups

Three linked record types:

- **Bow setup** – riser, limbs (length, poundage), string (material, strands), brace height, tiller, nocking point, plunger, rest, sight, stabilisers, …
- **Arrow setup** – shaft model, spine, length, point weight, nock, fletching, …
- **Tuning setup / session** – links to **one bow + one arrow**, records the settings and results of a tuning (which method, date, notes, final values).

Storage works the same for everyone through one storage interface:

- **Not logged in** → saved in the browser (IndexedDB/localStorage), with JSON export/import as a backup.
- **Logged in** → saved in the database.
- When a user logs in for the first time, offer to move their browser setups into their account.

## Change requests (tickets)

- Anyone can submit one, **no account needed** (protected with Turnstile).
- A request points at a specific part of a page (page + section anchor) and contains: what should change, why, and optional sources. It is a remark on a part, not a rewrite.
- All requests land in **one moderation queue** (a page on the site, only for moderators).
- Moderators can approve, decline (with a reason) or mark as done. The actual page edit is done by hand in this repo.
- Optional: submitter can leave an email to be notified of the result.

Roles: `user`, `moderator`, `admin` (stored in the database, enforced with row-level security).

## Phases

### Phase 1 – Content & guide (no backend)
1. Project setup: Astro + React + TypeScript, Cloudflare Pages deploy.
2. Layout, navigation, home page, basic styling.
3. Content collections for steps and guides, with schema validation.
4. Step pages: theory, methods per level, required equipment, sources.
5. Guide pages per level + the step-by-step wizard (progress through the steps, pick the method matching your equipment).
6. Bow / arrow / tuning setups saved **in the browser**, with export/import.
7. Olympic recurve content (written by Quinten from online research, see *Content workflow*).

### Content workflow (phase 1)
1. Write each step as an MDX file with `status: draft`, including the sources used.
2. Run the site locally (or a preview deploy) and share it with reviewers.
3. Process their feedback, set `status: published`.
4. Launch when the beginner guide is complete and reviewed.

### Phase 2 – Users & tickets
1. Supabase project: database tables, row-level security, auth.
2. Login / register.
3. Setups saved to the database when logged in + migration from the browser.
4. Change-request form on every section + moderation queue.
5. Turnstile spam protection.

### Later
- Other disciplines (compound, barebow, traditional, …).
- Multiple languages (English only for now).
- Link tuning sessions to the guide (start a session from the wizard, results saved automatically).

## Open questions

- Which login methods: email + password, magic link, Google, …? (phase 2)
- Supabase or Kotlin backend? (phase 2, see *Backend decision*)

## Decided

- Content is written by Quinten from online research and reviewed by people Quinten knows before launch.
- English only for now.
- Domain: **archerytheory.com**.
- Pages are edited by hand; change requests only point out what may need changing.
- No account needed to save setups or to submit change requests.
