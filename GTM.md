# Yearworm — Go-to-Market Plan

Plain-English plan for the first 90 days. Goal: **first £ in the bank from the
paid tier**, with a small budget (<£500) and weekends only. Sits alongside
`PRODUCT.md` (what the game is) and `ROADMAP.md` (what gets built).

---

## The bet (read this first)

The 90-day goal is **revenue, not signups**. That changes everything.

- A free-only public launch is a one-shot attention spike with nothing to
  capture. If we burn it before the paywall is live, we lose the only moment
  most products ever get.
- So the order is: **paywall → public launch with it live → push**.
- We don't need a *good* paywall in time. We need an *honest* one — one tier,
  one price, one click, no accounts. Cut the rest.
- The story is the wedge, not the tech. "**Hitster, but free, on a phone you
  already have, with UK chart history baked in.**" Everything in this plan
  serves that line.

---

## Positioning — the one paragraph everything else copies from

> **Yearworm is the free, no-download music year game for parties.**
> Hear a 30-second snippet, guess the year — bang-on for 2 points, within
> a year for 1. One phone, passed around the table. No app, no login, no cards
> to lose. Built on UK chart history, from Vera Lynn to the streaming era.

Three claims, in this exact order:

1. **Free + no install.** This is the wedge against Hitster (£25 box) and against
   every app-store music quiz (download, account, ads).
2. **Party game on one phone.** This is the wedge against Spotify-quiz bots, web
   trivia sites, and the Heardle generation — those are *solo* games. Yearworm
   is what you reach for when there are six people at the table.
