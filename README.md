# sample-test-repo

Sample agent skills repository (created for ISS-144).

## Escalating Issues That Need Product Input

Some issues surface behaviour that engineering alone cannot resolve: the
expected behaviour depends on product logic, configuration, or policy that
must be decided by the Product team. When these issues sit in an ambiguous
state they block investigation and can quietly slip release timelines.

This runbook describes how to triage, escalate, and unblock such issues so
they do not stall silently. It was added in response to **ISS-148**.

### When to use this runbook

Use this process when an issue meets **any** of the following:

- The expected/correct behaviour is undefined and needs a product decision.
- Resolving it requires a change to product configuration or business rules.
- It is blocking other work or a release and ownership is unclear.

### Triage checklist

Before escalating, capture enough context for the Product team to make a
decision quickly:

1. **Summarise the observed behaviour** alongside the behaviour you expected.
2. **Confirm it is not an engineering bug** — reproduce it and check logs to
   rule out a defect in code or configuration you already own.
3. **State the impact** — who or what is affected, and whether a release is
   at risk.
4. **List the open questions** that only Product can answer, framed as
   concrete yes/no or option-based decisions.
5. **Propose options** with trade-offs where you can, so the ask is a choice
   rather than an open-ended request.

### Escalation steps

1. **Label and assign.** Mark the issue as blocked / needs-product-input and
   assign (or @mention) the responsible Product owner.
2. **Link the context.** Attach the originating thread (e.g. the Slack
   thread), logs, and reproduction steps.
3. **Set priority and impact** explicitly, including any release deadline.
4. **Request a decision by a clear checkpoint** rather than an open-ended
   "please advise".
5. **Record the decision** on the issue once Product responds, then convert
   it into engineering follow-up tasks.

### Definition of done

The issue can leave the "needs product input" state when:

- The Product team has confirmed the expected behaviour, **and**
- That decision is recorded on the issue, **and**
- Either the fix is implemented, or a follow-up engineering task is created
  and linked.
