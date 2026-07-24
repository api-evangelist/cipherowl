---
name: Investigate a risky address and generate a report
description: Drill down from a risky screen verdict through risk score, category breakdown, and path-level evidence, then generate an analyst-ready risk assessment or draft SAR.
api: openapi/cipherowl-openapi.json
operations: [RiskReasonService_GetAddressRiskScore, RiskReasonService_GetAddressRiskBreakdown2, RiskReasonService_GetAddressRiskReasonDetail, risk_assessment_api, graph_api, sar_report_api]
---

# Investigate a risky address and generate a report

When `foundRisk` is `true`, walk the SRR funnel from cheap triage to full evidence and
a shareable report. Auth and headers as in `cipherowl-authenticate.md`; base URL
`https://svc.cipherowl.ai`; keep the same `config` (e.g. `co-institution` or
`co-sandbox`) across the funnel.

## Steps
1. **Score** - `RiskReasonService_GetAddressRiskScore`
   (`GET /api/reason/v2/chains/{chain}/addresses/{address}/risk-score`) returns a
   deterministic 0-100 `riskScore` + `riskBand` for threshold rules.
2. **Breakdown** - `RiskReasonService_GetAddressRiskBreakdown2`
   (`.../risk-breakdown`) returns matched risk categories and direct-vs-indirect
   exposure for an at-a-glance profile.
3. **Detail** - `RiskReasonService_GetAddressRiskReasonDetail` (`.../detail`) returns
   per-category hops, USD/percentage exposure, first/last timestamps, and the actual
   transaction `paths` (including cross-chain) - the evidence behind the verdict.
4. **Report** - generate a shareable artifact:
   - `risk_assessment_api`
     (`GET /api/report/v1/risk-assessment/chains/{chain}/addresses/{address}`) - a
     human-readable narrative assessment.
   - `sar_report_api` (`.../report/v1/sar/...`) - a draft Suspicious Activity Report
     narrative.
   - `graph_api` (`.../report/v1/graph/...`) - a risk-flow diagram (Mermaid / HTML / PNG).

## Rules
- The Report API strictly requires the `evm` chain key (not `ethereum`).
- Report endpoints echo the applied config in the `cipherowl-resolved-config` response
  header.
- `401` -> refresh+retry once; `429` -> back off honoring `Retry-After`.
