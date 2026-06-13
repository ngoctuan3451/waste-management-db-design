# Normalisation — UNF → 3NF

Normalising the **collection report** (the most complex data group: a collection
run, the truck used, and every bin emptied on that run) from un-normalised form
through to third normal form.

> Notation: primary-key attributes are **bold**. Nested repeating groups are shown
> with parentheses.

---

## UNF — Un-normalised form

A single structure with repeating groups (a collection covers many bins; a
contract covers many waste types over many dates):

```
COLLECTION (
  contract_no, waste_type_id, waste_type_desc, collection_freq,
  ( collection_date, truck_vin, truck_rego, truck_make, truck_model, truck_year,
    ( bin_rfid, bin_collected_kgs, bin_overweight ) ) )
```

## 1NF — Remove repeating groups

Each repeating group becomes its own relation with a key propagated down.

- **CONTRACT_WASTE** (**contract_no**, **waste_type_id**, waste_type_desc, collection_freq)
- **COLLECTION** (**contract_no**, **waste_type_id**, **collection_date**, truck_vin, truck_rego, truck_make, truck_model, truck_year)
- **COLLECTION_BIN** (**collection_date**, **bin_rfid**, contract_no, waste_type_id, bin_collected_kgs, bin_overweight)

**Partial dependencies identified:**
- `waste_type_id → waste_type_desc`
- `bin_rfid → waste_type_id`

## 2NF — Remove partial dependencies

Non-key attributes must depend on the *whole* key. `waste_type_desc` depends only
on `waste_type_id`, and `waste_type_id` depends only on `bin_rfid` — so both are
split out.

- **CONTRACT_WASTE** (**contract_no**, **waste_type_id**, collection_freq)
- **WASTE_TYPE** (**waste_type_id**, waste_type_desc)
- **COLLECTION** (**contract_no**, **waste_type_id**, **collection_date**, truck_vin, truck_rego, truck_make, truck_model, truck_year)
- **COLLECTION_BIN** (**collection_date**, **bin_rfid**, contract_no, bin_collected_kgs, bin_overweight)
- **BIN** (**bin_rfid**, waste_type_id)

**Transitive dependency identified:**
- `truck_vin → truck_rego, truck_make, truck_model, truck_year`

## 3NF — Remove transitive dependencies

The truck attributes depend on `truck_vin`, not on the collection key, so `TRUCK`
becomes its own relation.

- **CONTRACT_WASTE** (**contract_no**, **waste_type_id**, collection_freq)
- **WASTE_TYPE** (**waste_type_id**, waste_type_desc)
- **COLLECTION** (**contract_no**, **waste_type_id**, **collection_date**, truck_vin)
- **TRUCK** (**truck_vin**, truck_rego, truck_make, truck_model, truck_year)
- **COLLECTION_BIN** (**collection_date**, **bin_rfid**, contract_no, bin_collected_kgs, bin_overweight)
- **BIN** (**bin_rfid**, waste_type_id)

### Full functional dependencies (final)

```
contract_no, waste_type_id                 → collection_freq
waste_type_id                              → waste_type_desc
contract_no, waste_type_id, collection_date → truck_vin
truck_vin                                  → truck_rego, truck_make, truck_model, truck_year
collection_date, bin_rfid                  → contract_no, bin_collected_kgs, bin_overweight
bin_rfid                                   → waste_type_id
```

Every non-key attribute now depends on the key, the whole key, and nothing but the
key — the design is in **3NF**.
