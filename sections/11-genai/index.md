---
title: GenAI
has_children: false
nav_order: 5
---
# GenAI
This chapter documents how generative AI was used throughout the development of OrecchietTetris, following the AI-DECLARATION.md standard, and explains the general criteria we followed when prompting and reviewing AI-generated work.

## AI declaration
The table below summarizes our AI usage per process and per component, as tracked in [`AI-DECLARATION.md`](https://github.com/unibo-dtm-se-2425-OrecchietTetris/OrecchietTetris-artifact/blob/master/AI-DECLARATION.md) at the root of the artifact repository.

- version: `0.1.2`
- level: `copilot`
- Processes:
    - design: `none`
    - implementation: `copilot`
    - testing: `auto`
    - documentation: `pair`
    - review: `pair`
    - deployment: `assist`
- Components:
    - `OrecchietTetris/audio/`: `auto`
    - `OrecchietTetris/leaderboard/`: `auto`
    - `OrecchietTetris/model/`: `pair`
    - `OrecchietTetris/view/`: `copilot`
    - `tests/`: `auto`
    - `CLAUDE.md`: `auto`
    - `AGENTS.md`: `auto`
    - `README.md`: `pair`

### Notes
This project was developed using **Claude Code** (Anthropic) as an AI coding assistant.

- **Design (`none`):** all architectural decisions — Observer pattern, interface-first design, import hierarchy, Model-View separation, Singleton pattern — were made independently by the human developer.
- **Implementation (`copilot`):** the AI handled whole implementation tasks on request, with the human reviewing and approving each step. The Kivy view layer (`OrecchietTetris/view/`: `copilot`) was largely AI-driven; the model layer (`OrecchietTetris/model/`: `pair`) was developed more collaboratively with the human retaining stronger authorship. Audio and leaderboard subsystems (`OrecchietTetris/audio/`, `OrecchietTetris/leaderboard/`: `auto`) were generated autonomously by the AI, based on detailed instructions given by the human developer.
- **Testing (`auto`):** the test suite (`tests/`: `auto`) was generated autonomously by the AI based on the implemented behaviour, following a test-driven development approach.
- **Documentation (`pair`):** `README.md` and inline documentation were written collaboratively. `CLAUDE.md` and `AGENTS.md` were generated autonomously by the AI (`auto`) based on codebase inspection.
- **Review (`pair`):** code review was conducted jointly — the AI flagged type errors, linting issues, and architectural inconsistencies; the human evaluated and acted on findings.
- **Deployment (`assist`):** CI/CD pipeline configuration (semantic-release, commitlint, pre-commit hooks, Poetry packaging, PyPI publish) was set up with AI guidance; the human configured and owns the pipeline.

> The `docs/` folder in the artifact repository also contains the raw implementation plans produced by Claude Code while carrying out some of the requests below, kept as-is for further evidence of the process.

## How we used GenAI
We used **Claude Code** in **plan mode** as the entry point for almost every feature: before any code was written, we described the feature in detail to the agent and had it produce a plan, following a **TDD red-green-refactor** approach — tests were planned and written first, then the implementation to make them pass, then a refactor pass.

To keep the agent grounded in the project's structure, we maintained an `AGENTS.md` file describing the architecture, conventions, and commands, and **updated it after every code change** so that its context of the project never drifted out of sync with the actual codebase.

We also adopted **Superpowers**, a set of Claude Code skills for structuring GenAI-driven development (`/using-superpowers`, `/test-driven-development`, `/subagent-driven-development`), to standardize how we ran TDD cycles and delegated sub-tasks to the agent.

The `docs/` folder in the artifact repository also contains the raw implementation plans produced by Claude Code with the Superpowers skill enabled while carrying out some of the requests below, kept as-is for further evidence of the process.

Other practical habits we settled into:
- We started a **new chat for every feature**, to keep the agent's context window clean and focused rather than accumulating unrelated history.
- When something broke, we first tried to **diagnose the problem ourselves**; only if that wasn't enough did we feed the agent our own findings alongside the raw error message, rather than just pasting a stack trace and hoping for the best.
- Our **prompt engineering was fairly minimal** in a technical sense — no elaborate templates or few-shot examples — mostly a detailed, explicit description of the task, the desired behaviour, and any technological constraints.
- AI-assisted commits are marked accordingly, with a `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>` trailer, so AI involvement is traceable directly in the git history.

## Prompting and review criteria
Distilled from the workflow above, the general, reusable rules we tried to hold ourselves to were:
- **Plan before code, always in TDD order:** every feature request went through Claude Code's plan mode first, with tests planned and written before implementation (red → green → refactor), never the other way around.
- **Keep the agent's context authoritative and current:** `AGENTS.md` had to reflect the real codebase after every change, since a stale context file produces worse plans and worse code.
- **One chat, one feature:** context was never allowed to carry over between unrelated features, to avoid the agent reasoning from stale or irrelevant history.
- **Diagnose before delegating:** errors were investigated by us first; the agent was only given the error message on its own as a last resort, with our own findings attached whenever we had them.
- **Nothing merges without passing the full gate:** every AI-generated component had to pass `poe test`, `poe flake8`, and `poe mypy` before being accepted, with no exceptions for AI-authored code.
- **Detailed, explicit prompts over clever ones:** we prioritized clearly stating the task, desired behaviour, and constraints over prompt-engineering tricks.
- **Traceability by default:** any commit with meaningful AI contribution carries a `Co-Authored-By: Claude Sonnet 4.6` trailer, so the human/AI split is visible in the repository itself, not just in this report.

## Example interactions

> **Scenario 1a — Exploration before a single test is written.**
> Before writing anything, the agent reads the existing conftest.py, prior test files, and all five leaderboard/view source files to understand established testing patterns; nothing gets written until this is done.
>
![Exploration phase before leaderboard tests](../pictures/leaderboard1.png)
>
![Todo list scoping the leaderboard test suite](../pictures/leaderboard2.png)

> **Scenario 2 — A numeric fix justified from first principles.**
> Given a one-line request ("make the top three columns slightly taller"), the agent doesn't just guess a bigger value — it sums the actual fixed label heights, padding, and spacing to derive a 154px minimum, then adjusts CARD_HEIGHTS accordingly and re-runs the affected tests before committing.
![Podium card height fix with calculated minimum](../pictures/leaderboard3.png)
