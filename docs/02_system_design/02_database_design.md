# FinPilot - Database Design

## 1. Purpose

This document defines the logical database design for the FinPilot platform, including core entities, attributes, primary keys, foreign keys and relationships.

The database is designed to support customer and account management, data ingestion, migration, validation, reconciliation, exception management, governance, lineage, RBAC and auditability.

---

## 2. Database Technology

Initial database:

* PostgreSQL

The database shall support:

* Relational data storage
* Primary and foreign keys
* Referential integrity
* Constraints
* Indexing
* Transactions
* Audit fields

---

## 3. Customer Table

### Table: `customers`

| Column           | Data Type    | Constraint       | Description                  |
| ---------------- | ------------ | ---------------- | ---------------------------- |
| customer_id      | BIGSERIAL    | PK               | Unique customer identifier   |
| customer_number  | VARCHAR(50)  | UNIQUE, NOT NULL | Business customer identifier |
| first_name       | VARCHAR(100) | NOT NULL         | Customer first name          |
| last_name        | VARCHAR(100) | NOT NULL         | Customer last name           |
| date_of_birth    | DATE         | NULL             | Customer date of birth       |
| email            | VARCHAR(255) | NULL             | Customer email               |
| phone            | VARCHAR(30)  | NULL             | Customer phone               |
| address_line1    | VARCHAR(255) | NULL             | Address                      |
| city             | VARCHAR(100) | NULL             | City                         |
| state            | VARCHAR(100) | NULL             | State                        |
| country_code     | CHAR(2)      | NULL             | Country code                 |
| customer_status  | VARCHAR(30)  | NOT NULL         | Active, Inactive, Suspended  |
| source_system_id | BIGINT       | FK               | Source system                |
| created_at       | TIMESTAMP    | NOT NULL         | Creation timestamp           |
| created_by       | VARCHAR(100) | NOT NULL         | Creator                      |
| updated_at       | TIMESTAMP    | NULL             | Last update                  |
| updated_by       | VARCHAR(100) | NULL             | Last updater                 |

---

## 4. Account Table

### Table: `accounts`

| Column           | Data Type     | Constraint       | Description                 |
| ---------------- | ------------- | ---------------- | --------------------------- |
| account_id       | BIGSERIAL     | PK               | Unique account identifier   |
| account_number   | VARCHAR(50)   | UNIQUE, NOT NULL | Business account identifier |
| customer_id      | BIGINT        | FK, NOT NULL     | Account owner               |
| account_type     | VARCHAR(30)   | NOT NULL         | Savings, Current, etc.      |
| currency_code    | CHAR(3)       | NOT NULL         | Currency                    |
| balance          | NUMERIC(18,2) | NOT NULL         | Account balance             |
| account_status   | VARCHAR(30)   | NOT NULL         | Active, Closed, Suspended   |
| open_date        | DATE          | NOT NULL         | Account opening date        |
| close_date       | DATE          | NULL             | Account closing date        |
| source_system_id | BIGINT        | FK               | Source system               |
| created_at       | TIMESTAMP     | NOT NULL         | Creation timestamp          |
| created_by       | VARCHAR(100)  | NOT NULL         | Creator                     |
| updated_at       | TIMESTAMP     | NULL             | Last update                 |
| updated_by       | VARCHAR(100)  | NULL             | Last updater                |

---

## 5. Source System Table

### Table: `source_systems`

| Column           | Data Type    | Constraint       | Description                 |
| ---------------- | ------------ | ---------------- | --------------------------- |
| source_system_id | BIGSERIAL    | PK               | Unique source system ID     |
| system_code      | VARCHAR(50)  | UNIQUE, NOT NULL | System code                 |
| system_name      | VARCHAR(150) | NOT NULL         | System name                 |
| system_type      | VARCHAR(50)  | NOT NULL         | Legacy, API, Database, File |
| description      | TEXT         | NULL             | System description          |
| owner            | VARCHAR(100) | NULL             | System owner                |
| status           | VARCHAR(30)  | NOT NULL         | Active/Inactive             |
| created_at       | TIMESTAMP    | NOT NULL         | Creation timestamp          |