3. **UK chart history.** This is the wedge against the US-coded competition
   (Heardle, Hitster's US deck). It's a locality advantage and an SEO/PR
   advantage. Lean on it.

### Names for the tiers (use these everywhere)

- **Yearworm Free** — UK #1s, 2 players, all years.
- **Yearworm Plus** — UK Top 10 deck (3,800+ songs), unlimited players, pick
  your eras. *One price, one unlock, lifetime.*
- (Later) **Yearworm Quiz** — title + artist + year, 3 points per song.
- (Later) **Genre packs** — heavy metal, etc.

Resist subscription for v1. "Pay £4 once and it's yours" sells in a tweet.
"Subscription" doesn't — and you have no monthly content cadence to justify it
yet. (Revisit when genre packs exist and ship monthly.)

---

## Pricing — pick one number, ship it, move on

**Recommended launch price for Yearworm Plus: £3.99 (one-off, lifetime).**

Reasoning:

- Hitster retails ~£25 in the UK. £3.99 is "an eighth of a Hitster, infinitely
  more songs, never out of date." That comparison wins on its own.
- £3.99 sits below the impulse threshold for adults at a party. £4.99 is the
  next obvious step if conversion is fine and you want to test elasticity.
- Avoid £0.99 / £1.99 — they read as cheap and force Stripe fees to swallow a
  big % per sale.
- Avoid £9.99+ until you have social proof. You haven't earned it yet.

**When to raise:** if conversion (free → paid) stays >3% after the first
~1,000 free players, push to £4.99. If it sinks below 1%, the problem is the
*moment* of the prompt, not the price — keep the number, move the gate.

**Tax / VAT:** Stripe handles UK VAT inclusive pricing — set the price as
"£3.99 incl. VAT" so the EU/UK shopper sees one honest number. Worth ~10 min
of Stripe setup.

---

## The paywall MVP — exactly what to build, exactly what NOT to

This is the thing standing between you and revenue, and it's where you'll
overbuild if you're not careful. **One tier, one price, no accounts.**

### Build

1. **One paid unlock**: "Yearworm Plus" — bundles Top Tens + >2 players + era
   picker. That's it. Flip `ALPHA = false` so the gates exist, then unlock all
   three behind one Stripe purchase.
2. **Stripe Payment Link or Checkout** — no custom UI. Stripe-hosted page.
   Cheapest possible integration.
3. **Entitlement = signed token stored in localStorage.** After payment,
   Stripe redirects back to `yearworm.netlify.app/unlock?token=…`. A tiny
   Netlify Function (one file) verifies the token and writes
   `localStorage.yw_plus = <token>`. The game reads that flag at startup.
4. **One restore-purchase link** on the start screen: "Already bought?
   Restore" → enter the email used at checkout → Function looks up the Stripe
   customer and re-issues the token by email. ~30 lines.

### Do not build (yet)

- ❌ User accounts, passwords, sign-in. The roadmap calls for Supabase
  eventually — *not for launch*. localStorage-token + email-restore is enough.
- ❌ Subscription billing. One-off only.
- ❌ Per-device sync. If they buy on their iPad and want it on their phone,
  they tap "Restore" with their email. That's fine for v1.
- ❌ Manifest/per-pack refactor (in the roadmap). Ship as-is; the bundled
  `songs.toptens.json` is technically extractable by a determined user — that's
  the "fort knox vs friction" tradeoff already acknowledged. Refactor *after*
  you have paying customers asking for genre packs.

### Time estimate

Two solid weekends. Stripe (½ day) + Netlify Function (½ day) + UI plumbing
into the existing `scoring.html` gates (1 day) + restore flow + testing
(1 day).

### Pre-launch checklist (do all of these before pushing publicly)

- [ ] Deezer terms of service — confirm the live API call is allowed for a
  commercial product (the roadmap flags this).
- [ ] iPhone audio-unlock — verify the first-tap behaviour on a real iPhone,
  in Safari and as a Home Screen icon.
- [ ] One desktop layout pass — most press will open it on a Mac and the
  current "phone column in black sea" reads as broken. Doesn't need to be
  perfect; needs to not embarrass you.
- [ ] Open Graph image + title — every share needs to render properly.
- [ ] Refund line: state "£3.99, refund on request, no questions" on the
  Stripe success page. Cheap goodwill, kills nearly all chargebacks.

---

## The 90-day plan — week by week

Treat this as a Gantt in prose. Numbers are realistic for evenings/weekends,
not for someone on it full-time.

### Weeks 1–3: Build the paywall, fix the launch-killers

- W1: Stripe + Netlify Function + entitlement flow. End the week with a
  payment that actually unlocks Top Tens on your phone.
- W2: Restore-purchase, refund line, desktop pass, OG image, Deezer ToS check.
- W3: Soft alpha to 20–40 friends (use the tester-code path from the roadmap).
  Goal = **does the paywall moment actually convert any of them?** If even
  10–20% pay, the gate works. If 0%, fix the gate before any public push.

### Weeks 4–6: Landing page + content stockpile + first SEO hooks

- **Landing page** at `yearworm.netlify.app/` (or a separate `/about`). The
  current `index.html` jumps straight into the game; press and shares need a
  page that explains and converts. See "Landing page asks" below.
- **Content stockpile** (you'll need this for the channel push):
  - 10 short videos (TikTok/Reels/Shorts). Format: friend(s) at a table, snippet
    plays, big "GUESS" overlay, reveal, scores update. ~15s each.
  - 5 still screenshots for press/PR (start screen, mid-game scoreboard,
    winner screen with confetti, era picker, ±1 scoring moment).
  - 1 "why we built it" 200-word post, ready for HN/Reddit/your own blog.
- **SEO**: register `yearworm.com` (~£10/yr) if not already. Redirect to
  Netlify. The .com is worth it for press credibility.
- **Email capture** on the landing page — "Get the genre packs first" mailing
  list. Mailerlite or Buttondown free tier.

### Week 7: Launch week (the one shot)

This is the week the attention happens. Front-load it.

- **Monday:** post on **r/CasualUK** ("Made a free thing for British music
  nerds — guess the year of UK #1s"). UK-coded, ground-truth comparator to
  Hitster. Don't post the link as a self-promo if the sub bans them — use the
  "I made" framing and link in the first comment.
- **Tuesday:** post on **r/Hitster** (yes — they're the warmest audience you
  could possibly find; frame it as "free digital cousin," not "replacement").
  Also **r/InternetIsBeautiful** and **r/sideproject**.
- **Wednesday:** Product Hunt launch. Tagline: *"Hitster, but free and on the
  phone you already have."* Aim for the Tuesday→Thursday window; Wednesday is
  the safest. Schedule for 00:01 PST.
- **Thursday:** Hacker News **Show HN** — "Show HN: Yearworm — a free
  guess-the-year music game, single-page web app." HN cares about the
  technical story (no backend, Deezer JSONP, expiry-token workaround) — lead
  with that, not the marketing line. The ROADMAP's "Deezer URLs expire" note
  is genuinely interesting to that crowd.
- **Friday:** PR pitches sent. Targets (in order of how likely they are to
  bite):
  1. **The Guardian — games section** ("a digital Hitster, built free")
  2. **TimeOut London** (party game, UK-focused)
  3. **BBC R6 Music** social team (UK chart heritage angle, ping via
     @BBC6Music on socials with a clip)
  4. **Music Ally** / **CMU Daily** (music industry trade press — preview
     audio + chart data is a real story)
  5. **MetaFilter** (free, fun, no spam — they love this kind of thing)
- **All week:** post 1 TikTok/day. Different hook each day:
  - "Which year? You have 5 seconds." (gameplay, no commentary)
  - "Reaction to a 1968 snippet from a 22-year-old"
  - "Pub quiz, but make it a phone"
  - "Hitster costs £25. This is free. Same game."
  - "Tell me your year. I'll guess your decade." (POV bait)

### Weeks 8–9: Read the data, push what worked

- One channel will outperform. **Triple down on it; cut the others.**
- This is when you spend the **£500**. Don't spend before launch — you don't
  yet know what the hook is. Spend it on the channel + creative that already
  worked.
  - If TikTok worked: £300 boosting your best post, £200 to one mid-size
    creator (5–50k followers in the UK music-fan niche).
  - If Reddit/PR worked: £500 into Meta retargeting people who hit the
    landing page but didn't play, with the press quote as the ad.
  - If neither worked: do not spend; reread "Kill criteria" below.

### Weeks 10–13: Iterate the gate, ship the first follow-up

- Look at the funnel: open → play 1st song → finish 1st game → hit gate →
  pay. The drop-off tells you exactly which problem to solve next.
- Ship the **first genre pack** (heavy metal or 90s indie — pick whichever
  TikTok told you wants it). It's a re-engagement email AND a reason to
  return AND a second purchase point. Charge £1.99 for it.
- Start a fortnightly email to the list: "New pack" / "What we're working on."
- By day 90: revenue exists, you know your CAC, you know your conversion %,
  you know which channel works. That's the launchpad for everything else.

---

## Channel notes — what to ask of each one

| Channel | What it's good for | Cost | Realistic outcome |
|---|---|---|---|
| **Reddit (r/CasualUK, r/Hitster, r/sideproject, r/IIB)** | First proof, traffic spike, qualitative feedback | Free | 2,000–10,000 visits across the week if one post breaks out |
| **Product Hunt** | Tech/maker credibility, backlinks, follow-on press | Free | Top 5 of the day if launch is decent. Few sales, lots of authority |
| **Hacker News (Show HN)** | Technical credibility + a small-but-paying long-tail | Free | One-day spike; the technical story has to lead |
| **TikTok organic** | The single highest-ceiling channel for this product | Free + time | A hit reel = 100k+ views. Most won't hit. Volume beats polish |
| **TikTok paid boost** | Amplifying one already-working organic post | £200–£300 | Worth it only AFTER a video has proven product-market hook |
| **Creator post (5–50k UK music)** | Trusted recommendation in-niche | £100–£250 | One good creator post > 10 mediocre paid ads |
| **PR pitches** | Long-tail authority, "as featured in" logos | Time | Most won't bite. Even one Guardian mention pays for itself |
| **Meta/Google paid ads** | Only useful if you have a known CAC | £100+ | **Skip for v1**. Burn rate too high without a known funnel |

**Channel I'd skip for v1:** Twitter/X for distribution. Use it only for
indie-maker-Twitter posts (build-in-public) as colour, not as a launch
channel. The audience there is not your audience.

---

## Landing page asks

The current `index.html` opens the game. That's right for repeat visitors and
wrong for cold traffic. Build a separate `landing.html` (or new index, with
the game moved to `/play`) that does these jobs:

1. **Above the fold:** the brand, the one-line pitch, an embedded silent
   ~6-second video clip of a snippet → reveal → score moment. The *motion* is
   what sells the game; no still image will do it.
2. **One CTA: "Play now — no download."** Goes straight into the game.
3. **The Hitster comparison** — explicit. "Like Hitster, free and digital."
   This sentence sells.
4. **Three screenshots** — start screen, mid-game, winner. With captions, not
   just images.
5. **"What's inside" table** — Free vs Plus, side by side. £3.99 number is
   visible without scrolling on mobile.
6. **Email capture for genre packs.** Below the fold. One field.
7. **Press logos** (when you have them — leave space).
8. **FAQ at the bottom**: "Do I need an account?" "Will the Spotify song
   list go stale?" "Why UK only?" "Refunds?" Two-sentence answers.

Keep it ~3 screens on mobile. Ruthless cuts. No "Our story" page.

---

## Metrics — what to measure, and why

Don't ship analytics that take a week to set up. Use **Plausible** or **Umami**
(both privacy-friendly, GDPR-clean, no consent banner needed for the basics) —
both free tiers cover this volume. Set up these events:

- `landing_view` — landing page open
- `game_start` — Start button tapped (the real activation event)
- `round_complete` — a game played to its target
- `gate_hit` — paywall shown (split by which gate: deck / players / eras)
- `checkout_open` — Stripe Checkout opened
- `purchase` — Stripe webhook fires

The numbers you actually want to know each week:

- **Activation rate** = `game_start` / `landing_view`. Below 50% = landing
  page broken or hook unclear.
- **Completion rate** = `round_complete` / `game_start`. Below 30% = the
  game's not fun enough yet to finish (or the snippet isn't loading).
- **Gate exposure** = `gate_hit` / `game_start`. Need this >30% for the
  paywall to have a chance — if people never see it, nothing else matters.
- **Conversion** = `purchase` / `gate_hit`. North-star number for the bet.
  >3% = the gate is well-placed. <1% = move the gate, not the price.

Reread weekly. Don't add more metrics until these are stable.

### Kill criteria — when to change the plan, not just push harder

- **Week 3:** if zero friends pay in the alpha, the gate is wrong. Stop and
  redesign the moment, don't launch.
- **Week 7:** if launch week produces <500 `landing_view` and you've done all
  the pushes, the *positioning line* isn't pulling. Rewrite it before any
  paid spend.
- **Week 9:** if conversion is <1% after 1,000+ exposures, **do not raise the
  price and do not run more ads**. Move the gate (e.g., let people *play* one
  Top Tens song before the wall, so they feel the upgrade) and re-measure for
  two weeks.

---

## What I'd say is *not* a priority for launch

These are tempting and they all lose to "ship the paywall and push."

- **Real auth (Supabase, Netlify Identity).** Roadmap has it; ship it with
  the *second* paid feature, not the first.
- **Manifest + per-pack refactor.** Important for the catalogue, irrelevant
  to the wedge.
- **Genre packs at launch.** Ship after the first paid customer exists, as
  the re-engagement story.
- **A "real" desktop layout.** One pass to stop looking broken on Mac. That's
  it.
- **Anything that says "v2".** Save it for the post-mortem.

---

## The single most important thing on this page

The launch is a **moment**, not a product. Build the paywall before that
moment. Use the moment to find which one of (Reddit, TikTok, Hacker News, PR)
actually pulls for *this* product. Spend the £500 there, not in advance.
Everything else compounds from that.
