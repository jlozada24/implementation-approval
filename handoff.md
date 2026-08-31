# Implementation Approval Workflow (Proposal-Before-Edit)

## Authority and purpose

This workflow is the **mandatory gate** for all human-driven agent work that would mutate governed repository state. It exists to ensure:

- The user reviews **intent, scope, and concrete change shape** before anything lands on disk.
- Agents do not **silently implement**, **scope-creep**, or **partially execute** without an explicit go-ahead.
- Scope changes mid-flight are **re-negotiated**, not assumed.

**Source of truth:** `AGENTS.md` → **Implementation approval workflow**. User IDE overlays may be stricter but must not contradict this workflow.

**Communication mode does not override this:** Direct technical mode, conciseness, and code-only output preferences apply **around** the workflow — they do **not** waive proposal or approval requirements.

---

## Scope: what triggers the gate

### In scope — proposal required before action

The workflow applies to **any write** to **tracked repo content** this policy governs, including but not limited to:

- Source files (Swift, ObjC, C, C++, headers)
- Xcode project / workspace / scheme files
- Package manifests and generated code under version control
- Entitlements, build configs, installer scripts that drive builds
- Policy markdown governed by the repo (`AGENTS.md`, task docs when edited as deliverables, etc.)
- Ignore-file matrix edits (`.gitignore`, `.cursorignore`, `.claudeignore`)
- `git add`, `git commit`, `git push`, branch creation, index removal — **each requires its own explicit approval after outcomes are explained**, except where a standing task-end exception applies (see Git section)
- Destructive filesystem actions inside the repo (mass delete, broad overwrite)
- Delegating edit-capable work to subagents/MCP workers without prior user approval of the combined proposal

### Out of scope — scratch-only exception

**Exception:** writes under `Scratch/` **only when used as scratch**, per `Tasks/ORDER.md`:

- Allowed repo-local scratch paths: `Scratch/<slug>--evidence/` and `Scratch/<slug>--discard/`
- All `Scratch/` contents must remain **untracked**
- Non-retained transient output belongs in OS temp storage, not ad-hoc repo paths

**Still requires proposal:** any write **outside** the declared scratch contract, or any tracked-file edit even if "related to" a scratch task.

### Special cases

- **`CLAUDE.md`:** bootstrap stub only; any edit requires explicit user approval; do not work around OS-level write locks.
- **Question-only / review-only turns:** if the user is only asking for explanation or review, do not edit unless they explicitly request changes — and then run the full workflow before editing.

---

## The three-phase workflow

Every governed change follows exactly these phases. **Never skip or reorder.**

### Phase 1 — Draft in chat only

**Before any** of the following:

- `apply_patch` / file writes / creates / deletes
- Terminal commands that mutate repo files
- `git add` / `commit` / `push` / branch ops
- Spawning edit-capable delegated workers

…the agent must produce a **complete proposal entirely in chat** (or in a clearly linked continuation message that the user can treat as one combined proposal).

**Hard prohibition:** **No silent implementation first.** Do not "just fix it quickly" and show the diff afterward. Do not write to disk "to explore." Do not delegate a builder before approval.

### Phase 2 — Seek explicit approval

After presenting the proposal, **stop**.

Do not proceed until the user grants approval for **that combined proposal** using an accepted approval token (see Approval language).

If the user:

- **Approves** → proceed to Phase 3 for the approved scope only.
- **Narrows scope** → treat as a scope change; **re-propose** if the delta is material.
- **Redirects** (different approach, different files, different behavior) → **re-propose** before editing if the delta is material.
- **Rejects** (`no`, `n`, or equivalent) → do not edit; optionally offer a revised proposal.
- **Asks questions** → answer; remain in Phase 2 until explicit approval for implementation.
- **Approves only part of a menu** → only that part is authorized (see Numbered menu semantics).

**Material delta** (always re-propose):

- Different files or symbols than allowlisted
- Behavior change beyond what was described
- New public/API/CLI surface not in the original proposal
- Verification lane change that affects claims or scope
- Architecture change (funnel, dispatch, catalog, RT path, etc.)
- Expansion from investigation-only to implementation
- Any fix discovered during an audit/read-only task that was not in the approved defect list

**Immaterial delta** (may proceed without full re-proposal only if it is truly trivial clarification within the same allowlist — when in doubt, re-propose):

