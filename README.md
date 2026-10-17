# CovenantVault

CovenantVault is a reusable GenLayer intelligent contract for recurring,
evidence-backed obligations. An obligor escrows a performance bond, proves
compliance each period, receives a defined cure opportunity when proof fails,
and is deterministically slashed if the breach is not cured.

The primitive fits obligations that cannot be verified from chain state alone:

- reserve, solvency, and transparency reports;
- grant milestones and continuing grant conditions;
- vendor SLAs and recurring service deliverables;
- DAO delegate or service-provider mandates;
- licensing, certification, and policy commitments;
- data retention, deletion, or publication promises.

## Why it is reusable

The contract does not hard-code a business domain. Each covenant carries two
version-pinned JSON documents:

- `terms_json`: the actual obligation and prohibited conduct;
- `evidence_policy_json`: the required proof, acceptable sources, freshness,
  signatures, or other verification criteria.

Each period uses the same deterministic lifecycle:

```text
ACTIVE covenant
    |
    +-- submit_attestation --> SUBMITTED -- accepted --> CLEARED
    |                              |
    |                              +-- rejected --> CURE_REQUIRED
    |
    +-- deadline passes --> declare_missed_checkpoint --> CURE_REQUIRED

CURE_REQUIRED -- submit_cure --> CURE_PENDING -- accepted --> CLEARED
      |                               |
      |                               +-- rejected --> BREACHED + slash
      |
      +-- cure deadline passes --> BREACHED + slash

CLEARED/BREACHED --> next period, COMPLETED, or DEFAULTED
```

AI consensus only answers one constrained question: whether the supplied
evidence satisfies the fixed covenant. It cannot choose a beneficiary, bond,
slash amount, deadline, period, or transfer destination.

## Contract interface

### Write methods

`create_covenant(...) payable`

Creates a covenant and escrows exactly `bond` native units.

Parameters:

| Parameter | Meaning |
|---|---|
| `beneficiary` | Fixed address receiving breach proceeds |
| `terms_json` | JSON object describing the obligation |
| `evidence_policy_json` | JSON object describing acceptable evidence |
| `bond` | Total collateral; must equal transaction value |
| `slash_amount` | Amount paid per breach, unless less remains |
| `period_seconds` | Length of each rolling compliance period |
| `submission_grace_seconds` | Extra time after the due time to attest |
| `cure_window_seconds` | Time to submit and validate corrective evidence |
| `total_periods` | Number of periods, from 1 through 52 |
| `max_breaches` | Breach count that triggers default |

Other write methods:

- `submit_attestation(covenant_id, attestation_json)`
- `review_attestation(checkpoint_id)`
- `declare_missed_checkpoint(covenant_id)`
- `submit_cure(checkpoint_id, cure_json)`
- `review_cure(checkpoint_id)`
- `finalize_uncured_breach(checkpoint_id)`

The two deadline-enforcement methods are permissionless. No privileged keeper
or contract owner is required.

### View methods

- `get_covenant(covenant_id)`
- `get_checkpoint(checkpoint_id)`
- `list_checkpoints_for_covenant(covenant_id)`
- `covenant_count()`
- `checkpoint_count_total()`

## Example creation parameters

The following example creates three monthly-style periods in a compressed demo
configuration. Production values should reflect the real agreement.

```json
{
  "beneficiary": "0x1111111111111111111111111111111111111111",
  "terms_json": "{\"name\":\"Reserve transparency\",\"obligation\":\"Publish a signed reserve report covering all customer liabilities\",\"required_metrics\":[\"assets\",\"liabilities\",\"coverage_ratio\"],\"prohibited\":[\"omitting affiliated liabilities\",\"stale signatures\"]}",
  "evidence_policy_json": "{\"required\":[\"auditor signature\",\"report date\",\"asset and liability totals\"],\"max_report_age_days\":7,\"reject_if\":[\"self-authored claim only\"]}",
  "bond": 100000,
  "slash_amount": 25000,
  "period_seconds": 2592000,
  "submission_grace_seconds": 259200,
  "cure_window_seconds": 604800,
  "total_periods": 3,
  "max_breaches": 2
}
```

Attestation and cure payloads must be JSON objects containing an HTTPS
`primary_url`:

```json
{
  "primary_url": "https://reports.example/reserves/2026-10",
  "report_date": "2026-10-15",
  "auditor": "Veritas LLP"
}
```

## Development

From this directory:

```powershell
genvm-lint check contracts\covenant_vault.py --json
pytest -q
```

Current verification: 15 direct VM tests pass and the GenVM linter validates
all 12 public methods.

## Repository layout

```text
contracts/covenant_vault.py       contract implementation
tests/direct/                     deterministic VM and consensus-mock tests
DESIGN.md                         invariants, failure model, and threat model
gltest.config.yaml                gltest project configuration
```

## Important deployment note

This is testnet-oriented software, not an audited production escrow. Before
mainnet or material-value use, commission an independent review of economic
parameters, evidence policies, consensus behavior, and the target network's
native-value transfer semantics.
