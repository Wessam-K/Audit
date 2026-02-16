# Production Readiness Backlog

**Created:** 2026-02-16  
**Owner:** Agent  
**Last Updated:** 2026-02-16 (Session 4 — All P0/P1/P2 implemented)

---

## P0 — Data Integrity & Security (Sprint 1)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P0-01 | WebSocket branch rooms — join `branch:{branchId}` on connect, use `broadcastToBranch()` | ✅ DONE (S2-02) | Branch A client does NOT receive Branch B `data:invoice` events | `websocket.service.ts` |
| P0-02 | Apply `resolveBranchScope` to all branch-model routes | ✅ DONE (S2-04) | Non-admin user with Branch A access cannot list Branch B records | 17 route files |
| P0-03 | `getFullSyncData()` must filter stock by branch | ✅ DONE (S2-03) | Full sync returns only requested branch's stock levels | `websocket.service.ts` |
| P0-04 | Attendance routes: add branchId awareness + requirePermission | ✅ DONE (S2-04) | Clock-in records branchId; list filters by branch | `attendance.routes.ts` |
| P0-05 | Close remaining sync idempotency gaps (I-004, I-006, I-007, I-008) | ✅ DONE (S2-10..15) | Replay test: 10x retry → 1 record created | multiple sync files |
| P0-06 | Extend Express `Request` type → eliminate ~80 `as any` in critical paths | ✅ DONE (S2-01,16,17) | `npx tsc --noEmit` passes with fewer `as any` | `express.d.ts` + all route files |
| P0-07 | `SKIP_LICENSE_CHECK` production guard | ✅ DONE (S2-05) | Env `NODE_ENV=production` + `SKIP_LICENSE_CHECK=true` → throws on startup | `license-gate.middleware.ts` |
| P0-08 | Edge Agent dead-letter handling (EA-003) | ✅ DONE (S2-07) | After max_attempts, status = DEAD_LETTER + error logged | `outbox-processor.ts` |
| P0-09 | Edge Agent startup recovery for in-flight entries (EA-011) | ✅ DONE (S2-07, S4-05) | Crash → restart → IN_PROGRESS entries reset to PENDING + called at startup | `outbox-processor.ts`, `agent-service.ts` |

## P1 — Enforcement & Config (Sprint 2)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P1-01 | Expand license seed to 53 modules | ✅ DONE (S2-19,20) | `SELECT count(*) FROM Module` = 53 after seed | `seed-canonical.ts` |
| P1-02 | Apply `requireModule()` to route files (PRO+/ENTERPRISE routes) | ✅ DONE (S2) | BASIC license → 403 on `/api/v1/crm/*` | 20+ route files |
| P1-03 | Implement event listeners (sale.refunded, sale.voided, inventory.adjusted, customer.*) | ✅ DONE (S2-06, S4-21,22) | Event fires → audit log + notification + stock alert | `eventListeners.ts` |
| P1-04 | Add `take`/`limit` to unbounded queries | ✅ DONE (S2-03,21) | `getFullSyncData` returns max 5000 per entity | `websocket.service.ts`, 9+ files |
| P1-05 | Add Zod validation to unvalidated routes (18 endpoints) | ✅ DONE (S4-11..16) | Invalid body → 400 with structured error | 5 schemas + 5 route files |
| P1-06 | Edge Agent: post-ack attempt increment (EA-004) | ✅ DONE (S2-08) | Crash between mark & ack → attempt count not inflated | `outbox-processor.ts` |
| P1-07 | Edge Agent: WAL checkpoint on shutdown (EA-008) | ✅ DONE (S2-09) | Clean shutdown → WAL file truncated | `database/index.ts` |
| P1-08 | Edge Agent: remove dead dependencies (EA-010) | ✅ DONE (S4-08) | `electron-store`, `axios`, `uuid`, `node-cron` removed from deps | `package.json` |

## P2 — Polish & Documentation (Sprint 3)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P2-01 | Add Tenant relation to StockAlert + 7 models | ✅ DONE (S4-17,18) | Tenant delete cascades to all 8 models | `schema.prisma` |
| P2-02 | Track setTimeout refs (rbac.service, scheduled-report) | ✅ DONE (S4-19,20) | Graceful shutdown cancels all timers; no fire-and-forget | 2 files |
| P2-03 | Production runbooks | ✅ DONE (S2-22,23) | DEPLOYMENT_ON_PREM.md + RUNBOOK_OFFLINE_BRANCH.md in Audit/ | Audit/ folder |
| P2-04 | Local test scripts | 🔲 TODO | `scripts/test-*.ps1` exist and execute | scripts/ folder |
| P2-05 | Build + export all apps | ✅ DONE (S4) | API builds clean (`npm run build` passes), Prisma generates | multiple |

---

---

## Summary (Session 4)

- **P0 (9/9):** All critical data integrity and security items completed
- **P1 (8/8):** All enforcement and config items completed  
- **P2 (4/5):** 4 of 5 polish items done. P2-04 (local test scripts) deferred.
- **Total: 21/22 items completed (95.5%)**
- **Backend build:** Clean (`npm run build` — zero errors)
- **Prisma client:** Regenerated successfully (v5.22.0)
