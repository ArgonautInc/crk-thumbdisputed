# Thumb-disputed Championship — typing test (Phase 1)

`index.html` is fully self-contained: SimplyCricket fonts, the Cricket logo, styles and logic are all embedded. Double-click to run, or drop it on any static host. No build step, no dependencies, no data stored.

## How it works

**Test** — Type the full champion's speech (1,073 characters). The clock starts on the first keystroke and stops on the last character. Backspace is allowed, but a character that was wrong on first attempt still counts against accuracy. Pasting is blocked. Curly quotes in the copy accept straight typed equivalents (`'` for `’`, `-` for `—`).

**Score**

```
net WPM   = (correct characters ÷ 5) ÷ minutes      (capped at 250)
accuracy  = correct ÷ (correct + first-attempt errors)
points    = round(netWPM × 10 × accuracy² + longestCleanStreak × 0.5)
```

Accuracy is squared so sloppy speed-running loses to clean typing. The streak bonus rewards sustained clean runs.

| Points | Tier |
| --- | --- |
| 1000+ | Thumb-disputed Champion |
| 800–999 | Main Event |
| 600–799 | Title Contender |
| 400–599 | Rising Star |
| 200–399 | Undercard |
| 0–199 | Warm-up Act |

For calibration: 64 WPM at 99% accuracy scores ~764 (Title Contender). A 40 WPM typist lands around 410.

**Share** — The card is drawn on `<canvas>` at 1080×1920 and 1080×1080, exported as PNG. On mobile, *Share* opens the native share sheet with the image attached. Elsewhere it downloads. Instagram and TikTok don't accept web posts, so the flow is save → post from the app. Facebook opens the standard share dialog for the page URL.

## Look and feel

The site runs on black. The hero uses the ringside photo as its background with a scrim that darkens only the content bands, so the thumb and belt stay fully visible and uncovered:

- **Portrait / phone** — stacked. Title lockup on top, controls at the bottom, an open window between them holding the subject. The title art is capped at 24vh precisely so the top band can never reach the thumb.
- **Landscape / desktop** — two columns. A wide, short viewport can't fit a title band, the subject and a controls band stacked without crowding the belt, so the photo moves to its own column on the right and the copy sits beside it.

Layout was verified at 390×844, 834×1112, 1440×900 and 1920×1080 — see the two hero preview PNGs, where the red box marks the must-stay-clear subject.

Accent colors were re-picked for the dark background: `--green-bright #7BC144` (9.5:1 on black) carries text and correct-character highlights, while Cricket green `#40A829` stays on button fills with black text — the canonical brand pairing, and 6.7:1 either way.

## The title animation

The H1 is now the pixel lockup, revealed with a Colorburst-style arcade entrance (~1.9s):

1. The lockup zooms up from 14%, quantised to 13 discrete sprite steps so the scale-up reads as chunky arcade frames rather than a smooth CSS ease.
2. Six hue-rotated copies streak behind it as a rainbow echo trail, then collapse into the logo.
3. A CRT light sweep crosses it as it lands, masked by the artwork's own alpha (`mask-image: var(--art)`) so the shimmer only lights the lockup — never the empty corners of its box. Two details make that work: the sweep animates `background-position` rather than `transform`, because translating the element would drag the mask along with it and nothing would appear to move; and it's nested inside `.titleart` so it inherits the same scale during the zoom. Its timing is `linear` — an ease-out front-loads the travel so hard that the highlight crosses in a quarter of the duration and then sits offscreen.
4. Five sparkles pop, then the lockup settles into a slow idle bob.

`title-intro-preview.gif` is a frame-accurate replay of those keyframes. The intro replays every time the hero is shown (see `playTitle()`), and is disabled under `prefers-reduced-motion`.

## Screens

There are exactly three mutually exclusive screens: the landing page (`#start`), the game (`#test`), and the score/share screen (`#results`). `.screen:not(.is-active)` is explicitly removed from layout with `display:none!important`; this guard is necessary because the landing page's ID-based `display:grid` rule otherwise outranks the shared class rule and leaves a full hero viewport above the other two screens.

The hero and the test are pinned panels — `body.locked` kills document scrolling while either is up, so there's no scrolling back to a previous screen. The score/share screen is the one screen that scrolls normally in portrait. On landscape desktop it becomes a contained two-panel layout that fits beneath the header in one viewport.

