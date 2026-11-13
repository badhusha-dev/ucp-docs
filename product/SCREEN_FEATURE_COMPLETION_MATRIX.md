# Screen and Feature Completion Matrix (master tracking matrix)

**Baseline:** commit `962a7e58` on `feature/ucp-completion-modernization`, 2026-11-02. Measured from source, not from earlier documents.
Completion percentages are computed from this table by `scripts/product/completion.mjs` and written to [IMPLEMENTATION_PROGRESS.md](IMPLEMENTATION_PROGRESS.md). Edit the table, then re-run the script.

## Legend

| Column | Values |
|---|---|
| API, DB, Perm | `Y` exists and is used by the screen · `P` partial · `N` missing |
| UI test | `browser` (driven manually in a browser against the live stack) · `unit` (screen logic covered by a frontend unit test) · `N` none |
| Status | NOT STARTED · IN PROGRESS · BLOCKED · PARTIAL · COMPLETED · VERIFIED |

Status rules (from the product brief):
- **COMPLETED** means the frontend, backend, database, permission and business logic all exist for the screen.
- **VERIFIED** additionally requires the relevant automated tests (backend) and a manual browser check.
- Missing search, pagination or other screen standards (brief §5) keeps a screen at **PARTIAL**, as does missing core business behaviour, even when the screen already exists.

Role codes are defined in [ROLE_CATALOG.md](ROLE_CATALOG.md).

## Master matrix

