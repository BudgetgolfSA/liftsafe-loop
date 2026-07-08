# LiftSafe Session Handoff
Last updated: 2026-07-08T18:00:00+02:00
Branch: staging

## Protocol
- §17 handoff: mandatory after every task
- §18 Grok Slack post: mandatory before reporting to Piet

## ONLY PRIORITY
**LS-045 prod ship — DONE** (Piet "ship it" 2026-07-08). Proof: `logs/20260708-ls045-prod-ship.md`

| Item | Prod status |
|------|-------------|
| Stuck Mvoti sessions (`289410db…`, `0a5af4c7…`) | `signed`, `pending_left: 0` |
| `submit_job_card_v2` RPC recurrence fix | `signed_fix_present` on prod + staging |
| `liftsafe-brain` edge (mime_type magic-byte) | deployed prod |
| Web Standards Assistant bubble (`/api/ai/assistant`) | Vercel prod live |

**LS-041 web prod** still on `72eb8858` — promotion to `b0650ce2` unchanged (Piet gate).

**LS-044 SANS gap report** — `docs/LS-044-sans-forms-compliance-gap-report.md` (audit only, no form edits). Piet decision on standard mapping before any form work.

## Mobile (staging / preview OTA)
- LS-045 SCAN/SEARCH equipment register + dossier + callout preselect
- LS-045-D `mime_type` fix for AI photo queries
- LS-045 job-card UI overhaul (home banner → Job Orders READY tab) — on staging, verify on preview device

## Piet verify on device
1. Hard refresh `admin.liftsafe.co.za` → floating AI bubble on non-dashboard pages
2. Preview OTA → Equipment SCAN/SEARCH + AI photo query (PNG)
3. Mvoti home — OPEN JOB banners for JC-2026-0013/0014 should be cleared

## Next Action
1. Piet device smoke (items above)
2. LS-041 web prod promotion when ready (`72eb8858` → `b0650ce2`)
3. LS-044 standard-mapping gate before form edits

## Handoff URL (§17)
https://budgetgolfsa.github.io/liftsafe-loop/session-handoff.md