---

## 6. Dataset Table

### Table: `datasets`

| Column              | Data Type    | Constraint | Description                                |
| ------------------- | ------------ | ---------- | ------------------------------------------ |
| dataset_id          | BIGSERIAL    | PK         | Unique dataset ID                          |
| dataset_name        | VARCHAR(150) | NOT NULL   | Dataset name                               |
| dataset_type        | VARCHAR(50)  | NOT NULL   | Customer, Account, Transaction, etc.       |
| source_system_id    | BIGINT       | FK         | Source system                              |
| target_dataset_name | VARCHAR(150) | NULL       | Target dataset                             |
| data_owner          | VARCHAR(100) | NULL       | Business data owner                        |
| classification      | VARCHAR(50)  | NULL       | Public, Internal, Confidential, Restricted |
| status              | VARCHAR(30)  | NOT NULL   | Active/Inactive                            |
| created_at          | TIMESTAMP    | NOT NULL   | Creation timestamp                         |

---

## 7. Migration Batch Table

### Table: `migration_batches`

| Column                 | Data Type    | Constraint       | Description                                                    |
| ---------------------- | ------------ | ---------------- | -------------------------------------------------------------- |
| batch_id               | BIGSERIAL    | PK               | Unique batch                                                   |
| batch_number           | VARCHAR(50)  | UNIQUE, NOT NULL | Business batch identifier                                      |
| dataset_id             | BIGINT       | FK, NOT NULL     | Dataset being processed                                        |
| source_system_id       | BIGINT       | FK, NOT NULL     | Source system                                                  |
| source_record_count    | BIGINT       | NULL             | Source count                                                   |
| target_record_count    | BIGINT       | NULL             | Target count                                                   |
| processed_record_count | BIGINT       | NULL             | Processed records                                              |
| error_record_count     | BIGINT       | NULL             | Failed records                                                 |
| batch_status           | VARCHAR(30)  | NOT NULL         | Created/In Progress/Validation/Reconciliation/Completed/Failed |
| started_at             | TIMESTAMP    | NULL             | Processing start                                               |
| completed_at           | TIMESTAMP    | NULL             | Processing completion                                          |
| created_by             | VARCHAR(100) | NOT NULL         | Batch creator                                                  |
| created_at             | TIMESTAMP    | NOT NULL         | Creation timestamp                                             |

---

## 8. Validation Rule Table

### Table: `validation_rules`

| Column          | Data Type    | Constraint       | Description                                  |
| --------------- | ------------ | ---------------- | -------------------------------------------- |
| rule_id         | BIGSERIAL    | PK               | Unique validation rule                       |
| rule_code       | VARCHAR(50)  | UNIQUE, NOT NULL | Rule identifier                              |
| rule_name       | VARCHAR(150) | NOT NULL         | Rule name                                    |
| rule_type       | VARCHAR(50)  | NOT NULL         | Mandatory, Duplicate, Format, Business, etc. |
| dataset_id      | BIGINT       | FK               | Dataset                                      |
| rule_definition | TEXT         | NOT NULL         | Rule logic                                   |
| severity        | VARCHAR(20)  | NOT NULL         | Low, Medium, High, Critical                  |
| active_flag     | BOOLEAN      | NOT NULL         | Rule enabled/disabled                        |
| created_at      | TIMESTAMP    | NOT NULL         | Creation timestamp                           |
| created_by      | VARCHAR(100) | NOT NULL         | Creator                                      |

---

## 9. Validation Result Table

### Table: `validation_results`

