---
name: deep-code-review
description: Deep code review by three parallel reviewer subagents (quality, implementation, simplification), every finding verified against the code. Use whenever code needs a review - a diff, a branch, or freshly written or refactored code.
---

# Deep code review

## 1. Fix the scope and the goal

- **Scope**: the code under review. Take it from the request; otherwise the uncommitted changes, or the branch diff against its merge-base when the tree is clean.
- **Goal**: the requirement the code is meant to satisfy. Take it from the conversation, then from commit messages or a linked issue. A goal you had to derive from the code itself is marked "inferred" in the briefs and in the report.

Done when the scope is a command or file list a subagent can run as-is, and the goal fits in one sentence.

## 2. Spawn the reviewers

Spawn three `general-purpose` subagents in one message so they run in parallel, one per reviewer file beside this skill:

- `reviewers/quality.md`
- `reviewers/implementation.md`
- `reviewers/simplification.md`

Brief each with the absolute path of its reviewer file, to read first and follow as its instructions, plus the scope and the goal.

Done when all three reports are back.

## 3. Verify every finding

Merge findings that name the same location and the same issue, keeping every source reviewer.

Then give each finding a verdict by reading the code yourself: the cited lines, enough surrounding context to follow the logic, and every place a mitigation could live (callers, upstream validation, tests).

- **confirmed**: the issue is real and unmitigated.
- **false positive**: the code does not do what the finding claims, or a mitigation already covers it.

Done when every finding carries a verdict.

## 4. Report

State the goal the review was made against. List the confirmed findings, most severe first, each with location (`file:line`), issue, impact, fix, and source reviewers. Close with the count of false positives discarded.
