---
description: Conservatively apply evidence-backed corrections to one AWS SAA-C03 study-notes section.
---

# Task: Apply verified corrections to AWS SAA-C03 study notes

Apply only justified, sufficiently supported corrections from the verification report to the corresponding working notes section.

This is a controlled editing task, not a new research or rewriting task. Prioritise technical accuracy, minimal changes, preservation of correct content, and low token usage.

## Inputs

- Notes section: explicitly supplied file under `notes/edited/`
- Verification report: corresponding file under `review/verified/`
- Repository instructions: `.github/copilot-instructions.md`, if available
- Output: edit the supplied working notes section in place, and save an application report under `review/editing/`

Do not scan unrelated sections or read the entire repository unless necessary.

## 1. Eligibility rules — follow strictly

Use the verification report's outcome, confidence, and recommended disposition together. A proposed correction in the report is not automatically approved.

### Automatically eligible, subject to an unambiguous correction

- `CONFIRMED` + HIGH confidence + `CORRECT`, `CLARIFY`, or `EXPAND`: apply if the proposed change is explicit, technically coherent, and supported by the cited official evidence.
- `CONFIRMED` + MEDIUM confidence: apply only when the evidence directly supports the exact change and the replacement is unambiguous.
- `PARTIALLY SUPPORTED`: apply only a narrow correction to the portion explicitly supported by the evidence. If separating the supported portion requires interpretation, skip it for review.

### Never automatically apply

- LOW-confidence findings, regardless of verification outcome.
- `UNRESOLVED` findings.
- `REJECTED` findings: do not apply the proposed correction. Retain the original note unless another independently verified issue justifies a change.
- `INVESTIGATE_FURTHER` dispositions.
- `REMOVE_OR_DEPRIORITISE`: flag for manual review, even if the content appears outside explicit exam scope.
- Findings with missing or inaccessible source evidence, conflicting evidence, or no clear proposed correction.
- Changes requiring substantial rewriting, architectural judgement, or assumptions not established by the verification report.

If the outcome, confidence, and disposition conflict, choose the more conservative action: do not edit and flag the finding for review.

Do not lower these thresholds to increase the number of applied changes.

## 2. Editing principles

For each eligible finding:

1. Locate the exact passage in the specified notes file.
2. Confirm that the passage still matches the claim in the verification report.
3. Make the smallest change that resolves the verified issue.
4. Preserve correct explanations, useful examples, terminology, heading structure, links, formatting, and the author's learning style.
5. Retain useful context and caveats unless the evidence shows they are incorrect.
6. Do not add unrelated material, new services, tangential details, or speculative exam content.
7. Do not rewrite whole sections when a sentence-level change is sufficient.
8. Do not invent technical details or introduce claims unsupported by the verification report.
9. If the proposed replacement no longer fits the surrounding text, make a minimal adjustment only if the intended meaning remains clear and evidence-backed; otherwise flag it for review.
10. Do not modify the original source file under `notes/original/`.

## 3. Handling exact replacements

Prefer exact, local edits. Before replacing text, check that the target passage uniquely identifies the intended location.

- If the text is missing or has changed since verification, skip the finding and report the mismatch.
- If the same text occurs in multiple places, do not replace all occurrences blindly. Identify the intended passage using its heading and context.
- If applying the change would break a list, table, code block, link, or other formatting, preserve the structure.
- Do not make broad find-and-replace operations across the repository.

## 4. Application report

Create `review/editing/[section-id].md`. Check whether the report already exists; never silently overwrite it.

Use this concise format:

# Application Report: [Section Name]

## Summary

- Notes file:
- Verification report:
- Findings reviewed:
- Changes applied:
- Skipped for low confidence:
- Skipped for unresolved or conflicting evidence:
- Skipped for ambiguity or text mismatch:
- Flagged for manual review:

## Results

| Finding ID | Action                            | Reason              |
| ---------- | --------------------------------- | ------------------- |
| [ID]       | APPLIED / SKIPPED / MANUAL REVIEW | [Brief explanation] |

For each applied change, record:

- The heading or location.
- A short description of the change.
- The verification finding ID supporting it.

For skipped or manually reviewed findings, explain only why no automatic change was made. Do not reproduce the full verification report.

## 5. Safety and completion checks

- Do not edit any file other than the supplied notes section and the new application report.
- Do not change verification reports, correction ledgers, original notes, or unrelated sections.
- Do not perform additional research unless needed to understand an already verified correction. If new research is needed, flag the finding instead of expanding the task.
- Before finishing, inspect the diff for the edited section.
- Confirm that every change maps to an eligible verification finding.
- Check for accidental deletions, formatting damage, duplicated text, or changes beyond the intended scope.
- If the diff contains unrelated or unsupported changes, revert those changes before reporting completion.
- If no changes qualify, leave the notes unchanged and report that no corrections were applied.
- Do not claim success if saving or verification of the diff failed.

At completion, report the application-report path, number of changes applied, number skipped, and any items requiring manual review.

Edit the file given, do not create a new one alongside it for the edited version.
