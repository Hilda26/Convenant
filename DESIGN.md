# CovenantVault design and invariants

## Design boundary

CovenantVault deliberately separates semantic judgment from asset control.

Consensus may decide only:

```json
{"compliant": true, "reason_code": "COVENANT_SATISFIED"}
```

or a noncompliant result with one of five finite rejection codes. Contract code
validates that the boolean and reason code agree. An unknown, contradictory, or
malformed result enters a retryable error state and never causes a slash.

The contract deterministically fixes:

- the obligor and beneficiary;
- the total bond and per-breach slash amount;
- period, grace, and cure lengths;
- the current period and default threshold;
- every transfer amount and transfer destination.

## Economic invariants

For every covenant:

1. `0 <= bond_remaining <= bond_total`.
2. Only the fixed beneficiary receives slashed funds.
3. Only the fixed obligor receives a completion refund.
4. A checkpoint is settled at most once.
5. At most one checkpoint is open for a covenant.
6. A period advances only after its checkpoint becomes `CLEARED` or `BREACHED`.
7. Malformed consensus output and evidence-fetch failure do not move money.
8. Default transfers the full remainder to the beneficiary and is terminal.
9. Completion transfers the full remainder to the obligor and is terminal.

The accounting identity is:

```text
bond_total = cumulative beneficiary transfers
           + cumulative obligor refunds
           + bond_remaining
```

## Timing model

Each period begins with `next_due_at`. The obligor may attest until:

```text
next_due_at + submission_grace_seconds
```

If no attestation exists after that point, any address may create the missed
checkpoint. Rejected or missed checkpoints receive a cure deadline calculated
from the time the failure is recorded.

After a settled nonterminal checkpoint, the next due time is rolling:

```text
settled_at + period_seconds
```

This avoids a delayed review instantly making later periods overdue.

## Terminal rules

A covenant completes after the last period unless that settlement also reaches
the configured default threshold. Completion refunds the remaining bond.

A covenant defaults immediately if either:

- `breach_count >= max_breaches`; or
- the bond is exhausted.

Default takes precedence over ordinary period completion and transfers all
remaining collateral to the beneficiary.

## Failure model

Web fetch failure returns an out-of-band reason code that the deterministic
parser rejects. The checkpoint becomes `ERRORED` or `CURE_ERRORED`, remains open,
and can be reviewed again. No payment occurs.

Malformed, unknown, or boolean-inconsistent model output follows the same path.
Consensus timestamps are required and checked against evidence submission and,
for cures, the cure deadline.

## Prompt-injection boundary

Terms, policies, JSON payloads, URLs, and fetched pages are explicitly marked as
untrusted data. The prompt requires one finite JSON verdict, and deterministic
parsing rejects everything outside the schema. This narrows prompt injection's
effect, but does not make semantic judgment infallible. High-value deployments
should use precise terms, objective evidence policies, reputable evidence URLs,
and conservative default thresholds.

## Known tradeoffs

- Terms and evidence policy are immutable after creation. This prevents one
  party from changing an active bargain, at the cost of requiring a replacement
  covenant when both parties want an amendment.
- The schedule rolls forward from settlement rather than anchoring every period
  to the original creation time. This avoids deadline cascades but permits the
  overall end date to drift.
- HTTPS proves transport intent, not source truth. Evidence-policy quality and
  source reputation remain essential.
- A transient fetch or model failure can delay settlement. It cannot by itself
  slash the obligor.

## Tested transitions

The direct suite covers creation validation, sender authorization, payload and
URL validation, timing guards, parallel-checkpoint prevention, compliant
completion and refund, multi-period advancement, cure entry, successful cure,
failed cure and slashing, repeated-breach default, missed attestation,
permissionless uncured finalization, fetch retry, prompt injection, terminal
replay resistance, enumeration, and unknown-ID guards.
