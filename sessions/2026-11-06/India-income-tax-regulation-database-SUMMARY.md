# Session summary: India income-tax regulation database

Type: STRUCTURED SUMMARY, not a verbatim transcript. Prepared by Claude on 2026-11-07 from the full conversation, at the owner's request.
Conversation title: India income-tax regulation database
Project: Business Development - offshore
Session date: 2026-11-06 (IST)
Owner: Vikram (Vikram8118)
Reviewer (CA): RAMYAVIKRAM (listed as code owner for raw/, proposed/ and live/)

## 1. Goal
- Build a robust, auditable India income-tax knowledge base for corporate taxpayers and their statutory auditors.
- Claude is a layer over the database. Tax arithmetic is done by a deterministic engine, not from memory. Every figure and section cited must trace to a confirmed official source.
- India first. The same design is to be reused for 20 to 30 countries once India is validated.

## 2. Source policy (agreed)
- Official sources only: incometaxindia.gov.in, including CBDT pages and documents.
- Confirmed root sources (owner-confirmed, in the register as SRC-101 to SRC-103):
  - https://www.incometaxindia.gov.in/
  - https://www.incometaxindia.gov.in/all-rules
  - https://www.incometaxindia.gov.in/cbdt
- Other sources in the register (Acts, Rules, forms, circulars, notifications, Finance Acts, DTAA, crosswalk utilities, guidance) remain PROPOSED until the owner confirms them.
- User-supplied documents are accepted only with owner approval and are flagged USER_SUPPLIED.
- Case law is added only when the owner supplies it.
- No other websites are used for ingestion.

## 3. Legal position discussed and recorded
- Income-tax Act, 2025: in force from 1 April 2026. Applies from tax year 2026-27.
- Income-tax Act, 1961: still governs AY 2026-27 and earlier, and pending proceedings.
- Income-tax Rules, 2026: G.S.R. 198(E), dated 20 March 2026. Verified against the user-supplied copy (header and commencement clauses). Commencement 1 April 2026. Source remains USER_SUPPLIED pending owner decision (SRC-017). The 976-page PDF is not yet extracted.
- Fifth Amendment to the Rules: S.O. 830(E), dated 22 September 2026, effective 1 October 2026 (already in force). Per the Hindi text read so far (pages 1 to 3 only): amends Rule 215(1) table entry 3, Rule 218(3), Rule 219(5) and (8), and Appendix 3 Forms 132 and 141, to reference sections 393(1) and 393(2) of the 2025 Act. Earlier amendments (four before this one) are not yet collected. English text not yet checked (SRC-016, USER_SUPPLIED).
- Note: an earlier answer said G.S.R. 198(E) was dated 20 March 2026 without verification. It was later verified against the attached copy. The Gazette attached separately is S.O. 830(E), not G.S.R. 198(E).

## 4. Design decisions
- Two regimes (1961 and 2025) stored side by side, linked by an explicit crosswalk with overlap scope, verifier and date.
- Provision IDs carry a country prefix, for example IN-ITA25-393-1 (Act 2025, section 393, sub-section 1). Rules use the rule number, for example IN-ITR26-215-1. The engine never generates an ID; it looks up IDs in the table and rejects unknown ones. Section 393(1) and 393(2) are confirmed by the Gazette as existing references. Other example IDs used earlier (IN-ITA61-194C-2, IN-ITR26-R12) were format examples only and are not verified.
- Due dates, rates and thresholds are separate tables (due_dates.csv, rates.csv, thresholds.csv), each row linked to a provision and a source, each with effective dates.
- Every record has last_verified_on and next_review_due. Staleness is shown in outputs.
- Case law is a separate layer (13_case_law), USER_SUPPLIED only.
- Updates flow through a queue: detected, then validated by a reviewer, then released in a new version. Nothing applies automatically. Old records are end-dated, never overwritten.
- Client data is kept separate from the legal layer.
- Folder layout: Taxation_Solution / Direct_Taxation / Indian_Income_Tax, with raw/ (original files and checksums, read-only), proposed/ (converted markdown awaiting review), live/ (approved content only), and numbered working folders 00 to 13 and 99 (archive).

