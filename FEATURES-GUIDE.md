# WK POS Enterprise — Complete Features Guide

> Last Updated: 2026-02-16  
> A comprehensive guide to every feature in the WK POS Enterprise system.  
> Written for non-technical readers — business owners, managers, and evaluators.

---

## Table of Contents

- [System Overview](#system-overview)
- [1. Point of Sale (POS)](#1-point-of-sale-pos)
- [2. Inventory Management](#2-inventory-management)
- [3. Customer Management & CRM](#3-customer-management--crm)
- [4. Employee & HR Management](#4-employee--hr-management)
- [5. Accounting & Finance](#5-accounting--finance)
- [6. Reporting & Analytics](#6-reporting--analytics)
- [7. E-Commerce Integration](#7-e-commerce-integration)
- [8. Multi-Branch Operations](#8-multi-branch-operations)
- [9. Security & Compliance](#9-security--compliance)
- [10. Hardware Integration](#10-hardware-integration)
- [11. Licensing & Plans](#11-licensing--plans)
- [12. Admin Panel (Web Dashboard)](#12-admin-panel-web-dashboard)
- [13. Mobile App](#13-mobile-app)

---

## System Overview

WK POS Enterprise is a complete business management platform that includes:

| Component | What It Is | Who Uses It |
|-----------|-----------|-------------|
| **POS System** | Cash register + sales processing | Cashiers, Sales Staff |
| **Admin Panel** | Web dashboard for management | Managers, Admins, Owners |
| **Desktop App** | Windows application (wraps admin panel) | All users |
| **E-Commerce Store** | Online shopping website (Divamos) | Customers (online) |
| **Mobile App** | iOS/Android app | Sales staff on the floor |
| **License Server** | Controls access & subscriptions | Automatic (background) |
| **Edge Agent** | Offline capability at each branch | Automatic (background) |

### Supported Languages
- English
- Arabic (RTL layout support)

### Deployment Model
- **On-Premise / LAN**: Installed on your own server(s) in your office or store
- No cloud dependency required — everything runs on your local network
- Internet needed only for: license validation, e-commerce, and software updates

---

## 1. Point of Sale (POS)

### 1.1 Basic Sales

| Feature | Description |
|---------|-------------|
| **Quick Sale** | Scan barcode or search products, add to cart, process payment |
| **Barcode Scanning** | Supports EAN-13, UPC-A, internal codes, weighted barcodes |
| **Product Search** | Search by name, SKU, barcode, category |
| **Price Override** | Manager can override price (with permission) |
| **Quantity Adjustment** | Change quantity before finalizing |
| **Item Notes** | Add notes per item (e.g., "no onion") |
| **Product Modifiers** | Add-ons like "Extra cheese", "Large size" |

### 1.2 Payment Processing

| Feature | Description |
|---------|-------------|
| **Cash** | Accept cash, auto-calculate change |
| **Card** | Credit/debit card via terminal |
| **Split Payment** | Pay with multiple methods (e.g., $50 cash + $30 card) |
| **Multi-Currency** | Accept foreign currency with exchange rate |
| **Gift Cards** | Accept gift card as payment |
| **Store Credit** | Apply store credit balance |
| **Mobile Payment** | Mobile wallets, QR codes |
| **Bank Transfer** | Direct bank transfer |
| **Credit/Installment** | Sell on credit with installment plans |

### 1.3 Sale Management

| Feature | Description |
|---------|-------------|
| **Hold/Park Sales** | Save a sale to continue later (customer forgot wallet) |
| **Resume Held Sale** | Pick up a parked sale and complete it |
| **Cancel Sale** | Cancel an entire sale |
| **Void Sale** | Void same-day sales (before Z-Report) |
| **Refund** | Process full or partial refunds |
| **Exchange** | Exchange items (return one, buy another) |
| **Duplicate Invoice** | Reprint any past invoice |

### 1.4 Invoice & Receipt

| Feature | Description |
|---------|-------------|
| **Thermal Receipt** | 58mm or 80mm thermal printer support |
| **A4 Invoice** | Full-page invoice printing |
| **PDF Generation** | Download invoice as PDF |
| **Custom Templates** | Customize header, footer, logo, layout per branch |
| **Arabic/English** | Bilingual receipt support |
| **Barcode on Receipt** | Invoice barcode for easy re-lookup |
| **QR Code** | QR code for digital receipts |
| **E-Invoice (ETA)** | Electronic invoicing for tax authority compliance |

---

## 2. Inventory Management

### 2.1 Product Catalog

| Feature | Description |
|---------|-------------|
| **Categories** | Organize products in nested categories (e.g., Electronics > Phones > Samsung) |
| **Product Variants** | Size, color, material — each with own SKU, price, stock |
| **Product Tags** | Label products (e.g., "Best Seller", "New Arrival") |
| **Custom Fields** | Add any field to products (e.g., "Warranty Period") |
| **Bulk Import** | Import products from CSV/Excel |
| **Bulk Export** | Export product catalog |
| **Product Images** | Multiple images per product |
| **Product Attributes** | Define attribute types (Color, Size) with values |
| **Tax Configuration** | Per-product tax rate and type (inclusive/exclusive) |
| **Multi-Price** | Retail, wholesale, VIP, and cost price per variant |

### 2.2 Stock Management

| Feature | Description |
|---------|-------------|
| **Stock Levels** | Real-time stock per variant per branch |
| **Low Stock Alerts** | Automatic alerts when stock drops below threshold |
| **Out of Stock Alerts** | Immediate notification when stock hits zero |
| **Overstock Alerts** | Warning when stock exceeds maximum level |
| **Stock Adjustment** | Manual adjustments with reason (damage, theft, correction) |
| **Stock History** | Complete history of every stock change |
| **Negative Stock Prevention** | Optional: block sales when stock is zero |
| **Allow Negative** | Optional: allow selling below zero (backorder) |

### 2.3 Batch & Serial Tracking

| Feature | Description |
|---------|-------------|
| **Batch Tracking** | Track items by batch number (e.g., food production) |
| **Expiry Date Tracking** | Alerts for items expiring soon |
| **Serial Number Tracking** | Track individual items by serial number |
| **Serial Status History** | Full lifecycle of each serialized item |
| **Batch Consumption** | FIFO consumption of batches |

### 2.4 Stock Transfers

| Feature | Description |
|---------|-------------|
| **Transfer Request** | Request to move stock from one branch to another |
| **Approval Workflow** | Manager approval before transfer is processed |
| **Shipment Tracking** | Track transfer status (pending → shipped → received) |
| **Partial Receiving** | Receive less than shipped (record discrepancy) |
| **Transfer Reports** | History and analytics of transfers |

### 2.5 Physical Stock Counts (Stocktaking)

| Feature | Description |
|---------|-------------|
| **Full Count** | Count ALL items in a branch |
| **Partial Count** | Count specific categories or products |
| **Cycle Count** | Regular scheduled counting program |
| **Spot Check** | Quick count of selected items |
| **Variance Report** | System vs physical count differences |
| **Import Count Sheet** | Import count data from spreadsheet |
| **Export Template** | Download counting template |
| **Approval & Posting** | Manager approves adjustments before applying |

### 2.6 Purchasing

| Feature | Description |
|---------|-------------|
| **Purchase Orders** | Create and send POs to suppliers |
| **Supplier Management** | Supplier profiles, contact info, payment terms |
| **Goods Receiving** | Record received goods against POs |
| **Partial Receiving** | Receive a portion of a PO |
| **Reorder Suggestions** | Auto-generated suggestions based on stock levels and sales velocity |
| **Backorder Management** | Track items ordered but not yet available |
| **Supplier Payments** | Record payments to suppliers |

### 2.7 Reservations

| Feature | Description |
|---------|-------------|
| **Stock Reservations** | Reserve stock for specific purposes (hold, online order, layaway) |
| **Automatic Expiry** | Reservations auto-release after configured time |
| **Fulfillment** | Convert reservation to sale |
| **Availability Check** | Real-time available-to-sell calculation |

---

## 3. Customer Management & CRM

### 3.1 Customer Profiles

| Feature | Description |
|---------|-------------|
| **Customer Database** | Complete customer profiles (name, email, phone, address) |
| **Customer Groups** | Group customers (VIP, Wholesale, Regular) with group pricing |
| **Purchase History** | Every transaction linked to customer |
| **Total Spend Tracking** | Lifetime value calculation |
| **Tax Number** | Store customer tax/VAT number |
| **Individual or Company** | Support both customer types |

### 3.2 Loyalty Programs

| Feature | Description |
|---------|-------------|
| **Points-Based** | Earn X points per $ spent |
| **Cashback** | Earn % of purchase as credit |
| **Tiered Loyalty** | Bronze → Silver → Gold → Platinum tiers |
| **Point Redemption** | Use points as payment |
| **Tier Benefits** | Higher tiers earn faster, get bigger discounts |
| **Point Balance** | Track customer point balance |
| **Transaction History** | Complete loyalty activity log |

### 3.3 Gift Cards

| Feature | Description |
|---------|-------------|
| **Create Gift Cards** | Generate gift cards with set value |
| **Sell Gift Cards** | Sell as product |
| **Balance Check** | Check remaining balance |
| **Redeem** | Use as payment method |
| **Reload** | Add more value to existing card |
| **Cancel** | Cancel/deactivate a card |
| **History** | Track all card transactions |

### 3.4 CRM (Customer Relationship Management)

| Feature | Description |
|---------|-------------|
| **Lead Management** | Track potential customers through sales pipeline |
| **Pipeline Stages** | Customizable stages (New → Contacted → Qualified → Proposal → Won/Lost) |
| **Deals** | Track deal value, stage, expected close date |
| **Activities** | Log calls, emails, meetings, notes per lead/deal/customer |
| **Quotations** | Create formal price quotes for customers |
| **Convert to Sale** | Convert accepted quotation into a sale |
| **CRM Dashboard** | Pipeline overview, activity calendar, revenue forecast |

### 3.5 Promotions & Discounts

| Feature | Description |
|---------|-------------|
| **Coupons** | Create coupon codes with fixed or percentage discount |
| **Automatic Discounts** | Rules-based discounts (buy 2 get 1 free, 10% off for VIP) |
| **Discount Stacking** | Configure how multiple discounts combine |
| **Usage Limits** | Set maximum coupon uses |
| **Date Ranges** | Active date range for promotions |
| **Minimum Purchase** | Require minimum amount for discount |

---

## 4. Employee & HR Management

### 4.1 Employee Profiles

| Feature | Description |
|---------|-------------|
| **Employee Records** | Full profile (name, position, department, salary) |
| **User Account Linking** | Each employee linked to a user account |
| **Employee Number** | Auto-generated employee numbers |
| **Department Management** | Organize by department |

### 4.2 Attendance

| Feature | Description |
|---------|-------------|
| **Clock In / Out** | Via POS system or mobile |
| **Break Tracking** | Record break start/end times |
| **Overtime Calculation** | Automatic overtime hours calculation |
| **Attendance Reports** | Daily, weekly, monthly summaries |
| **Manual Entry** | Manager can add/edit attendance records |
| **Geofencing** | (Mobile) Ensure clock-in from correct location |

### 4.3 Leave Management

| Feature | Description |
|---------|-------------|
| **Leave Types** | Configure leave types (Annual, Sick, Personal, etc.) |
| **Leave Requests** | Employees submit leave requests |
| **Approval Workflow** | Manager approves or rejects |
| **Balance Tracking** | Automatic balance tracking per year |
| **Calendar View** | Visual leave calendar |

### 4.4 Payroll

| Feature | Description |
|---------|-------------|
| **Salary Components** | Earnings (basic, allowances) + Deductions + Benefits |
| **Payroll Periods** | Monthly payroll processing |
| **Payslip Generation** | Detailed payslips per employee |
| **Bulk Processing** | Process all payslips at once |
| **Bank Payment Export** | Export for bank transfer |
| **Payroll Summary** | Total costs by department |

### 4.5 Commissions

| Feature | Description |
|---------|-------------|
| **Commission Rules** | Percentage, fixed, or tiered commission structures |
| **Per-Category Rules** | Different commission rates by product category |
| **Per-Product Rules** | Commission on specific products |
| **Auto-Calculate** | Automatically calculated on each sale |
| **Approval Workflow** | Manager approves before payout |
| **Commission Reports** | Summary by employee, period, product |

---

## 5. Accounting & Finance

### 5.1 Chart of Accounts

| Feature | Description |
|---------|-------------|
| **Account Types** | Asset, Liability, Equity, Revenue, Expense |
| **Hierarchical** | Parent-child account structure |
| **Bilingual** | English + Arabic account names |
| **System Accounts** | Pre-configured essential accounts |
| **Custom Accounts** | Create additional accounts as needed |

### 5.2 Journal Entries

| Feature | Description |
|---------|-------------|
| **Manual Entries** | Create debit/credit journal entries |
| **Auto Entries** | Sales auto-generate journal entries |
| **Status Workflow** | Draft → Posted → Voided |
| **Change Log** | Full audit trail of entry modifications |

### 5.3 Expenses

| Feature | Description |
|---------|-------------|
| **Expense Recording** | Record business expenses |
| **Category Tracking** | Categorize expenses |
| **Branch Allocation** | Assign expenses to branches |
| **GL Integration** | Auto-post to general ledger |

### 5.4 Financial Reports

| Feature | Description |
|---------|-------------|
| **Trial Balance** | All account balances |
| **Profit & Loss (P&L)** | Revenue vs expenses |
| **Balance Sheet** | Assets, liabilities, equity snapshot |
| **Cost Center Reports** | Expenses by cost center |

### 5.5 Fixed Assets

| Feature | Description |
|---------|-------------|
| **Asset Register** | Track all company assets |
| **Depreciation** | Straight-line, declining balance, units of production |
| **Depreciation Schedule** | Automated depreciation calculations |
| **Disposal** | Record asset sales or write-offs |

### 5.6 Banking

| Feature | Description |
|---------|-------------|
| **Bank Accounts** | Multiple bank account management |
| **Transaction Recording** | Deposits, withdrawals, transfers, fees |
| **Bank Reconciliation** | Match bank statements to records |
| **Multi-Currency** | Support for foreign currency accounts |

### 5.7 Installment Plans

| Feature | Description |
|---------|-------------|
| **Create Plans** | Set up installment payment plans for customers |
| **Payment Tracking** | Track each installment payment |
| **Overdue Alerts** | Notifications for overdue payments |
| **Status Management** | Active → Completed / Defaulted |

---

## 6. Reporting & Analytics

### 6.1 Standard Reports

| Report | What It Shows |
|--------|--------------|
| **Sales Summary** | Total sales, average sale, transaction count |
| **Sales by Category** | Revenue breakdown by product category |
| **Sales by Payment** | Revenue breakdown by payment method |
| **Sales by SKU** | Individual product/variant performance |
| **Sales by Hour** | Hourly sales patterns (peak hours) |
| **Sales by Branch** | Branch comparison |
| **Sales Trend** | Sales over time (daily/weekly/monthly) |
| **Top Products** | Best-selling products |
| **Profit by Product** | Profit margin per product |
| **Category Performance** | Category-level performance metrics |
| **Inventory Report** | Current stock values |
| **Low Stock Alert** | Products below reorder point |
| **Customer Report** | Customer analysis and segmentation |
| **Customer Purchase History** | Per-customer transaction history |
| **Employee Performance** | Sales per employee |
| **Cashier Performance** | Cashier-level metrics |
| **Tax Summary** | Tax collected by rate/type |
| **VAT Report** | VAT-specific report for tax filing |
| **Cost of Goods Sold** | COGS analysis |
| **Branch Performance** | Multi-branch comparison |

### 6.2 Data Export

| Format | Supported |
|--------|-----------|
| **PDF** | All reports |
| **CSV** | Sales, inventory, customers, products, P&L |
| **Excel** | All export types |

### 6.3 Custom Report Builder

| Feature | Description |
|---------|-------------|
| **Drag & Drop** | Build custom reports with drag-and-drop fields |
| **Save Reports** | Save custom reports for reuse |
| **Share Reports** | Share with other users (view or export permission) |
| **Schedule Reports** | Auto-generate reports daily/weekly/monthly |
| **Email Delivery** | Reports emailed to recipients automatically |
| **Templates** | Pre-built report templates as starting point |

---

## 7. E-Commerce Integration

### 7.1 Divamos Online Store

| Feature | Description |
|---------|-------------|
| **Product Catalog** | Products automatically synced from POS |
| **Category Pages** | Browse by category |
| **Product Search** | Full-text product search |
| **Collections** | Curated product collections |
| **Product Variants** | Size/color selection on product pages |
| **Product Images** | Multiple images per product |
| **Shopping Cart** | Add to cart, update quantities |
| **Guest Checkout** | Purchase without creating account |
| **Customer Accounts** | Register, login, order history |
| **Google Login** | Sign in with Google |
| **Discount Codes** | Apply coupon codes at checkout |
| **Gift Card Payment** | Use gift cards from POS system |
| **Loyalty Points** | Redeem loyalty points online |
| **Multiple Addresses** | Manage billing/shipping addresses |
| **Order Tracking** | Track order status |
| **Stripe Payments** | Secure online payments via Stripe |

### 7.2 Admin CMS

| Feature | Description |
|---------|-------------|
| **Homepage Builder** | Configure homepage sections/layout |
| **Page Management** | Custom content pages |
| **Media Library** | Upload and manage product images |
| **Store Settings** | Store name, description, contact info |
| **Analytics Dashboard** | Online store performance metrics |

### 7.3 POS ↔ E-Commerce Sync

| Feature | Description |
|---------|-------------|
| **Product Sync** | Products from POS appear on website automatically |
| **Price Sync** | Price changes reflected on website |
| **Stock Sync** | Live stock levels on website |
| **Order Fulfillment** | Online orders appear in POS for fulfillment |
| **Category Sync** | Categories synced from POS |
| **Coupon Sync** | POS coupons available online |
| **Gift Card Sync** | Gift cards redeemable online |
| **Auto Sync** | Configurable automatic sync interval |
| **Manual Sync** | Trigger sync on demand |
| **Sync Logs** | View history of all sync operations |

---

## 8. Multi-Branch Operations

### 8.1 Branch Management

| Feature | Description |
|---------|-------------|
| **Unlimited Branches** | Create as many branches as your plan allows |
| **Branch Profiles** | Name, address, phone per branch |
| **Main/HQ Branch** | Designate one as headquarters |
| **Branch-Specific Settings** | Printer config, receipt template per branch |
| **Branch Stock View** | See stock levels at any branch |

### 8.2 Data Synchronization

| Feature | Description |
|---------|-------------|
| **Real-Time Sync** | Changes propagate in real-time when online |
| **Offline Capability** | Continue operating when internet is down |
| **Conflict Resolution** | Automatic and manual conflict handling |
| **Sync Dashboard** | Monitor sync status across all branches |
| **Force Sync** | Manually trigger immediate sync |
| **Master Data Push** | Push products/prices from HQ to all branches |

### 8.3 Offline Operations (Edge Agent)

| Feature | Description |
|---------|-------------|
| **Offline Sales** | Process sales without internet |
| **Local Data Cache** | Products, prices, customers cached locally |
| **Automatic Reconnection** | Detects internet and syncs automatically |
| **Offline License** | Works up to 30 days without license server |
| **Data Integrity** | WAL (Write-Ahead Log) prevents data loss |

---

## 9. Security & Compliance

### 9.1 User Access Control

| Feature | Description |
|---------|-------------|
| **Role-Based Access** | 7+ built-in roles (Owner, Admin, Manager, Cashier, etc.) |
| **Custom Roles** | Create custom roles with specific permissions |
| **Multi-Level RBAC** | Permission checks at route, module, and resource level |
| **Access Control Lists** | Per-resource access rights |
| **ABAC Policies** | Attribute-based access policies (time, location, conditions) |
| **Permission Audit** | Log every permission check for compliance |

### 9.2 Authentication Security

| Feature | Description |
|---------|-------------|
| **JWT Tokens** | Secure stateless authentication |
| **Refresh Tokens** | Token rotation for security |
| **MFA (Two-Factor)** | TOTP or email-based second factor |
| **PIN Authentication** | Quick PIN login for cashiers |
| **Account Lockout** | Lock after X failed login attempts |
| **Session Management** | View/terminate active sessions |
| **Password History** | Prevent password reuse |
| **Password Expiry** | Force periodic password changes |
| **Rate Limiting** | Prevent brute force attacks |

### 9.3 Data Security

| Feature | Description |
|---------|-------------|
| **Tenant Isolation** | Complete data separation between businesses |
| **Audit Trail** | Every create/update/delete operation logged |
| **Data Encryption** | Sensitive data encrypted at rest |
| **GDPR Compliance** | Data export, anonymization, consent management |
| **PCI Compliance** | Card data masking, no CVV storage |
| **Data Retention** | Configurable retention policies |
| **API Key Security** | Hashed storage, expiry, permission scoping |

### 9.4 Compliance Features

| Feature | Description |
|---------|-------------|
| **ETA E-Invoicing** | Compatible with tax authority electronic invoicing |
| **VAT Reports** | Generate VAT-compliant reports |
| **Audit Immutability** | Verify audit log integrity |
| **Tax Number Validation** | Validate customer/business tax numbers |
| **Encryption Key Rotation** | Periodic key rotation for security |

---

## 10. Hardware Integration

### 10.1 Supported Devices

| Device Type | Capabilities |
|-------------|-------------|
| **Receipt Printers** | 58mm and 80mm thermal, A4 laser |
| **Barcode Scanners** | All standard USB/wireless scanners |
| **Cash Drawers** | Auto-open on sale, security monitoring |
| **Kitchen Display (KDS)** | Route orders to kitchen screens |
| **Weighing Scales** | Weight-based pricing, tare, stabilization |
| **Customer Displays** | Show items and payment prompts |
| **Card Terminals** | Credit/debit card payment devices |
| **Label Printers** | Shelf labels, price tags (ZPL support) |

### 10.2 Hardware Features

| Feature | Description |
|---------|-------------|
| **Auto-Detection** | Automatic device discovery |
| **Multi-Printer** | Different printers for receipt vs kitchen |
| **Printer Fallback** | If printer fails, use backup printer |
| **No-Sale Drawer Open** | Open drawer without a sale (requires manager) |
| **Drawer Monitoring** | Track drawer open events for security |
| **Weighted Barcode Parsing** | Handle weight-embedded barcodes |

---

## 11. Licensing & Plans

### 11.1 Plan Tiers

| Tier | Max Branches | Max Users | Max Products | Offline Days | Modules |
|------|-------------|-----------|-------------|-------------|---------|
| **Trial** | 1 | 3 | 100 | 3 | Basic POS only |
| **Basic** | 2 | 10 | 1,000 | 7 | Core POS + Inventory |
| **Professional** | 5 | 25 | 10,000 | 14 | Most modules |
| **Enterprise** | Unlimited | Unlimited | Unlimited | 30 | All 51 modules |

### 11.2 Licensed Modules (51 Total)

| Category | Modules |
|----------|---------|
| **Core POS** (9) | POS Core, Sales Management, Payment Processing, Barcode Management, Shift Management, Exchanges & Returns, Invoicing & Receipts, Product Modifiers, Parked/Held Sales |
| **Inventory** (10) | Inventory Core, Stock Transfers, Batch Tracking, Serial Tracking, Stock Counting, Reorder Management, Stock Audit, Backorder Management, Demand Forecasting, Stock Reservations |
| **Finance** (2) | Accounting, Installment Management |
| **Admin** (7) | User Management, Branch Management, Settings, Permissions & RBAC, Feature Flags, API Key Management, Custom Fields |
| **Marketing** (4) | Coupons & Promotions, Discount Engine, Loyalty Program, Gift Cards |
| **CRM** (4) | Customer Management, Customer Groups & Pricing, CRM (Leads/Deals), Quotations |
| **HR** (5) | Employee Management, Attendance Tracking, Leave Management, Payroll, Commission Management |
| **Analytics** (3) | Reports & Analytics, Scheduled Reports, Report Builder |
| **Integrations** (5) | E-Commerce Bridge, Website Sync, Webhook Management, ETA E-Invoicing, Third-Party Integrations |
| **Compliance** (2) | Audit & Compliance, Data Retention |
| **Hardware** (2) | Hardware Management, Printing & Templates |

### 11.3 License Management Features

| Feature | Description |
|---------|-------------|
| **Serial Key Activation** | Activate with unique serial key |
| **Device Fingerprinting** | Hardware-based device identification |
| **Online Validation** | Real-time license checking |
| **Offline Mode** | Continue working without internet (grace period) |
| **Auto-Renewal** | Stripe-based subscription renewal |
| **Trial to Paid** | Seamless trial conversion |
| **Admin Dashboard** | License management console |
| **Usage Analytics** | API usage tracking and metering |
| **Violation Detection** | Detect tampering, clock manipulation, device limits |

---

## 12. Admin Panel (Web Dashboard)

The admin panel has **55+ pages** organized into these sections:

### Dashboard
- Business overview with key metrics
- Sales charts and trends
- Quick actions (new sale, add product, etc.)
- Branch comparison view

### POS
- Full point-of-sale interface
- Product grid and barcode scanning
- Cart management and checkout

### Products
- Product catalog management
- Category tree management
- Variant management
- Bulk import/export
- Tag management

### Inventory
- Stock level overview
- Stock adjustments
- Stock transfers
- Stocktaking/physical counts
- Item movement history
- Stock control dashboard

### Sales
- Sales history
- Invoice viewing
- Refunds and exchanges
- Held/parked sales

### Customers
- Customer directory
- Customer group management
- Loyalty program settings
- Gift card management
- Coupon management

### CRM
- Lead pipeline
- Deal management
- Activity tracking
- Quotation creation

### Reports
- All standard reports
- Custom report builder
- Scheduled reports
- Data exports

### Accounting
- Chart of accounts
- Journal entries
- Expenses
- Fixed assets
- Bank reconciliation
- Financial statements

### HR & Payroll
- Employee directory
- Attendance tracking
- Leave management
- Payroll processing
- Commission management

### E-Commerce
- Retail sync center
- Website orders
- Website sync configuration

### Settings
- System settings
- User management
- Branch management
- Permission/role configuration
- Printer settings
- Security settings
- API key management
- Webhook configuration
- Feature flags

---

## 13. Mobile App

The mobile app (React Native / Expo) provides:

| Feature | Description |
|---------|-------------|
| **Mobile POS** | Process sales from phone/tablet |
| **Product Lookup** | Scan barcode to check price/stock |
| **Stock Check** | View real-time stock at any branch |
| **Quick Sale** | Simplified sale interface for mobile |
| **Attendance** | Clock in/out from mobile |
| **Push Notifications** | Alerts for low stock, new orders, approvals |
| **Dashboard** | Quick business metrics |
| **Offline Mode** | Basic features work without internet |

---

## System Requirements

### Server

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Storage | 20 GB SSD | 100+ GB SSD |
| OS | Windows 10+, Ubuntu 20.04+ | Ubuntu 22.04 LTS |
| Database | PostgreSQL 14+ | PostgreSQL 16 |
| Runtime | Node.js 18+ | Node.js 20 LTS |

### Client

| Component | Minimum |
|-----------|---------|
| Browser | Chrome 90+, Firefox 90+, Edge 90+ |
| Desktop App | Windows 10+ (64-bit) |
| Mobile | iOS 15+ or Android 10+ |
| Network | LAN (100 Mbps recommended) |

---

## Value Proposition

| Metric | Value |
|--------|-------|
| **Total API Endpoints** | 820+ |
| **Database Models** | 175+ |
| **Admin Panel Pages** | 55+ |
| **Licensed Modules** | 51 |
| **Supported Languages** | 2 (English + Arabic) |
| **Deployment** | On-premise / LAN |
| **Max Branches** | Unlimited (Enterprise) |
| **Offline Capability** | Up to 30 days |
| **Security Layers** | 14 middleware layers |
| **Report Types** | 29+ standard + unlimited custom |
| **Hardware Support** | 8+ device types |

This is a professional, **enterprise-grade POS system** valued at **$6,000 — $9,000 USD**, designed for businesses that need:
- Multi-branch operations
- Complete business management (not just a cash register)
- Offline resilience
- Strict security and compliance
- E-commerce integration
- Full accounting capabilities
