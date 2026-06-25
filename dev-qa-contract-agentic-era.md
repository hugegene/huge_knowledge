# The DEV–QA Contract: Keeping QA a Gatekeeper in the Agentic Era

How to let QA *fix* bugs with coding agents (instead of throwing them back to DEV) without QA losing the independence that made it a gatekeeper in the first place. The mechanism is a **separation of powers** enforced by file ownership inside each feature directory of a repo.

This note builds on [Organizing a GitHub Repo for Agentic Coding](./agentic-coding-repo-organization.md), which establishes the `specs/<feature>/spec.md` + `test-cases.md` layout. Here we turn that layout into a *contract* between two agents.

> **Note:** This is my own personal thinking — a working idea I'm developing. Treat it as a draft to poke holes in.

## The shift: what agentic coding does to QA

Traditionally QA is the **gatekeeper of the product**: a structurally independent team that decides whether work is good enough to ship. Its power comes from *separation* — different people, different incentives, checking what DEV built. When QA finds a defect, it writes a ticket and pushes it back across the wall to DEV.

Agentic coding collapses that wall. A QA engineer with Claude Code can now:

- **Generate edge cases** at scale (boundary values, malformed input, race conditions, locale/timezone, adversarial cases) instead of hand-enumerating a handful.
- **Reproduce and fix** many of those defects directly, without a round-trip to DEV.

This is a huge throughput win — the bug-fix loop shrinks from *days (file ticket → triage → DEV picks up → review)* to *minutes*. But it creates a problem.

## The core tension: the "marking your own homework" bias

If QA writes the fix, QA is now motivated to declare the fix correct. The same actor that produced the work also signs off on it. This is the classic **self-verification bias** — confirmation bias plus incentive alignment — and it is exactly what the old DEV/QA wall existed to prevent.

> If QA takes up DEV work, is there an inherent bias to *pass whatever they just developed*?

Yes — **if independence lives in *people***. The instinctive fix is to keep two separate humans, but that throws away the speed that made agentic QA attractive in the first place. So the real question is:

**Can we preserve QA's independence structurally, without preserving the slow human hand-off?**

## The key insight: move independence from *people* to *artifacts*

QA's trustworthiness never actually came from it being a *different person*. It came from QA judging against a **standard that the implementer could not move**. The person was just the carrier of that standard.

So put the independence in the **files and their ownership rules**, not in the org chart. Then a single QA agent can both fix a bug *and* be held to an external standard, because the standard is frozen in a file it is forbidden to edit.

This is **separation of powers** applied to a repo:

| Branch of power | Owner | Artifact | Analogy |
|---|---|---|---|
| **Legislative** — defines intent | DEV | `spec.md` | What the law *says* |
| **Judicial** — defines & checks conformance | QA | `test-cases.md` + tests | What counts as compliance |
| **Executive** — produces the implementation | DEV *and* QA | source code | Carrying out the law |

QA gains executive power (it can now write code) **but never legislative power** (it can never redefine intent). That single asymmetry is what neutralizes the bias.

## The contract: per-feature file ownership

Every feature directory carries the contract as files. Ownership is strict and machine-enforced.

```
specs/
└── 014-checkout-discounts/
    ├── spec.md          # OWNED BY DEV  — QA may read, never write
    ├── test-cases.md    # OWNED BY QA   — DEV may read, never write
    └── bugs/
        └── 2026-06-25-negative-discount.md   # QA-filed defect reports
src/
└── checkout/            # code — either side may edit, but QA cannot step beyond to spec.md
tests/
└── checkout/            # code — either side may edit
```

### What each agent may do

**DEV agent** — generates code from `spec.md` **and** `test-cases.md`. It sees QA's acceptance criteria as a target (TDD-style) but cannot weaken them, because `test-cases.md` is read-only to DEV.

**QA agent** — three distinct jobs, all bounded by `spec.md`:
1. **Author `test-cases.md`** as an *independent* interpretation of `spec.md` into verifiable scenarios, including generated edge cases.
2. **Generate test scripts** in `tests/` that encode those cases as executable checks.
3. **Report and fix conformance bugs** — when code violates `spec.md`/`test-cases.md`, QA may write the minimal fix *and* a regression test. It **cannot overstep the logic set in `spec.md`.**

## The one clean rule that holds the whole thing together

> **If a fix would require changing `spec.md`, QA may not make it. The task is handed back to DEV.**

