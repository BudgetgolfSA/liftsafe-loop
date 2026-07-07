# liftsafe-loop

Public agent handoff mirror for LiftSafe.

**Agents read (HTTP 200, no auth):**
https://raw.githubusercontent.com/BudgetgolfSA/liftsafe-loop/main/session-handoff.md

**Source:** `BudgetgolfSA/liftsafe` → `docs/session-handoff.md` on `staging`

**Push after every task:**
```bash
node scripts/push-handoff-loop.mjs "task name"
```

No secrets. No PII.