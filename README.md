# Claude Throttle Tracker

A 100% client-side, single-page static site that tracks whether Anthropic's 2× usage limit boost is currently active.

## How It Works

### The Rules (from Anthropic's March 14, 2026 announcement)
- **Duration**: ~2 weeks, March 14 – March 28, 2026
- **2× boost** applies **outside peak hours** — automatic, no opt-in
- **Peak hours**: Weekdays (Mon–Fri), **5:00 AM – 11:00 AM Pacific Time**
- **Off-peak (2× active)**: Weekdays outside 5–11 AM PT + **all weekends**

### Technical Details

**Timezone handling**:
- Uses `Intl.DateTimeFormat` with `timeZone: 'America/Los_Angeles'` to get the current time in PT
- `ptToUTC()` converts PT wall-clock → UTC via a two-pass Intl algorithm (handles DST boundaries correctly)
- No hardcoded UTC offsets anywhere — `BOOST_START` and `BOOST_END` are computed dynamically via `ptToUTC()`
- `ptDayOfWeek()` uses `Date.UTC` to avoid local-timezone day-of-week bugs for users in UTC+12/+14

**State machine** — at any moment, the app is in one of four states:
| State | Condition | Countdown target |
|-------|-----------|-----------------|
| `not-started` | Before March 14, 2026 PT | Boost start |
| `boosted` | Weekend OR weekday outside 5–11 AM PT | Next peak start (or boost end) |
| `peak` | Weekday, 5–11 AM PT | Peak end (11 AM PT) |
| `expired` | After March 28, 2026 23:59:59 PT | — |

**Constants** (no magic numbers):
- `BOOST_START` = `ptToUTC(2026, 3, 14, 0, 0, 0)` — dynamically resolved
- `BOOST_END` = `ptToUTC(2026, 3, 29, 0, 0, 0) - 1ms` — end of March 28
- `PEAK_START_HOUR = 5`, `PEAK_END_HOUR = 11` — the 5 AM – 11 AM PT window

**Countdown precision**:
- `getState()` calculates exact milliseconds until the next state change
- `Math.max(0, ...)` guards prevent negative/NaN countdowns at boundaries
- `isFinite()` check in `formatCountdown()` handles edge cases
- Timer pauses when tab is hidden (visibility API), re-syncs immediately on return

### Security
- **No innerHTML with user data** — all DOM writes use `textContent` or `createElement`
- **No external data** — zero fetch calls, zero API dependencies
- **No prototype pollution** — no dynamic object merging
- **No string-based date parsing** — `ptToUTC` uses only `Intl.formatToParts` + `Date.UTC`

### Accessibility
- Semantic HTML: `<h1>`, `<h2>`, `<p>`, `role="main"`, `role="status"`, `role="progressbar"`
- `aria-live="polite"` on status elements for screen reader updates
- `aria-valuenow`/`aria-valuemin`/`aria-valuemax` on progress bar
- `prefers-reduced-motion: reduce` disables pulse animation and confetti
- High contrast colors (green #00ff88, red #ff6b6b on #0a0a0a background)

### Visual Design
- Dark theme with CRT-style scanlines overlay
- Green (#00ff88) for 2× active, Red (#ff6b6b) for peak hours
- Space Mono monospace font for the countdown timer
- Subtle pulse animation when boost is active
- Confetti burst on page load / state transition to boosted (self-cleaning, cancellable)
- Fully responsive, mobile-first (100dvh, fewer confetti particles on small screens)

### Files
- `index.html` — Everything in one file (HTML + CSS + JS). Zero dependencies, zero build step.

## Deploy
Drop `index.html` on any static host: Netlify, Vercel, GitHub Pages, Cloudflare Pages, S3, or just open it locally in a browser.

```bash
# Local
open index.html

# Or serve it
python3 -m http.server 8000
npx serve .
```
