# Cucunumic 2-Phase Model Rollout — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Roll out the locked 2-phase investment model across the master plan tooling, master plan markdown, investor deck, and GitHub Pages publish — replacing the 4-phase ladder, refreshing screenshots and copy, and locking the new thesis.

**Architecture:** Three sequential phases. **A** refactors the master plan source (Python + markdown) and regenerates screenshots. **B** rewrites the 17-slide investor deck against the spec, dropping one slide (s11-p3) and reflowing the phase narrative. **C** syncs the working copy to GitHub Pages. Each phase commits before the next begins; the deck depends on Phase A's screenshots.

**Tech Stack:** Python 3 + folium + shapely + pyproj (master plan render), Playwright (screenshot capture), HTML/CSS/JS (deck), git + gh CLI (publish). PowerShell on Windows; quote paths with spaces.

**Spec reference:** `C:\Users\Jon Berg\Projects\Cucunumic\docs\superpowers\specs\2026-05-10-cucunumic-2-phase-model-design.md`

---

## Conventions

- **Project roots:**
  - Master plan tooling: `C:\Users\Jon Berg\Projects\earth-engine\`
  - Cucunumic deck repo: `C:\Users\Jon Berg\Projects\Cucunumic\`
  - Desktop deck (working copy): `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html`
- **Verification before edits:** every task starts with a verification command that should *fail* (or show old state) before the edit and *pass* (or show new state) after. This is the TDD discipline applied to content edits.
- **Local servers** (start once at session beginning):
  ```bash
  cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe -m http.server 8765 &
  cd "C:/Users/Jon Berg/Projects/Cucunumic" && python -m http.server 8766 &
  ```
- **Open in browser:** `cmd.exe //c start http://localhost:PORT/file.html` (file:// is blocked).
- **Git push:** ALWAYS use `git -c credential.helper='!gh auth git-credential' push origin master` on this Windows machine.
- **Editing convention:** edit Desktop master first, sync to repo's `baja_wellness_deck.html` after each edit-cluster. Edit `index.html` only at publish moment (Phase C).

---

## Phase A — Master Plan Refactor (3-phase → 2-phase)

### Task A1: Add 2-phase color constants to master plan render

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\master_plan_render.py:81-86`

- [ ] **Step 1: Verify current 3-phase palette exists**

```bash
grep -n "^PH_" "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
Expected (5 lines): PH_GOLD, PH_TERRA, PH_WATER, PH_MOON, PH_RUST.

- [ ] **Step 2: Replace the palette comment block to encode the 2-phase model**

Edit lines 81–86. Old:
```python
# Phase palette (matches the deck)
PH_GOLD = "#C8A83A"   # Reception
PH_TERRA = "#A87850"  # Phase 1 circuit pavilions
PH_WATER = "#6B8A82"  # Water elements
PH_MOON = "#CCBE9C"   # Phase 2 casitas
PH_RUST = "#8A5A38"   # Phase 3 villas/suites
```

New:
```python
# Phase palette (matches the deck — 2-phase model 2026-05-10)
# Phase 1 = full integrated wellness boutique (uses GOLD/TERRA/WATER/MOON for visual variety
#           within the same phase tag; legend collapses to single Phase 1 color).
# Phase 2 = earned expansion (premium suites + villas).
PH_GOLD  = "#C8A83A"   # Phase 1 · Reception accent
PH_TERRA = "#A87850"   # Phase 1 · circuit pavilions
PH_WATER = "#6B8A82"   # Phase 1 · water elements
PH_MOON  = "#CCBE9C"   # Phase 1 · casitas
PH_RUST  = "#8A5A38"   # Phase 2 · suites + villas
PH_PHASE1 = PH_TERRA   # canonical Phase 1 color for legend
PH_PHASE2 = PH_RUST    # canonical Phase 2 color for legend
```

- [ ] **Step 3: Verify the file still parses**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe -c "import ast; ast.parse(open('master_plan_render.py').read()); print('OK')"
```
Expected: `OK`

- [ ] **Step 4: Commit (no functional change yet)**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add master_plan_render.py && git commit -m "Add canonical PH_PHASE1/PH_PHASE2 color constants for 2-phase model"
```

---

### Task A2: Remap element phase tags from 3 phases to 2

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\master_plan_render.py` (the `elements = [...]` block, around line 301)

The existing element table tags Phase 1 wellness items as `"P1"`, Phase 2 boutique items as `"P2"`, and Phase 3 suites/villas as `"P3"`. New mapping per spec §7.3: all old `P1` and `P2` items → new `P1`; all old `P3` items → new `P2`.

- [ ] **Step 1: Verify current 3-phase tags exist in element list**

```bash
grep -nE '^\s*\("P[123]"' "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
Expected: tagged elements for P1, P2, P3.

- [ ] **Step 2: Read the elements block to confirm exact lines**

```bash
grep -n 'elements = \[' "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
Note the line number — opens at ~301. Read 30 lines from there.

- [ ] **Step 3: Edit the element block**

Replace the entire `elements = [...]` block. The new block keeps the same coordinates and footprints; only the phase tag and the comment header change. Old block has three sections (`# Phase 1 wellness pilot`, `# Phase 2 boutique`, `# Phase 3 full resort`). New block has two sections (`# Phase 1 — full integrated boutique`, `# Phase 2 — earned expansion`).

```python
    elements = [
        # phase, name, lat, lon, w, d, bearing, fill, label_style, label_dir
        # bearing 0 = long axis N–S; bearing 90 = long axis E–W (along ridge)

        # OWNER residences (private, on Vivienda strips — far west)
        ("OWN", "Owner House 1", *H1_PT, 20, 12, 90, PH_OWNER, "normal", "S"),
        ("OWN", "Owner House 2", *H2_PT, 20, 12, 90, PH_OWNER, "normal", "S"),

        # Phase 1 — full integrated boutique (open day one)
        ("P1", "Reception",       *P1_RECEPTION, 18, 14, 90, PH_GOLD,  "headline", "N"),
        ("P1", "Spa reception",   *P1_SPA,        8,  6, 90, PH_GOLD,  "normal",   "S"),
        ("P1", "HBOT",            *P1_HBOT,      12, 13, 0,  PH_TERRA, "normal",   "SW"),
        ("P1", "Sauna · cold",    *P1_SAUNA,      9,  9, 0,  PH_TERRA, "normal",   "W"),
        ("P1", "Sound & breath",  *P1_SOUND,      8,  8, 0,  PH_TERRA, "normal",   "N"),
        ("P1", "Treatment house", *P1_TREAT,     14, 10, 90, PH_TERRA, "normal",   "N"),
        ("P1", "Hydrotherapy",    *P1_HYDRO,     12, 10, 0,  PH_WATER, "normal",   "SE"),
        ("P1", "Reflection pool", *P1_REFLECT,   20, 10, 90, PH_WATER, "normal",   "N"),
        ("P1", "Casita ridge — W (ocean)",  *P2_CAS_W_RIDGE, 30, 8, 90, PH_MOON, "normal", "N"),
        ("P1", "Casita south — E (canyon)", *P2_CAS_E_SOUTH, 25, 8, 90, PH_MOON, "normal", "S"),
        ("P1", "Casita — N ridge",          *P2_CAS_NORTH,   24, 14, 90, PH_MOON, "normal", "N"),
        ("P1", "F&B pavilion",              *P2_FB,          18, 14, 0,  PH_MOON,  "normal", "W"),
        ("P1", "Pool complex",              *P2_POOL,        20, 12, 0,  PH_WATER, "normal", "S"),

        # Phase 2 — earned expansion (post-gate)
        ("P2", "Suites — E ridge",     *P3_SUITES,    30, 12, 90, PH_RUST,  "normal", "N"),
        ("P2", "Villa — W (Pacific)",  *P3_VILLA_W,   16, 14, 0,  PH_RUST,  "normal", "SW"),
    ]
```

