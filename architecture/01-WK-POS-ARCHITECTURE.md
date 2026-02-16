# WK POS Enterprise — Architecture Document

> **Audit Date:** February 12, 2026  
> **System Value:** $6,000 – $9,000 USD  
> **Status:** Production-Ready  

---

## 1. System Overview

WK POS Enterprise is a full-featured Point of Sale system designed for multi-branch retail operations in the Middle East (Saudi Arabia / Egypt). It consists of three tightly integrated components:

| Component | Technology | Port | Purpose |
|-----------|-----------|------|---------|
| **Admin Panel** | React 18 + Vite + Tailwind CSS | 5173 (dev) | Web-based management dashboard |
| **Backend API** | Express 4 + TypeScript + Prisma | 5000 | RESTful API server |
| **Desktop App** | Electron 28 + SQLite + Local API | 45000 | Offline-capable desktop POS |

---

## 2. Frontend — Admin Panel

**Path:** `wk-pos-system/admin-panel/`

### 2.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| React | 18.2.0 | UI framework |
| React Router DOM | 6.21.2 | Client-side routing (**HashRouter**) |
| TypeScript | 5.3.3 | Type safety |
| Vite | 5.0.11 | Build tool & dev server |
| Zustand | 4.4.7 | State management (8 stores) |
| TanStack React Query | 5.17.0 | Server state & caching |
| TanStack React Table | 8.11.2 | Data tables |
| Tailwind CSS | 3.4.1 | Utility-first CSS |
| Axios | 1.6.5 | HTTP client |
| i18next | 25.7.1 | Internationalization (AR/EN) |
| React Hook Form | 7.49.3 | Form management |
| Zod | 3.22.4 | Schema validation |
| Socket.IO Client | 4.7.4 | Real-time updates |
| Chart.js + Recharts | 4.4.1 / 3.5.1 | Data visualization |
| Lucide React | 0.562.0 | Icon library |

### 2.2 Build Configuration

| Setting | Value |
|---------|-------|
| Base path | `./` (relative, for Electron `file://` compatibility) |
| Path alias | `@` → `./src` |
| Dev server | port 5173, host 0.0.0.0 |
| API proxy | `/api` → `http://localhost:5000` |
| WebSocket proxy | `/socket.io` → `http://localhost:5000` |
| Build output | `dist/` with sourcemaps |

### 2.3 Routing Architecture

Uses **HashRouter** for Electron `file://` protocol compatibility. All routes are prefixed with `/#/`.

#### Complete Route Map (55 pages, 47+ routes)

