# Thread Pitch Reference — BSP & Metric

## BSP Parallel Threads (G-series, ISO 228-1)

BSP pitch is calculated as: **pitch_mm = 25.4 / TPI**

| Thread Size | TPI | Pitch (mm) | Major Ø External (mm) | Major Ø Internal (mm) | G92 F-value |
|-------------|-----|------------|----------------------|----------------------|-------------|
| G1/8        | 28  | 0.907      | 9.728                | 8.566                | F0.907      |
| G1/4        | 19  | 1.337      | 13.157               | 11.445               | F1.337      |
| G3/8        | 19  | 1.337      | 16.662               | 14.950               | F1.337      |
| G1/2        | 14  | 1.814      | 20.955               | 18.631               | F1.814      |
| G5/8        | 14  | 1.814      | 22.911               | 20.587               | F1.814      |
| G3/4        | 14  | 1.814      | 26.441               | 24.117               | F1.814      |
| G1          | 11  | 2.309      | 33.249               | 30.291               | F2.309      |
| G1-1/4      | 11  | 2.309      | 41.910               | 38.952               | F2.309      |
| G1-1/2      | 11  | 2.309      | 47.803               | 44.845               | F2.309      |
| G2          | 11  | 2.309      | 59.614               | 56.656               | F2.309      |

### ⚠️ COMMON MISTAKE
**G1/2 BSP pitch is 1.814mm, NOT 1.5mm.** The 1.5mm value is the pitch of M20x1.5 metric thread, which has a similar diameter but is a completely different thread.

### How to Verify BSP Thread in a Program
1. Check the G92 F-value — it must match the table above
2. Check the X diameter range — it must be near the major/minor Ø from the table
3. Check the Fiche de réglage tool description — look for "P1,814" or "P1,337" etc.

## Metric Threads (ISO 261)

| Thread | Pitch (mm) | Major Ø (mm) | G92 F-value |
|--------|-----------|-------------|-------------|
| M3x0.5 | 0.5       | 3.0         | F0.5        |
| M4x0.7 | 0.7       | 4.0         | F0.7        |
| M5x0.8 | 0.8       | 5.0         | F0.8        |
| M6x1   | 1.0       | 6.0         | F1.0        |
| M8x1.25| 1.25      | 8.0         | F1.25       |
| M10x1.5| 1.5       | 10.0        | F1.5        |
| M12x1.75| 1.75     | 12.0        | F1.75       |
| M14x1  | 1.0       | 14.0        | F1.0        |
| M14x1.5| 1.5       | 14.0        | F1.5        |
| M14x2  | 2.0       | 14.0        | F2.0        |
| M16x1.5| 1.5       | 16.0        | F1.5        |
| M16x2  | 2.0       | 16.0        | F2.0        |
| M20x1.5| 1.5       | 20.0        | F1.5        |
| M20x2.5| 2.5       | 20.0        | F2.5        |

### How to Distinguish BSP from Metric
- BSP threads are designated "G" (e.g., G1/2, G1/4)
- Metric threads are designated "M" (e.g., M14x1)
- BSP has non-standard pitches (1.337, 1.814, 2.309)
- Metric fine pitches often overlap with BSP diameters — **always check the designation, not just the diameter**

## Thread Direction in G92 Cycle

```
EXTERNAL THREAD (cutting inward):
G92 X20.15 Z-13.5 F1.814    ← first pass at largest diameter
X19.85                        ← decreasing X
X19.7
...
X18.45                        ← last pass at smallest diameter (near minor Ø)

INTERNAL THREAD (cutting outward):
G92 X12.9 Z-8.0 F1.0        ← first pass at smallest diameter
X13.0                         ← increasing X
X13.1
...
X13.98                        ← last pass at largest diameter (near major Ø)
```

## Verified Examples from This Repository

### 100004050 — G1/4 BSP (pitch 1.337mm)
- **OP10 external**: G92 X12.66→11.37 Z-7.7 F1.337 (X decreasing = external)
- **OP20 internal**: G92 X11.70→13.3 Z-10 F1.337 (X increasing = internal)

### R10629730 — G1/2 BSP (pitch 1.814mm)
- **OP10 thread zone**: G92 X13.75→12.06 Z-9.1 F1.5 (⚠️ This is NOT the G1/2 thread — it is a separate M-thread zone on the OP10 side with pitch 1.5mm. The G1/2 BSP thread with pitch 1.814mm is cut in OP20.)
- **OP20 external G1/2**: G92 X20.15→18.45 Z-13.5 F1.814 (X decreasing = external G1/2 BSP)
