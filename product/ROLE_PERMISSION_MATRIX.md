# Role–Permission Matrix

**Baseline:** user-service V20 (role engine v2), 2026-11-09.

## 1. Current grants (generated, not hand-typed)

Read from a scratch Postgres database migrated through user-service V20:

```sql
SELECT p.module_name, p.permission_key, p.permission_scope, string_agg(rp.role_key, ', ')
FROM access_permissions p LEFT JOIN access_role_permissions rp USING (permission_key)
GROUP BY 1, 2, 3;
```

That database holds 119 permissions, 495 grants and 45 roles.

### How effective access is computed (V20)

- **`SUPER_ADMIN`** gets the wildcard `*` in code (`RbacService.permissionsFor`), and every service and the gateway honour `*`. So the grant table listing only a few of its permissions has no effect. (The Phase 0 audit called this a gap; reading the code showed it is not one.)
- **Scopes:**
  - Roles and permissions carry a scope: PLATFORM or TENANT.
  - PLATFORM roles (`SUPER_ADMIN`, `PLATFORM_OPS`, `PLATFORM_SUPPORT_AGENT`, `PLATFORM_AUDITOR`) can never be assigned inside a client.
  - PLATFORM permissions are dropped from every client token, even when a shared role holds one.
- **Client token:** the union of every live tenant-wide role of the member in that application, plus live direct grants. Expired, revoked and orphaned delegations do not count.
- **Scoped roles** (branch, warehouse or store) are reported separately in `scoped_permissions`. They never widen `permissions`, and they are accepted only for applications that declare the scope as enforced.
- **Gateway caveat:** a permission granted here but checked nowhere grants nothing, so check enforcement in code, not in this table.

