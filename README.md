# CNC Postprocessors

Postprocessors for AP4060 and CNC_T800 machines. Supports Fusion 360 and SolidCAM.

Naming convention: `..._MMDDYYYY.cps` (US date format, matches today''s file `_05222026`).

## Machines

### AP4060

**Fusion 360** (`.cps`)
| Date | File |
|------|------|
| 05/22/2026 | `TechCNC_AP4060_05222026.cps` |
| 05/08/2026 | `TechCNC_AP4060_05082026.cps` |
| 12/22/2025 | `TechCNC_AP4060_12222025.cps` |
| 11/19/2025 | `TechCNC_AP4060_11192025.cps` |
| 10/20/2025 | `TechCNC_AP4060_10202025.cps` |

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

### 05/22/2026 — Coolant rework + tool-type-aware MQL
- `vendorUrl` updated to include machine name (`/ CNC Router AP4060`) — visible as a header comment in G-code.
- Full rewrite of local `setCoolant` function:
  - Added tool-type override: drill (`TOOL_DRILL`, `TOOL_DRILL_CENTER`, `TOOL_DRILL_SPOT`, `TOOL_DRILL_BLOCK`), reamer (`TOOL_REAMER`) and T-slot mill (`TOOL_MILLING_SLOT`) force MQL = **Q30 ml/h** regardless of coolant mode.
  - `COOLANT_AIR`: emits only `M7` (was `M209`), MQL off.
  - `COOLANT_MIST`: `M7 + M8`, **Q5** base / Q30 override.
  - `COOLANT_FLOOD_MIST`: new case — `M7 + M8`, **Q10** base / Q30 override.
  - `COOLANT_FLOOD`: changed to `M7 + M8` (was `M8` only), **Q20** base / Q30 override.
  - Each case now writes a `(Coolant: <Mode>)` comment before the `G10 L80 P8133 Q..` line so the operator sees which mode was selected in Fusion.

### 05/08/2026 — Program-header metadata + MQL bump
- Added `formatTime(seconds)` helper.
- Program header now writes:
  - `(Source file: <document name>)` — derived from `document-name` / `document-path` / `job-description`.
  - `(Generated: YYYY-MM-DD HH:MM:SS)` timestamp.
  - `(Total time: HH:MM:SS)` — sum of all section cycle times.
- Per-operation comment `(Operation time: HH:MM:SS)`.
- MQL amounts restored / raised:
  - `COOLANT_FLOOD`: Q5 → **Q10**
  - `COOLANT_MIST`: Q10 → **Q30**
  - `deep-drilling` cycle: Q20 → **Q30**

### 12/22/2025 — MIST consistency fix
- `COOLANT_MIST`: fixed mismatch between comment (`Q30`) and block (`Q20`); both lowered to **Q10**.

### 11/19/2025 — Initial MQL tuning
- `COOLANT_FLOOD`: Q10 → **Q5**
- `COOLANT_MIST`: block `Q30` → **Q20** (comment stale `Q30` — fixed next release).
- `deep-drilling` cycle: Q30 → **Q20**.

### 10/20/2025 — Baseline
- First versioned post processor.
- Coolant handling: `FLOOD` = Q10 + M8, `MIST` = Q30 + M8, `AIR` = M209.
- `deep-drilling` cycle emits Q30 oil amount.

## Structure

```
Postprocessors/
├── AP4060/
│   ├── Fusion_360/
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