| Column               | Data Type    | Constraint   | Description           |
| -------------------- | ------------ | ------------ | --------------------- |
| validation_result_id | BIGSERIAL    | PK           | Validation result ID  |
| batch_id             | BIGINT       | FK, NOT NULL | Migration batch       |
| rule_id              | BIGINT       | FK, NOT NULL | Validation rule       |
| record_reference     | VARCHAR(100) | NULL         | Affected record       |
| result_status        | VARCHAR(20)  | NOT NULL     | Passed/Failed/Warning |
| error_message        | TEXT         | NULL         | Validation error      |
| source_value         | TEXT         | NULL         | Source value          |
| target_value         | TEXT         | NULL         | Target value          |
| executed_at          | TIMESTAMP    | NOT NULL     | Execution time        |

---

## 10. Reconciliation Result Table

### Table: `reconciliation_results`

| Column                | Data Type     | Constraint   | Description            |
| --------------------- | ------------- | ------------ | ---------------------- |
| reconciliation_id     | BIGSERIAL     | PK           | Reconciliation ID      |
| batch_id              | BIGINT        | FK, NOT NULL | Migration batch        |
| source_count          | BIGINT        | NOT NULL     | Source record count    |
| target_count          | BIGINT        | NOT NULL     | Target record count    |
| matching_count        | BIGINT        | NULL         | Matching records       |
| missing_count         | BIGINT        | NULL         | Missing records        |
| additional_count      | BIGINT        | NULL         | Additional records     |
| mismatch_count        | BIGINT        | NULL         | Mismatched records     |
| source_amount         | NUMERIC(20,2) | NULL         | Source financial total |
| target_amount         | NUMERIC(20,2) | NULL         | Target financial total |
| amount_difference     | NUMERIC(20,2) | NULL         | Difference             |
| reconciliation_status | VARCHAR(20)   | NOT NULL     | Passed/Failed/Warning  |
| executed_at           | TIMESTAMP     | NOT NULL     | Execution time         |

---

## 11. Exception Table

### Table: `exceptions`

| Column               | Data Type    | Constraint       | Description                              |
| -------------------- | ------------ | ---------------- | ---------------------------------------- |
| exception_id         | BIGSERIAL    | PK               | Unique exception                         |
| exception_number     | VARCHAR(50)  | UNIQUE, NOT NULL | Business exception ID                    |
| batch_id             | BIGINT       | FK               | Migration batch                          |
| validation_result_id | BIGINT       | FK               | Related validation                       |
| reconciliation_id    | BIGINT       | FK               | Related reconciliation                   |
| record_reference     | VARCHAR(100) | NULL             | Affected record                          |
| exception_type       | VARCHAR(50)  | NOT NULL         | Data Quality, Reconciliation, Processing |
| severity             | VARCHAR(20)  | NOT NULL         | Low, Medium, High, Critical              |
| description          | TEXT         | NOT NULL         | Exception description                    |
| status               | VARCHAR(30)  | NOT NULL         | Open/In Progress/Resolved/Closed         |
| assigned_to          | BIGINT       | FK               | Assigned user                            |
| resolution_comment   | TEXT         | NULL             | Resolution details                       |
| created_at           | TIMESTAMP    | NOT NULL         | Creation time                            |
| resolved_at          | TIMESTAMP    | NULL             | Resolution time                          |

---

## 12. Data Lineage Table

### Table: `data_lineage`

| Column              | Data Type    | Constraint | Description             |
| ------------------- | ------------ | ---------- | ----------------------- |
| lineage_id          | BIGSERIAL    | PK         | Lineage identifier      |
| dataset_id          | BIGINT       | FK         | Dataset                 |
| source_system_id    | BIGINT       | FK         | Source system           |
| source_field        | VARCHAR(150) | NULL       | Source field            |
| transformation_rule | TEXT         | NULL       | Transformation applied  |
| target_dataset      | VARCHAR(150) | NULL       | Target dataset          |
| target_field        | VARCHAR(150) | NULL       | Target field            |
| consumption_layer   | VARCHAR(100) | NULL       | Reporting/API/Analytics |
| created_at          | TIMESTAMP    | NOT NULL   | Creation time           |

