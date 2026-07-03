# Shabbat Elevator Cycle Comparison

An offline, single-file tool (`index.html`) that compares the resident wait+ride times
produced by different automatic Shabbat-elevator cycles used in Israeli buildings.

Open `index.html` in any browser — no build, no server, no internet required.

## Output targets / layout

The on-screen dashboard is designed for a **~15″ widescreen**: settings sidebar on the
left, results table and the two comparison charts laid out side-by-side.

- **A4 landscape print / PDF** — use the *"Save wide view as PDF"* button (top-right) or the
  browser's Print → "Save as PDF". A dedicated print stylesheet (`@page size:A4 landscape`)
  reflows the full wide dashboard onto A4-landscape pages, replaces the interactive controls
  with a compact settings summary, and forces chart colors to print.
- **Phone** — below ~760px the layout collapses to a **single stacked column**, one view at a
  time. To see the full wide dashboard on a phone, tap the PDF button to generate the
  A4-landscape PDF.

## What it compares

Cycle strategies (toggle any subset on):

- **Express ↑, stop-every ↓** — express lobby→top, then descend stopping at every floor.
- **Stop-every ↑, express ↓** — stop at every floor up, then express top→lobby.
- **Up evens, down odds** / **Up odds, down evens** — stop at alternating floors each leg.
- **+ mid stop** variants — add one stop at the middle floor during the express leg.

For the **lobby** plus five representative floors (2 above the lobby, the top floor, and
three evenly spaced between), it reports both directions side by side:

- **Floor → Lobby**
- **Lobby → Floor**

The **Lobby** entry is special: its value is the **full cycle time** (lobby doors open →
next time the lobby doors open), giving a reference for how long one complete cycle takes.

### Two cases, shown together

Results are always shown for **both** cases, stacked vertically:

- **No stairs** (top) — captive to the elevator (e.g. non-ambulatory). Pure elevator times.
- **With stairs — effective burden** (middle) — each leg is the *best of two bad options*:
  wait for the elevator, or take the stairs. The stair option's cost is the **perceived
  effort** of the climb/descent, not just its clock time (see below).
- **Overall — urgency-weighted** (bottom) — the two directions blended into one number per
  floor, and a per-cycle scoreboard (see *Ascent urgency* below).

Each set has its own results tables and absolute + difference charts. The cycle timeline at
the bottom is shared (the schedule is identical for all).

## The timing model

Positions are numbered from the lobby (`0`) to the top floor (`M`). One cycle is a fixed
schedule of door-open events; the elevator pauses for the configured dwell at each stop
(separate dwells for intermediate floors, the lobby, and the top).

**Travel uses two regimes.** A move between two consecutive stops covering `N` floors takes
`localStep + (N − 1) · expressStep`:

- a **1-floor hop** (with its accel/decel and stop) costs `localStep` (default 6 s);
- each **additional floor** crossed while already cruising costs `expressStep` (default
  1.85 s).

So a nonstop express run averages toward `expressStep` per floor: lobby↔top over 14 floors =
`6 + 13 · 1.85 ≈ 30 s` (~2.1 s/floor). This matches measured behaviour — short hops are
dominated by the start/stop, long express runs by the cruise speed. (Even/odd cycles, which
skip a floor between stops, use the same formula for their 2-floor hops.)

**Times are measured on the elevator's cycle clock, with `t = 0` the moment the lobby doors
open.**

- **Lobby → Floor** = from the lobby doors opening (you board) until the doors open at your
  floor. The initial lobby dwell is part of this leg.
- **Floor → Lobby** = from the doors opening at your floor (you board) until the doors next
  open at the lobby.

