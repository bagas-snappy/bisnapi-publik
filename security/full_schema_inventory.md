# SNAPPY Public Architecture Schema Manifest (v1.0.7)

> [!IMPORTANT]
> This file is GENERATED AUTOMATICALLY. Do not edit manually. Run `npm run sync-docs` to update.

**Public Transparency Edition.** Internal security and compliance tables have been redacted for institutional safety.

---

## 📦 Module: AI.TS

| Constant          | Table Name          | Columns                                                                                                                              |
| :---------------- | :------------------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| `aiCredits`       | `ai_credits`        | `id`, `org_id`, `balance`, `metadata_json`, `updated_at`                                                                             |
| `aiQueries`       | `ai_queries`        | `id`, `org_id`, `user_id`, `module_context`, `prompt_summary`, `tokens_used`, `cost_amount`, `status`, `metadata_json`, `created_at` |
| `aiAnalyticsLogs` | `ai_analytics_logs` | `id`, `org_id`, `insight_type`, `severity`, `content_json`, `is_read`, `metadata_json`, `created_at`                                 |

---

## 📦 Module: BENEFIT.TS

| Constant             | Table Name            | Columns                                                                                                                          |
| :------------------- | :-------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| `benefitPlans`       | `benefit_plans`       | `id`, `entity_id`, `name`, `type`, `provider`, `description`, `metadata_json`, `created_at`                                      |
| `benefitEnrollments` | `benefit_enrollments` | `id`, `employee_id`, `plan_id`, `coverage_amount`, `effective_from`, `effective_to`, `status`, `metadata_json`, `created_at`     |
| `benefitClaims`      | `benefit_claims`      | `id`, `enrollment_id`, `claim_date`, `amount`, `status`, `attachment_url`, `note`, `processed_by`, `metadata_json`, `created_at` |

---

## 📦 Module: COOP.TS

| Constant       | Table Name      | Columns                                                                                                                                       |
| :------------- | :-------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| `coopMembers`  | `coop_members`  | `id`, `employee_id`, `membership_no`, `status`, `joined_at`, `metadata_json`                                                                  |
| `coopDeposits` | `coop_deposits` | `id`, `member_id`, `type`, `amount`, `period`, `deposited_at`, `status`, `metadata_json`                                                      |
| `coopLoans`    | `coop_loans`    | `id`, `member_id`, `amount`, `interest_rate`, `tenor_months`, `monthly_installment`, `status`, `approved_by`, `disbursed_at`, `metadata_json` |

---

## 📦 Module: CORE.TS

| Constant           | Table Name          | Columns                                                                                                                                                                                                                                                                                                                                                                                       |
| :----------------- | :------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `users`            | `users`             | `id`, `email`, `google_id`, `name`, `avatar_url`, `status`, `hashed_password`, `email_verified_at`, `mfa_enabled`, `mfa_secret`, `mfa_secret_pending`, `metadata_json`, `deleted_at`                                                                                                                                                                                                          |
| `permissions`      | `permissions`       | `id`, `resource`, `action`, `description`                                                                                                                                                                                                                                                                                                                                                     |
| `orgs`             | `orgs`              | `id`, `slug`, `name`, `tier`, `status`, `d1_id`, `r2_bucket`, `country_code`, `currency_code`, `timezone`, `regulatory_ids_json`, `metadata_json`, `deleted_at`                                                                                                                                                                                                                               |
| `orgJitDomains`    | `org_jit_domains`   | `id`, `org_id`, `domain`, `default_role_id`, `is_active`                                                                                                                                                                                                                                                                                                                                      |
| `tierConfig`       | `tier_config`       | `tier`, `label`, `monthly_fee_usd`, `monthly_fee_idr`, `annual_fee_idr`, `daily_trx_limit`, `daily_pdf_limit`, `ai_credit_limit`, `branch_limit`, `row_limit`, `email_limit`, `doc_limit`, `max_employees`, `max_products`, `max_warehouses`, `max_suppliers`, `max_monthly_po`, `report_watermark`, `allowed_costing_methods`, `d1_cap_bytes`, `r2_cap_bytes`, `metadata_json`, `updated_at` |
| `invitations`      | `invitations`       | `id`, `org_id`, `email`, `role_id`, `status`, `inviter_id`, `expires_at`, `metadata_json`, `created_at`, `updated_at`                                                                                                                                                                                                                                                                         |
| `legalSignatures`  | `legal_signatures`  | `id`, `document_id`, `user_id`, `org_id`, `ip_address`, `user_agent`, `[REDACTED]`, `signed_at`                                                                                                                                                                                                                                                                                               |
| `invoices`         | `invoices`          | `id`, `agreement_id`, `number`, `client_name`, `client_email`, `total_amount`, `milestone`, `[REDACTED]`, `payment_url`, `status`, `created_at`                                                                                                                                                                                                                                               |
| `adminRoles`       | `admin_roles`       | `id`, `name`, `description`                                                                                                                                                                                                                                                                                                                                                                   |
| `adminPermissions` | `admin_permissions` | `id`, `role_id`, `statement`, `created_at`                                                                                                                                                                                                                                                                                                                                                    |
| `modules`          | `modules`           | `id`, `slug`, `name`, `description`, `icon`, `group_name`, `sort_order`, `is_addon`, `status`, `tier_access_json`, `subscription_fee_idr`, `annual_fee_idr`                                                                                                                                                                                                                                   |
| `systemStats`      | `system_stats`      | `id`, `total_merchants`, `total_users`, `total_transactions`, `version`, `updated_at`                                                                                                                                                                                                                                                                                                         |
| `platformConfig`   | `platform_config`   | `[REDACTED]`, `value`, `label`, `updated_at`                                                                                                                                                                                                                                                                                                                                                  |

