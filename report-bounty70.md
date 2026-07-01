# Delivery Report: runx skill: reply router (Bounty #70)

## What was built

A complete runx `reply-router` skill package that classifies inbound reply messages and either appends a suppression event for unsubscribes or emits a bounded routing decision for all other classifications.

## Files delivered

- `skills/reply-router/SKILL.md` — Full skill contract with all required sections (What this skill does, When to use, When not to use, Procedure, Edge cases and stop conditions, Output schema, Worked example, Inputs)
- `skills/reply-router/X.yaml` — Execution profile with typed inputs (inbound_reply, original_send_receipt, suppression_policy), typed outputs (classification, suppression_result, runx.reply.routing.v1), and inline harness cases
- `skills/reply-router/run.mjs` — Implementation: classification engine, CAS suppression store, routing decision emitter

## Harness results

```
$ runx harness ./skills/reply-router --json
{
  "status": "passed",
  "case_count": 2,
  "assertion_error_count": 0,
  "case_names": ["sealed_unsubscribe_suppression", "stop_ambiguous_or_unsealed"]
}
```

## Key design decisions

1. **Unsubscribe suppression**: Uses a local event store with compare-and-swap (CAS) version checking. The `idempotency_key` is derived from `aggregate_id:unsubscribe:checksum` to prevent duplicate suppressions. The `before_version` and `after_version` are returned for verification.

2. **Ambiguous reply stop**: When no classification meets the confidence threshold, the skill exits with code 64 (failure), no suppression is written, and no routing decision is emitted.

3. **Unsealed receipt stop**: If `original_send_receipt` lacks `receipt_id` or `checksum`, the skill stops immediately — no classification, no routing, no suppression.

4. **Never sends**: The skill only prepares routing decisions and suppression records. Any actual send requires a separate governed `send-as` run.

## PR

https://github.com/runxhq/runx/pull/211

## runx CLI version

runx-cli 0.6.15
