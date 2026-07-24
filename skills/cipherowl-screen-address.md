---
name: Screen a blockchain address at the gate
description: Run a low-latency yes/no compliance screen on a single or batch of blockchain addresses before allowing a deposit, withdrawal, or counterparty.
api: openapi/cipherowl-openapi.json
operations: [ScreenService_ScreenAddress, ScreenService_BatchScreenAddresses, RiskReasonService_GetChainCapabilities]
---

# Screen a blockchain address at the gate

Use CipherOwl's Screening API for a real-time risk verdict (`foundRisk`) you can put
inline on a deposit, withdrawal, or counterparty check.

## Prerequisites
- OAuth2 client-credentials token (see `cipherowl-authenticate.md`). Send
  `Authorization: Bearer <token>` and `Accept: application/json` on every request.
- Base URL `https://svc.cipherowl.ai`.

## Steps
1. (Once) Call `RiskReasonService_GetChainCapabilities`
   (`GET /api/chain-capabilities/v1`) to learn which chains expose `screening` /
   `reporting` tiers. Use the returned `chainClass` (e.g. `ethereum` -> `evm`) as the
   `chain` path parameter.
2. Screen one address with `ScreenService_ScreenAddress`
   (`GET /api/screen/v1/chains/{chain}/addresses/{address}?config=co-vasp`). Pass a
   `config` (`co-defi`, `co-vasp`, `co-institution`, or `co-sandbox` for free testing);
   omit it to use your org default (the resolved name is echoed on the response).
3. For lists, use `ScreenService_BatchScreenAddresses`
   (`POST /api/screen/v1/chains/{chain}/batch`) with `{config, addresses[]}`; each
   address gets an independent verdict.
4. Read `foundRisk`. If `true`, do not auto-approve - escalate to the investigation
   skill for score, breakdown, and evidence.

## Rules
- On `401`, refresh the token and retry once. On `429`, back off honoring `Retry-After`.
- Errors use the gRPC `rpcStatus` envelope (`code`, `message`, `details`); see
  `errors/cipherowl-problem-types.yml`.
- Test for free with `?config=co-sandbox` against the addresses in
  `sandbox/cipherowl-sandbox.yml`.
