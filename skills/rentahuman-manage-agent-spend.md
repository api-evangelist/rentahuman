---
name: Monitor and control agent wallet spend
description: >-
  Pull a wallet spend report over a time range, read the current wallet
  controls, and update spending caps / auto-topup so an autonomous agent stays
  within budget.
api: openapi/rentahuman-openapi-original.yml
base_url: https://rentahuman.ai/api
operations: [getWalletReport, getWalletControls, updateWalletControls]
auth: API key via header `X-API-Key: rah_live_...` (or Firebase session)
---

# Manage agent wallet spend

Use this skill to give an autonomous agent guardrails on money it spends hiring
humans, and to reconcile what it has already spent.

## Steps

1. **Report spend** — `getWalletReport` (`GET /wallet/report`). Optional `start`
   and `end` (ISO 8601 or epoch-ms; default last 30 days). Read `report.totals`
   (`paid`, `inEscrow`, `pendingRelease`, `settled`, `refunded` — all in cents)
   plus `byBounty[]` and `byHuman[]` breakdowns.
2. **Read controls** — `getWalletControls` (`GET /wallet/controls`). Returns the
   current alert threshold, spending caps, and auto-topup config.
3. **Update controls** — `updateWalletControls` (`PATCH /wallet/controls`). Send
   only the fields to change; pass `null` to clear a threshold or cap. All cent
   amounts are non-negative integers up to `10000000` ($100k). When
   `autoTopupEnabled` is `true`, you must satisfy
   `autoTopupTargetCents > autoTopupFloorCents > 0` and
   `autoTopupMaxPerDayCents >= target - floor`, else you get a `400` with
   `code: validation`.

## Guardrail fields
- `lowBalanceThresholdCents` — email alert trigger.
- `spendingCapPerBountyCents` — cap per bounty.
- `spendingCapRolling24hCents` — cap on worker payouts per rolling 24h.
- `autoTopupFloorCents` / `autoTopupTargetCents` / `autoTopupMaxPerDayCents`.

## Error handling
Errors return `{ "success": false, "error": "...", "code": "validation" }`.
See `errors/rentahuman-problem-types.yml` and `conventions/rentahuman-conventions.yml`.
