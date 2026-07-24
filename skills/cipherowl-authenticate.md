---
name: Authenticate to the CipherOwl API
description: Exchange client_id/client_secret for an OAuth2 client-credentials Bearer token and build a client that caches it, refreshes on 401, and backs off on 429.
api: openapi/cipherowl-openapi.json
operations: [ScreenService_ScreenAddress]
---

# Authenticate to the CipherOwl API

Every CipherOwl API shares one auth model: OAuth 2.0 client-credentials -> Bearer JWT.

## Steps
1. Get `client_id` / `client_secret` from onboarding at https://app.cipherowl.ai.
2. POST to `https://svc.cipherowl.ai/oauth/token` with
   `{ "client_id": ..., "client_secret": ..., "audience": "svc.cipherowl.ai", "grant_type": "client_credentials" }`.
   The response has `access_token`, `scope`, `token_type: Bearer`, and `expires_in`
   (default 86400s).
3. Send `Authorization: Bearer <access_token>` and `Accept: application/json` on every
   request (e.g. a first `ScreenService_ScreenAddress` call to verify).
4. Cache the token and its `expires_in`; refresh ~60s early.

## Rules
- On `401 Unauthorized`, discard the token, fetch a new one, retry once.
- On `429 Too Many Requests`, wait and retry honoring `Retry-After`; else exponential
  backoff. Tokens and the shared gateway are rate-limited.
- Never paste credentials into public repos. See
  `authentication/cipherowl-authentication.yml` and
  `conventions/cipherowl-conventions.yml`.
