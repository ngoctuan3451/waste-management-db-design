# Waste-Collection Management — Relational Database Design

**A normalised Oracle database designed from a business brief: conceptual model → logical model → physical schema, for a municipal waste-collection system.**

---

## Case study

**Client:** *Clear Away (CA)* — a garbage-collection company contracted by local government authorities (cities, towns, shires) to collect household waste.

**Business driver — Pay-As-You-Dump (PAYD).** To encourage more sustainable households, councils want to move from a flat waste-collection fee to a usage-based model that charges residents by the **type and weight** of waste collected from their property. Clear Away enables this by rolling out **sensor-based "smart" wheelie bins**, each carrying an **RFID tag** that uniquely identifies it. The weight captured at each collection feeds the charges councils invoice to residents — so the data has to be accurate and well-structured.

**What the database must capture:**
- **Local authorities** — name, CEO, contact, total area and type; each is responsible for its **streets** (street ID unique within an authority, plus name, length, surface and lane count).
- **Properties & owners** — properties sit on streets (street number unique per street) and may have several owners, each with contact details.
- **Waste types & bin types** — multiple waste streams (green, recycling, landfill, …); each bin type comes in a range of sizes, each with a standard supply cost.
- **Contracts** — each authority signs a contract that sets the collection cost per kilogram for each waste type and the (contract-specific) bin supply costs; a contract covers one or more waste types.
- **Bins & collections** — individual RFID bins are registered when supplied to a property under a contract (to track misuse and theft); each collection records the weight per bin for PAYD billing.

**The solution.** A normalised (3NF) Oracle database of 14 tables modelling authorities, streets, properties, owners, contracts, waste and bin types, bins and collections — enforcing the business rules through keys, foreign keys and constraints to give Clear Away a single source of truth for operations and PAYD invoicing.

## What I did

- **Conceptual modelling** — identified entities, relationships, and cardinalities into an ER model (`docs/conceptual_erd.pdf`).
- **Logical modelling** — mapped the ER model to relations with keys and foreign keys (`docs/logical_model.pdf`).
- **Normalisation** — normalised to 3NF to remove redundancy and update anomalies (`docs/normalisation.md`).
- **Documented assumptions** — recorded the business-rule assumptions that shaped the design (`docs/design-decisions.md`).
- **Physical implementation** — wrote the DDL for **14 tables** in Oracle 12c (`scripts/ca_schema.sql`).

## Design highlights

- **14 related tables**: `BIN`, `BIN_TYPE`, `COLLECTION`, `COLLECTION_BIN`, `CONTRACT`, `CONTRACT_BIN`, `CONTRACT_WASTE`, `LOCAL_AUTHORITY`, `OWNER`, `PROPERTY`, `PROPERTY_OWNER`, `STREET`, `TRUCK`, `WASTE_TYPE`.
- **Integrity enforced in the schema, not the app**:
  - Primary keys on every table, including composite keys (e.g. `COLLECTION_BIN(bin_rfid, collection_id)`).
  - 18 foreign-key constraints wiring the model together.
  - `UNIQUE` constraints for real-world business rules (e.g. one collection per contract / waste-type / date).
  - `CHECK` constraints for coded domains (e.g. bin-replacement reason ∈ `BF/DO/DW/ST`, authority type ∈ `B/C/D/S/T`).
- **Many-to-many relationships resolved** with associative tables (`PROPERTY_OWNER`, `COLLECTION_BIN`, `CONTRACT_BIN`).
- Column-level `COMMENT`s document every field's meaning.

## Design documentation

The full design write-up is in [`docs/`](docs/), rendered inline as markdown:

- **[Normalisation (UNF → 3NF)](docs/normalisation.md)** — step-by-step normalisation of the collection data, showing the partial, transitive, and full functional dependencies at each stage.
- **[Design decisions & assumptions](docs/design-decisions.md)** — the business assumptions, and the reasoning behind the lookup table, nullable columns, and surrogate-key choices.

The conceptual and logical ER diagrams are in `docs/` (`conceptual_erd.pdf`, `logical_model.pdf`).

## How to run

In Oracle SQL Developer (or any Oracle 12c+ connection):

```sql
@scripts/ca_schema.sql
```

The script drops any existing tables, recreates all 14, then adds constraints and foreign keys.

## Repository structure

```
.
├── scripts/
│   └── ca_schema.sql           # physical DDL (14 tables, constraints, FKs)
├── docs/
│   ├── normalisation.md        # UNF -> 3NF working
│   ├── design-decisions.md     # assumptions + key design rationale
│   ├── conceptual_erd.pdf      # ER / conceptual model (diagram)
│   ├── logical_model.pdf       # logical model (diagram)
│   └── conceptual_erd.drawio   # editable diagram source
├── readme.md
└── .gitignore
```

## Skills shown

Relational data modelling · normalisation (to 3NF) · Oracle SQL DDL · constraints & referential integrity · resolving M:N relationships.

## Author

**Tuan Ngoc Chu** — FIT2094 Databases, Monash University (Semester 1, 2026).
