# webook Winter Planner — Riyadh Season 2026-27

A mobile-first, swipe-to-plan prototype for **Riyadh Season 26-27**. The player swipes a deck of
real RS events (right = like, left = skip, down = maybe), marks the nights they're free, and the
"AI" assembles a **winter timeline** of outings. For each outing they invite a circle, get a
resale fallback when it's sold out, register interest for nights that aren't live yet, get matched
with a like-minded webook member when no circle is free, and add a **combo** (event + dinner,
dining + movie). Booking hands the plan to webook with the date/session/tickets prefilled.

Built on the real **RS 26-27 booklet** artwork and webook branding (WBK logo, Gellix type, pink
accent). This is a working front-end prototype for stakeholder demos.

- **Live site:** https://layalhadd-code.github.io/riyadh-season-game/
- **Claude artifact (latest working version):** https://claude.ai/artifact/8PXM85pkwGmoDgRMQrZzMM — **private**, opens only for the owner / people it's shared with.
- **Repo:** https://github.com/layalhadd-code/riyadh-season-game  *(repo still carries the old codename; the app is "webook Winter Planner")*

> 👋 **Handoff for Rawabi.** Start with the two ⚠️ callouts below. The short version: this prototype
> **persists nothing** and the "AI matching" runs on a **synthetic pool**, not webook data — both
> are front-end mocks by design. Where the data and the matching come from decides your whole
> backend approach.

---

## ⚠️ Read first — what's real vs mocked

| Area | In the prototype | For a real pilot |
| --- | --- | --- |
| **Persistence** | None. All state is in-memory JS and **resets on refresh** (circles, swipes, RSVPs, bookings). | Add a backend + webook auth. Nothing is captured today. |
| **AI matching** | Rule-based score over a **synthetic 8-person pool** (`POOL` + `scoreCandidate`). | Cross-check the real webook database (bookings, profile, taste). |
| **Circles / contacts** | Created locally; WhatsApp invites and friends' RSVPs are **simulated**. | Real invites; guess webook accounts from hashed phone numbers (PDPL). |
| **Booking / payment** | "Hand to webook" is a mock cart + payment-link UI; no money moves. | Real prefilled webook deep-links + pay-your-own split. |
| **Resale** | Hard-coded sample listings (`RESALE`). | Live `resell.webook.com` inventory. |
| **Catalogue** | 171 events **auto-parsed from the booklet PDF**; a few OCR-ish names, generic prices. | Data pass against the live bookable RS catalogue. |

---

## Repo contents
- **`index.html`** — the entire app in one file, no build step (~1,350 lines): every screen, the
  flow logic, the 171-event catalogue, the rule-based matching engine, and the base64-embedded
  **Gellix** brand font. This is a **flattened deploy build** — images are referenced at the repo
  root (e.g. `six-kings.jpg`), not under `img/`.
- **`*.jpg`** — ~200 event key visuals, all **1080×1350 (4:5)**, recompressed to ~150–200 KB each:
  - semantic names for the hero events (`six-kings.jpg`, `afc.jpg`, `nye.jpg`, `dolphin.jpg`, …)
  - `rs27-pNNN.jpg` — booklet-page exports for the long tail of attractions.
- **`logo-webook.png`** — the official WBK wordmark used in the header.
- **README.md** — this file.

> The editable **source** (`riyadh-season-game.html`, which uses `img/…` paths and is flattened
> into `index.html` at deploy time) lives in the design workspace, not the repo. If you keep
> iterating, bring the source into the repo and add a one-line build step (see *Deploy*).

## Tech at a glance
- **No framework, no build.** Plain HTML + vanilla JS + CSS in one file.
- **Routing:** screen-based. Each `<section class="screen" data-s="…">` is shown by `go(id)`. No
  URL state. The flow is Swipe → Calendar → Circles → Timeline → Outing → Book.
- **Swipe:** pointer events on the card deck (`deckDown/Move/End`); right/left/down thresholds map
  to like/skip/maybe. Native image-drag and text selection are disabled so a drag is always a swipe.
- **Cards:** the art is a real `<img>` with `object-fit: cover; object-position: top` on a
  fixed **4:5** full-bleed card, so every KV fills cleanly with nothing cut.
- **Fonts:** Gellix (Latin), base64-embedded so the file is self-contained. English only (the KVs
  themselves are bilingual art).
- **No persistence / no runtime capabilities.** Unlike the taste-challenge game, this one does not
  use the Claude `db` — it's pure front-end state.

## How it plays
1. **Swipe** — a deck of **10** events, balanced one-per-category (see *Deck logic*). Pick ≥5 to continue.
2. **My Winter Calendar** — tap the nights you're free (Oct–Feb).
3. **You & your circles** — your webook identity + create circles (friends/contacts), and a
   **"List me for stranger matching"** privacy toggle.
4. **Your winter** — the AI-built timeline (`OUTINGS`) across the months. Tap any plan.
5. **Outing** — depending on state: invite a circle, find **resale** (sold out), **notify me**
   (not live yet), or **match with someone going** (no circle free). Every outing also shows an
   **✨ Make it a combo** add-on. Then **Book** → webook handoff with a pay-your-own tracker.

