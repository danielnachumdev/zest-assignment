# zest-assignment

**Interview assignment: async job dispatcher** — load tasks from JSON, run mock jobs through a worker pool, retry failures with backoff, and write process-safe logs plus a run summary.

Python **3.11+** recommended. Runtime deps: `tqdm`, `danielutils` (see `requirements.txt`).

## Why this repo?

A staged take-home that grows from a simple sync loop into a resilient async dispatcher — useful as a reference for worker pools, retry/backoff, and structured task logging.

- **Stage 1** — sync dispatch from `jobs.json` → `tasks.log`
- **Stage 2** — parallel execution via `AsyncWorkerPool` (`danielutils`) sized to CPU count
- **Stage 3** — up to 3 retries with exponential backoff (1s, 2s, 4s), failure logging, and a printed/logged summary
- **Mock jobs** — `Prowler`, `LongRunningJob`, `Shooker` in `jobs.py` (async `execute`)
- Assignment briefs: `Stage1_Assignment_py.docx` … `Stage3_Assignment_py.docx`

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Stage 1 (sync)
python process_tasks.py

# Stages 2–3 (async pool + retries)
python process_tasks_resilient.py
```

Both writers create/overwrite `tasks.log` in the repo root. Edit `jobs.json` to change the task list.

## What you get

| Surface | Role |
| -------- | ---- |
| `jobs.json` | Task list (`job_name` + `payload`) |
| `jobs.py` | `BaseJob` + mock job classes |
| `process_tasks.py` | Stage 1 sync dispatcher |
| `process_tasks_resilient.py` | Async worker pool, retries, summary |
| `tasks.log` | Generated run log (gitignored if configured; recreate by running) |

## Contributing & development

Agent/developer workflow (layout, run commands, CI/semver labels, secrets): see [`AGENTS.md`](./AGENTS.md).

Report security issues privately via [GitHub advisories](https://github.com/danielnachumdev/zest-assignment/security/advisories/new) when available — please do not open a public issue for vulnerabilities.