---

## 📦 Module: CRM.TS

| Constant            | Table Name            | Columns                                                                                                                                                               |
| :------------------ | :-------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `crmContacts`       | `crm_contacts`        | `id`, `entity_id`, `name`, `email`, `phone`, `address_structured_json`, `type`, `points_balance`, `marketing_consent_at`, `consent_source`, `tax_id`, `metadata_json` |
| `crmLoyaltyLogs`    | `crm_loyalty_logs`    | `id`, `contact_id`, `action`, `points`, `balance_after`, `reference_type`, `reference_id`, `created_at`                                                               |
| `crmPipelines`      | `crm_pipelines`       | `id`, `name`, `is_active`                                                                                                                                             |
| `crmPipelineStages` | `crm_pipeline_stages` | `id`, `pipeline_id`, `name`, `sort_order`                                                                                                                             |
| `crmDeals`          | `crm_deals`           | `id`, `contact_id`, `stage_id`, `title`, `value`, `currency_code`, `status`, `closed_at`, `metadata_json`                                                             |

---

## 📦 Module: DELIVERY.TS

| Constant          | Table Name         | Columns                                                                                                                             |
| :---------------- | :----------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `deliveryDrivers` | `delivery_drivers` | `id`, `entity_id`, `employee_id`, `name`, `phone`, `vehicle_plate`, `is_active`, `metadata_json`, `created_at`                      |
| `deliveryRuns`    | `delivery_runs`    | `id`, `entity_id`, `driver_id`, `status`, `started_at`, `completed_at`, `metadata_json`, `created_at`                               |
| `deliveryStops`   | `delivery_stops`   | `id`, `run_id`, `transaction_id`, `address_json`, `sequence`, `status`, `proof_url`, `failed_reason`, `metadata_json`, `created_at` |

---

## 📦 Module: ERP.TS

