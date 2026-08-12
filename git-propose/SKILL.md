---
name: git-propose
description: Review full Git working tree changes and propose one or more safe, reviewable commit messages with mandatory title + 5-10 line body plus commit ordering. Use when the user asks for "git propose", asks how to split current changes into commits, or wants Conventional Commit messages from staged, unstaged, and untracked changes.
---

# Git Propose

Turn uncommitted changes into reviewable commits based on evidence, not file
layout or a target commit count.

## State machine

**Proposal mode** is read-only: do not stage, unstage, commit, amend, reset,
restore, switch branches, or push.

An unambiguous follow-up approval—`go`, `apply`, `commit`, `proceed`, or
equivalent—may execute only the plan returned immediately before it in the
current conversation. Revalidate repository root, branch, Git-operation state,
staging, paths, and assigned hunks first. If anything material changed, return
an updated plan instead. Execution stages only approved paths/hunks, preserves
unrelated user work, uses the approved messages and order, never pushes or
switches branches, and stops on a hook or Git error.

## Gather evidence safely

Honor the user's scope; otherwise cover staged, unstaged, and untracked files.

1. Confirm the worktree/root, branch, Git-operation state, and repository
   instructions/message conventions.
2. Inventory status, including files with both staged and unstaged edits,
   conflicts, renames, generated/binary/large files, ignored files, and
   submodules.
3. Screen paths before opening content. Do not read or print likely secrets
   (`.env*`, keys, credentials, tokens, certificate exports); record only the
   path and exclude it from a proposed commit.
4. Inspect safe staged/unstaged diffs and relevant untracked text. Consult
   nearby code only when a diff cannot establish intent or dependencies.
5. If a merge, rebase, cherry-pick, revert, or unresolved conflict is active,
   provide a conditional post-resolution plan rather than ready-to-run commits.

Use repository commit style when evident; otherwise use concise Conventional
Commit subjects. **Precedence:** The mandatory title + 5-10 line body rule
below overrides any repo title-only convention. Never invent motivation,
validation, issue links, or breaking changes not supported by evidence.

Run these commands to gather evidence (when inside a work tree):

```bash
git rev-parse --is-inside-work-tree
git status --porcelain=v1
git ls-files --others --exclude-standard
git diff --staged --name-status
git diff --name-status
git diff --staged
git diff
git diff --staged --stat
git diff --stat
```

Treat staged content as intentional unless incomplete or unsafe without related
unstaged changes. Assess all change buckets before proposing commits:
- staged tracked changes
- unstaged tracked changes
- untracked files

For large diffs, start with `--stat` and `--name-status` and then inspect only
targeted `git diff -- <path>` hunks to stay within context.

## Preflight and Empty-State Rules

- Run `git rev-parse --is-inside-work-tree` first. If it fails, do not run
  remaining git-state commands.
- If not inside a git work tree, return the required output format with:
  - `Repo State Summary`: `Not a git repository`.
  - `Proposed Commit Plan`: `0 commits`.
  - `Build-Mode Notes`: assumption that no git state is available.
- If working tree is clean (no staged, unstaged, or untracked changes), return
  the required output format with:
  - `Repo State Summary`: `Clean working tree`.
  - `Proposed Commit Plan`: `0 commits`.
  - `Commit Proposals (Ready to Paste)`: `No commit proposed`.
  - Or the legacy string: `No proposal: the requested scope has no uncommitted changes.`

## Commit Message Requirements — EXTREMELY IMPORTANT — MANDATORY — READ THIS FIRST

> [!CAUTION]
> **EVERY commit proposal MUST contain BOTH a message TITLE and a message BODY. This is NON-NEGOTIABLE and EXTRAORDINARILY EXTREMELY IMPORTANT. NEVER emit a title-only commit message. A missing body is INVALID.**

