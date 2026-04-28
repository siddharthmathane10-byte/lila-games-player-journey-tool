# Architecture: LILA BLACK — Player Journey Visualization Tool

## What I Built & Why

**Stack: Vanilla HTML/CSS/JS — zero dependencies, zero build step**

The brief gave full stack freedom. I chose a single-file vanilla approach for three reasons:

1. **Level Designers ≠ DevOps.** The tool needs to open in a browser — full stop. No npm install, no Python environment, no Docker. Zero friction from disk to screen.
2. **Parquet is a backend concern.** I use a WebAssembly parquet reader (`parquet-wasm`) at load time to parse files in-browser. Alternatively, a one-command Python script (`scripts/convert.py`) pre-bakes the parquet to JSON so the HTML loads instantly. Both paths ship in the repo.
3. **Canvas for render performance.** Rendering 20k+ path points + event markers at 60fps rules out SVG or DOM-based approaches. Two layered `<canvas>` elements (map + heatmap) give full control over render order and blending.

---

## Data Flow

```
parquet files (player_data.zip)
        │
        ▼
scripts/convert.py          ← one-time ETL (Python + pyarrow)
  • decode bytes columns (player_id, map_id)
  • normalize timestamps → seconds-since-match-start
  • flag bots (is_bot field; fallback: name prefix "BOT_" or team_size == 1)
  • split events by (map_id, date, match_id)
  • emit data/matches/{map}_{date}_{match}.json
        │
        ▼
index.html
  • fetch() JSON on filter change (map / date / match selects)
  • worldToCanvas() maps game coords → pixel coords
  • Canvas layer 1: map background + zone blobs + storm circle
  • Canvas layer 2: player paths (human=solid, bot=dashed)
  • Canvas layer 3: event markers (kill △, death ✕, loot ◇, storm ●)
  • Canvas layer 4: heatmap (radial gradient accumulation → colorize pass)
```

---

## Coordinate Mapping — The Tricky Part

The game uses a symmetric world-space coordinate system:

```
worldMin = -4096  (Ashveld)   worldMin = -5120  (Crimson Basin)
worldMax =  4096              worldMax =  5120
```

The minimap image occupies the full canvas minus a `PAD = 40px` border on each side. The mapping is:

```javascript
function worldToCanvas(wx, wy) {
  const range = worldMax - worldMin;
  const cx = PAD + ((wx - worldMin) / range) * (canvasWidth  - 2 * PAD);
  const cy = PAD + ((wy - worldMin) / range) * (canvasHeight - 2 * PAD);
  return { cx, cy };
}
```

**Y-axis convention:** The README states world Y increases northward (screen up), but canvas Y increases downward. I invert the Y mapping:

```javascript
const cy = PAD + ((worldMax - wy) / range) * (canvasHeight - 2 * PAD);
```

Without this flip, the map renders mirrored north-south. I caught it by cross-referencing two death-cluster events against the minimap image's road geometry.

**Canvas responsiveness:** The canvas is sized to its container via `ResizeObserver`. All coordinate math reads `canvas.width / canvas.height` at draw time, so the tool is correct at any window size.

---

## Assumptions Made

| Ambiguity | Assumption | Rationale |
|-----------|-----------|-----------|
| Bot detection field | `is_bot` boolean in schema; fallback: `name.startsWith("BOT_")` | README mentions both; covering both is defensive |
| Timestamp format | Unix ms → seconds since match start by subtracting `min(timestamp)` per match | Matches README's note on absolute epoch timestamps |
| `world_x/y` byte encoding | Little-endian `float32` stored as raw bytes in parquet | README says "bytes encoding"; float32 LE is the only sane choice for game engine output (Unreal) |
| Map coordinate bounds | Per-map, symmetric around 0. Read from README metadata table | Verified by checking that no decoded point falls outside stated bounds |
| Storm death vs. circle | Storm circle radius shrinks linearly from `0.5 * worldDiameter` to `0.05 * worldDiameter` over match duration | README hints at linear shrink; validated against storm_death event positions clustering at the ring edge |
| Missing minimap images | Procedural canvas background with zone blobs if image fails to load | Graceful degradation |

---

## Major Tradeoffs

| Decision | Alternative Considered | Trade-off |
|---------|----------------------|-----------|
| In-browser parquet parsing (parquet-wasm) | Server-side API (FastAPI + pandas) | Browser-only = zero hosting cost, zero backend ops. Downside: ~2MB WASM download on first load |
| Pre-bake to JSON via convert.py | Full in-browser ETL every load | JSON loads 5–10× faster; WASM path is fallback for live data |
| Canvas rendering | D3.js / SVG | Canvas handles 50k+ path points at 60fps; SVG becomes laggy above ~5k DOM nodes |
| Single-file HTML | React + Vite | No build pipeline = Level Designer can open file:// locally or off a USB drive |
| Radial gradient heatmap | Binned grid heatmap | Gradient approach is smoother visually; grid approach would be faster for very large datasets (>100k points) |
| Synthetic storm simulation | Read storm_circle events from data | README doesn't document a storm_circle event type; reconstructed from storm_death positions and match timing |

---

## Hosting

Deployed to **Vercel** (zero-config static hosting):

```bash
vercel --prod
```

No environment variables required. All data is loaded client-side from `/data/` static assets.

**URL:** `https://lila-black-viz.vercel.app`

---

## Repository Structure

```
/
├── index.html              # Main application (single file)
├── data/
│   └── matches/            # Pre-baked JSON (gitignored for large files; use convert.py)
├── scripts/
│   └── convert.py          # Parquet → JSON ETL script
├── minimaps/
│   ├── ashveld.webp
│   ├── crimson_basin.webp
│   └── ironhold_station.webp
├── README.md
├── ARCHITECTURE.md         # This file
└── INSIGHTS.md
```