Note: the coordinate constants (`P2_CAS_W_RIDGE`, `P3_SUITES`, etc.) keep their old prefixes — no need to rename them, only the phase tag in the tuple changes. The script's `phases_to_show=("P1", "P2", "P3")` default in `main()` should also be updated.

- [ ] **Step 4: Update the phases_to_show default in main()**

Find `def main(phases_to_show=("P1", "P2", "P3")):` (around line 197). Change to:
```python
def main(phases_to_show=("P1", "P2")):
```

- [ ] **Step 5: Verify**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && grep -cE '^\s*\("P3"' master_plan_render.py
```
Expected: `0`

```bash
grep -n 'phases_to_show=' master_plan_render.py
```
Expected: shows the updated 2-phase tuple.

- [ ] **Step 6: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add master_plan_render.py && git commit -m "Remap master plan elements to 2-phase model (P1/P2/P3 -> P1/P2)"
```

---

### Task A3: Update phase legend HTML

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\master_plan_render.py` (legend HTML block — search for `Phase 1` in the legend strings, around line 700-900)

- [ ] **Step 1: Locate the legend block**

```bash
grep -n "Phase 3" "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
Expected: lines that contain "Phase 3" in the legend HTML.

- [ ] **Step 2: Read the surrounding context (10 lines above and below each match) to understand the legend structure.**

- [ ] **Step 3: Edit the legend HTML**

In the legend HTML string, replace the three-phase legend rows with two:

Old (search for these exact lines):
```html
<div><span style="...background:{PH_TERRA}..."></span>Phase 1 · wellness</div>
<div><span style="...background:{PH_MOON}..."></span>Phase 2 · boutique</div>
<div><span style="...background:{PH_RUST}..."></span>Phase 3 · suites/villas</div>
```

(Exact text varies — match against `Phase 1`, `Phase 2`, `Phase 3` in the legend block.)

New:
```html
<div><span style="display:inline-block;width:14px;height:14px;background:{PH_PHASE1};margin-right:6px;border-radius:2px"></span>Phase 1 · full integrated boutique</div>
<div><span style="display:inline-block;width:14px;height:14px;background:{PH_PHASE2};margin-right:6px;border-radius:2px"></span>Phase 2 · earned expansion</div>
```

- [ ] **Step 4: Verify legend has exactly 2 phase entries**

```bash
grep -cE 'Phase [12]' "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
The exact count depends on how many places mention Phase 1/2 — but `Phase 3` should be `0` everywhere:

```bash
grep -c "Phase 3" "C:/Users/Jon Berg/Projects/earth-engine/master_plan_render.py"
```
Expected: `0`

- [ ] **Step 5: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add master_plan_render.py && git commit -m "Update master plan legend to 2-phase"
```

---

### Task A4: Apply the same phase remap to the SVG viewer generator

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\generate_master_plan_viewer.py`

This script imports from `master_plan_render` (per memory), so the element table is already remapped — but it has its own legend / color logic that must match.

- [ ] **Step 1: Verify the import dependency**

```bash
grep -n "import master_plan_render\|from master_plan_render" "C:/Users/Jon Berg/Projects/earth-engine/generate_master_plan_viewer.py"
```
Expected: shows the import.

- [ ] **Step 2: Search for "Phase 3" or "P3" hard-coded references**

```bash
grep -nE 'Phase 3|"P3"|PH_RUST.*Phase' "C:/Users/Jon Berg/Projects/earth-engine/generate_master_plan_viewer.py"
```

- [ ] **Step 3: For each match, replace with the 2-phase version**

For each "Phase 3" reference in the viewer SVG legend / phase-color logic, remove or remap:
- If it's a third legend row, delete the row.
- If it's a color mapping for `"P3"` tag, change to `"P2"` since the master_plan_render.py element table now uses `"P2"` for what was `"P3"`.

Show the engineer the exact diff using the Edit tool with the matched lines. Re-run the grep — should return zero results.

- [ ] **Step 4: Verify**

```bash
grep -c "Phase 3\|\"P3\"" "C:/Users/Jon Berg/Projects/earth-engine/generate_master_plan_viewer.py"
```
Expected: `0`

- [ ] **Step 5: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add generate_master_plan_viewer.py && git commit -m "Update SVG viewer generator to 2-phase legend"
```

---

### Task A5: Render the updated map and verify visually

**Files:**
- Outputs: `cucunumic_master_plan_map.html`, `cucunumic_master_plan_viewer.html`

- [ ] **Step 1: Run master_plan_render.py**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe master_plan_render.py
```
Expected console output: boundary check (all OK), distance report, setback compliance (all OK), no Python errors.

- [ ] **Step 2: Run generate_master_plan_viewer.py**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe generate_master_plan_viewer.py
```
Expected: writes `cucunumic_master_plan_viewer.html`, no errors.

- [ ] **Step 3: Open both in browser**

```bash
cmd.exe //c start "" "http://localhost:8765/cucunumic_master_plan_map.html"
cmd.exe //c start "" "http://localhost:8765/cucunumic_master_plan_viewer.html"
```

- [ ] **Step 4: Visual sanity checks**

Confirm in browser:
1. Legend shows exactly 2 phase entries: "Phase 1 · full integrated boutique" and "Phase 2 · earned expansion"
2. All wellness, casitas, F&B, pool elements show as Phase 1 color
3. Suites + Villa show as Phase 2 color (the rust/PH_RUST tone)
4. River 50m setback band still renders (red dashed)
5. Owner residences still render (purple)
6. No element appears missing
7. No element renders outside the property boundary

- [ ] **Step 5: Commit (regenerated outputs)**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add cucunumic_master_plan_map.html cucunumic_master_plan_viewer.html && git commit -m "Regenerate master plan map + SVG viewer for 2-phase model"
```

---

### Task A6: Capture fresh screenshots via Playwright

**Files:**
- Output: `C:\Users\Jon Berg\Projects\Cucunumic\Cucunumic\07. Fotos\masterplan_p1.png` (new)
- Output: `C:\Users\Jon Berg\Projects\Cucunumic\Cucunumic\07. Fotos\masterplan_p2.png` (new)
- Output: `C:\Users\Jon Berg\Projects\Cucunumic\Cucunumic\07. Fotos\masterplan_overview.png` (new)

Per memory at `reference/baja-wellness-files.md`, capture screenshots via Python Playwright (NOT MCP — conflicts with user's Chrome profile).

- [ ] **Step 1: Verify Playwright is available in the venv**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe -c "import playwright; print(playwright.__version__)"
```
If missing, install:
```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/pip install playwright && ./.venv/Scripts/playwright install chromium
```

- [ ] **Step 2: Write the screenshot script**

Create `C:\Users\Jon Berg\Projects\earth-engine\capture_masterplan_screenshots.py`:

