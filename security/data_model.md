# SNAPPY Data Model

Hybrid database architecture: a shared global control plane with physically isolated per-organization data planes.

---

## 1. Multi-Tenant Strategy

### Physical Isolation Protocol

| Plane             | Storage                                | Purpose                                           |
| ----------------- | -------------------------------------- | ------------------------------------------------- |
| **Control Plane** | Single Shared D1 (`DB_CORE`)           | Global identity, billing, org registry            |
| **Data Plane**    | 1 Dedicated D1 per Organization        | All business module data                          |
| **Asset Plane**   | 1 Dedicated R2 Bucket per Organization | Media, PDFs, attachments                          |
| **Cache Plane**   | Workers KV                             | Sub-10ms org routing resolution, quota fast-reads |

### Module Co-location

All business modules (POS, HR, ERP, CRM, FIN, WMS, etc.) are provisioned in the **same Organization D1 instance**. Module access is enforced at the application layer via `activation_gates` — physical separation is not used for modules, only for organizations.

---

## 2. Control Plane: DB_CORE (Shared D1)

Handles the master records for identity, billing, org routing, and platform governance.

### Identity & Session

- **`users`**: Global identity registry. One user can be a member of multiple organizations.
- **`sessions`**: Database-backed sessions with refresh token rotation and revocation support. Includes `last_activity_at` for idle session tracking.
- **`verification_tokens`**: Time-limited tokens for email verification, password reset, and magic link auth. `type` field differentiates use cases.
- **`user_recovery_codes`**: Pre-generated MFA recovery codes (hashed).
- **`user_login_attempts`**: Rate-limiting and brute force detection. Indexed on `(identifier, created_at)`.

### Organizations & Membership

- **`orgs`**: Master organization record. Key fields:
  - `d1_id` — physical database coordinate (set after provisioning)
  - `r2_bucket` — asset storage coordinate
  - `tier` — billing tier (maps to `tier_config`)
  - `regulatory_ids_json` — NPWP, NIB, and other Indonesian regulatory identifiers
  - `deleted_at` — soft delete
- **`members`**: Links a `user_id` to an `org_id` with an assigned `role_id`. Indexed on `(user_id, org_id)` for fast membership lookups.
- **`invitations`**: Pending invitations to an org. Uses `role_id` (not a flat role string) for precision.

### RBAC (Role-Based Access Control)

SNAPPY uses a permission-key based RBAC system. Roles can be system-wide or scoped to a specific org.

- **`permissions`**: Registry of all granular permission keys (e.g., `pos:transaction:create`, `erp:po:approve`). Keyed by a unique string `key`.
- **`roles`**: Named collections of permissions. `org_id = NULL` = system role. `org_id = X` = org-scoped custom role. Unique index on `(org_id, name)`.
- **`role_permissions`**: Composite join table linking roles to permission keys. Composite primary key `(role_id, permission_key)`.

### Admin Governance (Internal Platform)

Completely separate from tenant-facing RBAC. Used by SNAPPY platform operators only.

- **`admin_roles`**: Platform-level roles (e.g., `platform_admin`, `support_agent`).
- **`admin_members`**: Maps platform users to admin roles.
- **`admin_permissions`**: Statement-based permission grants (e.g., `allow:billing:*`, `deny:orgs:delete`).

### Billing & Lifecycle

- **`tier_config`**: Per-tier limits and pricing. Key columns:
  - `max_employees`, `max_products`, `max_warehouses`, `max_suppliers`, `max_monthly_po` — hard business limits
  - `d1_cap_bytes`, `r2_cap_bytes` — storage quotas
  - `report_watermark` — whether reports include the SNAPPY watermark
  - `allowed_costing_methods` — JSON array (e.g., `["fifo"]`)
- **`modules`**: Registry of all available addon modules with pricing per tier.
- **`activation_gates`**: Per-org, per-module feature flags (`org_id + module_slug → is_enabled`).
- **`usage_state`**: Generalized quota tracking. Uses `(org_id, metric)` key pairs. Example metrics: `trx_daily`, `ai_credits_monthly`, `pdf_daily`.
  - Uses `version` column for optimistic locking in concurrent multi-terminal environments.
