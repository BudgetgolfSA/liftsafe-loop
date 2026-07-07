# LiftSafe Session Handoff
Last updated: 2026-07-07T19:50:00+02:00
Branch: staging

## Protocol
- §17 handoff: mandatory after every task
- §18 Grok Slack post: mandatory before reporting to Piet

## ONLY PRIORITY
**LS-041 prod smoke test BLOCKED (not FAIL): prod web has never been deployed with the LS-041 UI.** DB schema + edge function ARE already on prod (verified read-only). The commit+undo/`created_ids` mechanism can't be exercised on https://admin.liftsafe.co.za until `web/` is promoted to production — a GATE requiring Piet's explicit go-ahead. Full detail: `logs/20260707-prod-import-smoke-test.md`.

**Recurring environment issue, flagged again**: the local `c:\liftsafe` working directory has switched itself from `staging` to `main` at least 3 times this session with no matching git reflog entry, and separately had two files silently revert mid-edit (no reflog entry either — consistent with an untracked `git checkout --` from a concurrent process, not this agent). Every task this session double-checked `git branch --show-current` before committing and diffed before pushing, so nothing has actually landed on `origin/main` — but this needs runtime-level investigation (are multiple agents/IDEs sharing this exact checkout?) since it's pure luck no work has been lost to it yet.

## Agent Status (2026-07-07)
| Agent | Task | Status | Handoff |
|---|---|---|---|
| Grok | LS-041 extraction layer | COMPLETE | pushed |
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
- Hierarchy step grid becomes 3 columns (`320px_1fr_360px`: viewer | tree | equipment items) when a viewable PDF/image is present, else falls back to the existing 2-column layout
- Also fixed: both item-label elements in the right panel used Tailwind `truncate` (hover-title only) — swapped for `break-words` so full `asset_description` always renders
- Staging-verified with a real generated test PDF through the actual AI extraction path: reached hierarchy step with "ALL 6 ITEMS PLACED" (incidental live re-confirmation the tonnage-line fix still holds), iframe `blob:` URL confirmed present, 0 `.truncate` on item labels (only unrelated sidebar user-name text still has it)
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
- Proof: `logs/20260707-ls040-lme-branding-ui.md` + screenshot `logs/20260707-ls040-sidebar-branding.png` (local, gitignored) — sidebar live-verified on staging (trading_name fallback rendered correctly + watermark); client portal's new query verified against real staging data (read-only) since no `client_portal` test account exists on staging to browser-test the render directly
- TSC: root `app/` + `web/` both 0 errors. `cd web && npm run build` PASS
- **Note for next session**: while editing `SideNav.tsx` and `client/layout.tsx`, both files were silently reverted mid-task by something in this shared working directory (no git reflog entry — likely a `git checkout --` from a concurrent process, not a ref-changing op). Re-applied and committed immediately with `BUILD-LOCK.json` entries held during the redo. Locks released after commit.

## Import hierarchy right panel (Claude Code) — COMPLETE staging, UI only
- Commit: `ebd38d4c` (`feat(web): add right panel to import hierarchy builder for per-item equipment visibility`)
- `web/app/import/page.tsx`: right panel added to the hierarchy step — itemized UNASSIGNED list with per-item assign dropdown, items grouped/listed under their section once assigned (with reassign/unassign dropdown), unassigned-count banner at the top of the step (`ALL N ITEMS PLACED` / `X OF N ITEMS UNASSIGNED`)
- Section item counts were already live off `section.rows.length` — new per-item moves (`moveItem()`) write to the same `hierarchyDraft` state so both panels + the left tree stay in sync from one source of truth
- No DB/migrations/edge functions touched — pure UI
- Proof: `logs/20260707-import-hierarchy-right-panel.md` + screenshot `logs/20260707-import-hierarchy-right-panel.png` (local only, gitignored) — verified live on staging: unassigned an item via dropdown (banner + counts updated), reassigned it to a different section via the unassigned-list dropdown (banner + counts updated again)
- TSC: root `app/` + `web/` both 0 errors. `cd web && npm run build` PASS

## LS-041 Extraction (Grok) — COMPLETE staging edge
- `154.65.98.90`: v2 `intakePrompt` + full foreman live
- `test-extract-parse.mjs` 4/4 PASS
- Prod edge: **actually deployed** (verified read-only 2026-07-07, byte-identical to staging) — this was a properly Piet-authorized "ship it" done concurrently in a parallel session, documented in `logs/20260707-prod-ls041-ship.md` / `-verify.md` (found after the fact via read-only check, not undocumented drift — correcting an earlier note in this same session that mischaracterized it)

## LS-041 UI + full build (Claude Code)
- Committed staging: `3da33cc`, `import-session.ts` wired
- Full build audit: `logs/20260707-ls041-fullbuild-verification.md` — all UI pieces confirmed in code (client confirm, hierarchy tree, inline create/edit/rename, unassigned panel, draft resume, atomic commit + undo), no TODO/stub markers
- Smoke test (staging): **PASS 12/12** — `logs/20260707-ls041-fullbuild-smoke-test.log`
- TSC: root `app/` 0 errors. `web/` has 1 pre-existing unrelated error in `web/e2e/helpers/rpc.ts` (not caused by this session, not fixed — shared `web/e2e/**` lane, flagged not touched)
- Import for Piet: **UNBLOCKED (staging)**

