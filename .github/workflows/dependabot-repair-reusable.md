---
description: Reusable Dependabot repair workflow for product repositories. Call this workflow from another repository to attempt one minimal safe repair on Dependabot pull requests.
on:
  workflow_call:
    inputs:
      mode:
        description: Operating mode for the caller repository.
        required: false
        default: baseline
        type: string
      safe-repair:
        description: Allow one minimal safe repair attempt.
        required: false
        default: true
        type: boolean
      automerge:
        description: Allow low-risk updates to be marked as automerge-eligible.
        required: false
        default: true
        type: boolean
      project-sync:
        description: Enable project-board coordination when the caller uses the advanced layer.
        required: false
        default: false
        type: boolean

inlined-imports: true

permissions: read-all
if: github.event.pull_request.user.login == 'dependabot[bot]'

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

# Dependabot Reusable Repair

## Scope

This workflow runs only for PRs authored by `dependabot[bot]`.

## Mission

Inspect failing checks and attempt one minimal safe repair.

Use `mode`, `safe-repair`, `automerge`, and `project-sync` as caller-provided operating hints.

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
[dependabot-repair] Fix CI for PR #[number]

## Comment

### Dependabot Review

Action: repaired | safe-out | no-op  
Checks reviewed: yes | no  
Repair PR: `[link]`

Summary: `[explanation]`  
Next Step: `[action]`
