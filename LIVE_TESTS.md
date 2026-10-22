# Live StudioNet evidence

CovenantVault was corrected, redeployed, and exercised on GenLayer StudioNet on October 22,
2026.

## Deployment

- Contract: `0x8fF28Cf68B492EEd5C911a913ef1E5Bf4fdb75d2`
- Explorer: `https://explorer-studio.genlayer.com/address/0x8fF28Cf68B492EEd5C911a913ef1E5Bf4fdb75d2`
- Deployment transaction: `0xdeea1e398da6042ad56a9e8e0aa64e9ca32fc3cdc49cff333122387a7f54f1b2`
- Network: GenLayer StudioNet, chain ID `61999`
- Deployed source commit: `21b2261`
- Constructor arguments: none (`[]`)

The deployment finalized with unanimous validator agreement, successful leader
execution, empty stdout and stderr, and no raw GenVM error. Live schema
retrieval exposed all 12 expected public methods.

The corrected comparative response contains exactly `compliant` and
`reason_code`. It contains no timestamp. Live state confirms the cure deadline
is exactly 900 seconds after deterministic review time and the rolling next due
date is exactly 3600 seconds after deterministic settlement time.

## Lifecycle transactions

| Action | Transaction |
|---|---|
| Deploy CovenantVault | `0xdeea1e398da6042ad56a9e8e0aa64e9ca32fc3cdc49cff333122387a7f54f1b2` |
| Create covenant with three-unit bond | `0x2f136807637cc0b58e0ad060be3de0c1287f0f1db5a5445f5f2703306f035027` |
| Submit compliant period 1 | `0xe3d8f26cb655d886c016b5ff45b5798d7cdcea7291fa7ee570551e52d857286e` |
| Review and clear period 1 | `0xbabfe2416a78956d04cb0ede4523056319b42a1c580ce44af7b0e714b0c999eb` |
| Submit prohibited period 2 | `0xdae162f9f616240cfbcce3aa34a1b2da04200417b787a59407c6bfc79f2f6557` |
| Review period 2 and require cure | `0x4176a74ef241ff959380d2b39045764260a36b6eded37e35ff4e07c3c2c3c1d5` |
| Submit deliberately failed cure | `0xd44f726e7d6d2dc9ba117d039df76aa03ff178d9419c79f26e0a97cd745e994b` |
| Review cure and settle default | `0xa2a53c516932e59cb73fbbb921840c007c4f3599070a9130bbe26eaa6de99e40` |

Every transaction above reached `FINALIZED`, used successful leader execution,
had empty stderr, and reported no raw GenVM error.

## Verified semantic and economic state

- Covenant ID: `0`
- Period 1 checkpoint ID: `0`
  - Status: `CLEARED`
  - Reason: `COVENANT_SATISFIED`
- Period 2 checkpoint ID: `1`
  - Attestation review: `CURE_REQUIRED`
  - Initial reason: `PROHIBITED_CONDUCT`
  - Final status after cure review: `BREACHED`
  - Final reason: `OBLIGATION_UNMET`
- Final covenant state: `DEFAULTED`
- Breach count: `1`
- Maximum breaches: `1`
- Original bond: `3`
- Bond remaining: `0`

The terminal review emitted transfers of `1` and the remaining `2` native units
to the fixed beneficiary, proving that default routed the full bond without an
AI-selected amount or destination.

## Reproduce

```powershell
node live_studionet_covenant_tests.mjs `
  --contract 0x8fF28Cf68B492EEd5C911a913ef1E5Bf4fdb75d2 `
  --out live-studionet-latest.json
```

The runner is resumable after transient RPC disconnections:

```powershell
node live_studionet_covenant_tests.mjs `
  --contract 0x8fF28Cf68B492EEd5C911a913ef1E5Bf4fdb75d2 `
  --resume-covenant 0 `
  --out live-studionet-latest.json
```

The full machine-readable state and receipt summary is in
`live-studionet-latest.json`. Public validator evidence is in `live-evidence/`.