---

## 13. Data Governance Metadata Table

### Table: `data_governance_metadata`

| Column              | Data Type    | Constraint | Description           |
| ------------------- | ------------ | ---------- | --------------------- |
| metadata_id         | BIGSERIAL    | PK         | Metadata identifier   |
| dataset_id          | BIGINT       | FK         | Dataset               |
| field_name          | VARCHAR(150) | NOT NULL   | Field name            |
| business_definition | TEXT         | NULL       | Business definition   |
| data_type           | VARCHAR(50)  | NULL       | Data type             |
| data_owner          | VARCHAR(100) | NULL       | Data owner            |
| classification      | VARCHAR(50)  | NULL       | Data classification   |
| retention_period    | VARCHAR(50)  | NULL       | Retention requirement |
| created_at          | TIMESTAMP    | NOT NULL   | Creation time         |
| updated_at          | TIMESTAMP    | NULL       | Last update           |

---

## 14. User Table

### Table: `users`

| Column     | Data Type    | Constraint       | Description     |
| ---------- | ------------ | ---------------- | --------------- |
| user_id    | BIGSERIAL    | PK               | User identifier |
| username   | VARCHAR(100) | UNIQUE, NOT NULL | Login username  |
| email      | VARCHAR(255) | UNIQUE, NOT NULL | User email      |
| full_name  | VARCHAR(150) | NOT NULL         | User name       |
| status     | VARCHAR(30)  | NOT NULL         | Active/Inactive |
| created_at | TIMESTAMP    | NOT NULL         | Creation time   |

---

## 15. Role Table

### Table: `roles`

| Column      | Data Type    | Constraint       | Description      |
| ----------- | ------------ | ---------------- | ---------------- |
| role_id     | BIGSERIAL    | PK               | Role identifier  |
| role_name   | VARCHAR(100) | UNIQUE, NOT NULL | Role name        |
| description | TEXT         | NULL             | Role description |
| created_at  | TIMESTAMP    | NOT NULL         | Creation time    |

---

## 16. Permission Table

### Table: `permissions`

| Column          | Data Type    | Constraint       | Description            |
| --------------- | ------------ | ---------------- | ---------------------- |
| permission_id   | BIGSERIAL    | PK               | Permission identifier  |
| permission_code | VARCHAR(100) | UNIQUE, NOT NULL | Permission code        |
| permission_name | VARCHAR(150) | NOT NULL         | Permission name        |
| description     | TEXT         | NULL             | Permission description |

---

## 17. User Role Table

### Table: `user_roles`

| Column      | Data Type | Constraint | Description          |
| ----------- | --------- | ---------- | -------------------- |
| user_id     | BIGINT    | PK, FK     | User                 |
| role_id     | BIGINT    | PK, FK     | Role                 |
| assigned_at | TIMESTAMP | NOT NULL   | Assignment timestamp |
| assigned_by | BIGINT    | FK         | Administrator        |

The combination of `user_id` and `role_id` shall form the composite primary key.

---

## 18. Role Permission Table

### Table: `role_permissions`

| Column        | Data Type | Constraint | Description          |
| ------------- | --------- | ---------- | -------------------- |
| role_id       | BIGINT    | PK, FK     | Role                 |
| permission_id | BIGINT    | PK, FK     | Permission           |
| assigned_at   | TIMESTAMP | NOT NULL   | Assignment timestamp |

The combination of `role_id` and `permission_id` shall form the composite primary key.

---

## 19. Audit Log Table

### Table: `audit_logs`

