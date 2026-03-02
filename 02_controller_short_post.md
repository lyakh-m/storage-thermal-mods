# [MOD] MegaRAID 9380/9480 Thermal Fix — From 105°C Crashes to 45°C Max

If your 9380 (SAS3108) is randomly dropping drives, degrading arrays, or hanging — and the drives themselves are fine — it's almost certainly thermal.

## The Short Version

The SAS3108 on the 9380 hits **105°C** under load in a workstation chassis. At that temperature, the controller throttles, misses SAS timeouts, drops drives. Your array degrades. You rebuild. It happens again.

The worst part: **mine worked fine for 5 years** (installed 2019). Factory thermal paste slowly degraded — 70°C became 80°C, then 90°C, then 105°C. No warning, no alarm. Just one day: arrays crashing. Five years of "rock solid storage" breeds the complacency that kills your data.

## Quick Fix (30 minutes)

Pull the card, remove heatsink, replace thermal paste. Factory paste is dried cement after 3+ years.

Result: **105°C → 65–70°C.** Crashes stop. But margin is thin.

## Permanent Fix

I replaced the 9380 with a 9480 (lower TDP), then:

1. Converted the passive finned heatsink to **channel design** (capped the open fin tops → enclosed air passages)
2. Added a **5V turbine fan** (~100mA, 0.5W) with a **plenum** to distribute airflow into all channels
3. Result: **45°C max under any load** at 22–24°C ambient

That's 60°C of thermal margin. The card now runs cooler at full tilt than the stock 9380 ran at idle.

## Why This Matters

- 9380 at 105°C: thermal throttling, data corruption risk, array crashes
- 9380 repasted at 70°C: stable but sweating
- 9480 stock at 85°C: better but still hot for a workstation
- 9480 modded at 45°C: bulletproof

**If you have a 9380/9400-series and random array issues — check `storcli /c0 show temperature` before you blame your drives.**

Full write-up with thermal analysis, heatsink modification details, and before/after measurements: [LINK TO FULL ARTICLE — PART 1]

Companion article on the same issue in MD1200/SC200 enclosure EMMs: [LINK TO PART 2]
