# DIVAMOS — Architecture Document

> **Audit Date:** February 12, 2026  
> **Platform:** Self-hosted Shopify-compatible Fashion E-Commerce  
> **Status:** Production-Ready  

---

## 1. System Overview

DIVAMOS is a self-hosted, Shopify-compatible fashion e-commerce platform designed for the Egyptian market. Built as a monorepo with pnpm workspaces, it provides a complete online retail solution with customer storefront, admin management panel, and a RESTful backend.

| Component | Technology | Port | Purpose |
|-----------|-----------|------|---------|
| **Storefront** | Next.js 14 (App Router) | 3000 | Customer-facing e-commerce site |
| **Admin Panel** | Next.js 14 (App Router) | 3001 | Management dashboard |
| **Backend API** | Express 4 + Prisma | 4000 | RESTful API server |
| **PostgreSQL** | PostgreSQL 15 | 5433 | Database |
| **MinIO** | S3-compatible storage | 9000/9001 | Media/file storage |
| **Nginx** | Alpine | 80/443 | Reverse proxy + SSL |

---

## 2. Storefront

**Path:** `divamos/storefront/`

### 2.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Next.js | 14.0.4 | App Router framework |
| React | 18.2.0 | UI framework |
| Tailwind CSS | 3.4.0 | Styling |
| Zustand | 4.4.7 | Client state management |
| Framer Motion | 10.16.16 | Animations |
| Headless UI | 1.7.17 | Accessible components |
| clsx + tailwind-merge | 2.0.0 / 2.2.0 | CSS utility merging |

### 2.2 Pages (App Router)

**Shopping:**
| Route | Purpose |
|-------|---------|
| `/` | Homepage (CMS-driven hero, categories, featured products, lookbook, trust badges) |
| `/women` | Women's collection |
| `/sale` | Sale items |
| `/products/*` | Product detail pages |
| `/collections/*` | Collection listing & detail |
| `/search` | Product search |
| `/checkout` | Checkout flow |
| `/order-success` | Order confirmation |

**Customer Account:**
| Route | Purpose |
|-------|---------|
| `/login` | Customer login |
| `/register` | Customer registration |
| `/forgot-password` | Password recovery |
| `/account` | Customer account dashboard |
| `/track-order` | Order tracking |
| `/wishlist` | Wishlist |

**Informational:**
| Route | Purpose |
|-------|---------|
| `/about` | About page |
| `/contact` | Contact page |
| `/careers` | Careers |
| `/press` | Press page |
| `/sustainability` | Sustainability statement |
| `/help` | Help center |
| `/shipping` | Shipping info |
| `/returns` | Return policy |
| `/size-guide` | Size guide |
| `/terms` | Terms of service |
| `/privacy` | Privacy policy |
| `/cookies` | Cookie policy |
| `/accessibility` | Accessibility statement |

### 2.3 API Client

- Connects to backend at `NEXT_PUBLIC_API_URL` (default `http://localhost:4000`)
- Shopify-compatible interface shape (Product, Collection, Cart, CartItem)
- Functions: `getProducts`, `getProductByHandle`, `getProductRecommendations`, `getCollections`, `getCollectionByHandle`, `searchProducts`, `createCart`, `getCart`, `addToCart`, `updateCartLine`, `removeFromCart`
- Mock data fallback for build-time static generation

### 2.4 Design

- **Fonts:** Inter, Playfair Display, JetBrains Mono
- **SEO:** Full OpenGraph + Twitter Card metadata
- **Domain:** divamos.com

---

## 3. Admin Panel

**Path:** `divamos/admin/`

### 3.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Next.js | 14.0.4 | App Router framework |
| React | 18.2.0 | UI framework |
| Tailwind CSS | 3.4.0 | Styling |
| Zustand | 4.4.7 | State management |
| React Hook Form | 7.49.0 | Form handling |
| Zod | 3.22.0 | Validation |
| Recharts | 2.10.0 | Charts & analytics |
| Radix UI | Various | Dialog, Dropdown, Select, Tabs, Tooltip |

### 3.2 Dashboard Pages

