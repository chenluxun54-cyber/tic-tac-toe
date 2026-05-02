# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

After every change, commit and push:
```bash
git add <files>
git commit -m "descriptive message"
git push origin main
```

GitHub repo: `git@github.com:chenluxun54-cyber/tic-tac-toe.git`  
SSH auth is configured via `~/.ssh/github_key`.

## Running the project

No build step. Open `tictactoe.html` directly in a browser:
```bash
open tictactoe.html
```

## Architecture

Single-file app — all HTML, CSS, and JS live in `tictactoe.html`.

**Game state** (module-level vars):
- `board` — `string[9]`, values `''`, `'X'`, or `'O'`
- `current` — whose turn (`'X'` or `'O'`)
- `gameOver` — boolean, blocks clicks after win/draw
- `scores` — `{ X, O, draw }`, persists across restarts

**Flow:** click on `#board` → update `board[i]` → `render()` → `checkWin()` → update status/scores. `init()` resets board but preserves scores.

`WINS` is a hardcoded array of the 8 winning index triples used by `checkWin()`.
