# Codex Handoff

This file is repo-level context for future Codex sessions in TradingAgents. Read it before making changes.

## Current Work

- Active branch: `codex/pr-749-safe-edit`
- Fork remote: `https://github.com/ea-kelvin-yeung/TradingAgents.git`
- Upstream remote: `https://github.com/TauricResearch/TradingAgents.git` with push disabled locally.
- This branch started from upstream PR 749, which adds Codex OAuth / ChatGPT subscription authentication as an LLM provider.
- The branch is intended to make TradingAgents work with ChatGPT subscription auth, including long streamed Codex responses.

## Important Commits In This Session

- `0c3d5c9 fix(codex): stream OAuth SSE responses robustly`
  - Fixes `ChunkedEncodingError: Response ended prematurely` during long agent responses.
  - Uses `requests.post(..., stream=True)` for SSE responses instead of buffering the full body.
  - Decodes SSE incrementally with `iter_lines(decode_unicode=True)`.
  - Closes response objects correctly.
  - Keeps the 401 refresh path.
  - Retries one time on premature chunked stream termination.
  - Adds focused tests in `tests/test_codex_oauth.py`.
- `c581ad2 docs: analyze A Sir framework repurpose path`
  - Adds `docs/a_sir_repurpose_analysis.md`.
  - Explains how to repurpose TradingAgents for an A Sir-style framework: Direction -> Value -> Timing -> Risk.
  - Recommends minimal graph changes: prompt overlay, structured scorecard, missing-data discipline, and deterministic metric snapshot tools.

## Local Setup

Use Python 3.13 and `uv`.

```bash
uv venv --python 3.13 .venv
uv pip install -e . pytest
uv pip check
.venv/bin/pytest -q
```

Verified locally in this session:

```text
125 passed, 1 warning, 42 subtests passed
```

Focused Codex OAuth tests:

```bash
.venv/bin/pytest tests/test_codex_oauth.py -q
```

## ChatGPT Subscription Auth

OAuth tokens are stored outside the repo:

```text
~/.tradingagents/cache/codex_oauth.json
```

Rules:

- Never commit this token file.
- Never copy this token file between machines unless the user explicitly accepts the security risk.
- Prefer running the login flow separately on each machine.

Normal login:

```bash
.venv/bin/python scripts/codex_oauth_login.py --timeout 600
.venv/bin/python scripts/codex_oauth_smoke.py
```

Headless or remote login via SSH tunnel:

```bash
ssh -N -L 1455:127.0.0.1:1455 100.118.50.123
```

In another terminal:

```bash
ssh 100.118.50.123 'cd ~/PycharmProjects/TradingAgents && .venv/bin/python scripts/codex_oauth_login.py --no-browser --timeout 600'
```

Open the printed URL locally. The callback to `localhost:1455` is forwarded to the remote process, so the token is saved on the remote machine.

Note: the device-code flow may fail from some networks with:

```text
unsupported_country_region_territory
```

If that happens, use the browser callback flow with the SSH tunnel above.

## Checkpointing

Long CLI runs should use checkpointing so crashes resume instead of restarting from the first analyst.

```bash
tradingagents analyze --checkpoint
tradingagents analyze --clear-checkpoints
```

Checkpoint databases live under:

```text
~/.tradingagents/cache/checkpoints/<TICKER>.db
```

They are cleared automatically after a successful run. Use `--clear-checkpoints` before a fresh run if a saved state is stale.

## Known Prior Failure

The original AMZN run crashed in the Bear Researcher stage with:

```text
ChunkedEncodingError: Response ended prematurely
```

Cause: the Codex OAuth payload requested streaming, but the HTTP client did not set `stream=True`, so `requests` tried to buffer a long SSE response and failed when the chunked response closed early.

Fix: commit `0c3d5c9` streams and decodes SSE correctly. If this error reappears, first confirm the repo includes that commit or a later descendant:

```bash
git log --oneline -5
```

## Remote Machine Setup

Machine:

```text
100.118.50.123
```

Repo path:

```text
~/PycharmProjects/TradingAgents
```

Verified remote state before this handoff file was added:

```text
remote_head=c581ad2
branch=codex/pr-749-safe-edit
17 passed in tests/test_codex_oauth.py
125 passed, 1 warning, 42 subtests passed
```

After new commits are pushed, update the remote with:

```bash
ssh 100.118.50.123 'cd ~/PycharmProjects/TradingAgents && git pull --ff-only'
```

Remote auth was not completed in the prior session because the browser approval callback was not received before the temporary tunnel was stopped.

## Generated Files

Keep generated analysis output out of commits unless the user explicitly asks to version it.

Common generated paths:

```text
reports/
.venv/
.pytest_cache/
__pycache__/
tradingagents.egg-info/
```

The local worktree may have untracked `reports/` from AMZN/GOOG CLI test runs. Do not delete them unless the user asks.

## A Sir Framework Work

The repurpose analysis is at:

```text
docs/a_sir_repurpose_analysis.md
```

Main recommendation: keep the existing agent graph and add an A Sir mode around it:

- Prompt overlays for Direction -> Value -> Timing -> Risk.
- A structured scorecard emitted alongside narrative reports.
- Explicit missing-data handling.
- Deterministic snapshot tools for market regime, sector relative strength, and company metrics.

Do not rewrite the graph first. The minimal-change path is to add structured metrics and scoring around the current analyst, research, trading, and risk stages.

