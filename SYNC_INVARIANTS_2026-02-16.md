# WK POS Enterprise — Sync Invariants (Hard Rules)

**Date:** 2026-02-16  
**Author:** Enterprise Readiness Audit (Phase 2)  
**Classification:** INTERNAL — Data Integrity Contract  
**Status:** PROPOSED — Must be enforced before production

---

## Purpose

These invariants are **non-negotiable rules** that protect data integrity across the WK POS ecosystem. Every sync path, every write endpoint, and every stock mutation MUST satisfy these rules. Violations produce duplicate sales, phantom inventory, or lost revenue.

---

## Invariant 1: Every Sale MUST Have an Idempotency Key

```
∀ sale ∈ Sale:
  sale.idempotencyKey IS NOT NULL
  AND sale.idempotencyKey IS UNIQUE (globally)
```

### Enforcement
- `CreateInvoiceDto` MUST require `idempotencyKey` field
- If client does not provide one, server MUST generate: `{tenantId}:{branchId}:{deviceId}:{timestamp}:{uuid}`
- Database constraint: `idempotencyKey String @unique` (already exists but field is nullable)
- **Migration required:** `ALTER TABLE "Sale" ALTER COLUMN "idempotencyKey" SET NOT NULL`

### Rationale
Without this, a client retry creates a second sale with a different auto-generated `saleNumber`, causing double revenue recording and double stock deduction.

---

## Invariant 2: Every Sale MUST Deduct Stock

```
∀ sale ∈ Sale WHERE sale.status IN ('COMPLETED', 'SYNCED'):
  ∀ item ∈ sale.items:
    ∃ log ∈ InventoryLog:
      log.referenceType = 'SALE'
      AND log.referenceId = sale.id
      AND log.variantId = item.variantId
      AND log.quantity = item.quantity
      AND log.type = 'OUT'
```

### Enforcement
- ALL code paths that create a completed sale MUST call `inventoryService.recordMovement()`
- This includes: `completeSaleFlow`, `salesService.createInvoice`, `modules/sync.service.syncSale`, `services/sync.service.applySale`
- **Currently violated by:** `modules/sync.service.syncSale` (creates sale, skips stock deduction)

### Rationale
A sale without stock deduction creates "phantom inventory" — the system reports items in stock that were actually sold. This causes overselling, incorrect financial reports, and inventory count mismatches.

---

## Invariant 3: Stock Deductions MUST Use Optimistic Locking

```
∀ stockDeduction:
  stockDeduction MUST check StockLevel.version
  AND stockDeduction MUST increment StockLevel.version
  AND IF version mismatch: RETRY or FAIL (never silently skip)
```

### Enforcement
- ALL stock mutations MUST go through `inventoryService.recordMovement()`
- Direct `prisma.stockLevel.updateMany({ decrement })` without version check is FORBIDDEN
- **Currently violated by:** `salesService.createInvoice` (direct decrement, no version check)

### Rationale
Without optimistic locking, two concurrent sales of the last item both succeed, creating negative stock. With optimistic locking, the second sale detects the version conflict and retries or reports out-of-stock.

---

## Invariant 4: SyncLog Deduplication MUST Be Database-Enforced

```
∀ (tenantId, clientId, entityType, entityId, clientSeq):
  COUNT(SyncLog matching above) ≤ 1
  ENFORCED BY: @@unique database constraint
```

### Enforcement
- Add Prisma schema: `@@unique([tenantId, clientId, entityType, entityId, clientSeq])`
- Convert application-level `findFirst()` checks to `upsert()` or `INSERT ... ON CONFLICT`
- Handle `P2002` (unique constraint violation) as "already synced" response

### Rationale
Application-level `findFirst()` is vulnerable to race conditions: two concurrent sync requests for the same entity both pass the `findFirst()` check (both return null) and both insert, creating a duplicate.

---

## Invariant 5: One Canonical Stock Counter Per (Variant, Branch)

