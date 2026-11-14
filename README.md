# template-python

A GitHub template for a new murphy360 Python project. It starts with everything
[murphy360/standards](https://github.com/murphy360/standards) asks of a project already in place: CI in one quick
`checks` job, the pre-commit hook, Dependabot, `AGENTS.md`, the Claude Code guard, a Dockerfile, the issue and pull
request templates and the milestone conventions.

## Start a project

```bash
gh repo create <name> --template murphy360/template-python --private --clone
```

CI is green on the first push, untouched. Then make it yours. These are the lines to change:

| File | What to change |
|---|---|
| `pyproject.toml` | `name`, `description`; add your dependencies |
| `src/my_service/` | rename the directory to your package name, and fix `tests/test_smoke.py` to import it |
| `.github/workflows/ci.yml` | `image: my-service` to your image name, and `push: false` to `push: ${{ github.event_name != 'pull_request' }}` (the template only builds, so it never publishes) |
| `AGENTS.md` | the title, and "This project" at the end; keep the two marker comments |
| `Dockerfile` | the `CMD`, once the project has a service to run |
| `README.md` | replace this file with your own |
| `docs/MILESTONES.md` | keep it, or change it where you work differently, and say why |

Keep the jobs in `ci.yml` that apply. Then apply the repository settings and rulesets once, as the owner:
`tools/repo-settings.sh <owner>/<name>` from a checkout of murphy360/standards (`--dry-run` first).

## Build and test

Tests run in Docker, not on the host. Use your own tag for the image.

```bash
docker build -t my-service:test .
docker run --rm my-service:test                          # runs pytest -q
docker run --rm -v "$PWD:/src" -w /src my-service:test pytest -q   # the same, on your working tree
```

The code rules, as CI runs them, are a pre-commit hook. Nothing else is installed: the hook brings its own ruff, the
one `pyproject.toml` pins.

```bash
pip install pre-commit && pre-commit install   # once per checkout
pre-commit run --all-files                     # every hook on every file: the one command before a push
```

Every file is clean, and CI checks every file on every run. There is no baseline: a finding anywhere fails.

## Take standards updates

CI calls the shared actions and workflows at a version tag, `murphy360/standards/...@v2`. A compatible change moves
the `v2` tag, so you get it with no edit. To move to a new major version, change every `@v2` in
`.github/workflows/ci.yml` to the new tag (for example `@v3`) and read the standards' release notes for what else to
change.

The other files are copies. `check-templates: true` fails CI when the shared text of `AGENTS.md` falls behind the
standard's; `python3 <standards>/tools/sync_templates.py .` rewrites it, and `--all` lists the other copies that
differ. Move the `code-rules` hook's `rev` in `.pre-commit-config.yaml` to each new release's commit. Dependabot
brings a new ruff in a pull request of its own: move the `ruff-pre-commit` rev to the same release in that pull
request (`check-ruff-version: true` fails until they agree).

## What is here

| Path | What it is |
|---|---|
| `.github/workflows/ci.yml` | CI: the quick checks in one job, the tests, the image, and the `result` job a ruleset requires |
| `.pre-commit-config.yaml` | the code rules, ruff format, shellcheck, actionlint and gitleaks, at the versions CI uses |
| `.github/dependabot.yml` | weekly grouped updates for actions, pip and docker; ruff alone |
| `.github/ISSUE_TEMPLATE/` | the ticket shape (`ticket.md`), and the blank issue beside it |
| `.github/pull_request_template.md` | the PR body: what and why, docs, known gaps, `Closes #N` |
| `AGENTS.md` | how an agent session works in the project; every agent tool reads it |
| `CLAUDE.md` | Claude Code's import of `AGENTS.md` |
| `.claude/` | what a Claude Code session refuses: deny rules and a guard hook before every shell command |
| `docs/MILESTONES.md` | the milestone conventions |
| `pyproject.toml`, `src/`, `tests/`, `Dockerfile` | a package with one passing test, and the image that runs it |
