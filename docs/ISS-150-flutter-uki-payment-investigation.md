# ISS-150 — Flutter UKI Payment Withdrawal Investigation & Triage Runbook

> **Status:** Triage / awaiting Product team input
> **Priority:** P0
> **Source:** Auto-created from a Slack thread by the Slack Message Agent and triaged by the On Call Agent.
> **DevRev issue:** ISS-150

## 1. Summary

A problem was reported in the **Flutter UKI** flow that surfaces as payment / withdrawal
behavior that does not match expectations. Initial investigation suggests the behavior may be
driven by **product logic or configuration** rather than a pure engineering defect, so Product
team input is required to confirm the *expected* behavior before an implementation fix can be
scoped.

Because the root cause is not yet confirmed, this document is a **triage runbook**: it records
what we know, the working hypotheses, the diagnostics to run, and the open questions that block
progress. It is intended to be updated as the investigation proceeds.

## 2. Impact

- Blocks further investigation / implementation until expected behavior is confirmed.
- May affect end-user experience (failed or delayed withdrawals).
- May affect release timelines.

## 3. Root-cause hypotheses

Ordered roughly from "most likely to be in product/config scope" to "most likely to be an
engineering defect". Each should be confirmed or ruled out using the diagnostics in Section 4.

| # | Hypothesis | Owner | How to confirm / rule out |
|---|------------|-------|---------------------------|
| 1 | **Product logic / configuration** — incorrect product settings or business rules drive the unexpected behavior. | Product | Review current config vs. intended config; check recent config changes (Section 5). |
| 2 | **Payment gateway failure** — integration or availability problem with the external payment gateway. | Eng + Payments vendor | Inspect gateway responses / error codes; check vendor status page. |
| 3 | **Internal wallet issues** — balance discrepancies or transaction limits in the internal wallet. | Eng | Verify wallet balances and limit configuration for affected users. |
| 4 | **Bank processing delays** — downstream bank delays affecting completion. | Payments ops | Correlate failed/pending transactions with bank settlement windows. |
| 5 | **System error / code defect** — bug in the payment-processing path. | Eng | Reproduce in a controlled environment; inspect stack traces / logs. |

## 4. Immediate diagnostic steps

1. **Verify transaction logs**
   - Pull logs for the affected payments/withdrawals.
   - Capture any error codes or messages that describe the failure mode.
2. **Check wallet balance & limits**
   - Confirm affected users have sufficient funds.
   - Confirm no withdrawal limits/holds are being applied unexpectedly.
3. **Contact the payment processor**
   - Ask about ongoing incidents or outages.
   - Request detailed reports/logs for the failed transactions.
4. **Review product configuration**
   - With Product, review the settings/logic governing payment processing.
   - Identify recent changes that could explain the behavior.
5. **Temporarily hold similar withdrawals** *(only if the issue looks systemic)*
   - Pause comparable withdrawal requests to limit user impact until the root cause is found.

## 5. Information to collect (for whoever picks this up)

- [ ] Exact reproduction steps and a concrete failing example (user/transaction ID, timestamp).
- [ ] Error codes/messages observed in transaction logs.
- [ ] Affected scope: how many users / what % of withdrawals, since when.
- [ ] Any recent deploys or configuration changes around the start of the issue.
- [ ] Payment gateway status / vendor incident reference, if any.

## 6. Open questions for the Product team

These must be answered before an engineering fix can be scoped:

1. **What is the expected behavior** for the Flutter UKI flow in the reported scenario?
2. Is the current behavior driven by an intended product rule or configuration?
3. Are there known recent product/config changes that could explain the change in behavior?
4. What is the desired resolution — adjust configuration, change the rule, or fix code?

## 7. Next steps

1. Route the open questions in Section 6 to the Product team and capture their answers here.
2. Run the diagnostics in Section 4 and fill in the checklist in Section 5.
3. Once expected behavior is confirmed, convert the confirmed root cause into a scoped
   engineering task (or a configuration change) and link it back to ISS-150.

---
*This runbook was created to resolve the triage step of ISS-150. It does not implement a code
fix because the underlying cause is not yet an engineering-scoped defect — confirming expected
behavior with the Product team is the gating next step.*