```python
"""Capture three screenshots of the master plan map at consistent zoom/extent.
Outputs: masterplan_overview.png (both phases visible),
         masterplan_p1.png (Phase 1 only),
         masterplan_p2.png (Phase 1 + Phase 2 visible).

Saves to: C:\\Users\\Jon Berg\\Projects\\Cucunumic\\Cucunumic\\07. Fotos\\
"""
from pathlib import Path
from playwright.sync_api import sync_playwright

OUT_DIR = Path(r"C:\Users\Jon Berg\Projects\Cucunumic\Cucunumic\07. Fotos")
SVG_URL = "http://localhost:8765/cucunumic_master_plan_viewer.html"
W, H = 1920, 1200


def capture(out_path, query):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        ctx = browser.new_context(viewport={"width": W, "height": H},
                                   device_scale_factor=2)
        page = ctx.new_page()
        page.goto(f"{SVG_URL}{query}")
        page.wait_for_load_state("networkidle")
        page.screenshot(path=str(out_path), full_page=False)
        browser.close()
        print(f"Wrote: {out_path}")


if __name__ == "__main__":
    OUT_DIR.mkdir(parents=True, exist_ok=True)
    # The viewer HTML may support query parameters for filtering phases.
    # If not, all three captures are identical (full overview).
    capture(OUT_DIR / "masterplan_overview.png", "")
    capture(OUT_DIR / "masterplan_p1.png", "?phase=1")
    capture(OUT_DIR / "masterplan_p2.png", "?phase=2")
```

If `generate_master_plan_viewer.py` does not currently support `?phase=` query params, either:
- (a) Add a small filter to the viewer that responds to the query param (15 min of work), OR
- (b) Modify `capture_masterplan_screenshots.py` to call the viewer 3 times with different phase filters configured before each render.

For first pass, accept option (b) as a stub that captures the same overview thrice — the deck still embeds three images, even if they're identical, until phase-filtered rendering is added later.

- [ ] **Step 3: Run the screenshot capture**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe capture_masterplan_screenshots.py
```
Expected output: 3 lines, "Wrote: …".

- [ ] **Step 4: Verify the screenshots exist**

```bash
ls -la "C:/Users/Jon Berg/Projects/Cucunumic/Cucunumic/07. Fotos/masterplan_"*
```
Expected: 3 files, each > 50 KB.

- [ ] **Step 5: Open one screenshot to visually confirm content**

```bash
cmd.exe //c start "" "C:/Users/Jon Berg/Projects/Cucunumic/Cucunumic/07. Fotos/masterplan_overview.png"
```
Confirm: 2-phase legend visible, all elements rendered, image is clear.

- [ ] **Step 6: Commit the capture script (the screenshots commit later with deck)**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add capture_masterplan_screenshots.py && git commit -m "Add Playwright screenshot capture for master plan viewer"
```

---

### Task A7: Rewrite the master plan markdown thesis section

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\cucunumic_master_plan.md` § 1 Thesis

- [ ] **Step 1: Verify the current 4-phase thesis is in place**

```bash
grep -n "^| \*\*[01-3]\*\*" "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
```
Expected: 4 rows (phases 0–3) in the thesis table.

- [ ] **Step 2: Replace § 1 Thesis (lines 9–22 in the current file)**

Old: 4-phase ladder with wellness pilot framing.

New (replace the entire `## 1 · Thesis` section through line 22):

```markdown
## 1 · Thesis

Cucunumic is a **Pacific-coast wellness destination, opened complete** on 11 hectares of escriturado private land. A 30-key boutique with the full integrated wellness program from day one. Phase 2 (40–50 keys) unlocks when guest demand pulls us beyond 30.

| Phase | Operating identity | Capital | Stabilized FTE |
|---|---|---|---|
| **1** | 30-key wellness boutique · complete integrated program | ~$22.7M | 32.0 |
| **2** | Earned expansion · 40 (base) or 50 (upside) keys | $18–22M (separate raise) | 47.0 |

The 2-phase ladder is gate-driven. Phase 2 unlocks only when all four conditions are simultaneously sustained: trailing-12 ADR ≥ $510, occupancy ≥ 65% × 4 quarters, documented turnaway demand ≥ 15% × 2 peak seasons (Nov–Apr), modeled DSCR ≥ 1.5× on Phase 2 expansion debt. If the gate doesn't clear, the project holds at 30 keys; nothing forces an expansion the market hasn't earned.
```

- [ ] **Step 3: Verify the replacement**

```bash
grep -nE "^## 1 · Thesis|^### 1\." "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
grep -c "wellness pilot" "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
```
Expected: § 1 Thesis section header still exists; "wellness pilot" count drops (pre-edit was likely 5+).

- [ ] **Step 4: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add cucunumic_master_plan.md && git commit -m "Rewrite cucunumic_master_plan.md thesis to 2-phase model"
```

---

### Task A8: Reflow remaining master plan sections to 2-phase

**Files:**
- Modify: `C:\Users\Jon Berg\Projects\earth-engine\cucunumic_master_plan.md` §§ 3.2, 4, 5

This is a sweep — the markdown has 3-phase structure throughout that needs collapsing. Per spec §7.2.

- [ ] **Step 1: Find every 3-phase reference**

```bash
grep -nE "Phase 3|Phase 0|wellness pilot|\| Phase \|" "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
```

- [ ] **Step 2: Update § 3.2 element table — collapse Phase column**

In the element table at § 3.2, change the "Phase" column values:
- Any `0` or `1` (wellness pilot stage) → `1`
- Any `2` (boutique stage) → `1`
- Any `3` (full resort stage) → `2`

Show the engineer the exact lines (use `grep -n "Phase 1\|Phase 2\|Phase 3"` on § 3.2 to find them) and apply Edit tool replacements.

- [ ] **Step 3: Update § 4 Engineering implications table — collapse from 3 columns to 2**

The current table has columns Phase 1 · Phase 2 · Phase 3. Replace with Phase 1 · Phase 2 columns. Numbers per spec §7.2 — typical:

```markdown
| System | Phase 1 (30 keys + wellness) | Phase 2 (40–50 keys stabilized) |
|---|---|---|
| Water demand | ~6,000 L/day | ~10,000 L/day |
| Header tank | 80 kL | 100 kL |
| Hotel cistern | 60 kL | 100 kL |
| PV | 80 kW | 150 kW (+ batteries 500 kWh) |
| Backup gen | 1 × 60 kW | 2 × 100 kW |
| Wastewater | 8,000 GPD MBR | 15,000 GPD |
| Greywater | constructed wetland | full hotel constructed wetland |
```

- [ ] **Step 4: Rewrite § 5 build sequence Gantt to 2-phase**

Replace the 4-phase ASCII Gantt with the 2-phase version:

```
2026 H2   ▓▓░░░░░░░░  Pre-development · permits · design basis
2027 H1   ░▓▓▓▓░░░░░  Phase 1 long-lead procurement · build mobilization
2027 H2   ░░░▓▓▓░░░░  Phase 1 build (30 keys + full wellness)
2028 H1   ░░░░▓▓▒░░░  Phase 1 opens · ramp begins
2028-Y3   ░░░░░░▓▓░░  Phase 1 stabilization · gate measurement
2031 Q2   ░░░░░░░▒░░  Gate clears (turnaway × 2 peak seasons binding)
2031 H2   ░░░░░░░▓▓░  Phase 2 raise · close · mobilize
2032 H1+  ░░░░░░░░▓▓  Phase 2 build (18–24 months)
2033 H1+  ░░░░░░░░░▓  Phase 2 opens · ramp begins
2036      ░░░░░░░░░░  Phase 1+2 stabilized (Y10 from Phase 1 raise)
```

- [ ] **Step 5: Update § 1.5 Spatial Narrative — drop "wellness pilot" references**

```bash
grep -n "wellness pilot" "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
```
For each match, replace "wellness pilot" with "wellness sanctuary" or remove the phrase contextually (use Edit tool per match).

- [ ] **Step 6: Verify**

```bash
grep -c "Phase 3\|Phase 0\|wellness pilot" "C:/Users/Jon Berg/Projects/earth-engine/cucunumic_master_plan.md"
```
Expected: `0`

- [ ] **Step 7: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git add cucunumic_master_plan.md && git commit -m "Collapse master plan markdown sections 3.2/4/5/1.5 to 2-phase model"
```

