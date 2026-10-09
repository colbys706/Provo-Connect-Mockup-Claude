# Momentum

Less scrolling, more living. A phone-first web app that helps college students in Provo build better phone habits and replace scrolling with real, in-person activities.

Plain **HTML + CSS + JavaScript**. No frameworks, no build step.

```
momentum/
├── README.md               ← you are here
├── database/
│   ├── schema.sql          ← run in Supabase (24 tables, rules, demo data)
│   └── ERD-GUIDE.md        ← Supabase setup + Lucidchart ERD steps
└── site/                   ← the website (upload this folder to host it)
    ├── index.html          ← sign in / sign up
    ├── home.html  streak.html  activities.html  activity.html  activity-new.html
    ├── calendar.html  lessons.html  lesson.html  social.html  friends.html
    ├── profile.html  settings.html
    ├── css/styles.css      ← ALL styling (colors at the top)
    └── js/
        ├── config.js       ← paste Supabase URL + anon key here
        ├── store.js        ← ALL data: habits, streaks, points, RSVPs, posts, messages
        ├── ui.js           ← shared header, nav, messages panel, pop-ups
        ├── lessons-content.js ← lesson text (edit lessons here)
        ├── pages/*.js      ← one file per page
        └── vendor/         ← Supabase's library (don't edit)
```

## Run it

- **On a laptop:** open `site/index.html` in Chrome. Done.
- **On a phone, or to share with the team:** host it (see below). Android won't run local multi-file sites, but a hosted link works everywhere.

## The vertical slice: Post an activity → Supabase

1. Set up the database: follow `database/ERD-GUIDE.md` step 1 (run `schema.sql`).
2. In Supabase, open **Project Settings → API** (or "API Keys"). Copy the **Project URL** and the **anon / publishable** key.
3. Paste both into `site/js/config.js`.
4. Reload. The Post-an-activity page now says **"Saved to: Supabase"**.

**Demo script:**
1. Post an activity on your phone.
2. Open Supabase → Table Editor → `activities`. The row is there.
3. A teammate opens Activities on *their* phone and sees it.
4. Show that `activity_addresses` has the exact address, but the public feed can't read it (PL-2, enforced by the database).

> ⚠️ Only the **anon/publishable** key goes in `config.js`. Never the `service_role`/secret key.

## What's saved where (right now)

| Feature | Saved in |
|---|---|
| Activities (list, details, posting) | **Supabase** once keys are in `config.js`, otherwise this browser |
| Everything else: habits, streaks, points, RSVPs, +1s, posts, friends, messages, lessons, settings | This browser (`localStorage`) |

Every feature goes through `js/store.js`, shaped like the tables in `schema.sql`, so each one can move to Supabase in a later slice without touching the pages. **Next slices, in a sensible order:** login (Supabase Auth) → RSVPs → habits/points → posts → messages.

Settings → **Reset demo data** restores the demo habits, friends, RSVPs, and messages in your browser. The demo activity dates are set relative to the day you first open the site, so reset if they've drifted into the past.

## Host it free (GitHub Pages)

1. Create a GitHub repo and upload the **contents of `site/`** (so `index.html` is at the top level).
2. Repo → **Settings → Pages** → Source: *Deploy from a branch* → `main` / root → Save.
3. In about a minute you get a link like `https://yourname.github.io/momentum/`. It works on any phone.

(Netlify works too: drag the `site` folder onto app.netlify.com/drop.)

## Feedback items → where they live

| Feedback | Where |
|---|---|
| Title goes home; profile always in nav | `ui.js` (header + nav, shared by every page) |
| Always-visible nav: Home, Activities, Calendar, Lessons, Social (+ profile avatar) | `ui.js`: bottom bar on phones, sidebar on desktop |
| "Share a post" on profile; "Post an activity" on Home | `profile.js`, `home.js` |
| Message host / message poster (activity chats skip the request step) | `activity.js`, `social.js`, `ui.js` (messages panel) |
| +1: friend with account (must accept, doesn't hold a spot) or guest first name | `activity.js` commit pop-up, `store.js` `commit()` |
| Who's going: friends = full profile, others = first name, or hidden by setting | `activity.js`, Settings → "On Who's going lists" |
| One points ledger for habits, lessons, RSVPs, attendance, posts; updates instantly, even across tabs | `store.js` (`addPoints`, `stats`, `streak`), `streak.html` |
| Streak front and center → stats page | `home.js`, `streak.js` |
| Commit pop-up says commitment counts; after commit → calendar + add-to-calendar | `activity.js`, `calendar.js`, `ui.js` (Google / Apple / Outlook) |
| Cancel: 2 per month, not within 24 hours, loses the RSVP points | `store.js` `cancel()`; the database enforces it too |
| Lessons as a Duolingo-style path; dedicated lesson pages; 30-word goal → daily habit | `lessons.js`, `lesson.js`, `lessons-content.js` |
| From Momentum: Encourage, activity-linked posts, "met at", lesson stat cards, frosted look | `social.js`, `friends.js`, `lesson.js`, `styles.css` |

## Before you present

- **Lesson stat numbers are placeholders** and are labeled that way. Find real sources.
- **The PRD says RSVPs can't be cancelled (PL-10)**, but the feedback added 2 cancellations a month. Update the PRD to match.
- **The PRD says posts show oldest first (SN-2)**; the app shows newest first, per the sprint prompt. Update one of them.
