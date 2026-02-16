# WK POS Enterprise — Database Schema Reference

> Last Updated: 2026-02-16  
> This document describes **every** database model and enum across all three backend systems.

---

## Table of Contents

1. [WK POS System](#1-wk-pos-system-128-models)
2. [Divamos E-Commerce](#2-divamos-e-commerce-25-models)
3. [License Server](#3-license-server-24-models)

---

## 1. WK POS System (128+ Models)

The WK POS API uses **PostgreSQL 16** with **Prisma 5.22** as the ORM.  
It supports **multi-tenant** architecture — every business (tenant) has isolated data.

### What is Multi-Tenant?

Think of multi-tenant like an apartment building:
- The **building** is the WK POS system
- Each **apartment** is a separate business (tenant)
- Every tenant has their own data, users, and settings
- They share the same system but cannot see each other's data

### Enums (Value Lists)

Enums are predefined lists of allowed values. For example, a sale can only be in one of these states:

| Enum Name | Allowed Values | Used For |
|-----------|---------------|----------|
| **SaleStatus** | COMPLETED, CANCELLED, REFUNDED, VOIDED | Status of a sale transaction |
| **PaymentMethod** | CASH, CARD, BANK_TRANSFER, MOBILE, CHECK, CREDIT, GIFT_CARD, ONLINE, STORE_CREDIT, OTHER | How a customer pays |
| **UserRole** | APP_OWNER, ADMIN, MANAGER, CASHIER, ACCOUNTANT, HR, INVENTORY | What a user can do |
| **ShiftStatus** | OPEN, CLOSED | Whether a cashier shift is active |
| **PurchaseOrderStatus** | DRAFT, SENT, PARTIALLY_RECEIVED, RECEIVED, CANCELLED | Purchase order lifecycle |
| **StockTransferStatus** | PENDING, APPROVED, REJECTED, IN_TRANSIT, PARTIALLY_RECEIVED, RECEIVED, CANCELLED, COMPLETED, SHIPPED, CLOSED | Inter-branch transfer lifecycle |
| **InventoryMovementType** | PURCHASE, SALE, ADJUSTMENT, TRANSFER_IN, TRANSFER_OUT, RETURN, DAMAGE, EXPIRED, INITIAL, EXCHANGE_IN, EXCHANGE_OUT | Why stock changed |
| **JournalEntryStatus** | DRAFT, POSTED, VOIDED | Accounting journal state |
| **CustomerType** | INDIVIDUAL, COMPANY | Customer classification |
| **AccountType** | ASSET, LIABILITY, EQUITY, REVENUE, EXPENSE | Chart of accounts type |
| **StockAdjustmentReason** | DAMAGE, THEFT, EXPIRY, CORRECTION, RECOUNT, RETURN, OTHER | Why stock was adjusted |
| **DiscountType** | PERCENTAGE, FIXED, BUY_X_GET_Y | How discounts are calculated |
| **TaxType** | INCLUSIVE, EXCLUSIVE | Tax calculation method |
| **CommissionRuleType** | PERCENTAGE, FIXED, TIERED | How commissions are calculated |
| **InstallmentFrequency** | WEEKLY, BIWEEKLY, MONTHLY, QUARTERLY | Payment plan frequency |
| **InstallmentStatus** | ACTIVE, COMPLETED, DEFAULTED, CANCELLED | Installment plan state |
| **QuotationStatus** | DRAFT, SENT, ACCEPTED, REJECTED, EXPIRED, CONVERTED | Quote lifecycle |
| **LeadStatus** | NEW, CONTACTED, QUALIFIED, PROPOSAL, NEGOTIATION, WON, LOST | CRM lead pipeline |
| **LeaveRequestStatus** | PENDING, APPROVED, REJECTED, CANCELLED | Employee leave request |
| **StockCountStatus** | DRAFT, IN_PROGRESS, COMPLETED, CANCELLED, APPROVED, POSTED | Physical stock count |
| **SerialStatus** | AVAILABLE, SOLD, RESERVED, DAMAGED, RETURNED, EXPIRED, IN_TRANSIT | Serialized item state |
| **ReservationStatus** | ACTIVE, FULFILLED, RELEASED, EXPIRED, CANCELLED | Stock reservation state |
| **AlertSeverity** | LOW, MEDIUM, HIGH, CRITICAL | Stock alert urgency |
| **SyncDirection** | PUSH, PULL, BIDIRECTIONAL | Data sync direction |
| **LoyaltyProgramType** | POINTS, CASHBACK, TIERED | Loyalty program type |
| **DepreciationMethod** | STRAIGHT_LINE, DECLINING_BALANCE, UNITS_OF_PRODUCTION | Asset depreciation |
| **PayslipStatus** | DRAFT, FINALIZED, PAID | Employee payslip state |
| **MfaMethodType** | TOTP, EMAIL | Two-factor authentication type |

*(60+ enums total — above are the most important ones)*

---

### Core Business Models

#### Tenant & Users

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Tenant** | A business/company using the system | Name, slug, domain, serial key, license status, plan tier, max branches/users/products, expiry date |
| **User** | A person who logs into the system | Email, name, PIN (for POS), role, branch assignment, MFA settings, login attempt tracking, session version |
| **UserLoginHistory** | Record of every login attempt | IP address, device info, success/failure, timestamp |
| **UserDevice** | Registered devices per user | Device ID, name, trust status |
| **PasswordHistory** | Past passwords (prevents reuse) | Hashed password, creation date |
| **Branch** | Physical store location | Name, address, phone, whether it's the main branch |

#### Products & Inventory

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Category** | Product categories (e.g., "Electronics", "Food") | Name (English + Arabic), parent category for nesting, sort order |
| **Product** | A product in the catalog | Name, SKU, barcode, cost/selling/wholesale/min prices, tax rate, reorder level, weight, unit |
| **ProductVariant** | A specific version of a product (e.g., "Red, Size L") | SKU, barcode, individual prices, weight, active status |
| **ProductTag** | Labels for organizing products | Tag name (e.g., "Best Seller", "New Arrival") |
| **ProductAttribute** | Attribute types (e.g., "Color", "Size") | Attribute name |
| **ProductAttributeValue** | Attribute options (e.g., "Red", "Blue") | Value text |
| **StockLevel** | How much stock exists at each location | Variant, branch, quantity, reserved quantity, version (for concurrency) |
| **StockAlert** | Automatic alerts when stock is low/out | Variant, branch, alert type, severity, message |
| **StockAdjustment** | Manual changes to stock quantities | Variant, branch, quantity change, reason, who did it |
| **Batch** | Product batches with expiry tracking | Batch number, variant, quantity, expiry date, manufacturing date |
| **SerializedItem** | Items with unique serial numbers | Serial number, variant, branch, status (available/sold/reserved) |
| **SerialStatusHistory** | Tracking serial number state changes | From/to status, reason, who changed it |
| **StockReservation** | Temporary stock holds | Variant, branch, quantity, source (POS hold, online order), expiry |
| **StockCount** | Physical inventory counts | Type (full/partial/cycle/spot), branch, status |
| **StockCountItem** | Individual items in a stock count | Expected vs counted quantity, variance |
| **InventoryLog** | Every stock movement recorded | Variant, branch, movement type, quantity before/after |
| **StockAudit** | Detailed audit trail for stock changes | Action, quantity change, cost, sale price reference |
| **StockLog** | Simplified stock change log | Action, quantity before/after, reference |
| **Forecast** | Demand forecasting data | Entity, period, forecast method, predicted vs actual quantity |
| **Backorder** | Items ordered but not yet in stock | Variant, quantity needed, fulfillment status |

#### Sales & Transactions

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Sale** | A completed sale transaction | Invoice number, customer, cashier, branch, shift, subtotal, discount, tax, total, paid amount, change, status |
| **SaleItem** | Individual items in a sale | Variant, product name, quantity, unit price, cost price, discount, tax, total, modifiers applied |
| **SalePayment** | Payment methods used in a sale | Method (cash/card/etc.), amount, reference number, foreign currency support |
| **HeldSale** | Sales put on hold (to resume later) | Customer name, items (JSON), subtotal, notes, expiry |
| **Shift** | Cashier work shift | Opening/closing balance, cash/card/other totals, start/end time |
| **CashMovement** | Cash added or removed during shift | Amount, direction (in/out), reason |
| **Commission** | Sales commission earned | Sale, user, amount, status (pending/approved/paid) |
| **CommissionRule** | Rules for calculating commissions | Type, applies to (all/category/product), value, tiers |

#### Customers & Loyalty

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Customer** | Customer profile | Name, email, phone, type (individual/company), tax number, loyalty points, total purchases/spent |
| **CustomerGroup** | Groups for bulk pricing | Name, description, discount percent |
| **GroupPriceRule** | Special pricing per group | Product/variant, price level, custom price or discount |
| **Coupon** | Discount coupons | Code, type (% or fixed), min purchase, max discount, usage limits, date range |
| **CouponUsage** | Coupon redemption records | Coupon, sale, user, discount amount |
| **DiscountRule** | Automatic discount rules | Name, type, value, conditions, priority, date range |
| **DiscountStackingRule** | Rules for combining discounts | Strategy (best price/compound/sequential), max discounts |
| **Installment** | Installment payment plans | Customer, sale, total/paid/remaining, frequency, status |
| **InstallmentPayment** | Individual installment payments | Due date, paid date, amount, status |
| **LoyaltyProgram** | Loyalty program configuration | Type (points/cashback/tiered), earn rate, redemption value |
| **LoyaltyTransaction** | Points earned/redeemed | Customer, points, type, reference |
| **LoyaltyTier** | Tier levels (e.g., Gold, Platinum) | Min points, multiplier, benefits |
| **LoyaltyPointsBalance** | Current point balance per customer | Current points, lifetime points, tier |

#### Suppliers & Purchasing

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Supplier** | Vendor/supplier profiles | Name, contact, tax number, payment terms |
| **SupplierPayment** | Payments to suppliers | Amount, method, reference, date |
| **PurchaseOrder** | Orders placed with suppliers | Order number, supplier, status, expected date, total |
| **PurchaseOrderItem** | Items in a purchase order | Variant, quantity ordered/received, unit cost |
| **GoodsReceipt** | Received goods from supplier | Purchase order reference, supplier, status |
| **GoodsReceiptItem** | Items received | Variant, quantity, unit cost |

#### Accounting & Finance

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **ChartOfAccount** | General ledger accounts | Code, name (English + Arabic), type (Asset/Liability/etc.), balance |
| **JournalEntry** | Accounting journal entries | Entry number, date, description, status (draft/posted/voided) |
| **JournalEntryLine** | Debit/credit lines in journal entry | Account, debit amount, credit amount |
| **JournalEntryChangeLog** | Audit trail for journal changes | Change type, who changed, old/new values |
| **Expense** | Business expenses | Description, amount, category, date, account |
| **AccountingPeriod** | Fiscal periods | Name, start/end date, closed status |
| **FixedAsset** | Company fixed assets (computers, furniture) | Name, purchase price, current value, depreciation method |
| **AssetDepreciationSchedule** | Depreciation schedule entries | Period, amount, accumulated depreciation, book value |
| **BankAccount** | Company bank accounts | Account number, bank name, type, balance, currency |
| **BankTransaction** | Bank deposits/withdrawals | Date, type, amount, description, reconciled status |
| **BankReconciliation** | Bank reconciliation sessions | Date range, opening/closing balance, status |
| **CostCenter** | Cost centers for expense tracking | Name, code |

#### Employees & HR

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Employee** | Employee profiles | Employee number, position, department, hire date, salary |
| **Attendance** | Clock in/out records | Clock in/out times, break times, hours worked, overtime |
| **LeaveType** | Types of leave (vacation, sick, personal) | Name, days per year, is paid |
| **LeaveRequest** | Leave requests from employees | Leave type, date range, days, status, reason |
| **LeaveBalance** | Leave days balance per employee | Year, total/used/remaining days |
| **SalaryComponent** | Salary components (allowances, deductions) | Name, type, amount/percentage |
| **PayrollPeriod** | Payroll processing periods | Start/end date, status |
| **Payslip** | Employee payslips | Basic/gross/net salary, earnings, deductions, payment method |
| **PayrollEntry** | Simplified monthly payroll | Month, year, basic salary, allowances, deductions, net |
| **PayrollRun** | Payroll batch processing | Month, year, total amount, employee count, status |

#### CRM (Customer Relationship Management)

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **Lead** | Sales leads/prospects | Name, email, company, status, value, assigned sales rep |
| **DealStage** | Pipeline stages (e.g., "Proposal", "Negotiation") | Name, order, win probability |
| **Deal** | Sales deals/opportunities | Name, value, stage, lead, expected close date |
| **Quotation** | Price quotations for customers | Number, customer, items, subtotal, tax, total, valid until |
| **QuotationItem** | Items in a quotation | Product, variant, quantity, unit price, discount |
| **Activity** | CRM activities (calls, emails, meetings) | Type, subject, scheduled/completed date, linked to lead/deal/customer |

#### Web & E-Commerce Bridge

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **WebStorefront** | Online store configurations | Name, domain, active status, settings |
| **WebOrder** | Orders from the online store | Order number, customer info, items, totals, status |
| **WebsiteSyncConfig** | Website sync configurations | URL, API key, sync direction, auto sync settings |
| **WebsiteSyncLog** | Sync operation logs | Operation, status, details |
| **WebsiteOrder** | External website orders | External order ID, fulfillment/payment status |

#### Security & Access Control

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **AuditLog** | Every action recorded for compliance | Action, entity, user, old/new values, IP address |
| **ActivityLog** | User activity tracking | Action type, entity, details |
| **CustomRole** | Custom user roles | Name, permissions list |
| **UserPermission** | User-role assignments | User, custom role |
| **AccessRule** | Fine-grained access rules | Resource, action, conditions, allow/deny |
| **AccessPolicy** | ABAC policies | Name, rules, effect, priority |
| **ResourceACL** | Per-resource access control | Resource type/ID, user/role, permissions |
| **PermissionAuditLog** | Permission check audit trail | User, action, resource, granted/denied, reason |
| **RolePermissionConfig** | Default role permissions | Role, permissions array |
| **ApiKey** | API keys for external integrations | Name, hashed key, permissions, expiry |
| **FeatureFlag** | Feature toggles | Name, enabled status, conditions |
| **IdempotencyKey** | Prevents duplicate operations | Key, response, status code |

#### System & Configuration

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **SystemSetting** | System-wide settings | Key, value, data type, category |
| **BranchPrinterSettings** | Printer config per branch | Default printer, receipt settings, kitchen printer |
| **BranchInvoiceTemplate** | Invoice templates per branch | Paper size, header/footer, logo, barcode, custom CSS |
| **CustomFieldDefinition** | User-defined custom fields | Name, field type, entity type, required |
| **CustomFieldValue** | Custom field data values | Definition, entity, value |
| **UserPreference** | User-specific preferences | Key-value pairs |
| **Notification** | In-app notifications | Title, message, type, read status |
| **Webhook** | Outbound webhook configurations | URL, events, secret, fail count |
| **WebhookLog** | Webhook delivery logs | Event, payload, status code, response |
| **DataRetentionPolicy** | Data retention rules | Entity type, retention days, archive enabled |
| **DataExportLog** | Data export records | Entity type, format, exported by, record count |
| **SyncLog** | Branch sync operation log | Operation, entity, direction, status |
| **SyncBatch** | Sync batch tracking | Batch ID, operation, total/processed items |

---

## 2. Divamos E-Commerce (25+ Models)

The Divamos platform is the **customer-facing online store**. It uses **PostgreSQL** with **Prisma 5.8**.  
It is a **single-tenant** system (one store per installation).

### Enums

| Enum Name | Values | Used For |
|-----------|--------|----------|
| **UserRole** | ADMIN, MANAGER, EDITOR, CUSTOMER | Store user roles |
| **PublishStatus** | DRAFT, PUBLISHED, ARCHIVED | Product visibility |
| **OrderStatus** | PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED, RETURNED | Order lifecycle |
| **PaymentStatus** | PENDING, PAID, REFUNDED, FAILED | Payment state |
| **FulfillmentStatus** | UNFULFILLED, PARTIALLY_FULFILLED, FULFILLED | Order fulfillment |
| **DiscountType** | PERCENTAGE, FIXED_AMOUNT | Discount calculation |
| **GiftCardStatus** | ACTIVE, USED, EXPIRED, CANCELLED | Gift card state |
| **RetailSyncAction** | IMPORT_PRODUCTS, IMPORT_CATEGORIES, SYNC_INVENTORY, SYNC_ORDERS, SYNC_PRICES, IMPORT_COUPONS, IMPORT_GIFT_CARDS, IMPORT_LOYALTY | POS sync actions |
| **NotificationType** | NEW_ORDER, ORDER_UPDATE, LOW_STOCK, SYSTEM | Notification types |

### Models

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **User** | Store users (admins + customers) | Email, name, role, phone |
| **RefreshToken** | JWT refresh tokens | Token, device, IP, expiry, revoked status |
| **Category** | Product categories | Name, slug, image, Arabic name, external POS ID for sync |
| **Product** | Products in the store | Name, slug, price, compare-at price, SKU, publish status, images, tags, external POS ID |
| **ProductVariant** | Product variants (size/color) | SKU, barcode, price, stock, low stock threshold, stock source (manual vs POS synced) |
| **ProductImage** | Product images | URL, alt text, sort order, primary flag |
| **ProductOption** | Product options (e.g., Color choices) | Name, values array |
| **Collection** | Curated product collections | Name, slug, description, image |
| **Cart** | Shopping cart | User, items, subtotal |
| **CartItem** | Items in a cart | Variant, quantity, price |
| **Order** | Customer orders | Order number, status, payment/fulfillment status, totals, addresses, coupon/gift card used, idempotency key |
| **OrderItem** | Items in an order | Variant, name, SKU, quantity, price |
| **Address** | Customer addresses | Street, city, state, country, zip, default flag |
| **InventoryLog** | Stock change records | Previous/new stock, change amount, reason |
| **AuditLog** | Admin action audit trail | User, action, entity, details, IP |
| **Discount** | Discount/coupon codes | Code, type, value, min purchase, max uses, date range |
| **Media** | Uploaded media files | Filename, URL, MIME type, size |
| **PageSettings** | CMS page configurations | Page slug, sections (JSON), active status |
| **RetailSyncConfig** | POS system connection config | POS URL, API key, tenant/branch IDs, what to sync, auto sync settings |
| **RetailSyncLog** | Sync operation history | Action, direction, status, items processed, errors |
| **OrderNotification** | Order notification tracking | Order, type, status, sent timestamp |
| **WebhookEvent** | Processed webhook events (dedup) | Event ID, payload, processed flag |
| **SyncLog** | Detailed sync records | Entity, action, source ID, status, batch info |
| **Coupon** | Coupons (from POS sync) | Code, type, value, POS external ID |
| **GiftCard** | Gift cards (from POS sync) | Code, initial/current balance, status, POS external ID |
| **LoyaltyProgram** | Loyalty program (from POS sync) | Points per currency, point value, min redemption |

---

## 3. License Server (24 Models)

The License Server manages **software licensing**, **activation**, **billing**, and **API metering**.  
It uses **PostgreSQL** with **Prisma 5.7**.

### Enums

| Enum Name | Values | Used For |
|-----------|--------|----------|
| **LicenseStatus** | ACTIVE, EXPIRED, SUSPENDED, REVOKED, TRIAL, GRACE_PERIOD | License state |
| **PlanTier** | TRIAL, BASIC, PROFESSIONAL, ENTERPRISE | Subscription tier |
| **ActivationStatus** | ACTIVE, DEACTIVATED, SUSPENDED | Device activation state |
| **DecisionType** | ALLOW, DENY, THROTTLE, GRACE, OFFLINE_VALID, OFFLINE_EXPIRED | License check result |
| **ViolationType** | LICENSE_EXPIRED, DEVICE_LIMIT, INVALID_LICENSE, TAMPERING, CLOCK_MANIPULATION, OFFLINE_EXPIRED | Security violations |
| **ViolationSeverity** | LOW, MEDIUM, HIGH, CRITICAL | Violation seriousness |
| **TrialRequestStatus** | PENDING, APPROVED, REJECTED, EXPIRED | Trial request state |
| **AdminUserRole** | SUPER_ADMIN, ADMIN, SUPPORT | License server admin roles |

### Models

| Model | What It Stores | Key Information |
|-------|---------------|-----------------|
| **LicensePlan** | Subscription plans (Basic, Pro, Enterprise) | Tier, price, max activations/branches/users/products, offline days, features list |
| **License** | Individual licenses | Serial key, tenant, plan, status, expiry, max activations, grace period, last heartbeat |
| **Activation** | Device activations per license | Device ID, name, IP, status, last seen, offline validity period |
| **OfflineLicense** | Offline license tokens | License, activation, token, issue/expiry dates, revoked status |
| **AuditLog** | Comprehensive audit trail | Event type (25+ types), license, tenant, device, IP, details |
| **AdminUser** | License server administrators | Email, name, role, MFA settings |
| **TrialRequest** | Trial license requests | Business name, email, country, business type, status, approval info |
| **Module** | Licensed feature modules (51 modules) | Name, key (e.g., "pos-core", "inventory"), description |
| **ModulePermission** | Permissions within each module | Module, permission name/key |
| **PlanModule** | Which modules each plan includes | Plan, module, configuration |
| **LicenseModule** | Modules assigned to specific licenses | License, module, active status, configuration |
| **LicenseModulePermission** | Granular permission grants | License module, permission, granted status |
| **FeatureFlag** | Feature toggles | Name, enabled, conditions |
| **SystemSetting** | Server configuration | Key, value, description |
| **LicenseDecision** | License validation decisions (audit) | License, device, decision type, reason, features, latency |
| **Entitlement** | Feature entitlements | License, feature name, active status |
| **OfflineToken** | Challenge-response offline tokens | Activation, token, challenge, expiry |
| **LicenseViolation** | Security violations detected | License, device, type, severity, resolution status |
| **DeviceFingerprint** | Hardware fingerprints for device verification | CPU ID, motherboard ID, disk ID, MAC address, OS info |
| **ApiUsage** | API call records per tenant | License, endpoint, method, status code, response time |
| **ApiQuota** | API rate limits per tenant | Daily/monthly limits, current usage, suspended/throttled flags |
| **ApiUsageSummary** | Aggregated usage statistics | Date, endpoint, total requests, avg response time, error count |
| **AdminApiKey** | API keys for admin access | Name, hashed key, admin user, expiry |
| **LicenseOrder** | Stripe payment orders | License, Stripe session/customer IDs, amount, plan, period |

---

## Relationships Overview

```
┌─────────────────────────────────────────────────────────┐
│                    LICENSE SERVER                         │
│  Plans → Modules → Permissions                          │
│  License → Activations → Devices                        │
│  License → Modules → Permissions                        │
│  TrialRequests → Licenses                               │
│  Billing (Stripe) → LicenseOrders                       │
└─────────────────────┬───────────────────────────────────┘
                      │ validates
                      ▼
┌─────────────────────────────────────────────────────────┐
│                    WK POS API                            │
│  Tenant → Branches → StockLevels                        │
│  Tenant → Users → Sales → SaleItems                     │
│  Tenant → Products → Variants → StockLevels             │
│  Tenant → Customers → Sales                             │
│  Tenant → Suppliers → PurchaseOrders → GoodsReceipts    │
│  Tenant → ChartOfAccounts → JournalEntries              │
│  Tenant → Employees → Attendance / Payroll              │
│  Tenant → Leads → Deals → Activities (CRM)             │
│  Users → Shifts → Sales                                 │
│  Variants → SerializedItems / Batches / Reservations    │
└─────────────────────┬───────────────────────────────────┘
                      │ syncs products/stock/orders
                      ▼
┌─────────────────────────────────────────────────────────┐
│                 DIVAMOS E-COMMERCE                       │
│  Products → Variants → OrderItems                       │
│  Users → Orders → OrderItems                            │
│  Carts → CartItems → Variants                           │
│  RetailSyncConfig → RetailSyncLogs                      │
│  GiftCards / Coupons / Loyalty (from POS)               │
└─────────────────────────────────────────────────────────┘
```
