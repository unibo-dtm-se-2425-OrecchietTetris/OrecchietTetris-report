---
title: CI/CD
has_children: false
nav_order: 9
---
# CI/CD
This chapter describes the automated pipeline that takes a commit from a developer's machine all the way to a published PyPI release, plus the automation that keeps dependencies up to date.

## Conceptual description
- What is automated:
    - Static analysis / code quality: a syntax compile check, strict-mode `mypy` type checking, and `flake8` linting all run before anything else.
    - Automated testing: the full test suite runs with coverage, then again across a matrix of 3 operating systems (Ubuntu, Windows, macOS) x 4 Python versions (3.11-3.14), for 12 combinations in total.
    - Release to TestPyPI: a build of the package is published to the TestPyPI staging index and then actually pip-installed back from it, as a smoke test that the package is installable before anything reaches the real index.
    - Automatic version bump: `semantic-release` inspects the Conventional Commit messages since the last release, computes the next semantic version, updates the changelog, and creates the corresponding git tag — with no manual version editing by a human.
    - Release to PyPI: once a new version has actually been computed, the exact tagged commit is checked out, rebuilt, and published to the real PyPI index.
    - Dependency updates: both Dependabot and Renovate are configured to open pull requests for outdated npm/pip dependencies and GitHub Actions versions on a recurring schedule.
- Why:
    - Catching type errors, lint issues, and syntax problems before tests even run keeps feedback fast and cheap.
    - Testing on a full OS x Python-version matrix, rather than just one environment, gives real confidence the package behaves the same for every user of the published package, not just on the maintainers' machines.
    - Publishing to TestPyPI first, and proving the package can actually be `pip install`-ed from there, catches packaging mistakes (missing files, broken metadata, wrong entry points) before they reach the real, non-revocable PyPI index.
    - Automating the version bump removes an entire class of human error (forgetting to bump, bumping the wrong part of the version) and ties the version number directly to the semantics of what actually changed, since it is derived from Conventional Commit types.
    - Automating dependency updates keeps the project from silently drifting onto outdated or vulnerable dependencies without needing a human to periodically remember to check.
- How: everything is implemented as a single GitHub Actions workflow (`deploy.yml`), structured as five chained jobs (`check` → `test` → `test-release` → `release` → `publish-pypi`), each depending on the previous one succeeding, plus two independent, declarative bot configurations (`dependabot.yml`, `renovate.json`) that GitHub/Renovate run on their own schedule outside of this workflow.

## Relevant details of GitHub Actions exploitation

### Triggers
The workflow (`.github/workflows/deploy.yml`) runs on:
- `push`, to any branch **except** `dependabot/**` and `renovate/**` (avoiding a redundant run on the bot's own branch, since the same change will also be tested via its pull request), and ignoring pushes that only touch non-code files (`README.md`, `CHANGELOG.md`, `LICENSE`, etc.) via `paths-ignore`.
- `pull_request`, so proposed changes are validated before merge.
- `workflow_dispatch`, allowing a maintainer to trigger a run manually from the GitHub UI.

### The five jobs
- **`check`** — Preliminary Checks: installs Poetry, then runs `poe compile` (syntax check), `poe mypy` and `poe flake8` (static analysis), and `poe coverage`/`coverage-report`/`coverage-html` (tests with coverage), uploading the HTML coverage report as a build artifact.
- **`test`** — depends on `check`; re-runs the test suite (`poe test`) across a matrix of `ubuntu-latest` / `windows-latest` / `macos-latest` and Python 3.11 through 3.14 (12 jobs), with `fail-fast: false` so one failing combination doesn't hide results from the others, and a 45-minute timeout per job.
- **`test-release`** — depends on `test`; runs only if the branch is `main`/`master` **and** the event is not a pull request (`if: (github.ref == 'refs/heads/main' || github.ref == 'refs/heads/master') && github.event_name != 'pull_request'`), i.e. only for real pushes to the mainline, never for PRs or feature branches. It builds the package with `poetry build` and publishes it to TestPyPI, then waits 60 seconds and attempts a real `pip install` from TestPyPI as a smoke test (tolerating failure here, since index propagation can occasionally take longer than 60 seconds).
- **`release`** — depends on `test-release`; runs `semantic-release` (via the `cycjimmy/semantic-release-action`, with the `@semantic-release/changelog`, `@semantic-release/exec`, and `@semantic-release/git` plugins) to compute the next version from Conventional Commits, update `CHANGELOG.md`, commit, and tag. It exposes whether a new release was actually published (`new-release-published`) and its version number as job outputs, and uses a `concurrency` group (`deploy`, `cancel-in-progress: false`) so two releases can never run over each other.
- **`publish-pypi`** — depends on `release`, and only runs `if: needs.release.outputs.new-release-published == 'true'` (i.e. skipped entirely if there was nothing to release — e.g. a `docs:` or `chore:` only commit). It checks out the exact tag semantic-release just created (`ref: v${{ needs.release.outputs.new-release-version }}`), rebuilds, and publishes that build to the real PyPI.

### Authentication without secrets: Trusted Publisher Management
Neither the TestPyPI nor the PyPI publish step uses a stored API token. Instead, the workflow requests `permissions: id-token: write` at the top level, and the `pypa/gh-action-pypi-publish` action uses that to obtain a short-lived OpenID Connect (OIDC) token from GitHub at run time. PyPI (and TestPyPI) are separately configured, on their side, with a **Trusted Publisher** entry that trusts this exact combination of GitHub repository, workflow filename, and GitHub Environment (`testpypi` / `pypi`, declared via the `environment:` key on the `test-release` and `publish-pypi` jobs). PyPI validates the OIDC token against that registration and issues a short-lived, scoped upload token for that run only.
The practical benefit: there is no long-lived `PYPI_API_TOKEN` sitting in GitHub Secrets that could leak or need periodic rotation — the trust relationship lives entirely in PyPI's own configuration, tied to the specific repo/workflow/environment triple.

### Other permissions and environment variables
- `permissions: contents: write` lets `semantic-release` push the changelog commit and the new tag back to the repository.
- `permissions: packages: write` is requested for potential GitHub Packages usage.
- `env: FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` was added to fix a warning: `deploy.yml` was originally pinning GitHub's JavaScript-based actions to Node v20, which GitHub started flagging as outdated. This flag forces those actions to run under a newer Node version instead. The underlying issue was later also addressed by updating some of the action/dependency versions used in `deploy.yml` directly, in a separate commit.

## Dependency automation
Automated dependency updates are handled by **Dependabot** (`.github/dependabot.yml`): it watches both the `npm` and `pip` ecosystems at the repository root, checking weekly (Mondays at 09:00, Europe/Rome), opening up to 10 pull requests at a time, labeling them `dependencies`/`npm` or `dependencies`/`python`, and prefixing commit messages with `chore(deps)` to stay Conventional-Commits-compliant.

