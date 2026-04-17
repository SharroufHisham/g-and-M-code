# Lessons Learned — CNC G-Code Generation Errors

This document catalogs every error made during AI-assisted G-code generation, with root causes and prevention rules.

---

## Error #1: WRONG THREAD TYPE
**Severity: 🔴 CRITICAL — would crash machine or produce scrap**

| | |
|---|---|
| **What happened** | Generated M14×1 internal thread (F1.0, X12.9→13.98) instead of G1/2 BSP external thread (F1.814, X20.15→18.45) |
| **Root cause** | Plan.json incorrectly stated OP20 thread as "M14x1 internal". The AI trusted this without verification. |
| **How to prevent** | 1. ALWAYS cross-check thread type against G92 X-value direction (decreasing = external, increasing = internal). 2. Verify pitch against BSP/metric reference tables. 3. Check the Fiche de réglage tool description — "Fileter Ebauche" with no pitch info should trigger a verification step. |

---

## Error #2: WRONG THREAD PITCH
**Severity: 🔴 CRITICAL — produces wrong thread**

| | |
|---|---|
| **What happened** | Used F1.0 (M14×1) instead of F1.814 (G1/2 BSP 14 TPI) |
| **Root cause** | Plan.json listed G1/2 pitch as 1.5mm (which is actually M20×1.5 metric). The 1.5mm value was wrong. |
| **How to prevent** | ALWAYS calculate BSP pitch from TPI: G1/2 = 14 TPI → 25.4/14 = 1.814mm. Never use memorized values — compute from the table. |

---

## Error #3: WRONG SAFE POSITIONS
**Severity: 🟠 MAJOR — could cause collision on sub-spindle**

| | |
|---|---|
| **What happened** | Used `G0G53X-150.Z-150.` (main spindle limit) for OP20 sub-spindle, which only has X-100 Z-100 travel |
| **Root cause** | Copied safe positions from 100004050-OP20 (Machine 1072, which uses X-150) instead of checking R10629730-OP10 (Machine 1069, which uses X-100 for sub-spindle) |
| **How to prevent** | ALWAYS read safe positions from the same part's OP10 program. Different machines have different sub-spindle travel limits. |

---

## Error #4: WRONG PROGRAM STRUCTURE
**Severity: 🟡 MODERATE — program may not run correctly**

| | |
|---|---|
| **What happened** | Used Machine 1072 style (repeated init blocks, N1/N2 numbering, no G18) for a Machine 1069 part (single init, N0100 numbering, G18 per block) |
| **Root cause** | Used 100004050-OP20 as the structural template instead of following R10629730-OP10's coding conventions |
| **How to prevent** | ALWAYS use the same part's OP10 as the structural template for OP20. Never mix machine styles. |

---

## Error #5: MISUNDERSTOOD T0404 "EBAUCHE INT"
**Severity: 🟠 MAJOR — completely wrong toolpath geometry**

| | |
|---|---|
| **What happened** | Programmed T0404 as multi-pass step-boring (10 passes, Ø10.5→12.8, Z-10) instead of single taper cut (Ø17.543→12, Z-4.8) |
| **Root cause** | Interpreted "Ebauche INT" as "rough internal bore" = cylindrical boring. Actually it cuts the 14° entry taper from face diameter down to bore diameter. The bore was already drilled in T0202. |
| **How to prevent** | 1. Check what the previous operation (drilling) already created. 2. "Ebauche INT" with a contour insert (R0.4) = profile/taper tool. 3. Look at the part's taper angles in Plan.json — if there's a 14° entry taper, that's what T0404 cuts. |

---

## Error #6: WRONG DNC PROGRAM NUMBER
**Severity: 🟡 MODERATE — program won't load on machine**

| | |
|---|---|
| **What happened** | Used O29335200 instead of O29235200 (digit "3" vs "2" in position 3) |
| **Root cause** | Fiche de réglage OCR/transcription ambiguity: "U 293 35200" was misread. The correct number is "U 292 35200". |
| **How to prevent** | Cross-check DNC number against the OP10 DNC. OP10 = U28529600, OP20 = U29235200. The numbers should be different but from the same series. If in doubt, flag the ambiguity. |

---

## Error #7: MISSING MACHINE MACROS
**Severity: 🟡 MODERATE — machine may not initialize correctly**

| | |
|---|---|
| **What happened** | Missing M2047 (turret indexing) and M2040 (spindle sync for threading) |
| **Root cause** | Machine macros were not documented in any input file. They only appear in the actual verified programs. |
| **How to prevent** | ALWAYS scan the existing OP10 program for M2xxx macros and replicate them in OP20 where the same tool types are used (roughing → M2047, threading → M2040). |

---

## Error #8: WRONG DRILL DEPTH
**Severity: 🟡 MODERATE — could under-drill or over-drill**

| | |
|---|---|
| **What happened** | Used Z-16 instead of Z-18.08 for drill depth |
| **Root cause** | Guessed drill depth from part geometry. The actual depth includes drill tip allowance and clearance that requires engineering calculation. |
| **How to prevent** | Drill depths must come from engineering data or be calculated as: bore_depth + drill_point_allowance + clearance. Never guess. |

---

## Error #9: MISSING DWELL (G4)
**Severity: 🟢 MINOR — affects surface finish at hole bottom**

| | |
|---|---|
| **What happened** | No G4 dwell at drill bottom |
| **Root cause** | Oversight — didn't include standard practice |
| **How to prevent** | ALWAYS add G4 dwell at drill bottom: `G4X.2` (0.2 seconds). This is standard practice for clean bottom finish and chip clearing. |

---

## Error #10: WRONG G50 SPEED LIMITS
**Severity: 🟢 MINOR — could affect surface speed or cause alarm**

| | |
|---|---|
| **What happened** | Used G50S2000 instead of G50S2500 for roughing |
| **Root cause** | Copied value from different part's program |
| **How to prevent** | G50 speed limits should match the tool capability and part diameter. When in doubt, use the same part's OP10 values for the same tool types. |

---

## Error #11: EXTRA/MISSING PROGRAM END CODES
**Severity: 🟢 MINOR**

| | |
|---|---|
| **What happened** | Had extra M9 (coolant already off), missing G54 (work offset reset) at program end |
| **Root cause** | Copied end sequence from wrong template |
| **How to prevent** | Match program end sequence exactly to the same machine's convention. Machine 1069 OP20: `M5` / `M85` / `G54` / `M30`. Machine 1072 OP20: `M9` / `M5` / `M85` / `M30`. |

---

## Summary: Prevention Checklist

Before finalizing any generated program, verify:

- [ ] Thread pitch matches BSP/metric reference table (compute from TPI, don't memorize)
- [ ] Thread direction matches G92 X-value progression (decreasing = external, increasing = internal)
- [ ] Safe positions match the same part's OP10 (not another part)
- [ ] Program structure matches same part's OP10 style (numbering, init blocks, G18)
- [ ] DNC number matches Fiche de réglage exactly (cross-check against OP10 DNC)
- [ ] Machine macros (M2047, M2040, etc.) are included where OP10 uses them
- [ ] T0404 operation matches actual geometry (taper vs bore — check what drill already created)
- [ ] Drill depths include proper allowances (not guessed from bore depth alone)
- [ ] G4 dwell included at drill bottoms
- [ ] Program end sequence matches machine convention
- [ ] All X/Z coordinates come from verified engineering data (not invented)
