# Production Readiness Backlog

**Created:** 2026-02-16  
**Owner:** Agent  
**Last Updated:** 2026-02-16

---

## P0 — Data Integrity & Security (Sprint 1)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P0-01 | WebSocket branch rooms — join `branch:{branchId}` on connect, use `broadcastToBranch()` | 🔲 TODO | Branch A client does NOT receive Branch B `data:invoice` events | `websocket.service.ts` |
| P0-02 | Apply `resolveBranchScope` to all branch-model routes (sale, inventory, shifts, held-sale, accounting, purchase-orders, stocktaking, stock-audit, stock-logs, approvals, serials, reservation, adjustments, stock-transfers) | 🔲 TODO | Non-admin user with Branch A access cannot list Branch B records | 14+ route files |
| P0-03 | `getFullSyncData()` must filter stock by branch | 🔲 TODO | Full sync returns only requested branch's stock levels | `websocket.service.ts` |
| P0-04 | Attendance routes: add branchId awareness + requirePermission | 🔲 TODO | Clock-in records branchId; list filters by branch | `attendance.routes.ts` |
| P0-05 | Close remaining sync idempotency gaps (I-004, I-006, I-007, I-008) | 🔲 TODO | Replay test: 10x retry → 1 record created | multiple sync files |
| P0-06 | Extend Express `Request` type → eliminate ~80 `as any` in critical paths | 🔲 TODO | `npx tsc --noEmit` passes with fewer `as any` | `express.d.ts` + all route files |
| P0-07 | `SKIP_LICENSE_CHECK` production guard | 🔲 TODO | Env `NODE_ENV=production` + `SKIP_LICENSE_CHECK=true` → throws on startup | `license-gate.middleware.ts` |
| P0-08 | Edge Agent dead-letter handling (EA-003) | 🔲 TODO | After max_attempts, status = DEAD_LETTER + error logged | `outbox-processor.ts` |
| P0-09 | Edge Agent startup recovery for in-flight entries (EA-011) | 🔲 TODO | Crash → restart → IN_PROGRESS entries reset to PENDING | `outbox-processor.ts` |

## P1 — Enforcement & Config (Sprint 2)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P1-01 | Expand license seed to 36 modules | 🔲 TODO | `SELECT count(*) FROM Module` = 36 after seed | `seed-canonical.ts` |
| P1-02 | Apply `requireModule()` to route files (PRO+/ENTERPRISE routes) | 🔲 TODO | BASIC license → 403 on `/api/v1/crm/*` | 14+ route files |
| P1-03 | Implement empty event listeners (sale.refunded, sale.voided, inventory.adjusted, customer.*) | 🔲 TODO | Event fires → audit log created + webhook triggered | `eventListeners.ts` |
| P1-04 | Add `take`/`limit` to unbounded queries | 🔲 TODO | `getFullSyncData` returns max 5000 per entity | `websocket.service.ts`, `sync.service.ts` |
| P1-05 | Add Zod validation to unvalidated routes (top 10 highest risk) | 🔲 TODO | Invalid body → 400 with structured error | 10+ route files |
| P1-06 | Edge Agent: post-ack attempt increment (EA-004) | 🔲 TODO | Crash between mark & ack → attempt count not inflated | `outbox-processor.ts` |
| P1-07 | Edge Agent: WAL checkpoint on shutdown (EA-008) | 🔲 TODO | Clean shutdown → WAL file truncated | `database/index.ts` |
| P1-08 | Edge Agent: remove dead dependencies (EA-010) | 🔲 TODO | `electron-store`, `axios`, `uuid`, `node-cron` removed from deps | `package.json` |

## P2 — Polish & Documentation (Sprint 3)

| ID | Task | Status | Acceptance Test | Files Affected |
|----|------|--------|-----------------|----------------|
| P2-01 | Add Tenant relation to StockAlert + 7 models | 🔲 TODO | Tenant delete cascades to StockAlert | `schema.prisma` |
| P2-02 | Track setTimeout refs (rbac.service, scheduled-report) | 🔲 TODO | Graceful shutdown cancels all timers | 2 files |
| P2-03 | Production runbooks | 🔲 TODO | 5 runbook docs in Audit/ | Audit/ folder |
| P2-04 | Local test scripts | 🔲 TODO | `scripts/test-*.ps1` exist and execute | scripts/ folder |
| P2-05 | Build + export all apps | 🔲 TODO | API builds, admin-panel builds, desktop exports | multiple |

---

*Backlog will be updated as tasks are completed.*
