# Organizing a GitHub Repo for Agentic Coding

A summary of how to structure documentation, specifications, and test cases in a GitHub repository for spec-driven development with coding agents (Claude Code, Copilot, etc.), and which GitHub features to lean on.

## Why structure matters for agents

Coding agents read your repository as context before writing code. A predictable structure means the agent can reliably find the *what* (specs), the *why* (research and decisions), and the *definition of done* (test cases) without you re-explaining them in every prompt. Clear separation also prevents the agent from confusing rejected ideas in research notes with actual requirements.

## Recommended folder structure

```
repo/
├── docs/
│   ├── research/            # Background investigations, option comparisons
│   │   └── 2026-06-auth-options.md
│   ├── adr/                 # Architecture Decision Records (numbered, immutable)
│   │   └── 0003-use-postgres.md
│   └── architecture.md      # Current high-level system overview
├── specs/
│   ├── 001-user-login/
│   │   ├── spec.md          # Feature specification (what & why)
│   │   ├── plan.md          # Implementation plan (how)
│   │   └── test-cases.md    # Acceptance criteria / test scenarios
│   └── 002-billing/
├── src/                     # Application code
├── tests/                   # Executable tests, mirroring src/
├── CLAUDE.md (or AGENTS.md) # Instructions for the coding agent
└── README.md
```

### Design principles

1. **Separate research from specs.** Research documents are exploratory and may contain rejected options; specs are the contract the agent implements against. Mixing them confuses agents.
2. **One folder per feature under `specs/`.** Spec, plan, and test cases live together so a single instruction ("implement specs/001-user-login") gives the agent everything it needs.
3. **ADRs capture the "why."** Short, numbered, immutable records of architectural decisions stop agents from "helpfully" undoing past choices.
4. **CLAUDE.md / AGENTS.md is the agent's onboarding doc.** Conventions, where specs live, how to run tests, coding style, and what not to touch. Most agentic tools read this file automatically.
5. **test-cases.md describes scenarios; `tests/` holds executable code.** Writing human-readable acceptance criteria *before* implementation gives the agent a target and lets you verify the tests are meaningful rather than trivially passing.

For a formalized version of this workflow, see GitHub's **spec-kit** project (constitution → spec → plan → tasks).

## GitHub Issues as agent work units

Issues are the natural work unit for agent-driven development: each issue becomes a self-contained instruction the agent can read, act on, and close. Agents read issues directly via the `gh` CLI (`gh issue view 42`) or the GitHub MCP server, so "work issue #42" becomes a complete instruction.

### Structured issue templates

A bug report or feature request stored as a GitHub **issue form** at `.github/ISSUE_TEMPLATE/*.yml` is, in effect, a prompt template for the agent. GitHub automatically detects any `.yml` file in that folder and renders it as a structured web form on the *New Issue* screen, with required fields (expected behavior, steps to reproduce, environment, etc.) that block submission until filled in. On submit, the answers collapse into a normal issue body with consistent markdown headings, and any labels declared in the template (e.g. `bug`, `triage`) are applied automatically.

The payoff: because every issue arrives in the same shape, the agent can reliably parse out the relevant sections and never receives a report missing critical context. Add a `config.yml` with `blank_issues_enabled: false` in the same folder to force everyone through the templates.

## Workflows

The three workflows below all converge on the same gates — a branch, a pull request, Continuous Integration (CI), and human review — but differ in what triggers them and what the agent produces.

### Bug-fix workflow

Triggered by a defect report. The goal is a verified fix plus a regression test so the bug can never silently return.

Bug reported via issue template → triaged and labeled `agent-ready` (report complete enough to act on without follow-up) → agent reads the issue and reproduces the bug locally → agent writes a **failing** test that captures the bug → agent implements the minimal fix that makes the test pass → full suite run to catch regressions → PR opened with `Closes #NN` → CI green → human reviews diff → merge auto-closes the issue.

The fix, its regression test, and the originating report stay permanently linked, giving a traceable history of every defect.

### Design–develop workflow

Triggered by a new feature request issue. The goal is to settle the *what* and *why* before any code is written, so the agent builds against an agreed contract rather than acting on a one-line wish.

Feature request filed via issue template → human-led refinement (clarify intent, research options in `docs/research/`, record consequential choices as an ADR in `docs/adr/`) → feature specification written in `specs/NNN-feature/spec.md` (the contract) and an implementation `plan.md` (the approach) → agent implements against the spec on a feature branch → PR opened referencing the request issue and spec → CI green → human review → merge.

Keeping research, decisions, and specs separate but co-located means the agent reads the current contract without tripping over rejected alternatives.

**Linking and closing the request:** the spec file and the request issue point at each other — the issue references the spec path (`specs/NNN-feature/spec.md`) and the spec records its tracking issue (`#NN`), which GitHub auto-links. The durable connection is made through the pull request: it contains the spec/code in its diff and uses a closing keyword (`Closes #NN`), so merging the PR both joins the spec to the issue via the merge history and automatically closes the request. (GitHub has no native "issue ↔ file" link, so the PR is what makes the link real rather than just rendered markdown.)

### Testing workflow

Test cases are authored as part of specification, *before* implementation, so they define done rather than rationalize whatever was built.

Human-readable acceptance criteria and edge cases written in `specs/NNN-feature/test-cases.md` → agent translates these scenarios into executable tests under `tests/`, mirroring `src/` → tests run locally during development and again in CI on every PR → no branch merges unless the suite is green. Bug fixes feed back into this suite as permanent regression tests.

Writing cases in human language first gives the agent a clear target and lets you confirm the tests are meaningful rather than trivially passing.

## Do you still need Jira and Confluence?

For agentic, spec-driven development, often **no** — and there are real arguments for consolidating in GitHub. But it depends on team size and organizational context.

### The case for staying GitHub-only

- **Agents work best where the code lives.** Specs, issues, and decisions in the repo are directly readable by the agent with no extra integrations, authentication, or sync. Context in Confluence or Jira requires MCP connectors or manual copy-paste, adding friction and staleness risk.
- **Docs-as-code benefits.** Markdown in the repo is version-controlled, reviewed via PRs, branchable, and diffable. A spec change can ship in the same PR as the code change, keeping them in lockstep. Confluence pages drift out of date because nothing forces them to change with the code.
- **One source of truth.** Splitting "the spec is in Confluence, the ticket is in Jira, the code is in GitHub" creates three places that disagree. Agents (and humans) suffer from this fragmentation.
- **Lower cost and tooling overhead.** Issues, Projects, Discussions, and Actions are included with GitHub; Jira and Confluence are separate licenses and admin burdens.
- **GitHub's tooling has matured.** Projects now supports custom fields, iterations/sprints, roadmap views, and automation — covering most of what small-to-mid teams used Jira for.
