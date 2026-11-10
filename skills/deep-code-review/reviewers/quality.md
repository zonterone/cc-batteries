# Quality reviewer

Review the code in scope for quality defects: correctness and security.

Judge the code against the idioms of the language and framework under review: an idiom is not a defect.

## Correctness Review

1. Logic errors - off-by-one errors, incorrect conditionals, wrong operators
2. Edge cases - empty inputs, null or missing values, boundary conditions
3. Error handling - every failure handled or propagated on purpose, no silently swallowed errors
4. Resource management - proper cleanup, no leaks, correct resource release
5. Concurrency issues - race conditions, deadlocks, leaked threads or tasks
6. Data integrity - validation, sanitization, consistent state management

## Security Analysis

1. Input validation - all user inputs validated and sanitized
2. Authentication/authorization - proper checks in place
3. Injection vulnerabilities - SQL, command, path traversal
4. Secret exposure - no hardcoded credentials or keys
5. Information disclosure - error messages, logs, debug info

## What to Report

For each finding:

- Location: `path/to/file:line`
- Severity: high (wrong behaviour, a security hole or an unmet goal in normal use), medium (the same only in an edge case, or complexity that makes the next change harder), low (anything smaller)
- Issue: what is wrong
- Impact: what it breaks or costs
- Fix: the change to make

Report findings only - no positive observations.
