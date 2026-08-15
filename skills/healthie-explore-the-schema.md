---
name: Explore the Healthie GraphQL schema
description: Find the right Healthie query or mutation out of 819 root fields without burning turns — using the saved SDL, the hosted explorer, or the first-party Dev Assist MCP server.
api: graphql/healthie-schema.graphql
endpoint: https://api.gethealthie.com/graphql
sandbox_endpoint: https://staging-api.gethealthie.com/graphql
operations:
  - __schema
  - createApiKey
  - apiKeys
  - organization
generated: '2026-08-14'
method: generated
source: graphql/healthie-schema.graphql + https://github.com/healthie/healthie-dev-assist
---

# Explore the Healthie GraphQL schema

Healthie's schema is large: **386 queries, 427 mutations, 6 subscriptions, 1,583 types**. There is
no OpenAPI to skim and no resource index to walk. Discovery is the first real problem.

## Four ways in, best first

### 1. The saved SDL (fastest, offline, free)

`graphql/healthie-schema.graphql` in this repository is the complete schema in SDL form with
descriptions and `@deprecated` reasons preserved. Grep it. No credentials, no network, no rate
limit, no turn cost.

```
grep -n "^  createAppointment" graphql/healthie-schema.graphql
grep -n "^type Appointment " graphql/healthie-schema.graphql
```

### 2. Introspect the sandbox directly

Anonymous introspection is **refused on production** (`https://api.gethealthie.com/graphql`
returns HTTP 500) but **open on the sandbox** (`https://staging-api.gethealthie.com/graphql`),
which serves the same schema. That is how the SDL above was captured:

```
POST https://staging-api.gethealthie.com/graphql
Content-Type: application/json

{"query":"{__schema{queryType{name}}}"}
```

Re-run the full introspection query when you need a fresher copy than the saved SDL.

### 3. Healthie's hosted explorer

- API Explorer / GraphiQL: https://docs.gethealthie.com/graphiql
- Version-pinned schema reference: https://docs.gethealthie.com/reference

The reference is pinned per API version (`/reference/2024-06-01`), which matters — the schema you
get depends on the `Healthie-GraphQL-API-Version` header you send.

### 4. Dev Assist, Healthie's own MCP server

https://github.com/healthie/healthie-dev-assist — MIT licensed, first-party, TypeScript.

It is **local stdio only**. There is no hosted endpoint; a human must clone and run it:

```
git clone https://github.com/healthie/healthie-dev-assist.git
cd healthie-dev-assist && npm install
cp .env.example .env        # set HEALTHIE_API_KEY
npm run regenerate-schema
claude mcp add healthie -- npx tsx /path/to/healthie-dev-assist/src/server.ts
```

Dev Assist 2.0 exposes exactly **one** working tool, `execute_healthie_code`, plus
`regenerate_schema`. You write a short async TypeScript program against a sandboxed `healthie`
object and it runs in a single turn:

```typescript
const mutations = await healthie.search("appointment", { kind: "mutation" });
const details   = await healthie.introspect("createAppointmentInput");
return { mutations, details };
```

Available methods: `healthie.search(query, {kind})`, `healthie.introspect(typeName, {depth})`,
`healthie.query(graphql, variables)`, `healthie.mutate(graphql, variables)`. Search and introspect
work with no API key; query and mutate require one. The sandbox has no filesystem, network or
shell access.

**Consequence for tool discovery:** because Dev Assist collapses everything into one general tool,
an MCP `tools/list` tells an agent nothing about appointments, claims or charting. The schema is
the capability map, not the tool list.

## Orientation — where things live

The schema uses `snake_case` field names on types and `camelCase` on root fields. Core entities:

- **`User`** — patients, providers and staff all. There is no `Patient` type.
- **`Appointment`**, `AppointmentType`, `Availability`, `AppointmentSetting`
- **`CustomModuleForm`** (template) / **`FormAnswerGroup`** (submission) — backs both intake forms
  and charting notes
- **`CarePlan`**, `Goal`, `Entry`, `Metric`
- **`Cms1500`**, `Claim`, `Policy`, `InsurancePlan`, `BillingItem`, `RequestedPayment`, `Offering`
- **`Conversation`** / `Note` — messaging
- **`Organization`**, `OrganizationMembership`, `UserGroup`

`data-model/healthie-data-model.yml` has the derived relationship graph for 48 core entities.

## Rules an agent must follow

**Check for `@deprecated` before you use anything.** The schema carries 513 deprecated fields and
20 deprecated enum values across 461 types. 425 of those are `clientMutationId` on mutation
payloads ("DO NOT USE"), leaving roughly 88 real ones. Deprecations are in the contract, so your
tooling can see them — read the reason, it usually names the replacement (`timesForRange` ->
`availableSlotsForRange`, `prescriptions` -> `prescriptionMedications`,
`createFormAnswerGroupSigning` -> `signFormAnswerGroup`).

**Pin the version header.** Without `Healthie-GraphQL-API-Version` you get the `2024-06-01`
baseline. Latest is `2026-01-01`. Versions are cumulative and only breaking changes are gated;
additive changes appear in every version immediately.

**Budget query cost, not request count.** Max complexity 2000, max depth 25. Connection cost is
multiplied by page size, and an omitted page size is scored as 100. Introspection queries are
large — run them against the sandbox.

**Never introspect production anonymously and conclude the API is down.** The HTTP 500 there is
the expected refusal, not an outage. Check https://status.gethealthie.com/ instead.

## Related

- Schema: `graphql/healthie-schema.graphql`
- MCP server: `mcp/healthie-mcp.yml`
- Tool crosswalk: `mcp/healthie-tool-crosswalk.yml`
- Data model: `data-model/healthie-data-model.yml`
- Lifecycle and deprecations: `lifecycle/healthie-lifecycle.yml`