## LS-041 prod smoke test (Claude Code) — BLOCKED, full detail in `logs/20260707-prod-import-smoke-test.md`
- Prod DB schema (`import_sessions`, `deleted_at` cols, `bulk_create_assets` returning `created_ids`) — confirmed present, read-only
- Prod `extract-document` edge function — confirmed present, byte-identical to staging
- **Prod web app has not been deployed since before LS-041 UI landed** (`vercel ls liftsafe --prod` shows last successful production deploy ~1 day old; a deploy attempt ~52min before this test, by account `dylsavtwa1-arch`, shows status `Error`) — so the live `/import` page still runs the pre-LS-041 flat-CSV flow (no `import_sessions` row, no undo button)
- Created a dedicated, fully-isolated smoke-test tenant via real `/register` signup (Piet-approved): `Prod Smoke LME 1783435869954` / `prod-smoke+1783435869954@mvotiweights.co.za` — still exists on prod, reusable once web is deployed
- First browser-driven commit attempt mis-mapped CSV columns (my error, not a product bug — column-mapping UI is *target field → source column*, not the reverse) → 2 garbled asset rows, found + removed (Piet-approved cleanup)
- Retried with corrected mapping → old flat-CSV path committed cleanly (5 created, 0 skipped) but created no session to test undo against
- **Outstanding, needs Piet's call**: 5 correctly-created TEST_IMPORT rows + client still on prod, uncleaned (no session to undo through); throwaway smoke tenant still on prod for reuse; actual commit+undo test still pending a prod web deploy

## Handoff URL (§17)
https://budgetgolfsa.github.io/liftsafe-loop/session-handoff.md

## AGENTS.md fixes (2026-07-07)
- §17/§14 URL: `liftsafe-loop/session-handoff.md` (canonical)
- §18 webhook: `SLACK_LOOP_WEBHOOK`

## LS-042 Navigation Audit (Codex) - COMPLETE local staging
- Commit: `3042507` (`fix(web): close navigation audit gaps`)
- Follow-up commit: `07cc6fe` (`fix(nav): guard mobile and programmatic exits`)
- Failing web screens fixed: `/admin`, `/admin/certificates`, `/equipment/new`, `/certificates/[id]`
- Failing shared mobile behavior fixed: form-like stack exits now warn with `You have unsaved changes. Leave anyway?`
- Failing shared web behavior fixed: dirty forms now guard programmatic `history.pushState` / `replaceState` navigation in addition to links/back
- Existing covered shells verified: `AdminLayout`, `ClientLayout`, `FinanceLayout`, `PublicRouteNavigation`, `MobileStackHeader`
- Redirect-only routes noted as no-screen exemptions: `/dashboard/safety`, `/dashboard/safety/[userId]`, `/lmi-tools/scan`
- Locked import screen untouched: `web/app/import/page.tsx`
- Proof: `npx.cmd tsc -p tsconfig.json --noEmit` PASS; `cd web && npm.cmd run build` PASS

## LS-044 Safety Harness (Codex) - IN PROGRESS local UI
- Screen list first reported to Piet:
  - `app/(lmi)/inspections/index.tsx` - add Safety Harness under Section 2 LIFTING TACKLE amber cluster
  - `app/(lmi)/inspections/safety-harness/index.tsx` - new UI-only landing screen
  - `app/(lmi)/inspections/safety-harness/register.tsx` - new UI-only local draft register/start screen
  - `app/(lmi)/inspections/safety-harness/[id].tsx` - new UI-only inspection draft screen
  - `app/(lmi)/_layout.tsx` - route stack registration
- DB untouched: no migrations, no Supabase calls, no submit path wired
- Existing forms untouched: `app/(lmi)/inspections/fall-arrest/*` not modified
- SANS 50358:2008 criteria added from Piet-confirmed clauses only:
  - Waist belt: cl.4.1.1.1, cl.4.1.1.2, cl.4.1.1.3, cl.4.1.1.4, cl.4.1.1.5, cl.4.1.1.6, cl.4.2.1.1, cl.4.2.3
  - Work positioning lanyard: cl.4.1.2.1, cl.4.1.2.3, cl.4.1.2.6, cl.4.1.3.4, cl.4.2.1.3, cl.6.2
- SANS 50360:2003 retractable fall arrester criteria added from Piet-confirmed clauses only:
  - cl.4.2 material, cl.4.2 connector swivel, cl.4.2 termination, cl.4.3.1 locking function, cl.4.4 static strength certificate, cl.4.5 dynamic performance certificate, cl.4.7 corrosion, cl.6 marking
- Result buttons are UI/local draft only and use only: CONFORMS / DOES NOT CONFORM / NOT APPLICABLE
- Source guard: SANS 50364 is test methods only and must not drive field inspection criteria
- Proof: `npx.cmd tsc -p tsconfig.json --noEmit` PASS; no DB touched

## Next Action
Codex: keep LS-044 DB/submission disabled until explicitly approved
