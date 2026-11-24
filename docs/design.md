# ICPilot — Design Specification (`design.md`)

> The UI must look **exactly like the 6 reference images** (glass dashboard, "Synthex Experience Platform" style).
> Same layout, same glass effect, same shapes, same type feel. **Only the data, labels, and colours listed here change.**
> Agents: if you are unsure how something looks, DO NOT improvise. Re-read this file. If still unsure, write `DESIGN QUESTION:` in your report.

Reference images (kept in `docs/reference/`, filenames in this order):
1. `ref-1-engagement-closeup.png` — big number + sub-stats
2. `ref-2-chart-closeup.png` — hatched band chart, filter chips, +24% callout
3. `ref-3-folder-cards.png` — "Visitors Insights" header, folder cards, drop zone
4. `ref-4-sidebar.png` — sidebar tree, badges, bottom icons, feedback pill
5. `ref-5-monitor-closeup.png` — full layout on monitor, dock, bar chart
6. `ref-6-full-monitor.png` — full app incl. top nav, header, gauge, bars

---

## 0. What is the product UI and what is only mockup

| Element in images | Build it? |
|---|---|
| macOS window chrome (red/yellow/green dots, URL bar `synthex.com/...`), monitor, stand, grey desk background | **NO.** Mockup only. |
| Perspective tilt / skew / depth-of-field blur in images 1–4 | **NO.** Those are camera crops of the same flat UI. Build flat, sharp, upright. |
| Everything inside the screen (sidebar, top nav, header, cards, charts, dock) | **YES, exactly.** |
| App name "Synthex Experience Platform" | Replace with **ICPilot** (logo mark in red-orange square kept in same position and size). |

---

## 1. Design tokens (single source: `src/styles/tokens.css`)

Colours were sampled from the images. Values are approximate because the images are soft mockups. **All components use tokens only. No hex values in components.**

```css
:root {
  /* Page background: vertical gradient seen on the monitor screen */
  --bg-top:      #4D697F;  /* slate blue, top edge of screen */
  --bg-upper:    #89A1AB;
  --bg-mid:      #C5D7D8;
  --bg-lower:    #DBE9E6;
  --bg-bottom:   #EAF6F2;  /* pale mint */
  --bg-base:     #F2F9F7;
  --page-gradient: linear-gradient(180deg,
      var(--bg-top) 0%, var(--bg-upper) 18%, var(--bg-mid) 38%,
      var(--bg-lower) 58%, var(--bg-bottom) 80%, var(--bg-base) 100%);

  /* Ink (text and navy buttons) */
  --ink-900: #173041;   /* navy ink: big numbers, titles, navy pills */
  --ink-800: #1F3A4C;
  --ink-600: #4A6272;   /* labels (readable muted) */
  --ink-400: #7E9099;   /* reference-grey for DECORATIVE text only (see 12.1) */
  --ink-200: #B7C4C8;

  /* Glass fills (white at different opacity over the gradient) */
  --glass-1: rgba(255,255,255,0.34);  /* large cards */
  --glass-2: rgba(255,255,255,0.50);  /* chips, small tiles */
  --glass-3: rgba(255,255,255,0.72);  /* active chip, dock selected, callouts */
  --glass-border: rgba(255,255,255,0.65);
  --glass-border-soft: rgba(255,255,255,0.35);
  --glass-blur: 18px;
  --glass-saturate: 140%;

  /* Shadows (very soft, bluish) */
  --shadow-card:  0 18px 40px -18px rgba(23,48,65,0.28), 0 2px 6px rgba(23,48,65,0.06);
  --shadow-float: 0 24px 48px -16px rgba(23,48,65,0.34);
  --shadow-inset: inset 0 1px 0 rgba(255,255,255,0.7);

  /* Accents */
  --accent-purple-glow: #A6A7E2;  /* glow under the first folder card */
  --accent-red-glow:    #F26A6A;  /* glow under the second folder card, notification dot */
  --accent-brand:       #E5483F;  /* ICPilot logo square (red-orange in reference) */
  --chart-dot-dark:     #173041;  /* upper series dots */
  --chart-dot-grey:     #8E9AA0;  /* lower series dots */
  --chart-hatch:        rgba(23,48,65,0.22);
  --chart-grid:         rgba(23,48,65,0.18);   /* dashed */

  /* Semantic (small use only, never large fills) */
  --ok:    #2E8F6B;
  --warn:  #C98A1B;
  --bad:   #C4473F;
  --info:  #3C6E9F;

  /* Radii */
  --r-card:   28px;   /* big glass cards */
  --r-folder: 30px;   /* folder cards */
  --r-tile:   22px;   /* Home/Dashboard tiles, + tile */
  --r-chip:   999px;  /* pills, chips, circle buttons */
  --r-callout:20px;

  /* Spacing scale (px) */
  --s-1: 4px; --s-2: 8px; --s-3: 12px; --s-4: 16px; --s-5: 20px;
  --s-6: 24px; --s-8: 32px; --s-10: 40px; --s-12: 48px;

  /* Layout */
  --sidebar-w: 248px;
  --topnav-h: 48px;
  --header-h: 72px;
  --panel-gap: 16px;

  /* Motion */
  --ease: cubic-bezier(.2,.7,.2,1);
  --dur-fast: 120ms; --dur: 220ms; --dur-slow: 420ms;
}
```

