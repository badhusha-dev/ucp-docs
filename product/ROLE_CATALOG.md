# Role Catalog

**Baseline:** user-service V20 (role engine v2), 2026-11-09. Source: user-service migrations V4–V20 (`access_roles`, `access_role_permissions`, `tenant_user_roles`).

## How roles work (V20, verified in source and tests)

| Concept | Where | Now |
|---|---|---|
| User identity | `users` | One row per person; `users.role` is the **platform** role used by platform login (customers, platform staff) |
| Tenant membership | `tenant_memberships` | ACTIVE, SUSPENDED, INVITED; clients suspend and reactivate their own members |
| Application entitlement | `tenant_application_entitlements` | Per client; `plan_code` waits for billing (step 5) |
| User application grant | `user_application_grants` | Per member and application, with revocation |
| Role assignment | `tenant_user_roles` | **Any number of roles** per member and application, each with optional scope, expiry and delegator. `tenant_role_assignments` is kept for history and no longer written |
| Permission assignment | `access_role_permissions` | Built-in roles are shared; **custom roles belong to one client** (`owner_tenant_id`) |
| User-specific grants | `tenant_user_permission_grants` | A single permission with a reason and optional expiry |
| Resource scope | `tenant_user_roles.scope_type/scope_id` | Modelled and carried in tokens as `scoped_permissions`; accepted only for applications that declare the scope enforced (none yet, step 8) |
| Delegation | `tenant_user_roles.delegated_by` | Lend your own role until a date (≤ 90 days); ends early if you lose the role |
| Platform vs client | `role_scope`, `permission_scope` | Platform roles never enter a client; platform permissions never enter a client token |

Application tokens (`/api/users/login/application`) carry `tenant_id`, `application_code`, `tenant_role` and `permissions`, and every tenant-scoped service authorizes from those permissions (QA round 2, B-42).

## Existing roles (45)

V20 added 12: PLATFORM_OPS (900), PLATFORM_SUPPORT_AGENT (300), PLATFORM_AUDITOR (250), TENANT_OWNER (900), CFO (750), CRM_MANAGER (450), QA_MANAGER (450), CATALOG_MANAGER (420), TEST_ENGINEER (300), CRM_ANALYST (150), FINANCE_VIEWER (130), STORE_VIEWER (120). The 33 earlier roles:

| Role key | Name | Level | Used by |
|---|---|---|---|
| SUPER_ADMIN | Super Admin | 1000 | Platform |
| ADMIN | Administrator | 800 | All tenant applications (broad) |
| DELIVERY_ADMIN | Delivery Administrator | 700 | Delivery |
| RETURNS_MANAGER | Returns Manager | 620 | E-commerce returns |
| ORDER_MANAGER | Order Manager | 600 | E-commerce orders |
| INVENTORY_MANAGER | Inventory Manager | 600 | Inventory |
| PROCUREMENT_MANAGER | Procurement Manager | 580 | Inventory procurement |
| STOREFRONT_PUBLISHER | Storefront Publisher | 550 | Storefront studio |
| PRICING_APPROVER | Pricing Approver | 550 | Pricing |
| REFUND_APPROVER | Refund Approver | 540 | Returns and refunds |
| MANAGER | Manager | 500 | Broad, legacy |
| DISPATCHER | Delivery Dispatcher | 500 | Delivery |
| STOREFRONT_MANAGER | Storefront Manager | 450 | Storefront studio |
| PRICING_MANAGER | Pricing Manager | 450 | Pricing |
| MARKETING_MANAGER | Marketing Manager | 400 | Promotions |
| ACCOUNTANT | Accountant | 350 | Accounting |
| ORDER_OPERATOR | Order Operator | 350 | Orders |
| RETURNS_AGENT | Returns Agent | 340 | Returns |
| PROCUREMENT_OFFICER | Procurement Officer | 330 | Procurement |
| CONTENT_EDITOR | Content Editor | 300 | Storefront studio |
| WAREHOUSE_OPERATOR | Warehouse Operator | 280 | Inventory (the level is 280; V6 tried to insert 220, and the conflict kept 280) |
| WAREHOUSE_INSPECTOR | Warehouse Inspector | 260 | Returns inspection |
| AUDITOR | Auditor | 250 | Read-only, accounting and others |
| ORDER_SUPPORT | Order Support Agent | 250 | Orders |
| DELIVERY_PARTNER | Delivery Partner | 200 | Delivery driver |
| QA_ANALYST | QA Analyst | 160 | Quality Center |
| ORDER_ANALYST | Order Analyst | 150 | Orders, read-only |
| RETURNS_ANALYST | Returns Analyst | 150 | Returns, read-only |
| DELIVERY_ANALYST | Delivery Analyst | 150 | Delivery, read-only |
| PRICING_ANALYST | Pricing Analyst | 150 | Pricing, read-only |
| INVENTORY_ANALYST | Inventory Analyst | 140 | Inventory, read-only |
| USER | Standard User | 100 | Shopper |
| DELIVERY_RECIPIENT | Delivery Recipient | 100 | Delivery tracking |

