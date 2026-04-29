---
description: Repair safe Dependabot PR failures locally inside a product repository.
on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions: read-all

tools:
  github:
    toolsets: [default]

network:
  allowed: [defaults, node, python, go, java, ruby, rust]

#observability:
#  otlp:
#    endpoint: ${{ secrets.OTEL_EXPORTER_OTLP_ENDPOINT }}
#    headers: ${{ secrets.OTEL_EXPORTER_OTLP_HEADERS }}

safe-outputs:
  add-comment:
    max: 5
  update-issue:
    max: 5
  create-pull-request:
    max: 1
  noop:
    max: 1
---

# Dependabot Local Repair

## Scope

Only act on PRs authored by `dependabot[bot]`.

If not, use `noop`.

## Mission

Inspect failing checks and attempt one minimal safe repair.

## Durable Policy vs One-Off Context

Keep durable repair policy in this workflow file.

- use workflow text for standing repair limits, escalation rules, and output expectations
- treat check-specific findings, temporary operator guidance, and run-specific exceptions as one-off runtime context unless the repository intentionally changes workflow policy
- do not promote a single repair situation into a permanent rule without updating the workflow on purpose

## Workflow Phase Boundaries

Respect the workflow split between precompute, agent, and post-process.

Precompute:

- gather deterministic facts such as failing checks, changed files, attempted repairs, repository tooling, and whether the PR fits scope
- identify the available repair surface without deciding whether a repair is acceptable

Agent:

- interpret the precomputed facts using the durable repair policy in this file
- decide whether to repair, safe-out, or no-op and draft the explanation for that choice
- keep decisions scoped to the current run rather than creating new standing rules

Post-process:

- apply labels, comments, and at most one repair PR based on the agent decision
- persist only the outputs needed to complete the chosen action
- do not broaden the repair or introduce new policy here

## Allowed Repairs

- lockfile updates
- import fixes
- config updates
- test/snapshot updates

## Forbidden

- business logic changes
- auth/crypto/db behavior
- deployment config
- secrets

## Safe-Out

If risky:
- label `agent:safe-out`
- label `needs-human-review`
- comment explanation

## Repair Rules

- max one attempt
- minimal diff
- no test deletion

## Repair PR

Title:
[dependabot-repair] Fix CI for PR #<number>

## Comment

### Dependabot Review

Action: repaired | safe-out | no-op  
Checks reviewed: yes | no  
Repair PR: <link>

Summary: <explanation>  
Next Step: <action>
