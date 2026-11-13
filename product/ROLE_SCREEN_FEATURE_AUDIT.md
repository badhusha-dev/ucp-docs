# Role, Screen and Feature Audit (Phase 0)

**Baseline:** commit `962a7e58` on `feature/ucp-completion-modernization`, 2026-11-02.
**Method:** read from source: frontend routes and screens, `@RestController` mappings, Flyway `CREATE TABLE` statements and role seeds. Earlier documents were not trusted; where they disagree with code, this audit follows the code. One statement was corrected during the audit after reading the code: tenant suspension *is* checked at application sign-in.
**Tracking:**
- Every screen is a row in [SCREEN_FEATURE_COMPLETION_MATRIX.md](SCREEN_FEATURE_COMPLETION_MATRIX.md) (284 rows).
- Figures come from `scripts/product/completion.mjs` into [IMPLEMENTATION_PROGRESS.md](IMPLEMENTATION_PROGRESS.md).

## 1. Headline

| | Value |
|---|---|
| Required screens and features tracked | 284 |
| COMPLETED or VERIFIED | 100 (35.2%) |
| PARTIAL | 67 |
| NOT STARTED | 117 |
| Applications with no implementation | Vendor / Restaurant, Support Management |
| Roles required by the brief | 80: 17 exist, 25 covered by a broader role, 38 missing ([ROLE_CATALOG.md](ROLE_CATALOG.md)) |
| Mock data, fake success messages, dummy buttons | None found (search for mock, dummy, sample, placeholder, TODO, `Math.random` in all 16 frontends) |

The existing applications are real: every screen found calls a backend that persists to Postgres, and QA rounds 1–3 fixed and verified their security. The gap is **breadth**:
- **Missing domains:** subscriptions and billing, payment, vendor and restaurant, support, file storage, tax, banking, accounts payable.
- **Role engine limits:**
  - one role per user per application;
  - global roles;
  - no scopes, delegation or expiry.

## 2. Existing applications

| # | Application | Frontend | Backend | Registry status | Matrix completed |
|---|---|---|---|---|---|
| 1 | Super Admin | `mfe-super-admin` (standalone, not in the shell), `mfe-user /admin` | user-service | ACTIVE | 8.9% |
| 2 | E-Commerce Admin | `mfe-product` (catalog, pricing, storefront studio), `mfe-order` (control tower, returns, analytics), `mfe-cart` (abandoned carts), `mfe-analytics` | product, order, analytics, cart | ACTIVE | 41.7% |
| 3 | Storefront | `shell-app`, `mfe-product`, `mfe-cart`, `mfe-order`, `mfe-user`, `mfe-loyalty` | product, cart, order, user, loyalty, notification | PLANNED (works through platform login) | 56.0% |
| 4 | Inventory | `mfe-inventory`, `mfe-stock`, `mfe-procurement` | inventory, stock, procurement | ACTIVE | 72.0% |
| 5 | Accounting | `mfe-accounting`, `mfe-order /ledger` | accounting | ACTIVE | 30.2% |
| 6 | Delivery | `mfe-delivery` (standalone, not in the shell) | delivery | ACTIVE (V7) | 81.0% |
| 7 | Vendor / Restaurant | none | none (order-service has restaurant order transitions) | PLANNED | 0% |
| 8 | CRM | `mfe-crm`, `mfe-loyalty /admin` | crm, loyalty | PLANNED (its APIs accept E-commerce Admin tokens) | 50.0% |
| 9 | Quality Center | `shell-app /admin/quality-center`, `mfe-error-stack` | qa, error-stack | ACTIVE | 37.5% |
| 10 | Support | none | none | PLANNED (`CUSTOMER_SUPPORT`) | 0% |

Backend: 16 business services with 570 mapped endpoints in 48 controllers (§6). Database: 202 tables across 15 service databases.

## 3. Roles

See [ROLE_CATALOG.md](ROLE_CATALOG.md) for the full list. Summary:

