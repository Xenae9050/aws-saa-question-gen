---
description: Triage an AWS SAA-C03 study-notes section for factual errors, outdated information, gaps, and ambiguity, with mandatory live verification against official AWS sources.
---

# Task: Triage and verify AWS SAA-C03 study notes

Review the specified study-notes section for factual errors, outdated information, incomplete explanations, ambiguity, and exam-relevant omissions.

The notes are approximately two years old. Your general knowledge may be outdated or incomplete. **You must perform live verification against official AWS sources during this triage task. Do not rely on pretrained knowledge alone.**

## Inputs

- Notes section: the file explicitly provided in the current request.
- Repository instructions: follow `.github/copilot-instructions.md` if available.
- Output: save findings to the corresponding file under `review/findings/`, using the same section identifier as the input.

Do not scan unrelated sections or the entire workspace unless necessary to resolve a specific issue.

## 1. Source and live-verification requirements

Use live web search and inspect the relevant source pages. Use only official AWS sources to establish whether a finding is valid:

1. **Technical accuracy:** official AWS service documentation, API references, FAQs, What's New announcements, architecture guidance, or other relevant AWS publications.
2. **Exam coverage:** the latest official AWS Certified Solutions Architect – Associate (SAA-C03) exam guide and official AWS certification information.

Official starting points:

- AWS documentation: https://docs.aws.amazon.com/
- AWS certification: https://aws.amazon.com/certification/

Locate the current official SAA-C03 exam guide rather than assuming an older guide is still current.

For every proposed finding:

- Search for relevant official AWS evidence.
- Open and inspect the relevant source page, not just its search-result snippet.
- Record the source title and direct URL.
- Identify the specific statement, documentation section, or other evidence supporting your conclusion.
- Explain how that evidence relates to the original note.
- Distinguish verified facts from your interpretation.

Do not use third-party blogs, forums, exam-prep websites, or your general knowledge as proof. Do not claim to have live-verified a finding if you could not access and inspect the relevant source.

If live web access is unavailable, stop the verification portion and clearly report that limitation. You may identify preliminary candidates from the notes, but label them UNVERIFIED and do not present them as confirmed findings.

## 2. What to examine

Read the complete provided section and identify only meaningful issues:

- **ERROR:** A demonstrably incorrect technical statement.
- **OUTDATED:** A statement that is no longer accurate due to a verified change in AWS services, behaviour, limits, recommendations, or terminology.
- **INCOMPLETE:** Missing context that materially affects understanding, technical correctness, or an important SAA-C03 concept.
- **AMBIGUOUS:** Wording that could reasonably lead to an incorrect understanding.
- **EXAM SCOPE:** Content that may no longer be relevant to the current SAA-C03 exam, or an important exam objective that is inadequately covered.
- **STYLE:** A clarity or organisation problem that materially affects learning. Do not prioritise minor stylistic preferences.

Prioritise technical correctness and exam relevance. Do not rewrite the notes simply to make them sound more modern.

## 3. Rules for interpreting exam scope

Use the current official SAA-C03 exam guide as the authority for exam-scope conclusions.

- Match concepts semantically, not only by exact service name.
- Recognise that a service or feature may fall under a broader exam objective without being named explicitly.
- Do not conclude that a topic is excluded merely because its exact name is absent from the guide.
- Distinguish explicit coverage, reasonable coverage under a broader objective, and weak or uncertain alignment.
- Do not confuse a topic being outside the explicit exam scope with its being technically obsolete or incorrect.
- Do not claim that AWS has removed a topic from the exam unless the current official material supports that conclusion.

## 4. Evidence and confidence

Assign every candidate one verification status:

- **CONFIRMED:** Live-inspected official AWS evidence supports the finding.
- **UNVERIFIED:** The claim has not been adequately checked against an accessible official source.
- **UNRESOLVED:** Official sources were inspected, but the available evidence does not support a reliable conclusion.

Also assign confidence: HIGH, MEDIUM, or LOW.

Confidence refers to confidence in the finding, not how confident the original note sounds.

A finding cannot be CONFIRMED without a source URL and a specific explanation of the supporting evidence. If sources appear to conflict, describe the conflict and mark the finding UNRESOLVED until it can be resolved.

Do not invent citations, URLs, source content, service behaviour, or exam requirements.

## 5. Output format

Save findings to the designated file under `review/findings/`. Use Markdown and the following structure.

# Triage: [section name]

## Summary

- Section reviewed:
- Official exam guide checked, including version/date if available:
- Live verification status:
- Number of findings by category and verification status:
- Important limitations:

## Findings

For each finding, include:

### [Finding ID] — [Short title]

- **Category:** ERROR / OUTDATED / INCOMPLETE / AMBIGUOUS / EXAM SCOPE / STYLE
- **Verification status:** CONFIRMED / UNVERIFIED / UNRESOLVED
- **Confidence:** HIGH / MEDIUM / LOW
- **Location:** Heading and identifiable passage in the notes
- **Original claim:** Quote the minimum text necessary to identify the issue
- **Issue:** Explain what may be wrong or missing
- **Official evidence:** Summarise the relevant evidence actually inspected
- **Source:** Source title and direct URL
- **Assessment:** Explain why the evidence does or does not support the finding
- **Suggested correction:** Provide a concise proposed correction only when sufficiently supported; otherwise state what needs further verification
- **SAA-C03 relevance:** HIGH / MEDIUM / LOW, with a brief reason
- **Further action:** None / Verify further / Review exam-scope interpretation

Assign stable, unique IDs, such as `FOUND-001`, within the section.

If there are no findings in a category, do not invent one. Avoid duplicating multiple findings for the same underlying issue.

## 6. Efficiency and scope control

- Review one supplied section at a time.
- Focus research on claims that are incorrect, potentially stale, materially incomplete, ambiguous, or relevant to current exam coverage.
- Do not research every sentence that appears correct.
- Do not perform extensive research into tangential AWS services or newer features that are irrelevant to the original claim.
- Keep evidence concise but sufficient for another person to verify the conclusion.
- If an authoritative source cannot resolve an issue efficiently, record the limitation rather than spending excessive effort speculating.
- Preserve correct explanations and the author's existing learning style.

## 7. Strict file-safety rules

- Do not modify, rewrite, rename, or delete the original notes.
- Do not apply any suggested corrections.
- Do not modify unrelated files.
- Before writing the findings file, check whether it already exists.
- If it exists, do not overwrite it silently. Save the new version to a clearly named alternative or report the conflict and ask for direction.
- At completion, report the output file path, the number of findings, how many were live-verified, and any limitations.

The task is complete only when the findings have been saved successfully and the verification limitations are explicitly reported.
