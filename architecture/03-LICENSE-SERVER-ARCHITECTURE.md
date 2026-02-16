# License Server — Architecture Document

> **Audit Date:** February 12, 2026  
> **System:** WK-HUB License Management Server  
> **Status:** Production-Ready  

---

## 1. System Overview

The WK-HUB License Server is a centralized license management platform that controls software activation, module entitlements, and usage metering for WK POS desktop installations. It provides RSA-signed license tokens, device fingerprinting, offline activation, and a full admin portal.

| Component | Technology | Port | Purpose |
|-----------|-----------|------|---------|
| **API Server** | Express 4 + Prisma | 3100 | License management API |
| **Admin Portal** | React Router 7 SPA | 3200 | Admin management dashboard |
| **PostgreSQL** | PostgreSQL 15 | 5432 | License database (`wk_licenses`) |

---

## 2. API Server

**Path:** `license-server/src/`

### 2.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Express | 4.18.2 | HTTP framework |
| TypeScript | 5.3.3 | Language (ES2020, CommonJS) |
| Prisma | 5.7.0 | ORM (PostgreSQL) |
| jsonwebtoken | 9.0.2 | JWT auth (RS256 + HS256) |
| bcrypt + bcryptjs | 5.1.1 / 3.0.3 | Password hashing |
| helmet | 7.1.0 | Security headers |
| express-rate-limit | 7.1.5 | Rate limiting |
| zod | 3.22.4 | Input validation |
| node-machine-id | 1.1.12 | Hardware identification |
| systeminformation | 5.21.22 | Device fingerprinting |
| pg | 8.16.3 | Direct PostgreSQL (POS DB) |
| google-auth-library | 9.15.1 | Google OAuth |

### 2.2 Server Startup Flow

```
1. Load env (dotenv)
2. Connect PostgreSQL (prisma.$connect())
3. Initialize RSA signing keys (auto-generate if missing)
4. Start Express on port 3100
5. Register graceful shutdown (SIGINT, SIGTERM)
```

### 2.3 Middleware Stack

1. `helmet()` — CSP disabled in non-production
2. `cors()` — Origins: localhost:3000, 3001, 3200, 127.0.0.1
3. Global rate limiter — 1000 req/min (dev), 100 (prod)
4. Activation rate limiter — 10 req/hour
5. `express.json({ limit: '10kb' })`
6. `express.urlencoded({ limit: '10kb' })`
7. `requestLogger` — timestamp/method/path/status/duration
8. Static files from `public/`

### 2.4 API Routes

#### Public Routes

| Method | Route | Purpose |
|--------|-------|---------|
| POST | `/api/v1/license/validate` | Serial validation |
| POST | `/api/v1/license/check` | License check |
| POST | `/api/v1/license/validate-tenant` | Tenant validation |
| POST | `/api/v1/activate` | Device activation (online) |
| POST | `/api/v1/activate/offline` | Device activation (offline) |
| POST | `/api/v1/heartbeat` | Device heartbeat |
| GET | `/api/v1/heartbeat/devices` | Device listing |
| POST | `/api/v1/offline/generate` | Generate offline license |
| POST | `/api/v1/offline/validate` | Validate offline license |
| POST | `/api/v1/offline/challenge` | Challenge-response (air-gapped) |
| POST | `/api/v1/trials/request` | Trial request |
| POST | `/api/v1/trials/verify` | Trial verification |
| GET | `/api/v1/trials/status` | Trial status |
| POST | `/api/v1/decision` | License decision (Policy Authority) |
| GET | `/api/v1/plans` | List plans (public) |
| GET | `/api/v1/plans/:id` | Plan detail (public) |

#### Admin Routes (JWT-protected, `/api/v1/admin`)

| Method | Route | Purpose |
|--------|-------|---------|
| POST | `/login` | Admin login |
| GET | `/dashboard` | Dashboard stats |
| GET | `/licenses` | List licenses (paginated, filtered) |
| POST | `/licenses` | Create license |
| GET | `/licenses/serial/:serialNumber` | Lookup by serial |
| GET | `/licenses/:id` | License detail |
| PATCH | `/licenses/:id` | Update license |
| POST | `/licenses/:id/revoke` | Revoke license |
| POST | `/licenses/:id/extend` | Extend license |
| GET | `/serials` | List serial keys |
| POST | `/serials/generate` | Bulk generate serials |
| POST | `/serials/:id/revoke` | Revoke serial |
| POST | `/devices/revoke` | Revoke device |
| GET | `/devices` | List devices |
| POST | `/devices/:id/deactivate` | Deactivate device |
| GET | `/tenants` | List tenants |
| GET | `/tenants/:id` | Tenant detail (includes POS users) |
| GET | `/tenants/:id/usage` | Tenant usage stats |
| GET | `/tenants/:id/licenses` | Tenant's licenses |
| POST | `/tenants/:id/suspend` | Suspend tenant |
| POST | `/tenants/:id/activate` | Activate tenant |