- **Existing:** 33 seeded roles, mostly operational (order, returns, pricing, storefront, inventory, procurement, delivery, accounting).
- **Missing whole families:**
  - platform operations, finance, support and auditor roles;
  - tenant owner;
  - catalog and product manager;
  - CFO, AP, AR, tax and treasury;
  - every vendor and restaurant role;
  - sales and campaign roles;
  - QA manager, developer and defect reviewer;
  - every support role;
  - wholesale and business customers.

### Role-engine gaps (brief §4)

> **Step 1 update (2026-11-09, V20):** every row below except branch, warehouse and store scope enforcement is now implemented, and covered by `RoleEngineV2PostgresTest` and `TenantTeamGatewayTest`. See [ROLE_PERMISSION_MATRIX.md](ROLE_PERMISSION_MATRIX.md) §3. The table is kept as the Phase 0 record.

| Brief requirement | Today | Gap |
|---|---|---|
| Identity, membership, entitlement, grant, role and permission kept separate | Yes: six separate tables (user-service V4, V6) | — |
| Tenant A roles not carried to tenant B | Yes: assignments are keyed by tenant, and services scope by the token's `tenant_id` (B-02, tested per service) | — |
| Several roles per user in one application | No: primary key `(tenant_id, user_id, application_code)` | Needs a new assignment table (forward migration) |
| Custom roles per tenant | Roles can be created but are **global** (`access_roles` has no `tenant_id`), so one tenant's custom role is visible to all | Tenant-owned roles |
| Permission groups | `module_name` on permissions only | Groups for the role editor |
| User-specific grants | None | Direct permission grants |
| Branch, warehouse and store scope | None (no branch entity exists anywhere) | Scope model, carried in the token and enforced by services |
| Read-only access | Analyst and auditor roles exist per module | Consistent viewer role per application |
| Approval permissions | Exist in places (`accounting:approve`, pricing and refund approvers) | Consistent naming |
| Delegated access with scope and expiry | None | Delegation table and token claims |
| Tenant owner manages own team | No: only platform operators grant tenant access (`/api/platform/tenants/**`) | Tenant-level user administration |

## 4. Screens

### Existing (by frontend)

| Frontend | Screens |
|---|---|
| shell-app | Home, login, Quality Center (overview, features, tests, runs, RBAC matrix, coverage), notification bell, navigation |
| mfe-super-admin | Tenant list and creation; application entitlements per tenant; members; grant and revoke application access with a role |
| mfe-user | Profile, address book, user administration (role, status), role and permission administration (`/api/access`) |
| mfe-product | Storefront catalog, product detail, reviews, wishlist; catalog admin; pricing and promotions; storefront studio |
| mfe-cart | Cart, cart drawer, abandoned-carts admin |
| mfe-order | Checkout, customer orders and returns, control tower (dashboard, orders, details, exceptions, bulk jobs, saved views, settings), returns console and policies, admin analytics, ledgers |
| mfe-loyalty | Customer loyalty dashboard, loyalty admin (stats, accounts, tiers) |
| mfe-inventory | Inventory sign-in, catalog, stock admin, reservations, operations workspace (warehouses, stock, transfers, adjustments, valuation, forecast, audit) |
| mfe-stock | Stock admin, reservations |
| mfe-procurement | Overview, suppliers, purchase orders, supplier invoices, replenishment |
| mfe-accounting | Customer ledger, admin ledger, accounts, journals and drafts, invoices, reports, periods |
| mfe-delivery | Overview, deliveries, dispatch board, partners, trips, incidents, earnings, reports, zones, pricing, configuration, audit, partner profile |
| mfe-crm | Overview, contacts, companies, opportunities, pipeline, tasks, segments, import and export |
| mfe-analytics | Admin analytics dashboard |
| mfe-error-stack | Error overview, groups, details |

### Missing

117 rows are NOT STARTED and 67 are PARTIAL; all are listed in the matrix. The largest groups:

| Area | Missing screens |
|---|---|
| Super Admin | Platform dashboard, tenant edit, suspension and configuration, application registry, all of subscriptions and platform finance, platform configuration, background jobs, login activity |
| E-Commerce Admin | Attributes, brands, catalog import and export, tax and payment configuration; category and variant administration; tenant team management |
| Storefront | Comparison, shipping selection, payment, password management, support tickets, wholesale and business pricing |
| Inventory | Purchase requisitions, stock counts and cycle counting, batch, lot, serial and expiry tracking |
| Accounting | Financial years, currencies and exchange rates, tax codes, cost centers, departments, branches, trial balance, credit and debit notes, all of accounts payable, aging, bank accounts and bank reconciliation |
| Delivery | Vehicles; pickup windows; reattempt policy |
| Vendor / Restaurant | Everything |
| CRM | Campaigns (email, SMS), feedback, communication preferences and consent |
| Quality Center | Test plans, manual test cases and editor, defects, release quality gates |
| Support | Everything |
| Shared | Password reset, tenant switcher, global search, file and media service, localization, session management |

### Structural issues found

1. **Two applications sit outside the shell:** Super Admin and Delivery are standalone sites, so there is no single launcher (SH-04).
2. **Duplicate ownership:**
   - Stock is owned by both `stock-service` and `inventory-service`.
   - Procurement has two implementations in one service (`ProcurementController` and `UcpController`, with two table sets).
   - Loyalty exists in both `user-service` (`customer_loyalty`) and `loyalty-service`.
   - Accounting and analytics keep copies of order tables.

   The brief forbids duplicated business logic; consolidating these is a data migration and needs a decision (§9).
3. **Screen standards:** many working screens lack search, pagination or sorting (for example catalog admin shows the first 100 products, and the tenant list has no search). These rows are PARTIAL even though the feature works.

## 5. Incomplete features (exists but not finished)

| Feature | Missing piece |
|---|---|
| Checkout | No shipping choice and no payment; orders are created with `payment_status` UNKNOWN |
| Refunds | Refund records and approvals exist, but no money moves without a payment service |
| Tenant lifecycle | Suspension is honoured at sign-in, but there is no way to suspend, and issued tokens stay valid until expiry |
| Application registry | Table only; no API or screen |
| Supplier invoices | Approved in procurement; no accounts payable ledger or payment |
| Restaurant orders | Order-service has restaurant transitions, but there are no vendors, menus or kitchen screens |
| Quality Center | The runner needs the repository mounted (A-09); no manual test management |
| Notifications | Message text is hardcoded; no templates, preferences, email or SMS provider |

## 6. Backend APIs

**Existing:** base paths, with mapped endpoint counts.

| Service | Base paths (endpoints) |
|---|---|
| user | `/api/users` (17), `/api/access` (6), `/api/platform/tenants` (9) |
| product | `/api/products` (20), `/admin` (7), `/admin/pricing` (35), `/admin/storefront` (38), `/public/storefront` (8), `/admin/analytics` (1) |
| order | `/api/orders` (22), `/admin` (40), `/admin/rma` (45) |
| cart | `/api/cart` (8), `/admin` (3) |
| inventory | `/api/inventory` (5), `/admin` (11), `/ops` (21) |
| stock | `/api/stock` (8), `/admin` (8) |
| procurement | `/api/procurement` (24), `/ucp` (21) |
| accounting | `/api/accounting` (11), `/admin` (38) |
| delivery | `/api/delivery` (44) |
| crm | `/api/crm` (45) |
| loyalty | `/api/loyalty` (5), `/admin` (4) |
| analytics | `/api/orders/admin/analytics` (7) |
| notification | `/api/notifications` (8) |
| qa | `/api/admin/qa` (16) |
| error-stack | `/api/errors` (2), `/admin` (6) |

Internal APIs (`/internal/**`) exist for order, pricing, inventory, stock, delivery and accounting.

**Missing APIs**, grouped by owner per brief §9:

| Owner | Missing |
|---|---|
| Identity (user-service) | Tenant update and suspend; application registry CRUD; multi-role assignment, tenant custom roles, scopes, delegation; tenant team administration; password change and reset; session list and revoke-all; login history; "my tenants" for the switcher |
| Billing (new) | Plans, features, packages, subscriptions, renewals, suspension, invoices, payments, refunds, revenue |
| Payment (new) | Payment intents, capture, refund, webhooks, reconciliation export. Owns payment transactions |
| Product | Categories, attributes, brands, variants administration; import and export; review moderation; customer-group price lists |
| Order | Shipping option at checkout; payment hand-off |
| Inventory | Requisitions, stock counts, lots, serials, expiry; warehouse update |
| Accounting | Fiscal years, currencies, exchange rates, tax codes, cost centers, departments, branches; trial balance; credit and debit notes; AP ledger and supplier payments; aging; bank accounts, bank transactions and reconciliation |
| Delivery | Vehicles; pickup windows; reattempt policy |
| Vendor (new) | Vendors, branches, hours, service areas, menus, modifiers, recipes, availability, kitchen queue, commissions, settlements |
| CRM | Campaigns, consent and preferences, feedback, leads |
| QA | Projects, plans, manual cases, defects, release gates |
| Support (new) | Tickets, assignment, SLA, escalation, notes, messages, knowledge base, FAQs, feedback, reports |
| Platform (new or existing) | File storage, settings, feature flags, notification templates, background job status |

## 7. Database

**Existing:** 202 tables across 15 databases (per-service list in the appendix).

**Missing structures:**
- **Billing:** plans, features, packages, subscriptions, invoices, payments.
- **Payment:** transactions, refunds, webhook inbox.
- **Vendor:** vendors, branches, hours, areas, menu tables, recipes, commissions, settlements.
- **Support:** tickets, messages, SLAs, knowledge base.
- **Identity:** multi-role assignments, tenant roles, scopes, delegations, login history, password reset tokens, tenant settings.
- **Accounting:** fiscal years, currencies, rates, tax codes, cost centers, departments, branches, credit and debit notes, AP documents, bank accounts and transactions.
- **Inventory:** requisitions, counts, lots, serials.
- **Catalog:** attributes, brands.
- **CRM:** campaigns, consent, feedback.
- **QA:** projects, plans, cases, defects, gates.
- **Delivery:** vehicles.
- **Shared:** files.

All are new tables through forward-only migrations; no existing migration needs editing.

## 8. Permissions

**Existing:** 113 permission keys and 344 role grants (user-service V4–V19, read from a database migrated to V19). **Missing families:**
- `platform:*` (operations, finance, support, audit);
- `billing:*`, `payment:*`;
- `vendor:*`, `menu:*`, `kitchen:*`;
- `support:*`;
- `catalog:attributes`, `catalog:import`;
- `tax:*`, `ap:*`, `ar:*`, `bank:*`;
- `crm:campaign`, `qa:defect`, `qa:gate`;
- `tenant:users` (tenant-level team administration).

These are listed role by role in [ROLE_PERMISSION_MATRIX.md](ROLE_PERMISSION_MATRIX.md).

## 9. Business workflows

The 17 workflows in brief §10 are graded in [BUSINESS_WORKFLOW_CATALOG.md](BUSINESS_WORKFLOW_CATALOG.md): **5 work end to end**, 7 are partial and 5 are blocked:
- **Work end to end** (backend tests or live scripts; no automated browser test yet): product publication, procurement and goods receipt, stock reservation, retail cart and checkout, delivery.
- **Partial:** onboarding, invitation, return and refund (no money moves), segmentation without campaigns, invoices without supplier payment, posting without bank reconciliation, test execution without release approval.
- **Blocked** (a whole domain is missing): subscription purchase, payment, restaurant, support tickets, subscription renewal.

## 10. Decisions