Light theme only for P1. Dark theme is `ADD-09` (Phase 20): same tokens re-mapped; never started earlier.

---

## 2. Typography

**Font: Urbanist** (Google Fonts, free) — the closest match found by eye to the thin geometric rounded sans in the references. Fallback chain: `"Urbanist", "Outfit", system-ui, sans-serif`.
`FONT IDENTIFIED BY EYE FROM A SOFT MOCKUP. IF YOU FIND THE REAL FONT NAME, CHANGE ONLY --font-sans IN tokens.css.`
Load with `next/font/google`, weights **300, 400, 500, 600**. `font-feature-settings: "tnum"` and `font-variant-numeric: tabular-nums` on all numbers and table cells.

| Role | Size / line-height | Weight | Colour | Where (reference) |
|---|---|---|---|---|
| Hero number | 56px / 60px | **300** (thin) | `--ink-900` | "40,439.00" |
| Sub-stat value | 28px / 32px | 300 | `--ink-900` | "3m 42s", "37.6%", "5.2", "18,320" |
| Gauge value | 44px / 48px | 300 | `--ink-900` | "40,439" in gauge |
| Card title | 18px / 24px | 500 | `--ink-900` | "User Engagement", "Session Duration", "Page Views" |
| Folder card title | 20px / 26px | 400 | `--ink-900` | "Traffic Overview", "Conversion Rate" |
| Page title | 20px / 26px | 400 | `--ink-900` | "Visitors Insights" |
| Sub-label | 13px / 18px | 400 | `--ink-600` | "In Last 7 days", "Avg. Session" |
| Sidebar item | 14px / 20px | 400 (active 500) | `--ink-900` | Overview, Insights, Campaigns |
| Chip text | 14px / 20px | 400 | `--ink-900` | "Device Type", "Traffic Source" |
| Axis label | 12px / 16px | 400 | `--ink-600` | "17 Sep", "20,000" |
| Badge | 11px / 14px | 600 | white | "2", "15+", "3" |
| Table text (dense) | 13px / 18px | 400 | `--ink-900` | data grids |
| Table header | 12px / 16px | 500 | `--ink-600` | uppercase **no**; sentence case |

Rules:
- No italic anywhere. (The slanted text in images 1–4 is camera perspective.)
- No bold above 600. Large numbers are always light (300).
- Number formatting: thousands separators; percentages 1 decimal; `.00` only for monetary or score values that need it. Duration as `3m 42s`.

---

## 3. Glass surface system

One CSS utility family in `src/styles/glass.css`:

```css
.glass       { background: var(--glass-1); backdrop-filter: blur(var(--glass-blur)) saturate(var(--glass-saturate));
               border: 1px solid var(--glass-border); border-radius: var(--r-card);
               box-shadow: var(--shadow-card), var(--shadow-inset); }
.glass-chip  { background: var(--glass-2); border: 1px solid var(--glass-border-soft); border-radius: var(--r-chip);
               backdrop-filter: blur(12px); }
.glass-solid { background: var(--glass-3); }            /* callouts, selected dock item */
.glass-tile  { border-radius: var(--r-tile); }
```

Recipe details (what makes it look like the reference):
- Fill is **white translucent**, not grey. The gradient shows through.
- Border is a **1px white hairline**, brighter on top edge (use `--shadow-inset`).
- Shadows are **soft and low-contrast**; never black, never hard.
- Blur is strong enough that content behind is a soft wash (blur ≈ 18px).
- Large radii everywhere (cards 28px, tiles 22px, pills fully round).
- No visible dividers inside cards; use spacing. Where a divider is needed: 1px `rgba(255,255,255,0.5)`.
- Fallback when `backdrop-filter` unsupported: solid `rgba(244,250,248,0.9)`.
- Performance: max ~12 simultaneous blurred layers per screen; tables inside cards must not each have their own blur.