---

### Task A9: Phase A complete — final verification

- [ ] **Step 1: Re-run renders to ensure they still work after markdown edits**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe master_plan_render.py && ./.venv/Scripts/python.exe generate_master_plan_viewer.py
```
Expected: both succeed, no Python errors.

- [ ] **Step 2: Re-capture screenshots**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && ./.venv/Scripts/python.exe capture_masterplan_screenshots.py
```

- [ ] **Step 3: Verify all earth-engine commits**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git log --oneline -10
```
Expected: 6+ recent commits referencing the 2-phase work.

---

## Phase B — Investor Deck Rewrite

**Working file (Desktop master):** `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html`

Edit the Desktop master, then sync to `C:\Users\Jon Berg\Projects\Cucunumic\baja_wellness_deck.html` after each major edit-cluster (or once at the end of Phase B). The repo's `index.html` is touched only in Phase C.

### Task B1: Land framing — replace 38.88 acres / 15.7 ha with 11 hectares

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (multiple lines)

- [ ] **Step 1: Find every land-area reference**

```bash
grep -nE "38\.88|15\.7|15,71|15\.71" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Note all matches — there will be 5–10.

- [ ] **Step 2: For each match, replace contextually**

Common patterns to replace:
| Old | New |
|---|---|
| `38.88 acres` | `11 hectares` |
| `38 acres` | `11 hectares` |
| `15.7 ha` | `11 ha` |
| `15.71 hectáreas` | `11 hectáreas` |
| `15,71 hectáreas` | `11 hectáreas` |

Use Edit tool with `replace_all: true` for the unambiguous strings (`38.88 acres`, `15.7 ha`). Use targeted Edits for the Spanish-formatted ones if any exist.

- [ ] **Step 3: Verify**

```bash
grep -cE "38\.88|15\.7|15,71|15\.71" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

```bash
grep -c "11 hectares\|11 ha\|11 hectáreas" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: positive count.

- [ ] **Step 4: Smoke test in browser**

```bash
cmd.exe //c start "" "http://localhost:8766/baja_wellness_deck.html"
```
(Note: Pages copy will lag — view Desktop master via a separate server or sync first.)

Click through slides 1, 2, 7 (where land area appears most prominently). Confirm "11 hectares" / "11 ha" reads correctly.

---

### Task B2: Rewrite s-origin (Thesis slide) — Wellness Destination framing

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (around lines 426–450, the `s-origin` section)

The current thesis slide uses old wellness-pilot language. Replace with the new thesis from spec §1.1.

- [ ] **Step 1: Read the current s-origin section**

```bash
sed -n '426,455p' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Replace the thesis copy**

Use the Edit tool to replace the visible thesis text. Specifically:
- Replace the headline / hook with: *"A wellness destination, opened complete."*
- Replace the body paragraph with: *"30 keys on 11 hectares of Pacific-coast Baja. The full integrated wellness program from day one. Phase 2 unlocks when guest demand pulls us beyond 30."*
- Keep the slide layout, eyebrow, and styling unchanged.

The exact diff depends on the current copy — read the section first, identify the prose blocks, then Edit each one.

- [ ] **Step 3: Verify**

```bash
grep -c "wellness destination, opened complete" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `1`

- [ ] **Step 4: Sync to repo working copy**

```bash
cp "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html"
```

- [ ] **Step 5: Visual check**

Reload `http://localhost:8766/baja_wellness_deck.html`, scroll to slide 2 (s-origin). Confirm thesis copy displays correctly.

---

### Task B3: Rewrite s8-phase (Phasing) — 2-phase ladder + gate visualization

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s8-phase section, ~lines 705–757)

The current phasing slide uses a 4-phase ladder. Replace with 2-phase ladder + gate visualization per spec §2.

- [ ] **Step 1: Read the current s8-phase section**

```bash
sed -n '705,757p' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Replace the ladder content**

Replace the 4 ladder rows with 2:

```html
<div class="ladder rv d4">
  <div class="ladder-row head">
    <div>Phase</div>
    <div>Operating identity</div>
    <div>Capital</div>
    <div>Stabilized FTE</div>
    <div>Gate to advance</div>
  </div>
  <div class="ladder-row">
    <div><span class="lr-tag">P1 · 2028</span></div>
    <div>
      <div class="lr-name">30-key wellness boutique</div>
      <div class="lr-detail">Open complete · 30 keys + integrated wellness program from day one</div>
    </div>
    <div><span class="lr-num">~$22.7M</span><div class="lr-num-sub">All-in</div></div>
    <div><span class="lr-num">32.0</span><div class="lr-num-sub">FTE</div></div>
    <div><span class="lr-num" style="font-size:0.95rem">Gate (4 conditions)</span><div class="lr-num-sub">ADR · OCC · turnaway · DSCR</div></div>
  </div>
  <div class="ladder-row">
    <div><span class="lr-tag">P2 · 2033+</span></div>
    <div>
      <div class="lr-name">Earned expansion</div>
      <div class="lr-detail">+10 (40 keys base) or +20 (50 keys upside) · suites + villas</div>
    </div>
    <div><span class="lr-num">$18–22M</span><div class="lr-num-sub">Separate raise</div></div>
    <div><span class="lr-num">47.0</span><div class="lr-num-sub">FTE stabilized</div></div>
    <div><span class="lr-num" style="font-size:0.95rem">Phase 1+2 stabilization</span><div class="lr-num-sub">Y9–Y10</div></div>
  </div>
</div>
<div class="ladder-foot rv d6">Two phases. One gate. Each phase a complete operating business; Phase 2 only happens when guests demand it.</div>
```

- [ ] **Step 3: Add the gate detail callout below the ladder**

Below the ladder, add a small callout box:

```html
<div class="rv d7" style="margin-top:2.4rem;display:grid;grid-template-columns:1fr 1fr 1fr 1fr;gap:1.4rem;border-top:1px solid rgba(240,235,224,0.08);padding-top:1.6rem">
  <div><div style="font-family:var(--sans);font-size:0.55rem;letter-spacing:0.4em;text-transform:uppercase;color:var(--mist);margin-bottom:0.4rem">ADR</div><div style="font-family:var(--serif);font-size:1.2rem;color:var(--moon)">≥ $510</div><div style="font-family:var(--body);font-size:0.78rem;color:var(--sand);margin-top:0.3rem">Trailing-12</div></div>
  <div><div style="font-family:var(--sans);font-size:0.55rem;letter-spacing:0.4em;text-transform:uppercase;color:var(--mist);margin-bottom:0.4rem">Occupancy</div><div style="font-family:var(--serif);font-size:1.2rem;color:var(--moon)">≥ 65%</div><div style="font-family:var(--body);font-size:0.78rem;color:var(--sand);margin-top:0.3rem">4 consecutive Q</div></div>
  <div><div style="font-family:var(--sans);font-size:0.55rem;letter-spacing:0.4em;text-transform:uppercase;color:var(--mist);margin-bottom:0.4rem">Turnaway</div><div style="font-family:var(--serif);font-size:1.2rem;color:var(--moon)">≥ 15%</div><div style="font-family:var(--body);font-size:0.78rem;color:var(--sand);margin-top:0.3rem">2 peak seasons</div></div>
  <div><div style="font-family:var(--sans);font-size:0.55rem;letter-spacing:0.4em;text-transform:uppercase;color:var(--mist);margin-bottom:0.4rem">DSCR</div><div style="font-family:var(--serif);font-size:1.2rem;color:var(--moon)">≥ 1.5×</div><div style="font-family:var(--body);font-size:0.78rem;color:var(--sand);margin-top:0.3rem">Modeled at trigger</div></div>
