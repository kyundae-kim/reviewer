---
name: branch-code-review
description: Perform a grounded code review of a feature branch against main, validate with tests, and return prioritized actionable findings.
---

# Branch code review (Python/FastAPI-friendly)

## When to use
Use this skill when the user asks for "code review", "PR review", or "review my changes" in a git repo.

## Core outcome
Produce a review that is:
1) diff-grounded,
2) test-grounded,
3) prioritized by severity,
4) actionable with concrete file/line pointers.

## Workflow
1. Establish branch context
   - Get current branch and commit.
   - Identify review base (usually `origin/main`).
2. Scope the change
   - List changed files and diff stats.
   - Skim high-impact files first (config, dependencies, auth, routers, persistence).
3. Read critical code paths
   - Validate behavior changes, not only style.
   - Check dependency injection lifecycle and import-time side effects.
4. Validate with project-appropriate test invocation
   - Prefer the project runner (`uv run pytest`, `poetry run pytest`, etc.) over raw `pytest` when lockfile/venv tooling indicates that.
5. Classify findings
   - High: correctness/reliability/production-risk bugs.
   - Medium: maintainability or subtle behavior hazards.
   - Low: docs drift, naming clarity, minor consistency issues.
6. Return concise review
   - Include test result summary.
   - Include file path and short rationale for each finding.
   - Include clear fix direction.

## FastAPI-specific pitfalls to actively check
- Conditional checks that still force dependency initialization due to eager `Depends(...)` resolution.
  - Example class: health/readiness toggles that appear optional but still instantiate DB/MinIO clients.
- Import-time settings capture (`EnvConfig()` at module import) that freezes runtime-configurable values.
  - Example class: OAuth2 scheme token URL locked at import time.
- Docs/packaging drift after file moves
  - README says "planned" for already-implemented features.
  - package data still points to removed paths.

## Verification checklist
- Findings are backed by concrete diff/code evidence.
- Proposed fixes match the current architecture.
- Test command used is reproducible in this repository.
- Final output separates confirmed issues from optional improvements.

## References
- See `references/fastapi-core-review-signals.md` for concrete signals observed in a real FastAPI branch review.
- See `references/readiness-dependency-resolution-probe.md` for a reproducible probe to catch eager dependency resolution that bypasses readiness toggles.