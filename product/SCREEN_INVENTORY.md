# Screen Inventory (as built)

**Baseline:** commit `962a7e58`, 2026-11-02. Lists every screen that exists in source, where it is mounted, and the API it calls. Required screens that do not exist are in the [master matrix](SCREEN_FEATURE_COMPLETION_MATRIX.md) as NOT STARTED.

**Guard column:**
- `ProtectedRoute(role / permission)` is the shell's route check on the platform token.
- `Gate(APP)` is the shared `ApplicationGate`: a tenant application sign-in.

Both are conveniences. Every API listed is authorized again by the gateway and by the owning service.

| Route (shell) | Workspace / component | Guard | API base | Matrix rows |
|---|---|---|---|---|
| `/` | shell `HomePage` | public | `/api/products` | SF-01 |
| `/login` | shell `LoginPage` | public | `/api/users/login` | SH-01 |
| `/products/*` | mfe-product `ProductPage` | public | `/api/products`, wishlist, reviews | SF-02..SF-06, SF-18, SF-21 |
| `/products/admin` | mfe-product `ProductAdminPage` | Gate(ECOMMERCE_ADMIN) | `/api/products/admin` | EC-03, EC-04 |
| `/products/pricing` | mfe-product `PricingPromotionsPage` | Gate(ECOMMERCE_ADMIN) | `/api/products/admin/pricing` | EC-13..EC-18 |
| `/products/storefront` | mfe-product `StorefrontManagementPage` | Gate(ECOMMERCE_ADMIN) | `/api/products/admin/storefront` | EC-31, EC-32 |
| `/cart/*` | mfe-cart `CartPage`, `CartDrawer` | signed in | `/api/cart` | SF-07 |
| `/cart/admin/abandoned` | mfe-cart `AbandonedCartsAdminPage` | staff | `/api/cart/admin` | EC-01 |
| `/orders/` | mfe-order `OrderPage` (checkout) | signed in | `/api/orders` | SF-08, SF-12 |
| `/orders/history` | mfe-order `CustomerOrdersPage` | signed in | `/api/orders`, `/api/orders/returns/user/{id}` | SF-13, SF-14, SF-16 |
| `/orders/control` | mfe-order `OrderControlTower` (dashboard, orders, details, exceptions, bulk, views, settings) | Gate(ECOMMERCE_ADMIN) | `/api/orders/admin` | EC-19..EC-21 |
| `/orders/returns` | mfe-order `ReturnsConsole`, policy screens | Gate(ECOMMERCE_ADMIN) | `/api/orders/admin/rma` | EC-22, EC-23 |
| `/orders/analytics`, `/orders/admin/analytics` | mfe-order `AdminAnalyticsDashboard` | Gate(ECOMMERCE_ADMIN) | `/api/orders/admin/analytics` | EC-34 |
| `/orders/ledger`, `/orders/ledger/admin` | mfe-order ledgers | signed in / Gate(ACCOUNTING) | `/api/accounting` | ACC-14 |
| `/profile/*` | mfe-user `ProfilePage`, `AddressBookPage` | signed in | `/api/users/{id}` | SF-09, SF-19 |
| `/profile/admin` | mfe-user `AdminUserList` | `roles:manage` | `/api/users`, `/api/access` | SA-22..SA-26 |
| `/loyalty/*` | mfe-loyalty `LoyaltyDashboardPage` | signed in | `/api/loyalty` | SF-24 |
| `/loyalty/admin` | mfe-loyalty `LoyaltyAdminPage` | Gate(ECOMMERCE_ADMIN) | `/api/loyalty/admin` | CRM-16 |
| `/accounting/*` | mfe-accounting customer ledger; admin ledger and `AdvancedAccountingPages` | `accounting:read` + Gate(ACCOUNTING) | `/api/accounting`, `/api/accounting/admin` | ACC-* |
| `/inventory/*` | mfe-inventory (sign-in, catalog, stock admin, reservations, ops workspace) | `inventory:read` + own INVENTORY sign-in | `/api/inventory`, `/api/inventory/ops` | INV-01..INV-13, INV-20, INV-25 |
| `/stock/*` | mfe-stock `StockAdminPage`, `ReservationsInspector` | `stock:read` + Gate(ECOMMERCE_ADMIN) | `/api/stock` | EC-12 |
| `/procurement/*` | mfe-procurement (overview, suppliers, purchase orders, supplier invoices, replenishment) | `procurement:manage` + Gate | `/api/procurement`, `/api/procurement/ucp` | INV-14..INV-18, ACC-26 |
| `/crm/*` | mfe-crm (overview, contacts, companies, opportunities, pipeline, tasks, segments, import and export) | `crm:read` + Gate(ECOMMERCE_ADMIN) | `/api/crm` | CRM-* |
| `/analytics/*` | mfe-analytics `AdminAnalyticsDashboard` | `analytics:read` + Gate | `/api/orders/admin/analytics` | EC-34 |
| `/error-stack/*` | mfe-error-stack (overview, groups, details) | `errors:manage` + Gate | `/api/errors/admin` | SA-43 |
| `/admin/quality-center` | shell `QualityCenterPage` (overview, features, tests, runs, RBAC matrix, coverage) | `qa:read` + Gate(QUALITY_CENTER) | `/api/admin/qa` | QA-* |
| (standalone, port 3013) | mfe-super-admin (tenants, applications, members, grants) | own sign-in, `application:admin` | `/api/platform/tenants` | SA-02..SA-11 |
| (standalone, port 3014) | mfe-delivery (overview, deliveries, dispatch, partners, trips, incidents, earnings, reports, zones, pricing, configuration, audit, partner profile) | own DELIVERY_OPS sign-in | `/api/delivery` | DLV-* |
| (component) | shell `NotificationBell` | signed in | `/api/notifications` | SF-22, SH-05 |

**Counts:** 16 frontend workspaces; 31 files under `pages/`, plus the section components of the control tower, returns, pricing, storefront and operations workspaces. No screen uses mock data (searched for mock, dummy, sample, placeholder and `Math.random`; the only random values are idempotency keys and a cart session id).
