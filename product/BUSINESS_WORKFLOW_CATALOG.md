# Business Workflow Catalog

**Baseline:** commit `962a7e58`, 2026-11-02. These are the 17 workflows from the brief (§10).

**Grades:**
- **WORKS:** every step exists and backend tests or live scripts cover the chain (no automated browser test yet).
- **PARTIAL:** some steps exist.
- **BLOCKED:** a whole domain is missing.

| # | Workflow | Grade | Steps that exist | Missing | Consistency mechanism | Evidence |
|---|---|---|---|---|---|---|
| 1 | Tenant registration and onboarding | PARTIAL | Platform operator creates tenant, adds members, enables applications, grants roles | Self-service sign-up, onboarding checklist, tenant settings, edit and suspend | Single service (user-service), transactional, audit log | `TenantAccessPostgresIntegrationTest` |
| 2 | Subscription purchase and application activation | BLOCKED | Entitlement can be enabled by hand | Plans, subscriptions, payment, activation driven by subscription | — | — |
| 3 | User invitation and role assignment | PARTIAL | Add an existing user by email; grant application with one role | Invitation email and acceptance (`INVITED` status unused); several roles; tenant owner self-service | Transactional in user-service | Postgres test |
| 4 | Product creation and publication | WORKS | Create product, price it, publish to storefront catalog | Approval workflow, media upload | Product-service transaction; storefront reads catalog via internal API | `ProductControllerIntegrationTest`, `StorefrontCatalogTest`, browser (round 3) |
| 5 | Procurement and goods receipt | WORKS | Supplier, purchase order, approval, receipt, stock increase, supplier return | Requisitions, AP posting of the supplier invoice | Procurement calls inventory internal API; outbox events | `UcpProcurementTest`, `OpsIntegrationTest` |
| 6 | Inventory reservation and stock movement | WORKS | Reservation at checkout, release, movement history, transfers, adjustments with approval | Lots and serials | Idempotent reservations; outbox | `InventoryServiceIntegrationTest`, `OrderCheckoutIntegrationTest` |
| 7 | Retail cart and checkout | WORKS | Cart, server-side pricing, stock check, order creation, cart conversion | Shipping selection | Order service reserves stock before saving; service token for cart conversion | live `live-checkout.sh` 16/16, `OrderPricingCheckoutTest` |
| 8 | Payment and order confirmation | BLOCKED | Order confirmation without payment | Payment service and provider | — | — |
| 9 | Restaurant order and kitchen processing | BLOCKED | Restaurant transitions on orders (backend only) | Vendors, menus, kitchen display | — | — |
| 10 | Delivery assignment and completion | WORKS | Delivery auto-created on fulfilment, dispatch, partner accepts, proof and OTP, completion syncs order status | Vehicles, pickup windows | Internal API plus delivery outbox; status sync consumer in order-service | `DeliveryLifecycleTest`, `DeliveryStatusSyncTest` |
| 11 | Return and refund | PARTIAL | Return request, approval, pickup, inspection, refund record and approval | Money movement (no payment service); refund to accounting is a record, not a provider refund | Return events through outbox to accounting | `ReturnsApiIntegrationTest`, `RmaDomainTest` |
| 12 | Customer support ticket | BLOCKED | — | Support domain | — | — |
| 13 | CRM campaign and customer segmentation | PARTIAL | Segments with criteria and members | Campaigns, consent, email and SMS sending | Single service | CRM Postgres tests |
| 14 | Customer invoice and supplier payment | PARTIAL | Customer invoices; supplier invoices with approval in procurement | AP ledger, payment vouchers, supplier payments | — | `AccountingServiceTest` |
| 15 | Financial posting and reconciliation | PARTIAL | Order and refund postings, balanced journals, approval, period close, reconciliation decisions | Bank accounts, bank statement import and matching, trial balance | Internal accounting API; immutable posted lines | `AccountingTenantIsolationPostgresTest` |
| 16 | Subscription renewal and suspension | BLOCKED | Tenant status is honoured at sign-in | Subscriptions, renewal job, suspension trigger | — | — |
| 17 | QA test execution and release approval | PARTIAL | Start, cancel and rerun test runs; results and reports | Release quality gates, approval | Run lock table | qa-service tests |

**Totals:** 5 WORKS, 7 PARTIAL, 5 BLOCKED.