| ID | Application | Role | Screen | Feature / evidence / gap | API | DB | Perm | UI test | Status |
|---|---|---|---|---|---|---|---|---|---|
| SA-01 | Super Admin | PLATFORM_OWNER, SUPER_ADMIN | Platform dashboard | No dashboard: mfe-super-admin opens on the tenant list. No metrics source for revenue or usage | N | N | Y | N | NOT STARTED |
| SA-02 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Tenant list | `GET /api/platform/tenants`; no search or pagination | Y | Y | Y | N | PARTIAL |
| SA-03 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Create tenant | `POST /api/platform/tenants` (slug, name) | Y | Y | Y | N | COMPLETED |
| SA-04 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Edit tenant | No update endpoint | N | Y | Y | N | NOT STARTED |
| SA-05 | Super Admin | SUPER_ADMIN, PLATFORM_OPS, PLATFORM_AUDITOR | Tenant details | Applications and members are shown; no tenant profile or configuration | P | Y | Y | N | PARTIAL |
| SA-06 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Tenant activation / suspension | `tenants.status` supports SUSPENDED and application sign-in requires ACTIVE; there is no endpoint or screen to change the status, and tokens already issued stay valid until they expire | N | Y | P | N | NOT STARTED |
| SA-07 | Super Admin | SUPER_ADMIN | Tenant configuration | No tenant settings model | N | N | N | N | NOT STARTED |
| SA-08 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Tenant users | List and add members, grant or revoke applications with a role | Y | Y | Y | N | COMPLETED |
| SA-09 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Tenant subscription | Only a `plan_code` column on entitlements | N | P | N | N | NOT STARTED |
| SA-10 | Super Admin | SUPER_ADMIN, PLATFORM_OPS | Tenant application access | `PUT /{tenantId}/applications/{code}` entitlements | Y | Y | Y | N | COMPLETED |
| SA-11 | Super Admin | SUPER_ADMIN, PLATFORM_AUDITOR | Tenant audit history | `GET /{tenantId}/audit` exists; not shown in the UI | Y | Y | Y | N | PARTIAL |
| SA-12 | Super Admin | SUPER_ADMIN | Application registry list | `application_registry` table; no registry endpoint or screen | N | Y | N | N | NOT STARTED |
| SA-13 | Super Admin | SUPER_ADMIN | Register / edit application | No endpoint | N | P | N | N | NOT STARTED |
| SA-14 | Super Admin | SUPER_ADMIN | Version, availability, URL, dependencies | Only `status` and `approved_origins` columns | N | P | N | N | NOT STARTED |
| SA-15 | Super Admin | SUPER_ADMIN | Application entitlement configuration | `required_permissions`, `supported_plan_codes` columns; not editable | N | P | N | N | NOT STARTED |
| SA-16 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Subscription plans and plan features | No plan model | N | N | N | N | NOT STARTED |
| SA-17 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Application packages and pricing | None | N | N | N | N | NOT STARTED |
| SA-18 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Subscription assignment | None | N | N | N | N | NOT STARTED |
| SA-19 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Renewals, upgrades, downgrades | None | N | N | N | N | NOT STARTED |
| SA-20 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE | Subscription suspension | None | N | N | N | N | NOT STARTED |
| SA-21 | Super Admin | SUPER_ADMIN, PLATFORM_FINANCE, PLATFORM_AUDITOR | Subscription history | None | N | N | N | N | NOT STARTED |
| SA-22 | Super Admin | SUPER_ADMIN | Platform users list | mfe-user `/profile/admin` lists all users; not separated into platform and tenant users; no search | P | Y | Y | N | PARTIAL |
| SA-23 | Super Admin | SUPER_ADMIN | Create platform user | Only self-registration exists | N | Y | N | N | NOT STARTED |
| SA-24 | Super Admin | SUPER_ADMIN | Assign platform roles | `PUT /api/users/{id}/role` assigns a platform role; V20 adds PLATFORM_OPS, PLATFORM_SUPPORT_AGENT, PLATFORM_AUDITOR, enforced by the platform API and gateway (`platform:*` permissions). Client-owned roles are refused | Y | Y | Y | N | COMPLETED |
| SA-25 | Super Admin | SUPER_ADMIN | Roles and permissions | `/api/access/roles` and `/permissions` manage built-in and platform roles; client-owned roles are hidden and protected; a role with a platform permission becomes a platform role (V20) | Y | Y | Y | N | COMPLETED |
| SA-26 | Super Admin | SUPER_ADMIN | Account activation / suspension | `PUT /api/users/{id}/status` | Y | Y | Y | N | COMPLETED |
| SA-27 | Super Admin | SUPER_ADMIN, PLATFORM_AUDITOR | Login activity | Every sign-in (successful, failed, locked, refused), application opening, password change and reset is recorded (V21 `account_security_events`, IP and device). User administration shows each person's sign-in activity (`GET /api/users/{id}/security-events`, `users:read`). Tests: `AccountSecurityPostgresTest`; live `live-account.sh` | Y | Y | Y | N | COMPLETED |
| SA-28 | Super Admin | PLATFORM_FINANCE | Subscription invoices | None | N | N | N | N | NOT STARTED |
| SA-29 | Super Admin | PLATFORM_FINANCE | Subscription payments | None, and no payment provider | N | N | N | N | NOT STARTED |
| SA-30 | Super Admin | PLATFORM_FINANCE | Subscription refunds | None | N | N | N | N | NOT STARTED |
| SA-31 | Super Admin | PLATFORM_FINANCE | Revenue reports | None | N | N | N | N | NOT STARTED |
| SA-32 | Super Admin | PLATFORM_FINANCE | Outstanding payments | None | N | N | N | N | NOT STARTED |
| SA-33 | Super Admin | PLATFORM_FINANCE | Subscription tax configuration | None | N | N | N | N | NOT STARTED |
| SA-34 | Super Admin | SUPER_ADMIN | Global settings | None | N | N | N | N | NOT STARTED |
| SA-35 | Super Admin | SUPER_ADMIN | Notification templates | notification-service has hardcoded message text | N | N | N | N | NOT STARTED |
| SA-36 | Super Admin | SUPER_ADMIN | Email configuration | None | N | N | N | N | NOT STARTED |
| SA-37 | Super Admin | SUPER_ADMIN | Branding | Storefront themes exist per storefront, not platform branding | N | N | N | N | NOT STARTED |
| SA-38 | Super Admin | SUPER_ADMIN | Localization | None | N | N | N | N | NOT STARTED |
| SA-39 | Super Admin | SUPER_ADMIN | Feature flags | None in the application (Flagger is deployment-only) | N | N | N | N | NOT STARTED |
| SA-40 | Super Admin | SUPER_ADMIN | Integration settings | None | N | N | N | N | NOT STARTED |
| SA-41 | Super Admin | PLATFORM_OPS | Service, application, database and Kafka health | Actuator health, Prometheus, Grafana and Kafka UI run in Compose; there is no in-app screen | P | N | N | N | PARTIAL |
| SA-42 | Super Admin | PLATFORM_OPS | Background jobs | None | N | N | N | N | NOT STARTED |
| SA-43 | Super Admin | PLATFORM_OPS, PLATFORM_SUPPORT_AGENT | System logs | Error Stack app (runtime errors) and Kibana; no log viewer in the platform UI | P | Y | Y | N | PARTIAL |
| SA-44 | Super Admin | PLATFORM_AUDITOR | Audit logs | `GET /api/access/audit` exists; no platform audit screen | P | Y | Y | N | PARTIAL |
| SA-45 | Super Admin | PLATFORM_SUPPORT_AGENT | Notification DLQ | `/api/notifications/dlq` list and replay; no screen | Y | Y | P | N | PARTIAL |
| EC-01 | E-Commerce Admin | TENANT_OWNER, STORE_ADMIN, ECOM_MANAGER | Store dashboard | Order control tower dashboard and sales analytics exist; no combined store dashboard | P | Y | Y | browser | PARTIAL |
| EC-02 | E-Commerce Admin | TENANT_OWNER, STORE_ADMIN | Store profile and settings | Only order settings exist; no store profile | P | P | Y | N | PARTIAL |
| EC-03 | E-Commerce Admin | PRODUCT_MANAGER, CATALOG_MANAGER | Product listing | Catalog admin loads live data (browser, round 3); shows the first 100; no search, filter or pagination | Y | Y | Y | browser | PARTIAL |
| EC-04 | E-Commerce Admin | PRODUCT_MANAGER, CATALOG_MANAGER | Product creation and editing | Create, edit, archive and restore with server validation | Y | Y | Y | N | COMPLETED |
| EC-05 | E-Commerce Admin | CATALOG_MANAGER | Product categories | Categories are read-only in the UI; no category administration | P | Y | P | N | PARTIAL |
| EC-06 | E-Commerce Admin | CATALOG_MANAGER | Attributes | No attribute model | N | N | N | N | NOT STARTED |
| EC-07 | E-Commerce Admin | CATALOG_MANAGER | Variants | `product_variant` table; no admin screen | P | Y | P | N | PARTIAL |
| EC-08 | E-Commerce Admin | CATALOG_MANAGER | SKU management | SKUs live in inventory items, not the catalog | P | Y | Y | N | PARTIAL |
| EC-09 | E-Commerce Admin | CATALOG_MANAGER | Brand management | None | N | N | N | N | NOT STARTED |
| EC-10 | E-Commerce Admin | CATALOG_MANAGER | Product media | Single image URL only; no upload | P | P | Y | N | PARTIAL |
| EC-11 | E-Commerce Admin | CATALOG_MANAGER | Bulk import / export | Inventory CSV import exists; catalog has none | N | N | N | N | NOT STARTED |
| EC-12 | E-Commerce Admin | WAREHOUSE_MANAGER, STORE_ADMIN | Inventory overview | Stock admin (tenant-scoped, B-02) | Y | Y | Y | unit | COMPLETED |
| EC-13 | E-Commerce Admin | PRICING_MANAGER | Pricing | Pricing and promotions workspace (`/api/products/admin/pricing`, 35 endpoints) | Y | Y | Y | unit | COMPLETED |
| EC-14 | E-Commerce Admin | PRICING_MANAGER | Price lists | Product prices only; no customer-group or branch price lists | P | P | Y | N | PARTIAL |
| EC-15 | E-Commerce Admin | PRICING_MANAGER | Scheduled prices | `scheduled_prices` with approval | Y | Y | Y | unit | COMPLETED |
| EC-16 | E-Commerce Admin | MARKETING_MANAGER, PRICING_MANAGER | Promotions | Campaigns, rules, conditions, rewards, approvals | Y | Y | Y | unit | COMPLETED |
| EC-17 | E-Commerce Admin | MARKETING_MANAGER | Coupons | Coupon codes and redemptions | Y | Y | Y | unit | COMPLETED |
| EC-18 | E-Commerce Admin | PRICING_MANAGER | Discount rules | Discount engine, tested | Y | Y | Y | unit | COMPLETED |
| EC-19 | E-Commerce Admin | ORDER_MANAGER | Orders | Control tower list with views, filters and bulk actions (browser, round 3) | Y | Y | Y | browser | VERIFIED |
| EC-20 | E-Commerce Admin | ORDER_MANAGER, CS_MANAGER | Order details | Details, notes, assignment, hold and cancellation | Y | Y | Y | unit | COMPLETED |
| EC-21 | E-Commerce Admin | ORDER_MANAGER, CS_MANAGER | Order timeline | `/orders/{id}/timeline` | Y | Y | Y | N | COMPLETED |
| EC-22 | E-Commerce Admin | ORDER_MANAGER, CS_MANAGER | Returns | RMA console (`/api/orders/admin/rma`, 45 endpoints) with inspections and policies | Y | Y | Y | unit | COMPLETED |
| EC-23 | E-Commerce Admin | FINANCE_MANAGER, CS_MANAGER | Refunds | Refund records and approvals; no payment provider, so no money moves | P | Y | Y | N | PARTIAL |
| EC-24 | E-Commerce Admin | CS_MANAGER | Customers | User directory only; no tenant customer list | P | Y | P | N | PARTIAL |
| EC-25 | E-Commerce Admin | CS_MANAGER | Customer details | CRM contact 360 exists in the CRM app, not in e-commerce admin | P | Y | P | N | PARTIAL |
| EC-26 | E-Commerce Admin | ECOM_MANAGER | Reviews and ratings | Customers post reviews; no moderation screen | P | Y | N | N | PARTIAL |
| EC-27 | E-Commerce Admin | STORE_ADMIN | Shipping configuration | Delivery zones and pricing live in the Delivery app | P | Y | Y | N | PARTIAL |
| EC-28 | E-Commerce Admin | STORE_ADMIN | Delivery configuration | Delivery rules (draft, validate, publish, rollback) in the Delivery app | Y | Y | Y | unit | COMPLETED |
| EC-29 | E-Commerce Admin | FINANCE_MANAGER | Tax configuration | None | N | N | N | N | NOT STARTED |
| EC-30 | E-Commerce Admin | FINANCE_MANAGER | Payment configuration | None, and no payment service | N | N | N | N | NOT STARTED |
| EC-31 | E-Commerce Admin | STORE_ADMIN, MARKETING_MANAGER | Storefront configuration | Storefront studio: sites, builder, themes, versions, approvals, domains | Y | Y | Y | unit | COMPLETED |
| EC-32 | E-Commerce Admin | MARKETING_MANAGER | SEO settings | Page metadata in storefront documents; no product-level SEO | P | P | Y | N | PARTIAL |
| EC-33 | E-Commerce Admin | ECOM_MANAGER, STORE_VIEWER | Reports | Order report summary | Y | Y | Y | N | COMPLETED |
| EC-34 | E-Commerce Admin | ECOM_MANAGER, STORE_VIEWER | Sales analytics | Analytics service, tenant-scoped | Y | Y | Y | unit | COMPLETED |
| EC-35 | E-Commerce Admin | TENANT_OWNER, STORE_ADMIN | User and role management | Client team screen (`/profile/team`) on `/api/tenant/**`: members with search and pagination, add, suspend, several roles, direct grants, custom roles, delegations, audit, with confirmations. Tests: `RoleEngineV2PostgresTest` (9), `TenantTeamGatewayTest` (3), `teamModel.test.ts` (11); live `live-team.sh` 26/26; browser journey (add role, remove role, audit) | Y | Y | Y | browser | VERIFIED |
| EC-36 | E-Commerce Admin | TENANT_OWNER, AUDITOR | Audit history | Separate order, product, pricing and storefront audit logs; no unified screen | P | Y | Y | N | PARTIAL |
| SF-01 | Storefront | GUEST, CUSTOMER | Home | Shell home page and published storefront pages | Y | Y | Y | N | COMPLETED |
| SF-02 | Storefront | GUEST, CUSTOMER | Product listing | Public catalog | Y | Y | Y | N | COMPLETED |
| SF-03 | Storefront | GUEST, CUSTOMER | Category listing | Category filter | Y | Y | Y | N | COMPLETED |
| SF-04 | Storefront | GUEST, CUSTOMER | Product detail | Detail with reviews | Y | Y | Y | N | COMPLETED |
| SF-05 | Storefront | GUEST, CUSTOMER | Product comparison | None | N | N | N | N | NOT STARTED |
| SF-06 | Storefront | GUEST, CUSTOMER | Product search and results | Search query on the public catalog (B-49 fixed) | Y | Y | Y | N | COMPLETED |
| SF-07 | Storefront | GUEST, CUSTOMER | Cart | Cart service with token identity (B-48); live checkout script 16/16 | Y | Y | Y | unit | COMPLETED |
| SF-08 | Storefront | CUSTOMER | Checkout | Server-side pricing and stock reservation (C-01, C-02); no shipping or payment step | Y | Y | Y | N | PARTIAL |
| SF-09 | Storefront | CUSTOMER | Address book | Create, edit and delete addresses | Y | Y | Y | N | COMPLETED |
| SF-10 | Storefront | CUSTOMER | Shipping selection | Delivery pricing exists but is not offered at checkout | N | P | N | N | NOT STARTED |
| SF-11 | Storefront | CUSTOMER | Payment | No payment service or provider; `payment_status` stays UNKNOWN | N | P | N | N | NOT STARTED |
| SF-12 | Storefront | CUSTOMER | Order confirmation | Shown after order creation | Y | Y | Y | N | COMPLETED |
| SF-13 | Storefront | CUSTOMER | Order history | Fixed in round 3 (B-52); browser and live checks | Y | Y | Y | browser | VERIFIED |
| SF-14 | Storefront | CUSTOMER | Order details | Owner-checked (C-03) | Y | Y | Y | N | COMPLETED |
| SF-15 | Storefront | CUSTOMER | Order tracking | Order tracking events and delivery recipient tracking; no unified customer tracking page | P | Y | Y | N | PARTIAL |
| SF-16 | Storefront | CUSTOMER | Return request | Customer returns | Y | Y | Y | N | COMPLETED |
| SF-17 | Storefront | CUSTOMER | Refund tracking | Customer refund requests in accounting; not linked to return refunds | P | Y | Y | N | PARTIAL |
| SF-18 | Storefront | CUSTOMER | Wishlist | Add, remove and list | Y | Y | Y | N | COMPLETED |
| SF-19 | Storefront | CUSTOMER | Customer profile | Profile page | Y | Y | Y | N | COMPLETED |
| SF-20 | Storefront | CUSTOMER | Password management | Account security page: change password (checks the current one, policy, signs out other sessions) and self-service reset pages. Tests: Postgres suite, `passwordModel.test.ts`; live change and reset | Y | Y | Y | N | COMPLETED |
| SF-21 | Storefront | CUSTOMER | Reviews | Customers submit reviews; no "my reviews" view | Y | Y | Y | N | PARTIAL |
| SF-22 | Storefront | CUSTOMER | Notifications | Notification bell and service; no preferences | Y | Y | Y | N | PARTIAL |
| SF-23 | Storefront | CUSTOMER | Support tickets | Depends on Support Management (not built) | N | N | N | N | NOT STARTED |
| SF-24 | Storefront | CUSTOMER | Loyalty account | Loyalty dashboard (B-46, B-47 fixed) | Y | Y | Y | unit | COMPLETED |
| SF-25 | Storefront | WHOLESALE, BUSINESS | Wholesale and business pricing | No customer type or group pricing | N | N | N | N | NOT STARTED |
| INV-01 | Inventory | INV_ADMIN, INV_MANAGER | Inventory dashboard | Ops dashboard | Y | Y | Y | unit | COMPLETED |
| INV-02 | Inventory | INV_MANAGER, INV_VIEWER | Product inventory | Inventory catalog | Y | Y | Y | unit | COMPLETED |
| INV-03 | Inventory | INV_ADMIN, WAREHOUSE_MANAGER | Warehouse management | Create and list warehouses; no edit or deactivate | P | Y | Y | N | PARTIAL |
| INV-04 | Inventory | WAREHOUSE_MANAGER | Warehouse details | Locations per warehouse | Y | Y | Y | N | COMPLETED |
| INV-05 | Inventory | STOCK_CONTROLLER, INV_VIEWER | Stock balances | Warehouse stock | Y | Y | Y | N | COMPLETED |
| INV-06 | Inventory | STOCK_CONTROLLER, INV_AUDITOR | Stock movements | Movement history | Y | Y | Y | N | COMPLETED |
| INV-07 | Inventory | STOREKEEPER | Goods receipt / receiving | Receipts against purchase orders | Y | Y | Y | unit | COMPLETED |
| INV-08 | Inventory | STOCK_CONTROLLER | Stock adjustments | Request, approve, reject | Y | Y | Y | N | COMPLETED |
| INV-09 | Inventory | WAREHOUSE_MANAGER | Stock transfers | Transfer lifecycle | Y | Y | Y | N | COMPLETED |
| INV-10 | Inventory | INV_MANAGER | Stock reservation | Reservations inspector | Y | Y | Y | N | COMPLETED |
| INV-11 | Inventory | INV_MANAGER, INV_AUDITOR | Reservation history | Reservation reconciliation | Y | Y | Y | N | COMPLETED |
| INV-12 | Inventory | INV_MANAGER | Low-stock alerts | Reorder thresholds on the dashboard; no notification | Y | Y | Y | N | COMPLETED |
| INV-13 | Inventory | INV_MANAGER, PROCUREMENT_MANAGER | Reorder suggestions | Replenishment and forecast; draft purchase order | Y | Y | Y | N | COMPLETED |
| INV-14 | Inventory | PROCUREMENT_MANAGER | Suppliers | Supplier lifecycle, items, performance | Y | Y | Y | unit | COMPLETED |
| INV-15 | Inventory | PROCUREMENT_MANAGER | Purchase requisitions | None | N | N | N | N | NOT STARTED |
| INV-16 | Inventory | PROCUREMENT_MANAGER | Purchase orders | Create, issue, cancel | Y | Y | Y | N | COMPLETED |
| INV-17 | Inventory | PROCUREMENT_MANAGER | Purchase order approval | Approve action with permission | Y | Y | Y | N | COMPLETED |
| INV-18 | Inventory | PROCUREMENT_MANAGER | Supplier returns | Create, ship, credit | Y | Y | Y | N | COMPLETED |
| INV-19 | Inventory | STOCK_CONTROLLER, INV_AUDITOR | Stock counts / cycle counting | None | N | N | N | N | NOT STARTED |
| INV-20 | Inventory | INV_MANAGER, FINANCE_MANAGER | Inventory valuation | FIFO and weighted-average cost layers | Y | Y | Y | N | COMPLETED |
| INV-21 | Inventory | INV_MANAGER | Batch and lot tracking | None | N | N | N | N | NOT STARTED |
| INV-22 | Inventory | INV_MANAGER | Serial number tracking | None | N | N | N | N | NOT STARTED |
| INV-23 | Inventory | INV_MANAGER | Expiry tracking | None | N | N | N | N | NOT STARTED |
| INV-24 | Inventory | INV_MANAGER, INV_VIEWER | Inventory reports | Export and analytics; no report catalog | P | Y | Y | N | PARTIAL |
| INV-25 | Inventory | INV_AUDITOR | Audit history | WMS audit log | Y | Y | Y | N | COMPLETED |
| ACC-01 | Accounting | FIN_ADMIN, ACCOUNTANT | Chart of accounts | Accounts create and update | Y | Y | Y | unit | COMPLETED |
| ACC-02 | Accounting | FIN_ADMIN | Account groups | Parent accounts only | P | P | Y | N | PARTIAL |
| ACC-03 | Accounting | FIN_ADMIN, CFO | Financial years | Periods only; no fiscal year | N | N | N | N | NOT STARTED |
| ACC-04 | Accounting | FIN_ADMIN, CFO | Accounting periods | Create, close, reopen | Y | Y | Y | N | COMPLETED |
| ACC-05 | Accounting | FIN_ADMIN, TREASURY | Currencies | Single currency | N | N | N | N | NOT STARTED |
| ACC-06 | Accounting | TREASURY | Exchange rates | None | N | N | N | N | NOT STARTED |
| ACC-07 | Accounting | TAX_MANAGER | Tax codes | None (a tax summary report only) | N | N | N | N | NOT STARTED |
| ACC-08 | Accounting | FIN_ADMIN | Cost centers | None | N | N | N | N | NOT STARTED |
| ACC-09 | Accounting | FIN_ADMIN | Departments | None | N | N | N | N | NOT STARTED |
| ACC-10 | Accounting | FIN_ADMIN | Branches | None (no branch entity anywhere) | N | N | N | N | NOT STARTED |
| ACC-11 | Accounting | ACCOUNTANT, AUDITOR | Journal entries | Journals list, reversal | Y | Y | Y | N | COMPLETED |
| ACC-12 | Accounting | ACCOUNTANT | Journal entry creation | Drafts with balanced-lines validation | Y | Y | Y | unit | COMPLETED |
| ACC-13 | Accounting | FINANCE_MANAGER, CFO | Journal approval | Submit, approve, reject | Y | Y | Y | N | COMPLETED |
| ACC-14 | Accounting | ACCOUNTANT, AUDITOR | General ledger | Admin ledger | Y | Y | Y | N | COMPLETED |
| ACC-15 | Accounting | ACCOUNTANT, AUDITOR | Trial balance | None | N | N | N | N | NOT STARTED |
| ACC-16 | Accounting | ACCOUNTANT | Account reconciliation | Reconciliation decisions on ledger transactions | P | Y | Y | N | PARTIAL |
| ACC-17 | Accounting | FINANCE_MANAGER, CFO | Period closing | Close and reopen with audit | Y | Y | Y | N | COMPLETED |
| ACC-18 | Accounting | CFO, FINANCE_VIEWER | Financial statements | Profit and loss, balance sheet, cash flow | Y | Y | Y | N | COMPLETED |
| ACC-19 | Accounting | AR_OFFICER | Customer invoices | Invoices create, update, status | Y | Y | Y | N | COMPLETED |
| ACC-20 | Accounting | AR_OFFICER | Credit notes | None | N | N | N | N | NOT STARTED |
| ACC-21 | Accounting | AR_OFFICER | Receipts | Invoice payment status only; no receipt document | P | P | Y | N | PARTIAL |
| ACC-22 | Accounting | AR_OFFICER | Customer balances | Balance per customer | Y | Y | Y | N | COMPLETED |
| ACC-23 | Accounting | AR_OFFICER | Outstanding invoices | Status filter only | P | Y | Y | N | PARTIAL |
| ACC-24 | Accounting | AR_OFFICER, CFO | Receivable aging | None | N | N | N | N | NOT STARTED |
| ACC-25 | Accounting | AR_OFFICER | Customer statements | Customer self-service statement export only | P | Y | Y | N | PARTIAL |
| ACC-26 | Accounting | AP_OFFICER | Supplier invoices | Held in procurement-service with approval; not an accounting AP ledger | P | P | Y | N | PARTIAL |
| ACC-27 | Accounting | AP_OFFICER | Debit notes | None | N | N | N | N | NOT STARTED |
| ACC-28 | Accounting | AP_OFFICER | Payment vouchers | None | N | N | N | N | NOT STARTED |
| ACC-29 | Accounting | AP_OFFICER, TREASURY | Supplier payments | None | N | N | N | N | NOT STARTED |
| ACC-30 | Accounting | AP_OFFICER | Outstanding payables | None | N | N | N | N | NOT STARTED |
| ACC-31 | Accounting | AP_OFFICER | Supplier statements | None | N | N | N | N | NOT STARTED |
| ACC-32 | Accounting | AP_OFFICER, CFO | Payable aging | None | N | N | N | N | NOT STARTED |
| ACC-33 | Accounting | TREASURY | Bank and cash accounts | Cash accounts are ledger accounts only; no bank account entity | P | P | Y | N | PARTIAL |
| ACC-34 | Accounting | TREASURY | Bank transactions | None | N | N | N | N | NOT STARTED |
| ACC-35 | Accounting | TREASURY, ACCOUNTANT | Bank reconciliation | None | N | N | N | N | NOT STARTED |
| ACC-36 | Accounting | TREASURY, CFO | Cash flow | Cash flow report | Y | Y | Y | N | COMPLETED |
| ACC-37 | Accounting | ACCOUNTANT | Payment reconciliation | Order and refund reconciliation decisions; no provider settlement data | P | Y | Y | N | PARTIAL |
| ACC-38 | Accounting | CFO, FINANCE_VIEWER | Trial balance report | Same gap as ACC-15 | N | N | N | N | NOT STARTED |
| ACC-39 | Accounting | ACCOUNTANT, AUDITOR | General ledger report | Ledger export | P | Y | Y | N | PARTIAL |
| ACC-40 | Accounting | TAX_MANAGER | Tax reports | Tax summary without tax codes | P | P | Y | N | PARTIAL |
| ACC-41 | Accounting | AR_OFFICER, CFO | Receivable and payable reports | None | N | N | N | N | NOT STARTED |
| ACC-42 | Accounting | CFO, FINANCE_VIEWER | Financial dashboard | Accounting dashboard and trends | Y | Y | Y | N | COMPLETED |
| ACC-43 | Accounting | ALL FINANCE | Double entry and posted-entry immutability | Balanced drafts enforced; posted lines protected (B-29) | Y | Y | Y | N | COMPLETED |
| DLV-01 | Delivery | DLV_ADMIN, DLV_MANAGER | Delivery dashboard | Operations overview | Y | Y | Y | unit | COMPLETED |
| DLV-02 | Delivery | DLV_MANAGER, DISPATCHER | Delivery orders | Delivery list, create from order | Y | Y | Y | N | COMPLETED |
| DLV-03 | Delivery | DISPATCH_MANAGER, DISPATCHER | Dispatch board | Dispatch queue and candidates | Y | Y | Y | N | COMPLETED |
| DLV-04 | Delivery | DISPATCHER | Shipment creation | Create delivery; order fulfilment auto-creates through `/internal/delivery` | Y | Y | Y | N | COMPLETED |
| DLV-05 | Delivery | DISPATCHER | Delivery assignment | Assign, partner accepts or declines | Y | Y | Y | N | COMPLETED |
| DLV-06 | Delivery | DLV_MANAGER | Driver management | Partner review and availability | Y | Y | Y | N | COMPLETED |
| DLV-07 | Delivery | DLV_MANAGER | Delivery partner management | Partner capabilities and zones | Y | Y | Y | N | COMPLETED |
| DLV-08 | Delivery | DLV_MANAGER | Vehicle management | None | N | N | N | N | NOT STARTED |
| DLV-09 | Delivery | DLV_ADMIN | Delivery zones | Create and edit zones | Y | Y | Y | N | COMPLETED |
| DLV-10 | Delivery | DISPATCH_MANAGER | Route planning | Trips and stops; no route optimisation | Y | Y | Y | N | COMPLETED |
| DLV-11 | Delivery | DISPATCHER | Pickup scheduling | Readiness flag only; no pickup window | P | P | Y | N | PARTIAL |
| DLV-12 | Delivery | DISPATCHER, CUSTOMER | Delivery tracking | Location updates and recipient tracking | Y | Y | Y | N | COMPLETED |
| DLV-13 | Delivery | DRIVER | Delivery status | Transitions with lifecycle rules | Y | Y | Y | unit | COMPLETED |
| DLV-14 | Delivery | DRIVER | Proof of delivery | Proof and OTP codes | Y | Y | Y | N | COMPLETED |
| DLV-15 | Delivery | DRIVER, DISPATCHER | Failed delivery | Incidents and resolution | Y | Y | Y | N | COMPLETED |
| DLV-16 | Delivery | DISPATCHER | Reattempt delivery | Transition back to dispatch; no reattempt policy or limit | P | Y | Y | N | PARTIAL |
| DLV-17 | Delivery | DISPATCHER | Return pickup | Reverse pickups are in order-service, not delivery | P | P | Y | N | PARTIAL |
| DLV-18 | Delivery | DLV_ADMIN | Delivery charges | Partner pricing policy | Y | Y | Y | N | COMPLETED |
| DLV-19 | Delivery | DLV_MANAGER | Delivery performance and reports | Operations reports, earnings | Y | Y | Y | N | COMPLETED |
| DLV-20 | Delivery | DLV_AUDITOR | Delivery audit history | Audit trail | Y | Y | Y | N | COMPLETED |
| DLV-21 | Delivery | DRIVER | Driver sees only own deliveries | `/my/deliveries`, tested | Y | Y | Y | N | COMPLETED |
| VEN-01 | Vendor / Restaurant | VENDOR_OWNER | Vendor dashboard | No vendor service | N | N | N | N | NOT STARTED |
| VEN-02 | Vendor / Restaurant | VENDOR_OWNER, VENDOR_ADMIN | Vendor profile | None | N | N | N | N | NOT STARTED |
| VEN-03 | Vendor / Restaurant | VENDOR_OWNER | Vendor registration | None | N | N | N | N | NOT STARTED |
| VEN-04 | Vendor / Restaurant | SUPER_ADMIN, STORE_ADMIN | Vendor approval | None | N | N | N | N | NOT STARTED |
| VEN-05 | Vendor / Restaurant | VENDOR_ADMIN | Branch management | None | N | N | N | N | NOT STARTED |
| VEN-06 | Vendor / Restaurant | BRANCH_MANAGER | Business hours | None | N | N | N | N | NOT STARTED |
| VEN-07 | Vendor / Restaurant | BRANCH_MANAGER | Service areas | None (delivery zones exist but are not per vendor) | N | N | N | N | NOT STARTED |
| VEN-08 | Vendor / Restaurant | VENDOR_ADMIN | Vendor users and roles | None | N | N | N | N | NOT STARTED |
| VEN-09 | Vendor / Restaurant | MENU_MANAGER | Menu categories | None | N | N | N | N | NOT STARTED |
| VEN-10 | Vendor / Restaurant | MENU_MANAGER | Menu items | None | N | N | N | N | NOT STARTED |
| VEN-11 | Vendor / Restaurant | MENU_MANAGER | Variants, add-ons, modifiers | None | N | N | N | N | NOT STARTED |
| VEN-12 | Vendor / Restaurant | KITCHEN_MANAGER | Ingredients and recipes | None | N | N | N | N | NOT STARTED |
| VEN-13 | Vendor / Restaurant | MENU_MANAGER | Availability | None | N | N | N | N | NOT STARTED |
| VEN-14 | Vendor / Restaurant | MENU_MANAGER | Menu pricing and promotions | None | N | N | N | N | NOT STARTED |
| VEN-15 | Vendor / Restaurant | RESTAURANT_MANAGER | Incoming orders | Order-service has restaurant transitions (`/orders/{id}/restaurant/{action}`); no vendor screen | P | P | P | N | PARTIAL |
| VEN-16 | Vendor / Restaurant | RESTAURANT_MANAGER | Order acceptance and rejection | Backend transitions only | P | P | P | N | PARTIAL |
| VEN-17 | Vendor / Restaurant | KITCHEN_MANAGER, KITCHEN_STAFF | Preparation queue and kitchen display | None | N | N | N | N | NOT STARTED |
| VEN-18 | Vendor / Restaurant | KITCHEN_STAFF | Preparation status, ready for pickup, handover | Backend transitions only | P | P | P | N | PARTIAL |
| VEN-19 | Vendor / Restaurant | RESTAURANT_MANAGER | Cancelled orders and order history | None per vendor | N | N | N | N | NOT STARTED |
| VEN-20 | Vendor / Restaurant | VENDOR_ACCOUNTANT | Commission configuration | None | N | N | N | N | NOT STARTED |
| VEN-21 | Vendor / Restaurant | VENDOR_ACCOUNTANT | Settlements and payouts | None | N | N | N | N | NOT STARTED |
| VEN-22 | Vendor / Restaurant | VENDOR_ACCOUNTANT | Statements and financial reports | None | N | N | N | N | NOT STARTED |
| CRM-01 | CRM | CRM_ADMIN, CRM_MANAGER | CRM dashboard | Dashboard | Y | Y | Y | unit | COMPLETED |
| CRM-02 | CRM | SALES_EXEC, SUPPORT_AGENT | Customer list | Contacts with saved views, bulk update | Y | Y | Y | unit | COMPLETED |
| CRM-03 | CRM | SALES_EXEC | Customer profile | Contact 360, custom fields, audit | Y | Y | Y | N | COMPLETED |
| CRM-04 | CRM | MARKETING_MANAGER, CRM_ANALYST | Customer segmentation | Segments with criteria and members | Y | Y | Y | N | COMPLETED |
| CRM-05 | CRM | CRM_MANAGER | Customer groups | Companies; no group model separate from segments | P | Y | Y | N | PARTIAL |
| CRM-06 | CRM | SALES_EXEC | Customer timeline | Activities in the 360 view | Y | Y | Y | N | COMPLETED |
| CRM-07 | CRM | SALES_MANAGER, SALES_EXEC | Leads | Lifecycle stage on contacts; no lead entity or conversion | P | Y | Y | N | PARTIAL |
| CRM-08 | CRM | SALES_MANAGER | Lead assignment | Owner field; no assignment rules | P | Y | Y | N | PARTIAL |
| CRM-09 | CRM | SALES_EXEC | Opportunities | Create, update, stage change | Y | Y | Y | N | COMPLETED |
| CRM-10 | CRM | SALES_MANAGER | Sales pipeline | Pipeline stages | Y | Y | Y | N | COMPLETED |
| CRM-11 | CRM | SALES_EXEC | Follow-up tasks | Tasks with completion | Y | Y | Y | N | COMPLETED |
| CRM-12 | CRM | SALES_EXEC, SUPPORT_AGENT | Notes and activities | Activities | Y | Y | Y | N | COMPLETED |
| CRM-13 | CRM | CAMPAIGN_MANAGER | Campaigns | None in CRM (pricing campaigns are discounts) | N | N | N | N | NOT STARTED |
| CRM-14 | CRM | CAMPAIGN_MANAGER | Email campaigns | No email provider | N | N | N | N | NOT STARTED |
| CRM-15 | CRM | CAMPAIGN_MANAGER | SMS campaigns | No SMS provider | N | N | N | N | NOT STARTED |
| CRM-16 | CRM | CRM_MANAGER | Loyalty programs | Loyalty tiers in the Loyalty app | Y | Y | Y | unit | COMPLETED |
| CRM-17 | CRM | CRM_MANAGER | Customer feedback | None | N | N | N | N | NOT STARTED |
| CRM-18 | CRM | CRM_ANALYST | Customer analytics | Customer analytics in the analytics service, not CRM | P | Y | Y | N | PARTIAL |
| CRM-19 | CRM | CRM_MANAGER | Communication preferences and consent | None | N | N | N | N | NOT STARTED |
| CRM-20 | CRM | CRM_ANALYST | CRM reports | Dashboard figures only | P | Y | Y | N | PARTIAL |
| QA-01 | Quality Center | QA_ADMIN, QA_MANAGER | QA dashboard | Overview | Y | Y | Y | unit | COMPLETED |
| QA-02 | Quality Center | QA_MANAGER | Projects | Features act as the catalog; no project entity | P | P | Y | N | PARTIAL |
| QA-03 | Quality Center | TEST_LEAD | Test plans | None | N | N | N | N | NOT STARTED |
| QA-04 | Quality Center | TEST_LEAD | Test suites | Read-only suites discovered from the repository | P | Y | Y | N | PARTIAL |
| QA-05 | Quality Center | TEST_ENGINEER | Test cases | Discovered from code; no manual test cases | P | Y | Y | N | PARTIAL |
| QA-06 | Quality Center | TEST_ENGINEER | Test case editor | None | N | N | N | N | NOT STARTED |
| QA-07 | Quality Center | TEST_ENGINEER, AUTOMATION_ENG | Test execution | Start, cancel and rerun runs; the runner needs the repository mounted (A-09) | Y | Y | Y | N | COMPLETED |
| QA-08 | Quality Center | QA_VIEWER | Test run history | Runs, results, logs | Y | Y | Y | N | COMPLETED |
| QA-09 | Quality Center | TEST_ENGINEER, DEVELOPER | Defect management | None (error-stack groups runtime errors, not defects) | N | N | N | N | NOT STARTED |
| QA-10 | Quality Center | DEFECT_REVIEWER | Defect details and assignment | None | N | N | N | N | NOT STARTED |
| QA-11 | Quality Center | TEST_ENGINEER | Test evidence | Run artifacts and logs; no manual evidence upload | P | Y | Y | N | PARTIAL |
| QA-12 | Quality Center | TEST_LEAD | Regression suites | Suites only | P | Y | Y | N | PARTIAL |
| QA-13 | Quality Center | AUTOMATION_ENG | Automation test results | Results per run | Y | Y | Y | N | COMPLETED |
| QA-14 | Quality Center | QA_MANAGER | Release quality gates | None | N | N | N | N | NOT STARTED |
| QA-15 | Quality Center | QA_MANAGER, QA_VIEWER | Test reports | Run report | Y | Y | Y | N | COMPLETED |
| QA-16 | Quality Center | QA_MANAGER | Coverage reports | Coverage explorer | Y | Y | Y | N | COMPLETED |
| SUP-01 | Support | SUPPORT_ADMIN, SUPPORT_MANAGER | Support dashboard | No support service | N | N | N | N | NOT STARTED |
| SUP-02 | Support | SUPPORT_AGENT | Ticket list | None | N | N | N | N | NOT STARTED |
| SUP-03 | Support | CUSTOMER, SUPPORT_AGENT | Ticket creation | None | N | N | N | N | NOT STARTED |
| SUP-04 | Support | SUPPORT_AGENT | Ticket details | None | N | N | N | N | NOT STARTED |
| SUP-05 | Support | SUPPORT_TEAM_LEAD | Ticket assignment | None | N | N | N | N | NOT STARTED |
| SUP-06 | Support | SUPPORT_MANAGER | Priority and category management | None | N | N | N | N | NOT STARTED |
| SUP-07 | Support | SUPPORT_ADMIN | SLA configuration | None | N | N | N | N | NOT STARTED |
| SUP-08 | Support | SUPPORT_ADMIN | Escalation rules | None | N | N | N | N | NOT STARTED |
| SUP-09 | Support | SUPPORT_AGENT | Internal notes | None | N | N | N | N | NOT STARTED |
| SUP-10 | Support | SUPPORT_AGENT, CUSTOMER | Customer communication | None | N | N | N | N | NOT STARTED |
| SUP-11 | Support | SUPPORT_AGENT, CUSTOMER | Attachments | No file storage service | N | N | N | N | NOT STARTED |
| SUP-12 | Support | SUPPORT_AUDITOR | Ticket history | None | N | N | N | N | NOT STARTED |
| SUP-13 | Support | SUPPORT_MANAGER | Knowledge base | None | N | N | N | N | NOT STARTED |
| SUP-14 | Support | SUPPORT_MANAGER, CUSTOMER | FAQs | None | N | N | N | N | NOT STARTED |
| SUP-15 | Support | SUPPORT_MANAGER | Customer feedback | None | N | N | N | N | NOT STARTED |
| SUP-16 | Support | SUPPORT_MANAGER | Support analytics and reports | None | N | N | N | N | NOT STARTED |
| SH-01 | Shared | ALL | Authentication | Platform login, application login, logout with revocation, lockout | Y | Y | Y | browser | VERIFIED |
| SH-02 | Shared | ALL | User profile | Profile page | Y | Y | Y | N | COMPLETED |
| SH-03 | Shared | ALL TENANT ROLES | Tenant switching | Application sign-in lists the person's workspaces that grant the application and opens one with the current sign-in (token exchange, no password); "Switch workspace" signs the application session out on the server. Tests: `AccountSecurityPostgresTest` (workspaces, exchange), `applicationSession.test.ts`; live; browser (switch and reopen) | Y | Y | Y | browser | VERIFIED |
| SH-04 | Shared | ALL | Application launcher | "My applications" (`/apps`) lists platform applications and every workspace's granted applications from live grants; in-shell applications open with one click; Super Admin and Delivery open their own sites in a new tab and ask for their own sign-in. Tests: `launcherModel.test.ts`; browser for in-shell opening. The two separate sites were not opened in the browser | Y | Y | Y | browser | COMPLETED |
| SH-05 | Shared | ALL | Notification center | Bell with unread, mark read; no full notification page | Y | Y | Y | N | PARTIAL |
| SH-06 | Shared | ALL | Global search | None | N | N | N | N | NOT STARTED |
| SH-07 | Shared | TENANT_OWNER, SUPER_ADMIN | Role and permission management | V20: several roles per application, client-owned custom roles, permission groups, direct grants, expiring delegation, escalation guard, immediate revocation. Branch, warehouse and store scopes are modelled but no service enforces them yet, so scoped roles are refused (step 8) | Y | Y | Y | N | PARTIAL |
| SH-08 | Shared | ALL | File and media management | No file storage service | N | N | N | N | NOT STARTED |
| SH-09 | Shared | AUDITOR | Audit history | Per-service audit logs; no shared viewer | P | Y | Y | N | PARTIAL |
| SH-10 | Shared | ALL | Localization, currency, time zone settings | None | N | N | N | N | NOT STARTED |
| SH-11 | Shared | ALL | Theme and branding | Light and dark theme in the shell | P | N | Y | N | PARTIAL |
| SH-12 | Shared | ALL | Account security and session management | Sessions recorded for every issued token (V21); list with device, IP and expiry; end one or all other sessions (revocation enforced by the gateway); sign-in activity; lockout. Tests: Postgres suite, gateway `AccountSecurityGatewayTest`; live; browser (list, sign out all others) | Y | Y | Y | browser | VERIFIED |
| SH-13 | Shared | ALL | Password reset | One-time, hashed, 30-minute tokens carried in the URL fragment; self-service by email over SMTP when `spring.mail.host` is configured (otherwise an honest "not set up" state); assisted reset link from User administration (`users:manage`, hierarchy-checked); reset signs out every session and clears lockout. SMTP delivery tested with a mocked sender, not a real mail server | Y | Y | Y | N | COMPLETED |
| SH-14 | Shared | ALL | API error handling | Shared error responses; 400 for malformed input (B-44) | Y | Y | Y | N | COMPLETED |
| SH-15 | Shared | PLATFORM_OPS | Application health status | Actuator only | P | N | N | N | PARTIAL |