- **Title (subject line):** First line, Conventional Commit format `type(scope): subject`, imperative present tense, capitalized, no trailing period, near 50 chars (stretch only when clarity requires).
- **Body:** **MANDATORY — MUST BE PRESENT — MINIMUM 5 LINES, MAXIMUM 10 LINES.** Every commit proposal MUST include a body of **at least 5 lines and at most 10 lines** (wrapped near 72 chars per line). Count blank lines and footers separately — the body itself is 5-10 substantive lines. The body MUST explain WHAT changed, WHY it was needed, HOW it was implemented, tradeoffs/alternatives considered, and validation/testing/follow-ups. Each body line must add distinct signal — no tautology, no filler repetition, no generic platitudes. If you cannot write 5 substantive lines, you have not analyzed the diff deeply enough — re-examine the changes until you can.
- **Evidence-tracing:** Every body claim must be traceable to `git diff --stat`, hunk content, or nearby code you inspected. Do not hallucinate motivation, validation, issue links, or breaking changes.
- **Format:** `title` → one blank line → `5-10 line body` → one blank line → optional footers (`Fixes #123`, `BREAKING CHANGE: ...`). Render the ENTIRE commit message (title + blank line + body + optional footers) as ONE Markdown code block.
- **File paths:** Never put file paths or line references in the commit body. File paths belong only in the per-commit `Include` / `Files included` list.
- **VALIDATION — COUNT BEFORE YOU EMIT:** Before returning ANY proposal, COUNT body lines. If body has fewer than 5 lines or more than 10 lines, REWRITE until compliant. **A proposal with <5 body lines is INVALID and MUST NOT be returned. A proposal with no body is INVALID and MUST NOT be returned.**
- **Applies to EVERY commit:** This 5-10 line body rule applies to every single commit in the plan, not just the first one. No exceptions for small, trivial, or chore commits.
- **Allowed types:** `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`, `style`, `ci`. Use the type that matches the primary concern of the commit.
- **Wrapping:** Wrap body near 72 chars; do not emit a single 400-char paragraph counted as one line.

## Design commits

Apply four invariants:

1. Keep a behavioral change with its required tests, migration metadata,
   generated companion, or maintained documentation.
2. Split only when each slice is independently truthful, reviewable, and valid.
3. Order prerequisites before dependents: foundation, contracts, providers,
   consumers, then optional hardening.
4. When a file spans slices, assign exact hunks or named regions—never assign
   the whole file to more than one commit.

Prefer fewer coherent commits over fragile micro-splits. Call out migrations,
external effects, partial staging, or non-reversible boundaries.

## Commit Strategy Rules

- Choose single commit for one cohesive concern.
- Split commits by concern when feature/refactor/fix/docs/tooling are mixed.
- Favor cohesion over raw size.
- Order by dependency and narrative:
  - enabling refactors first
  - functional changes next
  - tests with the behavior they verify
  - docs after behavior
- Keep each commit independently reviewable and preferably revertible.

## Required Output Format

Return exactly these section headings in this order for every non-trivial
proposal. For a simple single-commit plan you may omit empty subsections but
keep the heading order:

- `## Repo State Summary`
- `## Red Flags / Safety Notes (if any)`
- `## Proposed Commit Plan`
- `## Commit Proposals (Ready to Paste)`
- `## Build-Mode Notes`

Required content inside those sections:

- Repo State Summary:
  - Staged: brief summary
  - Unstaged: brief summary
  - Untracked: brief summary and include when needed for commit intent
  - Mixed staged/unstaged in same files: yes/no and impact
- Red Flags / Safety Notes:
  - Call out secrets/credentials, generated artifacts, lockfile-only drift, huge formatting churn, risky partial staging
- Proposed Commit Plan:
  - Number of commits
  - Rationale for split/merge choice
  - Why ordering is reviewable
- Commit Proposals:
  - Intent / Scope (no file paths)
  - Full commit message: render as one Markdown code block containing title + blank line + 5-10 line body + optional footers
  - Files included (required): explicit file path list to stage for the commit (not part of commit message) — use `[staged|unstaged|untracked, whole|hunk] path` granularity; never assign whole file to more than one commit
  - Depends on: none | Commit N
- Build-Mode Notes:
  - Explicit assumptions
  - Ask clarifying questions only for real risk

Use this exact Markdown skeleton (4-backtick outer fence avoids nested-fence ambiguity):

````markdown
## Repo State Summary
- Staged: ...
- Unstaged: ...
- Untracked: ...
- Mixed staged/unstaged in same files: ...

## Red Flags / Safety Notes (if any)
- ...

## Proposed Commit Plan
- Number of commits: ...
- Rationale for split/merge choice: ...
- Why ordering is reviewable: ...

## Commit Proposals (Ready to Paste)
### Commit 1 — purpose
- Intent / Scope: ...
- Full commit message:
  ```md
  type(scope): Subject line (title)

  Body line 1 — what changed and why this commit is needed
  Body line 2 — implementation approach and key files/areas touched
  Body line 3 — tradeoffs, alternatives considered, or decisions made
  Body line 4 — validation, tests, or verification performed
  Body line 5 — risks, follow-ups, or scope notes (5-10 body lines total)
  Body line 6 — additional context as needed to reach 5-10 lines

  Optional-Footer: value
  ```