## Deck logic
`sessionDeck()` builds each session's deck so no two players get the same set and it's never
one-category-heavy: it buckets `CATALOG` by category, shuffles, and round-robins **one card per
category** (then a second pass) until it has **10**, then shuffles the final order. A `decided{}`
set records every swiped card so it never reappears when you change the category filter or tap
"show more options."

## Data model (all in-memory, near the top of `index.html`)
- **`CATALOG`** (171) — every swipeable event: `{id, t, cat, when, venue, price, emo, g, demand, fr}`.
  `emo`/`g` are the emoji + gradient fallback used only if an image is missing.
- **`IMG`** — `id → image filename`.
- **`CATS`** — the filter chips: All, Sports, Concerts, Restaurants, Shopping, Kids, Zones, Shows,
  Experiences, Culture.
- **`OUTINGS`** (10) — the curated AI timeline: `{id, mon, d, t, venue, emo, img, price, state, why, ev?}`.
  `state` ∈ `ready | booked | sold | notify | match` drives the outing screen. `ev` links a
  sold-out outing to its `RESALE` entry.
- **`CIRCLES`** — user-created groups (in-memory; empty at start).
- **`USER`** — the player's taste profile (`picks`, `booked`, `layers`, `vibe`, `budget`, `outings`).
- **`POOL`** (8) + **`WEBOOK_USERS`** — synthetic webook members for the matching demo.
- **`NOTIFY`** — register-interest teasers for not-yet-announced nights.
- **`RESALE`** — sample resale listings per sold-out event.

## Matching engine
`scoreCandidate(c)` scores a pool member against `USER` and `runMatch()` ranks them:
- **35%** shared bookings/picks (Jaccard overlap)
- **30%** shared interest layers (Jaccard)
- **20%** vibe match + shared outing types
- **15%** budget proximity

Candidates ≥ 50 are shown (top 5), with the shared events/layers surfaced as the "why." It's a
transparent rule-based baseline — swap it for a model over real webook data, keeping names and
numbers hidden until **both** sides opt in (the UX already promises this).

## Images
All KVs are **1080×1350 (4:5)** to match the card. They were cropped/extended from the RS 26-27
booklet (provenance is in the original artwork pack's `game-source-map.csv` / `booklet-index.csv`).
The card uses `object-fit: cover` with a top anchor, so logos/titles at the top of each poster stay
visible. A handful of **unused** images from dropped catalogue entries are still committed — harmless,
but can be pruned.

## Deploy
Static site. **GitHub Pages serves the repo root on `main`** — committing `index.html` (+ images)
publishes to the live URL within ~1–2 minutes (hard-refresh to beat the Pages cache). There is no
CI and no build server.

> In the environment this was built in there were **no git credentials**, so every deploy was done
> through GitHub's web **"Add file → Upload files"** UI. If you clone it with normal git access you
> can just `git push`. Consider: (1) renaming the repo to `winter-planner`, (2) moving the editable
> source in and adding a tiny flatten step (`index.html` = source with `img/` stripped + a
> `<!doctype>`+viewport wrapper), so you're not hand-maintaining the deploy build.

## Where things are in `index.html`
Line numbers drift — search by name:
- `CATALOG` / `IMG` / `CATS` — catalogue & images (~670–860).
- `CIRCLES` / `WEBOOK_USERS` / `OUTINGS` / `USER` / `POOL` / `NOTIFY` / `RESALE` — state & reference data (~868–933).
- `sessionDeck()` / `buildDeck()` / `deckEnd()` — the swipe deck (~705+, ~807+).
- `scoreCandidate()` / `runMatch()` — matching (~914–924).
- `renderTimeline()` / `openOuting()` / `renderOuting()` — the plan & outing screens (~1062–1166).
- `addonFor()` / `outingAddonHTML()` — the combo add-on (~1168+).
- `setPay()` / `doBook()` — the webook booking handoff (~1208+).
- `openResale()` / `openMatch()` / `openGoMatch()` — resale & stranger-matching (~1271+).

## ⚠️ What to swap for production
1. **Persistence + auth (first).** Nothing is saved. Add webook sign-in and a store for the
   player's profile, circles, swipes and bookings before anything else.
2. **Matching on real data.** Replace `POOL`/`scoreCandidate` with a cross-check against webook's
   database; enforce privacy (hashed phone matching, mutual opt-in before any contact detail).
3. **Circles, contacts & invites.** Real WhatsApp invites and real RSVPs; guess webook accounts
   from the user's contacts by number.
4. **Booking & payment.** Wire the "hand to webook" screen to real prefilled carts/deep-links and a
   genuine pay-your-own split; the tracker is currently cosmetic.
5. **Resale.** Point the sold-out flow at live `resell.webook.com` inventory.
6. **Catalogue data pass.** The 171 events were auto-generated from the booklet — verify names,
   dates, prices, categories and bookability against the live RS catalogue; drop anything not
   individually bookable (the Boulevard World country pavilions were already removed for that reason).
7. **Assets & fonts.** Confirm KV usage rights and that Gellix's license covers web embedding on
   the production domain.

## Notes
- Single static file + images — deployable to any static host once persistence is wired.
- English only; prices/venues on the booklet-sourced long tail are placeholders ("On webook.com").
- No features beyond the approved flow; all "AI" output is constrained to the catalogue.
