---
title: Developer guide
has_children: false
nav_order: 11
---

# Developer Guide

If anybody wants to contribute to OrecchietTetris, feel free to contact us at <orecchiettetris@proton.me>. It is also possible to open [Issues on GitHub](https://github.com/unibo-dtm-se-2425-OrecchietTetris/OrecchietTetris-artifact/issues).

## Internal Conventions

### Naming

Files and modules are written in `snake_case`, while classes use `PascalCase`. Interfaces are abstract base classes with an `I` prefix. Enumerations for domain values combine a `PascalCase` type name with `UPPER_CASE` members (`EventType.BOARD_UPDATED`, `ShapeType.I_SHAPE`).

### Code Style

The style is enforced by flake8 and by a strict mypy configuration (untyped definitions are forbidden in production code, and only relaxed for `tests.*`). Full type annotations are therefore mandatory in the production code.

> It is strongly suggested to install pre-commit hooks to ensure code quality. See section "Development Environment Setup" below.

### Versioning and Development Workflow

The team follows the **Conventional Commits** specification, enforced by **commitlint** through a `commit-msg` hook. Please refer to [Development](../04-development/) chapter for more information regarding the adopted Git workflow.

> It is strongly suggested to install pre-commit hooks to enforce such specification. See section "Development Environment Setup" below.

## Development Environment Setup

The project is managed with **Poetry**, which creates a project-local `.venv/`. Starting from a fresh clone:

```bash
git clone https://github.com/unibo-dtm-se-2425-OrecchietTetris/OrecchietTetris-artifact
cd OrecchietTetris-artifact
poetry install     # Python dependencies → .venv/
npm install        # Node tooling (semantic-release, commitlint)
poe hooks          # (optional) install the commit-msg / pre-commit hooks
```

`poe hooks` also enables checks before creating a commit: runs a static analysis with `flake8` and `mypy`, then proceeds with the execution of the tests. If all the checks pass, the commit is created.

> `pre-commit` acts as a safety net: even without any IDE integration, the same static checks are executed automatically at commit time.

### Running tests and checks

All commands are defined as `poe` (poethepoet) tasks, so that they behave identically on a local machine and in the CI pipeline:

```bash
poe test            # full test suite (pytest -v)
poe coverage        # tests with coverage (term-missing)
poe coverage-html   # HTML coverage report
poe mypy            # strict type checking
poe flake8          # linting
```

> Quality checks run through `poe`, so any IDE can be connected to the same commands. Hence, no editor-specific configuration is needed to reproduce the CI results.