| Route | Page | Module Guard | Phase |
|-------|------|-------------|-------|
| `/#/login` | LoginPage | None | Core |
| `/#/register` | TenantRegistrationPage | None | Core |
| `/#/pos` | POSPage | ADMIN, MANAGER, CASHIER | Core |
| `/#/` | DashboardRouter | ProtectedRoute | Core |
| `/#/items` | ItemsPage | items | Core |
| `/#/products` | → Redirect to /items | — | Core |
| `/#/categories` | CategoriesPage | categories | Core |
| `/#/suppliers` | SuppliersPage | suppliers | Core |
| `/#/purchase-orders` | PurchaseOrdersPage | purchase-orders | Core |
| `/#/stock-control` | StockControlPage | inventory | Core |
| `/#/stocktaking` | StocktakingPage | stocktaking | Core |
| `/#/product-movement` | ProductMovementPage | inventory | Core |
| `/#/item-movement` | ItemMovementPage | inventory | Core |
| `/#/customers` | CustomersPage | customers | Core |
| `/#/invoices` | InvoicesPage | invoices | Core |
| `/#/invoices/:id` | InvoicesPage (detail) | invoices | Core |
| `/#/accounting` | AccountingPage | accounting | Core |
| `/#/reports` | ReportsPage | reports | Core |
| `/#/users` | UsersPage | users | Core |
| `/#/permissions` | PermissionsPage | permissions | Core |
| `/#/branches` | BranchesPage | branches | Core |
| `/#/settings` | SettingsPage | settings | Core |
| `/#/security` | SecurityPage | security | Core |
| `/#/website-sync` | RetailSyncCenter | marketing | Phase 4 |
| `/#/website-orders` | WebsiteOrdersPage | marketing | Phase 4 |
| `/#/online-payments` | OnlinePaymentsPage | marketing | Phase 4 |
| `/#/hardware` | HardwarePage | hardware | Phase 4 |
| `/#/rbac` | RBACPage | rbac | Phase 4 |
| `/#/scheduled-reports` | ScheduledReportsPage | scheduled-reports | Phase 4 |
| `/#/admin` | AdminDashboardPage | system | Phase 5 |
| `/#/admin/acl` | ACLManagementPage | system | Phase 5 |
| `/#/admin/policies` | ABACPoliciesPage | system | Phase 5 |
| `/#/admin/audit` | AuditLogsPage | system | Phase 5 |
| `/#/admin/access-control` | AccessControlPage | system | Phase 5 |
| `/#/report-builder` | ReportBuilderPage | report-builder | Phase 8 |
| `/#/api-keys` | ApiKeysPage | api-keys | Phase 8 |
| `/#/webhooks` | WebhooksPage | webhooks | Phase 8 |
| `/#/custom-fields` | CustomFieldsPage | custom-fields | Phase 8 |
| `/#/crm` | CRMPage | crm | Phase 8 |
| `/#/hr-payroll` | HRPayrollPage | hr | Phase 8 |
| `/#/ecommerce` | EcommerceBridgePage | ecommerce | Phase 8 |
| `/#/shifts` | ShiftsPage | shifts | Phase 14 |
| `/#/attendance` | AttendancePage | attendance | Phase 14 |
| `/#/commissions` | CommissionsPage | commissions | Phase 14 |
| `/#/coupons` | CouponsPage | coupons | Phase 14 |
| `/#/loyalty` | LoyaltyPage | loyalty | Phase 14 |
| `/#/exchanges` | ExchangesPage | exchanges | Phase 14 |
| `/#/transfers` | StockTransfersPage | transfers | Phase 14 |
| `/#/gift-cards` | GiftCardsPage | giftcards | Phase 14 |
| `/#/system-guide` | SystemGuidePage | None | — |

### 2.4 State Management (Zustand Stores)

| Store | Key | Purpose |
|-------|-----|---------|
| `auth.store.ts` | `wk-auth` | JWT tokens, user, permissions, role-based access |
| `branch.store.ts` | — | Multi-branch selection |
| `license.store.ts` | — | License info & module access control |
| `settings.store.ts` | — | Application settings |
| `preferences.store.ts` | — | User preferences |
| `theme.store.ts` | — | Dark/light/system theme (FOUC prevention) |
| `sync.store.ts` | — | Data sync state |
| `feature-flags.store.ts` | — | Feature toggles |

### 2.5 Authentication Flow

```
User → /#/login → POST /api/v1/auth/login
                     ↓
              { user, token, refreshToken }
                     ↓
              Zustand persist → localStorage["wk-auth"]
                     ↓
              Axios interceptor: Authorization: Bearer <token>
                                 X-Tenant-Id: <tenantId>
                     ↓
              401 → POST /api/v1/auth/refresh-token → retry
```

**Roles:** APP_OWNER, ADMIN, MANAGER, CASHIER, ACCOUNTANT, INVENTORY, SELLER

### 2.6 Key UI Components

| Component | Purpose |
|-----------|---------|
| CommandPalette | Ctrl+K command palette |
| FloatingQuickAccess | Quick action FAB |
| GlobalErrorBoundary | Root error boundary |
| KeyboardShortcuts | Global hotkey provider |
| ModuleProtectedRoute | License module-based route guard |
| TrialBanner | Trial license warning |
| QRCodeDisplay | QR code renderer |
| BranchSetupWizard | Multi-branch onboarding |
| `ui/` directory | Button, Card, Modal, Input, Badge, KPICard, ConfirmDialog, EmptyState, HelpButton, PageHeader, PermissionGate, Skeleton |

