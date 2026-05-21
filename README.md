# sample-test-repo

A sample repository of **agent skills** — self-contained, reusable capabilities that an AI agent can load on demand to perform a specialized task (parsing a PDF, building a slide deck, querying an API, etc.).

This README documents the conventions and best practices we follow when authoring skills in this repo. Following these guidelines keeps skills small, discoverable, safe to compose, and easy for agents to invoke correctly.

---

## Table of contents

- [Repository layout](#repository-layout)
- [Best practices](#best-practices)
  - [1. Scope each skill narrowly](#1-scope-each-skill-narrowly)
  - [2. Use clear, deterministic naming](#2-use-clear-deterministic-naming)
  - [3. Write a complete `SKILL.md`](#3-write-a-complete-skillmd)
  - [4. Choose strong trigger words](#4-choose-strong-trigger-words)
  - [5. Define explicit input / output contracts](#5-define-explicit-input--output-contracts)
  - [6. Prefer dedicated tools over shell glue](#6-prefer-dedicated-tools-over-shell-glue)
  - [7. Make skills idempotent and side-effect aware](#7-make-skills-idempotent-and-side-effect-aware)
  - [8. Handle errors loudly, fail safely](#8-handle-errors-loudly-fail-safely)
  - [9. Be deliberate about secrets and PII](#9-be-deliberate-about-secrets-and-pii)
  - [10. Keep skills token-efficient](#10-keep-skills-token-efficient)
  - [11. Ship tests with every skill](#11-ship-tests-with-every-skill)
  - [12. Version and document breaking changes](#12-version-and-document-breaking-changes)
- [Contributing a new skill](#contributing-a-new-skill)

---

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md          # Description, triggers, usage contract (required)
    skill.py          # Implementation entry point
    examples/         # Minimal, runnable examples
    tests/            # Unit + integration tests
```

Every directory under `skills/` is exactly one skill. No nested skills, no shared mutable state between siblings.

---

## Best practices

### 1. Scope each skill narrowly

A skill should do **one thing** well. If the description needs the word "and" more than once, split it.

- ✅ `pdf-extract-text` — extracts text from a PDF.
- ✅ `pdf-fill-form` — fills an interactive PDF form.
- ❌ `pdf-toolkit` — extracts, fills, merges, redacts, OCRs.

Narrow skills are easier for the agent to select, easier to test, and safer to compose.

### 2. Use clear, deterministic naming

- Use lowercase, hyphen-separated names: `xlsx-build-report`, not `xlsxBuildReport`.
- Lead with the domain (`pdf-`, `xlsx-`, `pptx-`, `github-`, `devrev-`).
- The name should read as a verb phrase describing the outcome.
- Never rename a skill in place — deprecate the old name (see [§12](#12-version-and-document-breaking-changes)).

### 3. Write a complete `SKILL.md`

Every skill **must** ship a `SKILL.md` with these sections:

| Section | Purpose |
|---|---|
| `name` | The canonical skill name. |
| `description` | One- or two-sentence summary of what it does. |
| `triggers` | The vocabulary that should cause the agent to load this skill. |
| `inputs` | Required and optional parameters, with types. |
| `outputs` | What the caller gets back (and in what shape). |
| `examples` | At least one end-to-end example. |
| `non-goals` | Things this skill explicitly does *not* do. |

`SKILL.md` is the contract the agent reads — treat it as production documentation, not a scratch note.

### 4. Choose strong trigger words

Triggers are the words that route a user request to your skill. Good triggers are:

- **Specific** — `.xlsx`, `pivot table`, `vlookup` beats generic terms like `data`.
- **Plural-aware** — include both `slide` and `slides`, `chart` and `charts`.
- **Symmetric** — include file extensions *and* the product name (`xlsx`, `Excel`, `spreadsheet`).
- **Non-overlapping** — if two skills compete for the same trigger, one of them is mis-scoped.

When in doubt, add a short "MANDATORY TRIGGERS" list at the end of the description so the router never misses them.

### 5. Define explicit input / output contracts

- Document every argument: name, type, whether it's required, default value, allowed range.
- Validate inputs at the entry point and return a typed error — never crash with a stack trace into the agent context.
- Return structured outputs (dicts / dataclasses), not free-form strings. The agent should not have to parse prose to find a result.

### 6. Prefer dedicated tools over shell glue

If the runtime exposes purpose-built tools (file read/write/edit, HTTP fetch, search), use them instead of shelling out to `cat`, `sed`, `awk`, or `curl`. Dedicated tools:

- show up cleanly in the agent's trace,
- carry the right permission scopes, and
- are easier for reviewers to audit.

Reserve `bash` for things that genuinely need a shell (builds, package managers, git, process management).

### 7. Make skills idempotent and side-effect aware

- Running the same skill twice with the same inputs should produce the same result.
- Any skill that writes to a filesystem, calls a paid API, or mutates external state must say so loudly in `SKILL.md` under a **"Side effects"** heading.
- Provide a `--dry-run` mode for anything destructive.

### 8. Handle errors loudly, fail safely

- Catch expected errors (network timeouts, missing files, validation failures) and return a structured error object.
- Let unexpected errors propagate — silent failures are worse than crashes.
- Never `except: pass`. Never swallow stack traces.

### 9. Be deliberate about secrets and PII

- Skills must **not** read environment variables for credentials directly; accept them as arguments so the caller controls scope.
- Never log full request/response bodies for authenticated calls.
- Redact obvious PII (emails, phone numbers, account numbers) from any debug output.
- Add a secret-scan step to the test suite for any skill that touches credentials.

### 10. Keep skills token-efficient

The agent pays for every byte it reads.

- Keep `SKILL.md` under ~200 lines. Move long examples into `examples/`.
- Don't dump entire files into the agent context — return paths, summaries, or paginated chunks.
- Prefer streaming or chunked outputs for anything larger than ~10 KB.

### 11. Ship tests with every skill

Each skill directory must include `tests/` with:

- a happy-path test using a realistic input fixture,
- at least one error-path test (bad input, missing file, network failure),
- a test that asserts the output schema (not just "didn't crash").

CI runs `pytest skills/<name>/tests/` for every changed skill on every PR.

### 12. Version and document breaking changes

- Bump a `version:` field in `SKILL.md` using semver.
- Breaking changes (renamed args, changed output shape, removed triggers) require:
  1. a major-version bump,
  2. an entry in `CHANGELOG.md`,
  3. a deprecation notice left in place for at least one minor version before removal.
- Never rewrite history of a released skill. Add a new skill if the contract has to change incompatibly.

---

## Contributing a new skill

1. Create `skills/<your-skill-name>/` with `SKILL.md`, an entry point, `examples/`, and `tests/`.
2. Run the test suite locally: `pytest skills/<your-skill-name>/tests/`.
3. Open a PR. The PR description should answer:
   - What problem does this skill solve?
   - Which triggers route to it?
   - What are its side effects, if any?
4. A reviewer will check the skill against the [best practices](#best-practices) above before merging.

Welcome aboard — keep your skills small, sharp, and well-documented.