---

## 4. App layout (grid)

Screen = full viewport with `--page-gradient`. Inside, the app frame fills the viewport with 16px outer padding.

```
┌ TOP NAV (48px): [logo][App name]   Project  Edit  Components  Share  Help        [search?]  ┐
├────────────┬───────────────────────────────────────────────────────────────────────────────┤
│  SIDEBAR   │  PAGE HEADER (72px): (←) Title      [filter input......🔍]  [Refresh][⤓][↗] [navy context pill ⌄ ✕] │
│  248px     │  PINNED VIEWS ROW: [+] [folder card] [folder card] [dashed drop zone........]  │
│  glass     │  MAIN GRID: hero card | chart card ; gauge card | bars card ; ...             │
│            │                                                       [floating DOCK, bottom-centre] │
│  bottom:   │                                                                               │
│  icon row  │  RIGHT DRAWER (Agent Thinking), collapsible, glass, overlays from the right    │
│  feedback  │                                                                               │
│  avatar    │                                                                               │
└────────────┴───────────────────────────────────────────────────────────────────────────────┘
```

- Sidebar is a glass panel with the same radius as cards, inset 16px from the left/top/bottom edges.
- Main area scrolls; sidebar and top nav stay fixed.
- Content max-width 1680px; centred beyond that.

---

## 5. Component specs (build each as its own component in `src/components/glass/`)

### 5.1 TopNav (`DSN-05`)
- Left: logo square 28×28, radius 8px, `--accent-brand`, white glyph; then app name "ICPilot" 14px 500.
- Tabs: **Project · Edit · Components · Share · Help** in reference position. For ICPilot: `Project` (workspace), `Leads`, `Outreach`, `Analytics`, `Share`, `Help`. Active tab: `--ink-900` 500 with 2px underline-less style (just darker weight); inactive `--ink-600`. 14px.
- Right: small utility icons (share/export, duplicate) as circle icon buttons 28px.

### 5.2 Sidebar (`DSN-04`) — mirrors reference 4/6 exactly
Top → bottom:
1. **Search field**: glass-chip, placeholder "Search components…", magnifier icon left; beside it a **filter circle button** (funnel icon).
2. **Two tiles** side by side: `Home` (house icon) and `Dashboard` (pie icon, **selected** = brighter glass-solid). Radius `--r-tile`, ~104×64px, icon above label.
3. **Tree** (14px): section header with chevron + title + `+` button on right, e.g. `Dimensions +`.
   - Group `Page` (chevron down) → children indented 20px: `Overview`, `Insights` (**active**: a small dark vertical pill indicator on the left edge, text weight 500, trailing count badge), `Campaigns` (badge), `Documents`.
   - Group `Quarter` (chevron).
   - Group with thumbnail: small 28×28 rounded image (mini chart) + `Metrics` label + `+` on right.
   - Items with chevrons: `Revenue / Order` (+ badge `15+`), `Conversion Rate`, `Visits`, `Unique Visitors` (badge `3`).
   - **Badges**: navy (`--ink-900`) pill, white 11px 600 text, min-width 20px, height 20px.
4. **Bottom icon row** (4 circle buttons 40px, glass-chip): sun (theme), database (data sources), hexagon-gear (settings), info (help).
5. **Feedback pill**: full-width-ish navy pill (`--ink-900`), white 14px text "Provide feedback", height 44px.
6. **Avatar** (circle 36px photo/initials) with a **bell** circle button holding a small red dot (`--accent-red-glow`) when there are pending approvals.

**ICPilot content for the tree** (labels replace data only, structure identical):
```
Home | Dashboard (tiles)
Workspace +
  v Setup
     Product Intake
     ICP & Expansion  [badge: pending suggestions]
  v Discover
     Leads            [badge: new leads]
     Competitors
     Knowledge Graph
  v Engage
     Enrichment
     Outreach Queue   [badge: pending approvals]   <- HITL
     Pipeline
  v Insights
     Analytics
     Agents (Band)
     Usage & Credits
Settings (hexagon icon at bottom)
```
The "Metrics" thumbnail row becomes `Saved views` with a mini sparkline thumbnail.

