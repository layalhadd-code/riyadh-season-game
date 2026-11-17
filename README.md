# webook Winter Planner

A mobile-first clickable prototype. A member says what they are into and when they are free, and the
planner builds a winter plan (Nov to Feb) from events they can actually book on webook: Riyadh
Season 26-27 plus other live webook listings. They invite friends with a WhatsApp link, friends
vote on the link, and the group books together.

This is a front-end prototype for stakeholder demos and a reference for the real build. Version
**V57**, 17 Nov 2026.

- **Live site:** https://layalhadd-code.github.io/riyadh-season-game/
- **Claude artifact (working copy):** https://claude.ai/artifact/8PXM85pkwGmoDgRMQrZzMM (private: opens only for the owner and people it is shared with)
- **Repo:** https://github.com/layalhadd-code/riyadh-season-game (the repo keeps its old codename)
- **Screen map:** an 11-page printable PDF of every screen for both users. Ask Layal for it.

> **Handoff for Rawabi.** Read "What is real and what is a stand-in" first. Nothing is saved, and
> every other person in the demo is simulated except the two switchable members. The section
> "How people are connected" is the part that decides the backend.

---

## The idea in one paragraph

Nobody is "the first user". Every member has **their own plan** and **the plans friends invited
them to**, on one screen. Friends are connected by the share link, not by groups: there are no
circles for friends. A circle exists in one place only, a gathering of new friends for a single
event.

## The journey (the same for everyone)

1. **Cover.** Animated poster wall and four steps that explain the planner. If the member arrived
   from a friend's link and has no plan yet, the cover says who invited them instead.
2. **Interests** (step 1). Pick 3 to 5 of 17 interests, grouped in shelves. City filter and search.
   Shows the webook bookings the planner already knows about.
3. **Swipe** (step 2). Only bookable events from the chosen interests. Right is like, left is skip,
   down is maybe, or use the three buttons. A two-second tutorial overlay shows on the first visit.
4. **My Winter Calendar** (step 3). Free days of the week and best times of day.
5. **Winter plans.** "Your plan" first, then every plan a friend invited you to. One opens at a time.
   - Your plan: outings on real dates, each with the reason it was picked, plus friends' responses.
   - A friend's plan: I'm in / Maybe / Can't on each outing.
6. **Invite friends.** One link, sent to as many WhatsApp groups as the member likes.
7. **Outing.** Who is in, change date, cancel, add a second event as a combo, find new friends.
8. **Checkout.** The outing and anything added to it are in one cart. Pay your own seat (everyone
   who voted in gets their own payment link, with a tracker) or pay for all.
9. **Sold out.** The same Book button offers resale listings.
10. **New friends.** Members already going to the event, with first name, age, gender, interests
    and a match score. Invite several and it becomes a gathering, kept together as a circle.

## What is real and what is a stand-in

| Area | In the prototype | For production |
| --- | --- | --- |
| **Persistence** | None. All state is in-memory JavaScript and resets on refresh. | webook sign-in and a store for profile, swipes, plans, invites, votes. |
| **Catalogue** | 62 events (37 Riyadh Season, 25 other webook listings) taken from webook.com on 16 Nov 2026, with their official posters. | Live webook catalogue and availability. |
| **Booking history** | Sample data (`HISTORY`). | The member's real webook orders. |
| **Members** | Two switchable demo members, Omar (returning) and Sultan (new, invited by Omar). They are fully live against each other. | Real accounts. |
| **Other friends** | Farrukh's and Yousef's plans are generated samples. Other friends' votes are simulated after a share. | Real invitees and votes. |
| **WhatsApp** | A mock of WhatsApp's picker. | A normal share link. webook cannot read groups or replies, and cannot create a WhatsApp poll. |
| **New friends** | An 8-person synthetic pool (`POOL`), rule-based score, simulated acceptances. | Matching over real members, with consent. |
| **Booking and payment** | Mock cart, payment links and tracker. No money moves. | Prefilled webook carts and real per-person links. |
| **Resale** | Hard-coded sample listings (`RESALE`). 6 Kings Slam is marked sold out on purpose to demo this path. | Live resell.webook.com inventory. |

## How people are connected

- The plan owner shares a link (`webook.com/w/<name>-winter`, or `/…/<eventId>` for one outing).
- Opening the link while signed in records "this member was invited to this plan". That record is
  what fills the invited member's plans list. No friend list, no groups.
- Each vote is stored per plan, outing and member, and shows on the owner's plan.
- If the invited member builds and shares their own plan, it appears in the first member's list the
  same way.

So the backend needs three things: a share token per plan, an invite record (plan, member), and a
vote record (plan, outing, member, answer).

## How the plan is built

`tasteProfile()` turns signals into a weight per interest tag:

- interest picked: +3
- each webook booking: +2
- swipe: like +2, maybe +1, skip −1.5

`buildPlan()` scores every bookable event (`tasteScore` = sum of weights ÷ √number of tags), keeps
what the member already booked, what they liked, strong maybes and a few unswiped picks that
score well, then places each on a real date the event runs. It prefers the member's free days,
spreads open-run events out, and avoids two outings on one day while the event has another date.
Each outing carries a plain-language reason (`why`).