This single boundary is what keeps QA honest. It forces every defect into one of two buckets:

| Bug type | What it means | Who fixes it |
|---|---|---|
| **Conformance bug** | Code disagrees with a spec that is itself correct (off-by-one, unhandled null the spec implies, wrong rounding). | **QA fixes directly.** Spec untouched → no bias risk. |
| **Spec bug / gap** | Spec is wrong, ambiguous, contradictory, or silent on the case. Fixing it means *deciding new intent*. | **Hand back to DEV.** Only the intent-owner may legislate. |

QA can never quietly redefine what "correct" means in order to make its own fix pass — because changing the definition of correct *is* editing `spec.md`, which is the one thing it cannot do. The escape valve is explicit and auditable.

## Why this actually neutralizes the bias

Walk the failure the user worried about — QA writes a sloppy fix and wants to wave it through:

1. **It can't lower the bar.** Passing is defined by `test-cases.md` and `spec.md`, both frozen and both off-limits to QA's edit rights. To make a bad fix "pass," QA would have to weaken the tests or bend the spec — both blocked.
2. **The bar was set before the fix.** `test-cases.md` is written from `spec.md`, *before and independent of* the implementation. The judge is not downstream of the defendant.
3. **Spec-redefining fixes can't hide.** Any fix that needs new intent trips the escalation rule and surfaces to DEV.
4. **Everything is tamper-evident.** Ownership is enforced in CI (below), so a violation is a failed check, not a matter of trust.

Net: QA keeps the *speed* of fixing things itself, but its sign-off is still measured against a standard it did not author and cannot move. The gate stays independent even though the gatekeeper now also carries tools.

### A bonus: writing tests *is* a spec review

Because QA authors `test-cases.md` by interpreting `spec.md` independently, any clause QA **can't turn into a concrete test** is, by definition, ambiguous or underspecified. That ambiguity becomes a finding routed back to DEV *before any code is written*. The contract surfaces bad specs early, for free.

## Lifecycle of a feature under the contract

```
DEV writes spec.md  ──►  QA writes test-cases.md (independent reading of spec)
        │                          │
        │              ┌───────────┴── ambiguity? → finding back to DEV
        ▼                          ▼
DEV agent generates code    QA generates test scripts (tests/)
        └───────────┬──────────────┘
                    ▼
            run the suite
                    │
        ┌───────────┴────────────┐
     failing                   passing → merge gate
        │
   QA classifies the bug
        │
 ┌──────┴────────────────┐
 conformance bug      spec bug / gap
 │                       │
 QA fixes + adds       hand back to DEV
 regression test       (only DEV edits spec.md)
 │                       │
 re-run suite ◄──────────┘ (after DEV revises spec → QA revises test-cases)
```

## Enforcing the contract (defense in depth)