- **`billing_cycles`**: Historical billing period records per org.
- **`topup_ledger`**: Prepaid credit history. Includes `expires_at` for weekly rollover enforcement.
- **`api_keys`**: Per-org API keys for programmatic access. Keys stored as `key_hash` (never plaintext).

### Legal & Compliance

- **`legal_documents`**: Version-controlled platform legal text (TOS, Privacy Policy, etc.). Indexed on `(system, slug)` for multi-system support (snappy, bisnapi).
- **`legal_signatures`**: Immutable record of user acceptance per document version. Includes IP and user agent for legal defensibility.

### Billing & Payments

- **`invoices`**: Platform-level invoices for tenant billing (distinct from customer-facing invoices in `pos_customer_invoices`).
- **`payment_audit`**: Immutable log of all payment provider events. `provider_trx_id` uniquely indexed to prevent duplicate processing.

### Infrastructure

- **`provision_jobs`**: Async queue for org provisioning operations. Tracks `retry_count`, `max_retries`, and `last_error` for resilient, observable provisioning.
- **`system_stats`**: Platform-wide aggregate counters. Uses `version` for optimistic locking.
- **`platform_config`**: Key-value runtime configuration for the platform.

---

## 3. Data Plane: Org Dedicated D1

The Genesis tenant baseline is applied to every new D1 at provisioning time. All tables below are provisioned by default; module access is controlled by `activation_gates`.

### POS & Transactions

- **`pos_products`**: Product catalog including `image_url` for digital storefront display.
- **`pos_transactions`**: Source of truth for transaction counts. Includes `public_id` for digital receipt URLs.
- **`pos_transaction_items`**: Line items per transaction.
- **`pos_transaction_payments`**: Payment split records per transaction (supports split-payment).
- **`pos_transaction_tax_lines`**: Tax breakdown per transaction.
- **`pos_returns`**: Guards against double-returns via unique constraint on `original_transaction_id`.
- **`pos_shifts`**: Open/closed cashier sessions. Transactions are validated against OPEN shifts.
- **`pos_credit_accounts`** _(Kasbon)_: Outstanding credit balances per customer. Uses `version` for optimistic locking.
- **`pos_credit_logs`**: Immutable debit/credit log per kasbon account. Type constrained: `CHECK(type IN ('DEBIT', 'CREDIT'))`.
- **`pos_customer_invoices`**: Customer-facing formal invoices (distinct from platform billing invoices). For B2B sales to resellers.
- **`pos_categories`**, **`pos_modifiers`**, **`pos_product_modifiers`**: Catalog organization and F&B modifier support.
- **`pos_bom`** _(Bill of Materials)_: Recipe/component definitions for manufactured products.
- **`pos_discounts`**: Configurable discount rules.
- **`pos_terminals`**: Multi-terminal POS device registry.
- **`pos_tables`**, **`pos_kitchen_stations`**, **`pos_kitchen_queue`**: F&B table management and kitchen display system.
- **`pos_price_lists`**: Tiered pricing per customer segment.

### Booking & Appointments

- **`pos_appointments`**: Service appointment bookings. Links `customer_id`, `product_id` (the service), and `staff_id`. Includes `duration_minutes`.
- **`pos_staff_availability`**: Recurring weekly schedule per staff member (`day_of_week`, `start_time`, `end_time`).
- **`pos_staff_overrides`**: Date-specific exceptions to the weekly schedule (holiday, changed hours, etc.).

### ERP & Inventory

