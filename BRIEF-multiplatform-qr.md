# Brief — Multi-platform QR Architecture

**Audience:** Claude Code (or any engineer picking this up).
**Status:** Approved direction, 2026-06-02. Supersedes the embedded-audio
decision in ROADMAP §"Snippet playback — move off the Spotify iframe".
**Why this exists:** the embedded-audio path uses Deezer previews, and
Deezer's developer ToS Section IV forbids commercial use of the Service or
any content accessed through it ("directly or indirectly, any moneys,
incomes, revenues, data or any other consideration"). A paid Yearworm tier
on the current stack is a clean breach. Full reasoning lives in
`ROADMAP.md`. This brief is the actionable version.

---

## TL;DR

Stop streaming audio inside Yearworm. Instead, at each turn, render a **QR
code that opens the song on the player's chosen streaming platform**
(Spotify or Apple Music — pick one at setup, stored in localStorage). The
user's own subscription/account streams the audio; Yearworm sells the
**experience and the deck**, not the music. Same legal architecture Hitster
uses.

Keep the embedded `<audio>` player **only** for the free "Just practising"
mode — that path stays free, single-device, and never carries commercial
intent, which is the cleanest reading of Deezer's non-commercial clause.

Drop Deezer entirely from the commercial paths. Do not use it for build-time
ID lookups for the paid product. Spotify + Apple Music IDs only.

---

## What changes — in one picture

| Path                  | Audio source            | When it's used                                    |
|-----------------------|-------------------------|---------------------------------------------------|
| **Paid game (Plus)**  | User's Spotify or Apple Music, opened via QR / deep-link | Every scored game; default once the player has picked Plus |
| **Free practice**     | Embedded `<audio>` (existing Deezer preview path) | "Just practising" mode only — solo, unscored, capped at `PRACTICE_LIMIT` |
| **Free scored game (#1s deck)** | User's Spotify or Apple Music (QR / deep-link) | Same as paid, but UK #1s deck only |

Net: the QR/deep-link is the default at every scored turn. The embedded
player survives **only** in the free, unscored practice flow.

---

## What stays the same

- Deck data model (`title`, `artist`, `year`, `id` (Spotify), `pop`, `dz`,
  `itunes` …). Add `am` (Apple Music track id) — see "Build pipeline" below.
- The whole scoring/multiplayer/winner/confetti/setup layer.
- The era picker, difficulty (easy/all), deck picker (#1s/Top Tens).
- The free-tier gates in `scoring.html` (Top Tens, >2 players, era selection
  remain Tier 1 unlocks; ALPHA flag still works).
- `songs.json` and `songs.toptens.json` as the runtime data files.

## What gets removed

- Deezer JSONP preview resolution (`resolvePreview` Deezer branch in
  `scoring.html` / `index.html`) **from the scored paths**. Keep it only on
  the practice path (or remove it entirely if we decide practice should also
  go via QR — see open questions).
- Live Deezer API calls at play time on any commercial path.
- The "couldn't load this clip — tap Reveal to skip" fallback messaging tied
  to live preview resolution, on scored paths.
- Eventually: the `dz` field can be dropped from the commercial deck files.
  Don't delete it yet — leave it during the transition in case we need to
  fall back to the embedded player as an "alpha" diagnostic switch.

## What gets added

1. **A "How do you listen?" preference** at first-run setup. Two options:
   - Spotify (default)
   - Apple Music
   Persists in `localStorage` as `yw_platform`. Editable from the setup
   screen later. No third option at v1 — see open questions.
2. **A QR card view** at each turn that renders the song's deep-link for
   the chosen platform. The existing `qrBlock` / `renderQR()` / `spotifyUrl()`
   in `index.html` and `scoring.html` is the starting point — extend it to
   route by platform.
3. **A same-device deep-link button** alongside (or above) the QR. Tapping it
   fires `spotify:track:{id}` or
   `https://music.apple.com/{country}/album/{albumId}?i={trackId}` — opens the
   song in the player's own app on the same device. QR is the fallback when
   that doesn't work (no app installed, or a different player wants to use
   their own phone).
4. **Honour-system copy** on the start/setup screen and inline at each turn:
   *"When the song opens in Spotify/Apple Music, the title is visible. Eyes
   off until you've guessed the year."*
5. **Apple Music ids on the deck** — see build pipeline.

---

## File-by-file plumbing

### `scoring.html` (primary target)

- Replace the `loadSnippet()` call inside `renderPlay()` with a router:
  - If `game.mode === 'practice'` AND practice mode is the embedded-audio
    one → call the existing `loadSnippet()`.
  - Otherwise → call new `renderPlatformCard(song)`.
- New function `renderPlatformCard(song)`:
  - Reads `localStorage.yw_platform` (`spotify` or `apple`).
  - Picks the URL:
    - Spotify: `https://open.spotify.com/track/${song.id}`
    - Apple Music: `https://music.apple.com/gb/song/${song.am}` (or the
      `album/{albumId}?i={trackId}` form — see open questions).
  - Renders the QR card via the existing `qrCode.js` integration.
  - Renders a "Play on this device" button above the QR that opens the same
    URL (use `window.location.href = url` or a regular `<a target="_blank">`).
- `revealBtn` logic stays — playing the song unlocks Reveal. On the QR path
  we can't reliably know when the player has actually tapped Play, so:
  **enable Reveal immediately** on QR cards. (The existing "must play before
  reveal" UX was for the embedded player. With QR, the user is offscreen in
  Spotify; we can't gate Reveal on real playback.)
- Replace the `playerLabel` "Mystery snippet" microcopy with "Tap to play, or
  scan with another phone".
- Add the honour-system one-liner under the QR card.

### `index.html`

The mode toggle is in this file. Two approaches:
- **Quickest:** keep `index.html` as the legacy single-device build, do the
  multi-platform work in `scoring.html`. They're already diverged; the
  ROADMAP treats `scoring.html` as the live development surface.
- **Cleanest:** port the changes back into `index.html` too, since the live
  site is on it. Decide which file is the canonical entry point and converge.

Recommend the **quickest** — make `scoring.html` the new `index.html`, archive
the old `index.html` as `index-legacy.html`. The Netlify root then points at
the new file. Saves duplicating work.

### `original.html`

Untouched. It's already documented as the historical reference (Spotify
iframe + QR mode). Leave as-is.

### Build pipeline (`hitster-build/`)

Add Apple Music IDs to the deck.

- Extend `match_previews.py` (or write `match_apple_music.py`) to:
  - For every song, query the iTunes Search API
    (`https://itunes.apple.com/search?term={title artist}&entity=song&country=gb`)
  - Pick the best match by title/artist/year proximity (the existing iTunes
    matching code already does fuzzy matching for previews — reuse it).
  - Capture `trackId` and `collectionId` (these are the Apple Music IDs).
  - Write to `songs.json` / `songs.toptens.json` as `am: <trackId>` and
    `amAlbum: <collectionId>`. (Two fields needed because Apple Music's
    deep-link form is `album/{collectionId}?i={trackId}`.)
- Coverage probe before rolling out: small stratified sample, check what %
  resolves to a usable Apple Music URL. Expect 80%+ (iTunes Store / Apple
  Music catalogue is broadly comparable to Deezer's).
- Don't drop `dz` yet — leave it in the file during transition.

---

## Acceptance criteria

A change set ships when **all** of these are true:

1. Setup screen has a "How do you listen?" picker; choice persists across
   reloads.
2. Starting any scored game (UK #1s or Top Tens, easy or all) lands the
   player on a QR card per turn, not an embedded player. The QR encodes the
   correct platform URL for the chosen song. A "Play on this device" button
   above the QR opens that URL on the same device.
3. Reveal Year works after either tapping Play, scanning the QR, or
   immediately (we don't gate it on the QR path — see plumbing notes).
4. Practice mode ("Just practising" button) **still uses the embedded
   `<audio>` player** with the existing Deezer/iTunes preview logic. No
   regression on the snappy first-tap UX for practice.
5. The honour-system copy appears at least once at game start and once on
   the QR card.
6. The deck files include an `am` (and `amAlbum`) field for at least 80% of
   playable songs.
7. The Netlify build still produces a working site on the root path; the
   old `index.html` is preserved at a clear archive path.
8. iPhone real-device test: tapping the same-device deep-link opens the
   Spotify app (if installed) or the web player (if not), plays a 30s
   preview to an unlogged user, and Yearworm is still in the back stack so
   the player can return to reveal. Same flow on Android Chrome.

---

## Open dev questions

Things I'd prefer the engineer to choose rather than dictate from this
brief. Pick during implementation; note the call in commit messages.

1. **Apple Music URL shape.** Two forms work:
   `music.apple.com/gb/album/{albumId}?i={trackId}` (the canonical) or
   `music.apple.com/gb/song/{trackId}` (the shorter modern form). Test both
   on iOS Safari and Android Chrome; pick the one that opens the song
   playing reliably, not the album page paused.
2. **Country code.** Default `gb` for UK Apple Music. Detect via
   `navigator.language` or just hardcode `gb` for v1? Hardcode is fine.
3. **Practice mode platform routing.** Should "Just practising" *also* go
   via QR/deep-link to the user's chosen platform, for consistency? Or stay
   on the embedded player for the snappier feel? Recommendation: **stay on
   embedded** — practice is the no-pressure warm-up; the QR friction works
   against that. The legal risk on a free, unmonetised practice mode is
   acceptable.
4. **Free scored game (#1s, no Plus purchase).** Same platform-routing as
   Plus? Recommendation: **yes** — the free scored game still has commercial
   *intent* (it's the lead-in to the paid tier), so it should be on the
   clean stack. Embedded-player only stays for unscored practice.
5. **Honour-system enforcement.** Pure copy, or do we visually obfuscate the
   transition (e.g. open the platform URL in a popup that auto-blurs)? Pure
   copy is fine for v1 — Hitster runs on honour and the category accepts
   it.
6. **Should the platform picker live in setup, or per-game?** Probably
   setup, with a small "change" link on the QR card if a player wants to
   switch mid-game. Don't over-engineer it.

---

## Out of scope for this change

- The Stripe paywall itself. Wire the platform pivot first; the paywall
  goes in after, on a clean architecture.
- A third platform (YouTube Music, Deezer-as-an-option). Two is enough;
  re-evaluate after launch metrics.
- Refactor to manifest + per-pack files (ROADMAP §1). Keep the existing
  `songs.json` / `songs.toptens.json` structure; just add the `am` field.
- Real accounts. Localised platform preference is enough for now.

---

## Risks worth naming

- **Apple Music deep-link reliability on Android.** Apple Music is
  available on Android, but adoption is lower and the open-in-app behaviour
  is patchier. Test early. If it's bad, downgrade Apple Music to "iPhone
  only" or fall back to the web player URL on Android.
- **Spotify open-link behaviour for unlogged users may change.** Currently
  (verified 2026-06-02 needed) Spotify plays a 30s preview to an unlogged
  web user. If Spotify withdraws that, the "no subscription needed" claim in
  marketing copy needs softening. Watch for it.
- **The QR friction tax in real play.** The whole pivot assumes parties
  will tolerate scan/listen/return. Watch this in the alpha; if energy
  drops after round 3, the same-device deep-link needs to become the
  default and QR demoted to fallback.

---

## What to do first

1. Add Apple Music ID matching to `hitster-build`. Run the coverage probe.
   Write the IDs into the deck files.
2. Prototype `renderPlatformCard()` in `scoring.html` for Spotify only.
   Get a single song's deep-link + QR working on a real phone.
3. Add the Apple Music branch.
4. Add the platform picker to setup; wire localStorage.
5. Honour-system copy + tidy the microcopy.
6. iPhone + Android real-device pass.
7. Cut over `index.html` to point at the new file; archive the legacy one.

Don't skip step 1. The whole pivot fails if the Apple Music catalogue
coverage is poor.