### 5.3 Page header (`DSN-05`) — reference 6
- Left: circular back button 40px (arrow-left), then page title 20px.
- Centre-right: **filter input** — borderless, placeholder "Drop a filter here (or any component)", magnifier at right, thin 1px bottom hairline only (no box).
- Right: `Refresh ↻` glass pill, then two circle icon buttons (download, share/pin), then **navy context pill** (`--ink-900`, white text, e.g. "Demo Data · Sample ⌄") with an `✕` circle next to it. The pill shows the **active dataset/ICP version**; for sample data it reads `SAMPLE DATA` (`ADD-02`).

### 5.4 Pinned-view folder cards (`DSN-06`) — reference 3
- Row height ~120px. Elements: `+` tile (square-ish glass-tile 88×88, "+" icon centred), then folder cards (≈ 220×110), then a **dashed drop zone** (1.5px dashed `--ink-200`, radius 24px, text "Drag a segment here (or any other component)" 13px `--ink-600`).
- **Folder card**: a glass card shaped like a folder front. 2–3 small **preview sheets** (mini white cards showing a tiny metric/chart) peek out of the top edge, slightly rotated (−3°, +2°) and overlapped. Front glass panel covers the lower ~65%. Contains title (20px) + sub-label ("In Last 7 days") + a `…` circle button bottom-right.
- **Glow**: soft coloured radial glow at the card's bottom-left behind the glass: first card purple (`--accent-purple-glow`), second red (`--accent-red-glow`). Opacity ~0.5, blur 24px. Extra cards cycle: purple, red, then neutral.
- ICPilot content: `Pipeline Overview — In Last 7 days`, `Conversion Rate — In Last 7 days`. Clicking opens a saved view. `+` creates a new saved view (stores current filters). Dropping a filter chip into the drop zone saves it as a segment (`dnd-kit`).

### 5.5 Hero metric card (`DSN-07`) — reference 1
- Glass card, padding 24px. Title (18px 500) + sub-label ("In Last 7 days") top-left.
- Label above number (13px `--ink-600`), hero number 56px/300.
- **Sub-stat grid 2×2** under it: each = label (13px `--ink-600`) over value (28px/300). Order: row1 left/right, row2 left/right.
- ICPilot mapping: label "Leads matched: Day" → hero = total qualified leads; sub-stats: **Avg. time to reply** `3m 42s`-style, **Reply rate** `37.6%`, **Touches per lead** `5.2`, **Re-engaged** count. (Values from real data; never hardcoded.)

### 5.6 Filter chips (`DSN-08`) — reference 2/3
- Row of glass-chips: text + small chevron-down. Height 40px, padding 0 16px, gap 12px.
- ICPilot chips: `Industry ⌄`, `Channel ⌄`, `Tier ⌄`, `Status ⌄` (replaces Device Type / Traffic Source / Geolocation / Status).
- Opens Radix `Popover` with glass panel + checkbox list. Active filter = chip uses `--glass-3` and shows a count.

### 5.7 Hatched band chart (`DSN-09`) — reference 2
- Plot area on glass card; no border box.
- **Two series** over time: **upper** dots dark navy (`--chart-dot-dark`, r=3.5px), **lower** dots grey (`--chart-dot-grey`, r=3.5px). Between them a **diagonal hatch fill** (45°, 1px lines, 6px gap, `--chart-hatch`). No solid lines connecting dots (dots only) — the hatch band's edge follows the dot polylines.
- **Dashed horizontal gridlines** (`--chart-grid`, dash 6 8), 3–4 lines. Subtle lighter vertical bands (alternating columns of white 8% opacity) behind date columns.
- X labels: `17 Sep`, `18 Sep`… 12px `--ink-600`, under the plot. Y labels at right: `20,000 / 15,000 / 10,000 / 0`.
- **Glass callout** floating over the chart: radius 20px, `--glass-3`, shows `+24%` (24px 300) + caption "Unique visitors per 7 days" (13px) + `✕` close. ICPilot caption: `Leads contacted vs 7 days ago`.
- ICPilot data: upper = leads contacted, lower = leads replied (or any two real series).
- Implement with plain SVG (not a chart lib) so the hatch pattern is exact. Tooltip on dot hover: glass-3 panel.

