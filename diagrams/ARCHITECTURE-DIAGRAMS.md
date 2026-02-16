# System Architecture Diagrams

> **Audit Date:** February 12, 2026  
> **All diagrams use Mermaid.js syntax for rendering**

---

## 1. Full Platform Overview

```mermaid
graph TB
    subgraph "WK-HUB Enterprise Platform"
        subgraph "WK POS System"
            WKF[Admin Panel<br/>React 18 + Vite<br/>Port 5173]
            WKD[Desktop App<br/>Electron 28<br/>Port 45000]
            WKB[API Server<br/>Express 4<br/>Port 5000]
        end
        
        subgraph "DIVAMOS E-Commerce"
            DSF[Storefront<br/>Next.js 14<br/>Port 3000]
            DAP[Admin Panel<br/>Next.js 14<br/>Port 3001]
            DBE[Backend<br/>Express 4<br/>Port 4000]
        end
        
        subgraph "License Server"
            LAP[Admin Portal<br/>React Router 7<br/>Port 3200]
            LBE[License API<br/>Express 4<br/>Port 3100]
        end
        
        subgraph "Databases"
            PG1[(PostgreSQL<br/>wk_pos_db)]
            PG2[(PostgreSQL<br/>divamos_db)]
            PG3[(PostgreSQL<br/>wk_licenses)]
            RD[(Redis Cache)]
            MG[(MongoDB Logs)]
            SQ[(SQLite<br/>Offline)]
        end
        
        subgraph "Storage"
            S3[MinIO/S3<br/>Media Files]
        end
    end
    
    WKF --> WKB
    WKD --> WKB
    WKD --> SQ
    WKB --> PG1
    WKB --> RD
    WKB --> MG
    WKB --> LBE
    
    DSF --> DBE
    DAP --> DBE
    DBE --> PG2
    DBE --> S3
    DBE --> WKB
    
    LAP --> LBE
    LBE --> PG3
    LBE --> PG1
    
    WKD -.->|License Check| LBE
    
    style WKF fill:#3b82f6,color:#fff
    style WKD fill:#6366f1,color:#fff
    style WKB fill:#2563eb,color:#fff
    style DSF fill:#10b981,color:#fff
    style DAP fill:#059669,color:#fff
    style DBE fill:#047857,color:#fff
    style LAP fill:#f59e0b,color:#fff
    style LBE fill:#d97706,color:#fff
    style PG1 fill:#7c3aed,color:#fff
    style PG2 fill:#7c3aed,color:#fff
    style PG3 fill:#7c3aed,color:#fff
```

---

## 2. WK POS — Authentication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Admin Panel (React)
    participant API as Express API (:5000)
    participant DB as PostgreSQL
    participant LS as License Server (:3100)
    
    U->>FE: Navigate to /#/login
    U->>FE: Enter email + password
    FE->>API: POST /api/v1/auth/login
    API->>DB: Verify credentials (bcrypt)
    API->>DB: Load user roles & permissions
    API->>LS: POST /api/v1/license/validate (serial check)
    LS-->>API: License status + modules
    API-->>FE: { user, token, refreshToken, modules }
    FE->>FE: Zustand persist → localStorage["wk-auth"]
    FE->>FE: Navigate to /#/ (dashboard)
    
    Note over FE,API: Subsequent Requests
    FE->>API: GET /api/v1/... + Bearer token + X-Tenant-Id
    API->>API: JWT verify + permission check
    API-->>FE: Response data
    
    Note over FE,API: Token Refresh
    FE->>API: GET /api/v1/... (401 expired)
    API-->>FE: 401 Unauthorized
    FE->>API: POST /api/v1/auth/refresh-token
    API-->>FE: { newToken, newRefreshToken }
    FE->>API: Retry original request
