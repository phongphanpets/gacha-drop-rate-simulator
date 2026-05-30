# Gacha Drop Rate Simulator

> Single-file HTML gacha simulator with rate configuration, item pool management, Monte Carlo player POV analysis, and 7-file CSV export pipeline. Built for the AI Workflow Exam — Part 2.

🔗 **Live Demo:** [https://YOUR_USERNAME.github.io/gacha-drop-rate-simulator/gacha-simulator.html](https://YOUR_USERNAME.github.io/gacha-drop-rate-simulator/gacha-simulator.html)

---

## 📂 Repository Contents

| File | Description |
|---|---|
| `gacha-simulator.html` | The main deliverable — single-file HTML, vanilla JS/CSS, no build step |
| `PROMPT_HISTORY.md` | Complete AI conversation log used to design and build the simulator |
| `REFLECTION.md` | Short reflection on the process, decisions, and AI collaboration |
| `README.md` | This file |

---

## 🚀 How to Run

**Option 1 — Live Demo (recommended for reviewers)**
Open the live demo link above. Works in any modern browser.

**Option 2 — Run locally**
1. Download `gacha-simulator.html`
2. Double-click the file — it opens in your default browser
3. No installation, no backend, no build required

Tested on Chrome, Edge, Safari, and Firefox (latest versions).

---

## ✨ Features

### Must-Have (Rubric: 82 pts)

- ✅ **Rate Settings** — SSR / SR / R / N with live sum validation (must equal 100%)
- ✅ **Item Pool Management** — add / edit / delete items, dropdown rarity selection, pool validation
- ✅ **Single Simulation** — rarity-first random logic (`Math.random()`), no hardcoded results
- ✅ **Result Display** — summary metrics, configured-vs-actual table, item count breakdown, full pull log
- ✅ **CSV Export** — UTF-8 BOM (Excel-safe), proper escaping
- ✅ **Input Validation** — 9 rules covering rates, pool, rolls, budget, simulation count, free-roll config

### Advanced (Rubric: 25 pts)

- ✅ **Player POV Monte Carlo** — budget → paid + free rolls → 1,000+ simulation runs
- ✅ **Free Roll Rule** — calculated from paid rolls only (no recursive free rolls)
- ✅ **Player POV Output** — chance ≥1 SSR, chance 0 SSR, averages, P10/P50/P90, best/worst, Thai insight text

### Bonus (Rubric: 20 pts)

- ✅ **Hard Pity** — guaranteed SSR every N rolls (counter resets on SSR)
- ✅ **Soft Pity (Rate-Up)** — TOSM-style: after threshold, sub-pool excludes N + R (proportional re-normalization)
- ✅ **History** — last 10 simulations, click to clear
- ✅ **Visualizations** — dynamic SSR distribution histogram (adaptive buckets), live rate-validation indicators
- ✅ **Advanced CSV** — 7 separate clean tabular files (vs. multi-section CSV anti-pattern)
- ✅ **Polish** — rate presets, budget presets, progress indicator, compare Pity ON vs OFF, soft-pity rate preview

---

## 📊 The 7-File CSV Export Pipeline

Designed so analysts can plug each file into Excel pivot tables, pandas, R, or Power Query without manual cleanup.

### Single Simulation (3 files)

| File | Columns | Use Case |
|---|---|---|
| `gacha-single-pulls-DATE.csv` | roll_no, rarity, item_name, pity_source, cost_thb, cumulative_cost_thb | Raw pull log — feed into pivot tables |
| `gacha-single-items-DATE.csv` | item_name, rarity, configured_rate_pct, times_obtained, was_obtained, pct_of_pulls, expected_count, diff_from_expected | Per-item analysis — find under/over-rolled items |
| `gacha-single-summary-DATE.csv` | category, metric, value, unit, notes | Overview — profit, player value, rates, config |

### Player POV — Monte Carlo (4 files)

| File | Columns | Use Case |
|---|---|---|
| `gacha-player-summary-DATE.csv` | category, metric, value, unit, notes | Comprehensive KPIs across revenue, player value, distribution, setup |
| `gacha-player-items-DATE.csv` | item_name, rarity, configured_rate_pct, total_obtained, sims_obtained_in, sims_obtained_pct, avg_per_sim, max_in_one_sim, extra_copies, dead_item | Per-item economics across all simulations |
| `gacha-player-runs-DATE.csv` | sim_id, ssr_count, sr_count, r_count, n_count, unique_ssr, dupe_ssr, unique_total, first_ssr_roll, hard_pity_triggered, soft_pity_pulls, collection_completeness_pct, cost_per_ssr_thb | Raw per-simulation data — one row per sim |
| `gacha-player-histogram-DATE.csv` | ssr_count_bucket, frequency, percentage, cumulative_pct + rolls-needed table | Distribution shape — feed into chart tools |

---

## 🎯 Key Design Decisions

### Why rarity-first sampling?
The rubric explicitly requires *"สุ่ม rarity ก่อนตาม rate แล้วค่อยสุ่ม item ใน rarity นั้น"*. Direct item-level sampling would couple the rate config to pool size — adding one SR item would lower every SR item's effective rate. Rarity-first decouples them, which is also how Genshin / TOSM / HSR work in production.

### Why 7 separate CSVs instead of one multi-section CSV?
Multi-section CSV breaks Excel filters, pandas `read_csv`, Power Query, and pivot tables. RFC 4180 and industry norm is *one file = one table*. Each export answers a single analyst question with a clean tabular shape.

### Why a separate "Soft Pity" toggle (not just Hard Pity)?
These are mechanically different. Hard pity *guarantees* an SSR at threshold (Genshin's pity at 90). Soft pity *boosts effective rate* by excluding N/R from the sub-pool (TOSM-style). Designers tune them independently and need to A/B test each.

### Why dynamic histogram buckets?
A whale with ฿30,000 averages 10+ SSR — a fixed 0/1/2/3/4+ bucket would collapse 95% of outcomes into "4+", hiding the actual distribution shape. The histogram adapts: 6 buckets when max ≤ 5, range-based buckets up to 20, adaptive 6-range when larger.

---

## 🛡️ QA & Validation

This project applied a QA-mindset throughout development. Key checks built into the code:

- Distribution accuracy verified at 100k rolls (±0.5% from configured rate)
- Monte Carlo statistical sanity (empirical ≈ `1 - (1-p)^n`)
- Free roll math: paid rolls drive free rolls, free rolls never produce more free rolls
- Pity counter resets on SSR (both soft and hard)
- Sub-pool re-normalization correct: `1:9 → 10%:90%`
- CSV column-count consistency across every file
- All histograms sum to total sims
- Item totals sum to total rolls across all simulations
- No `localStorage` (works under `file://`)
- UTF-8 BOM in every CSV (Thai text + Excel)

A bug was caught during QA — `runSimsAsync()` referenced `budget` outside its scope and would have crashed on the first Monte Carlo run. Fixed before submission.

---

## 🧪 Tech Stack

- **HTML / CSS / JavaScript** — vanilla, single file
- **No frameworks** — no React / Vue / build step
- **No external dependencies** — works offline, no CDN, no `localStorage`
- **No backend** — all simulation runs in the browser
- **CSV via Blob API** — `URL.createObjectURL` + `download` attribute

---

## 📝 Submission Notes

This is a deliverable for the AI Workflow Exam Part 2 (Gacha Drop Rate Simulator).
Built with iterative AI collaboration documented in [`PROMPT_HISTORY.md`](./PROMPT_HISTORY.md).
Process reflection in [`REFLECTION.md`](./REFLECTION.md).
