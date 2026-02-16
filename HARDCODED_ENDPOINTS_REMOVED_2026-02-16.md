# WK POS Enterprise — Hardcoded Endpoints Removed

**Date:** 2026-02-16  
**Author:** Enterprise Readiness Audit (Phase 1)  
**Classification:** INTERNAL — Remediation Log  
**Status:** ACTION REQUIRED

---

## Summary

**Total hardcoded endpoint locations found: 45 (source code) + 15 (config files)**  
**Risk breakdown: 12 HIGH, 18 MEDIUM, 15 LOW (config/docs only)**

---

## 1. HIGH-RISK — Must Fix Immediately

### H-001: Network Printer IP Hardcoded

| Field | Value |
|-------|-------|
| File | `wk-pos-system/server/api/src/modules/hardware/hardware.service.js` |
| Line | ~L30-40 |
| Current | `192.168.1.100` hardcoded as printer IP |
| Impact | Printer won't work on any network other than the dev environment |
| Fix | Read from `BranchPrinterSettings` table (already exists in schema) or `SystemSetting` |

**Before:**
```javascript
const PRINTER_IP = '192.168.1.100';
```

**After:**
```javascript
// Printer IP must come from BranchPrinterSettings or SystemSetting
async function getPrinterConfig(tenantId, branchId) {
  const settings = await prisma.branchPrinterSettings.findFirst({
    where: { tenantId, branchId }
  });
  if (!settings?.ipAddress) {
    throw new AppError('Printer not configured. Go to Settings → Hardware to set printer IP.', 400);
  }
  return settings;
}
```

---

### H-002: Legacy Client `sync.js` — No URL Override

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/src/js/sync.js` |
| Line | ~L10 |
| Current | `http://localhost:5000` hardcoded, no override mechanism |
| Impact | Legacy client cannot connect to any non-localhost server |
| Fix | Read from `localStorage` or Electron IPC, with setup prompt |

**Before:**
```javascript
const API_URL = 'http://localhost:5000';
```

**After:**
```javascript
const API_URL = localStorage.getItem('apiBaseUrl') 
  || window.__WK_API_URL__ 
  || null;
if (!API_URL) {
  throw new Error('Server URL not configured. Please set up the connection in Settings.');
}
```

---

### H-003: Legacy Client `pos.js` — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/src/js/pos.js` |
| Line | ~L54 |
| Current | `http://localhost:5000/api/v1` hardcoded |
| Fix | Same pattern as H-002 — shared config module |

---

### H-004: Legacy Client `app.js` — Multiple Hardcoded URLs

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/src/js/app.js` |
| Lines | ~L120, L213, L375, L540 |
| Current | `http://localhost:5000` at 4+ locations |
| Fix | Extract to shared `config.js` module, read from `localStorage`/IPC |

---

### H-005: Legacy Client `database.js` — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/src/js/database.js` |
| Line | ~L481 |
| Current | `http://localhost:5000` hardcoded |
| Fix | Import from shared config module |

---

### H-006: Legacy Client `sync.service.js` — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/services/sync.service.js` |
| Line | ~L12 |
| Current | `http://localhost:5000` hardcoded |
| Fix | Import from shared config module |

---

### H-007: Electron Desktop `main/index.ts` — Dev URL Hardcoded

| Field | Value |
|-------|-------|
| File | `wk-pos-system/desktop/src/main/index.ts` |
| Lines | ~L129, L133, L631 |
| Current | `http://localhost:5000` as API URL, `ELECTRON_RENDERER_URL` for Vite dev server |
| Impact | Desktop cannot connect to real server in production |
| Fix | Read API URL from persistent store (electron-store) with first-run setup dialog |

**Before:**
```typescript
const API_URL = 'http://localhost:5000';
```

**After:**
```typescript
import Store from 'electron-store';
const store = new Store();
const API_URL = store.get('apiBaseUrl') as string;
if (!API_URL) {
  // Show setup dialog to enter server URL
  createSetupWindow();
}
```

---

### H-008: Divamos CORS — Developer LAN IP

| Field | Value |
|-------|-------|
| File | `divamos/backend/src/server.ts` |
| Lines | L48-54 |
| Current | `192.168.0.157` in CORS origin array |
| Impact | Developer's local IP in production config — security risk + won't work elsewhere |
| Fix | Read CORS origins from `CORS_ORIGINS` env var |

**Before:**
```typescript
const corsOptions = {
  origin: ['http://localhost:3000', 'http://localhost:5173', 'http://192.168.0.157:3000'],
  // ...
};
```

**After:**
```typescript
const corsOrigins = process.env.CORS_ORIGINS
  ? process.env.CORS_ORIGINS.split(',').map(s => s.trim())
  : ['http://localhost:3000', 'http://localhost:5173'];

const corsOptions = {
  origin: corsOrigins,
  // ...
};
```

---

### H-009: Admin Panel API Fallback — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/admin-panel/src/services/api.ts` |
| Line | ~L46 |
| Current | Falls back to `http://localhost:5000/api/v1` |
| Impact | SPA silently connects to localhost when sessionStorage is empty |
| Fix | Remove fallback; require explicit configuration via parent window or setup |

---

### H-010: Mobile API Fallback — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/mobile/src/api.ts` |
| Line | ~L5 |
| Current | Falls back to `http://localhost:5000` when SecureStore is empty |
| Impact | Mobile app tries localhost (unreachable) on first launch |
| Fix | Return `null` and redirect to `ServerConnectionScreen` |

---

### H-011: Divamos Mobile API — Hardcoded Localhost

| Field | Value |
|-------|-------|
| File | `divamos/mobile/src/api.ts` |
| Line | ~L45 |
| Current | `http://localhost:4000` hardcoded |
| Fix | Use same SecureStore + ServerConnectionScreen pattern as POS mobile |

