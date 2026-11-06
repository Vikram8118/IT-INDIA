# Schema v0.1: entities for validation

Purpose: agree the data model before any source is compiled. Every entity below maps to a folder or table in this knowledge base. Fields marked (req) are mandatory.

## Layer A: Legal knowledge (shared, read-only to clients)

### A1. Instrument
A legal document as a whole: an Act, Rules, a notification, a circular, a form, an FAQ or a treaty.

| Field | Notes |
|---|---|
| instrument_id (req) | e.g. `IT_ACT_2025`, `CBDT_CIRC_2026_05` |
| instrument_type (req) | ACT, RULES, FORM, NOTIFICATION, CIRCULAR, ORDER, FAQ, TREATY, OTHER_STATUTE |
| regime (req) | IT_ACT_2025, IT_ACT_1961, IT_RULES_2026, IT_RULES_1962, ALLIED |
| number / date (req) | Official number and issue date |
| effective_from / effective_to | Null effective_to = still in force |
| source_id (req) | Link to 01_sources/source_register.csv |
| status (req) | in_force, superseded, withdrawn, not_yet_effective |

### A2. Provision
The atomic unit of law: a section, rule, sub-section, clause or form field. Claude cites provisions, never whole instruments.

| Field | Notes |
|---|---|
| provision_id (req) | See naming convention in README |
| instrument_id (req) | Parent instrument |
| parent_provision_id | Sub-section to section, rule to chapter |
| heading | Marginal heading as published |
| text_path (req) | Path to the markdown text |
| effective_from (req) / effective_to | Dated validity |
| status (req) | in_force, omitted, substituted, not_yet_effective, superseded, unverified |
| verification (req) | proposed, confirmed_by_reviewer, validated_by_CA, with reviewer and date |

### A3. Amendment link
How provisions change over time.

| Field | Notes |
|---|---|
| from_provision_id (req) | The amending provision or instrument |
| to_provision_id (req) | The affected provision |
| relation (req) | inserts, substitutes, omits, amends_proviso, clarifies (non-legislative) |
| effective_from (req) | Date the change took effect |

### A4. Crosswalk (1961 to 2025)
Explicit equivalence between old and new sections. Never inferred silently.

| Field | Notes |
|---|---|
| old_provision_id (req) | Section under IT_ACT_1961 or IT_RULES_1962 |
| new_provision_id (req) | Section under IT_ACT_2025 or IT_RULES_2026 |
| equivalence (req) | identical, changed, partial, no_equivalent |
| difference_note | Plain-language note on what changed |
| verified_by (req) | Reviewer who checked the mapping |

### A5. Rate
Any percentage used in a computation (TDS, TCS, corporate tax, surcharge, cess, MAT, AMT).

| Field | Notes |
|---|---|
| rate_id (req) | |
| provision_id (req) | Where the rate is legally set |
| rate_type (req) | TDS, TCS, CORP_TAX, SURCHARGE, CESS, MAT, AMT, OTHER |
| nature_of_payment | Codes for the TDS section (for example salary, contract, rent) |
| payee_condition | Resident, non-resident, PAN available or not, and so on |
| rate_value (req) | Decimal |
| effective_from (req) / effective_to | |
| applies_when | Machine-readable condition, for example `threshold_exceeded` |

### A6. Threshold
Monetary limits (exemption, aggregate, deduction ceiling, filing thresholds).

| Field | Notes |
|---|---|
| threshold_id (req) | |
| provision_id (req) | |
| applies_to (req) | What the threshold measures |
| amount_inr (req) | Integer |
| basis | single transaction, aggregate per FY, per payee, and so on |
| effective_from (req) / effective_to | |

### A7. Form
A prescribed return, certificate or challan, with its fields.

| Field | Notes |
|---|---|
| form_id (req) | e.g. `ITR-6`, `24Q`, `26Q`, `27Q`, `27EQ`, `FORM_16A`, `FORM_15CA_15CB` |
| regime (req) | Which Rules it belongs to |
| purpose (req) | |
| version / effective_from (req) | Forms change every year |
| field list | field_code, label, data_type, mandatory, validation_rule, source_ledger_hint |

### A8. Due date rule
Filing, payment and certificate deadlines.

| Field | Notes |
|---|---|
| due_rule_id (req) | |
| obligation (req) | e.g. TDS deposit, TDS return quarterly, ITR for company, Form 16A issue |
| provision_id (req) | |
| trigger (req) | Month end of deduction, FY end, date of audit report, and so on |
| offset (req) | Days or months, plus the calendar basis |
| effective_from (req) / effective_to | |

### A9. Section-to-form map
Which form and which field each TDS section, or each computation step, feeds.

| Field | Notes |
|---|---|
| section (req) | TDS section, regime-tagged |
| form_id (req) | Return it reports in |
| form_field (req) | Field code |
| rule_provision_id | Rule that prescribes the reporting |

## Layer B: Client data (per-client workspace, confidential)

### B1. Entity profile
Corporate identity, tax residency, elected regime (for example a concessional corporate tax option), financial year, PAN (stored masked), TAN, GSTIN, auditor, and applicable allied statutes.

### B2. Ledger transaction
Each books entry: date, account, party, party PAN (masked), gross amount, GST component, nature (for example professional fee, rent, interest), TDS applicability flag.

### B3. TDS deduction record
Date of deduction, section applied, rate applied, base amount, TDS amount, deposit challan reference, deposit date, return period, and status.

### B4. Challan record
BSR code, challan serial, date, amount, tax, interest and fee, mapping to TDS records.

### B5. Tax-credit statement entry (Form 26AS / AIS)
Deductor TAN, amount paid, TDS credited, year, and transaction type, as imported from the official statement.

### B6. Reconciliation match
Links between B2/B3, B4 and B5. Fields: match status (matched, timing difference, amount difference, missing in 26AS, missing in books), difference amount, reason code, reviewer comment.

### B7. Computation run
Inputs (snapshot of B1 to B6 and rule versions used), engine version, output tables, and a hash of the inputs. Every figure in a report points to a run.

### B8. Finding / exception
Rule that fired, severity, evidence links, auditor comment, client response, and closure status.

### B9. Audit trail
Who changed what, when, and from which source. Append-only.

## Cross-cutting rules (for the engine and for Claude)

1. Every output line carries provision_id(s) and rate_id / threshold_id used.
2. Queries are resolved against effective dates first. If no record matches the date, the answer says so.
3. If a provision status is unverified, Claude flags that in the answer.
4. Money is integer paise in the engine. Rounding follows the provision's rounding rule, which must be captured as a record.
5. Access: corporate users see A and B1 to B9 for their own entity. Auditors see A, and B for the clients they are assigned. Clients cannot see auditor workpapers unless released.
6. Claude may only write to outputs/ and never to the legal layer.

## Open questions for review

See 00_governance/decisions_and_open_questions.md.
