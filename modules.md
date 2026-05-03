# SNAPPY Module Catalog

Complete inventory of Business Engine modules — from RAKJAT to ADIKARYA.

---

## 🧠 Intelligence

### BISNAY AI (`bisnay`)

AI-powered business analytics and operational automation.

- **Availability**: All tiers (consumption-based via AI Credits).
- **Key Features**:
  - Natural Language Query — "Show me my top-selling product last week."
  - Automated stock anomaly detection and reorder suggestions.
  - Sales trend forecasting for inventory planning.
  - Conversational daily P&L summaries.

### OCR (`ocr`)

AI-driven data extraction from physical and digital documents.

- **Type**: Premium Add-on.
- **Key Features**:
  - Supplier invoice scanning → auto-create Purchase Orders.
  - Receipt scanning → auto-populate expense claims.
  - Employee KTP and NPWP extraction for HR onboarding.

---

## 🛒 Operations

### POS Lite (`pos-lite`)

Lightweight digital cashier for micro-enterprises.

- **Availability**: RAKJAT tier.
- **Key Features**:
  - Digital receipts via unique URL per transaction (`pos_transactions.public_id`).
  - One-click daily sales summary.
  - Basic stock tracking for up to 50 SKUs.
  - Kasbon (credit sale) tracking per customer.

### POS Pro (`pos`)

Full-scale Point of Sale system for high-volume retailers.

- **Availability**: PERINTIS and above.
- **Key Features**:
  - **Multi-Terminal**: Atomic transaction sync across multiple cashiers.
  - **Kitchen Display System**: Real-time order routing to kitchen stations.
  - **Split Bill & Payment**: Flexible payment handling including cash, QRIS, and e-wallet splits.
  - **Kasbon (Credit Accounts)**: Outstanding credit balances per customer with full debit/credit audit log.
  - **Customer Invoices**: Formal B2B invoice generation for wholesale/reseller transactions.
  - **Appointment & Booking**: Integrated booking calendar for service businesses.

### ERP Lite (`erp-lite`)

Basic stock management for growing businesses.

- **Availability**: PERINTIS and above.
- **Key Features**:
  - Low-stock alerts and automated reorder triggers.
  - Basic supplier directory.
  - Internal stock transfers between locations.

### ERP Pro (`erp`)

Enterprise-grade inventory and procurement management.

- **Availability**: MADJOE and above.
- **Key Features**:
  - **Procurement Engine**: Approval-based Purchase Order workflow with Goods Receipt confirmation.
  - **Multi-Warehouse**: Stock management across multiple physical locations.
  - **FIFO Batch Tracking**: SQL-level integrity constraint prevents negative stock (`erp_inv_qty_check`).
  - **Manufacturing Lite**: Production planning and Work Order management via Bill of Materials (`pos_bom`).
  - **Costing Methods**: FIFO inventory valuation.

### Omni-channel (`omni`)

Unified dashboard for marketplace integrations.

- **Type**: Premium Add-on.
- **Key Features**:
  - Real-time inventory sync with Tokopedia, Shopee, and TikTok Shop.
  - Centralized order management with fulfillment state machine.
  - OAuth-based channel authentication with lifecycle tracking.

### WMS (`wms`)

Advanced Warehouse Management System for complex logistics operations.

- **Type**: Premium Add-on (DJAJA only).
- **Key Features**:
  - **Zone & Bin Management**: Map warehouse storage down to specific rack and bin levels.
  - **Weight & Volume Capacity**: Physical capacity constraints per bin.
  - **Pick Lists**: Optimized picking assignments with requested vs. picked quantity tracking.
  - **Stock Movements**: Bin-to-bin internal transfer audit trail.

### Delivery (`delivery`)

Internal fleet and last-mile delivery management.

- **Type**: Premium Add-on.
- **Key Features**:
  - **Driver Registry**: Manage internal delivery staff with vehicle plate assignment.
  - **Run Management**: Batch delivery assignments per driver per day.
  - **Stop Sequencing**: Ordered delivery stops with status tracking (`pending`, `delivered`, `failed`).
  - **Proof of Delivery**: Photo URL upload and failure reason logging per stop.

---

## 👥 Human Resources

### HRIS (`hris`)

Human Resource Information System.

- **Availability**: OESAHA and above.
- **Key Features**:
  - Employee profile management with document registry (contracts, KTP, NPWP).
  - Geo-tagged attendance logging.
  - Leave request and approval management.
  - Shift schedule and availability management.

### Payroll (`payroll`)

Automated salary disbursement engine.

