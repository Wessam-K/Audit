# WK POS Enterprise — API Endpoints & Modules Reference

> Last Updated: 2026-02-16  
> Complete listing of every API endpoint across all three backend systems.  
> Total: **820+ API endpoints** across **60+ modules**.

---

## Table of Contents

1. [WK POS API (600+ endpoints)](#1-wk-pos-api)
2. [Divamos E-Commerce API (98 endpoints)](#2-divamos-e-commerce-api)
3. [License Server API (119 endpoints)](#3-license-server-api)

---

## 1. WK POS API

**Base URL:** `http://localhost:5000/api`  
**Authentication:** JWT Bearer tokens  
**Multi-tenant:** All requests scoped to authenticated tenant  
**Port:** 5000

### Module Summary

| # | Module | Endpoints | Description |
|---|--------|-----------|-------------|
| 1 | Auth | 11 | Login, register, password reset, profile |
| 2 | Dashboard | 4 | Business overview & statistics |
| 3 | Branches | 15 | Store location management & sync |
| 4 | Categories | 5 | Product categories |
| 5 | Products | 12 | Product catalog management |
| 6 | Product Tags | 7 | Product labeling |
| 7 | Sales | 8 | Point of sale transactions |
| 8 | Held Sales | 6 | Parked/held sale management |
| 9 | Sale PDF | 1 | Receipt/invoice PDF generation |
| 10 | Payments | 13 | Payment processing & webhooks |
| 11 | Parked Sales | 6 | Alternative held sale management |
| 12 | Exchanges | 3 | Product exchange processing |
| 13 | Inventory | 19 | Stock levels, movements, alerts |
| 14 | Reservations | 11 | Stock reservation management |
| 15 | Adjustments | 5 | Stock adjustment records |
| 16 | Batches | 8 | Batch/lot tracking |
| 17 | Backorders | 7 | Backorder management |
| 18 | Serials | 12 | Serialized item tracking |
| 19 | Stock Transfers | 8 | Inter-branch stock transfers |
| 20 | Stock Logs | 6 | Stock change history |
| 21 | Stock Audit | 6 | Audit trail for stock |
| 22 | Reconciliation | 6 | Inventory reconciliation |
| 23 | Reorder | 6 | Reorder point management |
| 24 | Item Movements | 4 | Item movement analytics |
| 25 | Purchase Orders | 5 | Purchase order management |
| 26 | Suppliers | 9 | Supplier management |
| 27 | Shifts | 9 | Cashier shift management |
| 28 | Users | 8 | User account management |
| 29 | Tenants | 9 | Tenant/business management |
| 30 | Employees | 9 | Employee profiles |
| 31 | Attendance | 16 | Clock in/out, breaks, reports |
| 32 | Commissions | 11 | Sales commission management |
| 33 | Customers | 5 | Customer profiles |
| 34 | Customer Groups | 11 | Group pricing & management |
| 35 | Settings | 14 | System settings & security |
| 36 | Coupons | 9 | Coupon management |
| 37 | Discounts | 8 | Discount rules |
| 38 | Discount Stacking | 6 | Discount combining rules |
| 39 | Loyalty | 14 | Loyalty program & tiers |
| 40 | Gift Cards | 10 | Gift card management |
| 41 | Barcode | 10 | Barcode generation & validation |
| 42 | Modifiers | 7 | Product modifier templates |
| 43 | Accounting | 25 | GL, journals, expenses, assets, bank |
| 44 | Reports | 29 | Business reports & analytics |
| 45 | Scheduled Reports | 10 | Automated report scheduling |
| 46 | Report Builder | 14 | Custom report creation |
| 47 | Stocktaking | 16 | Physical inventory counts |
| 48 | Webhooks | 8 | Outbound webhook management |
| 49 | Website Sync | 11 | E-commerce sync configuration |
| 50 | MFA | 6 | Multi-factor authentication |
| 51 | Sync | 24 | Branch sync & conflict resolution |
| 52 | Printing | 6 | Receipt & printer configuration |
| 53 | Batch Operations | 5 | Bulk data operations |
| 54 | Permissions | 8 | Permission management |
| 55 | RBAC | 30 | Role-based access control |
| 56 | Approvals | 8 | Approval workflows |
| 57 | Audit | 6 | Security audit & session management |
| 58 | API Keys | 7 | External API key management |
| 59 | Custom Fields | 12 | User-defined field management |
| 60 | CRM | 34 | Leads, deals, activities, quotations |
| 61 | HR & Payroll | 26 | Leave, salary, payslips |
| 62 | E-Commerce Bridge | 18 | Website integration |
| 63 | Hardware | 30 | Devices, printers, scales, drawers |
| 64 | ETA (E-Invoicing) | 7 | Electronic invoice submission |
| 65 | Integrations | 18 | ERP, accounting, logistics, CRM sync |
| 66 | Feature Flags | 8 | Feature toggle management |
| 67 | Network | 13 | Network status & simulation |
| 68 | Preferences | 7 | User preference management |
| 69 | Notifications | 4 | In-app notification management |
| 70 | Compliance | 22 | GDPR, PCI, data retention |
| 71 | Concurrency Testing | 19 | Stress test endpoints |
| 72 | Access Control | 4 | Admin access management |

---

### Detailed Endpoint Listing

#### 1. Authentication (`/api/auth`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| POST | /register | Rate limited | Register a new user account |
| POST | /register-tenant | Rate limited | Register a new business (tenant) |
| POST | /login | Rate limited | Log in and get JWT token |
| POST | /forgot-password | Rate limited | Request password reset email |
| POST | /reset-password | Rate limited | Reset password with token |
| POST | /logout | Authenticated | Log out and invalidate token |
| POST | /refresh-token | Rate limited | Get new JWT from refresh token |
| GET | /me | Authenticated | Get current user profile |
| PATCH | /update-password | Authenticated | Change own password |
| PATCH | /update-profile | Authenticated | Update own profile details |
| POST | /admin-reset-password | Authenticated | Admin resets another user's password |

#### 2. Dashboard (`/api/dashboard`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | / | dashboard:view | Main dashboard overview |
| GET | /stats | dashboard:view | Key business statistics |
| GET | /quick-stats | dashboard:view | Quick summary metrics |
| GET | /branch-comparison | dashboard:view | Compare branches side by side |

#### 3. Branches (`/api/branches`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | / | Authenticated | List all branches |
| GET | /:id | Authenticated | Get branch details |
| POST | / | ADMIN, MANAGER + plan limit | Create new branch |
| PUT | /:id | ADMIN, MANAGER | Update branch info |
| DELETE | /:id | ADMIN | Soft-delete branch |
| DELETE | /:id/permanent | ADMIN | Permanently delete branch |
| GET | /main-branch | Authenticated | Get the main/HQ branch |
| POST | /:id/set-main | ADMIN | Set a branch as main |
| GET | /sync/status | ADMIN, MANAGER | Global sync status |
| GET | /:id/sync/status | Authenticated | Branch sync status |
| PATCH | /:id/sync/settings | ADMIN, MANAGER | Update sync settings |
| POST | /:id/sync/force | ADMIN, MANAGER | Force immediate sync |
| POST | /:id/sync/push-master | ADMIN, MANAGER | Push master data to branch |
| GET | /:id/stock | Authenticated | Get branch stock levels |
| PATCH | /:id/stock/:variantId | Authenticated | Update stock for a variant |

#### 4. Products (`/api/products`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | / | products:view | List products (paginated, filtered) |
| GET | /stats | products:view | Product statistics |
| GET | /export | products:view | Export products (CSV/Excel) |
| GET | /:id | products:view | Get single product |
| GET | /:id/variant-grid | products:view | Variant matrix view |
| GET | /categories/list | Authenticated | Quick category list |
| POST | / | ADMIN, MANAGER + plan limit | Create new product |
| POST | /import | Authenticated | Bulk import products |
| PUT | /:id | ADMIN, MANAGER | Update product |
| DELETE | /:id | ADMIN | Delete product |
| POST | /:id/stock | ADMIN, MANAGER, INVENTORY | Adjust stock |
| GET | /movement/:identifier | products:view | Product movement history |

#### 5. Sales (`/api/sales`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | / | Authenticated | List sales (paginated) |
| GET | /sellers | Authenticated | List available sellers |
| GET | /by-invoice/:invoiceNumber | Authenticated | Find sale by invoice number |
| GET | /:id | Authenticated | Get sale details |
| POST | / | sales:create + idempotency | Create new sale (with stock deduction) |
| POST | /:id/cancel | Authenticated | Cancel a sale |
| POST | /:id/refund | Authenticated | Process a refund |
| POST | /:id/void | Authenticated | Void a sale |

#### 6. Inventory (`/api/inventory`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /stock-levels | inventory:view | All stock levels across branches |
| GET | /movements | inventory:view | Stock movement history |
| POST | /adjust | inventory:adjust | Manual stock adjustment |
| GET | /low-stock | inventory:view | Products below reorder point |
| GET | /out-of-stock | inventory:view | Out-of-stock products |
| GET | /reorder-suggestions | inventory:view | Auto-generated reorder suggestions |
| POST | /transfer | inventory:transfer | Initiate inter-branch transfer |
| GET | /transfers | inventory:view | List all transfers |
| POST | /transfers | inventory:transfer | Create transfer request |
| GET | /transfers/:id | inventory:view | Get transfer details |
| POST | /transfers/:id/approve | inventory:transfer | Approve a transfer |
| POST | /transfers/:id/ship | inventory:transfer | Mark transfer as shipped |
| POST | /transfers/:id/receive | inventory:transfer | Receive a transfer |
| POST | /transfers/:id/close | inventory:transfer | Close completed transfer |
| POST | /transfers/:id/cancel | inventory:transfer | Cancel a transfer |
| POST | /transfers/:id/reject | inventory:transfer | Reject a transfer |
| GET | /movements/:variantId/history | inventory:view | Variant movement history |
| GET | /balance/:variantId/:branchId | inventory:view | Stock balance check |
| GET | /alerts | inventory:view | Active stock alerts |

#### 7. Shifts (`/api/shifts`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | / | shifts:view | List all shifts |
| POST | /open | CASHIER, MANAGER, ADMIN | Open a new shift |
| GET | /current | Authenticated | Get current open shift |
| GET | /current/:branchId | Authenticated | Current shift for branch |
| GET | /x-report/:branchId | CASHIER, MANAGER, ADMIN | X-Report (mid-shift summary) |
| POST | /z-report/:branchId | MANAGER, ADMIN | Z-Report (end-of-day close) |
| POST | /:shiftId/close | CASHIER, MANAGER, ADMIN | Close a shift |
| GET | /history | MANAGER, ADMIN | Shift history |
| GET | /:shiftId | MANAGER, ADMIN | Get shift details |

#### 8. Accounting (`/api/accounting`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /accounts | accounting:view | Chart of accounts |
| POST | /accounts | accounting:create | Create GL account |
| PUT | /accounts/:id | accounting:update | Update GL account |
| DELETE | /accounts/:id | accounting:delete | Delete GL account |
| GET | /journal-entries | accounting:view | List journal entries |
| POST | /journal-entries | accounting:create | Create journal entry |
| PUT | /journal-entries/:id | accounting:update | Update journal entry |
| DELETE | /journal-entries/:id | accounting:delete | Delete journal entry |
| GET | /expenses | accounting:view | List expenses |
| POST | /expenses | accounting:create | Record expense |
| GET | /reports/trial-balance | accounting:view | Trial balance report |
| GET | /reports/profit-loss | accounting:view | Profit & loss statement |
| GET | /reports/balance-sheet | accounting:view | Balance sheet |
| GET | /fixed-assets | accounting:view | List fixed assets |
| POST | /fixed-assets | accounting:create | Register fixed asset |
| PUT | /fixed-assets/:id | accounting:update | Update fixed asset |
| POST | /fixed-assets/:id/depreciate | accounting:update | Run depreciation |
| POST | /fixed-assets/:id/dispose | accounting:update | Dispose of asset |
| GET | /cost-centers | accounting:view | List cost centers |
| POST | /cost-centers | accounting:create | Create cost center |
| GET | /cost-centers/report | accounting:view | Cost center report |
| GET | /reconciliations | accounting:view | Bank reconciliations |
| POST | /reconciliations | accounting:create | Start reconciliation |
| POST | /reconciliations/:id/complete | accounting:update | Complete reconciliation |
| GET | /advanced/meta | Authenticated | Advanced accounting metadata |

#### 9. Reports (`/api/reports`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /summary | reports:view | Overall business summary |
| GET | /sales | reports:view | Sales report |
| GET | /profit-loss | reports:view | Profit & loss report |
| GET | /inventory | reports:view | Inventory report |
| GET | /customers | reports:view | Customer analysis |
| GET | /dashboard | dashboard:view | Dashboard charts data |
| GET | /sales-by-category | reports:view | Sales breakdown by category |
| GET | /sales-by-payment | reports:view | Sales by payment method |
| GET | /cogs | reports:view | Cost of goods sold |
| GET | /top-products | reports:view | Top selling products |
| GET | /branch-performance | reports:view | Branch comparison |
| GET | /vat | reports:view | VAT report |
| GET | /sales-by-sku | reports:view | Sales by SKU |
| GET | /sales-by-hour | reports:view | Hourly sales analysis |
| GET | /employee-performance | reports:view | Employee performance |
| GET | /sales-by-branch | reports:view | Sales by branch |
| GET | /low-stock-alert | reports:view | Low stock alerts report |
| GET | /sales-trend | reports:view | Sales trend analysis |
| GET | /category-performance | reports:view | Category performance |
| GET | /customer-purchase-history | reports:view | Customer purchase history |
| GET | /tax-summary | reports:view | Tax summary report |
| GET | /profit-by-product | reports:view | Profit margin by product |
| GET | /cashier-performance | reports:view | Cashier performance |
| GET | /export/sales | reports:export | Export sales data |
| GET | /export/inventory | reports:export | Export inventory data |
| GET | /export/customers | reports:export | Export customer data |
| GET | /export/products | reports:export | Export product data |
| GET | /export/profit-loss | reports:export | Export P&L data |
| GET | /vat/export | reports:export | Export VAT data |

#### 10. CRM (`/api/crm`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /leads | Authenticated | List sales leads |
| POST | /leads | Authenticated | Create new lead |
| GET | /leads/:id | Authenticated | Get lead details |
| GET | /leads/pipeline | Authenticated | Pipeline view |
| PUT | /leads/:id | Authenticated | Update lead |
| POST | /leads/:id/convert | Authenticated | Convert lead to deal |
| DELETE | /leads/:id | Authenticated | Delete lead |
| GET | /deal-stages | Authenticated | List pipeline stages |
| POST | /deal-stages | Authenticated | Create stage |
| POST | /deal-stages/reorder | Authenticated | Reorder stages |
| PUT | /deal-stages/:id | Authenticated | Update stage |
| DELETE | /deal-stages/:id | Authenticated | Delete stage |
| GET | /deals | Authenticated | List deals |
| GET | /deals/pipeline | Authenticated | Deals pipeline view |
| POST | /deals | Authenticated | Create deal |
| GET | /deals/:id | Authenticated | Get deal details |
| PUT | /deals/:id | Authenticated | Update deal |
| POST | /deals/:id/move | Authenticated | Move deal to stage |
| DELETE | /deals/:id | Authenticated | Delete deal |
| GET | /activities | Authenticated | List CRM activities |
| GET | /activities/upcoming | Authenticated | Upcoming activities |
| POST | /activities | Authenticated | Create activity |
| GET | /activities/:id | Authenticated | Get activity details |
| PUT | /activities/:id | Authenticated | Update activity |
| POST | /activities/:id/complete | Authenticated | Mark activity complete |
| DELETE | /activities/:id | Authenticated | Delete activity |
| GET | /quotations | Authenticated | List quotations |
| POST | /quotations | Authenticated | Create quotation |
| GET | /quotations/:id | Authenticated | Get quotation |
| PUT | /quotations/:id | Authenticated | Update quotation |
| POST | /quotations/:id/send | Authenticated | Send quotation to customer |
| POST | /quotations/:id/convert | Authenticated | Convert quotation to sale |
| DELETE | /quotations/:id | Authenticated | Delete quotation |
| GET | /dashboard | Authenticated | CRM dashboard overview |

#### 11. HR & Payroll (`/api/hr-payroll`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /leave-types | Authenticated | List leave types |
| POST | /leave-types | Authenticated | Create leave type |
| GET | /leave-balances | Authenticated | Leave balances |
| GET | /leave-balances/:employeeId | Authenticated | Employee leave balance |
| GET | /leave-requests | Authenticated | List leave requests |
| POST | /leave-requests | Authenticated | Submit leave request |
| GET | /leave-requests/:id | Authenticated | Get leave request |
| POST | /leave-requests/approve | Authenticated | Approve leave |
| POST | /leave-requests/reject | Authenticated | Reject leave |
| GET | /salary-components | Authenticated | Salary components |
| POST | /salary-components | Authenticated | Create salary component |
| POST | /salary-components/:id/toggle | Authenticated | Toggle component |
| GET | /payroll-periods | Authenticated | Payroll periods |
| POST | /payroll-periods | Authenticated | Create period |
| POST | /payroll-periods/:id/open | Authenticated | Open period |
| POST | /payroll-periods/:id/close | Authenticated | Close period |
| POST | /payroll-periods/:id/lock | Authenticated | Lock period |
| GET | /payslips | Authenticated | List payslips |
| POST | /payslips/generate | Authenticated | Generate payslips |
| GET | /payslips/:id | Authenticated | Get payslip |
| PUT | /payslips/:id | Authenticated | Update payslip |
| POST | /payslips/:id/finalize | Authenticated | Finalize payslip |
| GET | /dashboard | Authenticated | HR dashboard |
| GET | /payroll-summary | Authenticated | Payroll summary |

#### 12. Hardware (`/api/hardware`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| GET | /devices | Authenticated | List hardware devices |
| GET | /devices/:deviceId | Authenticated | Get device details |
| POST | /devices | ADMIN | Register device |
| DELETE | /devices/:deviceId | ADMIN | Remove device |
| POST | /devices/:deviceId/test | Authenticated | Test device |
| POST | /print | Authenticated | Send print job |
| POST | /drawer/open | Authenticated | Open cash drawer |
| GET | /scale/weight | Authenticated | Read scale weight |
| POST | /scanner/parse | Authenticated | Parse scanned barcode |
| POST | /scanner/parse-weighted | Authenticated | Parse weighted barcode |
| POST | /scanner/batch | Authenticated | Batch barcode scan |
| POST | /printer/format-receipt | Authenticated | Format receipt |
| POST | /printer/print | Authenticated | Print receipt |
| POST | /printer/print-multi | Authenticated | Multi-copy print |
| POST | /display/show-item | Authenticated | Show item on customer display |
| POST | /display/payment-prompt | Authenticated | Payment prompt on display |
| POST | /cash-drawer/open | CASHIER+ | Open cash drawer |
| POST | /cash-drawer/monitor | MANAGER+ | Monitor drawer status |
| POST | /cash-drawer/no-sale-open | MANAGER+ | No-sale drawer open |
| POST | /cash-drawer/security-check | MANAGER+ | Security check |
| POST | /scale/calculate | Authenticated | Calculate weight price |
| POST | /scale/stabilize | Authenticated | Wait for stable weight |
| POST | /scale/tare | Authenticated | Tare scale |
| POST | /kds/route-order | MANAGER+ | Route to kitchen display |
| POST | /kds/print-with-fallback | MANAGER+ | Print with KDS fallback |
| POST | /kds/bump-order | CASHIER+ | Bump order from KDS |
| POST | /kds/sla-check | MANAGER+ | Check kitchen SLA |

#### 13. Sync (`/api/sync`)

| Method | Path | Permission | What It Does |
|--------|------|-----------|--------------|
| POST | /push | sync:push | Push data to server |
| POST | /pull | sync:pull | Pull data from server |
| GET | /status | sync:read | Sync status overview |
| POST | /resolve-conflict | sync:resolve | Resolve sync conflict |
| POST | /check-conflict | sync:read | Check for conflicts |
| POST | /resolve | sync:resolve | Resolve conflict (v2) |
| POST | /queue | sync:push | Queue sync operation |
| GET | /queue/:queueId/recover | sync:read | Recover failed queue |
| POST | /queue/process | sync:push | Process sync queue |
| POST | /queue/simulate-failure | sync:push | Simulate queue failure |
| POST | /queue/consolidate | sync:push | Consolidate queue items |
| GET | /connectivity | sync:read | Check connectivity |
| POST | /offline-mode | sync:push | Enter offline mode |
| POST | /simulate-reconnect | sync:push | Simulate reconnection |
| POST | /branch-conflict | sync:resolve | Branch-level conflict |
| POST | /global-override | ADMIN only | Global override sync |
| GET | /dashboard | sync:read | Sync dashboard |
| POST | /branch-transactions | sync:push | Sync branch transactions |
| GET | /master-data/:branchId | sync:read | Get master data package |
| POST | /conflicts/:conflictId/resolve | sync:resolve | Resolve specific conflict |
| POST | /queue-operation | sync:push | Queue single operation |
| GET | /pending/:branchId | sync:read | Pending sync items |
| POST | /heartbeat | sync:read | Branch heartbeat |
| GET | /data-categories | Public | Available data categories |

*(Additional modules: Compliance, Concurrency Testing, Feature Flags, Network, Preferences, Notifications, RBAC, Permissions, Approvals, Audit, Webhooks, Website Sync, MFA, Barcode, Modifiers, Coupons, Discounts, Loyalty, Gift Cards, Stocktaking, Report Builder, Scheduled Reports — see module summary table for counts)*

---

## 2. Divamos E-Commerce API

**Base URL:** `http://localhost:4000/api`  
**Authentication:** JWT Bearer tokens (customer + admin)  
**Port:** 4000

### Public Storefront Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /products | Browse products (paginated, filtered) |
| GET | /products/:slug | Get product by URL slug |
| GET | /collections | Browse collections |
| GET | /collections/:slug | Get collection by slug |
| GET | /categories | Browse categories |
| GET | /search | Search products |
| GET | /storefront/settings/homepage | Get homepage layout |
| GET | /storefront/settings/page/:slug | Get custom page layout |

### Cart Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /cart | Create new shopping cart |
| GET | /cart/:id | Get cart contents |
| POST | /cart/:id/items | Add item to cart |
| PATCH | /cart/:id/items/:itemId | Update cart item quantity |
| DELETE | /cart/:id/items/:itemId | Remove item from cart |

### Checkout Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /checkout/calculate | Calculate order totals (with discounts/tax) |
| POST | /checkout | Place order (with idempotency protection) |

### Customer Auth Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /auth/register | Register customer account |
| POST | /auth/login | Customer login |
| POST | /auth/google | Google OAuth login |
| POST | /auth/refresh | Refresh JWT token |
| POST | /auth/logout | Logout |
| POST | /auth/logout-all | Logout all devices |
| GET | /auth/sessions | List active sessions |
| GET | /auth/me | Get profile |
| PATCH | /auth/profile | Update profile |
| POST | /auth/addresses | Add shipping address |
| PUT | /auth/addresses/:id | Update address |
| DELETE | /auth/addresses/:id | Delete address |
| POST | /auth/forgot-password | Request password reset |
| POST | /auth/reset-password | Reset password |
| GET | /auth/orders | My order history |

### Order Tracking

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /orders/track | Track order by number + email |
| GET | /orders/:id | Get order details |

### Discount Validation

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /discounts/validate | Check if discount code is valid |

### Admin Endpoints (require admin login)

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /auth/admin/login | Admin login |
| **Products** | | |
| GET | /admin/products | List all products |
| GET | /admin/products/:id | Get product |
| POST | /admin/products | Create product |
| PATCH | /admin/products/:id | Update product |
| POST | /admin/products/:id/publish | Publish product |
| POST | /admin/products/:id/unpublish | Unpublish product |
| POST | /admin/products/bulk-publish | Bulk publish products |
| DELETE | /admin/products/:id | Delete product |
| **Orders** | | |
| GET | /admin/orders | List orders |
| GET | /admin/orders/:id | Get order details |
| PATCH | /admin/orders/:id | Update order status |
| **Customers** | | |
| GET | /admin/customers | List customers |
| **Inventory** | | |
| GET | /admin/inventory | Stock levels |
| PATCH | /admin/inventory/:id | Update stock |
| **Categories** | | |
| GET | /admin/categories | List categories |
| POST | /admin/categories | Create category |
| **Analytics** | | |
| GET | /admin/analytics | Store analytics |
| **Discounts** | | |
| GET | /admin/discounts | List discounts |
| POST | /admin/discounts | Create discount |
| PATCH | /admin/discounts/:id | Update discount |
| DELETE | /admin/discounts/:id | Delete discount |
| **Store Settings (CMS)** | | |
| GET | /admin/settings/homepage | Homepage settings |
| PUT | /admin/settings/homepage | Update homepage |
| GET | /admin/settings/categories | Category page settings |
| PUT | /admin/settings/categories | Update category settings |
| GET | /admin/settings/page/:slug | Get page settings |
| PUT | /admin/settings/page/:slug | Update page settings |
| GET | /admin/settings/pages | List all pages |
| GET | /admin/settings/store | Store config |
| PUT | /admin/settings/store | Update store config |
| **Media** | | |
| GET | /admin/media | List media files |
| POST | /admin/media/upload | Upload media |
| PATCH | /admin/media/:id | Update media |
| DELETE | /admin/media/:id | Delete media |
| **API Keys** | | |
| GET | /admin/api-keys | List API keys |
| GET | /admin/api-keys/:id/reveal | Reveal key value |
| POST | /admin/api-keys/generate | Generate new key |
| POST | /admin/api-keys/:id/regenerate | Regenerate key |
| DELETE | /admin/api-keys/:id | Delete key |
| **Stats** | | |
| GET | /admin/stats | Dashboard statistics |

### Retail Sync Endpoints (POS ↔ E-Commerce)

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /retail/register | Register POS connection |
| GET | /retail/configs | List sync configs |
| PATCH | /retail/configs/:id | Update sync config |
| DELETE | /retail/configs/:id | Delete sync config |
| GET | /retail/logs | View sync logs |
| GET | /retail/stats | Sync statistics |
| PUT | /retail/import/products | Import products from POS |
| PUT | /retail/import/categories | Import categories from POS |
| PUT | /retail/import/coupons | Import coupons from POS |
| PUT | /retail/import/gift-cards | Import gift cards from POS |
| PUT | /retail/import/loyalty | Import loyalty program from POS |
| POST | /retail/sync/products | Sync product updates |
| POST | /retail/sync/inventory | Sync inventory levels |
| GET | /retail/orders/pending | Pending orders for POS |
| POST | /retail/orders/:notificationId/ack | Acknowledge order notification |
| POST | /retail/orders/:orderId/fulfill | Fulfill order from POS |
| POST | /retail/webhook | Receive POS webhook events |

---

## 3. License Server API

**Base URL:** `http://localhost:3100/api`  
**Authentication:** API Key or Admin JWT  
**Port:** 3100

### Public License Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /license/validate | Validate a license key |
| POST | /license/check | Quick license check |
| GET | /license/:id/info | Get license information |
| GET | /license/tenant/:tenantId | Get tenant's license |
| POST | /license/validate-serial | Validate serial key |
| POST | /license/register-tenant | Register tenant with serial |

### Activation Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /activation | Activate license on device |
| POST | /activation/deactivate | Deactivate device |
| POST | /activation/offline-request | Request offline activation |
| POST | /activation/offline | Complete offline activation |

### Heartbeat Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /heartbeat | Send device heartbeat |
| GET | /heartbeat/devices/:licenseId | Get device heartbeat status |

### Offline License Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /offline/generate | Generate offline token |
| POST | /offline/validate | Validate offline token |
| POST | /offline/challenge | Issue offline challenge |

### Trial Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /trial/request | Request a trial license |
| POST | /trial/verify | Verify trial status |
| GET | /trial/status/:requestId | Check trial request status |

### Plan Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /plans | List available plans (public) |
| GET | /plans/:id | Get plan details |
| GET | /plans/admin/all | Admin: List all plans |
| POST | /plans/admin | Admin: Create plan |
| PUT | /plans/admin/:id | Admin: Update plan |
| DELETE | /plans/admin/:id | Admin: Delete plan |
| PATCH | /plans/admin/:id/toggle | Admin: Toggle plan active |

### Module Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /modules/available | List available modules |
| GET | /modules | List all modules |
| GET | /modules/:id | Get module details |
| POST | /modules | Create module |
| PUT | /modules/:id | Update module |
| DELETE | /modules/:id | Delete module |
| POST | /modules/:moduleId/permissions | Add permission to module |
| PUT | /modules/:moduleId/permissions/:id | Update permission |
| DELETE | /modules/:moduleId/permissions/:id | Delete permission |
| POST | /modules/:moduleId/permissions/bulk | Bulk add permissions |
| POST | /modules/license/:licenseId/assign | Assign module to license |
| GET | /modules/license/:licenseId | Get license modules |
| GET | /modules/validate/:serialKey | Validate module access |

### Admin Dashboard Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /admin/login | Admin login |
| GET | /admin/dashboard | Dashboard statistics |
| GET | /admin/licenses | List all licenses |
| POST | /admin/licenses | Create new license |
| GET | /admin/licenses/:id | Get license details |
| PUT | /admin/licenses/:id | Update license |
| DELETE | /admin/licenses/:id | Delete license |
| POST | /admin/licenses/:id/revoke | Revoke license |
| POST | /admin/licenses/:id/extend | Extend license |
| POST | /admin/licenses/:id/activate | Activate license |
| POST | /admin/licenses/:id/suspend | Suspend license |
| POST | /admin/licenses/:id/renew | Renew license |
| POST | /admin/licenses/:id/convert | Convert trial to paid |
| GET | /admin/licenses/:id/usage | License usage statistics |
| POST | /admin/serials/generate | Generate serial keys |
| GET | /admin/devices | List all devices |
| GET | /admin/tenants | List all tenants |
| GET | /admin/audit-logs | View audit trail |
| GET | /admin/analytics/revenue | Revenue analytics |
| GET | /admin/analytics/usage | Usage analytics |
| GET | /admin/trial-requests | Trial request management |

### Decision Engine Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| POST | /decision | Make access decision (API key auth) |
| POST | /decision/verify | Verify a previous decision |

### Billing Endpoints (Stripe)

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /billing/status | Get billing status |
| POST | /billing/checkout | Create Stripe checkout session |
| POST | /billing/webhook | Stripe webhook handler |
| GET | /billing/order/:sessionId | Get order by session |
| GET | /billing/orders | List all orders (admin) |

### API Metering Endpoints

| Method | Path | What It Does |
|--------|------|--------------|
| GET | /metering/quotas | List all quotas |
| GET | /metering/quotas/:licenseId | Get tenant quota |
| PUT | /metering/quotas/:licenseId | Update quota limits |
| POST | /metering/quotas/:licenseId/suspend | Suspend API access |
| POST | /metering/quotas/:licenseId/resume | Resume API access |
| POST | /metering/quotas/:licenseId/throttle | Enable throttling |
| POST | /metering/quotas/:licenseId/reset | Reset usage counters |
| GET | /metering/usage/:licenseId | Usage statistics |
| GET | /metering/top-consumers | Top API consumers |
| GET | /metering/summary | Usage summary |
| GET | /metering/alerts | Usage alerts |

---

## Security Middleware

Every request to the WK POS API passes through these middleware layers:

| Layer | Purpose |
|-------|---------|
| **Rate Limiting** | Prevents brute force attacks (different limits per endpoint) |
| **JWT Authentication** | Verifies user identity via Bearer token |
| **License Validation** | Checks tenant license is active and not expired |
| **Tenant Isolation** | Ensures queries only return data for the authenticated tenant |
| **Branch Scoping** | Optionally restricts to user's assigned branch |
| **Permission Check** | Verifies user has required permission (e.g., `reports:view`) |
| **Role Restriction** | Verifies user has required role (ADMIN, MANAGER, etc.) |
| **Plan Limit** | Enforces subscription limits (max users, products, branches) |
| **Idempotency** | Prevents duplicate operations (applied on critical writes) |
| **Request Context** | Enriches request with tenant/branch metadata |
| **Validation** | Validates request body/query/params with Zod schemas |
| **Error Handling** | Catches and formats all errors consistently |
| **Observability** | Records Prometheus metrics for monitoring |
