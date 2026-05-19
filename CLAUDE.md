# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

**Every project gets its own git repository and GitHub repo. As you do work, commit and push each completed step immediately — no work should ever remain local only.**

This is a hard requirement: after every meaningful change, Claude must run `git add`, `git commit`, and `git push` before moving on. The goal is that the GitHub repo always reflects the current state of the work so nothing is ever lost.

Rules:
- For every new project: `git init`, then `gh repo create <name> --public --source=. --remote=origin --push`
- Commit and push after each logical unit of work (feature added, bug fixed, file created, refactor done)
- Never batch unrelated changes into one commit
- Never finish a task without pushing — GitHub is the source of truth
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

No build step. All dependencies are CDN-loaded. Open any HTML file directly in a browser:

```bash
open tictactoe.html
open globe.html
```

## Architecture

Both apps are **single-file** (HTML + CSS + JS in one file, no bundler, no npm).

### tictactoe.html

Module-level state: `board` (`string[9]`), `current` (`'X'`|`'O'`), `gameOver` (boolean), `scores` (`{X, O, draw}`).

Flow: click on `#board` → `board[i] = current` → `render()` → `checkWin()` → update status/scores.

`scores` is initialized once at script load, then `init()` uses `scores = scores || ...` so scores survive restarts but reset on page reload. `WINS` is the 8 hardcoded winning index triples.

### globe.html

CDN deps: `globe.gl@2.30.0` (Three.js wrapper), `topojson-client@3`.

Data flow: `fetch` world-atlas `countries-50m.json` from jsDelivr → `topojson.feature()` → `globe.polygonsData()` renders white-stroke country borders over a transparent fill. Fetch failures are silently swallowed — the globe still renders without borders.

`COUNTRY_NAMES` maps ISO 3166-1 numeric codes → Chinese country names (hardcoded in script). Hover tooltip uses this map, falling back to the raw numeric ID.

Globe config: earth-day texture + topology bump map + night-sky background (all from unpkg `three-globe`). Initial POV: lat 28, lng 110, altitude 2.5 (centered over Asia). Auto-rotate at 0.35 speed; stops permanently on first user interaction via `controls().addEventListener('start', ...)`.