- Typo fix in the same symbol the user already approved
- Identical change with clearer naming, same files/symbols/behavior

When uncertain whether a delta is material: **re-propose**.

### Phase 3 — Execute

Only after Phase 2 approval:

1. Apply edits **within the approved allowlist**
2. Verify the result (build/test/verification per task and repo rules)
3. Report outcome honestly (implemented / validated tier / pending / blocked)

**Do not exceed approved scope.** Off-path defects found during execution: **report**, do not fix unless the user expands scope and a new proposal is approved.

---

## Proposal content requirements

A proposal must be **reviewable as if the code already existed** — the user should be able to say yes/no without reading a surprise diff.

### Mandatory elements

Every proposal must include **all** of the following:

1. **Reason / intent**
   - What problem is being solved or what deliverable is being produced
   - Why this approach (brief; cite rules/architecture when relevant)

2. **Exact file allowlist**
   - Every file path that will be created, modified, moved, or deleted
   - Use real repo-relative or absolute paths under the governed repo root
   - If generated files will change, name the generator inputs and outputs

3. **Exact symbol allowlist**
   - Types, protocols, functions, methods, properties, enums, macros, constants
   - For each: create / modify / delete / rename
   - Entry points affected (CLI verbs, Actions, GUI affordances, XPC operations)

4. **Proposed code changes**
   - Concrete enough to review **before** disk write
   - Prefer showing the actual changed regions, new signatures, or pseudocode tight enough to be unambiguous
   - For large files: changed regions with clear elision markers; not "I'll update the file"

5. **Behavior contract**
   - What will change for users, CLI, GUI, engine, persistence, RT paths
   - What will **not** change (explicit non-goals help prevent scope drift)

6. **Rules / architecture compliance** (when touching governed architecture)
   - How the plan respects applicable project rules
   - Cite `file:line` patterns to follow at critical touchpoints

7. **Verification plan**
   - Scheme(s) for build
   - Test scope
   - Verification tier: local / cloud / workspace-live / live-installed / installer-hardware
   - What evidence will be produced

8. **Risks and parked questions**
   - Only genuine ambiguity that rules and precedent cannot settle
   - Do not park to avoid doing approved work

### Optional but recommended elements

- **Out-of-scope list** — files/symbols explicitly not touched
- **Dependency order** — if multi-file change must land in sequence
- **Rollback note** — if change is hard to revert
- **Off-path findings** — observed but not fixing without separate approval

### Proposal quality bar

**Insufficient proposals (reject your own draft and expand):**

- "I'll fix the bug in AUScanService"
- "Update the settings UI"
- "Refactor to match style" without file/symbol list
- "Same as last time" when last approval was for different scope
- Delegating to a builder with only a one-line task string

**Sufficient proposals:**

- Named files + symbols + signatures + behavior delta + verify plan
- User can answer yes/no without follow-up archaeology

### Same-message rule

The **proposed code changes** and the **approval request** must appear in the **same message**, or in a **clearly linked continuation** the user treats as one combined proposal.

Splitting "here's the plan" and "approve?" across turns without the code detail in the combined thread violates the workflow.

---

## Approval language

### User approval tokens (case-insensitive intent; prefer exact forms)

The following count as explicit approval **for the combined proposal just presented**:

| Token | Meaning |
|-------|---------|
| `approved` | Full approval |
| `proceed` | Full approval |
| `yes` | Full approval (see menu semantics if a numbered menu was shown) |
| `y` | Full approval |

**Not approval by themselves:**

- Silence
- "Looks good" without clear implement intent (when ambiguous, ask)
- "Continue researching"
- "Tell me more"
- Thumbs-up emoji alone (when ambiguous, confirm)
- Prior approval for a **different** proposal
- Approval of step 2 orchestrator restatement without step 4 proposal approval

### Numbered menu semantics

When the agent presents a **numbered menu** of options:

| User input | Meaning |
|------------|---------|
| `12` | Run option 1, then option 2 (concatenated digits = order) |
| `135` | Run options 1, 3, 5 in that order |
| Single digit `N` | Run option N only |
| `yes` / `y` / `0` | Run **all** options 1…N in order |
| `no` / `n` | Run none |
| `c` | Emit one copiable `bash` block with explicit `cd` to repo root; not implementation approval |

**Critical:** Numbered menu selections count as approval **for those options only** — not for unlisted work.

