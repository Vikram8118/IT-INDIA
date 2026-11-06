# Decisions and open questions

Use this file for brainstorming. Mark each item DECIDED, OPEN or PARKED.

## Decisions taken so far

| # | Decision | Status |
|---|---|---|
| D1 | No source compilation until the owner confirms the source register | DECIDED |
| D2 | Two regimes stored side by side, linked by an explicit crosswalk | DECIDED (proposed) |
| D3 | Tax arithmetic done by a deterministic engine, not by the language model | DECIDED (proposed) |
| D4 | India first. Other countries after India is validated | DECIDED |
| D5 | Corporate and auditor share the legal layer; client data is separated per client | DECIDED (proposed) |

## Open questions

| # | Question | Why it matters | Status |
|---|---|---|---|
| Q1 | Which sources are confirmed? See 01_sources/source_register.csv. | Gate for compilation | OPEN |
| Q2 | Is the Google Drive shared yet, and which folder should be the master? | Where the tree is mirrored | OPEN |
| Q3 | Which regime for the first build: 1961 Act for FY 2025-26 and earlier, or 2025 Act from TY 2026-27? | Scope of first compile and tests | OPEN |
| Q4 | Which modules first: TDS, corporate tax computation, 26AS reconciliation, or audit checklists? | Sequencing | OPEN |
| Q5 | Corporate tax regime options: which concessional regimes should be modelled and which left out? | Computation coverage | OPEN |
| Q6 | How far to go on DTAA: all treaties, or only the main partner countries? | Volume | OPEN |
| Q7 | Will the Companies Act, Ind AS and GST be in scope as allied laws, or only as references? | Scope | OPEN |
| Q8 | Who signs off the rule logic (a qualified CA, and who)? Is there a sign-off step before outputs go to clients or to the auditor? | Liability and reliability | OPEN |
| Q9 | Should the auditor and the corporate see the same reconciliation, or separate working files? | Independence and access | OPEN |
| Q10 | Is the 26AS/AIS data to be imported as files from the portal, or entered manually? | Input pipeline | OPEN |
| Q11 | Retention: how many years of client data, and how is it deleted? | Data protection | PARKED |
| Q12 | Should the engine also produce return-filing JSON or only computation sheets? | Output format | PARKED |

## Brainstorm notes (not decisions)

- A test-first approach: write golden cases (known correct answers from a CA) before compiling large volumes. Each golden case then becomes a check on the knowledge base.
- Hallucination probes: questions where the right answer is "not in the knowledge base", or where the 1961 and 2025 answers differ. Claude must pass these before the system is used.
- Versioning: a new Finance Act creates a new version of affected records, not an overwrite. Older versions move to 99_archive.
- Ambiguity handling: where the law is unclear, the record carries a flag and the output says so, rather than picking one reading silently.