The ownership rules must be *mechanical*, not honor-system. The same rule — e.g. *"QA may not touch `spec.md`"* — is checked at several points, from the softest and fastest (the agent's own prompt) to the hardest and slowest (the CI guard). A cooperative agent is stopped instantly; a misbehaving one is still caught before merge.

One trick lets every layer agree on *who is acting*: **derive the role from the branch prefix** — `dev/*` vs `qa/*`. The local hook reads the current branch, CI reads the PR's head ref, and both apply the same rule with no extra role config to keep in sync.

### Layer 1 — Agent prompts (soft, instant)

Encode the contract in each role's system prompt — an `AGENTS.md`/`CLAUDE.md` block or a Claude Code subagent. Prompts are *soft*: they steer a cooperative agent but don't stop a misbehaving one, so this is the UX layer, not the security layer.

`.claude/agents/qa.md`:

```markdown
---
name: qa
description: QA agent — authors test-cases, writes tests, fixes conformance bugs.
tools: Read, Edit, Write, Bash, Grep
---
You enforce the DEV–QA contract. Hard rules:

1. `specs/**/spec.md` is DEV-owned. You may READ it, NEVER edit it.
2. You own `specs/**/test-cases.md` and `tests/**`. Derive test-cases from
   spec.md, NEVER from the implementation.
3. You may edit `src/**` ONLY to make code conform to spec.md + test-cases.md.
4. ESCALATION: if a fix would require changing spec.md, STOP. Do not edit code.
   Run: gh issue create --label spec-change --title "<feature>: spec change needed"
   and describe the gap. Hand back to DEV.
5. Never loosen a test to make a fix pass. A test may only become stricter, or
   change because spec.md changed (DEV's job, not yours).
```

The DEV agent's block mirrors it: it owns `spec.md`, and may read but never edit `test-cases.md`/`tests/`. Run QA sessions on `qa/<feature>` branches, DEV sessions on `dev/<feature>`.

### Layer 2 — Local hooks (hard for the agent, before any commit)

A Claude Code **`PreToolUse`** hook fires before any Edit/Write and can *deny* it, with the reason fed straight back to the agent — so a forbidden edit is blocked the instant it's attempted, no commit or CI round-trip needed.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/guard-contract.sh" }
        ]
      }
    ]
  }
}
```

`.claude/hooks/guard-contract.sh` (exit code 2 blocks the tool call; stderr is shown to the agent):

```bash
#!/usr/bin/env bash
# stdin: JSON with .tool_input.file_path
path=$(jq -r '.tool_input.file_path // empty')
[ -z "$path" ] && exit 0
rel=${path#"$CLAUDE_PROJECT_DIR"/}
branch=$(git -C "$CLAUDE_PROJECT_DIR" branch --show-current)

case "$branch" in qa/*) role=qa ;; dev/*) role=dev ;; *) role=unknown ;; esac
deny() { echo "BLOCKED by DEV–QA contract: $1" >&2; exit 2; }

case "$rel" in
  specs/*/spec.md)
    [ "$role" = qa ] && deny "spec.md is DEV-owned. If this fix needs a spec change, run: gh issue create --label spec-change" ;;
  specs/*/test-cases.md|tests/*)
    [ "$role" = dev ] && deny "$rel is QA-owned and immutable to DEV." ;;
esac
exit 0
```

Wire the same script as a [`pre-commit`](https://pre-commit.com) git hook so manual or CLI edits — not just the agent — are caught before they enter a commit.

### Layer 3 — CI guard (catches role↔file mismatch on the PR)

This **fails the build** if the wrong role touches a file, turning violations into red checks rather than judgment calls. Keep it dependency-free with native `git`.

`.github/workflows/contract-guard.yml`:

```yaml
name: contract-guard
on: { pull_request: { branches: [main] } }
jobs:
  ownership:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4        # pin to a SHA in production
        with: { fetch-depth: 0 }
      - name: Enforce ownership
        run: |
          case "${{ github.head_ref }}" in
            qa/*)  role=qa  ;; dev/*) role=dev ;; *) role=unknown ;;
          esac
          git fetch origin "${{ github.base_ref }}" --depth=1
          v=0
          while IFS= read -r f; do
            case "$f" in
              specs/*/spec.md)
                [ "$role" = qa ] && { echo "::error file=$f::QA may not edit spec.md"; v=1; } ;;
              specs/*/test-cases.md|tests/*)
                [ "$role" = dev ] && { echo "::error file=$f::DEV may not edit $f"; v=1; } ;;
            esac
          done < <(git diff --name-only "origin/${{ github.base_ref }}...HEAD")
          [ "$role" = unknown ] && { echo "::error::branch must be dev/* or qa/*"; v=1; }
          exit $v
```

Add this job to branch protection's required status checks so a violating PR cannot merge.

### Escalation path — handing back to DEV

Make the "this needs a spec change" route a single command so the agent takes it instead of forcing the edit. An issue form at `.github/ISSUE_TEMPLATE/spec-change.yml` (label `spec-change`) is the structured hand-back; the QA prompt (Layer 1) and the hook (Layer 2) both point to `gh issue create --label spec-change …`. DEV picks up the issue, edits `spec.md`, and QA re-derives `test-cases.md` and `tests/` against the revised contract.

```
QA fix blocked (needs spec change)
        │  gh issue create --label spec-change
        ▼
spec-change issue  ──►  DEV edits spec.md and codes  ──►  QA re-derives test-cases.md + tests/
                                                          │
                                                          ▼
                                                  fix re-attempted under new contract
```

### Layer summary

| Layer | Mechanism | Stops |
|---|---|---|
| 1 — Prompt | `.claude/agents/*.md` subagents | cooperative agent (soft) |
| 2 — Hook | Claude Code `PreToolUse` + `pre-commit` | a bad edit, instantly, locally |
| 3 — CI guard | GitHub Actions | role↔file violation on the PR |