### Delegation approval suffixes

When spawning **edit-capable** delegated workers (Codex builder, Antigravity builder, antigravity-delegate), the orchestrator's task prompt must end with:

```
Approved. Proceed.
```

Alternate form used in task briefs and Codex reply:

```
APPROVED, PROCEED
```

**Meaning:** the orchestrator has already obtained user approval for the combined proposal; the delegate must **apply patches immediately** and must **not** stop for a second proposal gate or ask the user again.

**When to include:**

- Builder deploys after user approved step 4 (orchestrator mode)
- Audit-fix builder rounds after user approved the fix scope
- `codex-reply` when user already approved and Codex hit an approval gate

**When NOT to include:**

- Read-only auditors (`NO EDITS. REPORT ONLY.`)
- Researcher phase before user approval
- Any worker where user has not yet approved the combined chat proposal

### Codex MCP approval gate handling

When Codex stops for repo approval:

1. Summarize the proposal briefly **only if** the user has not already approved.
2. If the user's message already means "go do it", `codex-reply` with `APPROVED. PROCEED.` plus clarifications.
3. Do not re-implement the task yourself unless Codex fails or the user asks you to take over.

---

## Re-proposal rules

**Always re-propose before editing when:**

- User narrows scope ("only fix X, not Y")
- User redirects approach ("do it via Action instead of direct SDK call")
- User adds requirements ("also add CLI coverage")
- Material delta from the approved allowlist
- Discovery invalidates the approved plan (wrong root cause, wrong file)
- Audit finds defects outside the approved defect list
- Task was read-only / evidence-only and now needs source edits
- User approval was for a menu subset and new work is outside that subset

**Re-proposal format:**

- State what changed since the last proposal (delta section)
- Present a **new combined proposal** (not a partial patch on the old one)
- Wait for fresh approval tokens

**Do not** treat "approved" from an earlier, superseded proposal as valid for new scope.

---

## Git operations and approval

Implementation approval is **separate from but related to** git approval:

- **Default:** no `git add`, `commit`, `push`, or branch ops without **explicit user approval for each**, after outcomes are explained.
- **Standing task-end exception (user-approved):** agents may commit and push completed task-end commits to `main` without per-push approval when the project's verification gate requires it.
- **Push scope exception:** push only when the commit contains build-impacting codebase changes; docs/planning/artifacts are commit-only, never pushed under the exception unless user explicitly requests push.

**Workflow interaction:**

1. Propose the **code/policy changes** → get approval → implement
2. Propose or explain the **git outcome** (what will be committed/pushed) → get approval if not covered by task-end exception → execute git

A user approving implementation does **not** automatically approve unrelated git operations (force push, branch delete, history rewrite).

---

## Orchestrator mode extensions

When orchestrator mode is active, the workflow adds **structured gates**:

### Gate A — Step 2: Restate + Rules framing

Before researcher deploy:

- Restate user intent
- Frame applicable project rules for code work
- **WAIT APPROVAL**

This approval is **not** implementation approval — it authorizes research only.

### Gate B — Step 4: Researcher proposal

Researcher deliverable must use this structure:

1. **Task restatement**
2. **Scope** — files/symbols in and out; scheme(s)
3. **Implementation plan** — end-to-end; no phased deferrals; no half measures
4. **Rules compliance** — with `file:line` citations at critical touchpoints
5. **Chat proposal + explicit approval** — reason + exact file/symbol allowlist
6. **Verify plan** — scheme, build script, test scope
7. **Risks / parked questions** — genuine ambiguity only

Present section 5 (the chat proposal) to the user → **WAIT APPROVAL** → only then deploy builder.

### Gate C — Audit-fix rounds

When verified findings require fixes:

- New fix scope may be presented as a focused proposal or as an explicit finding list already approved by the audit loop contract
- Builder prompt includes `Approved. Proceed.` when user approved step 4 or the fix round
- **No second user gate** inside the builder if orchestrator already holds approval for that exact fix list

### Gate D — Step 11 mechanical worker

Commit + task evidence runs **automatically** after 0 audit findings + green builder — **no extra user gate**, because user already approved at Gates A and B.

**Orchestrator must NOT** implement inline, skipping proposal gates, or commit before 0 findings unless user overrides in chat.

---

## Dynamic fix proposals (audit / evidence tasks)

