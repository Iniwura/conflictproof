# ConflictProof

ConflictProof is a GenLayer Intelligent Contract that determines whether two independently sourced rules materially conflict for a specific subject or action.

It is designed as a reusable proof primitive for governance, policy systems, agent permissions, compliance workflows, procurement rules, treasury controls, protocol operations, and other environments where two rules may both appear applicable but cannot necessarily be satisfied together.

## Verdicts

ConflictProof returns one of three semantic outcomes:

- `CONFLICT` — the two applicable rules impose materially incompatible obligations, permissions, prohibitions, limits, timing requirements, or outcomes for the same subject or action.
- `NO_CONFLICT` — the rules can coexist, apply to different scopes, subjects, times, thresholds, actions, or conditions, or the claimed incompatibility is positively disproven.
- `UNRESOLVED` — the available evidence is missing, ambiguous, conflicting, unavailable, or otherwise insufficient to establish either result.

## How it works

A case contains:

- a title
- the subject being evaluated
- an exact conflict claim
- Rule A with a public HTTPS evidence source
- Rule B with a separate public HTTPS evidence source

The contract normalizes and validates both URLs, rejects duplicate sources and unsafe/private hosts, fetches bounded evidence, creates content commitments, and evaluates the exact claimed conflict through GenLayer nondeterministic execution.

Validators independently re-fetch and re-evaluate the case through `run_nondet_unsafe`, so the leader result is not accepted blindly.

Transient source failures enter a bounded retry state. Missing or unusable evidence resolves to `UNRESOLVED` rather than being treated as proof of no conflict.

## Public methods

### Write

- `create_case(case_json: str) -> str`
- `evaluate(case_id: str) -> None`
- `retry_evaluation(case_id: str) -> None`

### View

- `get_case(case_id: str)`
- `get_evaluation(case_id: str)`
- `get_evidence(case_id: str, role: str)`
- `is_finalized(case_id: str) -> bool`
- `get_creator_case_count(creator: str) -> int`
- `get_creator_case_id(creator: str, index: int) -> str`

Evidence roles are `RULE_A` and `RULE_B`.

## Case schema

```json
{
  "schema_version": "1.0",
  "title": "Emergency transfer timing conflict",
  "subject": "Treasury transfer TX-104",
  "conflict_claim": "Rule A requires immediate execution while Rule B requires a 24-hour delay for the same transfer.",
  "rule_a": {
    "statement": "Emergency security transfers must execute immediately.",
    "label": "Emergency operations rule",
    "source_url": "https://example.com/rule-a.txt"
  },
  "rule_b": {
    "statement": "Transfers above the applicable threshold must wait 24 hours before execution.",
    "label": "Treasury delay rule",
    "source_url": "https://example.com/rule-b.txt"
  }
}
```

## Deployment

GenLayer Studio Dev deployment:

`0xb7206C18a35E00f8fb175B718C31c51Dc37EafD9`

Explorer:

https://explorer-studio-dev.genlayer.com/address/0xb7206C18a35E00f8fb175B718C31c51Dc37EafD9

## Runtime

Built for the GenLayer Studio `v0.123` release-candidate contract API using:

- `gl.contract.Contract`
- `gl.storage.TreeMap`
- GenLayer nondeterministic web access
- GenLayer LLM execution
- validator re-evaluation with `gl.vm.run_nondet_unsafe`

## Contract

The contract source is in [`contract/ConflictProof.py`](contract/ConflictProof.py).
