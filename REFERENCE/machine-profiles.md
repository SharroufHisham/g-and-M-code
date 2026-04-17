# Machine Profiles — MORI SEIKI NL1500/NL2000

## Machine N°1069

| Parameter | OP10 (Main Spindle) | OP20 (Sub-Spindle) |
|-----------|--------------------|--------------------|
| **Safe position** | `G0G53X-150.Z-150.` | `G0G53X-100Z-100` |
| **Init sequence** | `G28V0` / `M69` / `G99G18M46` once at top | Same, once at top |
| **G18 per block** | Yes — `G18` before each N-block | Yes — `G18` before each N-block |
| **Block numbering** | Mixed: N0030, N0630, N0700, N0780, N0860, N0940, N1220, N1350, N1540, N1600, N1745 | Hundreds: N0100, N0200, N0300, N0400, N0500, N0600, N0700 |
| **Tool call style** | `G28V0` then `T0101` on next line | `N0200T0202` on same line (after first block) |
| **G54** | Set once at top | Set once at top |
| **Operation comment** | `( OPERATION 100 )` at top | `(OP.200)` at top |
| **Speed format** | `S0120` (4-digit with leading zero) | `S0120` (4-digit with leading zero) |
| **Macros used** | M2047, M2040, M2042, M28/M29 | M2047, M2040, M28/M29 |
| **Bar stop** | `G65P1111` / `/2G65P1112` | N/A (sub-spindle) |
| **Part catcher** | `M1073` / `M74` | N/A |
| **Program end** | `M5` / `/M99` / `M85` / `M30` | `M5` / `M85` / `G54` / `M30` |

### Key Parts on Machine 1069
- **R10629730** — G1/2 male obturateur (RBE06/IC)

---

## Machine N°1072

| Parameter | OP10 (Main Spindle) | OP20 (Sub-Spindle) |
|-----------|--------------------|--------------------|
| **Safe position** | `G0G53X-150.Z-150.` | `G0G53X-150.Z-150.` |
| **Init sequence** | Repeated per N-block: `G28V0` / `M69` / `G99G18M46` / `G50S...` / `G54` | Same, repeated per N-block |
| **G18 per block** | No — not used between blocks | No — not used between blocks |
| **Block numbering** | Simple: N1, N2, N3, N4... N14 | Simple: N1, N3, N4, N5... N10 |
| **Tool call style** | `G0T0101` (G0 prefix) | `G0T0101` (G0 prefix) |
| **G54** | Repeated each N-block | Repeated each N-block |
| **Operation comment** | `(FACE)`, `(FORET)`, `(D.E.)`, `(D.I.)`, `(FILET)` | Same style |
| **Speed format** | `S130` (no leading zero) | `S130` (no leading zero) |
| **Macros used** | G65P1113 (bar stop), G65P1112 (bar change), M24, M28/M29, M73/M74 | M24, M28/M29, M05/M85/M0 (MECANOIL stop) |
| **Bar stop** | `G65P1113` / `/2G65P1112` | N/A |
| **Part catcher** | `M73` / `M74` | N/A |
| **Program end** | `M9` / `M5` / `/M99` / `M85` / `M30` | `M9` / `M5` / `M85` / `M30` |

### Special: MECANOIL Stop (Machine 1072 OP20 Threading)
Some parts require a MECANOIL (cutting oil) stop between threading phases:
```
M05
M85
M0(MECANOIL)
M86
M03
```
This pauses the machine for the operator to apply cutting oil before threading.

### Key Parts on Machine 1072
- **100004050** — G1/4 female obturateur (RBE03/IC)

---

## Common Macros Reference

| Macro | Description | When Used |
|-------|-------------|-----------|
| `M2047` | Turret indexing/safety | Before roughing operations (Machine 1069) |
| `M2040` | Spindle synchronization | Before threading cycles (Machine 1069) |
| `M2042` | Tool approach | Before some drilling operations (Machine 1069) |
| `M28` | C-axis clamp ON | Before finishing operations |
| `M29` | C-axis clamp OFF | After finishing operations |
| `M24` | Thread synchronization | Before G92 threading cycle |
| `M68` | Chuck clamp | Before hex milling |
| `M69` | Chuck unclamp / mode set | At program init |
| `M73` / `M1073` | Part catcher ON | Before cut-off |
| `M74` | Part catcher OFF | After cut-off |
| `M85` | Sub-spindle mode end | At program end |
| `M86` | Sub-spindle mode start | At program start |
| `G61` | Exact stop mode | For accurate contour profiling (no corner rounding) |
| `G65P1111` | Bar stop call | OP10 start (Machine 1072) |
| `G65P1112` | Bar change call | OP10 start (Machine 1072) |
| `G65P1113` | Bar stop variant | OP10 start (Machine 1072) |

---

## How to Determine Machine Profile

1. Read the Fiche de réglage — it states "Machine: MORI SEIKI - N° XXXX"
2. Read the existing OP10 program header — the DNC number and structure identify the machine
3. Check safe positions in OP10 — they reveal machine travel limits
4. Check for macros — M2047/M2040 = Machine 1069; G65P1111 = Machine 1072
