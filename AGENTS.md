# AGENTS.md — eNet

Networking subsystem placeholder for the EmbeddedOS platform (link technologies, protocols, discovery). Status: **Planned — no implementation in this repository yet**. Working code lives in [`eos`](https://github.com/embeddedos-org/eos) at `net/`.

## Repo contents

- `README.md` — project overview, scope, and code-location pointers (authoritative).
- `docs/` — design notes, plans, audits and studies (protocol hardening, bridge evaluations, home-systems and eHub-Pro work). `docs/wiki/` is the wiki mirror (see Contributions).
- `llms.txt` (docs index for LLM tools) and `llms-full.txt` (hand-maintained concatenation of the source docs it lists; update it in the same commit as a source doc).
- `ci/check_docs.py` — the docs check that CI runs (see below).
- `.github/workflows/ci.yml` (docs check on pushes and pull requests to `master`), `.github/workflows/linked-issue.yml` (org linked-issue policy on pull requests) and `.github/PULL_REQUEST_TEMPLATE.md`.
- `LICENSE` — MIT.
- `.gitignore` — build/dist/object/lib/Python/venv/OS ignores.
- No root `CONTRIBUTING.md`, no `SECURITY.md`, no build/dependency manifest (`package.json`, `pyproject.toml`, `CMakeLists.txt`, `Makefile`, `pom.xml`, `Cargo.toml`, `go.mod`), no source code, no tests directory. Re-check with `git ls-files` before relying on this list.

## Build / Test / Lint

There is no build and there are no tests. The one command is the docs check that CI runs (`.github/workflows/ci.yml`, job "Docs check"):

```sh
python3 ci/check_docs.py
```

It needs only the Python standard library. It checks that every Markdown file under `docs/` and the root `README.md` has exactly one H1 (`docs/wiki/_Sidebar.md` is exempt) and that every relative link in them resolves. It does not check this file or `llms*.txt`. Do not invent other commands (no `make`, `cmake`, `npm`, `pytest`, etc.). Work on eNet behavior happens in `eos/net/` until the split condition (README "When code moves here") is met.

## Contributions

- No root contributing guide exists. Before proposing a change, read `README.md` (especially scope and "When code moves here"), keep changes scoped, and follow current automation/review requirements.
- Wiki mirror (navigation layer only, not authoritative): `docs/wiki/` (copies of Home, Getting-Started, Development, Security, FAQ, _Sidebar).

## Security

- No root `SECURITY.md` exists. Do not disclose suspected vulnerabilities in public issues. Use GitHub's private security advisory workflow when available, or a private maintainer channel. See `docs/wiki/Security.md`.