```

---

## 3. WK POS — Desktop Startup Sequence

```mermaid
sequenceDiagram
    participant E as Electron Main
    participant S as Splash Window
    participant DB as SQLite
    participant LS as License Server
    participant L as Login Window
    participant M as Main Window
    
    E->>S: Show splash (400×300)
    E->>E: Phase -1: Device Identity (machine-id)
    E->>DB: Phase 0: Open SQLite (WAL mode)
    E->>DB: Phase 0.5: Run migrations
    E->>E: Phase 0.6: Init sync queue
    E->>E: Phase 0: Start local Express (:45000)
    
    E->>LS: Phase 1: Bootstrap (validate license)
    alt Online
        LS-->>E: License valid + modules
    else Cached
        E->>DB: Load cached license
    else Offline (grace)
        E->>E: Minimal fallback (7 days grace)
    end
    
    E->>E: Phase 2: Check saved auth token
    alt Token valid
        E->>M: Load main window (resources/frontend/index.html)
    else No token
        E->>L: Show login window (520×780)
        L->>E: Credentials via IPC
        E->>DB: Validate against SQLite
        E-->>L: Success
        L->>L: Close
        E->>M: Load main window
    end
    
    E->>S: Close splash
    M->>M: Inject auth into localStorage
```

---

## 4. DIVAMOS — Request Flow

```mermaid
graph LR
    subgraph "Client"
        Browser[Browser]
    end
    
    subgraph "Nginx"
        NG[Nginx<br/>:80/:443<br/>SSL + Rate Limit]
    end
    
    subgraph "Frontend"
        SF[Storefront<br/>Next.js :3000<br/>SSR + SSG]
        AP[Admin<br/>Next.js :3001<br/>CSR]
    end
    
    subgraph "Backend"
        BE[Express :4000<br/>Helmet + CORS<br/>Rate Limit]
    end
    
    subgraph "Data"
        PG[(PostgreSQL :5433)]
        S3[(MinIO S3<br/>:9000)]
        ST[Stripe<br/>Payments]
    end
    
    subgraph "Integration"
        POS[WK POS API<br/>:5000]
    end
    
    Browser -->|divamos.com| NG
    Browser -->|admin.divamos.com| NG
    NG --> SF
    NG --> AP
    SF --> BE
    AP --> BE
    BE --> PG
    BE --> S3
    BE --> ST
    BE <-->|Retail Sync| POS
    
    style NG fill:#f97316,color:#fff
    style SF fill:#10b981,color:#fff
    style AP fill:#059669,color:#fff
    style BE fill:#3b82f6,color:#fff
    style PG fill:#7c3aed,color:#fff
```

---

## 5. License Server — License Lifecycle

```mermaid
stateDiagram-v2
    [*] --> ISSUED: Serial Generated
    ISSUED --> ACTIVE: Device Activation
    ACTIVE --> ACTIVE: Heartbeat ✓
    ACTIVE --> SUSPENDED: Admin Suspend / Violation
    ACTIVE --> EXPIRED: Expiry Date Reached
    ACTIVE --> REVOKED: Admin Revoke
    SUSPENDED --> ACTIVE: Admin Reactivate
    EXPIRED --> ACTIVE: Renewal / Extension
    REVOKED --> [*]: Terminal State
    
    state ACTIVE {
        [*] --> Online
        Online --> Offline: No Heartbeat
        Offline --> Grace: Grace Period (7 days)
        Grace --> Online: Heartbeat Restored
        Offline --> Online: Heartbeat Restored
    }
```

---

## 6. License Server — Decision Authority Flow

```mermaid
flowchart TD
    A[POS Client Request] --> B{License Valid?}
    B -->|No| C[DENY]
    B -->|Yes| D{Expired?}
    D -->|Yes| E{Grace Period?}
    E -->|Yes| F[GRACE - Limited]
    E -->|No| G[DENY - Expired]
    D -->|No| H{Device Limit?}
    H -->|Exceeded| I[DENY - Device Limit]
    H -->|OK| J{Module Check}
    J -->|All OK| K[ALLOW]
    J -->|Some Blocked| L[LIMITED]
    
    K --> M[RSA Sign Decision]
    L --> M
    F --> M
    C --> M
    G --> M
    I --> M
    
    M --> N[Store in DB + Return]
    
    style K fill:#10b981,color:#fff
    style L fill:#f59e0b,color:#fff
    style F fill:#f59e0b,color:#fff
    style C fill:#ef4444,color:#fff
    style G fill:#ef4444,color:#fff
    style I fill:#ef4444,color:#fff
