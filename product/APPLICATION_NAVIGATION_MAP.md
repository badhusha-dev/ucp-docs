# Application Navigation Map

**Baseline:** commit `962a7e58`, 2026-11-02.

## Step 2 update (2026-11-09)

Problems 1, 2 and 4 below are addressed:
- **Launcher:** "My applications" (`/apps`, account menu) is built from live grants for every workspace, plus platform applications for platform staff.
- **Workspace switching:** each application's sign-in lists the person's workspaces that grant it and opens one without a password (token exchange); "Switch workspace" ends the application session on the server.
- **Separate sites:** Super Admin (3013) and Delivery (3014) open from the launcher in a new tab and still ask for their own sign-in, because browser storage is per site.

Problem 3 (role-based menu items) remains: the shell menu still uses the account's platform permissions, not client grants.

## Phase 0 baseline

There are three separate entry points; a user has to know which URL to open.

| Entry point | Port (local) | Contains |
|---|---|---|
| Shell (`shell-app`) | 3000 | Storefront, account, cart, orders, loyalty, and every back-office module loaded by module federation (catalog, pricing, storefront studio, order control tower, returns, analytics, stock, inventory, procurement, accounting, CRM, error stack, Quality Center) |
| Super Admin (`mfe-super-admin`) | 3013 | Tenants, application entitlements, members, grants |
| Delivery (`mfe-delivery`) | 3014 | Delivery operations and partner workspace |

The shell builds one navigation for all signed-in staff from the platform role (ADMIN, MANAGER, SUPER_ADMIN) and permissions. Within it, each back-office screen asks for its application sign-in (`ApplicationGate`), so one browser session can hold several application sessions.

### Problems

1. No application launcher. There is no "my applications" view based on the user's grants (SH-04).
2. No tenant switcher. A user must type the workspace slug at every application sign-in (SH-03).
3. Menus are role-based, not grant-based: a manager sees CRM, procurement and analytics links even without a grant, then meets a sign-in that refuses them. Not a security hole, since the backend refuses, but a dead end (brief §8, "no dead navigation items").
4. Super Admin and Delivery are not reachable from the shell.

## Target

One shell, one sign-in, then an application launcher built from `user_application_grants` for the selected tenant:

```
Sign in ──► Choose workspace (tenant) ──► Application launcher (only granted, entitled, ACTIVE applications)
                                             ├─ Super Admin        (platform roles only; no tenant)
                                             ├─ E-Commerce Admin   Dashboard · Catalog · Pricing · Orders · Returns · Customers · Storefront · Reports · Team · Audit
                                             ├─ Storefront         (customer-facing; separate public entry)
                                             ├─ Inventory          Dashboard · Stock · Warehouses · Movements · Procurement · Counts · Valuation · Reports · Audit
                                             ├─ Accounting         Dashboard · Setup · Ledger · Receivables · Payables · Banking · Reports · Periods
                                             ├─ Delivery           Dashboard · Dispatch · Deliveries · Partners · Vehicles · Zones · Reports · Audit
                                             ├─ Vendor/Restaurant  Dashboard · Profile · Branches · Menu · Kitchen · Orders · Finance
                                             ├─ CRM                Dashboard · Customers · Leads · Pipeline · Tasks · Segments · Campaigns · Reports
                                             ├─ Quality Center     Dashboard · Projects · Plans · Cases · Runs · Defects · Gates · Reports
                                             └─ Support            Dashboard · Tickets · Knowledge base · SLA · Reports
```

### Rules

- Every menu item maps to a permission. It is shown only when the selected tenant grants the application and the user's role holds the permission. The backend still enforces it.
- Each application opens on a role-specific dashboard: the first item the role can see.
- Breadcrumbs follow `Application › Section › Record`.

Built in plan steps 2 and 3 ([ROLE_SCREEN_FEATURE_AUDIT.md](ROLE_SCREEN_FEATURE_AUDIT.md) §11).