| Column         | Data Type    | Constraint | Description                  |
| -------------- | ------------ | ---------- | ---------------------------- |
| audit_id       | BIGSERIAL    | PK         | Audit record                 |
| user_id        | BIGINT       | FK         | User performing action       |
| action         | VARCHAR(100) | NOT NULL   | CREATE/UPDATE/DELETE/EXECUTE |
| entity_type    | VARCHAR(100) | NOT NULL   | Entity affected              |
| entity_id      | VARCHAR(100) | NOT NULL   | Entity identifier            |
| previous_value | JSONB        | NULL       | Previous state               |
| new_value      | JSONB        | NULL       | New state                    |
| result         | VARCHAR(30)  | NOT NULL   | Success/Failure              |
| timestamp      | TIMESTAMP    | NOT NULL   | Action time                  |

---

## 20. Requirement Table

### Table: `requirements`

This table will support automated requirements traceability.

| Column           | Data Type    | Constraint       | Description                    |
| ---------------- | ------------ | ---------------- | ------------------------------ |
| requirement_id   | BIGSERIAL    | PK               | Requirement identifier         |
| requirement_code | VARCHAR(50)  | UNIQUE, NOT NULL | BR/FSD/SR requirement ID       |
| requirement_type | VARCHAR(20)  | NOT NULL         | BRD/FSD/SRD                    |
| title            | VARCHAR(200) | NOT NULL         | Requirement title              |
| description      | TEXT         | NOT NULL         | Requirement description        |
| status           | VARCHAR(30)  | NOT NULL         | Draft/Approved/Changed/Retired |
| version          | VARCHAR(20)  | NOT NULL         | Requirement version            |
| created_by       | BIGINT       | FK               | Creator                        |
| created_at       | TIMESTAMP    | NOT NULL         | Creation time                  |
| updated_at       | TIMESTAMP    | NULL             | Last update                    |

---

## 21. Requirement Traceability Table

### Table: `requirement_traceability`

This table will automate the relationship between requirements and downstream delivery artifacts.

| Column                | Data Type   | Constraint | Description                           |
| --------------------- | ----------- | ---------- | ------------------------------------- |
| traceability_id       | BIGSERIAL   | PK         | Traceability record                   |
| source_requirement_id | BIGINT      | FK         | Parent requirement                    |
| target_requirement_id | BIGINT      | FK         | Related requirement                   |
| relationship_type     | VARCHAR(50) | NOT NULL   | DERIVED_FROM / IMPLEMENTS / TESTED_BY |
| created_at            | TIMESTAMP   | NOT NULL   | Creation time                         |
| created_by            | BIGINT      | FK         | Creator                               |

Example:

`BR-002 → FSD-007 → SR-016`

The system shall use these relationships to generate the RTM automatically.

---

## 22. Test Case Table

### Table: `test_cases`

| Column          | Data Type    | Constraint       | Description               |
| --------------- | ------------ | ---------------- | ------------------------- |
| test_case_id    | BIGSERIAL    | PK               | Test case identifier      |
| test_case_code  | VARCHAR(50)  | UNIQUE, NOT NULL | Test case ID              |
| title           | VARCHAR(200) | NOT NULL         | Test case title           |
| description     | TEXT         | NULL             | Test scenario             |
| test_type       | VARCHAR(50)  | NOT NULL         | Functional/UAT/API/Data   |
| expected_result | TEXT         | NOT NULL         | Expected outcome          |
| status          | VARCHAR(30)  | NOT NULL         | Draft/Ready/Passed/Failed |
| created_at      | TIMESTAMP    | NOT NULL         | Creation time             |

---

## 23. Requirement Test Mapping Table

### Table: `requirement_test_mapping`

| Column          | Data Type   | Constraint | Description         |
| --------------- | ----------- | ---------- | ------------------- |
| requirement_id  | BIGINT      | PK, FK     | Requirement         |
| test_case_id    | BIGINT      | PK, FK     | Test case           |
| coverage_status | VARCHAR(30) | NOT NULL   | Covered/Not Covered |
| created_at      | TIMESTAMP   | NOT NULL   | Mapping time        |

This allows FinPilot to calculate requirement test coverage automatically.