## Required roles by application

Role codes used in the [master matrix](SCREEN_FEATURE_COMPLETION_MATRIX.md). **Mapping:**
- **EXISTS:** a role with this meaning is seeded.
- **ALIAS:** an existing role covers it today; a dedicated role is proposed.
- **MISSING:** no role and no permissions exist.

### 1. Super Admin (platform roles, no tenant)

| Brief role | Code | Mapping | Note |
|---|---|---|---|
| Platform Owner | PLATFORM_OWNER | ALIAS of SUPER_ADMIN | Needed for break-glass actions (deleting tenants, changing owners) |
| Super Administrator | SUPER_ADMIN | EXISTS | |
| Platform Operations Manager | PLATFORM_OPS | EXISTS (V20) | `platform:tenants.read/manage`, `platform:audit.read` |
| Platform Finance Manager | PLATFORM_FINANCE | MISSING | Subscriptions, invoices, refunds. Needs the billing domain |
| Platform Support Agent | PLATFORM_SUPPORT_AGENT | EXISTS (V20) | `platform:tenants.read` |
| Platform Auditor | PLATFORM_AUDITOR | EXISTS (V20) | `platform:tenants.read`, `platform:audit.read` |

### 2. E-Commerce Admin

| Brief role | Code | Mapping |
|---|---|---|
| Tenant Owner | TENANT_OWNER | EXISTS (V20): every tenant permission, including team and custom-role management |
| Store Administrator | STORE_ADMIN | ALIAS of ADMIN |
| E-Commerce Manager | ECOM_MANAGER | ALIAS of MANAGER |
| Product Manager | PRODUCT_MANAGER | MISSING (catalog:manage is only on ADMIN, MANAGER, SUPER_ADMIN) |
| Catalog Manager | CATALOG_MANAGER | EXISTS (V20): `catalog:read`, `catalog:manage` |
| Pricing Manager | PRICING_MANAGER | EXISTS (with PRICING_APPROVER, PRICING_ANALYST) |
| Order Manager | ORDER_MANAGER | EXISTS (with ORDER_OPERATOR, ORDER_SUPPORT, ORDER_ANALYST) |
| Customer Service Manager | CS_MANAGER | ALIAS of RETURNS_MANAGER and ORDER_SUPPORT |
| Marketing Manager | MARKETING_MANAGER | EXISTS |
| Warehouse Manager | WAREHOUSE_MANAGER | ALIAS of INVENTORY_MANAGER |
| Finance Manager | FINANCE_MANAGER | ALIAS of REFUND_APPROVER and ACCOUNTANT |
| Store Viewer | STORE_VIEWER | EXISTS (V20): read-only catalog, orders, returns, pricing, storefront, stock, analytics |

### 3. Storefront

| Brief role | Code | Mapping |
|---|---|---|
| Guest | GUEST | EXISTS (unauthenticated) |
| Registered Customer | CUSTOMER | EXISTS (USER; the platform token maps to CUSTOMER in tenant services) |
| Wholesale Customer | WHOLESALE | MISSING: needs customer type and price lists |
| Business Customer | BUSINESS | MISSING: needs company accounts and invoicing terms |

### 4. Inventory

| Brief role | Code | Mapping |
|---|---|---|
| Inventory Administrator | INV_ADMIN | ALIAS of ADMIN |
| Inventory Manager | INV_MANAGER | EXISTS |
| Warehouse Manager | WAREHOUSE_MANAGER | ALIAS of INVENTORY_MANAGER; needs a warehouse scope |
| Stock Controller | STOCK_CONTROLLER | ALIAS of WAREHOUSE_OPERATOR |
| Storekeeper | STOREKEEPER | ALIAS of WAREHOUSE_OPERATOR |
| Procurement Manager | PROCUREMENT_MANAGER | EXISTS (with PROCUREMENT_OFFICER) |
| Inventory Auditor | INV_AUDITOR | ALIAS of AUDITOR |
| Inventory Viewer | INV_VIEWER | EXISTS as INVENTORY_ANALYST |

### 5. Accounting