- Files included:
  - [staged|unstaged|untracked, whole|hunk] path — purpose
- Depends on: none

### Commit 2 — purpose
- Intent / Scope: ...
- Full commit message:
  ```md
  type(scope): Subject line (title)

  Body line 1 — ...
  Body line 2 — ...
  Body line 3 — ...
  Body line 4 — ...
  Body line 5 — ...
  ```
- Files included:
  - [unstaged, role badge hunk] src/components/StaffRow.tsx — render the role
- Depends on: Commit 1

## Build-Mode Notes
- Explicit assumptions: ...
- Clarifying questions (only if real risk): ...
````

> [!IMPORTANT]
> **EVERY commit code block above MUST follow title + blank line + 5-10 line body structure. Title-only blocks are FORBIDDEN. Verify line count before returning.**

When evidence or safety is insufficient, label it `Conditional plan` and say
what must be resolved.

## Compact example

For a new API field whose UI use depends on the server contract, a useful plan
looks like this:

````markdown
## Repo State Summary
- Staged: none
- Unstaged: src/api/staff.ts, src/api/staff.test.ts, src/components/StaffRow.tsx, src/components/StaffRow.test.tsx
- Untracked: none
- Mixed staged/unstaged in same files: no

## Red Flags / Safety Notes (if any)
- none

## Proposed Commit Plan
- Number of commits: 2
- Rationale for split/merge choice: API contract must land before UI consumption; tests stay with behavior
- Why ordering is reviewable: Commit 1 is revertible foundation, Commit 2 depends on it

## Commit Proposals (Ready to Paste)
### Commit 1 — expose the staff role in the API
- Intent / Scope: Add `role` to staff response contract
- Full commit message:
  ```md
  feat(api): expose staff role in API response

  Add `role` field to the staff response contract and database projection.
  Required for downstream UI to render role badges correctly.
  Expose via existing staff serializer rather than new endpoint to keep API surface minimal.
  Alternative of separate lookup endpoint rejected due to extra round-trip cost.
  Verified with contract tests covering new field presence and serialization.
  No breaking change; field is additive and defaults to existing behavior for old clients.
  ```
- Files included:
  - [unstaged, whole] src/api/staff.ts — add `role` to the response contract
  - [unstaged, whole] src/api/staff.test.ts — verify the contract
- Depends on: none

### Commit 2 — display the staff role
- Intent / Scope: Render role badge in roster
- Full commit message:
  ```md
  feat(staff): display staff role badge in roster

  Render the newly exposed `role` as a badge in the StaffRow component.
  Reuse existing Badge primitive to keep styling consistent with roster.
  Considered tooltip-only display but badge improves scanability per design review.
  Covered by component tests asserting badge renders for each role variant.
  Follows Commit 1; no independent fallback needed since API field is now present.
  ```
- Files included:
  - [unstaged, role badge hunk] src/components/StaffRow.tsx — render the role
  - [unstaged, whole] src/components/StaffRow.test.tsx — verify the badge
- Depends on: Commit 1

## Build-Mode Notes
- Explicit assumptions: none
- Clarifying questions (only if real risk): none
````

The shared component file is assigned by named hunk rather than duplicated
across commits; tests stay with the behavior they establish. Each commit message
above demonstrates the MANDATORY title + 5-10 line body structure — title-only
messages are NEVER acceptable.

## Ambiguity Handling

- In build-like execution, continue with best-effort proposals and state assumptions in `Build-Mode Notes`.
- Ask a question only when proceeding may introduce material risk (secrets, data loss, non-reversible migration).
- When evidence or safety is insufficient, label the plan `Conditional plan` and enumerate what must be resolved before execution.

## Validation Checklist — Run Before You Emit

- [ ] Counted body lines for EVERY commit: 5-10 inclusive, blank line after title, footers not counted
- [ ] No file paths or line numbers inside any body
- [ ] Subjects are Conventional Commit, imperative, near 50 chars, no period
- [ ] Body wrapped near 72 chars, each line adds distinct signal, evidence-traceable
- [ ] `Files included` / `Include` lists are exact `[bucket, whole|hunk] path` and no file appears whole in two commits
- [ ] Ordering is dependency-correct and revertible
- [ ] No secrets/credentials in output; secrets excluded from commits