</div>
```

- [ ] **Step 4: Verify**

```bash
grep -c "30-key wellness boutique" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `1`

```bash
grep -c "Earned expansion" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `1`

- [ ] **Step 5: Visual check**

Sync to repo, reload, scroll to slide 8 (s8-phase). Confirm the 2-phase ladder + gate detail callout render correctly.

```bash
cp "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html"
cmd.exe //c start "" "http://localhost:8766/baja_wellness_deck.html#s8-phase"
```

---

### Task B4: Update s9-prog (Phase Progression) — masterplan_p1/p2 only

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s9-prog section, ~lines 761–790)

The current slide embeds masterplan_p1.png, masterplan_p2.png, masterplan_p3.png. Replace with masterplan_p1.png and masterplan_p2.png — drop the third.

- [ ] **Step 1: Read the current s9-prog section**

```bash
sed -n '761,791p' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Find and remove the masterplan_p3 image reference**

```bash
grep -n "masterplan_p3" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

For each match, identify the wrapping `<img>` or container `<div>`. Use Edit tool to remove the entire image tile (typically a `<div class="phase-tile">` or similar). Keep masterplan_p1 and masterplan_p2 tiles.

- [ ] **Step 3: Adjust the grid layout from 3 columns to 2**

If the grid is `grid-template-columns:repeat(3,1fr)`, change to `repeat(2,1fr)`. Find the inline style or class definition.

- [ ] **Step 4: Update the slide headline copy**

The current "09 — Phase Progression" eyebrow is correct. Replace any subhead that says "Built one phase at a time" with: *"Built complete in Phase 1. Expanded only when earned."*

- [ ] **Step 5: Verify**

```bash
grep -c "masterplan_p3" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

```bash
grep -c "masterplan_p1\|masterplan_p2" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `2`

- [ ] **Step 6: Visual check**

Reload, scroll to slide 9. Confirm only 2 phase images render side-by-side, with no third broken-image placeholder.

---

### Task B5: Rewrite s10-p1 (Phase 1 detail)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s10-p1 section, lines 794–829)

Per spec §7.1, rewrite this slide as Phase 1 · Wellness Boutique with the new numbers.

- [ ] **Step 1: Replace the Phase 1 detail content**

Find the s10-p1 section. Replace key text:

| Element | Old | New |
|---|---|---|
| Eyebrow | "10 — Phase 1 · Wellness Pilot" | "10 — Phase 1 · Wellness Boutique" |
| Phase number display | "01 / pilot" | "01 / boutique" |
| Headline | "Prove the science. Prove the price. Then scale." | "Open complete. Earn the brand from day one." |
| Lede paragraph | (rewrite to describe 30 keys + full wellness, 18-month build, $22.7M) | See below |
| Capital KPI | "$1.15M" | "$22.7M" |
| Break-even KPI | "81 / mo" | "65% / occ" (or remove and add new KPI) |
| IRR · base | "28–32%" | "18–22%" |
| Payback | "2.5 YR" | "Y6–Y7" |
| Y3 OpEx | "$552K" | "$5.25M" |
| Y3 EBITDA / client | "$235" | "$1.12M Y3 EBITDA" |
| Gate to advance | (current text about clients/mo @ $450) | Updated 4-criterion gate |

New lede paragraph:
> "A 30-key boutique with the full integrated wellness program — Reception, six circuit cells (HBOT, sauna+cold, hydrotherapy, sound+breath, recovery, reflection pool), Spa Reception, Treatment House, Recovery Garden, F&B pavilion, pool. 18-month build. Open H1 2028. Stabilization Y3–Y5. The boutique is itself the validation."

New "Gate to Phase 2" body:
> "All four conditions sustained: trailing-12 ADR ≥ $510, occupancy ≥ 65% × 4 consecutive quarters, documented turnaway demand ≥ 15% of inquiries × 2 consecutive peak seasons, modeled DSCR ≥ 1.5× on Phase 2 expansion debt."

Use Edit tool with the exact old text → new text for each replacement.

- [ ] **Step 2: Verify**

```bash
grep -c "Wellness Boutique\|30-key\|22\.7M" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: positive count.

```bash
grep -c "Wellness Pilot\|\$1\.15M\|81 clients" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

- [ ] **Step 3: Visual check**

Sync, reload, scroll to slide 10. Verify all KPIs and copy show the new numbers.

---

### Task B6: Rewrite s10-p2 (Phase 2 detail)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s10-p2 section, lines 835–870)

Replace with Phase 2 · Earned Expansion content.

- [ ] **Step 1: Replace the Phase 2 detail content**

| Element | Old | New |
|---|---|---|
| Eyebrow | "11 — Phase 2 · Integrated Boutique" | "11 — Phase 2 · Earned Expansion" |
| Phase number display | "02 / boutique" | "02 / expansion" |
| Headline | "Casitas around a courtyard. Wellness as the operating spine." | "Phase 2 happens when guests demand it — not when we decide to build it." |
| Lede | (rewrite for expansion, 40 base / 50 upside) | See below |
| Capital · cumulative | "$6.75–7.5M" | "$18–22M (separate raise)" |
| Keys | "30" | "40 base / 50 upside" |
| Stabilized FTE | "26.5" | "47.0" |
| Y2 Revenue | "$2.33M" | (remove — Y2 of Phase 2 is build year) |
| Y3 EBITDA | "$695K" | "$3.5–4.0M Phase 2 incremental EBITDA" |
| RevPOR | "$407" | "$610 blended (P1+P2)" |
| Gate to advance (now "Gate to expansion") | (current Phase 2 → 3 text) | Updated to reflect Y9–Y10 stabilization |

New lede:
> "Built only after Phase 1 has proved itself. +10 suites (E ridge canyon view) for the 40-key base case, or +10 suites + 10 villas (W ridge Pacific view) for the 50-key upside. ~$18–22M raised at uplifted valuation reflecting Phase 1 stabilized cash flow. Build 18–24 months. Open ~2033. Combined Phase 1+2 stabilized at Y9–Y10."

- [ ] **Step 2: Verify**

```bash
grep -c "Earned Expansion\|40 base / 50 upside\|18–22M" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: positive count.

- [ ] **Step 3: Visual check**

Sync, reload slide 11.

---

### Task B7: Delete s11-p3 (former Phase 3 detail)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s11-p3 section, lines 873–913)

The Phase 3 slide is no longer needed.

- [ ] **Step 1: Identify the s11-p3 boundaries**

```bash
grep -n 'id="s11-p3"\|<!-- ════' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html" | head -30
```
Find the opening `<section class="slide" id="s11-p3">` and the matching `</section>`. Include the preceding comment block.

