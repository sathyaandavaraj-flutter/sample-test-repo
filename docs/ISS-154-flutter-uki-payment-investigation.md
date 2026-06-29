# ISS-154 — Flutter UKI payment/withdrawal investigation & triage runbook

> **Status:** Duplicate — consolidated into the canonical investigation **ISS-148**
> **Priority:** P1
> **Source:** Slack thread (auto-filed by the Slack Message Agent, triaged by the On Call Agent)
> **DevRev:** `ISS-154` (`don:core:dvrv-us-1:devo/1pyquBtpWW:issue/154`)

## ⚠️ This is a duplicate

ISS-154 was auto-created from a Slack thread and carries the **same
Slack-sourced template** as a cluster of earlier issues: **ISS-148, ISS-149,
ISS-150, ISS-151, ISS-152, and ISS-153**. The on-call triage notes describe the
same **payment / withdrawal failure of unknown cause** (payment gateway, internal
wallet, bank/settlement, system error, or product configuration).

The earliest issue in the cluster, **ISS-148**, is the canonical record and is
already **In Development**. ISS-154 has therefore been linked as a duplicate of
ISS-148 in DevRev and moved to the `duplicate` stage so a single investigation
tracks the work instead of seven parallel ones.

This document is retained as the per-issue artifact for ISS-154 and mirrors the
runbook captured for its siblings; the live investigation continues on
**ISS-148**.

## Why this document exists

The issue text states the behavior *"may involve product logic or configuration
beyond engineering scope"* and that *"the Product team's guidance is needed."*
However, the on-call triage notes actually describe a **failed payment /
withdrawal of unknown cause**. There is **no engineering-scoped defect identified
yet**, and this repository contains no application code for Flutter UKI, so there
is nothing to patch here.

The risk with this class of issue is that it **stalls silently** while waiting on
the wrong team. This runbook captures the investigation so the next on-call can
either confirm an engineering bug *or* escalate to Product with a specific,
decidable question — instead of leaving the issue parked in Triage.

## Summary & impact

- A Flutter UKI payment/withdrawal is failing (or behaving unexpectedly).
- Root cause is **not yet confirmed** — it could be a code defect, a gateway/bank
  problem, a wallet/ledger problem, or intended-but-misconfigured product behavior.
- **Impact:** blocks further investigation/implementation; may affect user
  experience and release timelines.

## Root-cause hypotheses

| # | Hypothesis | Likely owner | How to confirm / rule out |
|---|------------|--------------|---------------------------|
| 1 | **Code / system error** in the payment-processing path | Engineering | Reproduce; check application/error logs and recent deploys for stack traces or error codes on the failing transactions. |
| 2 | **Payment-gateway failure** (integration or outage) | Engineering + Payments vendor | Check gateway status page and gateway-side response codes; confirm whether failures correlate with a vendor incident. |
| 3 | **Internal wallet issue** (balance discrepancy, ledger, transaction limits) | Engineering / Payments | Inspect the wallet ledger for the affected user(s): available balance, holds, and limit checks at the time of the attempt. |
| 4 | **Bank / settlement delay** | Payments / Ops | Check settlement timing and bank response codes; determine whether the payment is failing vs. merely pending. |
| 5 | **Product logic / configuration** (intended behavior or a config change) | **Product** | Confirm the *expected* behavior and whether a recent config/business-rule change is responsible. This is the only branch that genuinely needs Product input. |

## Immediate triage actions

1. **Capture the symptom precisely** — exact error message/code shown to the user,
   transaction ID(s), timestamp, amount, currency, user/account ID, and the Slack
   thread link.
2. **Verify transaction logs** — pull logs for the affected payment(s) and record
   any error codes or failure reasons.
3. **Check wallet balance/ledger** — confirm sufficient funds and review recent
   transactions that may have affected the balance or triggered a limit.
4. **Check the payment processor/gateway** — look for known outages and inspect the
   gateway's response for the failing transactions.
5. **Review recent changes** — deploys, feature flags, and product/config changes
   in the relevant window.
6. **Mitigate if systemic** — if a class of withdrawals is affected, consider a
   temporary, reversible hold on similar withdrawals until the root cause is found.

## The decision fork (do this before escalating)

Use the evidence above to classify the issue. **Do not default to "needs Product
input."**

- **Engineering bug** → convert to a scoped engineering task with the failing
  transaction IDs, logs, and a reproduction. Remove the "needs Product input" framing.
- **Product decision** → escalate to Product (below) with a single, decidable
  question. Only hypothesis #5 belongs here.

## Escalating to Product (only when warranted)

When the blocker is genuinely a product decision, escalate with:

- The **observed vs. expected** behavior.
- Confirmation that an **engineering bug has been ruled out** (with evidence).
- A **specific, decidable question** (e.g., "Should withdrawals above X be blocked
  when the wallet hold is pending, or allowed?").
- **Impact and a decision-by checkpoint**, plus the originating Slack thread and logs.
- Assign/label to the Product owner and record the decision back on the canonical
  issue (ISS-148).

## Information to collect (checklist)

- [ ] Failing transaction ID(s) and timestamps
- [ ] Exact error code/message
- [ ] Affected user/account ID(s) and wallet balance/ledger snapshot
- [ ] Gateway response codes / vendor incident status
- [ ] Recent deploys, flags, and config changes in the window
- [ ] Whether the failure is isolated or systemic
- [ ] Originating Slack thread link

## Definition of done

Because ISS-154 is a duplicate, it is **done when it is consolidated**: linked as a
duplicate of **ISS-148** and moved out of Triage to the `duplicate` stage (both
completed as part of this change). The underlying payment/withdrawal investigation
is tracked to completion on **ISS-148**, which can close when **either**:

- the root cause is confirmed to be an **engineering defect** and a scoped task
  (with reproduction + evidence) has been created and linked; **or**
- the root cause is confirmed to be a **product decision**, the specific question
  has been answered by Product, and the decision is recorded on the issue.

## Duplicate cluster

| Issue | Role |
|-------|------|
| **ISS-148** | Canonical — In Development; tracks the investigation |
| ISS-149 | Duplicate of the same Slack-sourced incident |
| ISS-150 | Duplicate of the same Slack-sourced incident |
| ISS-151 | Duplicate of the same Slack-sourced incident |
| ISS-152 | Duplicate of the same Slack-sourced incident — linked to ISS-148 |
| ISS-153 | Duplicate of the same Slack-sourced incident — linked to ISS-148 |
| **ISS-154** | Duplicate (this issue) — linked to ISS-148 |