The game screen includes a visible “Type it. All of it.” heading and supporting instructions above the passage. The typing card is intentionally capped at 860px wide and roughly half the viewport height, leaving a large wallpaper-ready field around it for future campaign branding. The card itself remains the scroll container, so reducing its size does not affect passage completion.

Orientation is treated as a layout decision rather than only a breakpoint: landscape gets a wider, height-aware reading measure, while portrait makes the passage nearly full-bleed and sizes it from viewport width. The score/share screen follows the same split: side-by-side score and sharing panels in desktop landscape, compact stacked cards with touch-sized actions in mobile portrait.

An earlier version used CSS Grid with `minmax(0,1fr)` rows and a `max-height:100%` on the card. That's spec-legal, but it depends on a percentage height resolving through an ancestor whose own size came from grid stretch — support for that isn't uniform, and when it fails the browser treats the height as unconstrained rather than 0, so the card silently grows to its full content height with nothing to stop it. That's exactly the "seeing half the page, can't scroll" report: the passage was overflowing past the visible screen with no indication anything was wrong, because on a wide-but-short external monitor there wasn't much headroom to begin with. `flex-grow` doesn't have that failure mode — it distributes an already-known pixel height directly, no percentage math involved, so there's no ambiguity for a browser to get wrong.

**Height-based breakpoints, separate from the width ones.** The old mobile breakpoint was `@media (max-width:600px)`, which only ever fires on narrow phones — it does nothing for a wide desktop window that's simply short (an external monitor with a lot of browser chrome, a laptop lid at an angle, a phone rotated to landscape). Two new breakpoints key on height instead: `max-height:820px` compacts the header, footer, and gaps; `max-height:560px` goes further and drops the stat labels down to bare numbers. These stack independently of the width rules, so a short *and* narrow window gets both sets of savings.

Fit was verified at 1920×1080, 1440×900, 1280×720, 390×844, 390×520 (phone with the keyboard up), and — the scenario that actually broke — a set of short external-monitor and landscape-phone sizes: 1920×950, 1920×860, 2560×1000, 3440×900 (ultrawide), 1680×820, and 844×390. All fit with the footer fully visible; the tightest ones just show fewer lines of the passage in the scrollable card, which is the intended degradation. See the four test-panel preview PNGs, including one at 1920×860 (external-monitor) and one at 844×390 (phone landscape).

## Dev tools

`DEV_TOOLS` at the top of the `<script>` controls a **DEV · Skip to results** button in the test footer, which jumps straight to the share screen without typing the whole passage. If you've started typing, it scores your real progress; if you haven't, it fakes a 3:14 / 66 WPM / 99% run so the card has plausible numbers.

Set `DEV_TOOLS = false` before shipping, or append `?dev=0` to the URL to hide the button without editing the file.

## Changing things

- **Copy** — the `RAW` constant at the top of the `<script>`.
- **Scoring** — `computeScore()` and the `TIERS` array.
- **Images** — both are base64 CSS custom properties, `--hero` and `--art`, injected by `build.py` from `hero_q82.jpg` and `t128.png`. Swap the source files and rebuild.
- **Animation timing** — the `spriteZoom`, `echoTrail`, `scanSweep` and `sparkPop` keyframes. The step count in `steps(13,end)` controls how chunky the zoom feels.
- **Card layout** — the `LAYOUT` object holds every baseline for both ratios; `drawCard()` just reads it. Story-card content sits between y=200 and y=1760 to stay clear of Instagram's top and bottom chrome.

## Phase 2 hooks

`computeScore()` returns a plain object (`score, wpm, acc, ms, errors, streak, name, tier`) — that's the leaderboard row. `finish()` is the single place a POST would go, and the results screen already has a slot above the share panel for a rank readout. Anti-cheat to consider before scores are public: the 250 WPM ceiling is in place, but a server-side check on elapsed time vs. character count is worth adding.

## Verification

Logic is exercised headlessly through a full simulated run (scoring math, tier thresholds, quote forgiveness, restart, paste blocking, screen exclusivity, and card bounds). A live Chrome pass also measures and captures the game and score/share screens at 1440×900 landscape and 390×844 portrait. The game is exactly one viewport at both sizes; desktop results fit in one viewport, while portrait results intentionally scroll as a compact score card followed by the sharing tools.
