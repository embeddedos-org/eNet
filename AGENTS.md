# AGENTS.md — eNet

Networking subsystem placeholder for the EmbeddedOS platform (link technologies, protocols, discovery). Status: **Planned — no implementation in this repository yet**. Working code lives in [`eos`](https://github.com/embeddedos-org/eos) at `net/`.

## Repo contents (master @ 67c544c)

- `README.md` — project overview, scope, and code-location pointers (authoritative).
- `LICENSE` — MIT.
- `.gitignore` — build/dist/object/lib/Python/venv/OS ignores.
- No root `CONTRIBUTING.md`, no `SECURITY.md`, no build/dependency manifest (`package.json`, `pyproject.toml`, `CMakeLists.txt`, `Makefile`, `pom.xml`, `Cargo.toml`, `go.mod`), no `.github/workflows`, no tests directory. Verified via `git ls-files` (3 tracked files).

## Build / Test / Lint

No build, test, or lint commands exist in this repository. The root README documents no commands, and there are no manifests, workflows, or test paths to run. Do not invent commands (no `make`, `cmake`, `npm`, `pytest`, etc.). Work on eNet behavior happens in `eos/net/` until the split condition (README "When code moves here") is met.

## Contributions

- No root contributing guide exists. Before proposing a change, read `README.md` (especially scope and "When code moves here"), keep changes scoped, and follow current automation/review requirements.
- Wiki mirror (navigation layer only, not authoritative): `docs/wiki/` (copies of Home, Getting-Started, Development, Security, FAQ, _Sidebar).

## Security

- No root `SECURITY.md` exists. Do not disclose suspected vulnerabilities in public issues. Use GitHub's private security advisory workflow when available, or a private maintainer channel. See `docs/wiki/Security.md`.
