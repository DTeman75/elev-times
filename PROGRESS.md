# Project state & handoff — Shabbat elevator cycle comparison

A resume point for picking this up later. Pairs with `README.md` (the user-facing description)
and `index.html` (the whole tool — single self-contained file, no dependencies).

## What it is
A single-file interactive tool comparing how different automatic **Shabbat-elevator cycles**
affect residents, by floor and direction. Opens by double-clicking `index.html`, or served via
`.claude/launch.json` (python `http.server` on :8755) for the preview workflow.

Cycles compared (6 total; 3 enabled by default):
`Express↑/stop↓`, `Stop↑/express↓`, `Up evens/down odds` (default on), plus
`Up odds/down evens`, `Express↑(mid stop)/stop↓`, `Stop↑/express↓(mid stop)`.

## The model, in four layers
1. **Cycle schedule.** Each cycle is a list of stops with dwells; a timeline of door-open
   events is built. Travel between consecutive stops covering `N` floors =
   `localStep + (N−1)·expressStep` (two-regime: a 1-floor hop costs `localStep`, each extra
   cruised floor costs `expressStep`). Hits the user's measured numbers (1-floor ≈ 6 s,
   lobby↔top express ≈ 30 s).
2. **Leg times referenced to the cycle clock** (t=0 = lobby doors open). `Lobby→Floor` =
   time to doors-open at the floor (includes initial lobby dwell); `Floor→Lobby` = from the
   floor's stop to the next lobby open. For pure elevator, the two sum to one full cycle.
3. **Effective time (the "with stairs" case).** `min(elevator time, stair effort)` — labelled
   **"effective time"** in the UI (the internal code key is still `'burden'`).
   Stair effort is **perceived**, not clock time: climbing `climbEffort(F) = climbC·F^1.3`
   (power; exponential `climbC·e^{k(F−1)}` alternative), descending `descentEffort = descentC·F`.
   Consequence: low floors escape via stairs; **upper floors going up are captive** (climbing
   14 flights ≈ 31 min of effort), so a slow ascent hits them fully. Descent is escapable for
   everyone → nearly cycle-independent. Basis = Borg RPE literature (see Notes & references,
   part A, in the tool).
4. **Overall (urgency-weighted).** Ascent is valued more than descent (a slow ascent strands
   you outside your home; a slow descent you wait out at home). Weight `r` (default 3:1):
   `overall = w_up·up + w_down·down`, `w_up=r/(r+1)`. A **scoreboard** ranks cycles by the mean
   over the floor cases. Rationale written up in Notes & references, part B.

## Current defaults (calibrated to the user's real ~14-storey building)
- floors **14**, **floor 1 above the lobby** (lobby/ground separate → positions 0..14, top=14)
- dwell/floor **30 s**, lobby **70 s**, top **45 s**
- travel: 1-floor hop **6 s**, each added floor **1.85 s**  → express lobby↔top ≈ 0:30
- climb model **Power**, exponent **1.3**, 1-flight effort **60 s**; descent **8 s/flight**
- ascent urgency **3:1**
- Express↑/stop↓ cycle period ≈ **10:19** (matches observed 10.25 min)
- 5 representative floors = positions **2, 5, 8, 11, 14**; **middle = Fl 8** (shaded everywhere)
- default anchor (for differences) = Express↑/stop↓

## Layout / view order (top → bottom)
0. **Summary** panel (leads the results column): **Objective**, then a **Conclusion** in a
   highlighted box (light accent background, accent left-border), then **Method** (empirical
   F^1.3 + subjective urgency + sources), a **How to use this tool** section (pick anchor +
   cycles, tweak the left panel, results appear below), and a guide to the illustrations.
   Copy is benefit-led (see Encoding conventions).
1. **Cycles to compare** legend — checkboxes to show, radio to pick the difference anchor.
2. **Overall — urgency-weighted** (the conclusion): scoreboard, then by-floor "overall time
   cost" table + difference table + bar chart + difference chart.
3. **With stairs** — "effective time", Floor→Lobby then Lobby→Floor, tables then graphs.
4. **No stairs** — elevator-only baseline, same structure.
5. **Cycle timeline** (position vs time for the selected cycle).
6. **Notes & references** — part A (F^1.3 / Borg) + part B (urgency rationale); its own print page.