---

## 3. Backend — API Server

**Path:** `wk-pos-system/server/api/`

### 3.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Express | 4.18.2 | HTTP framework |
| Prisma | 5.22.0 | ORM (PostgreSQL) |
| TypeScript | 5.3.3 | Language |
| Socket.IO | 4.6.0 | Real-time WebSocket |
| jsonwebtoken | 9.0.2 | JWT auth |
| bcrypt + bcryptjs | 6.0.0 / 2.4.3 | Password hashing |
| helmet | 7.1.0 | Security headers |
| express-rate-limit | 7.1.5 | Rate limiting |
| zod | 3.25.76 | Schema validation |
| multer | 1.4.5 | File upload |
| sharp | 0.33.1 | Image processing |
| pdfkit | 0.14.0 | PDF generation |
| exceljs | 4.4.0 | Excel export |
| qrcode | 1.5.3 | QR code generation |
| node-cron | 3.0.3 | Scheduled jobs |
| nodemailer | 7.0.12 | Email |
| twilio | 4.19.3 | SMS |
| stripe | 14.11.0 | Payment processing |
| otplib | 12.0.1 | MFA (TOTP) |
| ioredis | 5.3.2 | Redis caching |
| prom-client | 15.1.3 | Prometheus metrics |

### 3.2 Server Configuration

| Property | Value |
|----------|-------|
| Port | 5000 |
| API prefix | `/api/v1` |
| API docs | `/api/docs` (Swagger/OpenAPI) |
| Body limit | 50MB |
| Upload limit | 10MB (configurable) |
| Default timezone | Asia/Riyadh |
| Default currency | SAR |
| Default locale | ar |
| VAT rate | 15% |

### 3.3 Middleware Stack (execution order)

1. `helmet()` — Security headers
2. `cors()` — CORS (configurable origins)
3. `compression()` — gzip
4. `express.json({ limit: '50mb' })` — Body parser
5. `express.urlencoded({ extended: true, limit: '50mb' })`
6. `requestContextMiddleware()` — Correlation ID tracking
7. `performanceMiddleware()` — Response time metrics
8. `express.static('/uploads')` — Static files
9. `morgan('combined')` — HTTP logging
10. `idempotencyMiddleware()` — Idempotency key support

### 3.4 Security Middleware

| Middleware | File | Purpose |
|-----------|------|---------|
| JWT Auth | auth.middleware.ts | Token verification, role resolution |
| Permission | permission.middleware.ts | `requirePermission()` — DB-backed ACL |
| Rate Limit | rate-limit.middleware.ts | Auth: 5/15min, Password: 3/hr |
| RBAC | rbac.middleware.ts | Role-based access control |
| Tenant Isolation | tenant-isolation.middleware.ts | Multi-tenant data filtering |
| License | license.middleware.ts | License module validation |
| Idempotency | idempotency.middleware.ts | Duplicate request prevention |

### 3.5 API Route Modules (70+ route groups)

**Core Business:**
| Route | Description |
|-------|-------------|
| `/auth` | Login, register, refresh, MFA |
| `/products` | Product CRUD |
| `/categories` | Category hierarchy |
| `/sales` | Sale transactions |
| `/customers` | Customer management |
| `/suppliers` | Supplier management |
| `/purchase-orders` | PO lifecycle |
| `/branches` | Multi-branch management |
| `/inventory` | Stock levels, movements, adjustments, batches, serials |
| `/stock-transfers` | Inter-branch transfers |
| `/reports` | Standard + enterprise reports |
| `/settings` | System settings |

**Financial:**
| Route | Description |
|-------|-------------|
| `/accounting` | Chart of accounts, journals |
| `/payments` | Payment processing |
| `/eta` | Egyptian Tax Authority e-invoicing |
| `/discounts` | Discount rules + stacking |
| `/coupons` | Coupon management |
| `/giftcards` | Gift card lifecycle |

