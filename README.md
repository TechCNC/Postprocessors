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