- [ ] **Step 2: Delete the section + its preceding comment block**

Use Edit tool. Old (everything from the comment header down through `</section>` for s11-p3):
```html
<!-- ════════════════════════════════════════════════════════════
     11 · PHASE 3 — Full Resort
════════════════════════════════════════════════════════════════ -->
<section class="slide" id="s11-p3">
  ...
</section>
```

New: empty string (delete entirely).

- [ ] **Step 3: Verify**

```bash
grep -c 'id="s11-p3"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

```bash
grep -c '<section class="slide"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `16` (was 17).

- [ ] **Step 4: Visual check**

Sync, reload. Slide count at top of page should auto-update to "16" (from 17). Scroll-snap should advance from slide 11 (Phase 2 detail) directly to the Operating Model slide.

---

### Task B8: Update s12-op (Operating Model & Staffing)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s12-op section)

- [ ] **Step 1: Read the current section**

```bash
grep -n 'id="s12-op"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
sed -n '$START,+80p' "..."   # use the line returned above
```

- [ ] **Step 2: Update FTE numbers**

Replace any old FTE numbers (typically Phase 1 = 7, Phase 2 = 26.5, Phase 3 = 63) with the new model:
- Phase 1 stabilized: **32.0** FTE
- Phase 2 stabilized: **47.0** FTE

The slide likely has a staff-bars visualization with three rows. Reduce to two rows:

```html
<div class="staff-row">
  <div class="sr-label">Phase 1</div>
  <div class="sr-bar-track"><div class="sr-bar-fill" style="width:68%"></div></div>
  <div class="sr-num">32.0<small>FTE</small></div>
</div>
<div class="staff-row">
  <div class="sr-label">Phase 2</div>
  <div class="sr-bar-track"><div class="sr-bar-fill" style="width:100%"></div></div>
  <div class="sr-num">47.0<small>FTE</small></div>
</div>
```

(Width 68% for Phase 1 = 32/47 of the bar track; 100% for Phase 2.)

- [ ] **Step 3: Update role descriptions if any reference old phase ladder**

Search for "Phase 0", "Phase 3" within s12-op and revise.

- [ ] **Step 4: Verify**

```bash
grep -c "32\.0\|47\.0" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: ≥ 2

---

### Task B9: Update s13-cap (Capital Ladder & Funding Stack)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s13-cap section)

- [ ] **Step 1: Read the current section**

```bash
grep -n 'id="s13-cap"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Replace the capital stack visualization**

Current likely shows 4-phase progressive capital ladder. Replace with two-raise structure:

- **Raise 1 — Phase 1: $22.7M** (stack: equity + land contribution + optional construction debt)
- **Raise 2 — Phase 2: $18–22M** (stack: equity at uplifted valuation + optional construction debt at ≤65% LTC)

Use the existing `cap-stages` grid pattern, two rows.

```html
<div class="cap-stack rv d4">
  <div class="cap-row">
    <div class="cap-label">Raise 1 · 2027</div>
    <div class="cap-stages">
      <div class="cap-seg equity"></div><div class="cap-seg equity"></div><div class="cap-seg equity"></div>
      <div class="cap-seg land"></div><div class="cap-seg land"></div>
      <div class="cap-seg lp"></div><div class="cap-seg lp"></div>
      <div class="cap-seg reserve"></div>
    </div>
    <div class="cap-total">$22.7M<small>Phase 1 all-in</small></div>
  </div>
  <div class="cap-row">
    <div class="cap-label">Raise 2 · 2031+</div>
    <div class="cap-stages">
      <div class="cap-seg equity"></div><div class="cap-seg equity"></div>
      <div class="cap-seg lp"></div><div class="cap-seg lp"></div><div class="cap-seg lp"></div>
      <div class="cap-seg debt"></div><div class="cap-seg debt"></div>
      <div class="cap-seg reserve"></div>
    </div>
    <div class="cap-total">$18–22M<small>Phase 2 separate raise</small></div>
  </div>
</div>
```

- [ ] **Step 3: Update any prose around the cap stack**

Replace mentions of "Phase 0", "Phase 1 wellness pilot", or 4-phase financing language with the new narrative: equity-led Phase 1 with optional construction debt; Phase 2 raised at uplifted valuation post-gate.

- [ ] **Step 4: Verify**

```bash
grep -c "Raise 1 · 2027\|Raise 2 · 2031" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: ≥ 2

---

### Task B10: Update s14-ret (Returns Matrix)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s14-ret section)

- [ ] **Step 1: Read the current returns table**

```bash
grep -n 'id="s14-ret"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Replace the returns table**

Old: 4-phase returns matrix with old IRRs.

New table per spec §4.1 and §4.4:

```html
<table class="returns-table rv d4">
  <thead>
    <tr><th>Scenario</th><th>P1 Y5 IRR</th><th>P1 Y5 EBITDA</th><th>P1 Y5 Valuation</th><th>P1+P2 Y10 IRR</th><th>P1+P2 Y10 Multiple</th></tr>
  </thead>
  <tbody>
    <tr><td class="scen">Conservative</td><td class="metric">12–15%</td><td>$1.6M</td><td>$12.8M</td><td>14–17%</td><td>2.7×</td></tr>
    <tr><td class="scen">Base</td><td class="metric">18–22%</td><td>$3.05M</td><td>$33.6M</td><td>17–22%</td><td>3.5–4.5×</td></tr>
    <tr class="upside"><td class="scen">Upside</td><td class="metric">24–28%</td><td>$4.0M</td><td>$52M</td><td>22–26%</td><td>4.5–5.5×</td></tr>
  </tbody>
</table>
```

- [ ] **Step 3: Update headline / subhead copy**

Replace any "Phase-by-phase, the case is different" subhead with: *"Phase 1 stands alone. Phase 2 is upside on top."*

- [ ] **Step 4: Verify**

```bash
grep -c "12.8M\|33.6M\|52M" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: ≥ 3

---

### Task B11: Update s15-rm (Roadmap & Decision Gates)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s15-rm section)

- [ ] **Step 1: Read the current Gantt**

```bash
grep -n 'id="s15-rm"' "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Update the Gantt grid to 2-phase timeline**

The existing Gantt has 4 phases × 10 columns (years). Reduce to 2 row clusters spanning 2026–2036:
- **Pre-development (2026 H2 – 2027 H1):** 1.5 cells filled
- **Phase 1 build (2027 H1 – 2028 H1):** ~2 cells
- **Phase 1 ramp / gate measurement (2028 – 2031):** ~3 cells (with gate marker at 2031)
- **Phase 2 close + build (2031 – 2033):** ~2 cells
- **Phase 2 ramp + combined stabilization (2033 – 2036):** ~3 cells

The exact `<div class="gantt-cell fill">` HTML pattern depends on how many year columns are in use. Read the current structure and adjust.

- [ ] **Step 3: Add a clear "Gate" marker at 2031 Q2**

```html
<div class="gantt-cell fill gate" title="Gate clears Apr 2031"></div>
```
The `gate` class already styles this gold (per CSS at line 342).

- [ ] **Step 4: Update the Gantt foot caption**

Replace any "rolling gate program" language with: *"One gate. Phase 2 happens only when the four conditions sustain. If they don't, Phase 1 holds at 30 keys indefinitely — and stays a complete operating business."*

---

### Task B12: Update s16 (The Ask + Close)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html` (s16 section, ~lines 1238–1278)

The current Ask references "Phase 0 + 1 Capital · $1.65M" — entirely outdated.