**HR & Operations:**
| Route | Description |
|-------|-------------|
| `/employees` | Employee records |
| `/shifts` | Shift management |
| `/attendance` | Employee attendance |
| `/commissions` | Sales commissions |
| `/hr-payroll` | HR & payroll |

**Advanced:**
| Route | Description |
|-------|-------------|
| `/webhooks` | Webhook management |
| `/api-keys` | API key management |
| `/custom-fields` | Custom field definitions |
| `/crm` | CRM (leads, deals) |
| `/ecommerce` | E-commerce bridge |
| `/report-builder` | Custom report builder |
| `/rbac` | Role-based access config |
| `/audit-logs` | Audit logging |
| `/feature-flags` | Feature toggle system |

### 3.6 Database

| Property | Value |
|----------|-------|
| Engine | PostgreSQL |
| ORM | Prisma 5.22.0 |
| Schema | 3,151 lines |
| Models | **122 models** |
| Enums | 17 enums |

**Key Models:** Tenant, User, Product, ProductVariant, Category, Sale, SaleItem, SalePayment, Customer, Supplier, PurchaseOrder, Branch, StockLevel, StockTransfer, Employee, Shift, Commission, Coupon, GiftCard, LoyaltyProgram, JournalEntry, ChartOfAccount, Webhook, ApiKey, CustomFieldDefinition, CRM entities, RBAC entities, AuditLog

### 3.7 Background Jobs

| Job | Schedule | Purpose |
|-----|----------|---------|
| Reservation Expiry | Every 5 min | Expire stock reservations |
| Reconciliation | 2:00 AM daily | Inventory reconciliation |

### 3.8 Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `PORT` | 5000 | Server port |
| `DATABASE_URL` | — | PostgreSQL connection |
| `JWT_SECRET` | — | JWT signing (≥32 chars) |
| `JWT_REFRESH_SECRET` | — | Refresh token (≥32 chars) |
| `ENCRYPTION_KEY` | — | Data encryption (32-byte base64) |
| `JWT_EXPIRATION` | 10m | Access token TTL |
| `JWT_REFRESH_EXPIRATION` | 2d | Refresh token TTL |
| `REDIS_URL` | redis://localhost:6379 | Redis cache |
| `MONGODB_URL` | mongodb://localhost:27017/wk_logs | Log storage |
| `CORS_ORIGIN` | http://localhost:3001 | Allowed origins |
| `SKIP_RATE_LIMIT` | false | Bypass rate limiting (dev) |
| `LICENSE_SERVER_URL` | http://localhost:3100 | License server |
| `TRIAL_PERIOD_DAYS` | 30 | Free trial duration |
| `MULTITENANCY_MODE` | optional | disabled/optional/enforced |

---

## 4. Desktop — Electron Application

**Path:** `wk-pos-system/desktop/`

### 4.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Electron | 28.0.0 | Desktop shell |
| electron-vite | 2.0.0 | Build tooling |
| electron-builder | 24.9.0 | Installer packaging |
| electron-store | 8.1.0 | Encrypted persistent storage |
| better-sqlite3 | 9.6.0 | Local SQLite database |
| Express | 4.18.2 | Local API server |
| ws | 8.19.0 | WebSocket |

### 4.2 Application Identity

| Property | Value |
|----------|-------|
| App ID | `com.wkhub.pos` |
| Product Name | WK POS |
| Version | 1.5.0 (internal v2.55.0+) |
| Publisher | WK-HUB |

### 4.3 Startup Phase Architecture

```
Phase -1  → Device Identity (machine-id based UUID)
Phase 0   → Database Init (SQLite, WAL mode, 64MB cache)
Phase 0.5 → Migrations (checksum verification)
Phase 0.6 → Sync Queue Init (entity dependency ordering)
Phase 0   → Local API Server (Express on port 45000-45100)
Phase 1   → Bootstrap (License: online → cached → offline fallback)
Phase 2   → Auth Check (token validation or login window)
```

### 4.4 Window Management

