---
name: implementation-approval
description: Explicit-only proposal-before-edit gate for repository changes. Bare invocation governs one change; `on` keeps the gate active for the current primary task until `off`. Use only when the user explicitly invokes `$implementation-approval`.
disable-model-invocation: true
---

# Implementation Approval

Give the user a reviewable description of a repository change before changing governed state. Adapt the proposal to the current repository without importing conventions from another project.

## Resolve the mode

Support exactly these controls:

- Bare `$implementation-approval` activates one-shot mode for the request containing the invocation, or for the next change request when invoked alone. It remains active through discovery, questions, revised proposals, implementation, and verification. It ends when the change completes or the user cancels it.
- `$implementation-approval on` activates the gate for every subsequent governed change in the current primary task until the user turns it off. Each materially separate change requires its own proposal and approval.
- `$implementation-approval off` ends optional one-shot or persistent gating. It does not authorize a pending implementation and cannot waive a system, developer, user, or repository rule that independently requires approval.

Do not introduce additional modes, persist mode state to disk, or carry persistent mode into another task. Mode state belongs to the primary task and never propagates to delegated workers.

## Establish the governing contract

Before proposing work:

1. Locate the repository root and read every instruction file governing the target paths.
2. Identify protected paths, approval language, scratch exceptions, Git rules, deployment rules, and stricter local requirements.
3. Inspect relevant code, configuration, tests, architecture, and existing working-tree changes using read-only operations.
4. Preserve unrelated user-owned work. Do not run a command whose write behavior is unknown.

While the gate is active, require approval before:

- creating, modifying, moving, or deleting tracked content or files intended to become repository deliverables;
- running commands that generate or rewrite repository files;
- destructive filesystem actions inside the repository;
- branch, index, commit, tag, merge, rebase, push, release, deployment, or other external-state mutations; and
- starting a worker that may perform any governed action.

Read-only inspection and explanation do not require approval. Treat temporary output as exempt only when it is outside the repository or uses a repository-defined untracked scratch location.

Question-only and review-only requests remain read-only. A defect found during review does not authorize a fix.

## Present one complete proposal

Before the first governed action, present a complete proposal in chat. Put the concrete change description and approval request in the same message. Include:

- **Intent:** the problem, desired outcome, and rationale.
- **File allowlist:** every file to create, modify, move, or delete, including generator inputs and generated outputs.
- **Change-unit allowlist:** exact types, functions, methods, fields, constants, routes, commands, configuration keys, document sections, records, or artifacts. State whether each will be created, modified, moved, renamed, or deleted.
- **Concrete changes:** signatures, changed regions, data shapes, configuration shapes, or tight pseudocode sufficient to review the result before a diff exists.
- **Behavior:** current behavior, resulting behavior, user-visible effects, compatibility or persistence effects, and explicit non-goals.
- **Rules:** applicable repository instructions and architectural precedents, with live file-and-line evidence when important.
- **Verification:** exact checks, scope, environment, expected evidence, and the destinations of any write-producing checks.
- **Risks:** genuine ambiguity or unresolved choices that repository evidence cannot settle.

Scale the detail to the change, but reject proposals that hide the concrete change behind phrases such as "update the service" or "refactor to match style." The user must be able to judge the eventual change without discovering important details only in the diff.

Use this shape when no stricter repository template exists:

```markdown
## Proposal

**Intent:** <problem, outcome, and rationale>

**Files:**
- `path/to/file` — <create|modify|move|delete>; <exact change units>

**Concrete changes:**
- <reviewable signature, region, data shape, or tight pseudocode>

**Behavior:**
- Before: ...
- After: ...
- Non-goals: ...

**Rules:** <applicable instructions and precedents>

**Verification:**
- <exact checks, scope, environment, and evidence>

**Risks / unresolved choices:** <none or list>

Approve with: approved | proceed | yes | y
```

After presenting the proposal, stop before all governed actions.

## Resolve approval

Approval applies only to the latest complete proposal.

- Follow exact approval language when governing repository policy defines it.
- Otherwise accept `approved`, `proceed`, `yes`, or `y`, case-insensitively, for the current proposal.
- Do not treat silence, praise, questions, continued research, or approval of a superseded proposal as authorization.
- Invoking the skill or turning it on does not approve a proposal. Turning it off does not authorize pending work.
- A request such as "just do it" does not skip the proposal while the gate is active.
- If the user approves only part of the proposal, execute only a clearly identified, independently complete part. Otherwise present a revised complete proposal.

When approval is accompanied by a material scope change, treat the proposal as superseded and seek fresh approval.

## Keep delegation from re-gating

The primary agent exclusively owns this approval gate.

Before approval:

- Read-only researchers or reviewers may be delegated bounded inspection.
- Their prompts must state that they are read-only.
- Do not start an edit-capable worker.

After approval, an edit-capable worker may be delegated only the approved scope:

- Pass the exact file and change-unit allowlists, behavior contract, and verification assignment.
- Do not invoke `$implementation-approval` in the worker prompt or propagate one-shot or persistent mode.
- When the runtime supports context selection, give the worker fresh or minimum necessary context rather than relying on inherited approval state.
- State that approval has already been obtained and the worker must not ask for it again.
- End the edit-capable worker prompt with exactly:

```text
APPROVED, PROCEED
```

Never attach that marker before approval or to a strictly read-only worker. If a worker discovers a material delta, it must report the need without implementing it. The primary agent owns any re-proposal.

## Execute only the approved scope

After approval:

1. Recheck the approved file and change-unit allowlists.
2. Apply only the approved changes.
3. Run only the approved verification.
4. Report the implementation, observed validation level, remaining risks, and off-path findings.

Do not include drive-by refactors, speculative hardening, unprompted future-proofing, silent fallbacks, or off-path fixes.

## Re-propose material changes

Present a new complete proposal and obtain fresh approval when execution requires:

- additional or different files or change units;
- behavior beyond the approved contract;
- a new public, API, CLI, GUI, schema, migration, persistence, or architectural change;
- a destructive or external-state operation not already approved;
- a materially different verification basis;
- implementation after a read-only or evidence-only request; or
- a fix for an off-path finding.

Explain what changed and why the prior proposal is no longer sufficient. Earlier approval does not carry forward. A trivial clarification entirely inside the same allowlists and behavior contract does not require re-proposal.

## Keep Git and external authority separate

Implementation approval does not automatically authorize branch creation, staging, commits, tags, merges, rebases, pushes, releases, deployments, destructive actions, or third-party mutations. Follow the current repository's rules for those operations and obtain separate authorization unless the approved proposal explicitly includes them and governing policy permits combined approval.

Never generalize repository-specific auto-commit, auto-push, scratch, protected-file, validation-tier, release, or orchestrator exceptions.

## Follow precedence

Apply system and developer instructions, governing repository instructions, and explicit current user instructions before this skill's portable defaults. Turning optional mode off cannot waive a mandatory higher-priority gate.