| Module | Permission | Scope | Roles holding it (highest level first) |
|---|---|---|---|
| accounting | `accounting:approve` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER |
| accounting | `accounting:export` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, ACCOUNTANT, AUDITOR, FINANCE_VIEWER |
| accounting | `accounting:import` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, ACCOUNTANT |
| accounting | `accounting:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, ACCOUNTANT, AUDITOR, FINANCE_VIEWER |
| accounting | `accounting:reconcile` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, ACCOUNTANT |
| accounting | `accounting:write` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, ACCOUNTANT |
| analytics | `analytics:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CFO, MANAGER, AUDITOR, CRM_ANALYST, STORE_VIEWER |
| application | `application:admin` | PLATFORM | SUPER_ADMIN |
| catalog | `catalog:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, CATALOG_MANAGER |
| catalog | `catalog:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, CATALOG_MANAGER, STORE_VIEWER |
| crm | `crm:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, CRM_MANAGER |
| crm | `crm:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, CRM_MANAGER, CRM_ANALYST |
| delivery | `delivery:audit` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DELIVERY_ANALYST |
| delivery | `delivery:config` | TENANT | TENANT_OWNER, DELIVERY_ADMIN |
| delivery | `delivery:dispatch` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER |
| delivery | `delivery:incident` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER, DELIVERY_PARTNER |
| delivery | `delivery:manage` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER |
| delivery | `delivery:partner:self` | TENANT | DELIVERY_ADMIN, DELIVERY_PARTNER |
| delivery | `delivery:partners:manage` | TENANT | TENANT_OWNER, DELIVERY_ADMIN |
| delivery | `delivery:partners:read` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER |
| delivery | `delivery:proof` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DELIVERY_PARTNER |
| delivery | `delivery:read` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER, DELIVERY_PARTNER, DELIVERY_ANALYST |
| delivery | `delivery:report` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER, DELIVERY_ANALYST |
| delivery | `delivery:tracking` | TENANT | TENANT_OWNER, DELIVERY_ADMIN, DISPATCHER, DELIVERY_PARTNER, DELIVERY_ANALYST |
| delivery-recipient | `delivery:track:own` | TENANT | DELIVERY_RECIPIENT |
| error-stack | `errors:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| inventory | `inventory:adjust` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, WAREHOUSE_OPERATOR |
| inventory | `inventory:adjust-approve` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER |
| inventory | `inventory:audit` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER |
| inventory | `inventory:config` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER |
| inventory | `inventory:export` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, PROCUREMENT_MANAGER, INVENTORY_ANALYST |
| inventory | `inventory:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, WAREHOUSE_OPERATOR |
| inventory | `inventory:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, WAREHOUSE_OPERATOR |
| inventory | `inventory:transfer` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, WAREHOUSE_OPERATOR |
| inventory | `inventory:transfer-approve` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER |
| inventory | `inventory:view` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, PROCUREMENT_MANAGER, PROCUREMENT_OFFICER, WAREHOUSE_OPERATOR, INVENTORY_ANALYST |
| inventory | `warehouse:manage` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER |
| inventory | `warehouse:view` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, WAREHOUSE_OPERATOR, INVENTORY_ANALYST |
| loyalty | `loyalty:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, CRM_MANAGER |
| orders | `order:audit:view` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_ANALYST |
| orders | `order:bulk` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER |
| orders | `order:cancel` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER |
| orders | `order:confirm` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR |
| orders | `order:customer:view` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_SUPPORT |
| orders | `order:exception:manage` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR |
| orders | `order:export` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_ANALYST |
| orders | `order:fulfillment:view` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR, ORDER_SUPPORT |
| orders | `order:hold` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR |
| orders | `order:note:create` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR, ORDER_SUPPORT |
| orders | `order:process` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR |
| orders | `order:release` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_OPERATOR |
| orders | `order:report:view` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER, ORDER_ANALYST, STORE_VIEWER |
| orders | `order:settings` | TENANT | TENANT_OWNER, ADMIN, ORDER_MANAGER |
| orders | `order:view` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, ORDER_MANAGER, REFUND_APPROVER, ORDER_OPERATOR, RETURNS_AGENT, WAREHOUSE_INSPECTOR, ORDER_SUPPORT, ORDER_ANALYST, RETURNS_ANALYST, STORE_VIEWER |
| orders | `order:view-all-stores` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, ORDER_MANAGER, REFUND_APPROVER, ORDER_OPERATOR, RETURNS_AGENT, WAREHOUSE_INSPECTOR, ORDER_SUPPORT, ORDER_ANALYST, RETURNS_ANALYST |
| orders | `orders:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| platform | `platform:audit.read` | PLATFORM | PLATFORM_OPS, PLATFORM_AUDITOR |
| platform | `platform:tenants.manage` | PLATFORM | PLATFORM_OPS |
| platform | `platform:tenants.read` | PLATFORM | PLATFORM_OPS, PLATFORM_SUPPORT_AGENT, PLATFORM_AUDITOR |
| pricing | `coupon:manage` | TENANT | TENANT_OWNER, ADMIN, MARKETING_MANAGER |
| pricing | `pricing:approve` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER |
| pricing | `pricing:audit` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER, PRICING_ANALYST |
| pricing | `pricing:cost:read` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER |
| pricing | `pricing:create` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER |
| pricing | `pricing:export` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER |
| pricing | `pricing:publish` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER |
| pricing | `pricing:read` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER, PRICING_MANAGER, MARKETING_MANAGER, PRICING_ANALYST, STORE_VIEWER |
| pricing | `pricing:update` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER |
| pricing | `promotion:approve` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER |
| pricing | `promotion:create` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER, MARKETING_MANAGER |
| pricing | `promotion:read` | TENANT | TENANT_OWNER, ADMIN, PRICING_APPROVER, PRICING_MANAGER, MARKETING_MANAGER, PRICING_ANALYST, STORE_VIEWER |
| pricing | `promotion:update` | TENANT | TENANT_OWNER, ADMIN, PRICING_MANAGER, MARKETING_MANAGER |
| procurement | `goods-receipt:manage` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, PROCUREMENT_MANAGER, WAREHOUSE_OPERATOR |
| procurement | `procurement:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| procurement | `procurement:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| procurement | `purchase-order:approve` | TENANT | TENANT_OWNER, ADMIN, PROCUREMENT_MANAGER |
| procurement | `purchase-order:manage` | TENANT | TENANT_OWNER, ADMIN, PROCUREMENT_MANAGER, PROCUREMENT_OFFICER |
| procurement | `purchase-order:view` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, PROCUREMENT_MANAGER, PROCUREMENT_OFFICER, WAREHOUSE_OPERATOR, INVENTORY_ANALYST |
| procurement | `supplier-return:manage` | TENANT | TENANT_OWNER, ADMIN, PROCUREMENT_MANAGER |
| procurement | `supplier:manage` | TENANT | TENANT_OWNER, ADMIN, PROCUREMENT_MANAGER |
| procurement | `supplier:view` | TENANT | TENANT_OWNER, ADMIN, INVENTORY_MANAGER, PROCUREMENT_MANAGER, PROCUREMENT_OFFICER, INVENTORY_ANALYST |
| quality-center | `qa:cancel` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, QA_MANAGER |
| quality-center | `qa:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, QA_MANAGER, TEST_ENGINEER, QA_ANALYST |
| quality-center | `qa:run` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, QA_MANAGER, TEST_ENGINEER |
| returns | `return:analytics-view` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, REFUND_APPROVER, RETURNS_ANALYST |
| returns | `return:approve` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:audit-view` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, REFUND_APPROVER, RETURNS_ANALYST |
| returns | `return:config` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER |
| returns | `return:create` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:exception-manage` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER |
| returns | `return:exchange` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:export` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_ANALYST |
| returns | `return:inspect` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, WAREHOUSE_INSPECTOR |
| returns | `return:pickup` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:policy-approve` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER |
| returns | `return:policy-manage` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER |
| returns | `return:receive` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT, WAREHOUSE_INSPECTOR |
| returns | `return:refund-approve` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, REFUND_APPROVER |
| returns | `return:refund-process` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, REFUND_APPROVER |
| returns | `return:reject` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:review` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| returns | `return:view` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, REFUND_APPROVER, RETURNS_AGENT, WAREHOUSE_INSPECTOR, RETURNS_ANALYST, STORE_VIEWER |
| returns | `return:view-customer` | TENANT | TENANT_OWNER, ADMIN, RETURNS_MANAGER, RETURNS_AGENT |
| roles | `roles:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| stock | `stock:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| stock | `stock:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER, STORE_VIEWER |
| storefront | `storefront:approve` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_PUBLISHER |
| storefront | `storefront:audit` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_PUBLISHER |
| storefront | `storefront:domain` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_MANAGER |
| storefront | `storefront:edit` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_MANAGER, CONTENT_EDITOR |
| storefront | `storefront:manage` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_MANAGER |
| storefront | `storefront:media` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_MANAGER, CONTENT_EDITOR |
| storefront | `storefront:publish` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_PUBLISHER |
| storefront | `storefront:read` | TENANT | TENANT_OWNER, ADMIN, STOREFRONT_PUBLISHER, STOREFRONT_MANAGER, CONTENT_EDITOR, STORE_VIEWER |
| tenant | `tenant:roles.manage` | TENANT | TENANT_OWNER |
| tenant | `tenant:users.manage` | TENANT | TENANT_OWNER, ADMIN |
| tenant | `tenant:users.read` | TENANT | TENANT_OWNER, ADMIN |
| users | `users:manage` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |
| users | `users:read` | TENANT | SUPER_ADMIN, TENANT_OWNER, ADMIN, MANAGER |

