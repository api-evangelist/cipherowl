---
name: Manage private-data address overrides
description: Maintain your organization's allow/deny/label overrides on blockchain addresses that customize CipherOwl screening for your risk configuration.
api: openapi/cipherowl-openapi.json
operations: [PrivateDataService_ListEntriesForChain, PrivateDataService_UpsertEntry, PrivateDataService_GetEntry, PrivateDataService_DeleteEntry]
---

# Manage private-data address overrides

Curate org-scoped overrides (labels / list membership) that tune how CipherOwl screens
specific addresses. Auth and headers as in `cipherowl-authenticate.md`; base URL
`https://svc.cipherowl.ai`.

## Steps
1. **List** current overrides on a chain with `PrivateDataService_ListEntriesForChain`
   (`GET /api/private-data/v1/overrides/chains/{chain}/addresses`); page with the
   returned `nextPageToken`.
2. **Upsert** an entry with `PrivateDataService_UpsertEntry`
   (`POST /api/private-data/v1/overrides/chains/{chain}/addresses`) with
   `{listType, address, addLabels[], removeLabels[], notes, expectedVersion}`. Pass
   `expectedVersion` for optimistic concurrency; the response returns the new `entry`
   and `created` flag.
3. **Get** a single entry with `PrivateDataService_GetEntry`
   (`GET /.../addresses/{address}`) to read its current `version` before editing.
4. **Delete** with `PrivateDataService_DeleteEntry` (`DELETE /.../addresses/{address}`).

## Rules
- Overrides are scoped to your organization (determined by the API key/token).
- Use `version` / `expectedVersion` to avoid clobbering concurrent edits.
- `401` -> refresh+retry once; `429` -> back off honoring `Retry-After`. Errors use the
  `rpcStatus` envelope.
