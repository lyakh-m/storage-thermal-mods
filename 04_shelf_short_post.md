# [MOD] MD1200/SC200 EMM Expander Thermal Fix — 40mm Fan, $3, −10°C

Hey everyone,

Necro-posting in this thread because I've found what I believe is a root cause for the eternal "MD1200 fans won't stay quiet" problem, and a dirt-cheap fix.

## The Problem

We all know the symptom: shelf fans ramp up, we force them down with `set_speed`, they ramp back up. The usual assumption is the shelf is being dumb. It's not.

Connect to the serial console and run `_who`. Look at the EXP sensors — those are the LSI SAS2x36 expander chips. Mine were sitting at **63°C**. The backplane is 29°C, the SIM processors are 38°C, but the expanders are cooking.

Why? They sit in an aerodynamic dead zone behind the EMM connectors. The shelf's main airflow goes around them, not through them. Passive heatsink + stagnant air = 63°C on a 6–8W chip. The shelf sees this and ramps the fans trying to cool them — but main fan speed barely helps because the airflow doesn't reach the problem.

## The Fix

40mm turbine fan (40×40×10mm, 12V, ~80mA) mounted on the EMM cover, blowing directly across the expander heatsink.

## Results

```
Before:  EXP0 = 63°C, EXP1 = 60°C  (both stock)
After:   EXP0 = 63°C, EXP1 = 53°C  (fan on EXP1 only)
```

**−10°C on junction temperature at 1W.** Rough prototype — cleaner build should do better.

This means:
- Thermal cycling stress on BGA cut nearly in half
- Semiconductor lifetime roughly doubled (Arrhenius rule: −10°C ≈ 2× life)
- You can now safely reduce main fan speed without killing your expanders

## Why You Should Care

Those ECC errors some of you have seen? The EMMs that randomly die after a few years? Thermal cycling of BGA solder joints. 40°C delta (23°C room → 63°C operating) on every power cycle, for years. The joints crack. The chip loses contact. EMM dead.

Full write-up with serial console investigation, firmware notes, ECC analysis, and step-by-step mod instructions: [LINK TO FULL ARTICLE — PART 2]

Companion article on MegaRAID 9380/9480 thermal crisis (same root cause, same class of fix): [LINK TO PART 1]

Applies to: **MD1200, MD1220, SC200, SC220** — all use the same EMM design.

**Parts:** ~$3 fan, ~$13 spare EMM from eBay (for hot-swap mod). Total investment: one pizza.

Stop fighting the symptom. Fix the cause.