- [ ] **Step 1: Replace the Ask content**

Old:
```html
<p class="h3 rv d4 ph-text" ...>Phase 0 + 1 Capital</p>
<ul class="ask-list rv d5">
  <li>...$1.65M sponsor + LP equity...</li>
  <li>...wellness pilot all-in...</li>
  <li>...convertible note...</li>
  <li>...38 acres · escriturado...</li>
  <li>...Phase 1 soft-open Q3 2027 · Phase 2 close 2027 H2...</li>
</ul>
```

New:
```html
<p class="h3 rv d4 ph-text" style="font-size:clamp(1.3rem,1.8vw,1.65rem);margin-bottom:1.2rem">Phase 1 Capital</p>
<ul class="ask-list rv d5">
  <li><span class="nm">Round size</span><span class="vl">$22.7M sponsor + LP equity (range $20–25M)</span></li>
  <li><span class="nm">Use of proceeds</span><span class="vl">30-key boutique + integrated wellness program · all-in build</span></li>
  <li><span class="nm">Structure</span><span class="vl">Equity-led with optional construction debt · negotiated land contribution</span></li>
  <li><span class="nm">Land</span><span class="vl">11 hectares · escriturado · sponsor side</span></li>
  <li><span class="nm">Target dates</span><span class="vl">Build start 2027 · Open 2028 H1 · Gate measurement 2030–2031</span></li>
</ul>
```

And the right-side "What investors get" panel:
```html
<p class="h3 rv d4 ph-text" style="font-size:clamp(1.3rem,1.8vw,1.65rem);margin-bottom:1.2rem">What investors get</p>
<ul class="ask-list rv d6">
  <li><span class="nm">Phase 1 IRR</span><span class="vl">18–22% modeled base case · 24–28% upside</span></li>
  <li><span class="nm">Equity multiple</span><span class="vl">3.0–3.5× base · 3.8–4.5× upside (5-year)</span></li>
  <li><span class="nm">Phase 2 right of first offer</span><span class="vl">Pro-rata participation at uplifted valuation</span></li>
  <li><span class="nm">Phase 2 optionality</span><span class="vl">Phase 1 LPs can pass without losing Phase 1 position</span></li>
  <li><span class="nm">Reporting</span><span class="vl">Quarterly KPIs against the gate dashboard</span></li>
</ul>
```

- [ ] **Step 2: Update the close tagline**

Old: *"A place where wellness is the operating purpose, and where every phase is a complete business — not a bridge to one."*

New: *"A wellness destination, opened complete. Phase 2 happens when guests demand it."*

- [ ] **Step 3: Verify**

```bash
grep -c "Phase 0 + 1\|\$1\.65M\|wellness pilot" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

---

### Task B13: Update slide counters (17 → 16)

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html`

- [ ] **Step 1: Find every `/ 17` reference**

```bash
grep -n "/ 17\|of 17\|01 / 17" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```

- [ ] **Step 2: Replace `/ 17` with `/ 16` everywhere**

Use Edit with `replace_all: true`:

Old: `/ 17`
New: `/ 16`

This affects the initial counter (`<div id="counter">01 / 17</div>`) and all per-slide `<div class="slide-num">N / 17</div>` divs.

- [ ] **Step 3: Renumber the per-slide `slide-num` divs after the deletion**

After deleting s11-p3, the slide-num divs that came after it (s12-op was "13 / 17", etc.) are now off by one. They need to renumber:

| Slide | Old slide-num | New slide-num |
|---|---|---|
| s10-p2 | 11 / 17 | 11 / 16 |
| ~~s11-p3~~ | ~~12 / 17~~ | (deleted) |
| s12-op | 13 / 17 | 12 / 16 |
| s13-cap | 14 / 17 | 13 / 16 |
| s14-ret | 15 / 17 | 14 / 16 |
| s15-rm | 16 / 17 | 15 / 16 |
| s16 | 17 / 17 | 16 / 16 |

Use Edit tool with the exact old/new pairs.

- [ ] **Step 4: Verify**

```bash
grep -c "/ 17" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: `0`

```bash
grep -nE "slide-num\">[0-9]+ / 16" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: 16 lines, slides numbered 01–16 sequentially (with no gaps).

---

### Task B14: Update phase-tag eyebrows on s10-p1 and s10-p2

**Files:**
- Modify: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html`

After deleting s11-p3, the eyebrow numbers on the remaining slides should also renumber if they currently say "10/11/12 — Phase 1/2/3". Since we already updated s10-p1 ("10 — Phase 1 · Wellness Boutique") and s10-p2 ("11 — Phase 2 · Earned Expansion") in Tasks B5 and B6, these are correct — they remain at 10 and 11.

For s12-op through s16, the phase-tag eyebrows currently say:
- s12-op: "13 — Operating Model & Staffing" (was correct under 17-slide; should be **12** under 16-slide)
- s13-cap: "14 — Capital Ladder" → **13**
- s14-ret: "15 — Returns Matrix" → **14**
- s15-rm: "16 — Sequence & Decision Gates" → **15**
- s16: "17 — The Ask" → **16**

Wait — re-read: s10-p1 was "10 — Phase 1 · Wellness Pilot" (already at 10). s10-p2 was "11 — Phase 2 · Integrated Boutique" (was at 11). s11-p3 was "12 — Phase 3 · Full Resort". After deletion of s11-p3, the eyebrow numbering jumps from 11 to 13 — which is what we already had.

So the renumbering needed is:
- s12-op: 13 → **12**
- s13-cap: 14 → **13**
- s14-ret: 15 → **14**
- s15-rm: 16 → **15**
- s16: 17 → **16**

- [ ] **Step 1: Apply each renumber**

Use Edit tool. For each pair below, find the exact eyebrow line and replace just the leading number:

| Old | New |
|---|---|
| `>13 — Operating Model` | `>12 — Operating Model` |
| `>14 — Capital Ladder` | `>13 — Capital Ladder` |
| `>15 — Returns Matrix` | `>14 — Returns Matrix` |
| `>16 — Sequence` | `>15 — Sequence` |
| `>17 — The Ask` | `>16 — The Ask` |

- [ ] **Step 2: Verify**

```bash
grep -nE "[0-9]+ — (Operating|Capital|Returns|Sequence|The Ask)" "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html"
```
Expected: 12 / 13 / 14 / 15 / 16 in order.

---

### Task B15: Sync Desktop deck to repo and smoke-test all 16 slides

**Files:**
- Source: `C:\Users\Jon Berg\OneDrive\Desktop\baja_wellness_investor_deck.html`
- Destination: `C:\Users\Jon Berg\Projects\Cucunumic\baja_wellness_deck.html`

- [ ] **Step 1: Sync**

```bash
cp "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html"
```

- [ ] **Step 2: Verify file sizes match**

```bash
ls -la "C:/Users/Jon Berg/OneDrive/Desktop/baja_wellness_investor_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html"
```
Sizes should be identical.

- [ ] **Step 3: Open in browser**

```bash
cmd.exe //c start "" "http://localhost:8766/baja_wellness_deck.html"
```

- [ ] **Step 4: Walk through all 16 slides**

Manually scroll/arrow through every slide. Confirm:
1. Cover (s1) renders
2. Origen (s-origin) thesis copy is updated
3. Property (s2) shows 11 hectares
4. Architecture (s4-arch) renders (no changes expected)
5. Reception (s5-rec) renders
6. Circuits (s6-circ) renders (Spa Reception, Treatment House, Recovery Garden integral)
7. Market (s7-mkt) renders
8. Phasing (s8-phase) shows 2-phase ladder + gate detail
9. Phase progression (s9-prog) shows masterplan_p1 + masterplan_p2 only
10. Phase 1 detail (s10-p1) shows new $22.7M / Wellness Boutique copy
11. Phase 2 detail (s10-p2) shows Earned Expansion / 40 base / 50 upside
12. Operating model (s12-op) shows 32 / 47 FTE
13. Capital ladder (s13-cap) shows two-raise structure
14. Returns matrix (s14-ret) shows new IRR ranges
15. Roadmap (s15-rm) shows 2-phase Gantt with gate marker
16. Ask (s16) shows new $22.7M ask copy

Counter at top reads "01 / 16" through "16 / 16". Dot navigation has 16 dots. No broken images. No old "wellness pilot" or "$1.65M" text visible.

- [ ] **Step 5: Verify in source — no stale references**

```bash
cd "C:/Users/Jon Berg/Projects/Cucunumic" && grep -cE "Phase 0|Phase 3|wellness pilot|\\\$1\\.65M|38\\.88|15\\.7 ha|masterplan_p3" baja_wellness_deck.html
```
Expected: `0`

- [ ] **Step 6: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/Cucunumic" && git add baja_wellness_deck.html "Cucunumic/07. Fotos/masterplan_p1.png" "Cucunumic/07. Fotos/masterplan_p2.png" "Cucunumic/07. Fotos/masterplan_overview.png" && git commit -m "Rewrite investor deck for 2-phase model · 17 slides → 16

- Thesis: wellness destination, opened complete
- Property: 11 hectares (replacing 38.88 acres / 15.7 ha)
- Phasing: 2-phase ladder with 4-condition gate visualization
- Phase 1: 30-key wellness boutique · \$22.7M
- Phase 2: earned expansion · 40 base / 50 upside · \$18-22M separate raise
- Operating: 32 FTE Phase 1 / 47 FTE Phase 2
- Capital ladder: two-raise structure
- Returns: 18-22% Phase 1 IRR base · 17-22% combined Y10 IRR
- Removed Phase 3 detail slide (s11-p3)
- Renumbered slide counters and phase-tag eyebrows
- Fresh masterplan_p1.png · masterplan_p2.png · masterplan_overview.png from regenerated 2-phase render"
```

