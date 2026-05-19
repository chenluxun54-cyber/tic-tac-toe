# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

**Every project gets its own git repository and GitHub repo. Commit and push after every meaningful change — no work should remain local.**

Rules:
- For every new project: `git init`, then `gh repo create <name> --public --source=. --remote=origin --push`
- Commit after each logical unit of work (feature added, bug fixed, file created)
- Never batch unrelated changes into one commit
- Push to `origin main` immediately after every commit
- Write clean, descriptive commit messages (what changed and why, not just "update")

```bash
# New project setup
git init
gh repo create <project-name> --public --source=. --remote=origin --push

# Ongoing workflow
git add <files>
git commit -m "short description of what and why"
git push origin main
```

Current project GitHub repo: `https://github.com/chenluxun54-cyber/tic-tac-toe`  
Remote is configured for HTTPS with token auth.

## Running the project

No build step. Open any HTML file directly in a browser:
```bash
open tictactoe.html
open globe.html
```

## Files

### tictactoe.html — Tic-Tac-Toe game

Single-file app; all HTML, CSS, and JS in one file.

**Game state** (module-level vars):
- `board` — `string[9]`, values `''`, `'X'`, or `'O'`
- `current` — whose turn (`'X'` or `'O'`)
- `gameOver` — boolean, blocks clicks after win/draw
- `scores` — `{ X, O, draw }`, persists across restarts

**Flow:** click on `#board` → update `board[i]` → `render()` → `checkWin()` → update status/scores. `init()` resets board but preserves scores.

`WINS` is a hardcoded array of the 8 winning index triples used by `checkWin()`.

### globe.html — Interactive 3D Globe

Single-file app using **Globe.GL** (v2.30.0) for WebGL rendering.

**Key dependencies (CDN):**
- `globe.gl` — wraps Three.js for 3D earth rendering
- `topojson-client` — decodes `world-atlas` TopoJSON into GeoJSON polygon features

**Data flow:** `fetch` world-atlas countries-50m.json → `topojson.feature()` → `globe.polygonsData()` renders country borders as white stroke polygons over a transparent fill.

**Globe configuration:**
- Texture: `earth-day.jpg` + `earth-topology.png` bump map (both from `three-globe` package on unpkg)
- Background: `night-sky.png` (space look)
- Country labels: hover tooltip using `COUNTRY_NAMES` map (ISO numeric → Chinese name)
- Initial POV: lat 28, lng 110, altitude 2.5 (centered over Asia)
- Auto-rotate stops on user interaction via `controls().addEventListener('start', ...)`

`COUNTRY_NAMES` is a hardcoded ISO 3166-1 numeric → Chinese name lookup table in the script.