In the **no-stairs** case these two legs add up to **exactly one full cycle**, so no single
elevator trip exceeds the cycle (which is why every floor's *Lobby* row equals the full cycle).

### Effective burden (the "with stairs" case)

The elevator is not the only option — but the stairs are only a *viable* alternative for some
floors and directions. The cost of the stairs is the **perceived effort**, which is sharply
asymmetric:

- **Climbing up:** effort grows much faster than linearly. Following published Borg
  rating-of-perceived-exertion data, the default is a power law `climbEffort(F) = climbC · F^1.3`
  (with an exponential `climbC · e^{k·(F−1)}` alternative). By the upper floors this is
  enormous — climbing 14 flights ≈ 31 minutes of effort-equivalent — so it is effectively
  impossible and the resident is left with the elevator.
- **Descending down:** cheap and roughly linear, `descentEffort(F) = descentC · F`.

The basis for the `F^1.3` model — the Borg rating-of-perceived-exertion data, the
relative-difficulty table, the exponential alternative, the caveats, and the source citations
— is written up in the **Notes & references** section at the bottom of the tool. It prints as
its own page, the "with stairs" heading links to it (`[basis ↓]`), and the ascent chart cites
it. Helpful chart annotations also flag that the descent panels are near-flat by design
(escapable on foot) and that the ascent is the decisive direction.

For each leg the **effective burden = min(elevator time, stair effort)**. The consequence:

- **Low floors** are insulated — a 2-flight climb is a real fallback, so a slow cycle barely
  hurts them.
- **Upper floors going up** are fully exposed — there is no viable climb, so they absorb the
  entire ascent time. A slow floor-by-floor ascent hits floor 14 far harder than floor 4,
  exactly because `F^1.3` makes the escape impossible.
- **Going down** is escapable for nearly everyone, so the cycle's descent performance is
  almost irrelevant in this case — the comparison becomes a contest about the **ascent**.

The difference charts then show each cycle's real benefit/loss per floor: e.g. for the top
floor going up, the spread between an express-up cycle and a stop-every-up cycle is several
minutes, while for the 2nd floor it is seconds.

### Ascent urgency (the "Overall" view)

Delay hurts asymmetrically by direction. A slow **ascent** strands you outside your home —
urgent (and on a high floor there's no stair fallback). A slow **descent** just means waiting
in your own apartment, on your own schedule — low stress. So a minute saved going up is worth
more than a minute going down.

A single **ascent-urgency** weight `r` (default 3, i.e. up is valued 3× down) captures this.
With `w_up = r/(r+1)` and `w_down = 1/(r+1)`, the **Overall** view combines each floor's
effective burdens into `w_up·up + w_down·down`, and a **scoreboard** ranks the cycles by the
mean across the floor cases. Because the captive upper floors dominate the ascent term,
express-up tends to win, and its lead widens as you turn the urgency up. The fuller subjective
rationale (the "need a bathroom" use case, and why a medical emergency is out of scope) is in
part B of the **Notes & references** section at the bottom of the tool.

### Adjustable settings (with defaults)

| Setting | Default |
|---|---|
| Number of floors | 14 |
| Floor 1 = Lobby, or one above the Lobby | Floor 1 **above** the Lobby |
| Dwell at each (intermediate) floor | 30 s |
| Dwell at lobby | 70 s |
| Dwell at top floor | 45 s |
| Travel — 1-floor hop (with stop) | 6 s |
| Travel — each added floor (cruise) | 1.85 s |
| Climb difficulty model | Power (`F^1.3`); Exp alternative |
| Climbing — 1-flight effort (`climbC`) | 60 s |
| Climbing — difficulty exponent | 1.3 |
| Descending — per-flight effort (`descentC`) | 8 s |
| Ascent urgency (value of up vs down) | 3 : 1 |

Defaults are calibrated to a real, measured 14-storey building (lobby/ground separate, 14
floors above it). For the **express-up / stop-every-down** cycle the period works out to:

```
lobby dwell      70 s
express up        6 + 13·1.85  ≈ 30 s   (lobby → floor 14, nonstop)
top dwell        45 s
descent          13 stops·30 + 14 hops·6 = 474 s
                 ----------------------------------
total            ≈ 619 s  =  10:19   (≈ the observed 10.25 min)
```

> Note: the climbing-effort model (`climbC`, exponent/growth) and `descentC` are not in the
> original spec; they are required to model the stairs trade-off realistically and affect only
> the with-stairs set. The no-stairs set is always elevator-only. All effort values are in
> seconds-equivalent so they compare directly against the elevator times.

## Views

Each of the two sets (no-stairs on top, with-stairs below) lays out both directions in the
same sequence — **Floor → Lobby first, then Lobby → Floor** — as tables, then as graphs:

1. **Tables** — for each direction, a *totals* table and a *difference-vs-anchor* table
   (floors × cycles). In totals, the lowest cycle per floor is green, highest red; in
   difference tables, −better is green and +worse is red, with the anchor column shown as "—".
2. **Graphs** — for each direction, a grouped *absolute* bar chart and a *difference* chart.
   In the **difference charts** each bar is colored by its **cycle** (matching the legend),
   and its position carries the meaning: **above the zero line = worse than the anchor, below
   = better** (`↑ worse` / `↓ better`). Colour is reserved for identity, direction for
   better/worse — so red and green never do double duty.

The third view, **Overall**, leads with a **scoreboard** (each enabled cycle's single overall
score, ranked best-first with a proportional bar) and then a combined-burden table + difference
table + bar chart + difference chart, all on one blended metric driven by the ascent-urgency
slider.

All charts have vertical gridlines separating the floor groups, and the **middle floor**
(the 3rd of the five floor cases) is shaded across every table row and chart slice as a
visual anchor for comparison.

A shared **Cycle timeline** at the bottom shows elevator position over one full cycle (flat =
dwell, diagonal = travel) for the selected cycle.

At the top, the **Cycles to compare** legend lets you check which cycles to show and pick the
**difference anchor** with a radio button (the anchor's chip is highlighted). The anchor
choice applies to both sets and both directions.

## Sanity check

The no-stairs model is self-consistent: for every cycle and every floor, **Floor→Lobby +
Lobby→Floor = one full cycle**, so each floor's elevator round-trip equals the *Lobby*
(full-cycle) value and no single elevator leg ever exceeds the cycle. In the with-stairs
(effective-burden) set this no longer holds — a leg is capped by the stair effort, so burdens
can be far below the elevator time (cheap descent, or a short climb on a low floor).