| Brief role | Code | Mapping |
|---|---|---|
| Finance Administrator | FIN_ADMIN | ALIAS of ADMIN |
| Chief Financial Officer | CFO | EXISTS (V20): all accounting permissions |
| Finance Manager | FINANCE_MANAGER | ALIAS of MANAGER (accounting:approve) |
| Accountant | ACCOUNTANT | EXISTS |
| Accounts Payable Officer | AP_OFFICER | MISSING (no AP module) |
| Accounts Receivable Officer | AR_OFFICER | MISSING |
| Tax Manager | TAX_MANAGER | MISSING (no tax module) |
| Treasury Officer | TREASURY | MISSING (no banking module) |
| Auditor | AUDITOR | EXISTS |
| Finance Viewer | FINANCE_VIEWER | EXISTS (V20): `accounting:read`, `accounting:export` |

### 6. Delivery

| Brief role | Code | Mapping |
|---|---|---|
| Delivery Administrator | DLV_ADMIN | EXISTS as DELIVERY_ADMIN |
| Delivery Manager | DLV_MANAGER | ALIAS of DELIVERY_ADMIN |
| Dispatch Manager | DISPATCH_MANAGER | ALIAS of DISPATCHER |
| Dispatcher | DISPATCHER | EXISTS |
| Delivery Partner | DELIVERY_PARTNER | EXISTS |
| Delivery Driver | DRIVER | ALIAS of DELIVERY_PARTNER (only own deliveries, tested) |
| Delivery Auditor | DLV_AUDITOR | ALIAS of DELIVERY_ANALYST |
| Delivery Viewer | DLV_VIEWER | EXISTS as DELIVERY_ANALYST |

### 7. Vendor / Restaurant: all MISSING

VENDOR_OWNER, VENDOR_ADMIN, RESTAURANT_MANAGER, BRANCH_MANAGER, MENU_MANAGER, KITCHEN_MANAGER, KITCHEN_STAFF, VENDOR_ACCOUNTANT, VENDOR_SUPPORT_AGENT. The application is registered as `VENDOR_PORTAL` (PLANNED) with no permissions.

### 8. CRM

| Brief role | Code | Mapping |
|---|---|---|
| CRM Administrator | CRM_ADMIN | ALIAS of ADMIN (crm:manage) |
| CRM Manager | CRM_MANAGER | EXISTS (V20): CRM and loyalty management |
| Sales Manager, Sales Executive | SALES_MANAGER, SALES_EXEC | MISSING |
| Marketing Manager | MARKETING_MANAGER | EXISTS (no CRM permissions) |
| Campaign Manager | CAMPAIGN_MANAGER | MISSING |
| Customer Support Agent | SUPPORT_AGENT | MISSING |
| CRM Analyst | CRM_ANALYST | EXISTS (V20): `crm:read`, `analytics:read` |

CRM has no application of its own in practice: its APIs accept E-commerce Admin tokens (QA report 11, risk 1), while the registry lists `CRM` as PLANNED.

### 9. Quality Center

| Brief role | Code | Mapping |
|---|---|---|
| QA Administrator | QA_ADMIN | ALIAS of ADMIN |
| QA Manager | QA_MANAGER | EXISTS (V20): `qa:read`, `qa:run`, `qa:cancel` |
| Test Engineer | TEST_ENGINEER | EXISTS (V20): `qa:read`, `qa:run` |
| Test Lead, Automation Engineer | TEST_LEAD, AUTOMATION_ENG | ALIAS of TEST_ENGINEER |
| Developer | DEVELOPER | MISSING |
| Defect Reviewer | DEFECT_REVIEWER | MISSING (no defect module) |
| QA Viewer | QA_VIEWER | ALIAS of AUDITOR |

### 10. Support: all MISSING

SUPPORT_ADMIN, SUPPORT_MANAGER, SUPPORT_TEAM_LEAD, SUPPORT_AGENT, TECH_SUPPORT_ENGINEER, SUPPORT_AUDITOR; customers reuse CUSTOMER. Registered as `CUSTOMER_SUPPORT` (PLANNED).

### Matrix group codes

`ALL` (every signed-in user), `ALL TENANT ROLES`, `ALL FINANCE` (every Accounting role) group rows in the matrix that apply broadly.

## Totals

| | Count |
|---|---|
| Roles in the brief | 80 |
| Already seeded with that meaning (EXISTS) | 29 (17 before V20) |
| Covered by an alias today (ALIAS) | 22 |
| Missing | 29 |

Counted from the tables above. Customer in Support reuses CUSTOMER and counts as EXISTS.
