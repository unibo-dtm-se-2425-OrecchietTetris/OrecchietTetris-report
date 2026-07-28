---
title: CI/CD
has_children: false
nav_order: 9
---

# CI/CD

This section describes the automated pipeline that takes a commit from a developer's machine all the way to a published PyPI release, plus the automation that keeps dependencies up to date.

## Automated Actions

Here are summarised the automated actions automatised in the CI/CD workflow. Actions 1-5 are implemented as a single GitHub Actions workflow (`deploy.yml`), structured as chained jobs (`check` → `test` → `test-release` → `release` → `publish-pypi`), each depending on the previous one succeeding, plus two independent, declarative bot configurations (`dependabot.yml`) that GitHub run on their own schedule outside of this workflow, which correspond to Action 6.

| Action | Description | Reason |
| ------ | ----------- | ------ |
| 1 | Static analysis / code quality: a syntax compile check, strict-mode `mypy` type checking, and `flake8` linting all run before anything else | Catching type errors, lint issues, and syntax problems before tests even run keeps feedback fast and cheap |
| 2 | Automated testing: the full test suite runs with coverage across a matrix of 3 operating systems (Ubuntu, Windows, macOS) x 4 Python versions (3.11-3.14), for 12 combinations in total | Gives real confidence the package behaves the same for every user of the published package, not just on the maintainers' machines |
| 3 | Release to TestPyPI: a build of the package is published to the TestPyPI staging index and then actually pip-installed back from it, as a smoke test that the package is installable before anything reaches the real index | Checks whether the package can actually be installed via `pip install` from there, catches packaging mistakes (missing files, broken metadata, wrong entry points) before they reach the real, non-revocable PyPI index |
| 4 | Automatic version bump: `semantic-release` inspects the Conventional Commit messages since the last release, computes the next semantic version, updates the changelog, and creates the corresponding git tag — with no manual version editing by a human | Removes an entire class of human error (forgetting to bump, bumping the wrong part of the version) and ties the version number directly to the semantics of what actually changed, since it is derived from Conventional Commit types |
| 5 | Release to PyPI: once a new version has actually been computed, the exact tagged commit is checked out, rebuilt, and published to the real PyPI index | Distribute the application |
| 6 | Dependency updates: Dependabot are configured to open pull requests for outdated npm/pip dependencies and GitHub Actions versions on a recurring schedule | Keeps the project from silently drifting onto outdated or vulnerable dependencies without needing a human to periodically remember to check |

## GitHub Actions Exploitation

### Triggers

The workflow (`.github/workflows/deploy.yml`) runs on:

- `push`, to any branch except `dependabot/**` (avoiding a redundant run on the bot's own branch, since the same change will also be tested via its pull request), and ignoring pushes that only touch non-code files (`README.md`, `CHANGELOG.md`, `LICENSE`, etc.) via `paths-ignore`.
- `pull_request`, so proposed changes are validated before merge.
- `workflow_dispatch`, allowing a maintainer to trigger a run manually from the GitHub UI.

### Deploy Workflow

In the deployment workflow, each step depends on the previous one.

1. `check` (Preliminary Checks): installs Poetry, then runs `poe compile` (syntax check), `poe mypy` and `poe flake8` (static analysis), and `poe coverage`/`coverage-report`/`coverage-html` (tests with coverage), uploading the HTML coverage report as a build artifact.
2. `test`: runs the test suite (`poe test`) across a matrix of ubuntu/windows/macos and Python 3.11 through 3.14 (12 jobs in parallel), with `fail-fast: false` so one failing combination doesn't hide results from the others.
3. `test-release`: runs only if the branch is `main`/`master` and the event is not a pull request. It builds the package with `poetry build` and publishes it to TestPyPI, then waits 60 seconds and attempts a real `pip install` from TestPyPI as a smoke test (tolerating failure here, since index propagation can occasionally take longer than 60 seconds).
4. `release`: runs `semantic-release` to compute the next version from Conventional Commits, update `CHANGELOG.md`, commit, and tag. It exposes whether a new release was actually published (`new-release-published`) and its version number as job outputs, and uses a `concurrency` group (`deploy`, `cancel-in-progress: false`) so two releases can never run over each other.
5. `publish-pypi`: skipped entirely if there was nothing to release — e.g. a `docs:` or `chore:` only commit. It checks out the exact tag semantic-release just created, rebuilds, and publishes that build to the real PyPI.

#### Authentication without secrets: Trusted Publisher Management

Neither the TestPyPI nor the PyPI publish step uses a stored API token. Instead, the workflow requests `permissions: id-token: write` at the top level, and the `pypa/gh-action-pypi-publish` action uses that to obtain a short-lived OpenID Connect (OIDC) token from GitHub at run time. PyPI (and TestPyPI) are separately configured, on their side, with a Trusted Publisher entry that trusts this exact combination of GitHub repository, workflow filename, and GitHub Environment. PyPI validates the OIDC token against that registration and issues a short-lived, scoped upload token for that run only.

In this way, there is no long-lived token sitting in GitHub Secrets that could leak or need periodic rotation — the trust relationship lives entirely in PyPI's own configuration, tied to the specific repo/workflow/environment triple.

#### Other permissions and environment variables

- `permissions: contents: write` lets `semantic-release` push the changelog commit and the new tag back to the repository.
- `permissions: packages: write` is requested for potential GitHub Packages usage.
- `env: FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` was added to fix a warning: `deploy.yml` was originally pinning GitHub's JavaScript-based actions to Node v20, which GitHub started flagging as outdated. This flag forces those actions to run under a newer Node version instead. The underlying issue was later also addressed by updating some of the action/dependency versions used in `deploy.yml` directly.

### Dependency automation

Automated dependency updates are handled by Dependabot (`.github/dependabot.yml`): it watches both the `npm` and `pip` ecosystems at the repository root, checking weekly, opening up to 10 pull requests at a time, labeling them `dependencies`/`npm` or `dependencies`/`python`, and prefixing commit messages with `chore(deps)` to stay Conventional-Commits-compliant.
