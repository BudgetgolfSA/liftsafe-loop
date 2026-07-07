# LiftSafe Session Handoff
Last updated: 2026-07-07T20:15:00+02:00
Branch: staging

## Protocol
- §17 handoff: mandatory after every task
- §18 Grok Slack post: mandatory before reporting to Piet

## ⚠️ PROTOCOL CONFLICT — needs Piet's decision, not silently re-fixed again
`CLAUDE.md`'s "end session" trigger (step 2: `node scripts/session-end.mjs`) **overwrites this file
wholesale** with a terse auto-generated `git diff` summary. `AGENTS.md` §17 (added today,
2026-07-07) treats this same file as the persistent, detailed multi-agent handoff that Codex, Grok,
and Claude Chat all read for context — "stale handoff = wrong instructions = rework." These two
protocols directly conflict on the same file. It has now happened twice this session: once earlier
(caught and reported, not restored), and once for real just now — Grok ran `session-end.mjs`
following CLAUDE.md's own instruction, wiped this file down to its generic template, and the detailed
per-task history below had to be manually reconstructed from conversation context. Next time this
happens the content may not be recoverable. Needs a real fix: either `session-end.mjs` stops
overwriting `docs/session-handoff.md` (writes its generic repo-state summary somewhere else, e.g.
`docs/session-end-log.md`), or §17's content moves to a filename `session-end.mjs` doesn't touch.

## ONLY PRIORITY
**LS-041 prod smoke test still BLOCKED (not FAIL): prod web has never been deployed with the LS-041 UI.** DB schema + edge function are already on prod (verified read-only both by Claude Code earlier and re-verified by Grok). The commit+undo/`created_ids` mechanism can't be exercised on https://admin.liftsafe.co.za until `web/` is promoted to production — a GATE requiring Piet's explicit go-ahead (`cd web && vercel --prod`). Full detail: `logs/20260707-prod-import-smoke-test.md`.

## Agent Status (2026-07-07)
| Agent | Task | Status | Handoff |
|---|---|---|---|
| Grok | LS-041 extraction layer | COMPLETE | pushed |
| Grok | Prod smoke cleanup re-verify + repo junk-file cleanup | COMPLETE | pushed |
| Claude Code | LS-041 UI + full-build verification | COMPLETE | pushed |
| Claude Code | LS-041 prod smoke test | BLOCKED — prod web not deployed | pushed |
| Claude Code | Import hierarchy right panel (item visibility) | COMPLETE | pushed |
| Claude Code | LS-040 LME branding — sidebar + client portal | COMPLETE | pushed |
| Claude Code | extract-document: tonnage-line section-header bug | COMPLETE staging edge | pushed |
| Claude Code | Delete button on draft imports | COMPLETE | pushed |
| Claude Code | Original-document viewer + item-label truncation fix | COMPLETE | pushed |
| Claude Code | Import hierarchy layout: sidebar/zoom/3-col width | COMPLETE | pushed |
| Codex | LS-042 navigation | COMPLETE | pushed |
| Codex | LS-044 safety harness UI | IN PROGRESS | pushed |