| Constant                | Table Name                 | Columns                                                                                                                                                                  |
| :---------------------- | :------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `erpWarehouses`         | `erp_warehouses`           | `id`, `entity_id`, `parent_id`, `path`, `name`, `location`, `type`, `is_main`, `is_active`, `metadata_json`                                                              |
| `erpSuppliers`          | `erp_suppliers`            | `id`, `entity_id`, `name`, `contact_name`, `email`, `phone`, `address_structured_json`, `tax_id`, `metadata_json`                                                        |
| `erpInventoryLogs`      | `erp_inventory_logs`       | `id`, `product_id`, `warehouse_id`, `batch_id`, `type`, `quantity`, `[REDACTED]`, `indirect_costs_json`, `reference_type`, `reference_id`, `created_at`                  |
| `erpPurchaseOrders`     | `erp_purchase_orders`      | `id`, `entity_id`, `supplier_id`, `status`, `subtotal`, `tax_amount`, `total_amount`, `currency_code`, `payment_term_days`, `ordered_at`, `approved_by`, `metadata_json` |
| `erpPurchaseOrderItems` | `erp_purchase_order_items` | `id`, `po_id`, `product_id`, `quantity`, `unit_price`, `tax_code`, `tax_amount`, `received_qty`, `created_at`                                                            |
| `erpGoodsReceipts`      | `erp_goods_receipts`       | `id`, `po_id`, `warehouse_id`, `received_by`, `received_at`, `note`, `metadata_json`                                                                                     |
| `erpGoodsReceiptItems`  | `erp_goods_receipt_items`  | `id`, `receipt_id`, `po_item_id`, `batch_id`, `quantity_received`, `landed_cost_voucher_amount`, `note`, `created_at`                                                    |
| `erpProductionPlans`    | `erp_production_plans`     | `id`, `entity_id`, `product_id`, `planned_quantity`, `start_date`, `end_date`, `status`, `metadata_json`                                                                 |
| `erpWorkOrders`         | `erp_work_orders`          | `id`, `plan_id`, `product_id`, `quantity`, `status`, `started_at`, `completed_at`, `metadata_json`                                                                       |
| `erpBoms`               | `erp_boms`                 | `id`, `entity_id`, `product_id`, `name`, `is_active`, `is_default`, `quantity`, `process_loss_percentage`, `total_cost`, `metadata_json`                                 |
| `erpBomItems`           | `erp_bom_items`            | `id`, `bom_id`, `product_id`, `quantity`, `uom`, `scrap_percentage`, `metadata_json`                                                                                     |

---

## 📦 Module: FIN.TS

| Constant              | Table Name             | Columns                                                                                                                                             |
| :-------------------- | :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| `finEntities`         | `fin_entities`         | `id`, `name`, `code`, `country_code`, `currency_code`, `timezone`, `is_default`, `metadata_json`                                                    |
| `finAccounts`         | `fin_accounts`         | `id`, `entity_id`, `code`, `name`, `type`, `parent_account_id`, `currency_code`, `is_system`, `is_active`, `metadata_json`                          |
| `finCurrencies`       | `fin_currencies`       | `code`, `name`, `symbol`, `is_base`, `is_active`                                                                                                    |
| `finJournals`         | `fin_journals`         | `id`, `entity_id`, `journal_no`, `type`, `date`, `status`, `description`, `created_by`, `metadata_json`                                             |
| `finLedgers`          | `fin_ledgers`          | `id`, `journal_id`, `account_id`, `debit`, `credit`, `debit_base`, `credit_base`, `currency_code`, `exchange_rate`, `balance`, `note`, `created_at` |
| `finFixedAssets`      | `fin_fixed_assets`     | `id`, `entity_id`, `name`, `purchase_date`, `purchase_amount`, `salvage_value`, `coa_id`, `metadata_json`                                           |
| `bankAccounts`        | `bank_accounts`        | `id`, `entity_id`, `bank_name`, `[REDACTED]`, `account_holder`, `currency_code`, `coa_id`, `metadata_json`                                          |
| `bankTransactions`    | `bank_transactions`    | `id`, `bank_account_id`, `date`, `description`, `reference_no`, `amount`, `import_batch_id`, `raw_data`, `is_reconciled`, `created_at`              |
| `bankReconciliations` | `bank_reconciliations` | `id`, `bank_transaction_id`, `reconciled_by`, `reconciled_at`                                                                                       |
| `expenseClaims`       | `expense_claims`       | `id`, `entity_id`, `employee_id`, `title`, `total_amount`, `status`, `approved_by`, `payment_journal_id`, `metadata_json`                           |
| `expenseItems`        | `expense_items`        | `id`, `claim_id`, `category`, `amount`, `description`, `attachment_url`, `date`, `created_at`                                                       |
| `expenseRates`        | `expense_rates`        | `id`, `entity_id`, `name`, `rate`, `unit`, `metadata_json`                                                                                          |

---

## 📦 Module: FLOW.TS

| Constant          | Table Name         | Columns                                                                                                                                                          |
| :---------------- | :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `flowDefinitions` | `flow_definitions` | `id`, `entity_id`, `name`, `trigger_type`, `diagram_json`, `is_active`, `metadata_json`, `created_at`                                                            |
| `flowInstances`   | `flow_instances`   | `id`, `definition_id`, `entity_type`, `entity_id`, `current_step`, `status`, `metadata_json`, `started_at`, `completed_at`                                       |
| `flowTasks`       | `flow_tasks`       | `id`, `instance_id`, `task_name`, `priority`, `assignee_id`, `status`, `due_at`, `sla_breach_at`, `completed_at`, `feedback_json`, `metadata_json`, `created_at` |

