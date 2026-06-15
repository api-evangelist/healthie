# Healthie GraphQL API

The Healthie GraphQL API is the single contract behind the entire Healthie platform — the same API that powers the Healthie web, iOS, and Android applications is available to partners building branded care experiences. The schema exposes clients, appointments, availability, charting notes, custom forms and form-answer groups, care plans, goals, allergies, immunizations, medications, diagnoses, lab orders, CMS-1500 claims, insurance authorizations and eligibility checks, payments, online programs and courses, conversations, direct messaging, faxing, and announcements. Authentication is API-key based via `Authorization: Basic` + `AuthorizationSource: API` headers, with optional `AuthorizationShard` for sharded customers. Versioning is opt-in by date via the `Healthie-GraphQL-API-Version` header (current default version `2024-06-01`). Rate limits combine a dynamic per-account request rate with a per-query complexity score (max 2000) and depth limit (max 25). Sandbox is fully isolated at `https://staging-api.gethealthie.com/graphql`; production runs at `https://api.gethealthie.com/graphql`.

**Endpoint:** https://api.gethealthie.com/graphql

**Documentation:** https://docs.gethealthie.com/

**References:**

- Documentation: https://docs.gethealthie.com/
- Documentation: https://docs.gethealthie.com/reference
- Documentation: https://docs.gethealthie.com/guides/quickstart