```

---

## 7. Multi-System Data Flow

```mermaid
graph TB
    subgraph "Retail Operations"
        POS[WK POS<br/>Point of Sale]
        INV[Inventory<br/>Management]
        CRM[Customer<br/>Management]
    end
    
    subgraph "E-Commerce"
        WEB[DIVAMOS<br/>Storefront]
        ADMIN[DIVAMOS<br/>Admin]
    end
    
    subgraph "Sync Layer"
        SYNC[Retail Sync<br/>API]
    end
    
    subgraph "Licensing"
        LIC[License<br/>Server]
    end
    
    POS -->|Product Data| SYNC
    INV -->|Stock Levels| SYNC
    CRM -->|Customers| SYNC
    SYNC -->|Sync to Web| WEB
    SYNC -->|Sync to Admin| ADMIN
    WEB -->|Orders| SYNC
    SYNC -->|Orders to POS| POS
    
    POS -.->|License Check| LIC
    LIC -.->|Module Access| POS
    
    style POS fill:#3b82f6,color:#fff
    style WEB fill:#10b981,color:#fff
    style LIC fill:#f59e0b,color:#fff
```

---

## 8. Database Schema Summary

```mermaid
erDiagram
    TENANT ||--o{ USER : "has"
    TENANT ||--o{ BRANCH : "has"
    TENANT ||--o{ PRODUCT : "has"
    TENANT ||--o{ CUSTOMER : "has"
    TENANT ||--o{ SALE : "has"
    
    USER ||--o{ SALE : "creates"
    USER ||--o{ SHIFT : "works"
    
    PRODUCT ||--o{ PRODUCT_VARIANT : "has"
    PRODUCT ||--o{ SALE_ITEM : "in"
    PRODUCT }o--|| CATEGORY : "belongs to"
    PRODUCT }o--o{ SUPPLIER : "supplied by"
    
    SALE ||--o{ SALE_ITEM : "contains"
    SALE ||--o{ SALE_PAYMENT : "paid by"
    SALE }o--|| CUSTOMER : "by"
    
    BRANCH ||--o{ STOCK_LEVEL : "has"
    BRANCH ||--o{ STOCK_TRANSFER : "source/dest"
    
    PURCHASE_ORDER }o--|| SUPPLIER : "from"
    PURCHASE_ORDER ||--o{ PO_ITEM : "contains"
    
    LICENSE ||--o{ ACTIVATION : "has"
    LICENSE ||--o{ LICENSE_MODULE : "includes"
    LICENSE }o--|| LICENSE_PLAN : "on"
    MODULE ||--o{ MODULE_PERMISSION : "has"
```

---

## 9. Port Map

```mermaid
graph LR
    subgraph "WK POS Ports"
        P5000[5000 - API Server]
        P5173[5173 - Admin Panel]
        P45000[45000 - Desktop Local API]
    end
    
    subgraph "DIVAMOS Ports"
        P3000[3000 - Storefront]
        P3001[3001 - Admin Panel]
        P4000[4000 - Backend API]
        P5433[5433 - PostgreSQL]
        P9000[9000 - MinIO S3]
    end
    
    subgraph "License Server Ports"
        P3100[3100 - License API]
        P3200[3200 - Admin Portal]
    end
    
    subgraph "Infrastructure"
        P80[80/443 - Nginx]
        P6379[6379 - Redis]
        P27017[27017 - MongoDB]
    end
    
    style P5000 fill:#3b82f6,color:#fff
    style P5173 fill:#60a5fa,color:#fff
    style P3000 fill:#10b981,color:#fff
    style P3001 fill:#34d399,color:#fff
    style P4000 fill:#059669,color:#fff
    style P3100 fill:#f59e0b,color:#fff
    style P3200 fill:#fbbf24,color:#000
```