#### Module Routes (JWT-protected, `/api/v1/modules`)

| Method | Route | Purpose |
|--------|-------|---------|
| GET | `/available` | Available modules |
| GET/POST | `/` | List / Create module |
| GET/PUT/DELETE | `/:id` | Module CRUD |
| POST | `/:moduleId/permissions` | Add permission |
| PUT/DELETE | `/:moduleId/permissions/:permId` | Permission CRUD |
| POST | `/:moduleId/assign` | Assign to license |
| GET | `/license/:licenseId` | License modules |

#### Plan Admin Routes (role-based, `/api/v1/plans`)

| Method | Route | Auth |
|--------|-------|------|
| GET | `/admin/all` | `requireRole('admin')` |
| POST | `/admin` | `requireRole('superadmin')` |
| PUT | `/admin/:id` | `requireRole('superadmin')` |
| DELETE | `/admin/:id` | `requireRole('superadmin')` |
| PATCH | `/admin/:id/toggle` | `requireRole('admin')` |

### 2.5 Services

| Service | Purpose |
|---------|---------|
| **LicenseService** | Serial generation (WK-XXXX-XXXX-XXXX-XXXX), SHA-256 hashing, validation with module/permission loading |
| **DecisionService** | Policy Authority — auditable ALLOW/DENY/LIMITED/GRACE decisions per tenant |
| **SigningService** | RSA 3072-bit signing, signed decisions (5-min TTL), offline tokens |
| **TokenService** | RS256 JWT license tokens, offline license files, challenge tokens |
| **AuditService** | Full audit trail with 40+ event types |
| **ApiMeteringService** | Per-tenant API call tracking, daily/monthly quotas, throttling |
| **PosDbService** | Direct PostgreSQL connection to POS database for user management |

### 2.6 Security Middleware

| Middleware | Purpose |
|-----------|---------|
| `adminAuth` | JWT Bearer token verification |
| `requireRole` | RBAC: superadmin, admin, viewer |
| `requirePermission` | Permission-based ACL (read, write, delete, manage_admins, system_settings) |
| `apiKeyAuth` | X-API-Key header validation (server-to-server) |
| `flexibleAuth` | Accepts either JWT or API key |
| `apiMeteringMiddleware` | Quota enforcement, throttling, usage recording |

---

## 3. Database Schema

**591 lines, 20 models, 7 enums**

### 3.1 Core Models

| Model | Key Fields | Description |
|-------|-----------|-------------|
| **LicensePlan** | name, tier, priceMonthly/Yearly, maxDevices/Users/Branches, modules[] | Subscription tiers |
| **License** | serialKey (unique), serialHash, status, maxDevices, isTrial, expiresAt | License instances |
| **Activation** | deviceHash, deviceName, osInfo, status, lastHeartbeat, lastIpAddress | Device activations |
| **AdminUser** | email (unique), passwordHash, role, isActive | Admin portal users |
| **Module** | code (unique), name, category, priceMonthly/Yearly, isCore | License modules |
| **ModulePermission** | code (unique per module), isPremium | Module permissions |

### 3.2 Tracking Models

| Model | Description |
|-------|-------------|
| **AuditLog** | 24 event types, IP tracking, success/fail |
| **LicenseDecision** | Policy Authority decisions with RSA signature |
| **DeviceFingerprint** | CPU, disk, OS, MAC hash, trust score (0-100) |
| **LicenseViolation** | Violation detection (clone, limit exceeded, expired) |
| **Entitlement** | Module/feature entitlements per license |
| **OfflineLicense** | Challenge tokens, offline license files |
| **OfflineToken** | Signed offline tokens with module/limit payload |

### 3.3 API Metering Models

| Model | Description |
|-------|-------------|
| **ApiUsage** | Per-request tracking (endpoint, status, response time) |
| **ApiQuota** | Per-license quotas (daily: 10K, monthly: 300K) |
| **ApiUsageSummary** | Daily aggregation statistics |
| **AdminApiKey** | API key management with permissions |

### 3.4 Enums

