# WK POS Enterprise — Configuration Standard

**Date:** 2026-02-16  
**Author:** Enterprise Readiness Audit (Phase 1)  
**Classification:** INTERNAL — Configuration Reference  
**Status:** PROPOSED — Pending implementation

---

## 1. Design Principle

> **Every runtime-configurable value (URLs, IPs, ports, feature flags) MUST be:**
> 1. Stored in a **single canonical location** per application
> 2. Overridable via **environment variable** or **admin settings UI** — never hardcoded
> 3. Validated on startup with **clear error messages** for missing/invalid values
> 4. Documented in `.env.example` with **type, default, and description**

---

## 2. Configuration Schema Per Application

### 2.1 WK-POS API (Backend)

| Setting | Env Var | DB Table | Default | Type | Required |
|---------|---------|----------|---------|------|----------|
| API Port | `PORT` | — | `5000` | number | Yes |
| Database URL | `DATABASE_URL` | — | — | string (postgres://) | Yes |
| Redis URL | `REDIS_URL` | — | `redis://localhost:6379` | string | Yes |
| MongoDB URI | `MONGO_URI` | — | `mongodb://localhost:27017/wk_logs` | string | No |
| JWT Secret | `JWT_SECRET` | — | — | string (≥32 chars) | Yes |
| JWT Refresh Secret | `JWT_REFRESH_SECRET` | — | — | string (≥32 chars) | Yes |
| Encryption Key | `ENCRYPTION_KEY` | — | — | base64 (32 bytes) | Yes |
| CORS Origins | `CORS_ORIGINS` | — | `*` | comma-separated URLs | No |
| License Server URL | `LICENSE_SERVER_URL` | — | `http://localhost:3100` | URL | Yes |
| Stripe Secret Key | `STRIPE_SECRET_KEY` | — | — | string | No |
| Twilio Account SID | `TWILIO_ACCOUNT_SID` | — | — | string | No |
| Twilio Auth Token | `TWILIO_AUTH_TOKEN` | — | — | string | No |
| SMTP Host | `SMTP_HOST` | — | — | hostname | No |
| SMTP Port | `SMTP_PORT` | — | `587` | number | No |
| SMTP User | `SMTP_USER` | — | — | string | No |
| SMTP Password | `SMTP_PASSWORD` | — | — | string | No |
| Multi-tenancy Mode | `MULTI_TENANCY_MODE` | — | `optional` | `disabled\|optional\|enforced` | No |
| Rate Limit Max | `RATE_LIMIT_MAX` | — | `100` | number | No |
| Rate Limit Window (ms) | `RATE_LIMIT_WINDOW` | — | `900000` (15 min) | number | No |
| Printer IP | — | `SystemSetting` | — | IP address | No |
| Company Name | — | `SystemSetting` | — | string | No |
| Default Currency | — | `SystemSetting` | `SAR` | ISO 4217 | No |
| Default VAT Rate | — | `SystemSetting` | `15` | percentage | No |
| Default Timezone | — | `SystemSetting` | `Asia/Riyadh` | IANA timezone | No |

#### Storage Mechanism
- **Startup config**: `config/index.ts` reads `process.env` → validated → frozen object exported
- **Runtime settings**: `SystemSetting` table queried via `settings.service.ts`
- **Feature flags**: `FeatureFlag` table + `feature-flags` module

#### Admin UI
- **Settings Page**: `admin-panel/src/pages/SettingsPage/` — company info, localization, general settings
- **Hardware Page**: `admin-panel/src/pages/HardwarePage/` — printer/scanner config
- **⚠️ MISSING**: No "Server Connection" or "API URLs" settings page in admin panel

---

### 2.2 License Server

| Setting | Env Var | Default | Required |
|---------|---------|---------|----------|
| Port | `PORT` | `3100` | Yes |
| Database URL | `DATABASE_URL` | — | Yes |
| JWT Secret | `JWT_SECRET` | — | Yes |
| Stripe Secret Key | `STRIPE_SECRET_KEY` | — | No |
| Admin Email | `ADMIN_EMAIL` | — | No |
| SMTP Config | `SMTP_HOST`, `SMTP_PORT`, etc. | — | No |
| Google OAuth | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | — | No |
| Frontend URL | `FRONTEND_URL` | `http://localhost:5173` | No |
| POS Database URL | `POS_DATABASE_URL` | — | No |

#### Storage Mechanism
- `env-validation.ts` validates on startup
- `SystemSetting` table for runtime config
- No admin panel (API + CLI only)

---

### 2.3 Divamos E-Commerce Backend

| Setting | Env Var | Default | Required |
|---------|---------|---------|----------|
| Port | `PORT` | `4000` | Yes |
| Database URL | `DATABASE_URL` | — | Yes |
| JWT Secret | `JWT_SECRET` | — | Yes |
| Stripe Secret Key | `STRIPE_SECRET_KEY` | — | No |
| Stripe Webhook Secret | `STRIPE_WEBHOOK_SECRET` | — | No |
| Upload Dir | `UPLOAD_DIR` | `./uploads` | No |
| CORS Origins | hardcoded array | — | ⚠️ Should be env |

#### Storage Mechanism
- Direct `process.env` reads (no central config object)
- `PageSettings` table for storefront CMS
- `RetailSyncConfig` table for POS integration (API key, webhook URL, sync settings)
- **⚠️ CORS origins hardcoded with developer LAN IP `192.168.0.157`**

#### Admin UI
- Divamos admin panel (`divamos/admin/`) — product/order/settings management
- Retail Sync Config page — POS integration settings with API key and webhook URL

---

### 2.4 Desktop Shell (Electron)

| Setting | Source | Default | Required |
|---------|--------|---------|----------|
| API Base URL | IPC from main process | `http://localhost:5000` | Yes |
| Window Dimensions | Hardcoded in `main/index.ts` | 1200×800 | No |
| Dev Server URL | `ELECTRON_RENDERER_URL` env | `http://localhost:5173` | Dev only |

#### Storage Mechanism
- Main process: hardcoded in `main/index.ts`
- Renderer: `sessionStorage` cache of API URL
- **⚠️ No persist layer** — URL resets on restart
- **⚠️ No settings UI** for server URL in Desktop shell

---

### 2.5 Admin Panel (React SPA)

| Setting | Source | Default | Required |
|---------|--------|---------|----------|
| API Base URL | `api.ts` → `sessionStorage` → window ancestor → fallback | `http://localhost:5000/api/v1` | Yes |

#### Storage Mechanism
- `api.ts` line 46: cascading resolution:
  1. `sessionStorage.getItem('apiBaseUrl')`
  2. Parent/ancestor window message
  3. Fallback to `http://localhost:5000/api/v1`
- **⚠️ Fallback is hardcoded localhost** — works in dev, breaks in production LAN

---

### 2.6 Mobile POS (Expo / React Native)

| Setting | Source | Default | Required |
|---------|--------|---------|----------|
| API Base URL | `expo-secure-store` | `http://localhost:5000` | Yes |

#### Storage Mechanism
- `ServerConnectionScreen` UI → user enters IP → stored in `expo-secure-store`
- `api.ts` line 5: reads from SecureStore, **falls back to hardcoded localhost**
- **✅ Good pattern** — has dedicated UI for server connection
- **⚠️ Fallback should not exist** — force user to configure on first launch

---

### 2.7 WK-Edge Agent

| Setting | Source | Default | Required |
|---------|--------|---------|----------|
| Server URL | `agent-config.json` | `http://localhost:5000` | Yes |
| Branch ID | `agent-config.json` | — | Yes |
| Device ID | OS keychain (keytar) | Auto-generated ULID | Yes |
| SQLite Path | `agent-config.json` | `data/edge.db` | Yes |
| Sync Interval | `agent-config.json` | `30000` (30s) | No |
| Batch Size | `agent-config.json` | `50` | No |

#### Storage Mechanism
- `agent-config.json` file on disk
- `keytar` for secrets (device key, auth token)
- **✅ Good pattern** — file-based config + OS keychain for secrets
- **⚠️ Default localhost** should be replaced with setup wizard

---

### 2.8 Kotlin Dashboard (Android)

| Setting | Source | Default | Required |
|---------|--------|---------|----------|
| Base URL | `SharedPreferences` | Hardcoded in `AppModule.kt` | Yes |

#### Storage Mechanism
- `SessionManager` + `SharedPreferences`
- Settings screen for server URL
- **⚠️ Default URL is hardcoded** in Dagger module

---

## 3. Recommended Configuration Architecture

### 3.1 Server-Side Config Resolution Order

```
1. Environment variable (highest priority)
2. .env file (dotenv)
3. SystemSetting table (DB, for runtime-changeable settings)
4. Default value in config schema
```

### 3.2 Client-Side Config Resolution Order

```
1. User-configured value (stored persistently)
2. IPC/parent window injection (for Electron-hosted apps)
3. Auto-discovery (mDNS/network scan — future)
4. ⛔ NO hardcoded fallback — force setup wizard on first launch
```

### 3.3 Required Admin UI Additions

| App | Required UI | Current Status |
|-----|------------|----------------|
| Admin Panel | "Server Connection" settings page | ❌ Missing |
| Admin Panel | "Network Printer" settings in Hardware page | ⚠️ Exists but uses hardcoded IP |
| Desktop Shell | Server URL configuration on first launch | ❌ Missing |
| Divamos Admin | CORS origins configuration | ❌ Missing |
| Edge Agent | Setup wizard for server URL + branch binding | ❌ Missing |

---

## 4. Environment Variable Validation Rules

All backend applications MUST validate required env vars on startup:

```typescript
// ✅ CORRECT — Fail fast with clear error
function validateEnv() {
  const required = ['DATABASE_URL', 'JWT_SECRET', 'JWT_REFRESH_SECRET'];
  const missing = required.filter(key => !process.env[key]);
  if (missing.length > 0) {
    console.error(`❌ Missing required environment variables: ${missing.join(', ')}`);
    process.exit(1);
  }
  
  if (process.env.JWT_SECRET && process.env.JWT_SECRET.length < 32) {
    console.error('❌ JWT_SECRET must be at least 32 characters');
    process.exit(1);
  }
}
```

---

## 5. `.env.example` Template Standard

Every backend application MUST have a `.env.example` with this format:

```bash
# ============================================
# WK-POS API Configuration
# ============================================

# --- Required ---
DATABASE_URL=postgresql://user:password@localhost:5432/wk_pos
JWT_SECRET=your-jwt-secret-min-32-characters-here
JWT_REFRESH_SECRET=your-refresh-secret-min-32-characters
ENCRYPTION_KEY=base64-encoded-32-byte-key

# --- Optional (with defaults) ---
PORT=5000                                    # Server port (default: 5000)
REDIS_URL=redis://localhost:6379             # Redis connection (default: redis://localhost:6379)
CORS_ORIGINS=http://localhost:5173           # Comma-separated origins (default: *)
LICENSE_SERVER_URL=http://localhost:3100      # License server (default: http://localhost:3100)
MULTI_TENANCY_MODE=optional                  # disabled|optional|enforced (default: optional)

# --- Integrations (optional) ---
# STRIPE_SECRET_KEY=sk_test_...
# TWILIO_ACCOUNT_SID=AC...
# TWILIO_AUTH_TOKEN=...
# SMTP_HOST=smtp.gmail.com
# SMTP_PORT=587
# SMTP_USER=...
# SMTP_PASSWORD=...
```

---

*End of Configuration Standard — Phase 1 deliverable*
