# CLAUDE.md — The Pour-Over Bench

## What this is

An interactive pour-over coffee brewing guide and calculator covering the
Hario V60 and Chemex, at "full nerd" depth (TDS, extraction yield, agitation,
pulse-pour methods). Built July 2026. A two-page printable PDF reference
(`pour-over-bench-reference.pdf`) is a companion artifact; the web app is the
primary product.

**Live target:** https://zobott-dot.github.io/pour-over-bench/
**Owner:** Dave Zobott (GitHub: zobott-dot)

## About the owner — how to work with Dave

Dave is a builder, not a developer. He has reading fluency in HTML/CSS/JS and
uses it for code review and agency — he will read your diffs, but he won't
write code himself. This means:

- **Explain what you changed and why** in plain language with every unit of
  work. Reinforce understanding; he wants a mentor, not a black box.
- **Small, reviewable commits** with clear messages. One concern per commit.
- **Never assume he'll debug** — if something needs a judgment call, ask.
- Warm but direct communication. No flattery. State confidence levels on
  anything uncertain.

## Architecture

The project currently ships as a single `index.html` (~1,100 lines: CSS,
markup, and an IIFE of vanilla JS). Dave is **explicitly not wed to the
single-file format** — use the best tools for the job. Constraints and
guidance:

- **Deploy target is GitHub Pages (static hosting).** No server, no backend.
  Anything that builds to static files is fair game.
- If you restructure, prefer the simplest architecture that serves the
  feature set. Splitting into `index.html` + `styles.css` + `js/` modules is
  a fine first step. A build tool (Vite) + GitHub Actions Pages deploy is
  justified only if you're adding something that needs it (bundling,
  framework, PWA tooling) — don't add build complexity speculatively.
- Vanilla JS is currently sufficient. Introduce a framework only if a
  feature genuinely demands it, and explain the tradeoff to Dave first.
- No user data leaves the browser. If persistence is added (brew logging),
  use localStorage — this runs as a real site, not a claude.ai artifact.
- Must work well on desktop and mobile (Dave uses both, including iPhone).
  Keep the responsive breakpoint behavior (~820px) intact.

## Design system — do not drift casually

The aesthetic is a **coffee-bag spec sheet**: precise, instrument-like,
label-typography. It is intentional. Tokens live in `:root`:

- Palette: `--paper #EDE9E1` (stoneware), `--card #FBF9F4`, `--ink #26201A`
  (espresso), `--copper #B26A2B` / `--copper-deep #8F5321` (accent),
  `--slate #4E6470`, `--water #7FA8B8`, `--good #4C7A5A`, `--warn #A9502F`.
- Type: **Archivo** (display 800–900 uppercase + body) and **IBM Plex Mono**
  (all numerals, labels, data). Numbers are always mono — that's part of the
  identity.
- Structure: sections are numbered spec-sheet style (01/07 … 07/07) with
  mono eyebrow labels. Signature element: the SVG dripper cross-section that
  switches V60/Chemex shape and animates water level during the timer.
- Quality floor: visible keyboard focus, `prefers-reduced-motion` respected,
  responsive to ~375px.

## Domain invariants — the brewing content is load-bearing

These numbers were verified at build time. Do not change them without
flagging it to Dave as a content decision, not a code decision:

- **Ratio range:** 1:14–1:18, default 1:16.5. Water = dose × ratio.
- **Bloom:** 2–3× dose (default 2.5×), 45 s.
- **V60 continuous (Hoffmann-style):** bloom → 60% of water by 1:15 → 100%
  by 1:45 → drawdown target 2:45–3:30 (~500 g brew). Total timeline 210 s.
- **V60 4:6 (Kasuya):** first 40% of water in 2 pours (balance: standard
  50/50, brighter 62.5/37.5, sweeter 37.5/62.5); last 60% in 2–3 pours
  (strength). Grind coarser than continuous. Pour starts ≈ 0:00 / 0:45 /
  1:30 / 2:10 / 2:40. Cumulative pours must sum exactly to total water.
- **Chemex:** bloom + 3 equal pulse pours; drawdown 3:30–4:30; total 270 s.
- **Grind (µm):** V60 continuous 450–650; V60 4:6 700–850; Chemex 700–900.
  Scale runs 200–1200 µm.
- **Temp by roast (°C):** light 93–96, medium 88–93, dark 82–88.
- **Extraction math:** EY% = TDS% × beverage g ÷ dose g. Beverage estimate =
  water − 2 × dose (grounds absorb ~2× their weight). Ideal windows:
  EY 18–22%, TDS 1.15–1.45%.
- **Units:** metric ↔ US via 1 oz = 28.3495 g; °F = °C × 9/5 + 32. The unit
  toggle must convert *everything*, including the pour table and live timer
  targets. Grind stays in µm in both modes.

If a change touches the schedule builder, re-verify that cumulative pour
weights end exactly at total water for all three structures.

## Feature inventory (as shipped)

1. Recipe calculator — dose ⇄ water (bidirectional), ratio slider, bloom
   slider, pour-structure select, 4:6 balance/strength selects, quick-cup
   buttons (250 g per cup), live brew card, SVG dripper diagram.
2. Pour schedule table — clock windows, cumulative + incremental weights,
   technique notes; regenerates on any input change.
3. Guided timer — 200 ms tick, active-step highlight in the schedule table,
   step chime (WebAudio, wrapped in try/catch), water-fill animation,
   pause/resume/reset.
4. Grind chart — 200–1200 µm scale, marker moves per device/method.
5. Temperature — roast selector snaps slider to window; water chemistry notes.
6. Extraction lab — TDS + beverage weight → EY, zone feedback
   (under/ideal/over, weak/ideal/strong), quadrant-specific advice.
7. Technique & troubleshooting — agitation/pulse/bed-reading notes, six
   symptom → fix cards.

## Phase 2 candidates (discussed, not committed)

- PWA: manifest + service worker + home-screen icon (Dave adds his tools to
  his iPhone home screen).
- Brew logging: save brews (recipe + optional TDS/EY) to localStorage, plot
  them on an interactive brewing control chart, compare over time.
- Wake-lock during the timer so the phone screen stays on mid-brew.
- Shareable recipe URLs (encode state in query params).

Ask Dave to prioritize before starting any of these.

## Smoke-test checklist (run before every commit)

1. Open the site locally; no console errors.
2. Change dose → water, schedule, brew card, and lab beverage estimate all
   update. Change water → dose back-computes.
3. Toggle V60 ⇄ Chemex: pour-structure options swap, diagram shape swaps,
   grind marker moves, schedule regenerates.
4. Toggle units both directions: every weight and temperature converts,
   including quick-cup buttons and timer targets; toggling back is lossless
   (no drift from rounding).
5. Select 4:6: extras panel appears; balance/strength change the schedule;
   final cumulative weight equals total water.
6. Run the timer ≥ one full step change: chime fires, active row highlights,
   water fill rises, pause/resume/reset all work.
7. Narrow the window to ~375px: single column, nothing overflows.
