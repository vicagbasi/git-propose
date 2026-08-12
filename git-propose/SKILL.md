---
name: git-propose
description: Review full Git working tree changes and propose one or more safe, reviewable commit messages with mandatory title + 5-10 line idiomatic body plus commit ordering. Use when the user asks for "git propose", asks how to split current changes into commits, or wants Conventional Commit messages from staged, unstaged, and untracked changes.
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

- **Title (subject line) — 50/72, imperative, Conventional Commits:** First line `type(scope): subject`. Limit to ~50 chars (hard max 72); `git log --oneline` truncates >72 (GitHub ellipsis), `git shortlog`/`rebase` use subject as heading, and `git format-patch` puts subject on email Subject. Capitalize first word, no trailing period. Use imperative present tense — must complete the sentence `If applied, this commit will <subject>` (e.g., `Add feature` not `Added`/`Adds`/`Fixes`). Stretch from 50 toward 72 only when clarity requires. Token `type` via `feat|fix|refactor|docs|chore|test|perf|style|ci`; scope is noun in parentheses.
- **Body — MANDATORY 5-10 lines, git-idiomatic articulation (no filler):** **MINIMUM 5 LINES, MAXIMUM 10 LINES** of substantive prose (count blank lines and footers separately). Git idiom is free-form paragraphs hard-wrapped at 72; this skill enforces a 5-10 project floor on top of that idiom to guarantee signal. Each body line/paragraph must add distinct signal — no tautology, no generic platitudes, no verbatim title repeat. If you cannot write 5 substantive lines, you have not analyzed the diff deeply enough — re-examine `git diff --stat` and hunks. Hard-wrap every body line at 72 chars (git never wraps: `less -S` overflows on 80-col terminals; 72 = 80 − 4 indent left − 4 right symmetry; also `format-patch` email netiquette for nested replies). Separate `title` → one blank line → `body` → one blank line → `footers`; tools (`log`, `shortlog`, `rebase`, `format-patch`) mis-parse without those blank lines. Within body, use blank-line-separated paragraphs (pyramid: most important first). Bullets allowed: `"- "` or `"* "` + single space, blank line between bullets, hanging indent (2 spaces) for wrapped continuation lines.
- **Body content — what + why vs how (pyramid):** Focus on what and why; code already shows how. In order, cover: (1) Context/problem before this commit (what was wrong); (2) What this commit does after (effect on behavior/API); (3) Why this approach vs alternatives rejected (tradeoffs/constraints); (4) Side effects, unintuitive consequences, risks, or what is intentionally *not* changed; (5) Validation/testing/follow-ups. Make before/after explicit (`Previously … / Now …` or `Before this change … / This commit …`) so reader never guesses which state you describe. Omit how-details already evident from diff unless the diff is misleading (reindent/move) — then note it to guide the reader.
- **Evidence-tracing:** Every body claim must be traceable to `git diff --stat`, hunk content, or nearby code you inspected. Do not hallucinate motivation, validation, issue links, or breaking changes.
- **Format:** `title` → one blank line → `5-10 line body` (paragraphs + bullets, each hard-wrapped at 72) → one blank line → optional footers. Render the ENTIRE commit message (title + blank line + body + optional footers) as ONE Markdown code block.
- **Footers — git trailers (Co-authored excluded per project):** One blank line after body. Use trailer tokens `Fixes #123`, `Refs #123`, `Closes #123`, `BREAKING CHANGE: description`, or `BREAKING-CHANGE:` (hyphen synonymous with space). Breaking may also be signaled by `!` after type/scope (`feat!:`, `fix(api)!:`); then description must describe the break. Footer values may be multi-line but keep token on its own line `Token: value` or `Token #value`. Do not include `Co-authored-by` or other co-author trailers in this project.
- **File paths:** Never put file paths or line references in the commit body. File paths belong only in the per-commit `Include` / `Files included` list.
- **VALIDATION — COUNT BEFORE YOU EMIT:** Before returning ANY proposal, COUNT substantive body lines (exclude blank separator lines and footers). If <5 or >10, REWRITE until compliant with idiomatic content above. **<5 body lines is INVALID. Missing body is INVALID.**
- **Applies to EVERY commit:** This 5-10 line body rule applies to every single commit in the plan, not just the first one. No exceptions for small, trivial, or chore commits.
- **Allowed types:** `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`, `style`, `ci`. Use the type that matches the primary concern of the commit.

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

## Output

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
  - Include: explicit path/hunk list with staging bucket — never assign whole file to more than one commit
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
- Include:
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
- Include:
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
- Include:
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
- Include:
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

- [ ] Counted substantive body lines for EVERY commit: 5-10 inclusive; one blank line after title, one blank line before footers, footers not counted
- [ ] No file paths or line numbers inside any body
- [ ] Subjects are `type(scope): subject`, ~50 chars (hard 72), Capitalized, no period, imperative `If applied, this commit will …`
- [ ] Body hard-wrapped at 72, blank-line-separated paragraphs (pyramid: problem → what changed before/after → why this approach vs alternatives → side effects/risks → validation), bullets use `"- "`/`"* "` + space with blank line between and 2-space hanging indent, what+why vs how, distinct signal per paragraph/line, evidence-traceable
- [ ] Footers (if any) are trailer `Token: value` / `Token #value` one blank line after body; `BREAKING CHANGE:` or `!` syntax correct; no `Co-authored-by` per project
- [ ] `Include` lists are exact `[bucket, whole|hunk] path` and no file appears whole in two commits
- [ ] Ordering is dependency-correct and revertible
- [ ] No secrets/credentials in output; secrets excluded from commits
