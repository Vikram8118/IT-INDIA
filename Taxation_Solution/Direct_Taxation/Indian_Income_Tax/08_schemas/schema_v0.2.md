# Schema v0.2

Changes from v0.1 (archived at 99_archive/schema_v0.1.md):

1. Provision IDs carry a jurisdiction prefix (`IN-`), so the same scheme works for other countries.
2. Due dates, rates and thresholds are separate tables, each with its own effective dating and update cycle. They are not stored inside provision text.
3. Every record has a `last_verified_on` date and a `next_review_due` date, so stale records are visible.
4. Sources are limited to an official-domain allowlist. The engine rejects any record whose source is outside it.
5. Case law is a separate layer, with provenance recorded.

## 1. Identifiers

**provision_id** = `<JURISDICTION>-<REGIME>-<SECTION>[-<SUB>][-<CLAUSE>]`

Examples:
- `IN-ITA25-393-1`: Income-tax Act, 2025, section 393, sub-section (1)
- `IN-ITA61-194C-2`: Income-tax Act, 1961, section 194C, sub-section (2)
- `IN-ITR26-R12`: Income-tax Rules, 2026, rule 12

Rules:
- The section number is the official number as printed in the instrument. Nothing is renumbered or invented.
- The engine never generates a provision ID. It looks IDs up in the provision table. Any ID Claude cites that does not exist in the table is rejected. This is the main control against invented section numbers.
- Two regimes can share a number (for example section 194C under 1961 and a different section under 2025). The regime code keeps them apart.

## 2. Legal layer (shared, read-only to clients)

### 2.1 Instrument
| Field | Notes |
|---|---|
| instrument_id (req) | e.g. `IN-IT-ACT-2025`, `IN-CBDT-CIRC-2026-05` |
| instrument_type (req) | ACT, RULES, FORM, NOTIFICATION, CIRCULAR, ORDER, FAQ, TREATY, FINANCE_ACT, OTHER_STATUTE |
| jurisdiction (req) | IN (for now) |
| regime (req) | IT_ACT_2025, IT_ACT_1961, IT_RULES_2026, IT_RULES_1962, CBDT_CIRCULAR, CBDT_NOTIFICATION, FORM, ALLIED, TREATY, GUIDANCE |
| official_number | e.g. G.S.R. number, circular number |
| issue_date (req) | |
| effective_from (req) / effective_to | Null effective_to means in force |
| source_id (req) | Must be an allowlisted official source |
| status (req) | in_force, superseded, withdrawn, not_yet_effective |

### 2.2 Provision
| Field | Notes |
|---|---|
| provision_id (req) | See section 1 |
| instrument_id (req) | |
| parent_provision_id | |
| section_number (req) | Official number as printed, kept as text |
| heading | |
| text_path (req) | Markdown file in the knowledge base |
| effective_from (req) / effective_to | |
| status (req) | in_force, omitted, substituted, not_yet_effective, superseded, unverified |
| source_id (req) | |
| verification (req) | proposed, confirmed_by_reviewer, validated_by_CA; with reviewer and date |
| last_verified_on (req) | |
| next_review_due (req) | Set by the update protocol (see 00_governance/update_protocol.md) |
| ambiguity_flag | none, interpretive, conflicting_sources, awaiting_case_law |

### 2.3 Amendment link
| Field | Notes |
|---|---|
| from_instrument_id / from_provision_id | Amending instrument or provision |
| to_provision_id (req) | Affected provision |
| relation (req) | inserts, substitutes, omits, amends_proviso, clarifies |
| effective_from (req) | |
| source_id (req) | |

### 2.4 Crosswalk (1961 to 2025) and overlap
| Field | Notes |
|---|---|
| old_provision_id (req) | IN-ITA61-... |
| new_provision_id (req) | IN-ITA25-... |
| equivalence (req) | identical, changed, partial, no_equivalent, split, merged |
| overlap_scope | Which part of the old provision is covered by the new one, in words |
| difference_note | Plain-language note on the change |
| source_id (req) | Source of the mapping (official utility or FAQ) |
| verified_by (req) | Reviewer who checked the row |
| verified_on (req) | |

The crosswalk is reviewed row by row. Your review of "highlighted overlaps" maps directly onto this table: the overlap_scope and equivalence fields.

## 3. Rates, thresholds and due dates (separate tables)

Each table lives in 07_structured_tables/ as CSV, with its own update cycle. Each row points to a provision. A row cannot exist without a provision_id and a source_id.

### 3.1 rates.csv
rate_id, provision_id, rate_type (TDS, TCS, CORP_TAX, SURCHARGE, CESS, MAT, AMT, OTHER), nature_of_payment, payee_condition, rate_value, rate_unit, applies_when, effective_from, effective_to, source_id, last_verified_on, next_review_due, notes

### 3.2 thresholds.csv
threshold_id, provision_id, applies_to, amount_inr (integer), basis (single_transaction, aggregate_per_fy, per_payee, other), effective_from, effective_to, source_id, last_verified_on, next_review_due, notes

### 3.3 due_dates.csv
due_rule_id, provision_id, form_id, obligation, trigger_event, offset_days, offset_months, calendar_basis (fy, quarter, month, calendar_date), fixed_date (if any), effective_from, effective_to, source_id, last_verified_on, next_review_due, notes

Due dates change through two routes: the law (a new Act or Finance Act) and the department (for example extensions by CBDT notification). Both are recorded as separate rows, and the extension row points to its notification as the source.

## 4. Forms

| Field | Notes |
|---|---|
| form_id (req) | e.g. ITR-6, 24Q, 26Q, 27Q, 27EQ, FORM-16A |
| regime (req) | |
| purpose (req) | |
| version (req) | Forms change most years |
| effective_from (req) / effective_to | |
| source_id (req) | |
| field list | field_code, label, data_type, mandatory, validation_rule, ledger_hint |

## 5. Section-to-form map
| Field | Notes |
|---|---|
| section_provision_id (req) | |
| form_id (req) | |
| form_field_code (req) | |
| rule_provision_id | Rule that prescribes the reporting |
| effective_from / effective_to | |

## 6. Case law (separate layer)

| Field | Notes |
|---|---|
| case_id (req) | |
| citation (req) | Court, date, reference as supplied |
| provenance (req) | USER_SUPPLIED (only option for now) |
| provision_ids_discussed | |
| holding_summary | Owner-approved summary |
| status | binding, persuasive, overruled, under_appeal (to be checked by the owner) |
| file_path | Copy of the judgment, if supplied |

Case law is not sourced from the internet under the current source policy. Judgments are added only when you supply them, and their provenance is recorded.

## 7. Client layer (confidential)

Unchanged from v0.1: entity profile (B1), ledger transactions (B2), TDS deduction records (B3), challans (B4), 26AS/AIS entries (B5), reconciliation matches (B6), computation runs (B7), findings (B8), audit trail (B9). Each computation run stores the versions of the rates, thresholds and due-date rows it used.

## 8. Rules for the engine and for Claude

1. Every output line carries the provision_id, rate_id, threshold_id or due_rule_id it used.
2. Retrieval uses the effective date first. If no record covers that date, the answer says so.
3. Claude may cite only IDs that exist in the tables. Unknown IDs are rejected.
4. A record with ambiguity_flag other than "none" is shown with its flag.
5. A record past next_review_due is shown with a staleness warning.
6. Money is integer paise in the engine.
7. Claude writes only to 11_client_workspace/*/outputs and never to the legal layer.
