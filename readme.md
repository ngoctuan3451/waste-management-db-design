# Waste-Collection Management — Relational Database Design

**A normalised Oracle database designed from a business brief: conceptual model → logical model → physical schema, for a municipal waste-collection system.**

---

## Problem

A waste-management operator needs a database to track which **bins** are supplied to which **properties**, which **contracts** local authorities hold, how waste is **collected** (by which truck, on which date, how many kilograms), and how it's all billed. The task: turn the business requirements into a correct, normalised relational design and a working Oracle schema.

## What I did

- **Conceptual modelling** — identified entities, relationships, and cardinalities into an ER model (`ca_conceptual.pdf`).
- **Logical modelling** — mapped the ER model to relations with keys and foreign keys (`ca_logical.pdf`).
- **Normalisation** — normalised to remove redundancy and update anomalies, with the working documented (`ca_normalisation.pdf`).
- **Documented assumptions** — recorded the business-rule assumptions that shaped the design (`ca_assumptions.pdf`).
- **Physical implementation** — wrote the DDL for **14 tables** in Oracle 12c (`ca_schema.sql`).

## Design highlights

- **14 related tables**: `BIN`, `BIN_TYPE`, `COLLECTION`, `COLLECTION_BIN`, `CONTRACT`, `CONTRACT_BIN`, `CONTRACT_WASTE`, `LOCAL_AUTHORITY`, `OWNER`, `PROPERTY`, `PROPERTY_OWNER`, `STREET`, `TRUCK`, `WASTE_TYPE`.
- **Integrity enforced in the schema, not the app**:
  - Primary keys on every table, including composite keys (e.g. `COLLECTION_BIN(bin_rfid, collection_id)`).
  - 18 foreign-key constraints wiring the model together.
  - `UNIQUE` constraints for real-world business rules (e.g. one collection per contract / waste-type / date).
  - `CHECK` constraints for coded domains (e.g. bin-replacement reason ∈ `BF/DO/DW/ST`, authority type ∈ `B/C/D/S/T`).
- **Many-to-many relationships resolved** with associative tables (`PROPERTY_OWNER`, `COLLECTION_BIN`, `CONTRACT_BIN`).
- Column-level `COMMENT`s document every field's meaning.

## How to run

In Oracle SQL Developer (or any Oracle 12c+ connection):

```sql
@ca_schema.sql
```

The script drops any existing tables, recreates all 14, then adds constraints and foreign keys. It spools output to `ca_schema_output.txt`.

## Repository structure

```
.
├── ca_schema.sql           # the physical DDL (14 tables, constraints, FKs)
├── ca_conceptual.pdf       # ER / conceptual model
├── ca_logical.pdf          # logical model
├── ca_normalisation.pdf    # normalisation working
└── ca_assumptions.pdf      # documented business assumptions
```

## Skills shown

Relational data modelling · normalisation (to 3NF) · Oracle SQL DDL · constraints & referential integrity · resolving M:N relationships.

## Author

**Tuan Ngoc Chu** — FIT2094 Databases, Monash University (Semester 1, 2026).