## 2. Permissions still to add (by plan step)

Names follow the `module:action` convention. Each family is added by a forward-only migration in the step that builds its feature. None is added before its backend check exists.

**Added in step 1 (V20):** `tenant:users.read`, `tenant:users.manage`, `tenant:roles.manage`, `platform:tenants.read`, `platform:tenants.manage`, `platform:audit.read`.

| Family | Permissions | Proposed holders | Step |
|---|---|---|---|
| Platform | `platform:monitoring.read`, `platform:settings.manage`, `platform:tenants.suspend` | PLATFORM_OPS | 4 |
| Billing | `billing:plans.manage`, `billing:subscriptions.read`, `billing:subscriptions.manage`, `billing:invoices.manage`, `billing:refunds.approve` | PLATFORM_FINANCE (role added with billing) | 5 |
| Catalog | `catalog:attributes.manage`, `catalog:import` | PRODUCT_MANAGER, CATALOG_MANAGER | 7 |
| Payment | `payment:read`, `payment:refund`, `payment:configure` | finance and store administrators | 6 |
| Tax | `tax:read`, `tax:manage` | TAX_MANAGER | 7, 10 |
| Accounting | `ar:manage`, `ap:manage`, `bank:manage`, `bank:reconcile`, `accounting:setup` | AR_OFFICER, AP_OFFICER, TREASURY | 10 |
| Inventory | `inventory:count`, `inventory:requisition`, `inventory:lots` | STOCK_CONTROLLER, PROCUREMENT_MANAGER | 8 |
| Vendor | `vendor:read`, `vendor:manage`, `vendor:approve`, `menu:manage`, `kitchen:operate`, `vendor:finance` | vendor roles | 9 |
| CRM | `crm:leads.manage`, `crm:campaign.manage`, `crm:campaign.send`, `crm:consent.manage` | SALES_MANAGER, SALES_EXEC, CAMPAIGN_MANAGER | 11 |
| QA | `qa:plan.manage`, `qa:case.manage`, `qa:defect.manage`, `qa:gate.approve` | QA_MANAGER, TEST_LEAD, DEFECT_REVIEWER | 12 |
| Support | `support:ticket.read`, `support:ticket.manage`, `support:ticket.assign`, `support:sla.manage`, `support:kb.manage`, `support:reports.read` | support roles | 11 |

## 3. Grants, scopes and delegation (implemented in V20)

| Table | Holds |
|---|---|
| `tenant_user_roles` | Any number of roles per member and application. Optional `scope_type` / `scope_id` (TENANT, BRANCH, WAREHOUSE, STORE), optional `expires_at`, and `delegated_by` for loans |
| `tenant_user_permission_grants` | A single permission for one member, with a mandatory reason and optional expiry |
| `access_roles.owner_tenant_id` | Custom roles owned by one client; invisible to others, archived instead of deleted |

**Rules enforced by user-service and covered by `RoleEngineV2PostgresTest`:**
- Nobody grants, removes or puts into a custom role authority they do not hold anywhere in the client.
- A client keeps at least one active owner.
- A delegation needs an expiry within 90 days, cannot be re-delegated, and ends when the delegator loses the role.
- Changes take effect at the next service check (`/internal/authorization/check` compares the token with live access), not at token expiry.

**Scope enforcement:** no application declares a supported scope yet, so scoped assignments are refused with an explanation. Inventory (warehouse) and order management (store) will declare scopes when their services filter by them (plan step 8).