| Window | Size | Content Source |
|--------|------|---------------|
| Splash | 400×300 (transparent) | `src/renderer/splash.html` |
| Main | 92% screen (min 1200×800) | `resources/frontend/index.html` |
| Login | 520×780 | `src/renderer/login.html` |

### 4.5 Security

| Feature | Implementation |
|---------|---------------|
| CSP | `script-src 'self'`; no inline scripts |
| Encryption | machine-id derived AES key |
| JWT | machine-id derived secret |
| Context isolation | Enabled |
| Node integration | Disabled |
| Sandbox | Partially (preload scripts) |
| License | Cryptographic JWT verification |
| Offline grace | 7 days (configurable) |

### 4.6 Local Database

| Property | Value |
|----------|-------|
| Engine | SQLite (better-sqlite3) |
| Mode | WAL |
| Path | `%APPDATA%/WK POS/data/wkpos.db` |
| Cache | 64MB |
| Memory-mapped I/O | 256MB |

### 4.7 Multi-Branch Sync Architecture

| Mode | Role | Description |
|------|------|-------------|
| MAIN | Server | Acts as sync server for branch network |
| CLIENT | Client | Syncs with main branch |
| STANDALONE | Offline | Fully offline, no sync |

### 4.8 Build Targets

| Platform | Formats |
|----------|---------|
| Windows | Portable + NSIS installer (x64) |
| macOS | DMG + ZIP |
| Linux | AppImage + DEB |

---

## 5. Testing

### 5.1 Backend Unit Tests (30+ suites)

Auth, barcode, branches, compliance, concurrency, coupons, customers, e-invoicing, enterprise, exports, hardware, HR, integration, inventory, multi-branch, payments, products, purchasing, RBAC, reports, sales, security, shifts, suppliers, sync

### 5.2 E2E Tests (Playwright)

| Property | Value |
|----------|-------|
| Framework | Playwright 1.57.0 |
| Base URL | http://localhost:5173 |
| Viewport | 1280×720 |
| Auth Pattern | StorageState (login once, reuse) |
| Workers | 1 (sequential) |
| Timeout | 60s |

**Test Coverage:** 32 tests across 15 modules — Auth, Dashboard, Products, Categories, Suppliers, Purchase Orders, Inventory, Sales/POS, Customers, Reports, Settings, Users & Branches, Permissions (RBAC), Accounting, Shifts

**Last Run Result:** 30 passed, 2 failed (auth timing), 37 screenshots

---

## 6. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    WK POS ENTERPRISE                        │
└─────────────────────────────────────────────────────────────┘

  ┌──────────────────┐    ┌──────────────────┐
  │  Admin Panel      │    │  Electron Desktop │
  │  React 18 + Vite  │    │  + SQLite + Sync   │
  │  Port 5173        │    │  Port 45000        │
  │  HashRouter       │    │  Offline-capable   │
  └────────┬─────────┘    └────────┬──────────┘
           │ HTTP/WS                │ HTTP (local)
           ▼                        ▼
  ┌──────────────────────────────────────────┐
  │           Express API Server              │
  │         TypeScript + Prisma ORM           │
  │              Port 5000                    │
  │                                          │
  │  ┌─────────┐ ┌──────┐ ┌──────────────┐ │
  │  │JWT Auth  │ │RBAC  │ │Tenant Isolat.│ │
  │  │+ MFA    │ │+ ACL │ │Multi-tenant  │ │
  │  └─────────┘ └──────┘ └──────────────┘ │
  │                                          │
  │  70+ Route Groups • 122 DB Models       │
  │  Real-time (Socket.IO) • Cron Jobs      │
  └──────┬──────────┬──────────┬────────────┘
         │          │          │
    ┌────▼───┐ ┌───▼────┐ ┌──▼─────────┐
    │Postgres│ │ Redis  │ │ MongoDB    │
    │ (main) │ │ cache  │ │  (logs)    │
    └────────┘ └────────┘ └────────────┘
         ▲
         │ License Check
    ┌────┴────────────┐
    │ License Server  │
    │  Port 3100      │
    └─────────────────┘
```
