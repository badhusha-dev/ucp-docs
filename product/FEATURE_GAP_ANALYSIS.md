# Feature Gap Analysis

**Baseline:** commit `962a7e58`, 2026-11-02. Each gap names its matrix rows ([SCREEN_FEATURE_COMPLETION_MATRIX.md](SCREEN_FEATURE_COMPLETION_MATRIX.md)) and the plan step that closes it ([ROLE_SCREEN_FEATURE_AUDIT.md](ROLE_SCREEN_FEATURE_AUDIT.md) §11).

## Severity

- **Critical:** blocks a core commercial or financial workflow, or a security requirement of the brief.
- **High:** blocks a whole application or role family.
- **Medium:** a missing feature inside a working application.
- **Low:** screen-standard or convenience gaps.

## Critical

| Gap | Why critical | Rows | Step |
|---|---|---|---|
| No payment service or provider | Orders are never paid, and refunds move no money; workflows 8 and 11 cannot finish | SF-11, EC-23, EC-30, SA-29 | 6 (decision 1) |
| ~~Role engine: one role per application, global custom roles, no scope, no delegation~~ | **Closed in step 1 (V20)** except scope enforcement: several roles, client-owned custom roles, direct grants, expiring delegation, escalation guards. Scoped roles are modelled but refused until a service filters by scope | SH-07, SA-25, EC-35 | 1 (scopes: 8) |
| No subscriptions or billing | The SaaS model cannot charge tenants or activate applications on purchase; workflows 2 and 16 | SA-16..SA-21, SA-28..SA-33 | 5 |
| Tenant suspension cannot be operated | The status is honoured at sign-in, but there is no way to set it, and issued tokens stay valid | SA-06 | 4 |
| ~~No password reset~~ | **Closed in step 2 (V21):** one-time hashed reset links, self-service by email when SMTP is configured, assisted links from User administration, password change with session sign-out. A real SMTP server has not been connected yet (tested with a mocked sender) | SH-13, SF-20 | 2 |

## High

| Gap | Rows | Step |
|---|---|---|
| Vendor / Restaurant application missing entirely | VEN-* (22) | 9 |
| Support application missing entirely | SUP-* (16), SF-23 | 11 |
| Accounts payable, banking and bank reconciliation missing | ACC-26..ACC-35 | 10 |
| Accounting setup missing (fiscal years, currencies, tax codes, cost centers, branches) | ACC-03, ACC-05..ACC-10 | 10 |
| ~~Platform roles missing~~ Operations, support and auditor added in V20 and enforced by the platform API and gateway; Platform Finance waits for billing. (The "SUPER_ADMIN lacks 86 permissions" note was wrong: it holds `*` in code) | ROLE_CATALOG §1, ROLE_PERMISSION_MATRIX | 1, 5 |
| ~~Tenant owners cannot manage their own team~~ Closed in step 1: Client team screen and `/api/tenant/**` | EC-35 | 1 |
| Application registry has no API or screen | SA-12..SA-15 | 4 |
| ~~No launcher or tenant switcher~~ Closed in step 2: "My applications" launcher and one-click workspace sign-in. Super Admin and Delivery remain separate sites (opened from the launcher, own sign-in) | SH-03, SH-04 | 2 |

## Medium

| Gap | Rows | Step |
|---|---|---|
| Catalog: attributes, brands, category and variant administration, media upload, import and export | EC-05..EC-11 | 7 |
| Tax configuration | EC-29, ACC-07 | 7, 10 |
| Shipping selection at checkout | SF-10 | 6 |
| Inventory: requisitions, stock counts, lots, serials, expiry | INV-15, INV-19, INV-21..INV-23 | 8 |
| Delivery vehicles, pickup windows, reattempt policy | DLV-08, DLV-11, DLV-16 | 8 |
| CRM campaigns, consent, feedback, leads | CRM-07, CRM-13..CRM-19 | 11 |
| QA test plans, manual cases, defects, release gates | QA-02..QA-06, QA-09, QA-10, QA-14 | 12 |
| Trial balance, aging, credit and debit notes | ACC-15, ACC-20, ACC-24, ACC-27 | 10 |
| File and media storage (needed by catalog media, support attachments, QA evidence) | SH-08 | 7 |
| Platform configuration: settings, templates, flags, localization | SA-34..SA-40, SH-10 | 4 |

## Low

| Gap | Rows | Step |
|---|---|---|
| Search, filters and pagination missing on working lists (catalog admin, tenant list, users) | EC-03, SA-02, SA-22 and others | 3 |
| Product comparison | SF-05 | 7 |
| Global search | SH-06 | 13 |
| In-app monitoring screens (health already available in Grafana and Kafka UI) | SA-41, SA-42, SH-15 | 4 |

## Architectural debt affecting the plan

| Debt | Impact | Decision |
|---|---|---|
| Stock owned by stock-service and inventory-service | Two sources of truth for availability | Decision 3 |
| Two procurement implementations (`ProcurementController`, `UcpController`) | Duplicate screens and tables | Decision 3 |
| Loyalty in user-service and loyalty-service | Duplicate balances | Decision 3 |
| Accounting and analytics hold copies of order tables | Projection drift unless event-fed | Verify during step 10 |
