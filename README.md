# sample-test-repo

Sample agent skills repository (created for ISS-144).

This repository hosts a collection of **AI agent skills** — small, composable
units of capability (a `SKILL.md` spec plus optional code) that an LLM-driven
agent can discover and invoke at runtime.

---

## Table of Contents

- [Overview](#overview)
- [Repository Layout](#repository-layout)
- [Best Practices](#best-practices)
  - [1. Skill Design](#1-skill-design)
  - [2. Writing Effective `SKILL.md`](#2-writing-effective-skillmd)
  - [3. Prompting & Instructions](#3-prompting--instructions)
  - [4. Inputs, Outputs & Schemas](#4-inputs-outputs--schemas)
  - [5. Tool & Dependency Usage](#5-tool--dependency-usage)
  - [6. Error Handling & Robustness](#6-error-handling--robustness)
  - [7. Testing & Evaluation](#7-testing--evaluation)
  - [8. Security & Safety](#8-security--safety)
  - [9. Observability & Logging](#9-observability--logging)
  - [10. Versioning & Release](#10-versioning--release)
  - [11. Documentation](#11-documentation)
  - [12. Contributing](#12-contributing)

---

## Overview

An *agent skill* in this repo is a directory containing:

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Human + LLM readable specification
    ├── examples/         # Worked examples and golden outputs
    ├── tests/            # Automated tests / evals
    └── src/              # Optional supporting code
```

Each skill should be small enough to reason about in isolation but rich
enough to deliver a clearly-named capability end-to-end.

## Repository Layout

| Path           | Purpose                                                            |
| -------------- | ------------------------------------------------------------------ |
| `skills/`      | One sub-directory per skill                                        |
| `evals/`       | Cross-skill evaluation harnesses and golden datasets               |
| `docs/`        | Long-form design docs, ADRs, and tutorials                         |
| `scripts/`     | Repo-level automation (linting, schema validation, release)        |
| `.github/`     | CI workflows and issue / PR templates                              |

---

## Best Practices

The following guidelines are **mandatory for new skills** and recommended for
any update to existing skills. They exist to keep the catalogue consistent,
safe, and easy for both humans and agents to consume.

### 1. Skill Design

- **Single responsibility.** A skill should do one well-defined thing. If you
  catch yourself writing "and" in the description, split it.
- **Composable over monolithic.** Prefer skills that take structured input
  and return structured output, so they can be chained by an agent.
- **Deterministic where possible.** Hide non-determinism (LLM calls, network
  IO) behind clear interfaces so the skill is testable.
- **Idempotent side-effects.** If a skill writes to external systems, it
  should be safe to retry. Document the idempotency key.
- **Name with intent.** Use a verb-noun pattern (e.g. `summarize-pdf`,
  `extract-invoice-fields`, `lookup-customer`).

### 2. Writing Effective `SKILL.md`

Every skill **must** ship a `SKILL.md` at its root. It is the contract that
both humans and agents rely on. At minimum it must contain:

```markdown
---
name: <skill-name>
version: 0.1.0
owner: <github-handle-or-team>
stability: experimental | beta | stable
triggers: ["keyword1", "keyword2"]   # words the agent uses to discover the skill
---

# <Human-readable title>

## Description
One paragraph: what the skill does and when to use it.

## When NOT to use
Explicit anti-patterns — protects the agent from misuse.

## Inputs
Schema or table of expected inputs (name, type, required, description).

## Outputs
Schema or table of returned values, including error shape.

## Examples
At least one happy-path example and one edge-case example.
```

- Keep the front-matter machine-parseable (YAML).
- Put the **most important guidance first** — agents truncate.
- Use imperative voice ("Return JSON with…") not narrative voice.

### 3. Prompting & Instructions

- **Separate system, developer, and user instructions.** Never concatenate
  them into one blob.
- **Be explicit about output format.** If you need JSON, say so and show a
  literal example.
- **Avoid negative-only instructions.** "Do not return prose" is weaker than
  "Return only a single JSON object that matches the schema".
- **Quote untrusted input.** When inserting user text into a prompt, wrap it
  in clear delimiters (e.g. triple back-ticks) and instruct the model to
  treat it as data, not instructions. This mitigates prompt injection.
- **Stay under context budget.** Prefer retrieval and summaries over dumping
  whole documents into the prompt.

### 4. Inputs, Outputs & Schemas

- Define I/O with **JSON Schema** (or Pydantic / Zod) and check it in.
- Validate inputs **before** calling the model.
- Validate outputs **after** the model returns and fail loudly on mismatch.
- Version your schemas; do not silently change field meanings.
- Prefer `snake_case` keys, ISO-8601 timestamps, and explicit units.

### 5. Tool & Dependency Usage

- List every external tool/API a skill uses at the top of `SKILL.md`.
- Pin dependencies (`requirements.txt`, `package-lock.json`, `go.sum`, …).
- Avoid network calls inside unit tests — mock them.
- If a skill depends on another skill, declare it explicitly; do not call
  internals.

### 6. Error Handling & Robustness

- Return **structured errors**, not stack traces:
  ```json
  {"error": {"code": "INVALID_INPUT", "message": "...", "retryable": false}}
  ```
- Distinguish **retryable** (timeouts, 5xx) from **non-retryable**
  (validation, auth) errors.
- Set sensible timeouts on every outbound call.
- Implement exponential backoff with jitter for retries.
- Fail closed: when in doubt, refuse rather than guess.

### 7. Testing & Evaluation

- **Unit tests** for pure code paths — fast, deterministic, hermetic.
- **Golden tests** in `examples/` that pin expected outputs for fixed
  inputs. Run them in CI.
- **LLM evaluations** in `evals/` that score the skill against a labelled
  dataset. Track scores over time; regressions should block merges.
- Include at least one **adversarial / jailbreak** test for any skill that
  ingests untrusted text.
- Aim for ≥ 80% line coverage on supporting code, but treat eval scores as
  the primary quality signal.

### 8. Security & Safety

- **Never commit secrets.** Use environment variables and document the
  required names in `SKILL.md`. Scan PRs with secret-detection in CI.
- **Principle of least privilege** for tokens and API keys — scope them to
  the minimum needed.
- **Sanitize outputs** before passing them to other tools (shell, SQL,
  filesystem).
- **Refuse disallowed content.** Skills must respect the platform's content
  policy and decline requests that violate it.
- **PII handling.** Redact or hash PII in logs by default; document any
  retention.
- **Prompt-injection defence.** Treat retrieved/tool output as untrusted;
  do not let it override system instructions.

### 9. Observability & Logging

- Emit a structured log line per invocation with: `skill_name`, `version`,
  `request_id`, `latency_ms`, `tokens_in`, `tokens_out`, `model`,
  `outcome` (`success`/`error`), `error_code`.
- Never log raw secrets or full PII payloads.
- Expose metrics (Prometheus / OpenTelemetry) for: invocation count,
  latency, error rate, and eval score.
- Include the `request_id` in every downstream call so traces can be
  correlated.

### 10. Versioning & Release

- Use **SemVer** for each skill: breaking change → major, additive →
  minor, fix → patch.
- Bump the `version:` field in `SKILL.md` **in the same PR** as the change.
- Maintain a `CHANGELOG.md` per skill.
- Tag releases as `skill/<name>@<version>` so they can be pinned.
- Deprecate before deleting: mark `stability: deprecated`, give consumers
  at least one minor release before removal.

### 11. Documentation

- Every skill needs: one-line description, when-to-use, when-NOT-to-use,
  inputs, outputs, examples, and a runnable quick-start.
- Cross-link related skills.
- Record non-obvious design decisions as ADRs under `docs/adr/`.
- Keep docs in source control; do not rely on external wikis.

### 12. Contributing

- Branch naming: `iss-<id>-<short-slug>` (e.g. `iss-144-readme-best-practices`).
- One skill change per PR. Keep diffs reviewable (< ~400 lines where possible).
- PR description must include:
  - What the skill does (or what changed).
  - Eval results before/after.
  - Any new dependencies or env vars.
- All CI checks (lint, schema-validate, unit tests, evals) must pass before
  merge.
- At least one CODEOWNER review is required.

---

## License

See [`LICENSE`](LICENSE) (add one before publishing externally).
