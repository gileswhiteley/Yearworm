# Yearworm — Roadmap

Plain-English notes on where the game goes after v1. Nothing here is built yet.
This is the plan, captured so it isn't lost.

---

## Where v1 is now

- Single static page, hosted on Netlify, no backend.
- Two play modes via a top-bar toggle:
  - **Solo** — plays a ~30s Spotify snippet on the same device.
  - **QR** — scan with a second device to play on Spotify.
- Deck = UK #1 singles, matched to Spotify track IDs by the `hitster-build` pipeline.
- One flat `songs.json` is the whole deck.

---

## Gameplay — scoring & multiplayer

This is what turns Yearworm from a solo "guess for yourself" toy into a proper
**party game**. Hot-seat / pass-and-play on one device (fits Solo mode especially).

### The flow
1. **Setup** — ask how many players; optional name entry per player (default
   "Player 1", "Player 2"…). Set a **target score to win** (e.g. 10 / 20 / 30 — the
   user picks, it doesn't matter which).
2. **Turn loop** — it's Player 1's turn:
   - play the snippet → player guesses out loud → **reveal the year**
   - then ask **"Did Player 1 get it right?"**  →  **Yes = +1 point, No = nothing**
   - advance to Player 2, then 3… and cycle back round.
3. **Win** — first player to reach the target score wins; show a winner screen.

### Notes & open decisions
- **Adjudication is honour-based / host-judged** — the game asks Yes/No, the players
  decide. Simplest and matches the described flow. (Avoids the app having to define
  "correct" or do any string/date checking.)
- **Open: what counts as "right"?** Exact year? Within a year or two? Within the
  decade? This is a house-rules call — could leave it entirely to players, or offer
  a difficulty setting. Decide later; doesn't block v1.
- **Scoring is +1 per correct** for v1. A Hitster-style variable/placement scoring
  could come later, but keep it dead simple first.
- **Save game-in-progress to localStorage** so a refresh or accidental close doesn't
  wipe scores mid-game.
- **UI needs:** whose-turn indicator, a running scoreboard, the Yes/No prompt after
  reveal, and a winner screen.
- **Desktop layout (TODO):** the whole game is currently tuned for phones (where it'll
  mostly be played). On a desktop/Mac it looks lost — a tiny phone-width column floating
  in a sea of black. Needs a proper desktop treatment that fills the viewport: bigger
  type, the scoreboard/board using the width, not just centred phone UI scaled up.
- **Works in both modes** — the Yes/No adjudication slots in *after* the existing
  reveal step (Solo's Reveal button / QR's tap-to-reveal).
- A "just playing solo, no scoring" option should remain — scoring is opt-in at setup.

---

## The vision: from one game to a platform

Yearworm becomes a catalogue of **packs**, not a single fixed deck.

### Tiers (digital only — no physical/paper packs, ever)

| Tier | Content | Price |
|------|---------|-------|
| **Free taster** | Number Ones | Free |
| **Tier 1 (paid)** | Top Tens | Paid |
| **Tier 2 (paid)** | Genre packs (heavy metal, etc.) | Paid |

- **Number Ones is the free taster** — generous enough to hook people, then the
  paid tiers are the upsell.
- **Top Tens is the first thing you pay for**, not free.
- **Genre packs are the second paid tier.**

> Open decision: do the tiers stack (Tier 2 includes Tier 1), or are they
> independent unlocks you buy separately? Leaning toward stacking / "all-access"
> being simplest to explain, but this is a product call to make before building.

### Free vs paid — feature gates (decided; stubbed in the scoring prototype)
Separate from *content* packs, some *features* are free-tier limited. These are
wired into `scoring.html` now as UI stubs (no real entitlement check yet — that
arrives with the paywall backend in the Stripe/Netlify-Functions stage):

- **Players: free = 2.** More than two players is a **Tier 1** unlock. Setup would cap
  the counter at 2 and show a "✦ More than 2 players is a Tier 1 unlock" note when
  you try to add a third. (Scoreboard rendering handles up to 8 — wraps to a second
  row at 4+.) **Currently UNLOCKED in the Alpha build** via the `ALPHA = true` flag in
  `scoring.html` (full house up to 8 for testing); flip `ALPHA` to re-enable the gate.
- **Era selection: Tier 1.** Choosing *which decades* are in the deck (multi-select,
  e.g. exclude 50s–70s) is a Tier 1 unlock. **Free tier = all years only.** Made
  fully selectable for now (free), with a "✦ Tier 1 unlock — free for now" note, so
  it can be demoed before the paywall lands.
- **Solo practice: limited then sign-up.** "Just practising — play solo" is capped at
  `PRACTICE_LIMIT` goes (currently 10), then a sign-up gate appears ("That's your free
  round!"). The Sign-up button is a stub until accounts exist.

### Single-device focus (pivot away from the Hitster-clone look)
The app is becoming **one-device** (phone / tablet / PC), not a scan-to-a-2nd-device
game. The **Solo/QR toggle has been removed from the UI** in the scoring prototype to
lean into this and stop looking like a Hitster copy. The QR code itself
(`renderQR()` / `spotifyUrl()`, the `mode==='qr'` path) is **kept in the file** — not
deleted — in case a second-device mode is worth resurrecting later.

### Maybe later (not committed)
- **Bespoke packs as a service** — run the build pipeline on a customer's song
  list and hand them a private pack. The tooling is ~80% there already.

---

## What this changes technically

### 1. Pack architecture (the one cheap-now decision)
Stop shipping one flat `songs.json`. Move to a **manifest + per-pack files**:

```
packs/
  number-ones.json   (free, ships in the bundle)
  top-tens.json      (paid, server-gated)
  heavy-metal.json   (paid, server-gated)
```

The app loads a manifest, shows a **pack picker**, and builds the deck from
whichever packs the player has unlocked. Small refactor now; painful later once
there are paying customers and saved state. **Do this before the next big data push.**

The existing `hitster-build` pipeline (`uk_number_ones → matched → songs.json`)
generalises to this almost for free — same scripts, different source lists.

### Data storage: JSON at runtime, SQLite in the build pipeline
As the catalogue grows into thousands of tracks across many packs, the question
came up: move to SQLite? Answer is a **split**, because there are two data layers:

- **Runtime (what the browser loads): keep JSON per-pack files.** The app is a
  static page with no backend, so there's no SQL server to query; the only
  client-side option is shipping a ~1MB WASM SQLite engine — pure overhead for tiny
  data. Even the *whole* eventual catalogue (#1s + Top Tens + genres ≈ 10–15k tracks)
  is ~1MB of JSON that gzips small and parses in milliseconds, and the pack model
  only ever loads the unlocked packs. SQLite earns its keep at millions of rows /
  live queries — neither of which a deck game has.

- **Build pipeline (scrape → match → dedup): SQLite is genuinely worth it.** This
  side is becoming relational — multiple chart sources, match metadata (Spotify ID,
  confidence, MusicBrainz ID, ISRC), and the dedup-by-track-ID rule. A single
  `build.sqlite` (tables: songs, chart_entries, matches, packs) turns ad-hoc
  multi-JSON merging into clean queries.

**Pattern:** SQLite as the build-time source of truth → **export per-pack JSON as the
published artifact**. The app never changes; the pipeline gets the relational power.

**When:** not now. Do it as part of the **Top-Tens matching stage**, which is exactly
when the relational pain (dedup, MusicBrainz IDs, match tracking) actually shows up.
Migrating now would be premature.

### 2. The paywall (now mandatory, because paid content is digital-only)
A static site can't truly hide content client-side, so paid packs need a thin backend:

- **Front-end stays static on Netlify** — the game barely changes.
- **Paid pack files do NOT ship in the bundle.** They live server-side and are
  only returned after an entitlement check. (The free Number Ones pack can stay bundled.)
- **Netlify Functions** (serverless, native to Netlify — no server to run) do the gating.
- **Stripe** handles payment + subscription/purchase state.

**Honest caveat:** no digital paywall is unbreakable — a paying user could in
theory extract a pack's track IDs. That's fine. The goal is *enough friction that
honest people pay*, not Fort Knox. Every digital game lives with this.

### 3. Payment model — decide before building
| Model | Feels like | Best when |
|-------|-----------|-----------|
| Subscription (all packs + new ones) | Netflix | Packs added regularly |
| One-time per pack | App Store / Hitster | Packs are distinct things people pick |

Can do both (subscription = everything, or buy packs à la carte). This fork drives
the auth model, Stripe setup, and UI — so land it first.

---

## Top Tens — data sourcing & matching pipeline

The first paid pack. Bigger and messier than the #1s, so the plan splits into
three separate questions, each with its own best source.

### Definition
"Top tens" = **every unique song that *peaked* in the UK Top 10 (positions 1–10),
tagged with the year it did** — each song once, not every weekly Top-10 slot
(that's repetitive and not what a deck wants).

### The three questions, three sources
| Question | Best source |
|---|---|
| *Which songs were UK Top 10, and what year?* | Wikipedia year-lists / Official Charts Company |
| *What's the canonical identity of this song?* | **MusicBrainz** |
| *What's its Spotify track ID (for playback)?* | Spotify — seeded by MusicBrainz, not blind search |

### 1. The chart list (which songs were Top 10) — DONE
- **Source used:** Wikipedia's per-*year* series, "List of UK top-ten singles in YYYY".
  Scraped by `hitster-build/scrape_toptens.py` -> `uk_top_tens.json`.
- **Result: 4,281 unique Top-10 songs, full coverage 1952–2025** (all 74 years
  parsed non-zero — the early decades came through fine, so **no Official Charts
  backfill was needed** after all).
- Per-decade: 50s 264 · 60s 453 · 70s 583 · 80s 679 · 90s 788 · 00s 790 · 10s 522
  · 20s 202. (Recent years dip because streaming-era charts are "stickier" — fewer
  distinct songs break the Top 10.)
- Data is clean: short titles like *19*, *Go*, *Oh* are real songs; repeated titles
  (*Do They Know It's Christmas?* ×11) are genuinely different recordings/years.
- **Backfill source, unused but on record:** Official Charts Company
  (officialcharts.com) — definitive, every weekly chart back to 1952, but no public
  API and ToS-grey. Only needed if a future gap appears.

### 2. Identity resolution (MusicBrainz — the new layer)
MusicBrainz is **not a chart database** — it does not store chart positions, so it
can't tell us *which* songs were Top 10. But it's excellent at *identity*, which
is exactly the stage that currently hurts:
- Carries **ISRCs** (unique global recording codes) that map to a Spotify track far
  more reliably than fuzzy title+artist search.
- Often stores a **direct Spotify link** as a relationship.
- The **full DB is downloadable**, so this stage runs **locally with no rate limits**
  — sidestepping the Spotify lockout dance for identity resolution.
- Cost to note: the full dump is large (tens of GB, needs local Postgres); the live
  API is gentler but capped ~1 req/sec. At ~6,000+ songs the local dump is likely
  worth it precisely because it removes rate limits from matching.

### 3. Spotify ID (for playback)
Still Spotify (playback needs a Spotify track), **but seeded by the MusicBrainz
ISRC / Spotify-link instead of blind search** — more accurate, and far less exposed
to Spotify's rate-limiting, which is our real bottleneck.

### Free vs paid separation — dedup by Spotify ID, NOT by string
Every #1 is also a Top-10 song, so the lists overlap. A naive title+artist match
only found 411 of 1,292 #1s inside the Top-10 list — misleading, caused by artist
strings differing between scrapes ("Elvis Presley" vs "Elvis Presley with The
Jordanaires"). **Rule: do the free-#1s vs paid-Top-Tens split AFTER matching, by
Spotify track ID**, where the same recording resolves to the same ID regardless of
how the artist was written. Don't string-wrangle the lists beforehand.

### Scale & expectations
- **4,281 unique Top-10 songs** captured (the chart list is done). The genuinely-new
  paid content is whatever remains after removing the free #1s *by track ID* post-match.
- Matching is still a **phased, multi-day background job** (Spotify rate limits) — but
  the MusicBrainz layer should cut both the error rate and the rate-limit pain.

### First concrete step when this picks up
Quick coverage check: do the Wikipedia year-pages actually span 1952–2025, so we
know up front how much Official-Charts backfill the old years need.

---

## Explicitly OUT of scope
- **Physical / paper expansion packs.** Hard no — printing, fulfilment, storage,
  all a pain. Digital only.

---

## Suggested order when this picks up
1. Refactor to manifest + per-pack files (unblocks everything, no paywall yet).
2. Build the Top Tens pack data (bigger source list; Spotify rate limit is the bottleneck, not code).
3. Decide payment model (subscription vs per-pack).
4. Add Stripe + Netlify Functions to gate paid packs.
5. Genre packs (curated lists — editorial work more than technical).
6. (Maybe) bespoke-pack service.
