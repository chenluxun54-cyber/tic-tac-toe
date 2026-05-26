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

GitHub repos:
- `https://github.com/chenluxun54-cyber/tic-tac-toe` — tictactoe + globe
- `https://github.com/chenluxun54-cyber/carbon-compliance-agent` — carbon agent (active project)

All remotes use HTTPS with token auth.

## Running the projects

### tictactoe / globe (no build step, CDN-loaded)
```bash
open tictactoe.html
open globe.html
```

### carbon-compliance-agent (active project)
```bash
cd ~/Desktop/carbon_skill

# Use Claude (default)
export ANTHROPIC_API_KEY=sk-ant-...
python3 -m uvicorn agent:app --reload --port 8000

# Use MiniMax
export MODEL_PROVIDER=minimax
export MINIMAX_API_KEY=your-key
python3 -m uvicorn agent:app --reload --port 8000
```

Open `http://localhost:8000` in browser (NOT the HTML file directly).

## Architecture

### tictactoe.html / globe.html

Single-file apps (HTML + CSS + JS, no bundler, no npm).

**tictactoe.html** — `board` (`string[9]`), `current` (`'X'`|`'O'`), `gameOver`, `scores`. Click → `render()` → `checkWin()`. `WINS` = 8 hardcoded triples. Scores survive `init()` but reset on page reload.

**globe.html** — Globe.GL 2.30.0 + topojson-client@3. Fetches `world-atlas countries-50m.json` → white-stroke borders. `COUNTRY_NAMES` maps ISO numeric → Chinese names. POV: lat 28, lng 110, alt 2.5. Auto-rotates at 0.35, stops on first user interaction.

### carbon-compliance-agent (`~/Desktop/carbon_skill/`)

FastAPI backend + streaming SSE chat UI. Single-file architecture (no bundler).

**Files:**
- `agent.py` — FastAPI app, agent loop, SSE streaming, provider switching
- `data_loader.py` — loads `carbon_database.xlsx` (sheets: `company_data`, `industry_rankings`)
- `scorer.py` — scores companies across 6 dimensions (total 100 pts)
- `config.py` — dimension definitions, indicator weights, scoring rules
- `index.html` — 5-tab dashboard UI (评分概览, 维度详情, 历史趋势, AI对话, 政策库)
- `policies.py` — 15 global carbon policies database (Global×6, EU×3, China×6)
- `generate_mock_data.py` — generates mock Excel data for COMP_001~COMP_010, year 2024

**Provider switching** — controlled by `MODEL_PROVIDER` env var:
- `anthropic` (default): uses `anthropic.AsyncAnthropic`, model `claude-sonnet-4-6`
- `minimax`: same Anthropic SDK, `base_url="https://api.minimaxi.com/anthropic"`, model `MiniMax-Text-01`

**Tools (3 total, Anthropic tool-use format):**
- `carbon_score(company_id, report_year)` — Returns 6-dimension scores + industry percentile ranking
- `search_policies(keyword?, industry?, jurisdiction?)` — Search policy library by filter
- `get_policy_detail(policy_id)` — Full policy details with compliance examples

**API endpoints:**
- `POST /chat` — SSE streaming chat
- `POST /new_session` — create session
- `GET /companies` — list all companies
- `GET /score/{company_id}/{year}` — raw score JSON
- `GET /history/{company_id}` — last 3 years trend
- `GET /provider` — current provider + model name
- `GET /policies` — list policies (optional: `?keyword=&industry=&jurisdiction=`)
- `GET /policies/{policy_id}` — full policy detail
- `POST /gap_analysis` — SSE streaming compliance gap analysis (body: `{company_id, report_year}`)

