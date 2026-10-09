Verify the candidate findings in the supplied review file.

The goal is to determine whether each finding is genuinely incorrect,
outdated, incomplete or misleading as of the current AWS environment.

Use current official AWS documentation as the authority where available.

For each finding:

1. Determine whether the finding is valid.
2. Explain the technical reasoning.
3. Provide the corrected/current statement if a correction is required.
4. Distinguish between:
   - technically wrong
   - technically correct but outdated
   - technically correct but incomplete
   - technically correct and should remain unchanged
5. Identify whether the change is relevant to AWS Solutions Architect
   Associate-level study.
6. Give confidence: HIGH / MEDIUM / LOW.

Do not modify the study notes.

Return only verified findings and recommended changes.

Do not manufacture a correction when the evidence is unclear.

Save findings to the designated file under `review/verified/[section-id].md.`. Use Markdown and the following format:

Compare every finding in the triage report against live-inspected official AWS sources. Preserve the original finding IDs.

1. Summary table

Include every triage finding, using one row per finding.

Finding ID Verification outcome Recommended disposition Priority Change since triage?
[ID] [CONFIRMED / PARTIALLY SUPPORTED / REJECTED / UNRESOLVED] [CORRECT / RETAIN / CLARIFY / EXPAND / REMOVE_OR_DEPRIORITISE / INVESTIGATE_FURTHER] [HIGH / MEDIUM / LOW] [YES / NO]

Definitions:

CONFIRMED: Official evidence supports the triage allegation.
PARTIALLY SUPPORTED: Some, but not all, of the allegation is supported.
REJECTED: Official evidence contradicts the triage allegation.
UNRESOLVED: Available evidence is insufficient to reach a reliable conclusion.

A finding has changed since triage if its validity, category, confidence, exam-scope assessment, recommended action, or proposed correction materially differs from the original assessment.

2. Details for changed findings only

Provide a detailed entry only when verification materially changes or disputes the triage assessment. Do not repeat full details for findings whose assessment remains unchanged.

For each changed finding, include:

[Original Finding ID] — [Short title]
Change from triage: [What changed and why.]
Original assessment: [Brief summary of the triage conclusion.]
Verified conclusion: [What the evidence establishes.]
Official evidence: [Concise explanation of the relevant evidence and how it supports the conclusion.]
Source: [Official AWS source title and direct URL; identify the relevant section or short quotation.]
Recommended disposition: [CORRECT / RETAIN / CLARIFY / EXPAND / REMOVE_OR_DEPRIORITISE / INVESTIGATE_FURTHER]
Proposed replacement: [Concise replacement text if justified; otherwise state that no replacement is proposed.]
Remaining uncertainty: [Only if applicable.]

Include an entry if the triage allegation was rejected, even if the original note does not need changing. This makes important false positives visible.

3. Verification requirements
   Live-search and inspect official AWS source pages; do not rely solely on pretrained knowledge or search snippets.
   Use current official AWS service documentation for technical claims and the current official SAA-C03 exam guide for exam-scope claims.
   Include direct URLs and evidence supporting every changed assessment.
   Do not infer that a topic is outside the exam merely because its name is absent from the guide; assess broader exam objectives semantically.
   Do not claim a finding was verified if the source could not be inspected.
   If evidence is insufficient, mark the item UNRESOLVED and explain the limitation in its detailed entry.
   For unchanged findings, keep the report concise: the summary row is sufficient.
   Do not modify the original notes or apply corrections.
   Check whether the output file already exists. Never silently overwrite an existing report.
   End with counts for each verification outcome, the number of materially changed findings, and any live-verification limitations.
