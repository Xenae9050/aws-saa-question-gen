# QA Report: Messaging

## Summary

- **Scope reviewed:** `notes/edited/messaging.md` only, via `diff notes/working/messaging.md notes/edited/messaging.md` (3 hunks), against `review/verified/messaging.md` and `review/editing/messaging.md`, with surrounding context read for each changed passage. The only external page inspected during QA was the SQS message quotas page (see FOUND-003 note). Other evidence is taken from the verification report's cited sources, which I did not re-inspect.
- **Overall result:** PASS WITH ISSUES
- **Changes reviewed:** 3 applied (FOUND-003, FOUND-006, FOUND-013 candidate 2). Non-applied findings were checked for compliance only.
- **Outcomes:** PASS 2 · PASS WITH NOTE 1 · FAIL 0 · NEEDS VERIFICATION 0 · NOT APPLICABLE 10 (findings 001, 002, 004, 005, 007–012, none changed in the notes)
- **Unauthorised changes:** None. Every diff hunk maps to an applied finding. `notes/original/` and `notes/working/` are unmodified. The only uncommitted change in the repo is `.github/prompts/04-qa.prompt.md` (plus the untracked `review/qa/` and a sources PDF).
- **Application report inconsistency (minor):** it says 8 findings were skipped for ambiguity but lists 9 IDs (001, 002, 004, 005, 007, 008, 009, 010, 011). Notes are unaffected.

## Results

| Finding ID | Outcome | Issue / evidence | Recommended action |
| --- | --- | --- | --- |
| FOUND-003 | PASS WITH NOTE | Text matches the verified replacement exactly (256KB default, 1MiB via `MaximumMessageSize`, Firehose/SQS/Lambda only above 256KB, Extended Client Library above 1MiB up to 2GB, uses S3). The new wording says SQS subscriptions are allowed above 256KB, but the SQS section (line 185) still says "Message size = 1 byte -> 256KB… Extended Client Library required for larger messages". Verified FOUND-002 (SQS 256KB → 1MiB, HIGH priority, report supplied suggested wording in the triage report but not the verification report) was not applied, so the two sections now read inconsistently. I did not confirm the new SQS limit during QA: the quotas page fetch was truncated before the message size entry. | See required action 1. |
| FOUND-006 | PASS | A2P list "SNS" → "SMS" is a single-word fix. It is consistent with the destinations list ("Mobile phone numbers") and the Subscriptions protocol list (line 80: SMS). | None. |
| FOUND-013 (cand. 2) | PASS | The SNS FIFO bullet carries the 5-minute dedup interval, dedup ID/content-based dedup, the consumer-deletes-before-visibility-timeout condition, and at-most-once with filter policies, all matching the verified evidence. The "exactly once" claim is preserved but qualified. The other bullets in the section are unchanged and still consistent. Candidates 5, 6 and 7 were correctly not applied (unresolved or LOW, with no replacement authorised). | None. |

## Required actions

1. **`notes/edited/messaging.md` > SQS (line ~185, "Message size")**: the SNS note now implies SQS can receive payloads above 256KB, while the SQS note still states 256KB as the limit. Apply verified FOUND-002 (triage report suggests: "1 byte -> 1 MiB (1,048,576 bytes). Extended Client Library required above 1 MiB (max 2GB).") or confirm the current SQS maximum message size from the official SQS quotas page and update the line. The minimum change is that one sentence. It was deferred only because the verification report gave no replacement wording. Manual review is needed.
