# LiftSafe Session Handoff
Last updated: 2026-07-08T22:00:00+02:00
Branch: staging @ `1c509370`

## ONLY PRIORITY
**LS-045 + brain repo grounding — PROD SHIP COMPLETE** (Piet "ship it" ×2). Proof: `logs/20260708-ls045-prod-ship-final.md`

| Surface | Prod status |
|---------|-------------|
| DB stuck sessions | **0** (migrations 140000 + 150000 applied) |
| `submit_job_card_v2` RPC | `signed_fix_present` |
| Brain edge | mime_type + mobile prompt + **repo SANS grounding** live |
| Web assistant bubble | Vercel prod · smoke **200** |
| Mobile JS | **preview OTA `2cdfd208` only** — production channel NOT pushed (device verify gate) |

**LS-041 web prod** — still `72eb8858`; promote to `b0650ce2` when Piet ready.

**LS-044** — gap report only; no form edits until mapping gate.

## Piet device test (preview OTA)
1. Restart app → Job Orders READY — no stale CONTINUE OPEN JOB
2. SCAN/SEARCH → Northam PROV serial → dossier + START INSPECTION
3. AI Assistant → overhead crane → short corpus-cited answer (SANS 10375)

## Next
1. Piet preview smoke → then say go for `eas update --branch production`
2. LS-041 web prod promotion
3. LS-044 mapping gate

## Handoff URL
https://budgetgolfsa.github.io/liftsafe-loop/session-handoff.md