The brief says to ask only when a decision affects business behaviour, security, data migration or financial correctness. These four do. The owner decided them on 2026-11-02:

| # | Decision | Chosen |
|---|---|---|
| 1 | Payment | New payment-service with a provider interface, wired to **Stripe test mode**. Test keys come from `.env`, never committed; without keys the service reports "not configured". Production keys stay the owner's decision |
| 2 | Missing domains | **One new service per domain:** billing-service, payment-service, vendor-service, support-service, each with its own database, gateway routes and frontend |
| 3 | Duplicates | **Keep both, mark one legacy:** inventory-service owns stock, UCP procurement (`/api/procurement/ucp`) owns procurement, and loyalty-service owns loyalty. New features go only to the owner. Merging data is a later, separately approved and backed-up migration |
| 4 | Role model | **Additive:** new `tenant_user_roles` table (role list, scope, expiry, delegator) backfilled from `tenant_role_assignments`. The old table is kept and no longer written |

The options as they were put to the owner:

1. **Payment provider.** No payment code exists. Options:
   - integrate one provider's test mode (for example Stripe) behind a provider interface;
   - or build the payment service with a provider interface and an honest "not configured" state, with no live provider.

   This affects financial correctness and secrets handling.
2. **New services.** The brief's ownership rules point to new services:
   - billing / subscriptions;
   - payment;
   - vendor / restaurant;
   - support;
   - and, optionally, file storage.

   Each adds a database, a Compose service, gateway routes and an MFE. The alternative is folding billing into user-service and support into CRM. This is an architectural choice.
3. **Consolidating duplicates.** Stock in two services, two procurement implementations, and loyalty in two services. Merging means migrating data between service databases. The alternative is to keep both and mark one as legacy.
4. **Role model migration.** To support several roles per application, tenant-owned roles, scopes and delegation, I propose a new `tenant_user_roles` table (with scope and expiry) backfilled from `tenant_role_assignments`. The old table is kept, but no longer written. Token claims would then carry a list of roles and scopes; all eight tenant filters would read the list.

## 11. Prioritized implementation plan

Order follows dependencies: nothing is built on a foundation that will change under it. Each step ends with tests, matrix updates and a commit.

| Step | Phase | Work | Unblocks | Matrix rows |
|---|---|---|---|---|
| 1 | Foundation | Role engine v2: multi-role, tenant roles, permission groups, user grants, scopes, delegation with expiry; tenant team administration; new platform and application roles and permissions | Every role-specific screen | SH-07, EC-35, SA-24, SA-25 |
| 2 | Foundation | Password change and reset, session list and revoke-all, login history, tenant switcher ("my workspaces"), single application launcher that includes Super Admin and Delivery | Shared screens | SH-03, SH-04, SH-12, SH-13, SF-20, SA-27 |
| 3 | Foundation | Shared screen kit: data table with server search, sort and pagination; confirm dialog; empty, error and success states. Then apply it to existing PARTIAL screens | Screen standards across apps | ~20 PARTIAL rows |
| 4 | Platform | Tenant edit, suspend and reactivate (with token revocation), tenant settings, application registry API and screens, platform dashboard from real data | Super Admin | SA-01..SA-15 |
| 5 | Platform | Billing service: plans, packages, subscriptions, renewal and suspension job, invoices; entitlements driven by the subscription | Workflows 2 and 16 | SA-16..SA-21, SA-28..SA-33 |
| 6 | Commerce | Payment service (decision 1); checkout shipping and payment steps; refunds through the provider; accounting postings | Workflows 8 and 11 | SF-08, SF-10, SF-11, EC-23, EC-30 |
| 7 | Commerce | Catalog: categories, attributes, brands, variants, media upload (needs file storage), import and export, review moderation, price lists, tax configuration | E-Commerce Admin | EC-05..EC-11, EC-14, EC-26, EC-29 |
| 8 | Operations | Inventory: requisitions, stock counts, lots, serials, expiry; vehicles and pickup windows in delivery | Workflows 5 and 6 | INV-15, INV-19, INV-21..INV-23, DLV-08, DLV-11 |
| 9 | Operations | Vendor / restaurant service and MFE: vendors, branches, menus, kitchen display, settlements | Workflow 9 | VEN-* |
| 10 | Finance | Accounting setup (fiscal years, currencies, tax codes, cost centers, branches), trial balance, AR and AP documents, aging, banking and reconciliation | Workflows 14 and 15 | ACC-* |
| 11 | Customer | Support service and MFE; CRM campaigns, consent and feedback (needs an email and SMS provider decision) | Workflows 12 and 13 | SUP-*, CRM-13..CRM-19 |
| 12 | Quality | QA projects, plans, manual cases, defects, release gates | Workflow 17 | QA-* |
| 13 | Validation | Playwright suite over every application and role; fix defects; deploy all services; final report | Acceptance criteria | All |