```
∀ (variantId, branchId):
  COUNT(StockLevel records) = 1
  ENFORCED BY: @@unique([variantId, branchId]) — ✅ already exists
```

### Enforcement
- ✅ Already enforced by schema constraint
- External systems (Divamos) that maintain separate stock counters MUST be treated as independent systems requiring explicit reconciliation
- Cross-system stock sync MUST use SET semantics (absolute value), NOT INCREMENT semantics

### Rationale
If two systems both decrement independently, total stock drifts. SET semantics from a single source of truth prevent drift.

---

## Invariant 6: Every Tenant Operation MUST Include tenantId

```
∀ query ∈ CRUD operations on tenant-scoped models:
  query.where MUST include tenantId = currentUser.tenantId
```

### Enforcement
- `tenant-isolation.middleware.ts` injects `tenantId` into request context
- All service-layer queries MUST include `where: { tenantId }`
- **102 of 128 models** have `tenantId` field — these are tenant-scoped
- 26 child/join tables inherit isolation through parent relationships

### Rationale
Missing `tenantId` filter allows Tenant A to see/modify Tenant B's data — a critical security and data isolation violation.

---

## Invariant 7: Branch-Scoped Operations MUST Include branchId

```
∀ query ∈ CRUD operations on branch-scoped models:
  IF user.branchId IS SET:
    query.where MUST include branchId = user.branchId
  UNLESS user has 'branch:view_all' permission
```

### Enforcement
- `branch-scope.middleware.ts` injects `branchId` filter
- 18 models with `branchId` field MUST be filtered
- Users with multi-branch access see aggregated data
- **Audit needed:** Verify all 18 branch-scoped models are actually filtered in their route handlers

### Rationale
Without branch scoping, a cashier at Branch A sees sales/inventory from Branch B, leading to confusion and potential data manipulation.

---

## Invariant 8: WebSocket Broadcasts MUST Be Tenant-Scoped

```
∀ broadcast ∈ Socket.IO server events:
  broadcast.room = `tenant:{tenantId}`
  AND broadcast MUST NOT use io.emit() (global broadcast)
```

### Enforcement
- `websocket.service.ts` joins clients to `tenant:{tenantId}` rooms on authentication
- All data events (`data:invoice`, `data:customer`, etc.) MUST use `to(room).emit()`
- **Anti-echo:** Include `originDeviceId` in broadcast payload so originating client can skip self-updates

### Rationale
Global broadcast leaks Tenant A's data to Tenant B's connected clients.

---

## Invariant 9: Offline Changes MUST Be Conflict-Resolved Server-Side

```
∀ offlineChange ∈ Edge Agent Outbox:
  Server is the SINGLE SOURCE OF TRUTH
  Server-side version wins on conflict
  Client receives rejection with current server state
```

### Enforcement
- Edge Agent: push outbox entries with `idempotencyKey = {branchId}:{deviceId}:{localSequence}`
- Server: check SyncLog + version comparison
- On conflict: return `SyncAck.status = 'REJECTED'` with `serverVersion` attached
- Client: update local SQLite with server's version, mark outbox entry as `CONFLICT_RESOLVED`
- **NEVER** allow client to overwrite newer server data

### Rationale
Last-write-wins without version comparison causes data loss. Server-wins policy ensures a single source of truth.

---

## Invariant 10: Divamos Orders MUST Be Idempotent

```
∀ checkout ∈ Divamos POST /checkout:
  checkout.idempotencyKey IS NOT NULL
  AND IF existing order with same idempotencyKey exists:
    RETURN existing order (HTTP 200)
    DO NOT create new order
    DO NOT deduct stock again
```

### Enforcement
- Add `idempotencyKey String? @unique` to Divamos `Order` model
- Frontend generates key as `cart:{cartId}:{checksum(items)}` before submitting checkout
- Server checks `findFirst({ idempotencyKey })` before creating order

### Rationale
Without this, a customer who clicks "Pay" twice (or whose browser retries on timeout) gets charged twice and stock is double-deducted.

