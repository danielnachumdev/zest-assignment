# zest-assignment – Agent Context

Implementation guide for coding agents changing **zest-assignment**. Product pitch and quick start: [`README.md`](README.md).

---

## Orientation

| Question | Answer |
| -------- | ------ |
| What is this? | Interview assignment: JSON task loader + sync/async job dispatcher with retries |
| App source | Flat repo root scripts (`jobs.py`, `process_tasks*.py`) — not a packaged library |
| Toolchain | **pip** + `requirements.txt` (no `pyproject.toml` / uv yet) |
| Python | **3.11+** recommended |
| Tests | None yet — do not invent a suite unless asked; thin CI uses `allow-empty: true` when present |
| Secrets | Never commit API keys, tokens, or live cloud credentials in fixtures/docs |

```text
.
├── .github/                      # CODEOWNERS, dependabot, thin CI (landing via adopt PR)
├── jobs.json                     # Input tasks
├── jobs.py                       # BaseJob + Prowler / LongRunningJob / Shooker
├── process_tasks.py              # Stage 1 sync dispatcher
├── process_tasks_resilient.py    # Async pool + retries + summary
├── requirements.txt              # tqdm, danielutils
├── Stage*_Assignment_py.docx     # Assignment briefs (binary; do not rewrite)
├── README.md                     # Marketing / user docs
└── AGENTS.md                     # This file
```

Shared CI (when thin workflows are on `main`): [`danielnachumdev/actions`](https://github.com/danielnachumdev/actions) (`@latest`). How to adopt: [`HOW-TO-USE.md`](https://github.com/danielnachumdev/actions/blob/v1/HOW-TO-USE.md).

---

## Develop

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python process_tasks.py
python process_tasks_resilient.py
```

| Task | Command |
| ---- | ------- |
| Sync dispatch (Stage 1) | `python process_tasks.py` |
| Async + retries (Stages 2–3) | `python process_tasks_resilient.py` |
| Install deps | `pip install -r requirements.txt` |

Generated `tasks.log` is overwritten each run.

### Dependencies

- **Runtime:** `tqdm`, `danielutils` (`AsyncWorkerPool` used by the resilient path)
- Prefer editing `requirements.txt` when adding deps; keep versions honest to what the scripts import

---

## Code style & domain notes

- Keep the assignment shape unless asked to package/refactor: root-level scripts, explicit `JOB_MAPPING` / `get_job_class_mapping()` dicts.
- **Jobs** implement `async def execute(self, payload)`; the resilient path awaits via `AsyncWorkerPool`. The Stage 1 script calls `execute` without `await` (historical sync-era entrypoint) — do not “fix” that casually without aligning Stage 1 requirements.
- **Retries:** non-success status or raised exceptions → up to 3 attempts with `FixedSecondsBackoffStrategy([1, 2, 4])`; all attempts logged; exhausted failures logged as `failed`.
- **Logging:** file writes under an `asyncio.Lock` in the resilient pool so concurrent workers do not interleave lines.
- **Unknown jobs:** `jobs.json` may include unsupported `job_name` values (e.g. `NotARealJob`) — handle/log without crashing the pool.
- Prefer stdlib `logging` only if extending beyond the current one-line file format; do not expand scope for a take-home unless requested.
- Do not commit or rewrite the `.docx` assignment briefs.

---

## PRs, CI, and releases

1. Branch off `main`; open a PR into `main`.
2. Attach **exactly one** semver label: `noBump` | `bumpPatch` | `bumpMinor` | `bumpMajor`.
3. **CI** — when present, thin `.github/workflows/ci.yml` → shared `ci.yml@latest` (expect `allow-empty: true` while there is no test suite) + `semver-ci.yml@latest`. Prefer required checks **`CI / CI`** and **`Semver / Semver`**.
4. **CD** — this repo has **no** thin `cd.yml` / PyPI publish path; semver labels still gate PRs. Prefer `noBump` for docs-only and Dependabot.
5. Dependabot PRs stay on `noBump`.
6. Change shared CI/semver behavior in **`danielnachumdev/actions`**, not by re-vendoring workflows here.

### Repo gates

- Semver labels exist on the repo (`noBump`, `bumpPatch`, …)
- Thin CI + CODEOWNERS + Dependabot are adopted via shared actions (see open/merged HOW-TO PRs); keep docs aligned when those land on `main`
- Private vulnerability reporting via GitHub Security Advisories when enabled

---

## Sensitive / ignored

Do not commit: `.venv/`, `venv/`, `__pycache__/`, API keys, tokens, or live cloud account IDs beyond fabricated assignment fixtures. Prefer dummy payloads (e.g. fake `account_id` values already in `jobs.json`). Respect `.gitignore`.