### 5.8 Gauge card (`DSN-10`) — reference 6 bottom
- Large semi-circular ring (thin 2px track + 6px dark navy progress arc), centred big number (44px/300) beneath a small **navy delta pill** (`+7.9%`, white 11px) sitting on the arc's top.
- ICPilot use: **Credit balance used** or **Fit score of selected lead**. Value is real.

### 5.9 Thin bar chart card (`DSN-11`) — reference 5/6
- Title "Page Views" + sub-label "Per minutes". Y labels `150k / 50k` style at left. Bars: **vertical, 6px wide, dark navy**, rounded top, height by value; alternating bars slightly lighter; gap ~20px. A glass callout `+0.8% Unique visitors per 1 min` overlaps the right edge.
- ICPilot use: **Outreach sent per day/hour**.

### 5.10 Floating dock (`DSN-12`) — reference 5/6
- Glass pill, bottom-centre of main area, floating (fixed, 24px from bottom). 4 icon buttons 40px: **grid, monitor, card, chart**. Selected = `--glass-3` white circle with navy icon. Others: icon `--ink-600`.
- ICPilot meaning: view switcher for the current page → `Cards`, `Board (Kanban)`, `Table`, `Chart`. State in URL (`?view=`).

### 5.11 Buttons, icons, badges (`DSN-13`)
- **Circle icon button**: 36–40px, `glass-chip`, icon 18px stroke 1.6 `--ink-900`. Hover: `--glass-3`.
- **Navy pill button**: `--ink-900` bg, white 14px text, height 44px (36px compact), radius 999px. Hover: `--ink-800`.
- **Glass pill (Refresh)**: glass-chip with text + icon.
- **Badge**: see 5.2.
- Icons: **Lucide** (stroke style matches the reference). No emoji.

### 5.12 Data grid — the "dense spreadsheet" half (`UI-01`, `UI-02`)
The references are airy; the spec demands a dense operating surface. Reconcile like this, and do not go beyond it:
- A grid lives **inside** a glass card (one blur layer only).
- Row height 40px, text 13px, header 12px/500 `--ink-600`, sticky header, 1px row hairlines `rgba(23,48,65,0.08)`, hover row `rgba(255,255,255,0.45)`.
- Numbers right-aligned, tabular. Score column shows a compact bar + number. Tier shows pill `A` / `B` / `C`.
- **Inline enrichment indicator** cell: small dot + label — `● verified` (`--ok`), `● unverified` (`--warn`), `● risky` (`--warn`), `● not found` (`--ink-400`), `○ not configured` (`--ink-400`), `● error` (`--bad`) + confidence `0.92` in `--ink-600`.
- **Citation chip** in lead rows: `[1]` opens popover with quote, URL, fetched time, `verified ✓`.
- Virtualised (`@tanstack/react-virtual`) above 200 rows.

### 5.13 Kanban (`PIP-01`)
- 5 glass columns (`Matched`, `Contacted`, `Replied`, `Meeting Booked`, `Closed`), Closed has two sub-lanes (`Won`, `Lost`). Lead cards = glass-tile with company, person, tier pill, score, **actor chip** (`AI` navy / `Rep` outlined), time-ago. Drag with `dnd-kit`; keyboard accessible.