| Enum | Values |
|------|--------|
| `LicenseStatus` | ISSUED, ACTIVE, SUSPENDED, EXPIRED, REVOKED |
| `PlanTier` | TRIAL, BASIC, PROFESSIONAL, ENTERPRISE |
| `ActivationStatus` | ACTIVE, DEACTIVATED, REVOKED |
| `DecisionType` | ALLOW, DENY, LIMITED, GRACE |
| `ViolationType` | LIMIT_EXCEEDED, MODULE_BLOCKED, LICENSE_EXPIRED, DEVICE_LIMIT, CLONE_DETECTED, etc. |
| `ViolationSeverity` | LOW, MEDIUM, HIGH, CRITICAL |
| `AuditEventType` | 24 event types (license, device, heartbeat, serial, trial, tenant) |

---

## 4. Admin Portal

**Path:** `license-server/admin-portal/`

### 4.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| React Router | 7.10.1 | SPA mode (SSR disabled) |
| React | 19.x | UI framework |
| Vite | 7.1.7 | Build tool |
| Tailwind CSS | 4.1.18 | Styling |
| Zustand | 5.0.9 | State management |
| TanStack React Query | 5.90.15 | Server state |
| Axios | 1.13.2 | HTTP client |
| React Hook Form | 7.69.0 | Form handling |
| Zod | 4.2.1 | Validation |
| Recharts | 3.6.0 | Charts |
| Lucide React | 0.562.0 | Icons |
| qrcode.react | 4.2.0 | QR codes |

### 4.2 Route Structure

```
/login                    → Login page
/ (layout)                → Sidebar + Header
  /                       → Dashboard (home)
  /licenses               → License list
  /licenses/new           → Create license
  /licenses/:id           → License detail
  /modules                → Module list
  /modules/:id            → Module detail
  /trial-requests         → Trial management
  /tenants                → Tenant list
  /tenants/:id            → Tenant detail
  /devices                → Device list
  /customers              → Customer list
  /billing                → Billing management
  /analytics              → Analytics dashboard
  /audit-logs             → Audit trail
  /support                → Support tickets
  /settings               → System settings
```

**Total: 20 route files, 17 pages**

### 4.3 API Client (629 lines, 13 service groups)

| Service Group | Key Endpoints |
|---------------|---------------|
| `authApi` | login, logout, getProfile |
| `dashboardApi` | getStats, getRecentActivity |
| `licensesApi` | CRUD, activate, suspend, revoke, renew |
| `serialsApi` | getAll, generate, revoke, export |
| `plansApi` | getAll, create, update |
| `tenantsApi` | CRUD, suspend, activate, user management |
| `devicesApi` | getAll, deactivate |
| `trialRequestsApi` | getAll, approve, reject |
| `auditLogsApi` | getAll |
| `analyticsApi` | license, revenue, usage stats |
| `modulesApi` | CRUD, permissions, license assignment |
| `activationApi` | activate, offline request/complete |
| `apiKeysApi` | CRUD, regenerate |

### 4.4 Authentication Flow

```
Admin → /login → POST /api/v1/admin/login
                    ↓
              { token, admin }
                    ↓
              localStorage["wk_license_admin_auth"]
                    ↓
              Axios interceptor: Authorization: Bearer <token>
                    ↓
              401 → Clear storage → Redirect to /login
```

---

## 5. Security Architecture

### 5.1 RSA Key Infrastructure

| Property | Value |
|----------|-------|
| Algorithm | RSA 3072-bit |
| Format | PKCS8 PEM (private), SPKI PEM (public) |
| Auto-generation | On startup if keys missing |
| File permissions | Private: 0o600, Public: 0o644 |
| Optional encryption | AES-256-CBC passphrase |

### 5.2 License Key Format

```
PREFIX-XXXX-XXXX-XXXX-XXXX
  │      └─ 4 groups of 4 chars
  └─ Default: "WK"

Character set: ABCDEFGHJKLMNPQRSTUVWXYZ23456789
(excludes 0/O/1/I for readability)

Storage: SHA-256 hash of uppercased serial
```

### 5.3 Device Fingerprinting

| Data Point | Purpose |
|-----------|---------|
| CPU ID | Hardware identification |
| Disk ID | Storage fingerprint |
| OS Install ID | OS installation identity |
| MAC Address Hash | Network identity |
| Hostname | Device name |
| Trust Score | 0-100 confidence rating |
| Clone Detection | Duplicate fingerprint alert |

### 5.4 Decision Authority

The Policy Authority makes auditable decisions for each license operation:

| Decision | Meaning |
|----------|---------|
| **ALLOW** | Full access granted |
| **DENY** | Access blocked |
| **LIMITED** | Reduced functionality |
| **GRACE** | Expired but grace period active |

Each decision is RSA-signed with a 5-minute TTL and stored with correlation IDs.

### 5.5 Rate Limiting

