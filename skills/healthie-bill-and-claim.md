---
name: Bill a Healthie visit and file the claim
description: Take a completed Healthie visit through charge capture, patient payment, insurance eligibility and a CMS-1500 claim, using only operations that exist in the Healthie GraphQL schema.
api: graphql/healthie-schema.graphql
endpoint: https://api.gethealthie.com/graphql
sandbox_endpoint: https://staging-api.gethealthie.com/graphql
operations:
  - createBillingItem
  - createRequestedPayment
  - completeCheckout
  - runEligibilityCheck
  - createInsuranceAuthorization
  - createCms1500
  - updateCms1500
  - cms1500s
  - billingItems
generated: '2026-08-14'
method: generated
source: graphql/healthie-schema.graphql + https://docs.gethealthie.com/guides/billing/
---

# Bill a Healthie visit and file the claim

This is the money path. It touches PHI *and* payment data, and it has the worst consequence of
any Healthie flow if you get retries wrong.

## Before you start

```
Content-Type: application/json
Authorization: Basic <API_KEY>
AuthorizationSource: API
Healthie-GraphQL-API-Version: 2026-01-01
```

## Steps

### 1. Check coverage

`runEligibilityCheck` verifies the patient's insurance before you commit work. Failures come back
as `ClaimEligibilityCheckErrors { code, description }` — a domain-specific error object separate
from both the GraphQL `errors[]` array and the mutation `messages` list. The `code` values are
passed through from the clearinghouse and are **not** enumerated by Healthie, so match on them
defensively.

**Read the clearinghouse situation before you build.** The schema deprecated
`EligibilityCheckService.change_health` on 2026-06-26 with the reason "ChangeHealth integration
has been discontinued. Use `claim_md` instead", and deprecated
`ClaimDestinationIntegration.change_health` the same day. `uploadBatchToChangeHealth` is
deprecated for the same reason. Target **ClaimMD**. Note that
https://docs.gethealthie.com/guides/integrations/ still describes the Change Healthcare E-Labs
integration as current — the schema is the more recent source of truth here.

### 2. Authorization, where the payer requires it

`createInsuranceAuthorization` records the prior authorization.

### 3. Capture the charge

`createBillingItem` creates the charge. `BillingItem` gained a `metadata` JSON field in the
2026-07-10 changelog — **use it to carry your own correlation key**, because it is the closest
thing this API offers to an idempotency key (see the retry rule below).

For a patient-facing purchase flow use `completeCheckout` instead; it is the mutation that
replaced the deprecated `createClientViaForm`.

### 4. Request payment from the patient

`createRequestedPayment` raises the patient responsibility. Healthie's published processing rate
is 2.9% + $0.30 per transaction.

### 5. File the CMS-1500

`createCms1500` builds the professional claim; `updateCms1500` amends it; `cms1500s` lists them.
Claim adjustment reasoning uses the standard `AdjustmentGroup` enum — `CO`, `OA`, `PR`, `PI`.

Rejection detail from the clearinghouse is exposed through the
`ClaimMdRejectionMessagesInfo` enum, whose values explain why a rejection-messages field is
*empty* rather than what the rejection was:
`CLAIM_MD_REJECTION_MESSAGES_NOT_AVAILABLE`, `NO_RECENT_CLAIM_SUBMISSION`,
`NOT_A_CLAIM_MD_CLAIM`, `NO_REJECTION_MESSAGES`.

## Rules an agent must follow

**Never blind-retry a write on this path.** Healthie has no idempotency contract of any kind —
no `Idempotency-Key` header, no de-duplication window, and `clientMutationId` was removed from
all mutations in version `2025-10-15` after being documented as unused. A retried
`createBillingItem`, `createRequestedPayment` or `createCms1500` produces a duplicate charge or a
duplicate claim. Required pattern:

1. Write a unique correlation key into `metadata` on create where the type supports it.
2. On timeout or ambiguous failure, **query first** (`billingItems`, `cms1500s`) filtered to the
   patient, date and amount.
3. Only create again if the query proves nothing landed.

**Check `messages` on every mutation.** A validation failure is HTTP 200 with an empty `errors[]`
array and a populated `messages: [{ field, message }]` list. Silent on the transport, fatal to the
billing run.

**Sandbox limits are real here.** In the Healthie sandbox, Stripe works (use Stripe's published
test bank data), but only a limited subset of ICD and CPT codes is loaded, faxing is unavailable,
and Change Healthcare E-Labs is unavailable. If `createBillingItem` or `completeCheckout` returns
"Your destination account needs to have at least one of the following capabilities enabled:
transfers, crypto_transfers, legacy_payments", the sandbox test bank setup is incomplete.

**No test clock.** There is no time-simulation tooling, so recurring-billing and aging behaviour
cannot be fast-forwarded in test.

**PCI scope.** Healthie's PCI DSS Service Provider Level 1 certification is held by its *payment
processor*, not by Healthie. Do not treat "Healthie is PCI certified" as covering your own
handling of card data.

## Related

- Errors and reason codes: `errors/healthie-error-codes.yml`
- Conventions: `conventions/healthie-conventions.yml`
- Sandbox: `sandbox/healthie-sandbox.yml`
- Lifecycle and deprecations: `lifecycle/healthie-lifecycle.yml`