## 5. Repository and review workflow
- Master repository: Vikram8118/IT-INDIA (the connector lists it lowercase as it-india). GitHub is connected.
- Plan: Pro plan upgrade completed (required reviewers and protected branches in private repos).
- Branch review/initial-kb-structure on GitHub at 85d6099. It contains the structure, schema v0.2, source register, update protocol, update log, update queue, and .github/CODEOWNERS.
- CODEOWNERS: RAMYAVIKRAM and Vikram8118 own live/, raw/ and proposed/. Vikram8118 owns 00_governance/.
- Commit history was rewritten (owner approved) so every commit shows the Claude identity (Claude <noreply@anthropic.com>). Commits remain unsigned and show as Unverified. Branch protection must not require signed commits unless signing is set up.
- A CA review checklist (27 steps, line by line, with GitHub instructions) sits in a Google Sheet.
- Still to do in GitHub settings (owner): protect main (require pull request, 1 approval once the CA joins, require Code owners review, do not allow bypassing, block force pushes and deletions), add RAMYAVIKRAM as collaborator with Write access, and decide on Git LFS for the 976-page PDF.
- Pull request not yet opened by owner: https://github.com/Vikram8118/IT-INDIA/pull/new/review/initial-kb-structure

## 6. Google Drive (personal account linked to the connector)
- Folder "Claude Chat Indian IT Build", containing "Claude Chat Indian IT Build.md" (an earlier summary of the chat up to the Drive step).
- Google Sheet "Indian-IT Repository Build GITHUB" in My Drive root: the CA review checklist.
- Office Drive could not be linked (one Drive account per connector). The decision was to use GitHub as master. Drive items are to be moved to GitHub in phase 2.

## 7. Automation
- Weekly source watch: "India Tax KB official source watch", Mondays at 08:59 IST. Reads only CONFIRMED and allowlisted sources. Writes candidates to 00_governance/update_queue/queue.csv with status PENDING_VALIDATION. Does not apply anything. Currently reads the register from the Google Drive copy, which needs updating to the repository once the branch is merged.
- Daily session backup: "India Tax KB daily session backup", 23:59 IST every day. Scoped (per owner request) to this conversation only, identified by title "India income-tax regulation database" in project "Business Development - offshore". Writes to sessions/YYYY-MM-DD/ on branch sessions/daily-backup.
- Status at the time of this summary: the run on 6 November reported success, but the branch did not exist and nothing was written. Likely causes: the conversation tools were not available in that run, or the title did not match. Not yet confirmed. The next run is 7 November at 23:59 IST.
- Skill "retrieve-convo" proposed to the owner for saving. It reads the latest sessions/ day's INDEX.md from the backup branch and summarises a chosen conversation. Also sent as a file (SKILL.md).

## 8. Open items
1. Owner to confirm the remaining sources in the register (CONFIRMED or dropped).
2. Owner to decide whether the user-supplied Gazette copies (SRC-016, SRC-017) count as official sources.
3. Check the English text of the Fifth Amendment against the Hindi text.
4. Extract the 976-page Rules into raw/ with checksums, then convert to markdown after confirmation.
5. Decide the order of work. Suggested: Rules 2026, then Act 2025, then forms, then rates, due dates and circulars.
6. Owner to set branch protection on main and add the CA as collaborator (see section 5).
7. Decide Git LFS for large PDFs.
8. Verify the daily backup on 7 November at 23:59 IST. Consider a verbatim export from the app as the authoritative record, since this file is a summary.
9. When the CA joins, the reviewer handle is already in CODEOWNERS (RAMYAVIKRAM).
10. Scale to other countries (20 to 30) only after India is validated.

## 9. Confidentiality note
This summary contains no account numbers, passwords or tokens. It refers to the owner and reviewer by GitHub username and does not include personal identifiers beyond those.
