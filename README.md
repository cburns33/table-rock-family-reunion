# Table Rock Lake Family Reunion 2026

Single-file itinerary site for the family reunion, July 18–23, 2026, at The Village at Indian Point, Branson, MO. Deployed as a static [GitHub Pages](https://cburns33.github.io/table-rock-family-reunion/) site.

## Structure

Everything lives in `index.html` — no build step, no dependencies, no other files. CSS and JS are inline in the same document.

- **Hero** — title, address, and two pill buttons: **Photos** (links to a shared iCloud album) and **Directions** (toggles a popout with Apple Maps / Google Maps links).
- **Day nav** — a sticky row of pill links (`Sat 18`, `Sun 19`, …) shown only below 1000px width, for jumping straight to a day on mobile without scrolling through the whole grid.
- **Day cards** (`.days`) — six cards, Saturday through Thursday, each with Morning/Afternoon/Evening blocks, a "Dinner host" block (the visually heaviest block, intentionally), and a "Notes" block where relevant.
  - Each card carries `data-date="YYYY-MM-DD"` and an `id` (`day-sat`, `day-sun`, …) used by the today-highlight logic and the day-nav links.
  - A small badge row in the day-head shows: conference room / pontoon reservation badges (mobile only), a "Today" badge (auto-applied via JS if the visitor's local date matches), and a weather badge.
- **Reserved banners** (`.reserved-grid`) — sit **above** the day-card grid, spanning the grid columns that correspond to their reserved days (desktop only, ≥1000px), arrows pointing down at the columns below. Below 1000px the day cards show small inline reserve badges instead, since the column-aligned banners don't work once the grid collapses to 3 or 1 columns.
- **Surf boat schedule** (`.surf-panel`) — a dedicated panel directly below the day cards and reserved banners, showing the "S.S. Sheworthy" wakeboard-boat rotation (times/groups per day, cove-out notes, captain/capacity flavor text). Mirrors the day-column layout of the day cards themselves. Each relevant day card also carries a small teal `.surf-badge` pointing to it — unlike `.reserve-badge` (mobile-only), `.surf-badge` shows at **all** widths, since there's no desktop-only equivalent for the surf schedule elsewhere on the card.
- **Info panels** (`.info-grid`) — Conference Room Activities, Rainy Day Activities, Boating, Nearby Stores, Condo Assignments. Several rows link out to Google Maps / ticket pages.

## Design language

Built around ["details that make interfaces feel better"](https://jakub.kr/writing/details-that-make-interfaces-feel-better) principles:

- Concentric border radius: outer radius (20px) = inner radius (12px) + padding (8px).
- Shadows instead of borders (`--shadow-1` / `--shadow-1-hover`).
- `font-variant-numeric: tabular-nums` on dates and the weather badge.
- `text-wrap: balance` on the hero title, `text-wrap: pretty` on body copy.
- Staggered entrance animation on the day cards (`animation-delay` per card).
- Hover micro-interactions: `translateY` + shadow shift on cards, `translateX` on rows/blocks.
- New elements (weather badge) fade/scale/blur in on load rather than snapping into place, consistent with the "contextual icon animation" principle.

## Features added this round

- **Today highlight** — `.day.is-today` gets a gold ring; the matching day-nav pill gets highlighted gold too. Computed client-side from the visitor's local date, no backend.
- **Weather badges** — **static, hand-entered values**, not a live API call (see Decisions).
- **Sticky day-nav** — mobile/tablet only (<1000px); anchors to each day card's `id`.
- **Photos button** — links to a shared iCloud album for people to dump/view trip photos.
- **Surf boat schedule** — the "S.S. Sheworthy" rotation, added after the primary boat owner texted over a schedule. See Structure and Decisions above.

## Decisions worth knowing

- **Weather is static, not live.** Originally wired to Open-Meteo (free, no API key). Two problems surfaced:
  1. A longitude sign bug briefly pointed it at India instead of Branson, MO (fixed, then re-verified against NWS/geocoding).
  2. Even after fixing the location, Open-Meteo's GFS model showed wildly high temps (104–106°F) for the days furthest out (12–13 days from generation time), diverging hard from Apple Weather's smoother numbers for the same dates. Neither ECMWF, ICON, nor NWS (`api.weather.gov`) extend forecasts that far out — NWS in particular caps at ~7 days — so there's no free/keyless API that reliably covers the full reunion week this far in advance.
  - **Resolution:** replaced the fetch with a hardcoded `WEATHER` object in the script, keyed by date, copied from a screenshot of Apple Weather. **This will go stale** — it's a snapshot, not a live forecast. Update the values in the `WEATHER` object near the bottom of `index.html` as the trip gets closer, or re-introduce a live API call once one exists that's both free and accurate at this range.
- **Reserved banners live above the day cards**, arrows pointing down (`▼▼`). This flip-flopped mid-project: moved below once, then moved back above when the surf boat schedule panel needed the space directly under the day cards instead. If repositioning either element again, keep in mind they're two separate below-the-grid elements now (reserved banners could go either side of the surf panel) — check current DOM order in `index.html` rather than assuming.
- **Surf boat schedule went through three design iterations** before landing on the current one: (1) a colored block stacked inside each relevant day card — dropped because day cards have wildly different heights (Notes lists vs. near-empty cards), so a per-card "ribbon" made to overlap the card's bottom edge landed at different heights across the row and looked broken, not intentional; (2) the same per-card block pulled back in-flow (no overlap) plus a separate aligned row of ribbons under the whole grid — functional, but fragmented the schedule; (3) **current**: one consolidated panel (badges on cards for at-a-glance chips, full detail in one panel) directly below the day cards. If asked to move it again, resist re-attempting the overlapping-card-block approach — it's not a good match for this grid's uneven row heights.
- **Hero's `overflow` must stay non-`hidden`** or the Directions popout gets clipped at the card edge. This was already fixed once before this session; don't reintroduce `overflow:hidden` on `header.hero`.
- **Condo assignments** are manually maintained rows in `.condo-panel .row-grid` — add a new `<div class="row">` entry per household as assignments come in.

## Deploy

No build step. Pushing `index.html` to `main` is the entire deploy — GitHub Pages serves it directly. No CNAME/custom domain configured.
