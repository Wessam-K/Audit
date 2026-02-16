# WK POS Enterprise — Sync & Business Flows

> Last Updated: 2026-02-16  
> This document explains how data moves between systems and the key business workflows.

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Branch Sync Flow](#2-branch-sync-flow)
3. [Edge Agent (Offline Operations)](#3-edge-agent-offline-operations)
4. [POS ↔ E-Commerce Sync](#4-pos--e-commerce-sync)
5. [License Validation Flow](#5-license-validation-flow)
6. [Sale Transaction Flow](#6-sale-transaction-flow)
7. [Inventory Movement Flow](#7-inventory-movement-flow)
8. [Stock Transfer Flow](#8-stock-transfer-flow)
9. [Shift Management Flow](#9-shift-management-flow)
10. [Payment Processing Flow](#10-payment-processing-flow)
11. [Approval Workflow](#11-approval-workflow)

---

## 1. System Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        INTERNET / LAN                            │
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────────────────┐ │
│  │   LICENSE    │    │   WK POS    │    │      DIVAMOS         │ │
│  │   SERVER     │◄──►│    API      │◄──►│   E-COMMERCE         │ │
│  │  Port 3100   │    │  Port 5000  │    │   Port 4000          │ │
│  └──────┬──────┘    └──────┬──────┘    └──────────┬───────────┘ │
│         │                  │                      │              │
│         │           ┌──────┴──────┐               │              │
│         │           │  PostgreSQL │               │              │
│         │           │    + Redis  │               │              │
│         │           └─────────────┘               │              │
│         │                  │                      │              │
│  ┌──────┴──────┐    ┌──────┴──────┐    ┌─────────┴────────────┐│
│  │  Stripe     │    │ Admin Panel │    │   Customer           ││
│  │  Payments   │    │  (React)    │    │   Storefront         ││
│  └─────────────┘    └──────┬──────┘    │   + Mobile App       ││
│                            │           └──────────────────────┘ │
│                     ┌──────┴──────┐                              │
│                     │  Desktop    │                              │
│                     │  (Electron) │                              │
│                     └──────┬──────┘                              │
│                            │                                     │
│              ┌─────────────┼─────────────┐                      │
│              │             │             │                       │
│         ┌────┴────┐  ┌────┴────┐  ┌────┴────┐                  │
│         │ Branch 1│  │ Branch 2│  │ Branch 3│   (Edge Agents)  │
│         │ (Edge)  │  │ (Edge)  │  │ (Edge)  │                  │
│         └─────────┘  └─────────┘  └─────────┘                  │
└──────────────────────────────────────────────────────────────────┘
```

### How the pieces connect:

1. **License Server** — Controls which businesses can use the software, what features they have, and how many devices they can use
2. **WK POS API** — The main backend that handles all business logic (sales, inventory, accounting, etc.)
3. **Divamos** — The customer-facing online store that syncs products and orders with WK POS
4. **Admin Panel** — Web-based dashboard for managers to control everything
5. **Desktop App** — Electron app that wraps the Admin Panel for Windows
6. **Edge Agents** — Small programs running at each branch that handle offline operations

---

## 2. Branch Sync Flow

When a business has multiple store locations (branches), data needs to stay synchronized between them.

### What Gets Synced

| Data Type | Direction | Priority |
|-----------|-----------|----------|
| Products & Categories | HQ → Branches | High |
| Prices & Promotions | HQ → Branches | High |
| Customer Profiles | Bidirectional | Medium |
| Stock Levels | Branch → HQ | High |
| Sales Transactions | Branch → HQ | Critical |
| User Accounts | HQ → Branches | Medium |

### Sync Process

```
Step 1: Branch sends heartbeat to server every 30 seconds
         ──────── "I'm alive, last sync was 5 min ago" ────────►

Step 2: Server checks if there's new data for the branch
         ◄──────── "Yes, 3 product updates available" ──────────

Step 3: Branch pulls the updates
         ──────── POST /api/sync/pull {entities: ['products']} ──►
         ◄──────── [{id: 1, name: 'Updated Product', ...}] ──────

Step 4: Branch pushes its local changes (sales, stock adjustments)
         ──── POST /api/sync/push {sales: [...], stock: [...]} ──►
         ◄──── {success: true, conflicts: []} ────────────────────

Step 5: If conflicts exist, they're queued for resolution
         ◄──── {conflicts: [{entity: 'stock', branchValue: 50, 
                             serverValue: 45, resolution: 'manual'}]}
```

### Conflict Resolution

When two branches modify the same data, the system detects a conflict:

| Conflict Type | Auto-Resolution | Manual Resolution |
|---------------|----------------|-------------------|
| Product name changed at HQ and branch | HQ wins (last-write-wins) | Manager reviews |
| Stock adjusted at two branches | Sum the adjustments | Manager reviews if suspicious |
| Customer updated at two locations | Most recent update wins | Manager reviews |
| Sale recorded offline then server | Both kept (no conflict) | N/A |
| Price changed at HQ during branch sale | Old price honored for open sales | New price takes effect next sale |

---

## 3. Edge Agent (Offline Operations)

The Edge Agent is a small Electron application that runs at each store. It keeps the POS running even when the internet goes down.

### How Offline Mode Works

```
Normal Operation (Online):
┌──────────┐     ┌──────────┐
│  Branch   │◄───►│  Server  │  Real-time data exchange
│  POS App  │     │  (Cloud) │
└──────────┘     └──────────┘

Internet Goes Down:
┌──────────┐    ╳    ┌──────────┐
│  Branch   │◄─ ╳ ──►│  Server  │  Connection lost!
│  POS App  │    ╳    │  (Cloud) │
└─────┬────┘         └──────────┘
      │
      ▼
┌──────────┐
│   Edge   │  Local SQLite database takes over
│  Agent   │  All sales/transactions saved locally
│ (SQLite) │  WAL mode for crash safety
└──────────┘

Internet Restored:
┌──────────┐         ┌──────────┐
│  Branch   │◄──────►│  Server  │
│  POS App  │        │  (Cloud) │
└─────┬────┘         └─────┬────┘
      │                     │
      ▼                     ▼
┌──────────┐         Conflict
│   Edge   │─────►   Resolution
│  Agent   │         & Merge
└──────────┘
```

### What Works Offline

| Feature | Status | Notes |
|---------|--------|-------|
| Process Sales | ✅ Works | Stored locally, synced later |
| Accept Cash | ✅ Works | No network needed |
| Accept Card | ⚠️ Limited | Card terminal may need connection |
| View Products | ✅ Works | Cached locally |
| Check Stock | ✅ Works | Local copy (may be stale) |
| Open/Close Shifts | ✅ Works | Synced when back online |
| Print Receipts | ✅ Works | Direct printer connection |
| View Reports | ⚠️ Limited | Only local data available |
| Manage Users | ❌ Offline | Requires server connection |
| Sync with HQ | ❌ Offline | Queued until connectivity restored |

### Edge Agent Security

- **Offline License Token**: Allows operation for up to 30 days offline (configurable per plan)
- **WAL Mode**: Write-Ahead Logging ensures no data loss even if power goes out
- **Integrity Checks**: Data is checksummed before sync to prevent corruption
- **Automatic Recovery**: On reconnection, data is automatically merged with server

---

## 4. POS ↔ E-Commerce Sync

The WK POS system can sync with the Divamos e-commerce platform to keep the online store in sync with physical stores.

### Sync Configuration

A store owner configures the connection:
```
Retail Sync Config:
- POS System URL: http://192.168.1.100:5000
- API Key: sk-xxxx
- Tenant ID: abc123
- Branch ID: branch-1
- Auto Sync: Enabled
- Sync Interval: 5 minutes
```

### What Gets Synced

```
WK POS ────────────────────────────► Divamos
  Products (name, price, images)  ──►  Products
  Categories                      ──►  Categories
  Stock Levels                    ──►  Variant Stock
  Coupons                         ──►  Coupons
  Gift Cards                      ──►  Gift Cards
  Loyalty Program                 ──►  Loyalty Program
  
Divamos ────────────────────────────► WK POS
  Online Orders                   ──►  Order Notifications
  Customer Data                   ──►  Customer Records
```

### Order Flow (Online Purchase → POS Fulfillment)

```
1. Customer places order on website
   └── Divamos creates Order (status: PENDING)

2. Divamos sends notification to POS
   └── POST /retail/webhook → POS receives order notification

3. POS staff sees "New Online Order" in dashboard
   └── Reviews order details, picks items from inventory

4. POS fulfills the order
   └── POST /retail/orders/:orderId/fulfill
   └── Stock deducted from POS inventory
   └── Order status → PROCESSING → SHIPPED

5. Customer receives delivery
   └── Order status → DELIVERED
```

### Stock Sync Detail

```
POS adjusts stock (sale, return, adjustment)
  │
  ▼
StockLevel.quantity changes in POS database
  │
  ▼
Sync job runs (every 5 minutes or manual trigger)
  │
  ▼
POST /retail/sync/inventory
  Body: [{variantId: "v1", stock: 45}, {variantId: "v2", stock: 12}]
  │
  ▼
Divamos updates ProductVariant.stock for each matched variant
  │
  ▼
Website shows updated stock ("Only 12 left!")
```

---

## 5. License Validation Flow

Every WK POS installation must have a valid license. Here's how it works:

### Initial Activation

```
Step 1: Business owner gets a serial key (e.g., "WKP-XXXX-XXXX-XXXX")
         └── From: Trial request, direct purchase, or Stripe checkout

Step 2: First-time setup in POS
         └── POST /api/license/register-tenant
             {serialKey: "WKP-XXXX-XXXX-XXXX", businessName: "My Store"}

Step 3: License Server validates the serial
         └── Checks: valid format, not already used, not revoked

Step 4: License Server creates tenant record
         └── Tenant ID, plan tier, module access, expiry date

Step 5: Device activation
         └── POST /api/activation
             {licenseId, deviceId, deviceFingerprint}
         └── Server records the device, checks activation limit

Step 6: POS receives license details
         └── Plan tier, modules allowed, feature flags, offline days
```

### Ongoing Validation (Every Request)

```
POS App makes API request
  │
  ▼
License Middleware checks:
  ├── Is the license active? (not expired, suspended, or revoked)
  ├── Is the device activated? (deviceId matches record)
  ├── Has the heartbeat been recent? (within offline grace period)
  └── Does the plan include this module? (e.g., "accounting" module)
  │
  ├── YES → Request proceeds normally
  │
  └── NO → Returns 403 with:
           {error: "License expired" | "Module not available" | "Device limit reached"}
```

### Heartbeat System

```
Every 60 seconds, each POS device sends:
  POST /api/heartbeat
  {
    licenseId: "lic-123",
    deviceId: "dev-456",
    metrics: {
      uptime: 3600,
      salesCount: 15,
      syncStatus: "online"
    }
  }

Server responds:
  {
    status: "active",
    nextHeartbeatIn: 60,
    flags: {renewalDueSoon: true}
  }
```

### Offline Grace Period

If the POS can't reach the license server:
1. **0-24 hours**: Full functionality continues (cached license)
2. **24-72 hours**: Warning banner shown to user
3. **72+ hours**: Restricted mode (POS sales only, no new features)
4. **30+ days** (configurable): System locks, data preserved for sync when reconnected

---

## 6. Sale Transaction Flow

This is the most critical business flow — processing a sale:

```
Step 1: Cashier scans/selects items
         └── Frontend adds items to cart (local state)

Step 2: System calculates totals
         ├── Subtotal = Σ(quantity × price)
         ├── Apply discounts (coupons, rules, loyalty points)
         ├── Calculate tax (inclusive or exclusive per item)
         ├── Apply rounding (to nearest 0.05 if configured)
         └── Total = Subtotal - Discounts + Tax ± Rounding

Step 3: Customer pays
         ├── Single payment (cash, card, etc.)
         └── Split payment (part cash, part card)

Step 4: POST /api/sales (with idempotency key)
         │
         ├── Idempotency Check: Has this sale been processed before?
         │   └── If yes, return the existing sale (no duplicate)
         │
         ├── Begin Database Transaction
         │   ├── Create Sale record
         │   ├── Create SaleItems
         │   ├── Create SalePayments
         │   │
         │   ├── Stock Deduction (per item):
         │   │   ├── Find StockLevel for variant + branch
         │   │   ├── Check version (optimistic locking)
         │   │   ├── Deduct quantity (updateMany WHERE version = X)
         │   │   ├── If version mismatch → 409 Conflict (retry)
         │   │   └── Create InventoryMovement record
         │   │
         │   ├── Handle Serialized Items (if applicable):
         │   │   └── Mark serial numbers as SOLD
         │   │
         │   ├── Update Customer (if registered):
         │   │   ├── Increment totalPurchases
         │   │   ├── Add to totalSpent
         │   │   └── Add loyalty points
         │   │
         │   ├── Create AuditLog entry
         │   └── Commit Transaction
         │
         ├── Generate Invoice Number (sequential per branch)
         │
         ├── Fire Domain Events:
         │   ├── onSaleCompleted → Log activity
         │   └── If configured → Create journal entry (accounting)
         │
         └── Return Sale object to frontend

Step 5: Print receipt
         └── POST /api/printing/:branchId/receipt-data/:saleId
         └── Printer formats and prints

Step 6: Shift totals updated
         └── Cash/card/other sales totals incremented
```

### Sale Cancellation / Refund

```
Cancel: POST /api/sales/:id/cancel
  └── Status → CANCELLED
  └── Stock restored (reverse deduction)
  └── Loyalty points reversed

Refund: POST /api/sales/:id/refund
  └── Status → REFUNDED
  └── Stock restored
  └── Refund payment recorded
  └── If installment plan → adjust remaining balance

Void: POST /api/sales/:id/void
  └── Same day only (before Z-Report)
  └── Status → VOIDED
  └── Stock restored
  └── Treated as if sale never happened
```

---

## 7. Inventory Movement Flow

Every time stock changes, multiple records are created:

```
                    ┌─────────────────┐
                    │  Stock Change    │
                    │  (any reason)    │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │StockLevel│  │Inventory │  │ StockLog │
        │ (update) │  │   Log    │  │ (audit)  │
        └──────────┘  └──────────┘  └──────────┘
              │              │              │
              │              ▼              │
              │       ┌──────────┐         │
              │       │ StockAudit│         │
              │       │  (detail) │         │
              │       └──────────┘         │
              │                             │
              ▼                             ▼
        ┌──────────┐                 ┌──────────┐
        │StockAlert│                 │ Activity │
        │(if low)  │                 │   Log    │
        └──────────┘                 └──────────┘
```

### Types of Inventory Movements

| Movement Type | Trigger | Stock Effect |
|--------------|---------|--------------|
| SALE | Customer purchase | Decrease |
| RETURN | Customer return | Increase |
| PURCHASE | Goods received from supplier | Increase |
| ADJUSTMENT | Manual correction | Increase or Decrease |
| TRANSFER_OUT | Sent to another branch | Decrease |
| TRANSFER_IN | Received from another branch | Increase |
| DAMAGE | Damaged goods written off | Decrease |
| EXPIRED | Expired goods removed | Decrease |
| INITIAL | Initial stock count | Set |
| EXCHANGE_IN | Exchange - received item | Increase |
| EXCHANGE_OUT | Exchange - sent item | Decrease |
| RESERVATION | Stock reserved | Decrease (available) |
| RELEASE | Reservation released | Increase (available) |

---

## 8. Stock Transfer Flow

Moving stock between branches follows a multi-step approval process:

```
Step 1: PENDING
  └── Manager at Branch A creates transfer request
  └── Specifies: items, quantities, destination branch

Step 2: APPROVED (or REJECTED)
  └── Manager at Branch B (or admin) reviews request
  └── Checks stock availability at source branch

Step 3: SHIPPED (IN_TRANSIT)
  └── Branch A picks and packs items
  └── Stock deducted from Branch A

Step 4: RECEIVED (or PARTIALLY_RECEIVED)
  └── Branch B receives the shipment
  └── Counts items, reports any discrepancies
  └── Stock added to Branch B

Step 5: COMPLETED / CLOSED
  └── Transfer finalized
  └── All records updated
```

---

## 9. Shift Management Flow

```
OPEN SHIFT
  └── Cashier: POST /api/shifts/open
  └── Records: opening balance (cash in drawer), user, branch, timestamp

DURING SHIFT
  ├── Process sales → sale totals accumulated
  ├── Cash movements (add/remove cash from drawer)
  │   └── POST /api/shifts/:id/cash-movement {amount, direction, reason}
  └── X-Report (mid-shift summary, non-destructive)
      └── GET /api/shifts/x-report/:branchId

CLOSE SHIFT
  └── Cashier: POST /api/shifts/:id/close
  └── Records: closing balance, calculates variance
  └── System compares: expected vs actual cash
  └── If variance > threshold → alert manager

Z-REPORT (End of Day)
  └── Manager: POST /api/shifts/z-report/:branchId
  └── Closes ALL open shifts for the branch
  └── Generates end-of-day summary
  └── Cannot be reversed (auditable record)
```

---

## 10. Payment Processing Flow

### Cash Payment

```
Customer presents cash
  └── Cashier enters paid amount
  └── System calculates change
  └── Sale completed instantly
  └── Cash drawer opens (if hardware connected)
```

### Card Payment

```
Customer taps/inserts card
  └── POS sends to payment terminal
  └── Terminal processes with bank
  ├── Approved → Sale completed
  └── Declined → Cashier notified, customer retries
```

### Split Payment

```
Customer wants to pay $100 total:
  ├── $60 cash + $40 card
  │   └── Two SalePayment records created
  │   └── Cash portion: method=CASH, amount=60
  │   └── Card portion: method=CARD, amount=40, reference=TXN123
  │
  └── Or any combination of methods:
      CASH + CARD + GIFT_CARD + STORE_CREDIT + MOBILE
```

### Multi-Currency

```
Tourist pays in foreign currency:
  └── System uses configured exchange rate
  └── Calculates equivalent in local currency
  └── Records: foreignAmount, foreignCurrency, exchangeRate
  └── Accounting receives local currency amount
```

---

## 11. Approval Workflow

Some operations require manager/admin approval:

```
HIGH-VALUE TRANSACTIONS:
  Sale > $X → Requires manager PIN/approval
  Discount > Y% → Requires manager approval
  Void/Refund → Requires manager approval

PROCESS:
  1. Cashier initiates action
  2. System checks if approval required
  3. If yes → Creates approval request
  4. Manager receives notification
  5. Manager reviews and approves/rejects
  6. If approved → Action proceeds
  7. If rejected → Cashier notified with reason

PURCHASE ORDERS:
  Draft → Sent → Partially Received → Received
  Each step may require different approval level

STOCK TRANSFERS:
  Pending → Approved → Shipped → Received
  Source and destination managers both involved
```

---

## Data Flow Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA FLOW OVERVIEW                           │
│                                                                  │
│  Customer ──► POS Sale ──► Stock Deduction ──► Accounting Entry │
│     │              │              │                    │          │
│     │              ▼              ▼                    ▼          │
│     │         Commission    Stock Alert          Journal Entry   │
│     │              │              │                    │          │
│     │              ▼              ▼                    ▼          │
│     │         Loyalty      Reorder Suggestion    Trial Balance  │
│     │         Points                                             │
│     │                                                            │
│     └──► Customer Profile ──► CRM Lead ──► Deal ──► Quotation  │
│                                                                  │
│  Supplier ──► Purchase Order ──► Goods Receipt ──► Stock In     │
│     │              │                    │              │          │
│     │              ▼                    ▼              ▼          │
│     │         Supplier           Stock Level     Inventory Log  │
│     │         Payment            Updated                        │
│     │                                                            │
│  Admin ──► Settings ──► Branch Config ──► Sync Config           │
│     │         │                                                  │
│     │         ▼                                                  │
│     │    Audit Log (every action recorded)                      │
│     │                                                            │
│  HR Admin ──► Employee ──► Attendance ──► Payroll ──► Payslip  │
│                   │              │              │                │
│                   ▼              ▼              ▼                │
│              Leave Request  Overtime Calc   Bank Transfer       │
└─────────────────────────────────────────────────────────────────┘
```
