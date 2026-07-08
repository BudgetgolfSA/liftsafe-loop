# LiftSafe Session Handoff
Last updated: 2026-07-08T20:30:00+02:00
Branch: staging

## Protocol
- §17 handoff: mandatory after every task
- §18 Grok Slack post: mandatory before reporting to Piet

## ONLY PRIORITY
**LS-045 prod ship — DONE** (Piet "ship it" 2026-07-08). Proof: `logs/20260708-ls045-prod-ship.md`

| Item | Prod status |
|------|-------------|
| All stuck SUBMITTED+pending_signature sessions | **0** (round 2 fixed JC-0011/0012) |
| `lib/openSession.ts` | Excludes submitted job cards from CONTINUE OPEN JOB banner |
| Mobile AI prompt | `FIELD_MOBILE_LMI_PROMPT` — ≤150 words, standard→scope→checks |
| `submit_job_card_v2` RPC recurrence fix | `signed_fix_present` on prod + staging |
| `liftsafe-brain` edge (mime_type magic-byte) | deployed prod |
| Web Standards Assistant bubble (`/api/ai/assistant`) | Vercel prod live |

**LS-041 web prod** still on `72eb8858` — promotion to `b0650ce2` unchanged (Piet gate).

**LS-044 SANS gap report** — `docs/LS-044-sans-forms-compliance-gap-report.md` (audit only, no form edits). Piet decision on standard mapping before any form work.

## Mobile preview OTA (live)
Update group `81fc33a9-a46d-4fd0-9e56-76d0fdadfefd` — LS-045 scan/search + open session fix + AI mobile prompt

## Piet verify on device (pull OTA / restart app first)
1. Job Orders READY — no CONTINUE OPEN JOB for submitted Mvoti cards
2. SCAN/SEARCH → Northam PROV serial → dossier with START INSPECTION
3. AI Assistant — overhead crane question → short answer (not standards dump)
4. Web `admin.liftsafe.co.za` — AI bubble works (Vercel prod shipped)

## Next Action
1. Piet device smoke (items above)
2. LS-041 web prod promotion when ready (`72eb8858` → `b0650ce2`)
3. LS-044 standard-mapping gate before form edits

## Handoff URL (§17)
https://budgetgolfsa.github.io/liftsafe-loop/session-handoff.md