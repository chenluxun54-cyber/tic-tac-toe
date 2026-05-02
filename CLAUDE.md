# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

**Commit and push after every meaningful change** — no work should remain local. This ensures the project is never lost and any state can be reverted.

Rules:
- Commit after each logical unit of work (feature added, bug fixed, file created)
- Never batch unrelated changes into one commit
- Push to `origin main` immediately after every commit
- Use clear, specific commit messages describing what changed and why

```bash
git add <files>
git commit -m "short description of what and why"
git push origin main
```

GitHub repo: `https://github.com/chenluxun54-cyber/tic-tac-toe`  
Remote is configured for HTTPS with token auth.

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
