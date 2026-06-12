# Runbook: Payment / Withdrawal Failure Triage

**Status:** Active · **Severity it applies to:** up to P0 · **Owner:** On-call engineer (escalates to Product when flagged)

This runbook turns a recurring class of high-priority report — *"a payment or
withdrawal is failing and it's unclear whether it's an engineering bug or a
product decision"* — into a repeatable triage path. It exists because these
issues tend to be filed as *"needs Product-team input"* and then stall silently
while the root cause is actually unknown.

The goal is to **diagnose first, then escalate with a precise question** so the
issue does not block a release while waiting on the wrong team.

> Source: on-call triage notes on DevRev issue **ISS-149**
> (`don:core:dvrv-us-1:devo/1pyquBtpWW:issue/149`).

---

## 1. When to use this runbook

Use it when **all** of the following are true:

- A user-facing **payment, deposit, or withdrawal** did not complete as expected.
- The expected behaviour is **not obviously a code bug** (i.e. someone has
  suggested it "may involve product logic or configuration").
- The issue is **blocking** further investigation, a release, or users.

If the failure is clearly a code defect with a known fix, skip the runbook and
file/patch the bug directly.

---

## 2. Symptoms to capture first

Before investigating, record the concrete facts so the issue is reproducible and
escalation-ready:

- **What the user did** (flow, amount, currency, region/brand).
- **What happened** vs. **what was expected**.
- **Scope:** one user, a cohort, or systemic? Approximate count and start time.
- **Identifiers:** transaction ID(s), wallet/account ID, payment-processor
  reference, request/trace ID, timestamp (with timezone).

---

## 3. Root-cause hypotheses

Work through these in parallel; they are ordered roughly from most to least
self-serviceable by engineering.

| # | Hypothesis | What it looks like | How to confirm / rule out |
|---|------------|--------------------|---------------------------|
| 1 | **System / code error** | Errors or exceptions in the payment path; recent deploy correlates with onset | Check error logs and the deploy/change timeline (§4) |
| 2 | **Payment-gateway failure** | Declines, timeouts, or 5xx from the processor; provider incident | Check processor status page and gateway response codes (§4) |
| 3 | **Internal wallet issue** | Balance discrepancy, insufficient-funds despite deposit, transaction-limit hit | Inspect wallet balance and recent ledger entries (§4) |
| 4 | **Bank / settlement delay** | Funds debited but not settled; provider reports "pending" | Confirm settlement status with the processor/bank (§4) |
| 5 | **Product logic / configuration** | System behaved "as configured" but the configured behaviour is wrong or undefined | Only after 1–4 are ruled out → escalate to Product (§6) |

> Do **not** jump to hypothesis #5 first. "Needs Product input" should be a
> *conclusion* reached after ruling out engineering causes, not the opening move.

---

## 4. Immediate triage actions

1. **Verify transaction logs.** Pull logs for the affected transaction ID(s).
   Look for error messages, declined/failed codes, and the last successful step
   in the flow.
2. **Check wallet balance & ledger.** Confirm the account had sufficient funds
   and review recent transactions that could have affected the balance or hit a
   limit.
3. **Check the payment processor.** Review gateway response codes and the
   provider's status page for known outages; if needed, request detailed logs
   for the failed transactions.
4. **Review recent changes.** Correlate onset time with recent deploys, config
   changes, or feature-flag flips to the payment/wallet path.
5. **Review product configuration.** Inspect the relevant settings/business
   rules (limits, eligibility, regional rules) that govern the flow. Note any
   value that looks wrong or undefined — this becomes the question for Product.

---

## 5. Decide: engineering bug or product decision?

This is the key fork that keeps the issue from stalling.

- **Engineering bug** → reproducible fault in code, gateway, wallet, or config
  *application*. Action: fix it (or open a scoped bug with the failing step and
  logs). No Product decision required.
- **Product decision** → the system did what it was configured to do, but the
  **expected** behaviour is undefined or disputed (e.g. "should a withdrawal be
  allowed when the wallet is below the regional minimum?"). Action: escalate
  per §6.

If you cannot decide, default to continuing engineering investigation and state
explicitly what evidence would settle it.

---

## 6. Escalate to Product (only when §5 says so)

When the blocker is genuinely a product decision, escalate with a **specific,
decidable question** — not "please advise":

1. **Reassign / label** the issue to the Product owner and set
   priority/impact so it is visible.
2. **State the decision needed** as a yes/no or option-pick, e.g.
   *"Expected behaviour when X? Option A (…) vs Option B (…)."*
3. **Attach evidence:** transaction IDs, logs, current configuration, and the
   originating Slack thread.
4. **List the options** with trade-offs (user impact, risk, effort).
5. **Request a decision checkpoint** and record the answer on the issue once
   received.

---

## 7. Mitigation while you investigate

If the failure looks systemic and is actively harming users:

- Consider **temporarily holding similar withdrawals/payments** to prevent
  further impact until the root cause is found.
- Communicate the hold and its scope on the issue.
- Revert the hold as soon as the cause is fixed or ruled benign.

---

## 8. Definition of done

The issue can leave the "needs triage / needs Product input" state when:

- Root cause is identified (one of the §3 hypotheses confirmed), **and**
- Either a fix/mitigation is in place **or** a Product decision is recorded on
  the issue, **and**
- Any temporary hold (§7) has been lifted or has an owner and end condition.

---

## References

- DevRev issue: **ISS-149** — *Flutter UKI Issue Requiring Product Team Input
  Blocking Progress and Affecting Releases.*
- Related process runbook: *Escalating Issues That Need Product Input* (ISS-148).
- Related issues: ISS-100 (*Wallet Balance Not Updating After Deposit*),
  ISS-42 (*Payment Gateway Expansion*).