| Route | Purpose |
|-------|---------|
| `/` | Login page (username + password) |
| `/dashboard` | Main dashboard (stats, revenue chart, recent orders, top products) |
| `/dashboard/orders` | Order management (list, detail, status, tracking) |
| `/dashboard/products` | Product CRUD (publish/unpublish, bulk publish) |
| `/dashboard/customers` | Customer management |
| `/dashboard/discounts` | Discount/coupon management |
| `/dashboard/inventory` | Inventory management (low stock alerts, adjustments) |
| `/dashboard/analytics` | Analytics (revenue, top products, conversion rates) |
| `/dashboard/media` | Media library (S3/MinIO upload) |
| `/dashboard/homepage` | Pages & CMS (homepage settings, category CMS) |
| `/dashboard/retail-sync` | Retail POS sync integration |
| `/dashboard/settings` | Admin settings (API keys, general) |

### 3.3 Navigation Sidebar

1. **Dashboard** — Overview
2. **Commerce** — Orders, Products & Stock, Customers, Discounts
3. **Content** — Media Library, Pages & CMS
4. **Insights** — Analytics
5. **Integrations** — Retail Sync
6. **Settings** — Configuration

### 3.4 Authentication Flow

```
Admin → / (login page)
  ↓
POST /api/v1/auth/admin/login { username, password }
  ↓
{ token, admin } → localStorage["admin_token"] + ["admin_user"]
  ↓
Redirect → /dashboard
  ↓
API Client: Authorization: Bearer <token>
  ↓
401/403 → Clear localStorage → Redirect to /
```

**Admin Roles:** ADMIN, MANAGER, EDITOR

---

## 4. Backend API

**Path:** `divamos/backend/`

### 4.1 Tech Stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| Express | 4.18.2 | HTTP framework |
| TypeScript | 5.3.3 | Language |
| Prisma | 5.8.0 | ORM (PostgreSQL 15) |
| jsonwebtoken | 9.0.2 | JWT auth |
| argon2 | 0.31.2 | Password hashing |
| stripe | 14.10.0 | Payment processing |
| AWS SDK | 3.490.0 | S3-compatible storage |
| helmet | 7.1.0 | Security headers |
| express-rate-limit | 7.1.5 | Rate limiting |
| multer | 1.4.5 | File upload |
| winston | 3.11.0 | Logging |

### 4.2 Server Configuration

| Property | Value |
|----------|-------|
| Port | 4000 |
| API prefix | `/api/v1` |
| Body limit | 10MB |
| Global rate limit | 500 req / 15 min |
| Auth rate limit | 15 req / 15 min (prod), 100 (dev) |
| Checkout rate limit | 5 req / 1 min |
| Currency | EGP (Egyptian Pound, stored as cents) |
| Tax | 14% VAT (Egypt) |
| Free shipping threshold | EGP 2,000 |

### 4.3 API Routes

**Public (no auth):**

| Method | Route | Purpose |
|--------|-------|---------|
| GET | `/health` | Health check |
| GET | `/api/v1/products` | List products (filters, pagination, sort) |
| GET | `/api/v1/products/:slug` | Product by slug |
| GET | `/api/v1/collections` | List collections |
| GET | `/api/v1/collections/:slug` | Collection with products |
| GET | `/api/v1/search?q=` | Product search |
| GET | `/api/v1/categories` | List categories |
| POST | `/api/v1/cart` | Create cart |
| GET | `/api/v1/cart/:id` | Get cart |
| POST | `/api/v1/cart/:id/items` | Add to cart |
| PATCH | `/api/v1/cart/:id/items/:itemId` | Update cart item |
| DELETE | `/api/v1/cart/:id/items/:itemId` | Remove from cart |
| POST | `/api/v1/checkout` | Create checkout (rate-limited) |
| POST | `/api/v1/webhook/stripe` | Stripe webhook |
| GET | `/api/v1/orders/track` | Track order |
| POST | `/api/v1/discounts/validate` | Validate discount code |
| GET | `/api/v1/storefront/settings/homepage` | Homepage CMS |
| GET | `/api/v1/storefront/settings/page/:slug` | Page CMS |

