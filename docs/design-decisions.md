# Design Decisions & Assumptions

The business assumptions behind the model, and the reasoning for the key logical
design choices.

---

## Business assumptions

These shaped which relationships are optional vs mandatory:

- A **local authority** may be recorded before signing a contract with Clear Away — it may still be negotiating.
- A **waste type** may be recorded before it is included in any contract.
- A **bin type** may be recorded before it is made available under any contract.
- An **owner** is recorded only if they currently own at least one property in the system.
- A **truck** may exist in the system before being assigned to any collection run.
- A **contract waste-type coverage** may exist before any collection runs are made under it.
- A **bin** may exist in the system before it is ever collected.

## Logical-model decisions

### WASTE_TYPE as a lookup table (not a CHECK constraint)
The case study states the company wants to **easily add new waste types**. A lookup
table supports that by simply inserting a new row, whereas a `CHECK` constraint
would require an `ALTER TABLE` every time — so `WASTE_TYPE` is modelled as its own
table.

### `bin_replacement_reason` is nullable
Not every bin is a replacement. A bin supplied to a property for the first time has
no replacement reason, so the column must allow `NULL`.

### Surrogate-key choices

- **BIN_TYPE** — introduced a surrogate `bin_type_id` for the natural pair
  `(waste_type_id, bin_size)`. This stops `waste_type_id` from having to appear
  twice in `CONTRACT_BIN` while keeping a clean primary key. (`waste_type_id` is
  still carried in `CONTRACT_BIN` as part of the composite FK to `CONTRACT_WASTE`,
  to preserve referential integrity with that table's composite key.)
- **CONTRACT_BIN** — a surrogate `ctb_id` reduces the relation from five identifying
  attributes to a minimal key and avoids update anomalies: if a natural-key value
  changed, every child row would otherwise have to change too.
- **COLLECTION** — given its own surrogate so that `COLLECTION_BIN` (already
  identified by `BIN`, which carries `contract_no` and `waste_type_id`) does not
  duplicate those attributes — keeping `COLLECTION_BIN` simpler and avoiding
  redundancy.
- **PROPERTY** — `authority_name` is a text-based natural key, so using it directly
  would force cascading updates across `STREET`, `PROPERTY`, and `PROPERTY_OWNER`
  (update-anomaly risk). A surrogate `pro_id` avoids that and keeps `BIN` cleaner.

---

*See [`normalisation.md`](normalisation.md) for the UNF → 3NF working, and the
conceptual/logical ER diagrams in this folder.*