- **`erp_warehouses`**: Physical warehouse locations.
- **`erp_suppliers`**: Supplier registry.
- **`erp_inventory_batches`**: FIFO batch tracking. SQL-level constraint `erp_inv_qty_check` enforces `remaining_quantity >= 0`.
- **`erp_inventory_logs`**: Immutable stock movement audit trail.
- **`erp_purchase_orders`**, **`erp_purchase_order_items`**: Approval-based procurement workflow.
- **`erp_goods_receipts`**, **`erp_goods_receipt_items`**: GRN (Goods Received Note) for PO fulfillment.
- **`erp_production_plans`**, **`erp_work_orders`**: Manufacturing lite — production scheduling and execution.

### Finance & Accounting

- **`fin_entities`**: Legal entity definitions for multi-entity accounting.
- **`fin_accounts`**: Chart of Accounts (COA). Supports hierarchical accounts via `parent_account_id`.
- **`fin_currencies`**: Multi-currency support with `is_base` flag.
- **`fin_journals`**, **`fin_ledgers`**: Double-entry general ledger. Ledger supports multi-currency with `exchange_rate` and base-currency equivalents.
- **`fin_fixed_assets`**: Asset registry with COA linkage for depreciation accounting.
- **`bank_accounts`**: Bank account registry linked to COA.
- **`bank_transactions`**: Raw bank statement import records.
- **`bank_reconciliations`**, **`bank_reconciliation_items`**: Matching bank transactions to ledger journal entries.
- **`expense_claims`**, **`expense_items`**, **`expense_rates`**: Employee expense reimbursement with OCR-ready attachment support.

### Human Resources

- **`hr_employees`**: Employee profile records.
- **`hr_positions`**: Org chart positions. Soft-delete enforced to prevent orphan employee references.
- **`hr_salaries`**: Compensation structures per employee.
- **`hr_attendance`**: Geo-tagged clock-in/out records.
- **`hr_leaves`**: Leave requests and approval tracking.
- **`hr_shifts`**: Shift schedule definitions.
- **`hr_overtime_rules`**: Overtime calculation policy definitions.
- **`hr_commissions`**: Commission calculation records per employee/period.
- **`hr_payroll_runs`**, **`hr_payroll_items`**, **`hr_payroll_tax_calculations`**: Full payroll execution history.
- **`hr_assets`**: Company assets assigned to employees.
- **`hr_employee_documents`**: Document registry per employee (contracts, KTP, NPWP, etc.).

### CRM & Pipeline

- **`crm_contacts`**: Unified customer and supplier registry. Includes `points_balance` and `marketing_consent_at` for loyalty and PDPA compliance.
- **`crm_loyalty_logs`**: Immutable ledger of loyalty point transactions. Includes `balance_after` for audit trail.
- **`crm_pipelines`**, **`crm_pipeline_stages`**: Configurable sales pipeline definitions.
- **`crm_deals`**: Opportunity tracking with stage assignment and close-value tracking.

### Delivery & Logistics

- **`delivery_drivers`**: Driver registry with vehicle plate and active status.
- **`delivery_runs`**: Delivery batch assignment per driver per day.
- **`delivery_stops`**: Individual delivery stops within a run. Includes `sequence`, `status`, `proof_url`, and `failed_reason`.

### WMS (Warehouse Management)

- **`wms_zones`**: Named zones within a warehouse (receiving, storage, dispatch).
- **`wms_bins`**: Physical storage locations within zones. Tracks `max_weight` and `max_volume`.
- **`wms_pick_lists`**: Outbound picking assignments.
- **`wms_pick_items`**: Individual SKUs to pick per list. Tracks `requested_quantity` vs `picked_quantity`.
- **`wms_stock_movements`**: Internal bin-to-bin transfer audit trail.

### Payments

- **`pay_methods`**: Configurable payment methods with provider type and fee structure.
- **`pay_settlement_batches`**: Provider settlement batch records.
- **`pay_payouts`**: Disbursement records to bank accounts.

### Tax Compliance

- **`tax_configs`**: Jurisdiction-specific tax rate definitions.
- **`tax_invoices`**: e-Faktur compatible VAT invoice records.
- **`tax_djp_reports`**: DJP SPT reporting records.
- **`tax_umkm_summaries`**: PPh Final (0.5%) summary reports for UMKM.

