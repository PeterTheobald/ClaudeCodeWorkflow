---
name: qa
description: >-
  Run a thorough quality-assurance pass over the codebase. Reads project docs,
  scans the code for best-practice violations, security issues, modularity and
  maintainability problems, and safety gaps (e.g. missing type annotations),
  runs a linter such as ruff, and checks unit and integration test coverage.
  TRIGGER when the user asks to "run QA", "do a quality pass", "audit the code",
  or otherwise wants a broad correctness/security/maintainability + test-coverage
  review of the project.
---

# QA

A structured quality-assurance pass over the current project. Work through the
phases below in order. Gather findings as you go, then deliver a single
prioritized report at the end — do not stop after each phase.

## 0. Orient

Before auditing, understand the project:

1. Read the available docs: `README*`, `CLAUDE.md`, `docs/`, `CONTRIBUTING*`,
   architecture notes, and any `pyproject.toml` / `package.json` / `Cargo.toml`
   to learn the language, frameworks, entry points, and existing conventions.
2. Identify the project's stack and toolchain (test runner, linter, formatter,
   type checker) from config files — prefer the tools the project already uses
   over imposing new ones.
3. Map the source layout: where the application code, tests, and configuration
   live.

State briefly what you found (stack, key tools, layout) so the user can confirm
you're auditing the right thing.

## 1. Best practices

Scan the codebase for departures from idiomatic, well-established practice for
the project's language and frameworks:

- Anti-patterns, dead code, copy-paste duplication, and overly long functions.
- Misused stdlib/framework APIs, deprecated calls, reinvented wheels.
- Inconsistent error handling; bare `except:` / swallowed errors; broad catches.
- Configuration or secrets hardcoded where config/env belongs.
- Logging vs. `print`, resource leaks (unclosed files/connections), mutable
  default arguments, and similar footguns.

## 2. Security

Look for security weaknesses. Focus on real, exploitable issues, not theater:

- Injection (SQL/command/template), unsafe deserialization (`pickle`, `yaml.load`),
  `eval`/`exec`, `subprocess(..., shell=True)` with untrusted input.
- Hardcoded credentials, tokens, or keys; secrets committed to the repo.
- Missing input validation / output encoding; path traversal; SSRF.
- Weak crypto, insecure randomness for security purposes, disabled TLS verification.
- AuthN/AuthZ gaps and insecure defaults.
- Vulnerable or outdated dependencies (check lockfiles; run `pip-audit` /
  `npm audit` / `cargo audit` if available).

## 3. Modularity & maintainability

Assess structure and long-term health:

- Tight coupling, leaky abstractions, god objects/modules, circular imports.
- Single-responsibility violations; logic that should be extracted/shared.
- Clear seams for testing (dependency injection vs. hardcoded globals).
- Naming, file organization, and module boundaries.
- Documentation/comment gaps for non-obvious logic.

Suggest concrete refactors, but keep them proportional — note them as
recommendations, don't rewrite the codebase unprompted.

## 4. Safety (types & robustness)

- Check for type annotations on functions, parameters, and return values.
  Flag missing or `Any`-heavy annotations in Python; missing types in TS, etc.
- Run the project's type checker if configured (`mypy`, `pyright`, `tsc`).
  If Python and none is configured, note that adding one would help.
- Look for unhandled edge cases: empty collections, `None`/null, off-by-one,
  numeric overflow, timezone/encoding assumptions.

## 5. Lint

Run the project's linter. For Python, prefer **ruff**:

```bash
ruff check .
```

If ruff isn't installed, try `python -m ruff`, then fall back to `flake8` /
`pylint`. For JS/TS use `eslint`; for Rust `cargo clippy`; for Go `go vet` /
`golangci-lint`. Run any configured formatter in `--check` mode (e.g.
`ruff format --check`, `prettier --check`, `black --check`).

Report the lint summary. Do **not** auto-fix unless the user asks — surface the
counts and notable categories first.

## 6. Test coverage

- Locate the test suite and run it (e.g. `pytest`, `npm test`, `cargo test`).
- Measure coverage if tooling is available (`pytest --cov`, `coverage`, `nyc`).
- Distinguish **unit** tests (isolated, fast) from **integration** tests
  (cross-component, I/O, external services). Note whether both exist.
- Identify untested or under-tested modules, especially critical paths and the
  security-sensitive code found in phase 2.
- Recommend specific tests worth adding; offer to write them.

## Reporting

Deliver one consolidated report grouped by the categories above. For each finding:

- **Severity**: critical / high / medium / low.
- **Location**: `path/to/file.py:line` (clickable).
- **Issue**: what's wrong.
- **Recommendation**: concrete fix.

Lead with a short summary (counts per category, top 3 most important things to
fix). Then offer to act: apply lint fixes, add missing types, write missing
tests, or perform a specific refactor. Make changes only after the user picks —
this skill audits by default; it does not rewrite code unprompted.