- **Type**: Premium Add-on (MADJOE and above).
- **Key Features**:
  - Automatic PPh 21, BPJS Kesehatan, and BPJS Ketenagakerjaan computation.
  - Commission tracking integrated with POS performance data.
  - Overtime calculation based on configurable rules.
  - Encrypted digital payslips sent directly to employees.
  - Idempotent reprocess guard — prevents double-deduction on payroll re-runs.

### Benefit (`benefit`)

Employee benefits and insurance administration.

- **Type**: Premium Add-on (DJAJA only).
- **Key Features**:
  - Insurance claim portal for health and life coverage.
  - Flexible allowance management (meal, transport, custom).
  - Enrollment and claim status tracking.

### BE-COOP (`coop`)

Integrated Employee Cooperative (Koperasi Simpan Pinjam).

- **Type**: Premium Add-on (OESAHA and above).
- **Key Features**:
  - Automated monthly savings via payroll deduction.
  - Loan application and approval workflow.
  - Automatic installment repayment via salary, with unique constraint on `(loan_id, period)` to prevent double-deduction.

### Recruitment (`recruitment`)

Applicant Tracking System (ATS).

- **Type**: Premium Add-on (DJAJA only).
- **Key Features**:
  - Configurable hiring pipeline with stage management.
  - Candidate tracking from application through offer.
  - Designed for one-click job board posting integration.

---

## 🏢 Service & Project Management

### Service Management (`service`)

Field service and repair order management.

- **Type**: Premium Add-on.
- **Key Features**:
  - **Service Orders**: Track repair or service jobs from intake to completion.
  - **Site Visits**: Log technician visits with timestamps and outcomes.
  - Designed for service centers, repair shops, and field service businesses.

### Project Management (`project`)

Lightweight project tracking for service businesses.

- **Type**: Premium Add-on.
- **Key Features**:
  - **Projects**: Top-level project containers with status tracking.
  - **Milestones**: Time-bound deliverable checkpoints.
  - **Tasks**: Granular work items assigned to project milestones.

---

## 📈 Sales & Compliance

### CRM (`crm`)

Customer Relationship Management.

- **Availability**: OESAHA and above.
- **Key Features**:
  - Unified contact registry for customers and suppliers.
  - Loyalty point balance tracking with full audit log (`crm_loyalty_logs`).
  - Marketing consent tracking for PDPA/UU PDP compliance.
  - Configurable sales pipelines with deal stage tracking.

### SIGN — TTE (`sign`)

Certified Electronic Signature (Tanda Tangan Elektronik).

- **Availability**: OESAHA and above.
- **Key Features**:
  - Legally binding document approval with atomic token validation.
  - Signature expiry enforcement.
  - Audit trail per document.

### TAX (`tax`)

Automated tax calculation and fiscal compliance.

- **Availability**: MADJOE and above.
- **Key Features**:
  - **e-Faktur Ready**: Automatic VAT serial number generation for PPN.
  - **e-Bupot Unifikasi**: PPh withholding certificate support.
  - **PPh Final (UMKM)**: 0.5% final tax summary reporting.
  - DJP SPT reporting records.

---

## 💰 Finance

### Expense Management (`expense`)

Operational expenditure tracking and reimbursement.

- **Availability**: PERINTIS and above.
- **Key Features**:
  - Approval-based reimbursement workflow.
  - OCR receipt scanning for auto-fill.
  - Configurable expense rate tables for travel and per diem.

### Accounting (`fin`)

Full double-entry general ledger and bank reconciliation.

- **Availability**: MADJOE and above.
- **Key Features**:
  - **Chart of Accounts (COA)**: Hierarchical account structure with multi-entity support.
  - **Multi-Currency**: Exchange rate-aware journal entries with base-currency equivalents.
  - **Fixed Assets**: Asset registry with COA-linked depreciation accounting.
  - **Bank Reconciliation**: Import bank statement transactions and match to journal entries.

### PAY (`pay`)

Integrated payment gateway settlement management.

- **Availability**: All tiers.
- **Standard Rates**:
  - QRIS: 0.7% per transaction.
  - Virtual Account: Rp 4,500 flat.
  - E-Wallet: 1.5% per transaction.
- **Key Features**:
  - Settlement batch tracking per provider.
  - Payout records to registered bank accounts.

---

## ⚙️ Platform

### FLOW (`flow`)

Workflow automation and multi-step approval engine.

- **Type**: Premium Add-on (MADJOE and above).
- **Key Features**:
  - Visual approval chain builder.
  - Position-based escalation (fallback if approver is unavailable).
  - Guards against orphan workflow steps when positions are deactivated.

---

_© 2026 PT Snappy Angkasa Media. Proprietary & Confidential._