---

## Invariant 11: Stripe Webhooks MUST Be Deduplicated

```
∀ stripeEvent ∈ incoming webhook events:
  IF stripeEvent.id was already processed:
    RETURN 200 (acknowledge but skip processing)
```

### Enforcement
- Store `stripeEventId` on Order/Payment when processing webhook
- Check `findFirst({ stripeEventId: event.id })` before processing
- Stripe already has `stripePaymentId @unique` on Order — extend to cover `event.id` for idempotent webhook handling

### Rationale
Stripe may redeliver webhook events. Without dedup, stock is decremented again and order processing runs twice.

---

## Invariant 12: BullMQ Job Keys MUST Be Deterministic

```
∀ job ∈ BullMQ syncQueue:
  job.opts.jobId = deterministic_function(eventType, tenantId, entityId, eventId)
  WHERE deterministic_function does NOT include Date.now() or Math.random()
```

### Enforcement
- Change `sync-handlers.ts` key generation from `{type}:{tenantId}:{entityId}:{Date.now()}` to `{type}:{tenantId}:{entityId}:{sourceEventId}`
- `sourceEventId` should be the triggering domain event's ID or the entity's update version

### Rationale
Non-deterministic keys mean the same logical event retried twice produces two different job IDs, bypassing BullMQ's built-in deduplication.

---

## Verification Queries

These SQL queries can be run periodically to detect invariant violations:

### Check for sales without stock deductions (Invariant 2)
```sql
SELECT s.id, s."saleNumber", s."createdAt"
FROM "Sale" s
JOIN "SaleItem" si ON si."saleId" = s.id
LEFT JOIN "InventoryLog" il ON il."referenceType" = 'SALE' AND il."referenceId" = s.id
WHERE s.status IN ('COMPLETED', 'SYNCED')
  AND il.id IS NULL
ORDER BY s."createdAt" DESC
LIMIT 100;
```

### Check for duplicate SyncLog entries (Invariant 4)
```sql
SELECT "tenantId", "clientId", "entityType", "entityId", "clientSeq", COUNT(*)
FROM "SyncLog"
GROUP BY "tenantId", "clientId", "entityType", "entityId", "clientSeq"
HAVING COUNT(*) > 1;
```

### Check for sales without idempotency keys (Invariant 1)
```sql
SELECT id, "saleNumber", "createdAt"
FROM "Sale"
WHERE "idempotencyKey" IS NULL
ORDER BY "createdAt" DESC
LIMIT 100;
```

### Check for negative stock levels (Invariant 3)
```sql
SELECT sl.id, pv.sku, sl.quantity, sl."branchId"
FROM "StockLevel" sl
JOIN "ProductVariant" pv ON pv.id = sl."variantId"
WHERE sl.quantity < 0;
```

---

## Implementation Priority

| Invariant | Severity | Effort | Priority |
|-----------|----------|--------|----------|
| INV-1 (Sale idempotency key required) | CRITICAL | Small | P0 |
| INV-2 (Sale must deduct stock) | CRITICAL | Medium | P0 |
| INV-3 (Optimistic locking on stock) | CRITICAL | Medium | P0 |
| INV-4 (SyncLog DB constraint) | CRITICAL | Small | P0 |
| INV-10 (Divamos checkout idempotency) | HIGH | Small | P1 |
| INV-11 (Stripe webhook dedup) | HIGH | Small | P1 |
| INV-12 (BullMQ deterministic key) | HIGH | Small | P1 |
| INV-6 (Tenant isolation) | CRITICAL | Verify | P0 (audit) |
| INV-7 (Branch scoping) | HIGH | Verify | P1 (audit) |
| INV-8 (WebSocket tenant scope) | HIGH | Verify | P1 (audit) |
| INV-9 (Offline conflict resolution) | MEDIUM | Verify | P2 |
| INV-5 (One stock counter) | MEDIUM | Document | P2 |

---

*End of Sync Invariants — Phase 2 deliverable*
