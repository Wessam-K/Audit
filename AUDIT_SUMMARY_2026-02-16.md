# WK POS Enterprise — Readiness Audit Summary

**Date:** 2026-02-16  
**Author:** Enterprise Readiness Audit  
**Classification:** INTERNAL — Executive Summary  
**System Value:** $6,000–$9,000 USD

---

## Audit Deliverables

| # | Document | Phase | Status |
|---|----------|-------|--------|
| 1 | [SYSTEM_MAP_2026-02-16.md](SYSTEM_MAP_2026-02-16.md) | Phase 0 | ✅ Complete |
| 2 | [CONFIGURATION_STANDARD_2026-02-16.md](CONFIGURATION_STANDARD_2026-02-16.md) | Phase 1 | ✅ Complete |
| 3 | [HARDCODED_ENDPOINTS_REMOVED_2026-02-16.md](HARDCODED_ENDPOINTS_REMOVED_2026-02-16.md) | Phase 1 | ✅ Complete |
| 4 | [SYNC_PROTOCOL_2026-02-16.md](SYNC_PROTOCOL_2026-02-16.md) | Phase 2 | ✅ Complete |
| 5 | [SYNC_INVARIANTS_2026-02-16.md](SYNC_INVARIANTS_2026-02-16.md) | Phase 2 | ✅ Complete |
| 6 | [MULTI_BRANCH_AUDIT_2026-02-16.md](MULTI_BRANCH_AUDIT_2026-02-16.md) | Phase 3 | ✅ Complete |
| 7 | [EDGE_AGENT_HARDENING_2026-02-16.md](EDGE_AGENT_HARDENING_2026-02-16.md) | Phase 4 | ✅ Complete |
| 8 | [LICENSE_ALIGNMENT_2026-02-16.md](LICENSE_ALIGNMENT_2026-02-16.md) | Phase 5 | ✅ Complete |
| 9 | [BUG_SCAN_2026-02-16.md](BUG_SCAN_2026-02-16.md) | Phase 6 | ✅ Complete |

---

## Code Fixes Applied

| # | Fix | File(s) Modified | Invariant |
|---|-----|-------------------|-----------|
| 1 | Added `tenantId` to sale idempotency check | `sales.service.ts` | INV-1 |
| 2 | Server-generates `idempotencyKey` if not provided | `sales.service.ts` | INV-1 |
| 3 | Added stock deduction to `syncSale` (was skipping inventory entirely) | `sync.service.ts` | INV-2 |
| 4 | Imported canonical `recordMovement` from `inventory.service.ts` | `sync.service.ts` | INV-2, INV-3 |
| 5 | Added `tenantId` to sync idempotency check | `sync.service.ts` | INV-1 |
| 6 | Added `@@unique([tenantId, clientId, entityType, entityId, clientSeq])` to SyncLog | `schema.prisma` | INV-4 |
| 7 | Fixed BullMQ idempotency key to be deterministic (removed `Date.now()`) | `sync-handlers.ts` | INV-12 |
| 8 | Removed developer LAN IP `192.168.0.157` from Divamos CORS | `server.ts` | H-008 |
| 9 | Made OpenAPI server URL dynamic via `API_BASE_URL` env var | `openapi.ts` | M-002 |

---

## Findings by Severity

### CRITICAL (12 items)

| # | Finding | Phase | Fixed? |
|---|---------|-------|--------|
| G-001 | `idempotencyKey` optional on Sale creation | Phase 2 | ✅ Server-generates if missing |
| G-002 | SyncLog has no DB-level unique constraint | Phase 2 | ✅ `@@unique` added |
| G-003 | 3 separate stock deduction paths, only 1 uses optimistic locking | Phase 2 | ⚠️ Partial (syncSale fixed) |
| G-004 | `syncSale` doesn't deduct stock | Phase 2 | ✅ Fixed |
| L-001 | 38 of 60 modules have no license gating | Phase 5 | ❌ Needs seed + middleware |
| L-002 | `SKIP_LICENSE_CHECK` can bypass all enforcement | Phase 5 | ❌ Needs production guard |
| EA-001 | Static auth token in plain file (Edge Agent) | Phase 4 | ❌ Needs keytar + JWT |
| EA-002 | Branch binding is config-only, no server validation | Phase 4 | ❌ Needs device→branch binding |
| EA-003 | Dead-letter handling absent in Edge Agent | Phase 4 | ❌ Needs DEAD_LETTER status |
| I-005 | Cross-tenant idempotency key collision (sale) | Phase 2 | ✅ Added tenantId filter |
| I-006 | Divamos checkout has no idempotency | Phase 2 | ❌ Needs idempotencyKey field |
| I-007 | Divamos Stripe webhook doesn't check event.id | Phase 2 | ❌ Needs dedup check |

