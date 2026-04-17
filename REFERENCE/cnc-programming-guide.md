# CNC Programming Guide — MORI SEIKI NL1500/NL2000

## 1. Program Header Format

```
#PROG:MORI-NL1500-NL2000-U{DNC}-{REV}-2-TST
%
O{DNC}({PART_NUMBER} ={REV_LETTER}= OP{NN})
```

### Fields
| Field | Source | Example |
|-------|--------|---------|
| `{DNC}` | Fiche de réglage "DNC" field (remove spaces) | `29235200` |
| `{REV}` | Program revision (from fiche or source control) | `02` |
| `{REV_LETTER}` | Part revision letter | `F` |
| `{PART_NUMBER}` | Part number with spaces | `R 106 297 30` |
| `{NN}` | Operation number: `10` or `20` | `20` |

### Example (Machine 1069, OP20)
```
#PROG:MORI-NL1500-NL2000-U29235200-02-2-TST
%
O29235200(R 106 297 30 =F= OP20)
```

### Example (Machine 1072, OP20)
```
#PROG:MORI-NL1500-NL2000-U26090801-01-2-TST
%
O26090801(1 000 040=A= OP 20)
```

---

## 2. Program Initialization

### Machine 1069 Style (Single Init Block)
```
M86
G0G53X-100Z-100          ← OP20 safe position (Machine 1069)
G28V0
M69
G99G18M46
G54
(OP.200)
G18
```

### Machine 1072 Style (Repeated per N-Block)
```
M86
G0G53X-150.Z-150.        ← Safe position (Machine 1072)

N1
(FACE)
G28V0
M69
G99G18M46
G50S3000
G54
G0T0101
...
```

### Key Differences
| Feature | Machine 1069 | Machine 1072 |
|---------|-------------|-------------|
| Init blocks | Once at top | Repeated per N-block |
| G18 | Before each N-block | Not repeated |
| G54 | Once at top | Per N-block |
| G50 | Per tool block | Per N-block |
| Tool call | `T0101` alone | `G0T0101` |

---

## 3. Operation Block Structure

### Machine 1069 OP20 Template
```
G18                           ← Plane select before each block
N0{X}00T{TT}{TT}             ← Block number + tool call
G0Y0                          ← Y-axis home
[G50S{XXXX}]                  ← Max RPM limit (CSS mode only)
G96S0{XXX}M3  or  G97S{XXXX}M3  ← CSS or constant RPM
[M28]                         ← C-axis clamp (finishing only)
G0G99X{xx.xxx}Z{zz.zzz}      ← Rapid to start position
[M2047 or M2040]              ← Machine macro if needed
M8                            ← Coolant ON
... toolpath ...
M9  or  ...M9                 ← Coolant OFF (can be on last move)
G0G53X-100Z-100               ← Safe position
G28V0                         ← V-axis home
[M29]                         ← C-axis unclamp (if M28 was used)
M01                           ← Optional stop
```

### Machine 1072 OP20 Template
```
N{X}
({OPERATION_NAME})
G28V0
[M28]                         ← If finishing
M69
G99G18M46
G50S{XXXX}
G54
G0T{TT}{TT}
G0Y0
G96S{XXX}M3  or  G97S{XXXX}M3
G0X{xx.}Z{zz.z}
M8
... toolpath ...
M9
[Z2.1]                        ← Retract Z
G28V0
G53X-150.Z-150.
[M29]                         ← If M28 was used
M01
```

---

## 4. Speed Mode Selection

| Operation | Mode | Format (1069) | Format (1072) |
|-----------|------|---------------|---------------|
| Roughing | CSS (G96) | `G96S0120M3` | `G96S130M3` |
| Drilling | Constant RPM (G97) | `G97S1200M3` | `G97S1200M3` |
| Threading | Constant RPM (G97) | `G97S1000M3` | `G97S1000M3` / `G97S1400M3` |
| Finishing | CSS (G96) | `G96S0140M3` | `G96S160M3` |
| Internal finish | CSS (G96) | `G96S0140M3` | `G96S150M3` |

### G50 Max RPM Limits (typical)
| Operation | G50 Limit |
|-----------|-----------|
| Roughing (external) | S2000–S2500 |
| Finishing (external) | S2000–S3000 |
| Finishing (internal) | S1500–S2000 |
| Drilling | S3000 |

---

## 5. Typical OP20 Operation Sequence (Obturateur Family)

### Step 1: Face + Rough External (T01)
- Face the OP20 end
- Rough the external profile: shoulders, tapers, thread area
- Uses G96 CSS mode with G61 exact stop for contour accuracy
- Includes G3 arcs for radius blends at taper entries

### Step 2: Drill (T02)
- Center drill or through-drill from OP20 side
- Uses G97 constant RPM
- Always includes G4 dwell at bottom
- Depth includes drill point allowance

### Step 3: Rough Internal Profile (T04)
- **NOT always cylindrical boring** — if bore exists from OP10 drill, this cuts the entry taper/chamfer
- Example: 14° taper from Ø17.543 down to Ø12 at Z-4.8

### Step 4: Thread (T05)
- G92 threading cycle
- Multiple passes from major → minor Ø (external) or minor → major Ø (internal)
- F-value = thread pitch in mm
- Machine macro M2040 before threading (Machine 1069)

### Step 5: Finish External (T03)
- Complete external profile with arc interpolation (G2/G3)
- M28/M29 C-axis clamp for finish accuracy
- Multiple sections: face profile, shoulder transitions, thread relief

### Step 6: Thread Spring Pass (T05 again)
- Single G92 pass at final diameter for thread finishing
- Same pitch (F-value) as roughing