**Policy Library tab (5th tab):**
- Left panel: 3-level navigation stack (Level 1: card list → Level 2: policy detail → Level 3: example detail)
- Each level uses CSS `translateX` slide animation; policyCache dict avoids re-fetching on back navigation
- Right panel: dedicated policy AI chatbox with its own session (separate from main AI Chat tab)
- `policies.py` schema per policy: `id`, `name`, `jurisdiction`, `effective_date`, `category`, `source_url`, `summary`, `key_requirements`, `industries`, `compliance_examples` (with `problem`, `action`, `result`, `example_url`), `tags`

**Gap Analysis (Tab 1 — 评分概览):**
- "📊 生成合规差距分析" button appears after querying a company
- Calls `POST /gap_analysis` → stateless `gap_analysis_stream()` in `agent.py`
- Claude receives score JSON directly (no carbon_score tool call needed), calls search_policies/get_policy_detail, streams 4-section Markdown report
- Report sections: 📊总体评价 · ⚠️主要差距 (table) · 🗺️改进路线图 · 📜关键合规政策
- SSE field is `content` (not `text`) — important for frontend handler

**CSS layout gotcha (flex-shrink):**
- `.tab-content.active` is `display: flex; flex-direction: column` — all direct children need `flex-shrink: 0` or they compress instead of triggering scroll
- `.gap-content` uses `height: calc(100vh - 280px)` (not max-height) for reliable scrollbar

**Scoring dimensions (100 pts total):**
- D1 碳排放强度 (28) · D2 能源结构清洁度 (17) · D3 减碳动态表现 (18)
- D4 资源利用效率 (11) · D5 碳管理成熟度 (21) · D6 信息披露透明度 (5)

**Known issues / gotchas:**
- Run via `python3 -m uvicorn` not `uvicorn` (not on PATH)
- Must open `http://localhost:8000` through server — opening HTML directly breaks API calls
- MiniMax error 1008 = insufficient balance, top up account
- Port conflict: `kill $(lsof -t -i:8000)` then restart

### 双碳表现 Excel Analysis (`~/Desktop/`)

**Files:**
- `双碳表现.xlsx` — source data: A股数据 sheet, 5,524 rows × 29 cols, A-share ESG metrics 2021–2024
- `双碳分析.py` — analysis script (pandas + statsmodels)
- `双碳分析结果.xlsx` — output workbook (8 sheets)

**Data cleaning steps (in order):**
1. **Column normalisation** — `str.replace(r'\s+', ' ')` collapses embedded newlines in column headers
2. **Fossil fuel text parsing** — `化石能源消耗量` cols are free-text; regex extracts first numeric token, rest → NaN
3. **Force-numeric** — `pd.to_numeric(..., errors="coerce")` on all metric cols; unparseable → NaN
4. **Derived cols** — `total_emissions = scope1 + scope2`; `fossil_share = fossil / (fossil + non_fossil)`, div-by-zero → NaN
5. **No imputation** — missing values stay NaN; no mean/median fill
6. **Per-analysis row filtering** — rows dropped only when both target variables are missing for that specific model (not globally)
7. **Outlier handling (charts only)** — 99th-percentile cap on scatter plots for visual clarity; regression samples are unwinsorized

**Column name gotcha:** raw headers contain `\n` — must use `.str.replace(r'\s+', ' ', regex=True)` not just `.str.strip()` before any column lookup.

**Fossil col gotcha:** `find_cols(df, "化石能源消耗量")` also matches `非化石能源消耗量` (substring). Filter with `"非化石" not in c`.

**Output sheets:** 1_清洗数据 · 2_描述统计 · 3_相关性矩阵 · 4_简单回归_2024 · 5_多元回归_2024 · 6_面板回归 · 7_行业分层回归 · 8_趋势图

**Regression models:**
- Simple OLS (2024): `GHG_intensity = β₀ + β₁·energy_intensity`
- Multiple OLS (2024): adds clean energy ratio, fossil share, industry dummies
- Panel FE (2021–2024): within-estimator (demeaned), HC1 robust SE
- Industry-stratified OLS: top 10 industries by sample size