### 5.14 HITL queue (`OUT-04`)
- Split view: left list of pending messages (glass-tile rows with channel icon, lead, step #, risk flags); right detail glass card with editable body (signals highlighted with soft yellow underline, hover shows source), source links list, buttons: **Approve** (navy pill), **Edit & Approve** (glass pill), **Reject** (glass pill, danger text; asks reason).

### 5.15 Agent Thinking drawer (`UI-03`)
- Right drawer 380px, glass, toggled by a button in the page header (brain/sparkle icon) and the `Agents` sidebar item. Collapsed = 40px glass tab at right edge with a live dot when events are streaming.
- Event rows: time, type icon, one-line text; expandable for details (Cypher query, URL, raw citation). Types: `reason`, `search`, `fetch`, `cite_ok`, `cite_reject`, `graph`, `enrich`, `validate`, `error`.
- Footer shows tokens + cost for the current run (`BND-07` extends it).

---

## 6. Screens (every page uses the same shell)

| Route | Hero / main content |
|---|---|
| `/` (Home) | Welcome, pipeline summary, quick actions, pinned views |
| `/dashboard` | Hero metric card, hatched chart, gauge, bars, pinned view cards |
| `/intake` | Chat left, structured form right, "missing info" checklist |
| `/icp` | ICP field cards with confidence rings, expansion suggestion cards with Accept/Adjust/Dismiss |
| `/leads` | Dense grid + dock views (Table/Cards/Board), filter chips, Excluded toggle |
| `/leads/[id]` | Lead detail: score breakdown, citations, signals, enrichment, messages, audit timeline |
| `/competitors` | Category lanes (direct/indirect/alternative), comparison panel |
| `/graph` | Force-graph canvas, path finder, saved queries |
| `/enrichment` | Provider status cards, waterfall results grid |
| `/outreach` | HITL queue (5.14), sequence editor |
| `/pipeline` | Kanban (5.13) |
| `/analytics` | Funnel, segment filters, A/B table, hatched chart |
| `/agents` | Band orchestration view: message stream, tokens, cost, approval cards |
| `/usage` | Flexprice credits, events, per-lead cost ledger |
| `/settings` | Founder profile (for warm intros), integrations status, dispatch mode, thresholds |
| `/login` | Glass card on gradient, "Use demo account" button |

---

## 7. States and interaction (`DSN-14`)

- **Hover**: glass brightens one step (`--glass-2` → `--glass-3`), 120ms. Cards lift `translateY(-2px)` with `--shadow-float`.
- **Active/pressed**: scale 0.98.
- **Focus-visible**: 2px ring `--ink-900` with 2px white offset. Never remove outlines.
- **Loading**: skeleton shimmer in glass shapes (white 40% sweep), never spinners over cards. Buttons show inline spinner.
- **Empty**: glass card with a thin line icon, one sentence, one navy pill action.
- **Error**: glass card with `--bad` icon, plain message, `Retry`. Never raw stack traces.
- **Transitions**: 220ms `--ease`. Page content fades up 8px. Respect `prefers-reduced-motion`.
- Panels (popover, dialog, drawer) = glass with `--shadow-float`, Radix primitives.

## 8. Icons and imagery
- Lucide icons, 18px default, stroke 1.6.
- Avatar: initials on `--ink-900` if no photo.
- Folder-card preview sheets use live mini-renders (tiny metric + mini bars), not screenshots.

## 9. Responsive (`DSN-15`)
- ≥1440: as designed. 1280–1439: sidebar 232px, grid gaps 12px.
- 1024–1279: sidebar collapses to icon rail (72px) with tooltips.
- 768–1023: sidebar becomes overlay drawer; cards stack 1 column; dock stays.
- <768: not designed; show a notice "Best viewed on desktop" but must not crash.

## 10. Accessibility rules
- Min text contrast 4.5:1 for anything informational (use `--ink-600` or darker on glass).
- `--ink-400` is for **decorative** text only (e.g. axis hints) — see 12.1.
- All interactive elements keyboard reachable; Radix handles focus management.
- Charts have a data table alternative (visually hidden or toggled with dock "Table").

## 11. Design acceptance checklist (agent must tick every item in the P1 report, with a screenshot)
- [ ] Gradient background matches tokens (slate-blue top → mint bottom)
- [ ] Sidebar identical in structure to ref 4/6 (search+filter, 2 tiles, tree, badges, active pill, bottom icons, navy pill, avatar+bell)
- [ ] Top nav + page header identical to ref 6
- [ ] Folder cards with peeking sheets, + tile, dashed drop zone (ref 3)
- [ ] Hero card thin numerals + 2×2 sub-stats (ref 1)
- [ ] Filter chips with chevrons (ref 2)
- [ ] Hatched band chart, two dotted series, dashed grid, glass callout (ref 2)
- [ ] Gauge with delta pill (ref 6)
- [ ] Thin bar chart (ref 5/6)
- [ ] Floating dock with 4 icons (ref 5/6)
- [ ] Font Urbanist, weights 300/400/500/600, no italics
- [ ] No mockup chrome (no mac window, no monitor, no skew)
- [ ] Screenshot of `/design-preview` at 1440px attached and compared side by side to ref 6

## 12. Known deviations (declared, not hidden)

1. **12.1 Contrast.** The reference uses very light grey labels (~2.5:1). We use `--ink-600` for informational labels. Layout, size, and weight are unchanged. To get the pure reference look, change only `--ink-600` to `--ink-400` in tokens.
2. **12.2 Density.** Data grids are denser than the reference cards because the spec asks for a spreadsheet-style surface (5.12).
3. **12.3 Colours.** Only the two glows (purple, red) and the brand square use colour; both sampled from references. The reference shows no other chromatic colours.
4. **12.4 Font.** Matched by eye (section 2).