### Step 7: Finish Internal (T06)
- Precision bore profile per drawing/profile spec
- G2/G3 arcs for entry radii
- M28/M29 C-axis clamp

---

## 6. G92 Threading Cycle

### Format
```
G92 X{first_pass_diameter} Z{thread_end} F{pitch}
X{second_pass}
X{third_pass}
...
X{final_pass}
```

### Rules
1. **External thread**: X values DECREASE (cutting inward)
2. **Internal thread**: X values INCREASE (cutting outward)
3. **Pitch (F)**: Must match thread specification exactly
4. **Pass progression**: Typically starts with larger cuts, finishing with smaller increments
5. **Spring pass**: Last operation repeats the final diameter for clean finish

### External G1/2 BSP Example (R10629730 OP20)
```
G92X20.15Z-13.5F1.814     ← Major Ø 20.955, first cut at 20.15
X19.85
X19.7
X19.5
X19.32
X19.15
X19.
X18.87
X18.75
X18.63
X18.53
X18.47
X18.449
X18.45                     ← Near minor Ø 18.631
```

### Internal G1/4 BSP Example (100004050 OP20)
```
G92X11.70Z-10.F1.337      ← Minor Ø ~11.445, first cut at 11.70
X11.75
X11.85
...
X13.3                      ← Near major Ø 13.157
```

---

## 7. Arc Interpolation (G2/G3)

### Format
```
G2 X{end_x} Z{end_z} R{radius}    ← Clockwise arc
G3 X{end_x} Z{end_z} R{radius}    ← Counter-clockwise arc
```

### When to Use
- Taper entry blends (radius at transition from cylinder to taper)
- Shoulder radius transitions
- Thread relief entry/exit radii
- Internal bore profile entry radii (per profile spec)

### Typical Radius Values from Reference Programs
| Feature | Radius | Context |
|---------|--------|---------|
| Taper entry blend | R0.93–R1.1 | External roughing |
| Shoulder transition | R0.4–R0.5 | External finishing |
| Thread relief | R0.4–R0.42 | Finish profile |
| Internal bore entry | R0.22–R0.3 | Internal finishing |
| Profile spec entry | R0.23 | Per spec document |

---

## 8. Feed Rate Guidelines

| Operation | Feed Range (mm/rev) | Notes |
|-----------|-------------------|-------|
| Facing | F0.15–F0.3 | Slower near center |
| External roughing | F0.12–F0.4 | Varies by cut depth |
| External finishing | F0.04–F0.08 | Precision surface |
| Drilling | F0.06–F0.12 | Depends on drill type |
| Internal roughing | F0.15 | Conservative |
| Internal finishing | F0.03–F0.06 | Precision bore |
| Threading | F{pitch} | Always = thread pitch |
| Cut-off | F0.02–F0.05 | Slow for clean cut |

---

## 9. Coolant and Dwell

### Coolant
- `M8` — Coolant ON (at start of cutting)
- `M9` — Coolant OFF (at end of cutting, before rapid retract)
- Can combine: `G0Z2.M9` (retract with coolant off)

### Dwell
- `G4X.2` — 0.2 second dwell (standard for drilling)
- `G4X0.1` — 0.1 second dwell (for bore finishing)
- `G4X1.` — 1.0 second dwell (for long retracts)

---

## 10. Program End

### Machine 1069 OP20
```
M5          ← Spindle stop
M85         ← Sub-spindle mode end
G54         ← Reset work offset
M30         ← Program end + rewind
%
```

### Machine 1072 OP20
```
M9          ← Coolant off (if not already)
M5          ← Spindle stop
M85         ← Sub-spindle mode end
M30         ← Program end + rewind
%
```

### Machine 1069 OP10
```
M5          ← Spindle stop
/M99        ← Conditional subprogram return
M85         ← Mode end
M30         ← Program end + rewind
%
```

---

## 11. Data Flow: Input Files → Program

```
┌─────────────────────┐
│ Plan.json / Plan.md  │ → Part dimensions, thread specs, tolerances
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Fiche de réglage     │ → Machine number, DNC, tool assignments, pitches
│ (CSV + MD)           │
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Existing OP10 Program│ → Machine style, macros, safe positions, structure
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Profile Spec (if any)│ → Internal bore profile geometry
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Thread Reference     │ → Verified pitch from TPI calculation
└─────────┬───────────┘
          │
     ┌────▼────┐
     │ GENERATE │
     │  OP20    │
     └────┬────┘
          │
     ┌────▼─────────┐
     │ VALIDATE      │ ← Check every value against sources
     └───────────────┘
```

---

## 12. Validation Checklist

Before declaring a program complete:

1. **Header**: DNC matches fiche exactly, revision is correct
2. **Safe positions**: Match same part's OP10 (or OP20 sub-spindle limit)
3. **Init structure**: Matches same part's OP10 coding style
4. **Block numbering**: Matches same part's OP10 scheme
5. **Tool numbers**: Match Fiche de réglage tool chart exactly
6. **Thread pitch**: Computed from TPI reference table, verified against OP10
7. **Thread direction**: X progression matches internal/external correctly
8. **Machine macros**: All M2xxx from OP10 are included in OP20
9. **Coordinates**: Every X/Z value traces to engineering data
10. **Feeds/speeds**: Reasonable for operation type and material
11. **Coolant**: M8 before cutting, M9 before retract
12. **Dwells**: G4 at drill bottoms and bore bottoms
13. **M28/M29**: Bracket all finishing operations
14. **Program end**: Matches machine convention exactly
