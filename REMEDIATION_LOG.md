# Remediation Log (Append-Only)

**Created:** 2026-02-16  
**Format:** Each entry: `[YYYY-MM-DD HH:MM] | What | Why | How Verified`

---

## Log

| Timestamp | Change | Reason | Verification |
|-----------|--------|--------|-------------|
| 2026-02-16 00:00 | Added tenantId to sale idempotency check | Cross-tenant collision prevention (INV-1) | Code review + build passes |
| 2026-02-16 00:00 | Server-generates idempotencyKey if missing | Prevent duplicate sales on retry (INV-1) | Code review + build passes |
| 2026-02-16 00:00 | Added stock deduction to syncSale | Phantom inventory bug — synced sales didn't deduct stock (INV-2) | Code review + build passes |
| 2026-02-16 00:00 | Added @@unique to SyncLog | DB-level dedup for concurrent sync requests (INV-4) | Schema review |
| 2026-02-16 00:00 | Fixed BullMQ key to deterministic | Non-deterministic Date.now() in idempotency key (INV-12) | Code review |
| 2026-02-16 00:00 | Removed developer LAN IP from Divamos CORS | Security: hardcoded developer IP in production CORS | Code review |
| 2026-02-16 00:00 | Made OpenAPI server URL dynamic | Hardcoded localhost in API docs | Code review |

---

*New entries will be appended below as changes are made.*

## Session 2 — 2026-02-16 (Production Readiness Sprint)

| Timestamp | Change | Reason | Verification |
|-----------|--------|--------|-------------|
| 2026-02-16 S2-01 | Extended Express Request type in express.d.ts | Typed user, branchScope, requestId (P0-06) | Build clean |
| 2026-02-16 S2-02 | WebSocket branch rooms + scoped broadcasts | Branch isolation (B-001, B-002) — 17 route files | Build clean |
| 2026-02-16 S2-03 | Added `take: 5000` to WebSocket getFullSyncData | OOM prevention on full sync (Q-003) | Build clean |
| 2026-02-16 S2-04 | Added resolveBranchScope to 17 route files | Multi-branch scoping gap (B-001) | Build clean |
| 2026-02-16 S2-05 | License-gate production guard | SKIP_LICENSE_CHECK cannot be enabled in NODE_ENV=production (L-002) | Build clean |
| 2026-02-16 S2-06 | Event listeners implemented (5 handlers) | Empty TODO stubs replaced with ActivityLog audit entries (Q-001) | Build clean |
| 2026-02-16 S2-07 | Edge Agent dead-letter + startup recovery | EA-003: DEAD_LETTER status, EA-011: recoverInFlight() | Code review |
| 2026-02-16 S2-08 | Edge Agent post-ack increment fix | EA-004: Attempt no longer inflated before server ack | Code review |
| 2026-02-16 S2-09 | Edge Agent WAL checkpoint on shutdown | EA-008: TRUNCATE WAL on closeDatabase() | Code review |
| 2026-02-16 S2-10 | Sales stock deduction wrapped in $transaction | I-004: Optimistic locking via StockLevel.version | Build clean |
| 2026-02-16 S2-11 | Divamos checkout idempotency key | I-006: Prevents duplicate orders from same cart | Schema updated |
| 2026-02-16 S2-12 | Divamos Stripe webhook event.id dedup | I-007: WebhookEvent model prevents double-processing | Schema updated |
| 2026-02-16 S2-13 | OrderNotification @@unique constraint | I-008: Prevents duplicate notifications per order/type/target | Schema updated |
| 2026-02-16 S2-14 | Divamos checkout + webhook stock in $transaction | Atomic stock deduction on order completion | Build clean |
| 2026-02-16 S2-15 | Idempotency middleware wired to sale POST route | I-009: Redis-cached response dedup on sale creation | Build clean |
| 2026-02-16 S2-16 | Eliminated ~96 `(req as any).user` casts | P0-06: Typed via express.d.ts + TokenPayload alignment | Build clean |
| 2026-02-16 S2-17 | Fixed `(req as any).cookies` → `req.cookies` | P0-06: Proper typing in auth middleware | Build clean |
| 2026-02-16 S2-18 | Aligned TokenPayload in rbac.middleware.ts | Missing `id`, `email`, `fullName`, `isActive` fields | Build clean |
| 2026-02-16 S2-19 | License seed expanded from 22 to 51 modules | P1-01: All business modules now seeded with categorization | Code review |
| 2026-02-16 S2-20 | Enterprise plan expanded to include all modules | P1-01: Enterprise plan covers all 51 seeded modules | Code review |
| 2026-02-16 S2-21 | Added take/limit to 9 unbounded findMany queries | P1-04: Safety caps on reports, exports, user lists | Build clean |
| 2026-02-16 S2-22 | Created DEPLOYMENT_ON_PREM.md runbook | Production deployment guide for LAN/on-prem | Documentation |
| 2026-02-16 S2-23 | Created RUNBOOK_OFFLINE_BRANCH.md | Offline branch operations guide for operators | Documentation |