## Session-end (Claude Code, 2026-07-07T20:15)
- Working tree clean of all Claude Code work: committed + pushed to `origin/staging`
- TSC: root `app/` + `web/` both **0 errors, PASS**
- OTA published to `preview` branch (env=production, matches CLAUDE.md "session-end auto-pushes to preview"): update group `c0b8feef-155d-473b-93ed-3a830e84b7f4`. No mobile (`app/`/`lib/`/`constants/`) code was touched this session by Claude Code — this republishes the current already-committed state (includes Codex's LS-044 UI-only work, safe: no DB/submission wiring per its own gate)
- 2 files intentionally left uncommitted: `.claude/settings.json`, `mcps/chrome-devtools/tools/evaluate_script.json` — Piet's own local config changes, not mine to sweep into a commit (classifier correctly blocked me from doing so)
- **This file was reconstructed** after `session-end.mjs` overwrote it (see protocol conflict note above) — content below rebuilt from this session's actual work, cross-checked against `git log`

## Grok — prod smoke cleanup re-verify + repo cleanup (2026-07-07)
- Re-verified read-only: all `TEST_IMPORT` + throwaway prod-smoke tenant counts are **0** (`logs/prod-smoke-cleanup-verify.json`) — confirms the cleanup Claude Code flagged as pending earlier in the day was actually executed and is holding
- Removed an accidental repo-root junk file (`--on-error-stop`)
- Noted `git checkout` is now blocked by `.claude/settings.json` policy, confirmed HEAD was already correct on `staging`

## Import hierarchy layout fixes (Claude Code) — COMPLETE staging, UI only
- Commit: `03dc646e` (`feat(web): import hierarchy step - hide sidebar, PDF zoom/resize, full-width 3-col layout`)
- `AdminLayout` gained `hideSideNav` prop, set with `fullWidth` only for `step==='hierarchy'` — sidebar + 7xl cap both restore automatically for every other step
- Document viewer: zoom in/out (0.25 steps, 50-300%) via CSS transform scale, native `resize:both` handle, larger default height (75vh)
- 3-column grid: `minmax(320px,1fr) minmax(340px,1fr) minmax(340px,1fr)` + `min-width:1040px` inside `overflow-x-auto` — columns expand to fill available width, wrapper scrolls instead of any column compressing below its floor
- Staging-verified: sidebar confirmed absent on hierarchy step (present on `/dashboard` baseline), zoom to 150% confirmed via computed transform, `resize:both` confirmed via computed style, grid columns computed at ~372px each (expanding past their 320/340/340 minimums) at a 1264px test viewport
- TSC: root + `web/` both 0 errors. `npm run build` PASS
- Proof: `logs/20260707-import-layout-fixes.md` + screenshot `logs/20260707-import-layout-fixes.png` (local, gitignored)

## Original-document viewer + truncation fix (Claude Code) — COMPLETE staging, UI only
- Commit: `3f1fec1c` (`feat(web): original-document preview panel + fix item-name truncation`)
- Premise checked first: `import_session.source_url` / `raw_extraction.source` don't exist — `extract-document` deliberately never persists the uploaded file (POPIA, stated in its own code comment). Built a browser-local `blob:` object-URL preview instead (from the `File` object `onFile()` already has) — works for the *current* upload only, correctly absent when resuming a saved draft (nothing to show there, by design, not a bug)
- Hierarchy step grid becomes 3 columns when a viewable PDF/image is present, else falls back to the existing 2-column layout (superseded in width/behavior by the layout-fixes pass above)
- Also fixed: both item-label elements in the right panel used Tailwind `truncate` (hover-title only) — swapped for `break-words` so full `asset_description` always renders
- TSC: root + `web/` both 0 errors. `npm run build` PASS
- Proof: `logs/20260707-import-doc-viewer-and-truncation.md` + screenshot `logs/20260707-import-doc-viewer.png` (local, gitignored)

## Delete draft import (Claude Code) — COMPLETE staging, UI only
- Commit: `fc6d28ef` (`feat(web): add delete button to draft imports in Continue Incomplete Import list`)
- Premise checked first: no delete RPC/RLS policy exists for `import_sessions` (SELECT/INSERT/UPDATE only) — the existing UPDATE policy + `status='rolled_back'` (already a valid CHECK-constraint value) is the real mechanism, since `fetchDraftImportSessions()` only returns `status='draft'` rows. True soft-delete, no new DB object.
- DELETE button added next to each draft, native `confirm()` matching the existing "UNDO THIS IMPORT" pattern in the same file
- Staging-verified: cancel path leaves draft intact; confirm path removes it immediately and it stays gone after a full page reload; read-only DB check confirms the row still exists with `status=rolled_back` (nothing hard-deleted)
- TSC: root + `web/` both 0 errors. `npm run build` PASS
- Proof: `logs/20260707-import-draft-delete.md` + screenshot `logs/20260707-delete-draft-import.png` (local, gitignored)

## extract-document tonnage-line fix (Claude Code) — COMPLETE staging edge only
- Commit: `04999c49` (`fix(edge): equipment lines with inline tonnage no longer misread as section headers`)
- Bug: lines like "Overhead crane 15Ton", "Hoist 5Ton" (equipment type + trailing tonnage, no serial) were discarded as fake section separators by `isSectionHeaderRow`'s `SECTION_HINT_RE` (matches any label starting with an equipment-type word, regardless of what follows) — same bug existed in the AI prompt's own judgment
- Fix (`supabase/functions/_shared/extract-parse.ts`): `extractInlineTonnage()` short-circuits `isSectionHeaderRow` to false when a line carries its own tonnage; `cleanRowList` backfills `swl_tonnes`/`equipment_type` from the label; `LARGE_REGISTER_RULES` (the actual AI-facing prompt) now explicitly distinguishes bare category headers from `[Type][Tonnage]` equipment lines
- **Row count: 0 → 66** on a 66-row/7-section Northam-style test fixture (`scripts/ls-extract-northam-rowcount.mjs`, direct edge-function call, no UI). Section-separator false positives: 66 → 0. Bare headers (e.g. "OVERHEAD CRANES") still correctly used as section names, not rows.
- Regression-checked: existing staging fixtures (CSV + register snippet) still 12/12 PASS after the fix
- Deployed staging only (scp + `docker restart supabase-edge-functions`, SHA256-verified identical) — prod has its own separately-shipped copy of this file (from an earlier, unrelated "ship it") and would need its own explicit go-ahead to update
- **Judgment call, not implemented**: the ask also listed `serial: PROV-[padded index] if no serial present`. Left `serial_number: null` instead — the import UI already has its own opt-in provisional-serial mechanism (`allowProvisionalIds` checkbox), and having the edge function pre-fill `PROV-XXX` would silently bypass that consent gate. Flagged for Piet to confirm intent.
- No migrations, no UI touched. `supabase/functions/**` is excluded from the root tsconfig (Deno runtime) — TSC pass is N/A for this file; correctness validated via the live before/after test instead.

## LS-040 LME branding (Claude Code) — COMPLETE staging, UI only
- Commit: `8a00612b` (`feat(web): LS-040 LME branding on dashboard sidebar + client portal`)
- Audited first: dashboard header (`InspectorTopBar.tsx`), CoF PDFs, and other mobile-generated docs already pulled per-LME logo + `Powered by LiftSafe` watermark from `lme_branding`/`lme_companies.logo_url` — not touched, already correct
- Fixed 2 real gaps: `SideNav.tsx` (hardcoded "LIFTSAFE" wordmark → live `lme_branding` fetch, logo or trading-name fallback, watermark added) and `web/app/client/layout.tsx` (hardcoded `/liftsafe-logo.jpeg` → resolves servicing LME via `client_lme_assignments` + `lme_companies.logo_url` through the server-only admin client — same pattern already live in `client/equipment/[id]/actions.ts` — watermark added)
- No migrations, no new RLS — `lme_branding` table already existed from a prior session
- TSC: root `app/` + `web/` both 0 errors. `cd web && npm run build` PASS
- Proof: `logs/20260707-ls040-lme-branding-ui.md` + screenshot `logs/20260707-ls040-sidebar-branding.png` (local, gitignored)

## Import hierarchy right panel (Claude Code) — COMPLETE staging, UI only
- Commit: `ebd38d4c` (`feat(web): add right panel to import hierarchy builder for per-item equipment visibility`)
- `web/app/import/page.tsx`: right panel added to the hierarchy step — itemized UNASSIGNED list with per-item assign dropdown, items grouped/listed under their section once assigned (with reassign/unassign dropdown), unassigned-count banner at the top of the step (`ALL N ITEMS PLACED` / `X OF N ITEMS UNASSIGNED`)
- Section item counts were already live off `section.rows.length` — new per-item moves (`moveItem()`) write to the same `hierarchyDraft` state so both panels + the left tree stay in sync from one source of truth
- No DB/migrations/edge functions touched — pure UI
- TSC: root `app/` + `web/` both 0 errors. `cd web && npm run build` PASS
- Proof: `logs/20260707-import-hierarchy-right-panel.md` + screenshot `logs/20260707-import-hierarchy-right-panel.png` (local, gitignored)

## LS-041 Extraction (Grok) — COMPLETE staging edge
- `154.65.98.90`: v2 `intakePrompt` + full foreman live
- `test-extract-parse.mjs` 4/4 PASS
- Prod edge: deployed via a properly Piet-authorized "ship it" done concurrently in a parallel session, documented in `logs/20260707-prod-ls041-ship.md` / `-verify.md`

## LS-041 UI + full build (Claude Code)
- Committed staging: `3da33cc`, `import-session.ts` wired
- Full build audit: `logs/20260707-ls041-fullbuild-verification.md` — all UI pieces confirmed in code (client confirm, hierarchy tree, inline create/edit/rename, unassigned panel, draft resume, atomic commit + undo), no TODO/stub markers
- Smoke test (staging): PASS 12/12 — `logs/20260707-ls041-fullbuild-smoke-test.log`
- Import for Piet: UNBLOCKED (staging)

## LS-041 prod smoke test (Claude Code) — BLOCKED, full detail in `logs/20260707-prod-import-smoke-test.md`
- Prod DB schema (`import_sessions`, `deleted_at` cols, `bulk_create_assets` returning `created_ids`) — confirmed present, read-only
- Prod `extract-document` edge function — confirmed present, byte-identical to staging
- Prod web app has not been deployed since before LS-041 UI landed — the live `/import` page still runs the pre-LS-041 flat-CSV flow (no `import_sessions` row, no undo button)
- **Cleanup COMPLETE (2026-07-07, re-verified by Grok)**: the throwaway `prod-smoke+...@mvotiweights.co.za` tenant, all `TEST_IMPORT` asset rows, and the client created during earlier testing were all removed via `scripts/prod-import-smoke-cleanup.mjs`. Re-verified read-only, all counts 0.
- **Outstanding**: to actually test "commit and undo with `created_ids`", prod web needs a deploy (`cd web && vercel --prod`) — GATE, needs Piet's explicit go-ahead. A fresh throwaway tenant will need to be created on retry.

## Handoff URL (§17)
https://budgetgolfsa.github.io/liftsafe-loop/session-handoff.md

## LS-042 Navigation Audit (Codex) - COMPLETE local staging
- Commit: `3042507` (`fix(web): close navigation audit gaps`)
- Follow-up commit: `07cc6fe` (`fix(nav): guard mobile and programmatic exits`)
- Failing web screens fixed: `/admin`, `/admin/certificates`, `/equipment/new`, `/certificates/[id]`
- Failing shared mobile behavior fixed: form-like stack exits now warn with `You have unsaved changes. Leave anyway?`
- Failing shared web behavior fixed: dirty forms now guard programmatic `history.pushState` / `replaceState` navigation in addition to links/back
- Locked import screen untouched at the time: `web/app/import/page.tsx`
- Proof: `npx.cmd tsc -p tsconfig.json --noEmit` PASS; `cd web && npm.cmd run build` PASS

## LS-044 Safety Harness (Codex) - IN PROGRESS local UI
- Screens: `app/(lmi)/inspections/safety-harness/index.tsx`, `register.tsx`, `[id].tsx`, wired into `app/(lmi)/inspections/index.tsx` (Section 2 amber) and `app/(lmi)/_layout.tsx`
- DB untouched: no migrations, no Supabase calls, no submit path wired
- SANS 50358:2008 (waist belt + work positioning lanyard) and SANS 50360:2003 (retractable fall arrester) criteria added from Piet-confirmed clauses only
- Result buttons are UI/local draft only: CONFORMS / DOES NOT CONFORM / NOT APPLICABLE
- Source guard: SANS 50364 is test methods only, must not drive field inspection criteria
- Proof: `npx.cmd tsc -p tsconfig.json --noEmit` PASS; no DB touched

## Next Action
1. **Piet decision needed**: fix the `session-end.mjs` vs `AGENTS.md` §17 handoff-file conflict (see banner at top) before it destroys handoff content again.
2. **Piet GATE**: prod web deploy (`cd web && vercel --prod`) to actually enable LS-041's commit+undo test on production.
3. Codex: keep LS-044 DB/submission disabled until explicitly approved.
