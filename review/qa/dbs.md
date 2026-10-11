# QA Report: Databases

## Summary

- **Scope reviewed:** `notes/edited/dbs.md` only, via `diff notes/working/dbs.md notes/edited/dbs.md` (all 14 hunks), against `review/verified/dbs.md` and `review/editing/dbs.md`, with surrounding context read for each changed passage. No new AWS research was done. Evidence was taken from the verification report's cited sources, and I did not re-inspect those pages.
- **Overall result:** PASS WITH ISSUES
- **Changes reviewed:** 11 applied (001, 002, 003, 005, 006, 007, 008, 010, 011, 012, 013) plus 3 not-applied findings (009, 014, 015) checked for compliance.
- **Outcomes:** PASS 4 · PASS WITH NOTE 7 · FAIL 0 · NEEDS VERIFICATION 0 · NOT APPLICABLE 3
- **Unauthorised changes:** None. Every diff hunk maps to a finding. `notes/original/` and `notes/working/` are unmodified. The only uncommitted change in the repo is `.github/prompts/04-qa.prompt.md`.
- **Application report inconsistency:** `review/editing/dbs.md` says "Changes applied: 11" but lists 10 IDs without 011. It also counts 011 as both applied (by user request) and manual review. I could not confirm the user request from the available files.

## Results

| Finding ID | Outcome | Issue / evidence | Recommended action |
| --- | --- | --- | --- |
| FOUND-001 | PASS WITH NOTE | Instance vs cluster Multi-AZ split matches the evidence, and the table header is qualified. The added "Aurora clusters are different: shared storage across 3 AZs" is not in the verification evidence, which only says Aurora DB clusters are not the same. It is likely true. | Confirm the Aurora storage wording or reduce it to "Aurora clusters are different". |
| FOUND-002 | PASS WITH NOTE | Uses the proposed replacement. "clients must connect using TLS" sits awkwardly with "enforcement is engine-specific" and could read as TLS being mandatory. | Optional: "clients connect using TLS". |
| FOUND-003 | PASS | The bullet and table row match the evidence (0 ACUs on newer versions, 0–256, version dependent). 2 GiB/ACU kept. | None. |
| FOUND-005 | PASS WITH NOTE | Multi-AZ for RA3/RG is correct. The sentence "Would need to run clone in different AZ manually" was dropped. It still applies to single-AZ (e.g. DC2) clusters, and the report did not authorise its removal. | Optional: restore it for single-AZ clusters. |
| FOUND-006 | PASS WITH NOTE | The node types and the 1–32 / up to 128 node bullets match the report. "Start at large \- no smaller" is unchanged and now sits under RA3/RG/DC2 where it is unverified. | See required action 1. |
| FOUND-007 | PASS | 15 per source matches the Quotas page. The report notes the PostgreSQL page gives no engine-specific figure. | None. |
| FOUND-008 | PASS | 5 → 10 matches the Aurora Global Database page. | None. |
| FOUND-010 | PASS WITH NOTE | "Serverless Provisioned" is removed correctly. "traditional/default" keeps "default", which the evidence does not state (it says "oldest and most common"). | See required action 2. |
| FOUND-011 | PASS WITH NOTE | The one-line shutdown note matches the proposed text and is supported (full shutdown 31 Jul 2025). The disposition was REMOVE_OR_DEPRIORITISE, which the apply rules say never to auto-apply. The application report claims a user request that I cannot verify. No other QLDB references remain in the file. | Confirm the user authorisation. Reconcile the application report counts. |
| FOUND-012 | PASS WITH NOTE | Valkey/Redis OSS wording is supported. (a) The public endpoint caveat omits the evidence qualifiers (Valkey 9.0+, IAM auth and TLS 1.3 required), so it may overstate availability. (b) The MemoryDB sentence "Microsecond reads and single-digit millisecond writes as a trade-off" is ambiguous, because microsecond reads are not a trade-off, and the ElastiCache comparison was dropped. The report lists the ElastiCache write-latency comparison as unverified. | See required actions 3 and 4. |
| FOUND-013 | PASS | The capacity-mode bullet matches the evidence. The 1000 WCU line and the unverified items (10 GB split, DAX/Global Tables) were correctly left unchanged. | None. |
| FOUND-009 | NOT APPLICABLE | Skipped (LOW). "Unlimited requests per second" (line ~725) is unchanged, which is consistent with the apply rules. | Left open in verification (claim unsupported). |
| FOUND-014 | NOT APPLICABLE | Skipped (LOW). The Strongly consistent bullets (lines ~845–849) are unchanged, which is consistent. | None. |
| FOUND-015 | NOT APPLICABLE | Manual review. The Glue Studio "AWS Code Commit" bullet (line ~207) is correctly untouched. The QLDB part was handled under 011. | Author decision. |

## Required actions

1. `notes/edited/dbs.md` → Amazon Redshift → Node types: "Start at large \- no smaller" is unverified for RA3/RG node sizes. Verify against the Redshift node-type tables, or qualify it as "(DC2)" if it is not correct for RA3.
2. `notes/edited/dbs.md` → Aurora (line ~667): "traditional/default" is not supported by the evidence. Minimum fix: "traditional/most common". Alternatively, verify that "default" is accurate.
3. `notes/edited/dbs.md` → ElastiCache (line ~10): add the qualifier that the public endpoint applies to serverless caches with IAM auth and TLS (Valkey 9.0+) per the evidence, or mark it for verification.
4. `notes/edited/dbs.md` → Amazon MemoryDB (line ~61): reword the "trade-off" sentence so it is clear what is being traded. Do not assert the ElastiCache write latency until it is verified.
5. `review/editing/dbs.md`: correct the summary counts and the 011 applied/manual-review inconsistency. This is an application-report fix, not a notes fix.

No edits were made during QA.