Also: A4-landscape print/PDF (button top-right) with page breaks per section; responsive
(stacks to one column on phones).

## Encoding conventions (important, settled after iteration)
- **Case colors** = blue `#2563eb`, orange `#ea580c`, purple `#7c3aed`, pink `#db2777`,
  cyan `#0891b2`, amber `#ca8a04`. **No red/green among cases** (deliberately).
- **Red/green = semantic only**: green = best/lowest, red = worst/highest (tables), and
  −better/+worse in difference *tables*.
- **Difference charts**: bar colored by its **case** (matches legend); **direction** encodes
  quality — above the zero line = worse than anchor, below = better (`↑ worse`/`↓ better`,
  signed axis). No red/green, no thin linking tags.
- Charts have vertical group gridlines; the middle floor (Fl 8) slice is shaded as an anchor.
- **Copy is benefit-led, not burden-led** (user preference, saved to memory
  `framing-benefits-not-burdens`): user-facing text says "effective time" / "overall time cost" /
  "most beneficial cycle"; the word "burden" now survives only as an internal code key/comment.

## Key decisions & why (so they aren't re-litigated)
- **Round-trip = full cycle** for elevator-only — fixed an earlier bug where random-arrival
  averaging let a single leg exceed the cycle. Times are now scheduled, not averaged.
- **Removed** the "Round trip" metric and the metric selector; both directions shown together.
- **Removed** the "Lobby — full cycle" row from tables/charts — it was an outlier that
  compressed the floor spreads; `niceMax` was also refined (steps 1,1.5,2,3,4,5,6,8,10).
- **Effort, not choice**: the "with stairs" model is the *difficulty of the best option*
  (`min`), not a prediction of who walks. The user rejected the "captive/probability" framing.
- **`F^1.3`** chosen from the user's shared Borg-RPE analysis (exponential alt available).

## Open ideas / possible next steps (not built)
- Weight the building score by **residents per floor** (population), not just a flat mean.
- A **logistic propensity** model (probability of taking stairs) as an alternative lens — was
  discussed and illustrated, but the deterministic `min()` burden was chosen instead.
- **Partial walks** (walk a flight or two then ride) were simplified out; could be reintroduced.
- A **wait-aversion** multiplier (waiting feels worse than moving) — discussed, not added.
- Possibly de-emphasize/collapse the near-flat descent difference panels.

## Hosting
- Target: **IONOS / Plesk**, as a path under **dteman.com** → **`https://dteman.com/elevator/`**.
- Confirmed setup: dteman.com is Plesk-managed; its document root is **`httpdocs`** (holds the
  live `index.html` and path-folders like `releaseari`, `shul-dev`). Deploy = in Plesk File
  Manager, create a folder **`elevator`** inside `httpdocs` and upload **`index.html`** into it
  (that one file is the whole app). Full walkthrough + subdomain alternative in `DEPLOY.md`.
- One self-contained `index.html` (~51 KB, no external deps); `<title>`, description, OG/Twitter
  meta, and an SVG favicon are set. HTTPS comes from dteman.com's existing certificate.
- I cannot deploy to the user's account (no creds; outward-facing) — `DEPLOY.md` is their hand-off.
- **Status: not yet uploaded** as of last session (user was about to do it via Plesk).

## Bilingual (Hebrew + English) — PENDING / not started
- User is interested ("possibly … Hebrew and English"). English-first for the initial host.
- Plan when greenlit: add a language toggle, externalize all strings into an `{en, he}` dict,
  RTL layout for Hebrew (page chrome flips; numeric charts stay LTR), translate UI + Summary +
  Notes (draft Hebrew for user review). Stays a single `index.html`.

## Working notes
- Verify via the preview MCP (`.claude/launch.json` server name `elev-times`).
- Known quirk: `preview_screenshot` returns blank for sections deep down the page; verify
  those via DOM (`preview_eval`) instead.
- The tool stores all state in a single `state` object; `render()` is debounced and redraws
  everything. Strategy colors come from one `PALETTE` array.