---

## 📦 Module: HR.TS

| Constant              | Table Name              | Columns                                                                                                                                                                                                                                             |
| :-------------------- | :---------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hrEmployees`         | `hr_employees`          | `id`, `user_id`, `entity_id`, `name`, `email`, `phone`, `address_structured_json`, `position_id`, `status`, `npwp`, `ptkp_status`, `bpjs_kesehatan_no`, `bpjs_tk_no`, `is_bpjs_kesehatan_active`, `is_bpjs_tk_active`, `joined_at`, `metadata_json` |
| `hrSalaries`          | `hr_salaries`           | `id`, `employee_id`, `base_salary`, `effective_from`, `effective_to`, `created_at`                                                                                                                                                                  |
| `hrPositions`         | `hr_positions`          | `id`, `name`, `department`, `metadata_json`                                                                                                                                                                                                         |
| `hrAttendance`        | `hr_attendance`         | `id`, `employee_id`, `shift_id`, `check_in`, `check_out`, `location_json`, `status`, `created_at`                                                                                                                                                   |
| `hrShifts`            | `hr_shifts`             | `id`, `name`, `start_time`, `end_time`, `work_days`, `metadata_json`                                                                                                                                                                                |
| `hrLeaves`            | `hr_leaves`             | `id`, `employee_id`, `type`, `start_date`, `end_date`, `status`, `approved_by`, `metadata_json`                                                                                                                                                     |
| `hrCommissions`       | `hr_commissions`        | `id`, `employee_id`, `transaction_id`, `type`, `amount`, `status`, `created_at`                                                                                                                                                                     |
| `hrOvertimeRules`     | `hr_overtime_rules`     | `id`, `name`, `multiplier`, `condition`                                                                                                                                                                                                             |
| `hrPayrollRuns`       | `hr_payroll_runs`       | `id`, `entity_id`, `period`, `total_amount`, `status`, `executed_at`, `metadata_json`                                                                                                                                                               |
| `hrPayrollItems`      | `hr_payroll_items`      | `id`, `payroll_run_id`, `employee_id`, `type`, `name`, `amount`, `is_deduction`, `is_auto_calculated`, `created_at`                                                                                                                                 |
| `hrAssets`            | `hr_assets`             | `id`, `name`, `serial_number`, `assigned_to`, `status`, `metadata_json`                                                                                                                                                                             |
| `hrEmployeeDocuments` | `hr_employee_documents` | `id`, `employee_id`, `type`, `title`, `file_url`, `is_visible_to_employee`                                                                                                                                                                          |

---

## 📦 Module: OCR.TS

| Constant  | Table Name | Columns                                                                                                                                                                       |
| :-------- | :--------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ocrJobs` | `ocr_jobs` | `id`, `entity_id`, `media_id`, `target_module`, `target_table`, `extracted_json`, `confidence_score`, `status`, `error_message`, `verified_by`, `metadata_json`, `created_at` |

---

## 📦 Module: OMNI.TS

| Constant              | Table Name              | Columns                                                                                                                                                      |
| :-------------------- | :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `omniStores`          | `omni_stores`           | `id`, `entity_id`, `platform`, `name`, `shop_id`, `[REDACTED]`, `[REDACTED]`, `[REDACTED]`, `token_expires_at`, `status`, `last_error_json`, `metadata_json` |
| `omniProductMappings` | `omni_product_mappings` | `id`, `store_id`, `local_product_id`, `platform_product_id`, `platform_sku`, `is_synced`, `metadata_json`                                                    |
| `omniOrders`          | `omni_orders`           | `id`, `store_id`, `platform_order_no`, `status`, `total_amount`, `raw_json`, `synced_at`, `metadata_json`                                                    |
| `omniOrderItems`      | `omni_order_items`      | `id`, `order_id`, `local_product_id`, `quantity`, `unit_price`, `platform_discount_amount`                                                                   |

---

## 📦 Module: PAY.TS

| Constant     | Table Name    | Columns                                                                                                                      |
| :----------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------- |
| `payPayouts` | `pay_payouts` | `id`, `bank_account_id`, `settlement_batch_id`, `amount`, `currency_code`, `status`, `reference`, `settled_at`, `created_at` |

---

## 📦 Module: POS.TS

