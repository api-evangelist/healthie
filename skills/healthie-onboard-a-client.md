---
name: Onboard a Healthie client
description: Create a client record in Healthie, send them the practice's intake forms, and read back the completed submissions through the GraphQL API.
api: graphql/healthie-schema.graphql
endpoint: https://api.gethealthie.com/graphql
sandbox_endpoint: https://staging-api.gethealthie.com/graphql
operations:
  - createClient
  - updateClient
  - users
  - user
  - customModuleForms
  - customModuleForm
  - createFormAnswerGroup
  - formAnswerGroups
  - formAnswerGroup
generated: '2026-08-14'
method: generated
source: graphql/healthie-schema.graphql + https://docs.gethealthie.com/guides/forms/
---

# Onboard a Healthie client

In the Healthie schema a patient/client is a **`User`**. There is no separate `Patient` type and
no `createPatient` mutation — clients, providers and staff are all `User` records differentiated
by role and organization membership.

## Before you start

```
Content-Type: application/json
Authorization: Basic <API_KEY>
AuthorizationSource: API
Healthie-GraphQL-API-Version: 2026-01-01
```

Decide your key strategy first, because it determines who the audit log blames. An API key
inherits the permissions and identity of the Healthie user account it is attached to, and every
action taken with it is recorded as that user. For headless patient-facing experiences Healthie
recommends a **key per end user**; for backend/data work a single **service-account key**.

## Steps

### 1. Create the client

Call `createClient`. Select `messages { field message }` on the payload — Healthie added
validation and nullability changes to common mutations in version `2024-11-01`, and a rejected
create returns HTTP 200 with a null user and a populated `messages` array.

Note the deprecated path: `createClientViaForm` exists but is deprecated with the reason
"Replaced by completeCheckout". Use `createClient` for plain onboarding and `completeCheckout`
only when the flow includes buying an offering.

### 2. Find the intake forms

`customModuleForms` lists the practice's form templates. A `CustomModuleForm` holds
`CustomModule` questions. Since the 2026-07-17 changelog a form also carries `is_scribe_form`,
flagging templates that are targets for AI-scribe generation rather than patient intake — filter
those out of an onboarding flow.

### 3. Submit or request the intake

`createFormAnswerGroup` records a submission of a `CustomModuleForm` for a user. A
`FormAnswerGroup` is Healthie's universal "filled-out form" object — it backs both patient intake
and clinical charting notes, so the same operations serve both.

### 4. Read submissions back

Use `formAnswerGroups` (list) or `formAnswerGroup` (single). Both accept `start_date` and
`end_date` arguments, added by accretion in the 2026-07-10 and 2026-07-24 changelog entries, so
you can poll a window instead of walking the whole history. `formAnswerGroupsCount` gives you the
total for the same filters.

### 5. Prefer events over polling

Two published surfaces beat a poll loop:

- **Webhooks** — Healthie posts a thin envelope
  `{ resource_id, resource_id_type, event_type }` where `resource_id_type` is one of
  `Appointment`, `FormAnswerGroup`, `Entry` or `Note`. You must call back into GraphQL to fetch
  the record. Deliveries are HMAC-SHA256 signed over method, path, query, content digest, content
  type and content length; verify before acting. See `asyncapi/healthie-webhooks-asyncapi.yml`.
- **Subscriptions** — `formAnswerGroupModifiedSubscription` pushes changes over WebSocket. Note
  Healthie uses the **ActionCable/AnyCable** protocol, *not* the Apollo websocket subprotocol; the
  Apollo client's default subscription transport will not connect. See
  `asyncapi/healthie-subscriptions-asyncapi.yml`.

## Rules an agent must follow

**No idempotency.** There is no `Idempotency-Key` and no de-duplication window. Before retrying a
failed `createClient`, query `users` for the email or name to check whether the record already
exists — otherwise you create a duplicate patient chart.

**Read `messages`, not just `errors`.** Validation failures never populate the GraphQL `errors[]`
array.

**PHI discipline.** Everything here is protected health information. Do not echo names, dates of
birth, form answers or note content into logs, prompts, traces or third-party tools. Healthie
operates under HIPAA with a BAA (https://www.gethealthie.com/baa); the same obligation flows to
anything you build on it.

**Sandbox first.** `https://staging-api.gethealthie.com/graphql` is fully isolated. IDs do not
transfer to production.

## Related

- Conventions: `conventions/healthie-conventions.yml`
- Authentication: `authentication/healthie-authentication.yml`
- Data model: `data-model/healthie-data-model.yml`