**Auth Routes (`/api/v1/auth`):**

| Method | Route | Purpose |
|--------|-------|---------|
| POST | `/admin/login` | Admin login |
| POST | `/register` | Customer registration |
| POST | `/login` | Customer login |
| GET | `/me` | Get current user |
| PATCH | `/profile` | Update profile |
| POST/PUT/DELETE | `/addresses` | Address management |
| GET | `/orders` | Customer order history |

**Admin Routes (`/api/v1/admin/*`, requireAdminAuth):**

| Method | Route | Purpose |
|--------|-------|---------|
| GET | `/stats` | Dashboard statistics |
| GET/POST/PATCH/DELETE | `/products` | Product CRUD |
| POST | `/products/:id/publish` | Publish product |
| POST | `/products/bulk-publish` | Bulk publish |
| GET/PATCH | `/orders` | Order management |
| GET | `/customers` | Customer list |
| GET/PATCH | `/inventory` | Inventory management |
| GET/POST | `/categories` | Category management |
| GET | `/analytics` | Analytics data |
| GET/POST/PATCH/DELETE | `/discounts` | Discount management |
| POST/GET/DELETE | `/media` | Media management |
| GET/PUT | `/settings` | Settings management |
| GET/POST/DELETE | `/api-keys` | API key management |

**Retail Sync Routes (`/api/v1/retail`):**

| Feature | Description |
|---------|-------------|
| Product Sync | Inbound/outbound product sync with POS |
| Inventory Sync | Real-time stock level sync |
| Order Sync | Order synchronization |
| Coupon Sync | POS coupon integration |
| Gift Card Sync | Gift card sync |
| Loyalty Sync | Loyalty program sync |

### 4.4 Database Schema

**22 models, 6 enums**

| Model | Description |
|-------|-------------|
| `User` | Users (admin + customer), roles, auth |
| `Category` | Product categories (hierarchical, POS sync) |
| `Product` | Products (SEO, publish workflow, POS sync) |
| `ProductVariant` | SKU, price (cents), stock, attributes |
| `ProductImage` | Product images with position |
| `ProductOption` | Options (Size, Color) with values |
| `Collection` | Product collections (smart rules) |
| `CollectionProduct` | Collection ↔ Product M2M |
| `Cart` | Shopping carts (guest + user) |
| `CartItem` | Cart line items |
| `Order` | Orders (Stripe integration, financial tracking) |
| `OrderItem` | Order line items (product snapshots) |
| `Address` | Shipping/billing addresses |
| `InventoryLog` | Inventory change tracking |
| `AuditLog` | Admin action audit trail |
| `Discount` | Native discount codes |
| `Media` | Uploaded media/assets (S3) |
| `PageSettings` | CMS page settings |
| `RetailSyncConfig` | Retail POS system configs |
| `RetailSyncLog` | Sync activity logs |
| `OrderNotification` | Notification queue |
| `SyncLog` | Idempotent sync batch log |
| `Coupon` | POS-synced coupons |
| `GiftCard` | POS-synced gift cards |
| `LoyaltyProgram` | POS-synced loyalty programs |

**Enums:** UserRole (ADMIN, MANAGER, EDITOR, CUSTOMER), PublishStatus (DRAFT, PENDING_REVIEW, PUBLISHED, ARCHIVED), OrderStatus, PaymentStatus, FulfillmentStatus, DiscountType, AuditAction

### 4.5 Environment Variables

| Variable | Purpose |
|----------|---------|
| `PORT` | Server port (4000) |
| `DATABASE_URL` | PostgreSQL connection |
| `JWT_SECRET` | JWT signing key |
| `JWT_REFRESH_SECRET` | Refresh token key |
| `ADMIN_SERVICE_KEY` | SSR admin service key |
| `STRIPE_SECRET_KEY` | Stripe payment |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook verification |
| `S3_*` | Object storage (MinIO in dev) |
| `SMTP_*` | Email via SendGrid |
| `CORS_ORIGIN` | Allowed origins |
| `USE_SHOPIFY` | Feature flag (false = self-hosted) |

