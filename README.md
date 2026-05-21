# sample-test-repo

Sample agent skills repository (created for ISS-144).

This repository hosts a collection of **agent skills** — small, composable units of
capability that an AI agent can load on demand to accomplish a task. Each skill
lives in its own directory under `skills/` and is described by a `SKILL.md` file
that documents what the skill does, when it should trigger, and how to use it.

## Repository layout

```
sample-test-repo/
├── README.md          # You are here
└── skills/
    └── <skill-name>/
        ├── SKILL.md   # Skill description + trigger words
        └── ...        # Supporting scripts, templates, examples
```

## Best Practices

The guidelines below are intended for anyone authoring or reviewing skills in
this repository. They are written for an AI agent skills repo and are mocked up
for the purposes of ISS-144 — adjust to match your team's conventions.

### 1. Skill design

- **Single responsibility.** A skill should do one thing well. If a `SKILL.md`
  needs more than two top-level sections to describe what it does, split it.
- **Composable, not monolithic.** Prefer small skills that an agent can chain
  (e.g. `extract_pdf_text` + `summarize_text`) over large skills that bundle
  unrelated steps.
- **Deterministic by default.** Given the same inputs, a skill should produce
  the same outputs. Where non-determinism is unavoidable (LLM calls, network),
  document it explicitly in the skill description.
- **No hidden side effects.** A skill that touches the filesystem, network, or
  external APIs must declare it in its `SKILL.md` under a `Side effects` section.

### 2. `SKILL.md` authoring

- **Lead with a one-sentence description.** Agents use this for routing; bury
  the lede and the skill won't be picked up.
- **List explicit trigger words.** Use a `MANDATORY TRIGGERS:` line so the
  agent's skill selector has unambiguous keywords to match against.
- **Show, don't tell.** Include at least one worked example of input → output.
- **Document failure modes.** State what happens when an input is malformed,
  missing, or out of scope, and which fallback skill (if any) the agent should
  reach for next.
- **Keep it under ~200 lines.** If a skill needs more, link out to supporting
  docs and keep `SKILL.md` focused on the contract.

### 3. Inputs and outputs

- **Validate inputs at the boundary.** Reject malformed input early with a
  clear error rather than passing junk downstream.
- **Use safe defaults.** When a parameter is omitted, choose the least-surprising,
  least-destructive default (e.g. dry-run over write).
- **Return structured data.** Prefer JSON-shaped responses over free-form prose
  when the output will be consumed by another skill.
- **Never silently truncate.** If output is shortened, surface that fact in the
  response (e.g. `"truncated": true`).

### 4. Tool and model usage

- **Idempotent operations.** Write skills so that a retry never produces a
  duplicate side effect (use natural keys, upserts, or pre-flight checks).
- **Minimize tokens.** Pass only what the model needs. Strip boilerplate,
  collapse whitespace, and prefer references (IDs, URIs) over inlined blobs.
- **Pick the smallest sufficient model.** Don't reach for a frontier model
  when a smaller one will do; document the choice in `SKILL.md`.
- **Respect rate limits.** Back off exponentially on `429`/`503` and surface
  the wait time to the caller rather than blocking silently.

### 5. Error handling

- **Fail loudly, fail early.** Raise a typed error the agent can route on,
  rather than returning a string that "looks like" success.
- **Distinguish user errors from system errors.** A bad input is not the same
  as a downstream outage; the agent's recovery path differs.
- **Never swallow exceptions.** If you catch, you log and re-raise (or convert
  to a structured error response).

### 6. Security and privacy

- **No secrets in the repo.** API keys, tokens, and credentials live in the
  environment or a secret manager — never in `SKILL.md`, examples, or fixtures.
- **Redact PII in logs.** Treat user inputs as sensitive by default; mask
  emails, phone numbers, and free-text fields before emitting them.
- **Principle of least privilege.** A skill should request the narrowest scope
  it needs (read-only > read-write, single repo > org-wide).
- **Scan for leaked secrets.** Run a secret scanner in CI on every PR.

### 7. Observability

- **Structured logs.** Emit JSON logs with a stable schema (`skill_name`,
  `invocation_id`, `duration_ms`, `outcome`).
- **Trace every invocation.** Propagate a correlation/trace ID through every
  sub-call so failures can be reconstructed end-to-end.
- **Emit metrics.** At minimum: invocation count, success/failure rate, p50/p95
  latency, and token usage.

### 8. Testing and evaluation

- **Unit tests for pure logic.** Cover input validation and edge cases without
  network or model calls.
- **Golden-file tests for prompts.** When a prompt changes, the diff against
  the golden output should be reviewed deliberately, not auto-accepted.
- **Eval suite for behavior.** Maintain a small set of representative tasks
  with expected outcomes; run it on every PR that touches a skill.
- **Regression-test fixed bugs.** Every bug fix lands with a test that would
  have caught it.

### 9. Versioning and compatibility

- **Semantic versioning per skill.** Bump `MAJOR` on breaking input/output
  changes, `MINOR` on additive changes, `PATCH` on fixes.
- **Deprecate, don't delete.** Keep the old skill available for at least one
  release with a `Deprecated:` notice pointing to the replacement.
- **Document the contract.** The input/output schema in `SKILL.md` is the
  contract — change it deliberately and with a version bump.

### 10. Documentation and review

- **Every skill ships with a `SKILL.md`.** No exceptions; a skill without docs
  is effectively invisible to the agent.
- **Two-person review for new skills.** One reviewer for correctness, one for
  fit with the existing skill catalog (to avoid duplicates).
- **Keep examples runnable.** Examples in `SKILL.md` should be copy-pasteable;
  CI should fail if they drift from the implementation.

## Contributing

1. Create a feature branch from `main`.
2. Add or update a skill under `skills/<skill-name>/`.
3. Make sure `SKILL.md` follows the best practices above.
4. Open a PR; include the eval results for any behavioral changes.

## License

See the repository's license file (TBD).