## Rules the catalogue follows

- Only events a member can book on webook on their own, each with its own official poster.
- Real dates only. Fixed-date events use `on:[…]`; open runs use `run:[from,to]`.
- No invented prices or venues. Where webook shows no price the text is "On webook.com".
- An event with tickets on webook or on resale is shown the same way. An event with no tickets
  anywhere is left out.
- Interests are explicit tags on each event, not guessed from titles.

## Repo contents

- **`index.html`** is the whole app in one file with no build step (about 1,870 lines): markup,
  styles, logic, catalogue and the base64-embedded Gellix font. It is a flattened deploy copy:
  images are referenced at the repo root, not under `img/`.
- **`rs-*.jpg`, `wb-*.jpg`, `wbk-*.jpg`** are the posters the app uses now (square, from webook).
- **`logo-webook.png`** is the header wordmark.
- About 200 older images (`rs27-pNNN.jpg` and others) are from the earlier booklet-based version.
  A few are still used by the "not announced yet" screen; the rest can be pruned.

The editable source uses `img/…` paths and lives in the Claude artifact, not in this repo. If you
keep iterating, bring the source in and add a one-line flatten step.

## Tech at a glance

- Plain HTML, CSS and vanilla JavaScript. No framework, no build.
- Screens are `<section class="screen" data-s="…">`, shown by `go(id)`. No URL state.
- Styles follow webook's own look: zinc greys on near-black, pink as the single accent, Gellix at
  weights 400 and 600, line icons from an inline SVG sprite.
- Animations respect `prefers-reduced-motion`.
- English only.

## Data model (all in memory)

- **`CATALOG`** (62): `{id, t, cat, tags, venue, city, on | run, tm, p, price, emo, img, demand, url}`.
- **`INTERESTS`** (17) and **`IGROUPS`**: the interests and their shelves.
- **`PROFILES`**: per-member state for the two demo members. `saveMe()` / `loadMe()` swap it in
  and out of the globals; `seeAsFriend()` switches member.
- Per member: `SELECTED`, `SWIPES`, `HISTORY`, `freeDays`, `bestTimes`, `OUTINGS`, `PLAN_OVR`
  (moved dates and cancellations), `VOTES`, `SHARE`, `COMBO`, `CIRCLES`, `FPV` (my answers on
  sample friends' plans).
- **`FRIEND_PLANS`**: the two sample friends' plans, generated by `friendPlanItems()`.
- **`POOL`**: the synthetic members for new-friend matching. **`RESALE`**, **`NOTIFY`**: sample data.

## Where things are in `index.html`

Search by name; line numbers drift.

- Catalogue and dates: `CATALOG`, `eventDates`, `whenText`, `available`.
- Taste and deck: `tasteProfile`, `tasteScore`, `sessionDeck`, `buildDeck`, `doSwipeCard`, `moreOptions`.
- Plan: `buildPlan`, `rebuildPlan`, `altDates`, `setOutingDate`, `doCancel`.
- Members and invites: `freshProfile`, `saveMe`, `loadMe`, `seeAsFriend`, `invites`, `planItems`, `setVote`.
- Plans screen: `renderTimeline`, `myPlanHtml`, `friendPlanHtml`, `respCardHtml`, `toggleHub`.
- Sharing: `openShare`, `renderShare`, `sendWa`, `simulateVotes` (demo only).
- Outing and checkout: `renderOuting`, `addonFor`, `addAddon`, `renderBook`, `setPay`, `doBook`, `renderTracker`.
- New friends and circle: `scoreCandidate`, `matchList`, `renderGoMatch`, `createCircle`, `renderCircle`, `renderNfProfile`.
- Cover and tutorial: `renderCover`, `showSwipeTut`.

## New-friend matching

`scoreCandidate(c)` scores a pool member against the current member: shared bookings and likes 35%,
shared interests 30%, taste answers 20%, budget 15%. The five closest are listed. It is a
transparent rule-based baseline to be replaced by matching over real webook data.

## Deploy

Static site. GitHub Pages serves the repo root on `main`; committing `index.html` publishes within
a minute or two. There is no CI. Deploys so far were done through GitHub's web "Upload files" page
because the build environment had no git credentials. With normal git access you can `git push`.

## What to build for production

1. **Sign-in and persistence.** Nothing is saved today.
2. **Plans, invites and votes** as described in "How people are connected".
3. **Live catalogue** with availability, dates and prices, including resale.
4. **Real booking history** as the taste signal, and age and gender from the account for the
   new-friend profile.
5. **Booking and payment**: prefilled carts, per-person payment links, payment status.
6. **New-friend matching** on real members, with consent, verification and reporting.

## Decisions still open

- Should voting on a shared plan require a webook sign-in? Viewing does not.
- Should a plan owner be able to remove someone who opened a forwarded link?
- New friends: is showing first name, age and gender to matched members acceptable, and should the
  Women / Men filter default to the member's own gender?
- Does the Gellix licence cover web embedding on the production domain, and are the posters cleared
  for this use?