---

## 5. Infrastructure

### 5.1 Docker Compose Services

| Service | Image | Port | Description |
|---------|-------|------|-------------|
| `postgres` | postgres:15-alpine | 5433:5432 | Database |
| `minio` | minio/minio:latest | 9000/9001 | S3-compatible storage |
| `backend` | Build: ./backend | 4000 | API server |
| `storefront` | Build: ./storefront | 3000 | Customer site |
| `admin` | Build: ./admin | 3001 | Admin panel |
| `nginx` | nginx:alpine | 80/443 | Reverse proxy |

**Docker Networks:** `divamos-network` (bridge)  
**Docker Volumes:** `postgres_data`, `minio_data`

### 5.2 Nginx Configuration

| Domain | Backend | Features |
|--------|---------|----------|
| `divamos.com` | storefront:3000 | SSL (TLS 1.2/1.3), gzip, static cache (60d) |
| `admin.divamos.com` | admin:3001 | SSL, rate limit (10r/s), conn limit (20), `X-Frame-Options: DENY` |

### 5.3 Monorepo

- **Manager:** pnpm 8.10.0
- **Node:** ≥18.0.0
- **Packages:** storefront, admin, backend, shared
- **Shared TypeScript:** ^5.3.0

---

## 6. Testing

### 6.1 E2E Tests (Playwright 1.58.2)

| File | Tests | Coverage |
|------|-------|----------|
| `admin-flows.spec.ts` | 11 | Login, dashboard, navigation, auth guard, sign out |
| `storefront-flows.spec.ts` | 12 | Homepage, search, cart, customer auth, mobile nav |
| `admin-storefront-sync.spec.ts` | 8 | Product sync, inventory, prices, search |
| `api-sync.spec.ts` | 5 | API health, auth, product lists |
| `full-coverage.spec.ts` | 12+ | Full storefront + admin + API coverage |

**Last Run:** 54 passed, 4 skipped, 0 failed • 28 screenshots

### 6.2 Unit Tests

- Backend: Vitest + Supertest
- Frontend: Vitest + Testing Library

---

## 7. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        DIVAMOS                              │
│           Shopify-Compatible Fashion E-Commerce            │
└─────────────────────────────────────────────────────────────┘

                    ┌───────────────┐
                    │    Nginx      │
                    │   Port 80/443 │
                    │  SSL + Rate   │
                    └───┬───────┬───┘
                        │       │
        divamos.com     │       │    admin.divamos.com
                        ▼       ▼
  ┌──────────────────┐     ┌──────────────────┐
  │   Storefront     │     │   Admin Panel    │
  │   Next.js 14     │     │   Next.js 14     │
  │   Port 3000      │     │   Port 3001      │
  │                  │     │                  │
  │  • App Router    │     │  • Dashboard     │
  │  • Zustand       │     │  • Recharts      │
  │  • Framer Motion │     │  • Radix UI      │
  │  • 27 pages      │     │  • 11 pages      │
  └────────┬─────────┘     └────────┬─────────┘
           │                        │
           └────────┬───────────────┘
                    │  HTTP / REST
                    ▼
  ┌──────────────────────────────────────────┐
  │          Backend API (Express 4)         │
  │           TypeScript + Prisma            │
  │              Port 4000                   │
  │                                          │
  │  Auth: JWT + Argon2 | Payment: Stripe   │
  │  Storage: S3/MinIO  | Email: SendGrid   │
  │  Security: Helmet + Rate Limit + CORS   │
  │                                          │
  │  22 DB Models • 50+ API Endpoints       │
  └──────┬──────────────────┬───────────────┘
         │                  │
    ┌────▼───┐         ┌───▼────┐
    │Postgres│         │ MinIO  │
    │   15   │         │  (S3)  │
    │Port 5433│        │Port 9000│
    └────────┘         └────────┘
         ▲
         │ Retail Sync
    ┌────┴────────────┐
    │   WK POS API    │
    │   Port 5000     │
    └─────────────────┘
```
