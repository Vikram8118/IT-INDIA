# India Tax Knowledge Base: working draft v0.1

Status: **DRAFT. Structure and schema proposal only. No sources have been compiled or ingested.**
Owner: to be confirmed. Scope: India, corporate taxpayers and their statutory auditors. Other countries come later.

## Design principles

1. **Law is data, not memory.** Every rule, rate, threshold and due date must trace to a provision_id, which traces to a confirmed source. Claude answers from retrieved records and cites them. If a record is missing, the answer is "not in the knowledge base", not a guess.
2. **Computation is code, not prose.** Tax arithmetic runs in a deterministic engine that reads the structured tables. Claude orchestrates, explains and drafts. It does not do the arithmetic from memory.
3. **Two regimes, one crosswalk.** The Income-tax Act 2025 (applies from TY 2026-27) and the Income-tax Act 1961 (still governs AY 2026-27 and earlier, and pending matters) are stored side by side. Section equivalences are explicit records, each with a verifier.
4. **Effective dating everywhere.** Each record has effective_from / effective_to. A query for FY 2025-26 must hit the rules in force for that year, not the latest version.
5. **Nothing enters without confirmation.** Sources sit in 01_sources/source_register.csv with status PROPOSED until the owner marks them CONFIRMED. Compilation is blocked until then.
6. **Client data is separated from law.** Corporate data and auditor workpapers live under 11_client_workspace/, with separate access. The legal layer is read-only to clients.

## Folder map

| Folder | Purpose | Written by |
|---|---|---|
| 00_governance | Decisions, open questions, change log, glossary | Owner, with Claude |
| 01_sources | Source register and downloaded official copies | Owner confirms; Claude registers |
| 02_law | Acts and Rules, both regimes, in markdown | Compilation pipeline (after confirmation) |
| 03_forms | ITR, TDS/TCS returns, certificates, challans | Pipeline |
| 04_circulars_notifications | CBDT circulars, notifications, orders | Pipeline |
| 05_allied_law | DTAA, Companies Act, Ind AS, Black Money Act, other statutes | Pipeline |
| 06_guidance_faqs | Official FAQs and interplay guidance | Pipeline |
| 07_structured_tables | Machine-readable rates, thresholds, due dates, crosswalks, form map | Pipeline, then reviewer |
| 08_schemas | Schema definitions (this proposal) | Owner approves |
| 09_computation_specs | Rules the engine implements (TDS, corporate tax, MAT/AMT, 26AS recon) | Tax specialist |
| 10_audit_framework | Checklists, exception rules, report templates | Auditor side |
| 11_client_workspace | Per-client inputs, workpapers and outputs (template only for now) | Corporate and auditor |
| 12_validation | Golden cases, regression tests, hallucination probes | Owner and tax specialist |
| 99_archive | Superseded versions, kept for historical queries | Pipeline |

## Naming conventions (proposed)

- provision_id: `<REGIME>-<SECTION>[-<SUB>]`, for example `ITA25-393-1` or `ITA61-194C-2`. REGIME codes are defined in 08_schemas/provision.schema.json.
- File names: `<provision_id>.md`, with YAML front matter that repeats the key fields.
- Dates: ISO 8601. Financial years are written as FY 2025-26 and tax years as TY 2026-27. The old AY terms are kept only for 1961-Act records.
- Amounts: INR, stored as integers in paise or as decimals with explicit units. No floats for money in the engine.

## Status of this draft

- Folder tree: created locally at /root/india-tax-kb/.
- Schema: v0.1 proposal in 08_schemas/. Needs your review.
- Source register: candidate list only, all PROPOSED.
- Google Drive: the corporate Drive mirror will be created once you share it. The local tree will then be copied over, and the Drive copy becomes the master.
