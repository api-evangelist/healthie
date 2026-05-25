# Healthie (healthie)

Healthie is a cloud, API-first EHR and practice management platform purpose-built for digital health startups, virtual care companies, and modern clinical practices. Healthie packages an EHR, scheduling, charting, telehealth, intake forms, care plans, online programs, messaging, billing, claims (CMS-1500 / ClaimMD), and patient portal behind a single GraphQL API and modular SDKs that power both the Healthie product UI and partner applications. The same API that backs the Healthie web and mobile apps is fully exposed to customers building branded patient and provider experiences on top of the platform.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/healthie/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Kind:** contract
- **Position:** Consumer
- **Access:** 3rd-Party

## Tags

API-First, Appointments, Billing, Care Plans, Charting, Claims, Clinical, Digital Health, EHR, EMR, Forms, GraphQL, Health Tech, Healthcare, Insurance, Intake, Online Programs, Patient Engagement, Patient Portal, Practice Management, Programs, Scheduling, Telehealth, Wellness, Webhooks

## Timestamps

- **Created:** 2026-05-25
- **Modified:** 2026-05-25

## APIs

### Healthie GraphQL API

The Healthie GraphQL API is the single contract behind the entire Healthie platform — the same API that powers the Healthie web, iOS, and Android applications is available to partners building branded care experiences. The schema exposes clients, appointments, availability, charting notes, custom forms and form-answer groups, care plans, goals, allergies, immunizations, medications, diagnoses, lab orders, CMS-1500 claims, insurance authorizations and eligibility checks, payments, online programs and courses, conversations, direct messaging, faxing, and announcements. Authentication is API-key based via `Authorization: Basic` + `AuthorizationSource: API` headers, with optional `AuthorizationShard` for sharded customers. Versioning is opt-in by date via the `Healthie-GraphQL-API-Version` header (current default version `2024-06-01`). Sandbox is fully isolated at `https://staging-api.gethealthie.com/graphql`; production runs at `https://api.gethealthie.com/graphql`.

**Human URL:** [https://docs.gethealthie.com/](https://docs.gethealthie.com/)

**Base URL:** `https://api.gethealthie.com/graphql`

#### Tags

Appointments, Billing, Charting, Claims, Clients, EHR, Forms, GraphQL, Healthcare, Insurance, Patients, Practice Management, Programs, Scheduling, Telehealth, Webhooks

#### Properties

- [Documentation](https://docs.gethealthie.com/)
- [GraphQL Schema Reference](https://docs.gethealthie.com/reference)
- [Quickstart](https://docs.gethealthie.com/guides/quickstart)
- [Authentication](https://docs.gethealthie.com/guides/api-concepts/authentication)
- [Environments](https://docs.gethealthie.com/guides/api-concepts/environments)
- [Versioning](https://docs.gethealthie.com/guides/api-concepts/versioning)
- [Rate Limits](https://docs.gethealthie.com/guides/api-concepts/rate-limits)
- [Webhooks](https://docs.gethealthie.com/guides/webhooks)
- [Staging Sandbox](https://staging-api.gethealthie.com/graphql)
- [GraphiQL](https://docs.gethealthie.com/graphiql)

## Common Properties

- [Website](https://www.gethealthie.com/)
- [API & Platform Portal](https://www.gethealthie.com/api)
- [Developer Docs](https://docs.gethealthie.com/)
- [Quickstart](https://docs.gethealthie.com/guides/quickstart)
- [Authentication](https://docs.gethealthie.com/guides/api-concepts/authentication)
- [Versioning](https://docs.gethealthie.com/guides/api-concepts/versioning)
- [Rate Limits](https://docs.gethealthie.com/guides/api-concepts/rate-limits)
- [Webhooks](https://docs.gethealthie.com/guides/webhooks)
- [Environments](https://docs.gethealthie.com/guides/api-concepts/environments)
- [@healthie/sdk on npm](https://www.npmjs.com/org/healthie)
- [Healthie Dev Assist (MCP)](https://github.com/healthie/healthie-dev-assist)
- [Sample Booking Widget](https://github.com/healthie/healthie_sample_booking_widget)
- [GitHub Organization](https://github.com/healthie)
- [LinkedIn](https://www.linkedin.com/company/gethealthie/)
- [X / Twitter](https://x.com/gethealthie)
- [Blog](https://www.gethealthie.com/blog)
- [Status Page](https://status.gethealthie.com/)
- [Pricing](https://www.gethealthie.com/pricing)

## Plans

- **Core** — $19/mo ($18/mo annual). 10 active clients, scheduling, payment processing, charting, telehealth, client portal, mobile app.
- **Essentials** — $49/mo ($45/mo annual). 250 active clients, free outbound fax, custom forms, CMS-1500 claims, group messaging, ClaimMD.
- **Plus** — $129/mo ($115/mo annual). Unlimited clients, group telehealth, online programs, dedicated eFax.
- **Group** — $149+/mo ($135+/mo annual). Shared calendar, team management, roles & permissions, internal chat; +$50/mo per additional clinician.
- **API & Enterprise** — Call for pricing. Full GraphQL access, modular SDKs, webhooks, sandbox, sub-organizations, sharded data residency.

## Features

- GraphQL API powering Healthie's own EHR, mobile, and patient portal — same contract for first-party and partners
- Resources span clients, appointments, availability, charting, forms, programs, goals, allergies, immunizations, medications, lab orders, CMS-1500 claims, insurance authorizations, eligibility checks, payments, messaging, fax, conversations, and announcements
- Date-based opt-in API versioning via the `Healthie-GraphQL-API-Version` header — default `2024-06-01`
- API-key authentication via `Authorization: Basic` + `AuthorizationSource: API` (plus `AuthorizationShard` for sharded customers)
- `createApiKey` GraphQL mutation for programmatic key issuance scoped to individual user accounts
- Two-environment model — `staging-api.gethealthie.com` sandbox and `api.gethealthie.com` production with no data transfer between them
- Query complexity scoring (max 2000) and 25-level depth limit instead of fixed RPS rate limits
- HMAC-SHA256 signed webhooks with exponential-backoff retry up to 3 days, auto-disable, and email alerts
- Webhook `resource_id_type` covers Appointment, FormAnswerGroup, Entry, and Note events with `changed_fields` deltas
- Parent-organization webhooks aggregate events across sub-organizations
- Modular TypeScript / JavaScript SDKs on npm under `@healthie/sdk`
- Healthie Dev Assist — official MCP server for AI-assisted Healthie development
- Sample Booking Widget reference implementation on GitHub
- Healthie Harbor app marketplace (Google Fit, Apple Health, Fitbit, Stripe, Zoom, ClaimMD, Fullscript Labs, Change Healthcare)
- HIPAA and PCI compliant infrastructure operating ~2.5B API calls/month at 99.99% uptime
- US-based data residency options for enterprise

## Maintainers

**FN:** Kin Lane

**Email:** info@apievangelist.com

**X:** [apievangelist](https://x.com/apievangelist)

**URL:** [https://apievangelist.com](https://apievangelist.com)