| Constant                 | Table Name                  | Columns                                                                                                                                                                                                                                                                                                                   |
| :----------------------- | :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `posCategories`          | `pos_categories`            | `id`, `name`, `parent_category_id`, `sort_order`, `metadata_json`                                                                                                                                                                                                                                                         |
| `posKitchenStations`     | `pos_kitchen_stations`      | `id`, `entity_id`, `name`, `printer_ip`, `is_active`, `metadata_json`                                                                                                                                                                                                                                                     |
| `posTerminals`           | `pos_terminals`             | `id`, `entity_id`, `name`, `device_id`, `is_active`, `metadata_json`                                                                                                                                                                                                                                                      |
| `posProducts`            | `pos_products`              | `id`, `category_id`, `kitchen_station_id`, `name`, `sku`, `barcode`, `description`, `price`, `unit`, `image_url`, `is_subscription`, `billing_interval_json`, `uom_conversions_json`, `costing_method`, `track_inventory`, `is_variant_parent`, `parent_product_id`, `attributes_json`, `is_active`, `metadata_json`      |
| `posSerialStatuses`      | `pos_serial_statuses`       | `id`, `label`, `description`                                                                                                                                                                                                                                                                                              |
| `posProductSerials`      | `pos_product_serials`       | `id`, `product_id`, `serial_number`, `status_id`, `sale_transaction_id`, `metadata_json`                                                                                                                                                                                                                                  |
| `posPriceLists`          | `pos_price_lists`           | `id`, `name`, `product_id`, `type`, `min_quantity`, `price`, `currency_code`, `is_active`, `metadata_json`                                                                                                                                                                                                                |
| `posBom`                 | `pos_bom`                   | `id`, `finished_product_id`, `material_product_id`, `quantity`, `unit`, `metadata_json`                                                                                                                                                                                                                                   |
| `posModifiers`           | `pos_modifiers`             | `id`, `name`, `price`, `is_optional`, `metadata_json`                                                                                                                                                                                                                                                                     |
| `posProductModifiers`    | `pos_product_modifiers`     | `product_id`, `modifier_id`                                                                                                                                                                                                                                                                                               |
| `posTransactions`        | `pos_transactions`          | `id`, `public_id`, `invoice_no`, `entity_id`, `shift_id`, `type`, `status`, `is_fully_returned`, `subtotal`, `discount`, `tax_amount`, `total_amount`, `paid_amount`, `currency_code`, `customer_id`, `cashier_id`, `terminal_id`, `table_id`, `receipt_url`, `payment_link_url`, `note`, `completed_at`, `metadata_json` |
| `posCreditAccounts`      | `pos_credit_accounts`       | `id`, `entity_id`, `customer_id`, `credit_limit`, `balance`, `version`, `status`, `metadata_json`                                                                                                                                                                                                                         |
| `posAppointments`        | `pos_appointments`          | `id`, `entity_id`, `customer_id`, `product_id`, `staff_id`, `start_at`, `end_at`, `duration_minutes`, `status`, `metadata_json`                                                                                                                                                                                           |
| `posStaffAvailability`   | `pos_staff_availability`    | `id`, `staff_id`, `day_of_week`, `start_time`, `end_time`, `is_active`, `metadata_json`                                                                                                                                                                                                                                   |
| `posStaffOverrides`      | `pos_staff_overrides`       | `id`, `staff_id`, `override_date`, `is_off`, `start_time`, `end_time`, `reason`                                                                                                                                                                                                                                           |
| `posCustomerInvoices`    | `pos_customer_invoices`     | `id`, `transaction_id`, `invoice_number`, `total_amount`, `due_at`, `status`, `pdf_url`, `metadata_json`                                                                                                                                                                                                                  |
| `posTransactionTaxLines` | `pos_transaction_tax_lines` | `id`, `transaction_id`, `tax_code`, `rate`, `base_amount`, `tax_amount`, `created_at`                                                                                                                                                                                                                                     |
| `posTransactionPayments` | `pos_transaction_payments`  | `id`, `transaction_id`, `method_id`, `amount`, `reference`, `settlement_status`, `settlement_batch_id`, `created_at`                                                                                                                                                                                                      |
| `posTransactionItems`    | `pos_transaction_items`     | `id`, `transaction_id`, `product_id`, `product_name`, `quantity`, `price`, `discount_amount`, `cost`, `cost_base`, `subtotal`, `created_at`                                                                                                                                                                               |
| `posShifts`              | `pos_shifts`                | `id`, `opened_by`, `opening_cash`, `closed_at`, `closing_cash`, `status`, `metadata_json`                                                                                                                                                                                                                                 |
| `posTables`              | `pos_tables`                | `id`, `number`, `capacity`, `status`, `metadata_json`                                                                                                                                                                                                                                                                     |
| `posKitchenQueue`        | `pos_kitchen_queue`         | `id`, `transaction_id`, `item_id`, `status`, `note`, `updated_at`                                                                                                                                                                                                                                                         |
| `posDiscounts`           | `pos_discounts`             | `id`, `entity_id`, `name`, `type`, `value`, `code`, `max_usage`, `used_count`, `valid_from`, `valid_until`, `is_active`, `metadata_json`                                                                                                                                                                                  |
| `posPricingRules`        | `pos_pricing_rules`         | `id`, `entity_id`, `name`, `priority`, `apply_on`, `price_or_product_discount`, `rate_or_discount`, `discount_percentage`, `discount_amount`, `rate`, `min_qty`, `max_qty`, `min_amount`, `max_amount`, `valid_from`, `valid_until`, `is_active`, `metadata_json`                                                         |
| `posPricingRuleItems`    | `pos_pricing_rule_items`    | `id`, `rule_id`, `target_id`, `created_at`                                                                                                                                                                                                                                                                                |

