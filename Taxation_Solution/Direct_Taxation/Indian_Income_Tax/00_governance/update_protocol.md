# Update protocol

Purpose: keep the knowledge base current without silent drift. Applies to every record in 02 to 07 and 13.

## Source policy

1. Official sources only: the Income Tax Department and CBDT websites (incometaxindia.gov.in and its subpages and documents).
2. Any source_id must have allowlisted = YES in 01_sources/source_register.csv. The engine rejects records from other domains.
3. Other sites (news, consultancy blogs, aggregators, gazette mirrors) are not used for ingestion. They may only be noted in governance files for context, and never cited as a source of law.
4. Case law is not fetched from the internet under this policy. Judgments are added only when supplied by the owner, and marked USER_SUPPLIED.
5. Where a document is only available outside the allowlist, the owner decides whether to supply the official copy. The record stays unverified until then.

## Update cycles

| Trigger | What is checked | Output |
|---|---|---|
| Union Budget / Finance Act (annual) | Full diff of Acts, Rules, forms and rate/threshold tables | New record versions; amendment links; regression tests run |
| Circulars and notifications index (weekly) | New items on the official index pages, compared with a stored snapshot hash | Candidate items added to the review queue; none ingested automatically |
| Forms (on each new notified form set) | Form versions and field changes | New form versions; section-to-form map updated |
| Due dates (monthly, plus on any CBDT extension) | Due-date rules and extension notifications | due_dates.csv rows added or end-dated |
| Rates and thresholds (quarterly) | Spot check against the Act and notified rates | Verified or corrected rows |
| Stale-record sweep (monthly) | Any record past next_review_due | Warning list to the owner |

## Change procedure (for every change)

1. Detect: a candidate change is found on an official page, or flagged by a cycle above.
2. Record: add a row to 00_governance/update_log/update_log.csv, with the change, source_id, date found and reviewer.
3. Version: the old record gets effective_to set. The new record gets a new effective_from. Nothing is overwritten. Superseded text moves to 99_archive.
4. Verify: a reviewer checks the change against the official text and sets verification.status.
5. Test: run the golden cases and regression tests in 12_validation. A failing test blocks the release.
6. Release: the knowledge base is tagged with a version (for example kb-2026.11.1) so each computation run can record which version it used.

## Staleness

- Every record has last_verified_on and next_review_due.
- Records past next_review_due are shown with a staleness warning in any output that cites them.
- Computations made on a stale record are marked as such in the audit trail.

## Versioning of outputs

Each computation run stores the knowledge-base tag, the rate, threshold and due-date row IDs it used, and the engine version. A report can then be reproduced exactly later, even after the law has changed.

## Validation gate (added 2026-11-06)

1. Watch runs write candidates to 00_governance/update_queue/queue.csv only, status PENDING_VALIDATION.

2. A reviewer checks each candidate against the official text, then sets status to VALIDATED (or REJECTED) with validated_by and validated_on.

3. Only VALIDATED rows are applied to the legal layer, in a new kb version. Applied rows record applied_in_kb_version.

4. Sources are added to the register only by the owner (owner_decision = CONFIRMED).