---

## 24. Defect Table

### Table: `defects`

| Column         | Data Type   | Constraint       | Description                      |
| -------------- | ----------- | ---------------- | -------------------------------- |
| defect_id      | BIGSERIAL   | PK               | Defect identifier                |
| defect_number  | VARCHAR(50) | UNIQUE, NOT NULL | Defect number                    |
| requirement_id | BIGINT      | FK               | Related requirement              |
| test_case_id   | BIGINT      | FK               | Related test                     |
| severity       | VARCHAR(20) | NOT NULL         | Severity                         |
| priority       | VARCHAR(20) | NOT NULL         | Priority                         |
| description    | TEXT        | NOT NULL         | Defect description               |
| status         | VARCHAR(30) | NOT NULL         | Open/In Progress/Resolved/Closed |
| assigned_to    | BIGINT      | FK               | Assigned user                    |
| created_at     | TIMESTAMP   | NOT NULL         | Creation time                    |
| resolved_at    | TIMESTAMP   | NULL             | Resolution time                  |

---

## 25. Key Relationships

### Customer and Account

`customers 1 → many accounts`

One customer can have multiple accounts.

### Source System and Dataset

`source_systems 1 → many datasets`

One source system can provide multiple datasets.

### Dataset and Migration Batch

`datasets 1 → many migration_batches`

One dataset can be processed through multiple migration batches.

### Migration Batch and Validation

`migration_batches 1 → many validation_results`

One batch can generate many validation results.

### Migration Batch and Reconciliation

`migration_batches 1 → many reconciliation_results`

One batch can have reconciliation results.

### Validation/Reconciliation and Exception

Validation and reconciliation failures can generate exceptions.

### User and Role

`users many ↔ many roles`

Implemented through `user_roles`.

### Role and Permission

`roles many ↔ many permissions`

Implemented through `role_permissions`.

### Requirement Traceability

`requirements many ↔ many requirements`

Implemented through `requirement_traceability`.

### Requirement and Test Case

`requirements many ↔ many test_cases`

Implemented through `requirement_test_mapping`.

### Requirement and Defect

A requirement can have multiple associated defects.

---

## 26. Data Integrity Rules

The database shall enforce:

1. Primary-key uniqueness.
2. Foreign-key referential integrity.
3. Required-field NOT NULL constraints.
4. Unique business identifiers.
5. Valid status values where applicable.
6. Positive or valid financial amounts where applicable.
7. Valid customer-account relationships.
8. Unique requirement codes.
9. Unique test-case codes.
10. Unique role and permission codes.

---

## 27. Indexing Strategy

Indexes shall be created on frequently searched fields, including:

* `customers.customer_number`
* `accounts.account_number`
* `accounts.customer_id`
* `migration_batches.batch_number`
* `migration_batches.batch_status`
* `validation_results.batch_id`
* `validation_results.result_status`
* `exceptions.exception_number`
* `exceptions.status`
* `exceptions.severity`
* `requirements.requirement_code`
* `test_cases.test_case_code`

---

## 28. Data Security

Sensitive data shall be protected through:

* Role-based access control
* Database permissions
* API authorization
* Encryption in transit
* Encryption at rest in production
* Controlled access to sensitive datasets
* Audit logging

Initial development shall use synthetic/non-production financial data.

---

## 29. Database Design Principles

The database shall follow:

* Normalized relational design
* Referential integrity
* Separation of master and transactional data
* Auditability
* Traceability
* Scalability
* Least-privilege access
* Clear business keys
* Reusable configuration tables

---

## 30. Future Database Enhancements

Future releases may introduce:

* Transaction tables
* Payment instruction tables
* Data-quality score tables
* Workflow tables
* Notification tables
* AI agent execution history
* Multi-tenant architecture
* Tenant-level data isolation
* Advanced event/audit storage
* Data warehouse/star schema for analytics
* Azure SQL / cloud database implementation
