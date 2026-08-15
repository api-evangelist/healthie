---
name: Subscribe to Healthie events
description: Register and verify Healthie webhooks, and open a GraphQL subscription over Healthie's ActionCable/AnyCable WebSocket, so an integration reacts to changes instead of polling.
api: graphql/healthie-schema.graphql
endpoint: https://api.gethealthie.com/graphql
websocket: wss://ws.gethealthie.com/subscriptions
sandbox_websocket: wss://ws.staging.gethealthie.com/subscriptions
operations:
  - createWebhook
  - updateWebhook
  - deleteWebhook
  - webhooks
  - noteAddedSubscription
  - conversationChangedSubscription
  - conversationMembershipAddedSubscription
  - conversationMembershipUpdatedSubscription
  - formAnswerGroupModifiedSubscription
  - userUpdatedSubscription
generated: '2026-08-14'
method: generated
source: graphql/healthie-schema.graphql + https://docs.gethealthie.com/guides/webhooks + https://docs.gethealthie.com/guides/websockets-and-subscriptions/getting-started/
---

# Subscribe to Healthie events

Healthie has **two** independent event surfaces. They are not interchangeable and most
integrations need both.

| | Webhooks | GraphQL subscriptions |
|---|---|---|
| Direction | Healthie POSTs to your HTTPS endpoint | You hold a WebSocket open to Healthie |
| Protocol | HTTP + HMAC-SHA256 signature | ActionCable via AnyCable |
| Durability | Retried up to 3 days with backoff | Nothing is replayed |
| Use for | Server-side integrations, back-office sync | Live UI (chat, charting) |
| Coverage | Appointment, FormAnswerGroup, Entry, Note | 6 named subscriptions |

## Part 1 — Webhooks

### Register

`createWebhook` registers an endpoint for an event. Healthie supports different URLs per event
and multiple URLs for the same event. List with `webhooks`, amend with `updateWebhook`, remove
with `deleteWebhook`.

### Expect a thin payload

The body identifies the record; it does not contain it:

```json
{
  "resource_id": "...",
  "resource_id_type": "Appointment | FormAnswerGroup | Entry | Note",
  "event_type": "..."
}
```

You must call back into the GraphQL API with `resource_id` to fetch the record. Budget for that
second call — and for the fact that it counts against the same undocumented request-rate limit.

### Verify the signature before you act

Each delivery carries `Content-Type`, `Content-Digest` (SHA-256 of the payload), `Signature-Input`
and `Signature`. The signature is HMAC-SHA256 over a data string built from the lowercased HTTP
method, path, query string, content digest, content type and content length. Reconstruct it with
your webhook secret and compare. Healthie publishes a JavaScript reference implementation at
https://docs.gethealthie.com/guides/webhooks.

Production deliveries originate from a fixed egress allowlist —
`52.4.158.130`, `3.216.152.234`, `54.243.233.84`, `50.19.211.21` — which you can enforce in
addition to, never instead of, signature verification.

### Handle the retry and disable behaviour

Non-2xx responses are retried with exponential backoff for about **3 days**. Healthie emails you
after roughly 24 hours of failures and **auto-disables the webhook** after about 3 days. An
integration that goes down over a long weekend can come back to a silently disabled subscription
— so poll `webhooks` on a schedule and alert on a disabled endpoint. There is no test-fire tool,
so you cannot trigger a delivery to prove the endpoint works.

Return 2xx fast and process asynchronously.

## Part 2 — GraphQL subscriptions

Six subscriptions exist, verified in the schema:

- `noteAddedSubscription(conversationId)` -> `Note` — the chat/messaging feed
- `conversationChangedSubscription(id)` -> `Conversation`
- `conversationMembershipAddedSubscription(notesType)` -> `ConversationMembership`
- `conversationMembershipUpdatedSubscription(notesType)` -> `ConversationMembership`
- `formAnswerGroupModifiedSubscription(id)` -> `FormAnswerGroup` — charting/intake changes
- `userUpdatedSubscription` -> `UserNotificationsCount`

### Connect

**Do not reach for Apollo's default subscription transport.** Healthie states plainly that the
ActionCable spec differs from the Apollo WebSocket spec and that Apollo's default subscription
connection functionality cannot be used. A translation layer is described as under evaluation and
is not shipped.

Sequence:

1. Open `wss://ws.gethealthie.com/subscriptions?token=<API_KEY>`. The WebSocket handshake has no
   header phase, so the key travels as a query parameter — treat that URL as a secret, and keep it
   out of logs, browser history and error reports.
2. Wait for `{"type":"welcome","sid":"..."}` and expect periodic `{"type":"ping","message":<epoch>}`.
3. Generate your own unique channel ID client-side.
4. Send `{"command":"subscribe","identifier":"{\"channel\":\"GraphqlChannel\",\"channelId\":\"<ID>\"}"}`.
5. Send `{"command":"message","identifier":"<same identifier>","data":"<stringified subscription document>"}`.

Results arrive as `{"identifier":"...","message":{"more":true,"result":{"data":{...}}}}`.

## Rules an agent must follow

**Subscriptions are not durable.** Nothing is replayed on reconnect. Anything that must not be
missed belongs on a webhook, which retries for 3 days — or on a reconciliation query after
reconnect.

**Idempotency is yours.** Healthie makes no delivery guarantee beyond retry, and it has no
idempotency contract on the write side either. Key your handlers on `resource_id` +
`event_type` and make them safe to run twice.

**No `changed_fields` on the wire.** The published payload carries only `resource_id`,
`resource_id_type` and `event_type`; diff against your own last-known state.

**Watch the event catalogue grow.** New webhook resource types ship by accretion — the 2026-07-24
changelog added `DOCUMENT_SHARING` to `SentWebhookResourceType`. Treat unknown `event_type`
values as ignorable rather than fatal.

## Related

- Webhooks AsyncAPI: `asyncapi/healthie-webhooks-asyncapi.yml`
- Subscriptions AsyncAPI: `asyncapi/healthie-subscriptions-asyncapi.yml`
- Authentication: `authentication/healthie-authentication.yml`
- Conventions: `conventions/healthie-conventions.yml`