### Omni-channel

- **`omni_stores`**: Marketplace channel registrations (Tokopedia, Shopee, TikTok) with OAuth lifecycle tracking.
- **`omni_product_mappings`**: Maps internal `pos_products` to external marketplace SKU IDs.
- **`omni_orders`**: Inbound marketplace orders with fulfillment state machine enforcement.
- **`omni_order_items`**: Line items per omni order.

### Other Modules

- **`sign_documents`**, **`sign_requests`**, **`sign_stamps`**: Certified electronic signature (TTE) module.
- **`project_projects`**, **`project_milestones`**, **`project_tasks`**: Project management for service businesses.
- **`service_orders`**, **`service_visits`**: Field service management (repair shops, service centers).
- **`recruitment_jobs`**, **`recruitment_candidates`**, **`recruitment_stages`**: Applicant tracking system.
- **`benefit_plans`**, **`benefit_enrollments`**, **`benefit_claims`**: Employee benefits administration.
- **`coop_members`**, **`coop_deposits`**, **`coop_loans`**, **`coop_loan_repayments`**: Employee cooperative (Koperasi) module. Loan repayments include unique constraint on `(loan_id, period)` to prevent double-deductions.
- **`ocr_jobs`**: Async OCR processing queue for document scanning.
- **`ai_credits`**, **`ai_queries`**, **`ai_analytics_logs`**: AI module usage tracking.
- **`flow_definitions`**, **`flow_instances`**, **`flow_tasks`**: Visual workflow automation engine.

### System & Platform

- **`org_settings`**: Per-org configuration key-value store.
- **`approval_chains`**, **`approval_steps`**, **`approval_history`**: Configurable multi-step approval workflows.
- **`recurring_templates`**: Recurring task definitions with failure circuit breaker (`failure_count >= max_failures → is_active = false`).
- **`webhook_subscriptions`**, **`webhook_delivery_logs`**: Outbound event webhooks with retry and circuit breaker.
- **`system_audit_logs`**: Immutable mutation audit trail across all modules.
- **`system_incident_logs`**: Incident and security event records for compliance reporting.
- **`notifications`**: In-app notification queue.
- **`media_registry`**: Centralized R2 asset index per org.
- **`custom_field_definitions`**: Runtime schema extension for custom fields without migrations.
- **`data_retention_policies`**: UU PDP and regulatory retention rules per data category.

---

## 4. Cache Plane & Quota Logic

### Quota Enforcement Flow

1. **Read (Fast Path)**: `usageGuard` checks Workers KV for `(org_id, metric, period)` → allow or deny in < 10ms.
2. **Write (Atomic)**: On successful action, execute an optimistic-locked D1 update:
   ```sql
   UPDATE usage_state
   SET current_value = current_value + 1, version = version + 1
   WHERE org_id = ? AND metric = ? AND current_value < limit_value AND version = ?
   ```
3. **Sync**: Background process syncs authoritative D1 counts back to KV and DB_CORE for unified reporting.

---

## 5. ID Standard (Prefix-Based ULID)

All identifiers use `createId("prefix_")` from `@snappy/internal`. Raw `Math.random()` and `crypto.randomUUID()` are **prohibited**.

| Prefix  | Entity             |
| ------- | ------------------ |
| `usr_`  | User               |
| `org_`  | Organization       |
| `ses_`  | Session            |
| `tok_`  | Verification Token |
| `inv_`  | Invoice            |
| `sku_`  | Product            |
| `trx_`  | Transaction        |
| `emp_`  | Employee           |
| `crm_`  | CRM Contact        |
| `doc_`  | Document (Sign)    |
| `ord_`  | Order              |
| `run_`  | Delivery Run       |
| `job_`  | Provision Job      |
| `role_` | Role               |
| `perm_` | Permission         |
| `akey_` | API Key            |

---

_© 2026 PT Snappy Angkasa Media. Proprietary & Confidential._