| Scope | Window | Limit |
|-------|--------|-------|
| Global (dev) | 1 min | 1000 |
| Global (prod) | 1 min | 100 |
| Activation | 1 hour | 10 |
| API Metering (daily) | 24 hours | 10,000 |
| API Metering (monthly) | 30 days | 300,000 |

### 5.6 Admin Role Matrix

| Role | Read | Write | Delete | Manage Admins | System Settings |
|------|------|-------|--------|---------------|-----------------|
| superadmin | ✅ | ✅ | ✅ | ✅ | ✅ |
| admin | ✅ | ✅ | ✅ | ❌ | ❌ |
| viewer | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## 6. Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `LICENSE_SERVER_PORT` | 3100 | API port |
| `LICENSE_DATABASE_URL` | postgresql://...5432/wk_licenses | License DB |
| `POS_DATABASE_URL` | postgresql://...5432/wk_pos_db | POS DB (direct) |
| `JWT_SECRET` | — | Admin JWT signing (HS256) |
| `KEY_PASSPHRASE` | — | RSA key encryption |
| `LICENSE_API_KEY` | — | Server-to-server auth |
| `ADMIN_EMAIL` | admin@wk-hub.com | Default admin |
| `DEFAULT_TRIAL_DAYS` | 14 | Trial duration |
| `MAX_DEVICES_TRIAL` | 1 | Trial device limit |
| `MAX_DEVICES_BASIC` | 3 | Basic device limit |
| `MAX_DEVICES_PRO` | 10 | Pro device limit |
| `MAX_DEVICES_ENTERPRISE` | 50 | Enterprise device limit |

---

## 7. Testing

### 7.1 E2E Tests (Playwright 1.58.2)

**Suite 1: Admin Portal** (6 tests)
- Login page screenshot
- Login succeeds  
- Dashboard loads
- Tenants list
- Licenses list
- Modules/Entitlements

**Suite 2: API Endpoints** (4 tests)
- Health check (GET /health)
- Plans list API
- Login API
- License validation

**Last Run:** 10 passed, 0 failed • 7 screenshots

### 7.2 Unit Tests (Jest 30.2.0)
- Token service: RS256 sign/verify, offline license generation, challenge tokens

---

## 8. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                  WK-HUB LICENSE SERVER                      │
└─────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────┐
  │         Admin Portal (SPA)           │
  │   React Router 7 + Tailwind 4       │
  │   Zustand + React Query             │
  │            Port 3200                 │
  │                                      │
  │  Pages: Dashboard, Licenses,         │
  │  Modules, Tenants, Devices,          │
  │  Analytics, Audit Logs, Settings     │
  └──────────────┬───────────────────────┘
                 │ Axios + JWT Bearer
                 ▼
  ┌──────────────────────────────────────┐
  │       License Server API             │
  │     Express 4 + TypeScript           │
  │     Prisma ORM + RS256 JWT           │
  │           Port 3100                  │
  │                                      │
  │  ┌────────────────────────────────┐  │
  │  │ License Engine                 │  │
  │  │  • Serial generation (WK-XXX) │  │
  │  │  • RSA 3072-bit signing       │  │
  │  │  • Policy Authority decisions │  │
  │  │  • Offline token support      │  │
  │  │  • Device fingerprinting      │  │
  │  │  • Clone detection            │  │
  │  │  • API metering + quotas      │  │
  │  └────────────────────────────────┘  │
  │                                      │
  │  ┌────────────────────────────────┐  │
  │  │ Security Layer                 │  │
  │  │  • JWT Auth (HS256 admin)      │  │
  │  │  • RBAC (superadmin/admin/     │  │
  │  │    viewer)                     │  │
  │  │  • Rate limiting              │  │
  │  │  • API key auth               │  │
  │  │  • Audit trail (40+ events)   │  │
  │  └────────────────────────────────┘  │
  └──────┬───────────────────┬──────────┘
         │                   │
    ┌────▼────────┐    ┌────▼─────────┐
    │ PostgreSQL  │    │  PostgreSQL   │
    │ wk_licenses │    │  wk_pos_db   │
    │ 20 models   │    │  (read POS   │
    │ 7 enums     │    │   users)     │
    └─────────────┘    └──────────────┘
         ▲
         │ RSA 3072-bit keys
    ┌────┴──────┐
    │ keys/     │
    │ *.pem     │
    └───────────┘
    
  ┌──────────────────────────────────────┐
  │        WK POS Desktop Client         │
  │        (License Consumer)            │
  │                                      │
  │  Startup → /api/v1/license/validate  │
  │  Activate → /api/v1/activate         │
  │  Heartbeat → /api/v1/heartbeat       │
  │  Offline → /api/v1/offline/generate  │
  └──────────────────────────────────────┘
```
