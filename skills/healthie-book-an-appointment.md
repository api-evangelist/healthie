---
name: Book a Healthie appointment
description: Find real availability for a Healthie provider and book, reschedule or cancel an appointment through the Healthie GraphQL API, handling Healthie's validation-message pattern and its lack of idempotency.
api: graphql/healthie-schema.graphql
endpoint: https://api.gethealthie.com/graphql
sandbox_endpoint: https://staging-api.gethealthie.com/graphql
operations:
  - availableSlotsForRange
  - appointmentTypes
  - appointments
  - createAppointment
  - updateAppointment
  - deleteAppointment
generated: '2026-08-14'
method: generated
source: graphql/healthie-schema.graphql + https://docs.gethealthie.com/guides/scheduling/appointments/
---

# Book a Healthie appointment

Healthie is a **GraphQL** API. There is no REST surface and no OpenAPI — every step below is a
GraphQL document POSTed to the single endpoint.

## Before you start

Send these headers on every request:

```
Content-Type: application/json
Authorization: Basic <API_KEY>
AuthorizationSource: API
Healthie-GraphQL-API-Version: 2026-01-01
AuthorizationShard: <SHARD_ID>     # only if this customer is on a shard
```

`Authorization: Basic` is a literal — send the raw API key, not a base64 `user:password` pair.
Pin the version header; without it you are served the `2024-06-01` baseline, which paginates
several of the fields below differently.

Develop against `https://staging-api.gethealthie.com/graphql`. IDs are **not** portable between
sandbox and production, so never hard-code one.

## Steps

### 1. Find the appointment type

`appointmentTypes` lists what the practice offers. You need an appointment type ID before you can
look for slots, because slot length and provider eligibility are properties of the type.

Use a connection page size. An unpaginated connection is scored against a default page size of
100, which is the usual way to trip the complexity ceiling of 2000.

### 2. Find real availability

Query `availableSlotsForRange` for the provider and date range. Do **not** use `timesForRange` —
it is deprecated in the schema in favour of `availableSlotsForRange`.

Times are `ISO8601DateTime`. Confirm the practice timezone before interpreting them; see
https://docs.gethealthie.com/guides/api-concepts/timezones.

### 3. Book it

Call `createAppointment`. Two validations were added as breaking changes and will reject an
otherwise-plausible booking:

- **In-person appointments require `appointment_location_id`** (added in version `2025-04-01`).
- **Individual appointments accept only one attendee** when specified via `attendee_ids`
  (added in version `2025-05-15`). For individual appointments Healthie still prefers `user_id`.

### 4. Read the result correctly — this is the step people get wrong

Always select `messages { field message }` alongside the appointment:

```graphql
mutation BookAppointment($input: createAppointmentInput!) {
  createAppointment(input: $input) {
    appointment { id datetime contact_type appointment_type { id name } }
    messages { field message }
  }
}
```

A **validation failure returns HTTP 200 with an empty GraphQL `errors[]` array** and a populated
`messages` list, with `appointment` null. If you branch only on `errors`, you will report a
booking that never happened. Treat a non-empty `messages` array as a failure.

### 5. Reschedule or cancel

- Reschedule: `updateAppointment` with the appointment `id` and the new `datetime`.
- Cancel: `deleteAppointment`.
- Verify afterwards with `appointments` or `appointment`.

## Rules an agent must follow

**There is no idempotency.** Healthie has no `Idempotency-Key` header and no request
de-duplication. `clientMutationId` is documented as unused and was removed from all mutations in
version `2025-10-15`. If `createAppointment` times out, **do not blindly retry** — first query
`appointments` filtered to that client, provider and datetime to see whether the booking landed.
A blind retry double-books a patient.

**Respect the cost limits.** Max query complexity 2000, max query depth 25. Always pass `first:` /
`page_size:` on connections. Errors read:
`Query has complexity of 2274, which exceeds max complexity of 2000`.

**Back off blindly on rate limits.** A rate-limit rejection arrives as
`{"errors":[{"message":"Too many requests. Please try again later.","extensions":{"code":"TOO_MANY_REQUESTS"}}]}`.
Healthie publishes **no** `X-RateLimit-*`, `RateLimit-*` or `Retry-After` headers, so you cannot
compute a backoff from the response — use exponential backoff with jitter.

**Paginate with cursors.** Use `after` plus the `cursor` returned on the last item, and reset
pagination whenever you change `order_by`. Use the sibling count field
(`appointmentsCount`) to size the walk.

## Related

- Conventions: `conventions/healthie-conventions.yml`
- Errors: `errors/healthie-error-codes.yml`
- Rate limits: `rate-limits/healthie-rate-limits.yml`
- Sandbox: `sandbox/healthie-sandbox.yml`