### HIGH (15 items)

| # | Finding | Phase | Fixed? |
|---|---------|-------|--------|
| G-005 | ~45 hardcoded IP/URL locations in source code | Phase 1 | ⚠️ Partial (2 fixed) |
| G-006 | BullMQ sync job key uses `Date.now()` | Phase 2 | ✅ Fixed |
| B-001 | WebSocket broadcasts tenant-only, not branch-scoped | Phase 3 | ❌ Needs branch rooms |
| B-002 | Only 1 of 60 routes uses `resolveBranchScope` middleware | Phase 3 | ❌ Needs adoption |
| L-003 | Hardcoded license fallback limits mismatch | Phase 5 | ❌ Needs sync |
| L-004 | Dual seed files with conflicting module definitions | Phase 5 | ❌ Delete duplicate |
| L-008 | No per-route `requireModule()` middleware | Phase 5 | ❌ Needs creation |
| EA-004 | Pre-ack attempt increment causes premature exhaustion | Phase 4 | ❌ |
| EA-005 | No incoming packet checksum validation | Phase 4 | ❌ |
| EA-006 | No license gate on local events | Phase 4 | ❌ |
| EA-007 | No full resync mechanism | Phase 4 | ❌ |
| Q-001 | 5 empty event listeners in `eventListeners.ts` | Phase 6 | ❌ |
| Q-002 | Express `Request` type not extended (~80 `as any` casts) | Phase 6 | ❌ |
| Q-003 | Unbounded queries in WebSocket/sync services (OOM risk) | Phase 6 | ❌ |
| Q-004 | 28 route files without Zod input validation | Phase 6 | ❌ |

### MEDIUM (12 items)

| Finding | Phase |
|---------|-------|
| 9 backend config localhost defaults | Phase 1 |
| Attendance has zero branch awareness | Phase 3 |
| Branch selector not global in admin panel | Phase 3 |
| Full sync data ignores branch | Phase 3 |
| Edge Agent RSA key is placeholder | Phase 4 |
| No frontend module gating in admin panel | Phase 5 |
| No module assignment admin UI | Phase 5 |
| StockAlert + 7 models missing Tenant relation | Phase 6 |
| 8 TODOs in production code (5 empty handlers) | Phase 6 |
| `rbac.service.ts` untracked setTimeout ref | Phase 6 |
| `scheduled-report.routes.ts` fire-and-forget timer | Phase 6 |
| OrderNotification missing unique constraint | Phase 2 |

---

## Remediation Roadmap

### Sprint 1 (Immediate — Data Integrity)

- [x] Sale idempotency key: server-generate if missing
- [x] SyncLog: add DB unique constraint
- [x] syncSale: add stock deduction via canonical inventory service
- [x] BullMQ: fix non-deterministic idempotency key
- [x] Divamos: remove developer LAN IP from CORS
- [ ] `SKIP_LICENSE_CHECK`: production guard
- [ ] Edge Agent: dead-letter handling
- [ ] Divamos checkout: add idempotency key
- [ ] Divamos Stripe webhook: check event.id

### Sprint 2 (Branch & Config)

- [ ] WebSocket: join branch rooms, use `broadcastToBranch()`
- [ ] Apply `resolveBranchScope` to all branch-scoped route files
- [ ] Move branch selector to global layout
- [ ] Fix remaining hardcoded endpoints (12 HIGH files)
- [ ] Extend Express `Request` type
- [ ] Add `take`/`limit` to unbounded queries

### Sprint 3 (License & Security)

- [ ] Expand license seed to 36 modules
- [ ] Create `requireModule()` middleware
- [ ] Apply per-route module gating
- [ ] Add Zod validation to unvalidated routes
- [ ] Frontend module gating hook
- [ ] Implement empty event listener handlers

### Sprint 4 (Edge Agent & Polish)

- [ ] Edge Agent: JWT with refresh token rotation
- [ ] Edge Agent: server-side device→branch validation
- [ ] Edge Agent: incoming packet checksum validation
- [ ] Edge Agent: full resync mechanism
- [ ] Remove dead Edge Agent dependencies
- [ ] TypeScript cleanup (`as any` reduction)

---

## Metrics

| Metric | Value |
|--------|-------|
| Total findings | **~307** |
| Critical | **12** |
| High | **15** |
| Medium | **12** |
| Type warnings (`as any`) | **200+** |
| Info | **7** |
| Fixed this session | **9 code changes** |
| Documents produced | **9 audit reports** |
| Applications audited | **9** (API, License, Divamos, Desktop, Admin, Mobile, Edge, Legacy, Kotlin) |
| Models analyzed | **178** (128 POS + 26 Divamos + 24 License) |
| Source files scanned | **60+ modules, 15 middleware, 57 pages** |

---

*End of Enterprise Readiness Audit Summary*