---

## Phase C — Pages Publish

### Task C1: Sync repo working copy to index.html

**Files:**
- Source: `C:\Users\Jon Berg\Projects\Cucunumic\baja_wellness_deck.html`
- Destination: `C:\Users\Jon Berg\Projects\Cucunumic\index.html`

- [ ] **Step 1: Verify what's currently in index.html**

```bash
grep -c '<section class="slide"' "C:/Users/Jon Berg/Projects/Cucunumic/index.html"
```
Expected: 12 (the live Pages copy is currently 5 slides behind).

- [ ] **Step 2: Sync**

```bash
cp "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/index.html"
```

- [ ] **Step 3: Verify**

```bash
grep -c '<section class="slide"' "C:/Users/Jon Berg/Projects/Cucunumic/index.html"
```
Expected: `16`

```bash
diff "C:/Users/Jon Berg/Projects/Cucunumic/baja_wellness_deck.html" "C:/Users/Jon Berg/Projects/Cucunumic/index.html"
```
Expected: no diff output (identical).

- [ ] **Step 4: Local smoke test**

```bash
cmd.exe //c start "" "http://localhost:8766/index.html"
```
Walk first/middle/last slide quickly. Confirm parity with baja_wellness_deck.html.

---

### Task C2: Commit Pages publish

- [ ] **Step 1: Commit**

```bash
cd "C:/Users/Jon Berg/Projects/Cucunumic" && git add index.html && git commit -m "Publish 2-phase deck to GitHub Pages (index.html)"
```

- [ ] **Step 2: Verify gh authentication**

```bash
gh auth status
```
Expected: logged in as Jonwberg with valid token.

- [ ] **Step 3: Push to remote**

```bash
cd "C:/Users/Jon Berg/Projects/Cucunumic" && git -c credential.helper='!gh auth git-credential' push origin master
```
Expected: push succeeds, no auth prompt.

- [ ] **Step 4: Wait ~30 seconds, then verify Pages URL**

```bash
sleep 30 && curl -s -o /dev/null -w "%{http_code}\n" https://jonwberg.github.io/cucunumic/
```
Expected: `200`

```bash
curl -s https://jonwberg.github.io/cucunumic/ | grep -c '<section class="slide"'
```
Expected: `16`

---

### Task C3: Final verification

- [ ] **Step 1: View live deck**

```bash
cmd.exe //c start "" "https://jonwberg.github.io/cucunumic/"
```
Walk all 16 slides on the live site.

- [ ] **Step 2: Confirm clean git state**

```bash
cd "C:/Users/Jon Berg/Projects/Cucunumic" && git log --oneline -10 && echo "---" && git status
```
Expected: 3 new commits (spec, deck rewrite, Pages publish). Status clean except pre-existing untracked files (baja_wellness_deck.html, analysis.html, analyze.py, drone mp4 modifications) which are out of scope for this plan.

- [ ] **Step 3: Confirm earth-engine commits intact**

```bash
cd "C:/Users/Jon Berg/Projects/earth-engine" && git log --oneline -10
```
Expected: 6+ commits referencing the 2-phase work.

---

## Plan Self-Review

Run after writing the plan:

**1. Spec coverage:** Every section of the spec maps to a task:
- Spec §1 Thesis → Task B2 (s-origin rewrite)
- Spec §2 Phase Structure → Tasks B3 (s8-phase), B12 (s16 ask)
- Spec §3 Revenue Model → Tasks B5 (s10-p1), B10 (s14-ret)
- Spec §4 Returns & Valuation → Task B10 (s14-ret)
- Spec §5 Operational Model → Task B8 (s12-op)
- Spec §6 Phase 2 Capital Raise — Time-to-Close → Task B11 (s15-rm Gantt)
- Spec §7.1 Deck changes → Tasks B1–B14
- Spec §7.2 Master plan markdown → Tasks A7, A8
- Spec §7.3 Master plan render → Tasks A1–A6
- Spec §7.4 Land framing → Task B1
- Spec §8 Open Items → not in scope (deferred)
- Spec §9 Decision History → recorded in spec, not implemented
- Spec §10 Next Step → this plan

**2. Placeholder scan:** Searched for "TBD", "TODO", "fill in", "implement later", "similar to" — none in execution steps. Two intentional references to spec §-numbers in self-review section above.

**3. Type consistency:** Color constants (`PH_PHASE1`, `PH_PHASE2`) defined in A1 and used in A3 by same name. FTE numbers (32 / 47) consistent across A8 and B8. Capital figures ($22.7M / $18–22M) consistent across spec, B5, B6, B9, B10, B12.

**4. Frequent commits:** 12 commits in the plan (A1, A2, A3, A4, A5, A6, A7, A8, A9 final, B15, C2). Each scoped to one logical change.

**5. Out of scope items flagged:**
- Pre-existing untracked files (analysis.html, analyze.py, baja_wellness_deck.html before this plan, drone mp4 modifications) — flagged in Task C3 as out of scope.
- Phase 2 footprint pre-permitting — spec §8 open item, deferred to raise structuring.
- Comp-set valuation history (Aman/Habitas/Paradero) — spec §8 optional enhancement, deferred.
