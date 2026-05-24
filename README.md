# CNC Postprocessors

Postprocessors for AP4060 and CNC_T800 machines. Supports Fusion 360 and SolidCAM.

Naming convention: `..._revNN_MMDDYYYY.cps`. `rev01` is always the **latest** version, higher numbers = older. Files are listed newest -> oldest in alphabetical order.

## Machines

### AP4060

**Fusion 360** (`.cps`)
| Rev | Date | File |
|-----|------|------|
| rev01 | 05/23/2026 | `TechCNC_AP4060_rev01_05232026.cps` |
| rev02 | 05/22/2026 | `TechCNC_AP4060_rev02_05222026.cps` |
| rev03 | 05/08/2026 | `TechCNC_AP4060_rev03_05082026.cps` |
| rev04 | 12/22/2025 | `TechCNC_AP4060_rev04_12222025.cps` |
| rev05 | 11/19/2025 | `TechCNC_AP4060_rev05_11192025.cps` |
| rev06 | 10/20/2025 | `TechCNC_AP4060_rev06_10202025.cps` |

**Coolant / MQL behavior (rev01)**
| Fusion mode | M-codes | MQL Q (ml/h) | Q for drill / reamer |
|---|---|---|---|
| Disabled | M9 | Q0 (off) | Q0 (off) |
| Air | M7 + M8 | Q0 (air only) | **Q30** (override) |
| Mist | M7 + M8 | Q5 | **Q30** (override) |
| Flood and Mist | M7 + M8 | Q10 | **Q30** (override) |
| Flood | M7 + M8 | Q20 | **Q30** (override) |

> **Drill / reamer auto-override (since 05/23/2026):** for any tool of type `TOOL_DRILL`, `TOOL_DRILL_CENTER`, `TOOL_DRILL_SPOT`, `TOOL_DRILL_BLOCK` or `TOOL_REAMER`, the post forces **Q30 ml/h** MQL regardless of the Coolant dropdown — except when **Disabled** is selected (then everything is fully off). T-slot mill and other tool types are NOT overridden — Q value follows the operator''s Coolant choice. For T-slot mills the recommended mode is **Flood** (Q20).

**SolidCAM** (`.gpp` / `.vmid`)
| File | Axes |
|------|------|
| `MD-CNC_3x.gpp` + `MD-CNC_3x.vmid` | 3-axis |
| `MD-CNC_4x.gpp` + `MD-CNC_4x.vmid` | 4-axis |

### CNC_T800

**Fusion 360** (`.cps`)
| Rev | Date | File |
|-----|------|------|
| rev01 | 10/20/2025 | `Fusion360_T800_rev01_10202025.cps` |

## Changelog (AP4060 / Fusion 360)

### rev01 — 05/23/2026 — Simplified coolant + drill/reamer Q30 override
- **Disabled mode** now fully turns off both air and MQL: emits `G10 L80 P8133 Q0` then `M9`.
- **All other modes** (Air, Mist, Flood and Mist, Flood) always enable `M7 + M8` together.
- **Air** mode: `M7 + M8` with `Q0` (air only).
- Q values per mode: Mist=Q5, Flood and Mist=Q10, Flood=Q20.
- **Drill / reamer Q30 override:** any tool of type `TOOL_DRILL`, `TOOL_DRILL_CENTER`, `TOOL_DRILL_SPOT`, `TOOL_DRILL_BLOCK` or `TOOL_REAMER` forces **Q30** regardless of the selected Coolant — except Disabled (which stays Q0/off).
- Robust drill detection: tries `tool.isDrill()`, then `tool.type` constants, then `operation-strategy` parameter. Includes a DEBUG comment in G-code output showing which method matched.
- T-slot mill is NOT auto-overridden — operator should select Flood for it.
- **Removed hardcoded `Q30` in `deep-drilling` cycle** — Q now comes from the unified drill/reamer override at the top of `setCoolant`.

### rev02 — 05/22/2026 — Coolant rework + tool-type-aware MQL
- `vendorUrl` updated to include machine name (`/ CNC Router AP4060`) — visible as a header comment in G-code.
- Full rewrite of local `setCoolant` function:
  - Added tool-type override: drill, reamer and T-slot mill force MQL = Q30. *(T-slot removed in rev01.)*
  - `COOLANT_AIR`: emits only `M7` (was `M209`), MQL off. *(Changed to M7+M8/Q0 in rev01.)*
  - `COOLANT_MIST`: M7+M8, Q5 base / Q30 override.
  - `COOLANT_FLOOD_MIST`: new case — M7+M8, Q10 base / Q30 override.
  - `COOLANT_FLOOD`: changed to M7+M8 (was M8 only), Q20 base / Q30 override.
  - Each case writes `(Coolant: <Mode>)` comment before the G10 line.

### rev03 — 05/08/2026 — Program-header metadata + MQL bump
- Added `formatTime(seconds)` helper.
- Program header now writes Source file / Generated timestamp / Total time / Operation time.
- MQL: FLOOD Q5 -> Q10, MIST Q10 -> Q30, deep-drilling Q20 -> Q30 *(deep-drilling removed in rev01)*.

### rev04 — 12/22/2025 — MIST consistency fix
- `COOLANT_MIST`: fixed mismatch between comment (Q30) and block (Q20); both lowered to Q10.

### rev05 — 11/19/2025 — Initial MQL tuning
- `COOLANT_FLOOD`: Q10 -> Q5
- `COOLANT_MIST`: block Q30 -> Q20 (comment stale Q30 — fixed next release)
- `deep-drilling` cycle: Q30 -> Q20

### rev06 — 10/20/2025 — Baseline
- First versioned post processor.
- Coolant: FLOOD=Q10+M8, MIST=Q30+M8, AIR=M209.
- `deep-drilling` cycle emits Q30 oil amount.

## Structure

```
Postprocessors/
├── AP4060/
│   ├── Fusion_360/
│   │   ├── TechCNC_AP4060_rev01_05232026.cps   <- latest
│   │   ├── TechCNC_AP4060_rev02_05222026.cps
│   │   ├── TechCNC_AP4060_rev03_05082026.cps
│   │   ├── TechCNC_AP4060_rev04_12222025.cps
│   │   ├── TechCNC_AP4060_rev05_11192025.cps
│   │   └── TechCNC_AP4060_rev06_10202025.cps   <- oldest
│   └── Solid_CAM/
│       ├── MD-CNC_3x.gpp
│       ├── MD-CNC_3x.vmid
│       ├── MD-CNC_4x.gpp
│       └── MD-CNC_4x.vmid
└── CNC_T800/
    └── Fusion360_T800_rev01_10202025.cps
```

## How to Install

**Fusion 360 (.cps)**
1. Fusion 360 -> Manufacture workspace
2. Post Process -> click the library icon
3. Import -> select the `.cps` file (always pick the `rev01_*` file at the top)
4. When a new revision is added, all older files are renumbered (rev01 becomes rev02, etc.) — always re-import `rev01` to stay current.

**SolidCAM (.gpp / .vmid)**
1. Copy both `.gpp` and `.vmid` files to your SolidCAM GPP folder
2. Select the postprocessor in SolidCAM Machine settings