**Scale:** about 184 rows are not complete, and four new services are needed. This is several months of engineering, not a single pass. I will work step by step, commit each step, and keep the completion figures generated from the matrix so progress is never overstated.

## Appendix: tables by service

| Service | Tables |
|---|---|
| user (16) | users, access_roles, access_permissions, access_role_permissions, access_audit_log, tenants, tenant_memberships, application_registry, tenant_application_entitlements, user_application_grants, tenant_role_assignments, tenant_access_audit_log, customer_address, customer_loyalty, loyalty_transaction, revoked_tokens |
| product (31) | product, product_category, product_variant, product_review, product_audit_log, customer_wishlist, recently_viewed, back_in_stock_alert, stock_reservation, product_prices, scheduled_prices, price_history, pricing_settings, promotion_* (7), coupon_codes, coupon_redemptions, storefront* (10) |
| order (35) | orders, order_items, order_status_history, order_tracking_event, order_notes, order_fulfillments, order_fulfillment_items, order_exceptions, order_bulk_jobs, order_settings, order_return, return_* (13), exchange_requests, reverse_pickup_requests, outbox_events, and others |
| accounting (21) | accounts, journal_entries, journal_lines, journal_drafts, journal_draft_lines, ledger_transactions, invoices, invoice_line_items, accounting_periods, refund_requests, financial_report_snapshots, accounting_audit_log, accounting_imports, customer_payment_methods, customer_accounting_preferences, and order projections |
| delivery (21) | delivery, delivery_assignment, delivery_trip, delivery_trip_stop, delivery_zone, delivery_partner_*, delivery_proof, delivery_otp, delivery_incident, delivery_location_update, delivery_pricing_policy, delivery_rule_set, delivery_configuration, delivery_status_history, delivery_audit_event, delivery_outbox, delivery_idempotency, delivery_order_sync |
| procurement (18) | suppliers, supplier_products, purchase_orders, purchase_order_lines, goods_receipts, goods_receipt_lines, supplier_invoices, supplier_invoice_lines, procurement_audit; proc_supplier, proc_supplier_item, proc_po, proc_po_line, proc_receipt, proc_receipt_line, proc_return, proc_return_line, proc_audit |
| inventory (14) | inventory_item, stock_movement, stock_reservation, back_in_stock_alert, wms_warehouse, wms_location, wms_stock, wms_transfer, wms_transfer_line, wms_adjustment, wms_cost_layer, wms_config, wms_audit_log, outbox_events |
| crm (13) | crm_contacts, crm_companies, crm_contact_company, crm_activities, crm_opportunities, crm_pipeline_stages, crm_tasks, crm_segments, crm_segment_members, crm_custom_fields, crm_custom_field_values, crm_saved_views, crm_audit_log |
| qa (9) | qa_features, qa_feature_tests, qa_test_suites, qa_test_runs, qa_test_results, qa_active_run_lock, qa_rbac_roles, qa_rbac_role_permissions, qa_audit_log |
| stock (6), analytics (6), cart (4), loyalty (4), notification (2), error-stack (2) | see migrations |
