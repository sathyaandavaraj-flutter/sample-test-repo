# sample-test-repo

A reference repository for authoring, packaging, and shipping **AI agent skills**.

A *skill* in this repo is a self-contained capability that an agent can load
on demand to perform a focused task — for example, generating a `.xlsx`
spreadsheet, drafting a sales email, or summarising a PDF. Each skill is a
directory under `skills/` that ships:

- a `SKILL.md` describing what the skill does and when to use it,
- the code/prompts/templates that implement it,
- tests and evaluation fixtures that prove it still works.

This README is the canonical place for our authoring conventions.

## Table of contents

- [Repository layout](#repository-layout)
- [Best practices](#best-practices)
  1. [Skill design](#1-skill-design)
  2. [Writing `SKILL.md`](#2-writing-skillmd)
  3. [Inputs, outputs, and schemas](#3-inputs-outputs-and-schemas)
  4. [Tool and model usage](#4-tool-and-model-usage)
  5. [Error handling and robustness](#5-error-handling-and-robustness)
  6. [Security and privacy](#6-security-and-privacy)
  7. [Observability](#7-observability)
  8. [Testing and evaluation](#8-testing-and-evaluation)
  9. [Versioning and compatibility](#9-versioning-and-compatibility)
  10. [Documentation and review](#10-documentation-and-review)
- [Contributing](#contributing)

## Repository layout

```
.
├── skills/            # One directory per skill (xlsx/, docx/, pdf/, ...)
│   └── <name>/
│       ├── SKILL.md   # Trigger words, description, examples
│       ├── src/       # Implementation
│       ├── prompts/   # Prompt templates, if any
│       └── tests/     # Unit tests and golden fixtures
├── evals/             # Cross-skill evaluation suites
├── scripts/           # Dev tooling (lint, package, release)
└── .github/           # CI workflows and PR/issue templates
```

## Best practices

> The guidance below is intentionally generic. Tailor any specifics
> (e.g. exact version policy, review group) to your team's conventions.

### 1. Skill design

- **Single responsibility.** Each skill should do one thing well. If a
  description needs the word "and", split it.
- **Composable, not monolithic.** Prefer small skills the agent can chain
  over one mega-skill that hides logic behind flags.
- **Deterministic where possible.** Pure transformations (parsing,
  formatting, validation) should not call an LLM. Reserve model calls for
  steps that genuinely require reasoning.
- **Explicit side effects.** A skill that writes files, calls APIs, or
  costs money must say so in its `SKILL.md` under a **Side effects**
  heading.
- **Stateless by default.** Skills should not rely on hidden global state.
  Pass everything you need through inputs.

### 2. Writing `SKILL.md`

Every skill **must** have a `SKILL.md` at its root. The agent reads this
file to decide when and how to use the skill, so treat it as a contract,
not as marketing copy.

Required sections:

- **Description** — one paragraph, plain English. Lead with the verb
  ("Generates...", "Extracts...", "Sends...").
- **Triggers** — concrete words/phrases or task patterns that should cause
  the agent to load this skill. Be specific: `"xlsx"`, `"spreadsheet"`,
  `"pivot table"` is better than `"data"`.
- **Inputs** — name, type, whether required, and a one-line meaning.
- **Outputs** — what the caller gets back, including the shape on success
  and on failure.
- **Examples** — at least one runnable example, ideally two: a typical
  case and an edge case.
- **Failure modes** — what the skill explicitly will *not* do, plus the
  errors it raises and when.

Keep the file short (target < 300 lines). If you need more, link out to
docs in the same directory.

### 3. Inputs, outputs, and schemas

- **Validate inputs at the boundary.** Use a schema (JSON Schema,
  Pydantic, zod — pick one per language and stick to it). Reject early
  with a clear message.
- **Prefer structured outputs.** Return typed objects, not free-form
  prose, when a caller will parse the result.
- **Document units.** `timeout_ms`, `price_usd`, `weight_kg` —
  always suffix the unit so callers cannot guess wrong.
- **Default conservatively.** Defaults should be the safest, cheapest
  behaviour. Power users can opt in to the expensive option.
- **Signal truncation.** If you cap output length, return a flag
  (`truncated: true`) instead of silently cutting data.

### 4. Tool and model usage

- **Idempotency.** Re-running a skill with the same inputs should produce
  the same outputs (or the same well-defined failure). Use idempotency
  keys when you call external APIs that create resources.
- **Token economy.** Don't paste an entire file into a prompt when a
  summary will do. Prefer references (paths, IDs) over inlined blobs.
- **Pick the right model.** Cheaper/faster models for classification and
  extraction; larger models only for genuine reasoning or generation.
  Justify model choices in `SKILL.md`.
- **Respect rate limits.** Bound concurrency, back off on `429`, and
  surface quota errors to the caller rather than retrying forever.
- **Tool-call discipline.** Make independent calls in parallel; chain
  only when one call's output is the next call's input. Never call a
  destructive tool speculatively.

### 5. Error handling and robustness

- **Typed errors.** Distinguish user errors (bad input), transient errors
  (retryable), and system errors (bug, page someone). Don't return all
  three as a generic `Exception`.
- **No silent failures.** If something is wrong, the caller must hear
  about it. Empty results without context are a bug.
- **Timeouts everywhere.** Every external call gets a timeout. There is
  no acceptable default of "infinity".
- **Retries with backoff.** Retry only what is genuinely retryable
  (network blips, `5xx`, `429`). Never retry a `400`.
- **Fail loudly in dev, gracefully in prod.** Use feature flags or
  environment checks to decide whether to raise or to degrade.

### 6. Security and privacy

- **No secrets in the repo.** Ever. Use the org secret manager and
  reference secrets by name. Pre-commit hooks should run a secret
  scanner.
- **Least privilege.** A skill that only needs to read should not be
  handed write credentials. Scope tokens per skill where possible.
- **Treat user input as hostile.** Prompt injection is real. Strip or
  fence untrusted text before concatenating it into a prompt, and never
  let model output decide which tool to call without policy checks.
- **Redact PII in logs.** Names, emails, phone numbers, payment data —
  redact at the logger, not at the call site, so it is impossible to
  forget.
- **Pin dependencies.** Lockfiles checked in; review transitive updates
  the same way you review code.

### 7. Observability

- **Structured logs.** Emit JSON with `skill`, `version`, `request_id`,
  `latency_ms`, `outcome`. Free-text logs are not greppable at scale.
- **Trace across tool calls.** Propagate a correlation id through every
  external call so a single user request is one trace, not twenty.
- **Metrics that matter.** Track latency (p50/p95/p99), error rate by
  type, token usage, and cost per invocation. Alert on the deltas, not
  the absolutes.
- **Sample, don't store everything.** Keep enough to debug; don't keep
  so much that you have a privacy incident waiting to happen.

### 8. Testing and evaluation

- **Unit tests** for deterministic logic (parsing, formatting,
  validation). These must run offline with no model calls.
- **Golden files** for outputs that should be byte-stable
  (e.g. generated spreadsheets, rendered templates). Diff failures are
  signal, not noise — investigate before regenerating.
- **Eval suites** for model-backed steps. Maintain a small, curated set
  of input/expected pairs and run them in CI; track pass rate over time.
- **Adversarial tests.** Include prompt-injection attempts, malformed
  inputs, oversized inputs, and empty inputs. Skills should refuse, not
  crash.
- **Regression tests for every bug.** A bug that reaches `main` gets a
  test before it gets a fix.

### 9. Versioning and compatibility

- **Per-skill SemVer.** Each skill has its own version. Breaking changes
  to inputs, outputs, or behaviour bump the major version.
- **Deprecation, not deletion.** Mark a skill `deprecated: true` in
  `SKILL.md` for at least one release before removing it, and point
  users at the replacement.
- **Contract-first changes.** Update `SKILL.md` and the schema in the
  same PR as the code change. README and code must never drift.

### 10. Documentation and review

- **`SKILL.md` is mandatory** to merge. CI should fail if it is missing
  or empty.
- **Two-person review.** Any skill that calls a paid model, touches
  user data, or makes outbound network calls requires two reviewers,
  one of whom is on the platform team.
- **Runnable examples.** Every example in `SKILL.md` should be
  copy-pasteable into the test runner. Stale examples are worse than no
  examples.
- **Changelog entries.** Each merged PR appends a one-line entry to the
  skill's `CHANGELOG.md` under `## Unreleased`.

## Contributing

1. Branch from `main`: `git checkout -b <your-handle>/<short-topic>`.
2. Add or update a skill under `skills/<name>/`.
3. Update `SKILL.md`, tests, and `CHANGELOG.md` together.
4. Open a PR using the template in `.github/`. Link the DevRev issue.
5. Make sure CI is green before requesting review.

See `.github/PULL_REQUEST_TEMPLATE.md` for the full checklist.
