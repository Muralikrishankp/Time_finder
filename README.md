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
- This correctly handles DST transitions (PST ↔ PDT) via the browser's built-in tz database
- No hardcoded UTC offsets — everything goes through the Intl API

**State machine** — at any moment, the app is in one of three states:
| State | Condition | Countdown target |
|-------|-----------|-----------------|
| `boosted` | Weekend OR weekday outside 5–11 AM PT | Next peak start (or boost end) |
| `peak` | Weekday, 5–11 AM PT | Peak end (11 AM PT) |
| `expired` | After March 28, 2026 23:59 PT | — |

**Magic numbers**:
- `BOOST_END_PT` = `2026-03-29T06:59:59.999Z` — This is March 28 at 23:59:59.999 in PDT (UTC-7). In March 2026, PT is PDT because DST starts March 8, 2026.
- `PEAK_START_HOUR = 5`, `PEAK_END_HOUR = 11` — the 5 AM – 11 AM PT window

**Countdown precision**:
- `getState()` calculates exact milliseconds until the next state change
- Timer updates every 1 second via `setInterval`
- Progress bar shows elapsed % within current period

### Visual Design
- Dark theme with CRT-style scanlines overlay
- Green (#00ff88) for 2× active, Red (#ff6b6b) for peak hours
- Space Mono monospace font for the countdown timer
- Subtle pulse animation when boost is active
- Confetti burst on page load if 2× is currently active
- Fully responsive, mobile-first

### Files
- `index.html` — Everything in one file (HTML + CSS + JS). Zero dependencies, zero build step.

## Deploy
Drop `index.html` on any static host: Netlify, Vercel, GitHub Pages, Cloudflare Pages, S3, or just open it locally in a browser.

```bash
# Local
open index.html

# Or serve it
npx serve .
```