---

### H-012: Legacy Electron `main.js` — Loading Localhost

| Field | Value |
|-------|-------|
| File | `wk-pos-system/client/electron/main.js` |
| Line | ~L38 |
| Current | `mainWindow.loadURL('http://localhost:3000')` |
| Impact | Legacy Electron loads dev server in production |
| Fix | Load from file (`loadFile`) or configurable URL |

---

## 2. MEDIUM-RISK — Should Fix Before Release

### M-001: Backend Config Localhost Defaults

| Files | Lines | Current Default |
|-------|-------|----------------|
| `server/api/src/config/index.ts` | L68, L69, L78, L86, L111 | `localhost` for Redis, MongoDB, License Server |
| `server/api/src/middleware/license.middleware.ts` | L15 | `http://localhost:3100` |
| `server/api/src/modules/license/license-client.service.ts` | L12 | `http://localhost:3100` |

**Risk:** These are `process.env.X || 'localhost:...'` fallbacks. They work in dev but silently degrade in production if env vars are unset.

**Fix:** Log a warning when falling back to localhost defaults:
```typescript
const REDIS_URL = process.env.REDIS_URL || (() => {
  console.warn('⚠️ REDIS_URL not set, falling back to localhost:6379');
  return 'redis://localhost:6379';
})();
```

### M-002: Backend Route Hardcoded URLs

| Files | Lines | Issue |
|-------|-------|-------|
| `server/api/src/modules/system/system.routes.ts` | L14 | Localhost reference |
| `server/api/src/routes/index.ts` | L125, L131 | API URL in response |
| `server/api/src/routes/api.routes.ts` | L41, L51 | API URL reference |
| `server/api/src/openapi.ts` | L35 | `http://localhost:5000` in OpenAPI spec |

**Fix:** Use `req.protocol + '://' + req.get('host')` for dynamic URL generation.

### M-003: Worker Queue Redis Default

| File | Line | Issue |
|------|------|-------|
| `server/api/src/workers/queue.ts` | L24 | `localhost:6379` Redis fallback |
| `server/api/src/workers/event-bus.ts` | L193 | `localhost:6379` Redis fallback |

### M-004: Email Service Localhost

| File | Line | Issue |
|------|------|-------|
| `server/api/src/services/email.service.ts` | L10 | `localhost` SMTP fallback |

### M-005: Divamos Transformers Hardcoded URL

| File | Line | Issue |
|------|------|-------|
| `divamos/backend/src/utils/transformers.ts` | L13 | Hardcoded base URL for image transforms |

### M-006: Divamos Checkout Hardcoded Redirect

| File | Lines | Issue |
|------|-------|-------|
| `divamos/backend/src/routes/checkout.ts` | L331-332 | `http://localhost:3000` as Stripe success/cancel URL |

### M-007: License Server Billing Redirect

| File | Lines | Issue |
|------|-------|-------|
| `license-server/src/routes/billing.routes.ts` | L30-31 | Hardcoded redirect URLs |
| `license-server/src/index.ts` | L67-69 | Localhost references |

### M-008: Edge Agent Server Defaults

| File | Lines | Issue |
|------|-------|-------|
| `APPS/wk-edge-agent/src/services/agent-service.ts` | L16-17 | `http://localhost:5000` default |
| `APPS/wk-edge-agent/src/main/index.ts` | L45-46 | Localhost default |

### M-009: Kotlin AppModule Hardcoded Base URL

| File | Line | Issue |
|------|------|-------|
| `wkpos-mobile-dashboard/app/src/.../AppModule.kt` | L79 | Base URL hardcoded in Dagger DI module |

---

## 3. LOW-RISK — Config Files (Acceptable with Documentation)

These are environment/config files where localhost defaults are expected:

| File | Notes |
|------|-------|
| `.env` files | ✅ Expected — development defaults |
| `.env.example` files | ✅ Expected — template values |
| `vite.config.ts` proxy settings | ✅ Expected — dev proxy to API |
| `docker-compose.*.yml` | ✅ Expected — container networking |
| GitHub Actions workflows | ✅ Expected — CI/CD config |
| Seed scripts | ✅ Expected — test data |
| Documentation files (50+) | ✅ Expected — examples/references |

---

## 4. Remediation Priority

| Priority | Count | Items |
|----------|-------|-------|
| **P0 — Fix Now** | 12 | H-001 through H-012 |
| **P1 — Fix Before Release** | 9 | M-001 through M-009 |
| **P2 — Document Only** | 15+ | Low-risk config/docs |

---

## 5. Implementation Plan

### Step 1: Create Shared Config Modules

1. **Legacy client** — create `wk-pos-system/client/src/js/config.js`:
   ```javascript
   export function getApiUrl() {
     return localStorage.getItem('apiBaseUrl') 
       || window.__WK_API_URL__ 
       || null;
   }
   ```

2. **Admin panel** — modify `api.ts` to remove localhost fallback, require parent window injection or setup

3. **Desktop** — add `electron-store` persistence with first-run setup dialog

4. **Divamos** — centralize config in `config.ts`, read CORS from env

### Step 2: Fix Hardcoded Values (12 HIGH files)

Apply fixes file-by-file as documented in Section 1 above.

### Step 3: Add Startup Warnings (9 MEDIUM files)

Add console.warn for all localhost fallbacks.

### Step 4: Verify

- Search codebase for remaining `192.168.`, `localhost:`, `127.0.0.1` in `.ts`, `.js`, `.kt` files
- Confirm zero hardcoded IPs in source code (excluding `.env*`, `docker-compose*`, docs)

---

*End of Hardcoded Endpoints Report — Phase 1 deliverable*
