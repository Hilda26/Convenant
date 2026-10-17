# Live StudioNet evidence

CovenantVault was deployed and exercised on GenLayer StudioNet on October 17,
2026.

## Deployment

- Contract: `0xfF956A669432b4BcAe765d4a7B2B3c7041BFCf68`
- Explorer: `https://explorer-studio.genlayer.com/address/0xfF956A669432b4BcAe765d4a7B2B3c7041BFCf68`
- Deployment transaction: `0xb4f358a989f844b1d9032efbcabdd11f1722f0f7def6e3f695de8ab2d88fa6a6`
- Network: GenLayer StudioNet, chain ID `61999`
- Deployed source commit: `d3db79f`
- Constructor arguments: none (`[]`)

The deployment finalized with successful leader execution, empty stdout and
stderr, and no raw GenVM error. Live schema retrieval exposed all 12 expected
public methods.

## Lifecycle transactions

| Action | Transaction |
|---|---|
| Deploy CovenantVault | `0xb4f358a989f844b1d9032efbcabdd11f1722f0f7def6e3f695de8ab2d88fa6a6` |
| Create covenant with three-unit bond | `0xcd537b24e9c6f781d1cdd2b1839c4eda9289ac7557ad68b35a2af35b4496a2d8` |
| Submit compliant period 1 | `0xdbfc9bed3ed7f6c2903ec392f8f619db90f92c2d13c5f4069b3db5cf29179cb0` |
| Review and clear period 1 | `0x2a3871f3f8379a063e286bd28a56a9a5c03af005346158ad45c019a427061a30` |
| Submit prohibited period 2 | `0x214c7890338890677c8f8943c3c7bd7358da792b043c8eb9a1fcfeaf5a273240` |
| Review period 2 and require cure | `0xddaff4006e3ad3e52e40e651a4e46e68fe146683646f862fd481e2e1135bbfcc` |
| Submit deliberately failed cure | `0x7814d61a91cfe17bfe4e20cc20860fe68ef786277c84c54fb0fc96250ec20d9e` |
| Review cure and settle default | `0x206fa093173f75469294939e86a2970e49444f8cce6c6271a00ca86494e6065c` |

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
  --contract 0xfF956A669432b4BcAe765d4a7B2B3c7041BFCf68 `
  --out live-studionet-latest.json
```

The runner is resumable after transient RPC disconnections:

```powershell
node live_studionet_covenant_tests.mjs `
  --contract 0xfF956A669432b4BcAe765d4a7B2B3c7041BFCf68 `
  --resume-covenant 0 `
  --out live-studionet-latest.json
```

The full machine-readable state and receipt summary is in
`live-studionet-latest.json`. Public validator evidence is in `live-evidence/`.
