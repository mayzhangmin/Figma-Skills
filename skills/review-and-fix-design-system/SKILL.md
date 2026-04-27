---
name: review-and-fix-design-system
description: Review a Figma screen or component for design-system integration findings, then fix one approved high-signal finding in a narrow follow-up pass.
---

# Review And Fix Design-System Drift

Use this skill when the user wants a tighter review-and-repair loop on a Figma screen, frame, or component.

This skill orchestrates two existing workflows:
- [audit-design-system](../audit-design-system/SKILL.md) for read-only review
- [fix-design-system-finding](../fix-design-system-finding/SKILL.md) for one narrow write pass

It is a workflow skill, not a replacement for either underlying skill.

## Use This Skill For

- starting from a Figma URL when the exact finding to fix is not known yet
- reviewing a screen for the highest-signal design-system drift, then repairing one approved issue
- keeping the write scope narrow after a review instead of widening into a full reconnect pass

## Do Not Use This Skill For

- broad screen-wide or multi-section migration work that needs coordinated sequencing across several sections
- situations where the user already knows the exact single finding to repair
- tasks that require creating new design-system components, redefining library architecture, or inventing new token rules

Use [apply-design-system](../apply-design-system/SKILL.md) instead when the task becomes a broader reconnect or migration.
Use [fix-design-system-finding](../fix-design-system-finding/SKILL.md) instead when the target finding is already known and the review step is unnecessary.

## Core Contract

This skill has two phases:
1. review the target and produce concrete findings
2. fix one approved, high-signal finding without widening scope

Treat the audit result as the scope contract for the fix step.

Do not silently turn this workflow into a screen-wide cleanup pass.

## Required Workflow

### 1. Normalize The Input

Accept either:
- a Figma URL
- a `fileKey` and `nodeId`
- a prior audit result plus a target finding

If the user supplies a direct finding, you may skip the new audit and continue at the fix phase.

### 2. Run The Review Pass

Use [audit-design-system](../audit-design-system/SKILL.md) to inspect the target.

The review output should:
- identify concrete design-system findings with evidence
- prioritize them
- stay focused on systemization failures, not visual taste

If the review finds no issues, stop and report that no fix is needed.

### 3. Choose The Fix Target

Pick exactly one finding for the write pass.

Selection rules:
- if the user approved a finding, use that finding
- otherwise, use the highest-priority finding only when it is narrow and safe to fix
- if several findings are similarly important or the replacement choice is ambiguous, ask the user to choose instead of guessing

Do not batch multiple findings into one write pass.

### 4. Validate Scope Before Writing

Before calling the fix workflow, confirm the selected finding is still a narrow repair.

Route away from this skill when:
- several coupled sections need coordinated changes
- the review suggests a broader reconnect or migration pass
- the missing library coverage makes a narrow fix impossible

In those cases, recommend [apply-design-system](../apply-design-system/SKILL.md).

### 5. Fix The Selected Finding

Use [fix-design-system-finding](../fix-design-system-finding/SKILL.md) for the write phase.

The fix must remain:
- one finding at a time
- one offending node or local cluster at a time
- minimal read scope
- minimal write scope

### 6. Validate The Repair

After the write:
- confirm the original finding is no longer true
- confirm the updated node is library-backed or token-bound when that was the goal
- confirm the local layout and text still make sense

Do not claim the whole screen is fixed after repairing one finding.

## Decision Heuristics

- If the audit returns one strong, narrow finding, this skill is the right entry point.
- If the audit returns several related findings in one section cluster, consider whether they really collapse into one narrow repair; if not, route to [apply-design-system](../apply-design-system/SKILL.md).
- If the user wants “review this and fix the top issue,” this skill is the right default.
- If the user wants “fix this exact node,” skip this skill and use [fix-design-system-finding](../fix-design-system-finding/SKILL.md).

## Prompt Shapes

- `Review and fix design-system drift on this Figma link: <FIGMA_URL>`
- `Use [review-and-fix-design-system] on this Figma node: <FIGMA_URL>`
- `Audit this screen and fix the top approved design-system finding: <FIGMA_URL>`

## Deliverable Format

When closing the task, report in this order:

1. `Review summary`: the highest-signal findings and which one was selected
2. `Finding fixed`: what changed and on which node
3. `Validation`: how the original issue was confirmed resolved
4. `Follow-up`: only when broader reconnect work or additional approved fixes remain