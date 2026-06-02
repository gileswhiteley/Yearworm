# Yearworm — Product Summary

Plain-English overview of what Yearworm is, how it plays, and why it works.
Written so anyone can understand the game in two minutes — and so it can be
handed to a marketing agent as a brief.

Live at **yearworm.netlify.app**.

---

## The one-liner

**Hear a song, guess the year it was a hit.** Yearworm plays a 30-second snippet
of a famous track and you place it on the timeline — was it 1977, 1991, 2008?
Get the year bang-on for full marks, land one year either side for a point, miss
and you score nothing. First to the finish wins.

## The 10-second pitch

A music guessing game you can start in one tap, with no app to download, no
login, and no cards to buy. Pass one phone around the table, or play solo against
your own best score. It's the "name the year" party game — built for the device
already in your pocket.

---

## How it plays

1. **Set up.** Choose how many players, pick a song pool and era, set a target
   score, and go. The whole setup is a single screen.
2. **Listen.** Tap play to hear a ~30-second snippet. Nothing about the song is
   shown — no title, no artist, no year. (You can't reveal the year until you've
   actually played the track, so nobody peeks early.)
3. **Guess the year.** Say your guess out loud, then tap **Reveal year**. The
   button glows in the current player's colour, so it's always clear whose go it
   is.
4. **Score.** The screen shows the song, artist and year, and you tap how you did:
   - **Got it** — bang on the year → **2 points**
   - **One year off** — within a year either side → **1 point**
   - **Missed** → **0 points**
5. **Pass and repeat.** The next player takes the phone. First to the target
   score wins.

That three-tier scoring is deliberate: "one year off" keeps near-misses fun and
rewards real knowledge, instead of the harsh all-or-nothing of most year games.

---

## Ways to play

- **Multiplayer (pass-and-play).** 2+ players share one device, first to a target
  score wins. Each player has their own colour on the scoreboard.
- **Solo (scored).** Play a set number of rounds and get a final score out of the
  maximum possible (e.g. "14 / 20"), with a grade. Great for a daily challenge or
  beating your own best.
- **Just practising (free, unscored).** A no-pressure warm-up — play through
  songs without keeping score.

## Choosing the music

Players tailor the deck to the room before they start:

- **Which songs?**
  - **UK #1s** — the chart-toppers (≈990 playable songs, 1952–2013).
  - **UK Top 10** — a much deeper pool of Top-10 hits (≈3,800 playable songs,
    1952–2025).
- **Which eras?** Filter to any mix of decades — 50s through 20s — or play all
  years.
- **Difficulty.**
  - **Easier (big hits)** — narrows to roughly the 10 best-known songs per year
    (ranked by popularity), so you mostly hear songs people actually recognise.
  - **All tracks** — the full pool, including deeper cuts, for a tougher game.

Difficulty filters whatever pool and eras you've already picked, so "Easier 80s
UK #1s" and "All-tracks everything" both just work.

---

## Why it's good

- **Instant.** No download, no sign-up, no account. Open a link and you're
  playing. Works on any phone.
- **Social by design.** One device passes round the table — no everyone-on-their-
  own-screen, no syncing, no setup friction. It's a game for a room, not a
  feed.
- **Fair, forgiving scoring.** The "one year off" tier means a good guess still
  counts. Knowledge is rewarded; near-misses sting less.
- **Tunable for any crowd.** Big-hits mode for a casual party; all-tracks for the
  music nerds; pick the decade you grew up in. The same game suits a family
  Christmas and a pub music quiz.
- **Genuinely nostalgic.** Hearing the opening bars of a song you'd forgotten is
  the whole hook — and "what year was that?!" is an argument every group enjoys
  having.
- **No clutter.** No physical cards to lose, no box, nothing to store. The deck
  is always up to date.

## Who it's for

- Friends and families at the table — parties, holidays, dinners.
- Music fans who want a real challenge (all-tracks, deep eras).
- Casual players who just want a quick laugh (big-hits, one phone).
- Quiz nights and icebreakers.

## How it compares

Think of the physical card game **Hitster**, but digital and free: no box of
cards to buy or lose, the song list stays current, and you can dial difficulty
and era to suit the people in the room. Yearworm focuses on UK chart history,
which makes it feel local and personal to a UK audience.

---

## Under the hood (for context, not marketing)

- A **single static web page** — no backend, no database, hosted on Netlify.
- Everything runs on the one device; nothing is uploaded or tracked.
- Song snippets are 30-second previews fetched fresh at play time (from Deezer,
  with an iTunes fallback), so links never go stale.
- The deck is built from UK chart data and refreshed by a small build pipeline
  (`hitster-build`); each song carries a popularity score that powers the
  "Easier" difficulty.

## Status & what's next

v1 is playable now. The roadmap (see `ROADMAP.md`) covers polish and planned
tiers — e.g. the song-pool and era choices are framed as unlocks for a future
free/paid split, currently open to everyone.

---

## Quick facts for a brief

- **Name:** Yearworm
- **Category:** Music year-guessing party game
- **Platform:** Web (any phone/tablet/computer), no install, no login
- **Price model:** Free now; some options framed as future unlocks
- **Decks:** UK #1s (~990 songs, 1952–2013) and UK Top 10 (~3,800 songs, 1952–2025)
- **Modes:** Multiplayer pass-and-play, scored solo, free practice
- **Hook:** Hear a snippet, guess the year; 2 points bang-on, 1 point within a year
- **Closest comparison:** A free, digital, always-current take on Hitster, focused on UK charts