---

## 📦 Module: PROJECT.TS

| Constant            | Table Name           | Columns                                                                                                                                                                   |
| :------------------ | :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `projectProjects`   | `project_projects`   | `id`, `entity_id`, `name`, `customer_id`, `budget`, `status`, `start_date`, `end_date`, `metadata_json`, `created_at`                                                     |
| `projectMilestones` | `project_milestones` | `id`, `project_id`, `name`, `due_date`, `status`, `metadata_json`, `created_at`                                                                                           |
| `projectTasks`      | `project_tasks`      | `id`, `project_id`, `milestone_id`, `assignee_id`, `title`, `description`, `priority`, `status`, `due_date`, `is_billable`, `actual_hours`, `metadata_json`, `created_at` |

---

## 📦 Module: RECRUITMENT.TS

| Constant                | Table Name               | Columns                                                                                                                             |
| :---------------------- | :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `recruitmentJobs`       | `recruitment_jobs`       | `id`, `entity_id`, `position_id`, `title`, `description`, `requirements_json`, `status`, `posted_at`, `metadata_json`, `created_at` |
| `recruitmentCandidates` | `recruitment_candidates` | `id`, `job_id`, `name`, `email`, `phone`, `resume_url`, `source`, `status`, `metadata_json`, `created_at`                           |
| `recruitmentStages`     | `recruitment_stages`     | `id`, `candidate_id`, `stage_name`, `feedback`, `rating`, `scheduled_at`, `created_by`, `metadata_json`, `created_at`               |

---

## 📦 Module: SERVICE.TS

| Constant        | Table Name       | Columns                                                                                                                                                                   |
| :-------------- | :--------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `serviceOrders` | `service_orders` | `id`, `entity_id`, `customer_id`, `asset_id`, `issue_description`, `status`, `priority`, `metadata_json`, `created_at`                                                    |
| `serviceVisits` | `service_visits` | `id`, `order_id`, `technician_id`, `scheduled_at`, `arrival_time`, `completion_time`, `service_report`, `parts_used_json`, `signature_url`, `metadata_json`, `created_at` |

---

## 📦 Module: SIGN.TS

| Constant        | Table Name       | Columns                                                                                 |
| :-------------- | :--------------- | :-------------------------------------------------------------------------------------- |
| `signDocuments` | `sign_documents` | `id`, `title`, `version_no`, `file_url`, `replaces_document_id`, `status`, `created_at` |
| `signStamps`    | `sign_stamps`    | `id`, `request_id`, `stamp_provider`, `serial_number`, `cost_amount`, `applied_at`      |

---

## 📦 Module: SYSTEM.TS

