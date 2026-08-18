# AGENTS.md

General developer/agent guidance for this repository lives in `CLAUDE.md` (Dagster
Development Guide), `docs/docs/about/contributing.md`, and the `.claude/` docs. Prefer
those for standard commands (`make ruff`, `make pyright`, `pytest`, `just`/`yarn`
targets, etc.).

## Cursor Cloud specific instructions

This section captures non-obvious, durable facts for agents running in the Cursor Cloud
VM, where the environment update script has already installed dependencies.

### Python environment

- Python dependencies live in a virtualenv at `/workspace/.venv` (Python 3.12). It is
  auto-activated for interactive shells via `~/.bashrc`, so most `Shell` tool calls and
  tmux terminals already have `dagster`, `dagster-webserver`, `dg`, `pytest`, and `ruff`
  on `PATH`. In a non-interactive script that does not source `~/.bashrc`, either
  `source /workspace/.venv/bin/activate` first or call binaries by full path
  (e.g. `/workspace/.venv/bin/python`).
- `uv` and `just` are installed at `/usr/local/bin`. Dagster uses `uv` (not `pip`) for
  installs; all Dagster packages are installed **editable**, so Python source edits are
  picked up without reinstalling. Only rerun
  `python scripts/install_dev_python_modules.py` when a `setup.py`'s dependencies or
  console entry points change.

### Web UI

- `yarn` here is Yarn 4 (via Corepack, pinned by `packageManager`); the first
  `yarn install` may download the Yarn 4 bundle. JS workspaces live under `js_modules/`.
- The webserver serves a **prebuilt** UI from
  `python_modules/dagster-webserver/dagster_webserver/webapp/build`. That build is
  produced by `just rebuild_ui` (runs `yarn install` + `yarn workspace @dagster-io/app-oss build`)
  and is NOT rebuilt on every startup. After changing UI source, run `just rebuild_ui`
  (or, for live-reload UI dev, run the GraphQL server on one port and `cd js_modules &&
  make dev_webapp` on another, per `contributing.md`).

### Running the app

- Fastest way to run Dagster end-to-end (webserver + daemon) against a code location:
  `DAGSTER_HOME=/tmp/dagster_home dagster dev -f <path/to/defs.py> -h 0.0.0.0 -p 3000`.
  Set `DAGSTER_HOME` to a writable dir (a missing `dagster.yaml` there is only a
  warning). The UI is then at `http://localhost:3000`; GraphQL is at `/graphql`.
- To materialize assets from the CLI instead of the UI:
  `dagster asset materialize --select '*' -f <path/to/defs.py>`.
