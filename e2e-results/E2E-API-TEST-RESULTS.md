# E2E API Test Results

**Date**: 2026-02-16 18:59:38  
**Environment**: Development (localhost)  
**Total Tests**: 44 | **Passed**: 41 | **Failed**: 3

---

## WK POS API (port 5000)

| # | Test | Method | Status | Code | Time |
|---|------|--------|--------|------|------|
| 1 | Health | GET | PASS | 200 | 66ms |
| 2 | Health Ready | GET | PASS | 200 | 29ms |
| 3 | Health Live | GET | PASS | 200 | 8ms |
| 4 | Auth Login | POST | PASS | 200 | 174ms |
| 5 | Auth Me | GET | PASS | 200 | 24ms |
| 6 | Dashboard | GET | PASS | 200 | 2728ms |
| 7 | Products | GET | PASS | 200 | 80ms |
| 8 | Categories | GET | PASS | 200 | 44ms |
| 9 | Customers | GET | PASS | 200 | 23ms |
| 10 | Branches | GET | PASS | 200 | 19ms |
| 11 | Sales | GET | PASS | 200 | 46ms |
| 12 | Users | GET | PASS | 200 | 23ms |
| 13 | Settings | GET | PASS | 200 | 20ms |
| 14 | Shifts | GET | PASS | 200 | 29ms |
| 15 | Suppliers | GET | PASS | 200 | 22ms |
| 16 | Audit Logs | GET | PASS | 200 | 18ms |
| 17 | Notifications | GET | PASS | 200 | 39ms |
| 18 | Coupons | GET | PASS | 200 | 28ms |
| 19 | Purchase Orders | GET | PASS | 200 | 43ms |
| 20 | Stock Transfers | GET | PASS | 200 | 42ms |
| 21 | Bad Login | POST | PASS | 401 | 21ms |
| 22 | No Auth | GET | PASS | 401 | 3ms |
## License Server (port 3100)

| # | Test | Method | Status | Code | Time |
|---|------|--------|--------|------|------|
| 1 | Health | GET | PASS | 200 | 17ms |
| 2 | Ready | GET | PASS | 200 | 10ms |
| 3 | Admin Login | POST | PASS | 200 | 248ms |
| 4 | Plans Public | GET | PASS | 200 | 18ms |
| 5 | Modules Available | GET | PASS | 200 | 17ms |
| 6 | Modules List | GET | FAIL | 401 | 1ms |
| 7 | Dashboard | GET | PASS | 200 | 28ms |
| 8 | Licenses | GET | PASS | 200 | 12ms |
| 9 | Tenants | GET | PASS | 200 | 13ms |
| 10 | Profile | GET | PASS | 200 | 11ms |
| 11 | Statistics | GET | PASS | 200 | 17ms |
| 12 | Audit Logs | GET | PASS | 200 | 13ms |
| 13 | Serials | GET | PASS | 200 | 16ms |
| 14 | Trial Requests | GET | PASS | 200 | 12ms |
| 15 | Plans Admin | GET | PASS | 200 | 19ms |
## Divamos E-Commerce (port 4000)

| # | Test | Method | Status | Code | Time |
|---|------|--------|--------|------|------|
| 1 | Health | GET | PASS | 200 | 2091ms |
| 2 | API Info | GET | PASS | 200 | 10ms |
| 3 | Products | GET | FAIL | 500 | 203ms |
| 4 | Collections | GET | PASS | 200 | 23ms |
| 5 | Search | GET | PASS | 200 | 19ms |
| 6 | Categories | GET | PASS | 200 | 14ms |
| 7 | Admin Login | POST | FAIL | 500 | 20ms |

## Failure Details
| System | Test | Code | Detail |
|--------|------|------|--------|
| LICENSE | Modules List | 401 |  |
| DIVAMOS | Products | 500 |  |
| DIVAMOS | Admin Login | 500 |  |

## Test Environment

- **WK POS**: Express 4.18.2, PostgreSQL (port 5432), Redis, BullMQ
- **License Server**: Express 4.18.2, PostgreSQL (port 5434)
- **Divamos**: Express 4.18.2, PostgreSQL (port 5433)
- **Node.js**: v22.13.1
- **OS**: Windows

## Notes

- WK POS running with SKIP_LICENSE_CHECK=true for E2E testing (license middleware bypassed)
- Divamos has known double-path bugs in public routes (e.g., /api/v1/products/products instead of /api/v1/products)
- License Server freshly seeded with canonical seed data
- All 3 PostgreSQL databases accessible and connected