| Constant                 | Table Name                 | Columns                                                                                                                                                          |
| :----------------------- | :------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `orgSettings`            | `org_settings`             | `id`, `org_id`, `[REDACTED]`, `value`, `updated_at`                                                                                                              |
| `usageState`             | `usage_state`              | `id`, `org_id`, `metric`, `current_value`, `limit_value`, `version`, `updated_at`                                                                                |
| `billingCycles`          | `billing_cycles`           | `id`, `org_id`, `period_start`, `period_end`, `status`, `total_billed`, `metadata_json`                                                                          |
| `activationGates`        | `activation_gates`         | `id`, `org_id`, `module_slug`, `is_enabled`, `metadata_json`                                                                                                     |
| `topupLedger`            | `topup_ledger`             | `id`, `org_id`, `amount`, `type`, `reference_id`, `expires_at`, `metadata_json`                                                                                  |
| `approvalHistory`        | `approval_history`         | `id`, `entity_type`, `entity_id`, `step_id`, `approver_id`, `status`, `note`, `created_at`                                                                       |
| `recurringTemplates`     | `recurring_templates`      | `id`, `entity_id`, `entity_type`, `template_data_json`, `frequency`, `next_run_at`, `last_run_at`, `failure_count`, `max_failures`, `is_active`, `metadata_json` |
| `dataRetentionPolicies`  | `data_retention_policies`  | `id`, `data_category`, `applies_to_table`, `retention_period_months`, `jurisdiction_code`, `created_at`                                                          |
| `customFieldDefinitions` | `custom_field_definitions` | `id`, `entity_type`, `name`, `field_type`, `is_required`, `options_json`, `created_at`                                                                           |
| `systemAuditLogs`        | `system_audit_logs`        | `id`, `user_id`, `action`, `entity_type`, `entity_id`, `old_data_json`, `new_data_json`, `ip_address`, `user_agent`, `created_at`                                |
| `sysBackups`             | `sys_backups`              | `id`, `backup_type`, `r2_key`, `file_size_bytes`, `status`, `created_at`                                                                                         |
| `notifications`          | `notifications`            | `id`, `user_id`, `title`, `content`, `is_read`, `created_at`                                                                                                     |
| `webhookSubscriptions`   | `webhook_subscriptions`    | `id`, `url`, `event`, `[REDACTED]`, `is_active`, `metadata_json`                                                                                                 |
| `webhookDeliveryLogs`    | `webhook_delivery_logs`    | `id`, `subscription_id`, `status_code`, `request_body`, `retry_count`, `max_retries`, `delivered_at`, `created_at`                                               |
| `mediaRegistry`          | `media_registry`           | `id`, `file_name`, `file_type`, `r2_key`, `owner_id`, `metadata_json`                                                                                            |
| `systemIncidentLogs`     | `system_incident_logs`     | `id`, `action_category`, `severity`, `description`, `resolved_at`, `resolved_by`, `metadata_json`                                                                |

---

## 📦 Module: TAX.TS

| Constant           | Table Name           | Columns                                                                                                                     |
| :----------------- | :------------------- | :-------------------------------------------------------------------------------------------------------------------------- |
| `taxConfigs`       | `tax_configs`        | `id`, `entity_id`, `name`, `rate`, `type`, `is_active`, `metadata_json`                                                     |
| `taxUmkmSummaries` | `tax_umkm_summaries` | `id`, `entity_id`, `period`, `total_revenue`, `tax_rate`, `tax_payable`, `billing_code`, `ntpn`, `is_paid`, `metadata_json` |
| `taxInvoices`      | `tax_invoices`       | `id`, `entity_id`, `transaction_reference`, `tax_invoice_no`, `status`, `metadata_json`                                     |
| `taxDjpReports`    | `tax_djp_reports`    | `id`, `entity_id`, `type`, `period`, `status`, `receipt_no`, `metadata_json`                                                |

---

## 📦 Module: WMS.TS

| Constant            | Table Name            | Columns                                                                                                                   |
| :------------------ | :-------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| `wmsZones`          | `wms_zones`           | `id`, `warehouse_id`, `name`, `description`, `type`                                                                       |
| `wmsBins`           | `wms_bins`            | `id`, `zone_id`, `name`, `barcode`, `max_weight`, `max_volume`, `is_active`                                               |
| `wmsPickLists`      | `wms_pick_lists`      | `id`, `entity_id`, `order_reference`, `status`, `assigned_to`                                                             |
| `wmsPickItems`      | `wms_pick_items`      | `id`, `pick_list_id`, `product_id`, `batch_id`, `bin_id`, `requested_quantity`, `picked_quantity`, `status`, `created_at` |
| `wmsStockMovements` | `wms_stock_movements` | `id`, `entity_id`, `product_id`, `from_bin_id`, `to_bin_id`, `quantity`, `reason`, `created_at`                           |

---