Read-only or evidence-capture tasks often forbid tracked edits. When an in-path defect requires fixing, a **new dynamic fix proposal** is required containing:

- Orchestrator-confirmed defect list (or single defect)
- Exact **files**
- Exact **symbols**
- **Behavior** change
- **Tests** to add/update
- **CLI implications**
- **Verification lanes** (local / cloud / workspace-live / live-installed / installer-hardware)

Then:

1. Explicit **user approval** for the combined dynamic fix proposal
2. Edit-capable delegation prompt ends with **`APPROVED, PROCEED`**

**Preauthorization:** audit tasks do **not** preauthorize fixes. A failed check does not grant edit rights.

---

## Agent behavior during proposal phase

### Must do

- Stop all write tools after presenting proposal
- Use `AskQuestion` when structured choices clarify scope before proposing
- Cite existing code with Cursor citation format when grounding the proposal
- Keep proposal in direct technical mode — facts, files, symbols, behavior
- Record proposal and approval state in owned ledger before material action (when ledger task applies)

### Must not do

- Write files "to help explain"
- Run mutating shell commands
- Spawn edit-capable subagents
- Assume approval from enthusiasm or vague assent
- Bundle unapproved work into an approved menu option
- Expand scope during execution without re-proposal
- Fix off-path defects without separate approval

### After approval

- Touch **only** what the allowlist authorizes
- No drive-by refactors
- No unprompted future-proofing or silent fallbacks
- Report verification tier honestly
- Report off-path findings at task end without fixing them

---

## Proposal templates

### Minimal chat proposal (general)

## Proposal

**Intent:** <one paragraph>

**In scope — files:**
- `path/to/File.swift` — modify `SymbolName`
- `path/to/NewFile.swift` — create

**In scope — symbols:**
- `TypeName.methodName(...)` — <create|modify|delete>: <what changes>
- ...

**Out of scope:**
- <files/symbols explicitly not touched>

**Behavior:**
- Before: ...
- After: ...
- Non-goals: ...

**Proposed changes:**
<signatures, snippets, or tight pseudocode>

**Rules touchpoints:** <applicable rules and architecture constraints>

**Verify:**
- Scheme: ...
- Tests: ...
- Tier: local-validated | cloud-validated | workspace-live | ...

**Risks / questions:** <none | list>

---
Approve with: approved | proceed | yes | y

### Orchestrator researcher packet (full)

Use the 7-section researcher deliverable; section 5 is the user-facing approval gate.

### Dynamic fix proposal (audit escape hatch)

## Dynamic fix proposal

**Defect:** <id + symptom + file:line evidence>

**Root cause:** <confirmed, not inferred>

**Files:** ...
**Symbols:** ...
**Behavior:** ...
**Tests:** ...
**CLI:** ...
**Verification lanes:** ...

---
Approve with: approved | proceed | yes | y

---

## Edge cases and precedence

| Situation | Rule |
|-----------|------|
| User says "just do it" with no prior proposal | Produce expedited but **complete** proposal in same turn; still need approval token unless they simultaneously provide `yes`/`proceed` |
| User approves then immediately adds requirement | Re-propose |
| Scratch evidence write during read-only task | Allowed in `Scratch/<slug>--evidence/` without implementation approval; tracked edits still gated |
| Task-end auto-commit/push | Does not waive proposal-before-edit for source changes; commit/push may proceed under standing exception after implementation |
| Delegate returns without applying | Orchestrator verifies; does not self-implement without own proposal unless user requests takeover |
| `CLAUDE.md` edit | Always explicit approval; proposal required |
| Docs-only change | Proposal still required for tracked markdown; push may be commit-only under exception |
| Numbered menu `yes` | Approves all listed options only — not extra work |
| Prior chat approved similar work | Not valid; scope must match current combined proposal |

**Precedence:** User explicit IDE rules > this workflow > agent convenience. Stricter user rules apply in addition, not instead, unless they explicitly waive proposal (rare).

---

## Skill frontmatter suggestion

```yaml
---
name: implementation-approval-workflow
description: >-
  Mandatory proposal-before-edit gate. Use before any tracked write, git
  mutation, or edit-capable delegation. Requires chat proposal with file/symbol
  allowlist, concrete change detail, and explicit user approval
  (approved/proceed/yes/y) before executing.
---
```
