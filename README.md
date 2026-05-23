# CNC Postprocessors

Postprocessors for AP4060 and CNC_T800 machines. Supports Fusion 360 and SolidCAM.

Naming convention: `..._MMDDYYYY.cps` (US date format).

## Machines

### AP4060

**Fusion 360** (`.cps`)
| Date | File |
|------|------|
| 05/23/2026 | `TechCNC_AP4060_05232026.cps` |
| 05/22/2026 | `TechCNC_AP4060_05222026.cps` |
| 05/08/2026 | `TechCNC_AP4060_05082026.cps` |
| 12/22/2025 | `TechCNC_AP4060_12222025.cps` |
| 11/19/2025 | `TechCNC_AP4060_11192025.cps` |
| 10/20/2025 | `TechCNC_AP4060_10202025.cps` |

**Coolant / MQL behavior (latest)**
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
| Date | File |
|------|------|
| 10/20/2025 | `Fusion360_T800_10202025.cps` |

## Changelog (AP4060 / Fusion 360)

### 05/23/2026 — Simplified coolant + drill/reamer Q30 override
- **Disabled mode** now fully turns off both air and MQL: emits `G10 L80 P8133 Q0` then `M9`.
- **All other modes** (Air, Mist, Flood and Mist, Flood) always enable `M7 + M8` together.
- **Air** mode: `M7 + M8` with `Q0` (air only).
- Q values per mode: Mist=Q5, Flood and Mist=Q10, Flood=Q20.
- **Drill / reamer Q30 override:** any tool of type `TOOL_DRILL`, `TOOL_DRILL_CENTER`, `TOOL_DRILL_SPOT`, `TOOL_DRILL_BLOCK` or `TOOL_REAMER` forces **Q30** regardless of the selected Coolant — except Disabled (which stays Q0/off). Comment line shows `(Coolant: <Mode> (drill/reamer override))` for visibility.
- T-slot mill is NOT auto-overridden — operator should select Flood for it.
- **Removed hardcoded `Q30` in `deep-drilling` cycle** — Q now comes from the unified drill/reamer override at the top of `setCoolant`.

### 05/22/2026 — Coolant rework + tool-type-aware MQL
- `vendorUrl` updated to include machine name (`/ CNC Router AP4060`) — visible as a header comment in G-code.
- Full rewrite of local `setCoolant` function:
  - Added tool-type override: drill (`TOOL_DRILL`, `TOOL_DRILL_CENTER`, `TOOL_DRILL_SPOT`, `TOOL_DRILL_BLOCK`), reamer (`TOOL_REAMER`) and T-slot mill (`TOOL_MILLING_SLOT`) force MQL = **Q30 ml/h** regardless of coolant mode. *(T-slot removed from override list in 05/23/2026.)*
  - `COOLANT_AIR`: emits only `M7` (was `M209`), MQL off. *(Changed to M7+M8/Q0 in 05/23/2026.)*
  - `COOLANT_MIST`: `M7 + M8`, **Q5** base / Q30 override.
  - `COOLANT_FLOOD_MIST`: new case — `M7 + M8`, **Q10** base / Q30 override.
  - `COOLANT_FLOOD`: changed to `M7 + M8` (was `M8` only), **Q20** base / Q30 override.
  - Each case writes a `(Coolant: <Mode>)` comment before the `G10 L80 P8133 Q..` line.

### 05/08/2026 — Program-header metadata + MQL bump
- Added `formatTime(seconds)` helper.
- Program header now writes:
  - `(Source file: <document name>)` — derived from `document-name` / `document-path` / `job-description`.
  - `(Generated: YYYY-MM-DD HH:MM:SS)` timestamp.
  - `(Total time: HH:MM:SS)` — sum of all section cycle times.
- Per-operation comment `(Operation time: HH:MM:SS)`.
- MQL amounts restored / raised:
  - `COOLANT_FLOOD`: Q5 -> **Q10**
  - `COOLANT_MIST`: Q10 -> **Q30**
  - `deep-drilling` cycle: Q20 -> **Q30** *(Removed in 05/23/2026.)*

### 12/22/2025 — MIST consistency fix
- `COOLANT_MIST`: fixed mismatch between comment (`Q30`) and block (`Q20`); both lowered to **Q10**.

### 11/19/2025 — Initial MQL tuning
- `COOLANT_FLOOD`: Q10 -> **Q5**
- `COOLANT_MIST`: block `Q30` -> **Q20** (comment stale `Q30` — fixed next release).
- `deep-drilling` cycle: Q30 -> **Q20**.

### 10/20/2025 — Baseline
- First versioned post processor.
- Coolant handling: `FLOOD` = Q10 + M8, `MIST` = Q30 + M8, `AIR` = M209.
- `deep-drilling` cycle emits Q30 oil amount.

## Structure

```
Postprocessors/
├── AP4060/
│   ├── Fusion_360/
│   │   ├── TechCNC_AP4060_05232026.cps
│   │   ├── TechCNC_AP4060_05222026.cps
│   │   ├── TechCNC_AP4060_05082026.cps
│   │   ├── TechCNC_AP4060_12222025.cps
│   │   ├── TechCNC_AP4060_11192025.cps
│   │   └── TechCNC_AP4060_10202025.cps
│   └── Solid_CAM/
│       ├── MD-CNC_3x.gpp
│       ├── MD-CNC_3x.vmid
│       ├── MD-CNC_4x.gpp
│       └── MD-CNC_4x.vmid
└── CNC_T800/
    └── Fusion360_T800_10202025.cps
```

## How to Install

**Fusion 360 (.cps)**
1. Fusion 360 -> Manufacture workspace
2. Post Process -> click the library icon
3. Import -> select the `.cps` file
4. Use the latest dated version

**SolidCAM (.gpp / .vmid)**
1. Copy both `.gpp` and `.vmid` files to your SolidCAM GPP folder
2. Select the postprocessor in SolidCAM Machine settings