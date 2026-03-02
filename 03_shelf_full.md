# Dell MD1200/SC200 EMM Expander Overheating: The Silent Killer

## Or: Why Your Shelf Is Fighting You On Fan Speed

**TL;DR:** The SAS expander chips (LSI SAS2x36) inside MD1200/MD1220/SC200/SC220 EMMs run at 63–65°C due to a dead airflow zone behind the EMM connectors. This slowly kills BGA solder joints through thermal cycling. A 40mm turbine fan on the EMM cover ($3, 1W) drops expander temperature by 10°C+, roughly doubling the chip's lifespan. Here's the full investigation.

---

## The Problem Everyone Knows (But Misidentifies)

*After dealing with a MegaRAID 9380 that was crashing arrays at 105°C (see Part 1), I had a newfound suspicion of passive heatsinks in tight spaces. So I pointed a serial console at my SC200 shelf's EMM modules. What I found was the same disease — just slower-acting.*

You've seen the threads. "MD1200 fans won't spin down." "SC200 fans at 100% driving me insane." The standard advice: get a serial cable (MN657 or equivalent), connect at 38400-8-N-1, and `set_speed 20`. Peace and quiet.

Except the fans spin back up. The shelf "fights you." People set up cron jobs and scripts to keep hammering `set_speed` every few seconds. Some upgrade to firmware 1.06 and use `_shutup` for persistent settings. Problem solved?

**No. Problem hidden.**

The shelf isn't being stubborn. It's trying to tell you something.

## What the Console Actually Shows

Connect to the EMM serial console and run `_who`:

```
Host Links UP          : 4
Expansion Links UP     : 0
Drive(s)               : 0 1 2 3 4 5 6 7 8 9 10 11
EMM (I'm primary and active) : 0 1
Power Supplies         : 1 2
BP_1[2] = 29c
BP_2[3] = 27c
SIM0[0] = 38c
SIM1[1] = 36c
EXP0[4] = 63c
EXP1[5] = 60c
```

Look at those numbers:

- **Backplane sensors** (BP): 27–29°C — fine, ambient plus a bit
- **SIM processors** (SIM0/SIM1): 36–38°C — the ARM cores running EMM firmware, well within spec
- **Expanders** (EXP0/EXP1): 60–63°C — the LSI SAS2x36 chips. **This is the problem.**

The SAS2x36 is a 36-port SAS expander. 6–8W TDP. Not a lot of heat — your average laptop CPU does 15–45W. But the heatsink on the expander is passive. And here's the thing nobody talks about:

## The Aerodynamic Dead Zone

Look at the EMM from behind. Two SAS connectors, a serial port, LEDs. Now look at where the expander chip sits — directly behind those connectors. The shelf's main fans pull air through the drive cages front-to-back, past the backplane, and out through the PSU exhausts. The airflow path goes *around* the EMM modules, not *through* them.

The EMM sits in the connector shadow. The PSU fans create a vacuum on the exhaust side, but the air takes the path of least resistance — through the drive cage openings, not through the tiny gaps around the EMM heatsink. The heatsink sits in effectively stagnant air.

**A passive heatsink in stagnant air with 6–8W of heat = 63°C.** Perfectly logical. And perfectly dangerous.

## Why 63°C Is a Problem

63°C doesn't sound catastrophic for electronics. Lots of chips run hotter. But consider:

**Thermal cycling.** In a homelab, the shelf turns on and off. Room temperature: ~23°C. Operating temperature: 63°C. That's a **40°C delta** on every power cycle. The BGA solder balls under the SAS2x36 expand and contract with each cycle. Mechanical stress on solder joints is proportional to the square of the temperature delta. This is what kills BGA packages — not steady-state heat, but repeated expansion and contraction.

**ECC errors confirm this.** During my investigation, the EMM's memory controller showed:

```
Single BIT errors ID 0/1: 0x00000001
Double BIT errors ID 0/1: 0x00000001
```

These cleared on `_ecc clear` and didn't return — likely POST artifacts from marginal solder joints that settled after warmup. But they're a canary. When the joints degrade further, those errors become permanent, and the EMM dies.

**Arrhenius equation.** The rule of thumb: every 10°C reduction in junction temperature roughly doubles semiconductor lifetime. This isn't marketing — it's the physics of electromigration and crystal lattice degradation.

## The Investigation

### Serial Console Deep Dive

The EMM serial console (38400-8-N-1, needs MN657 cable or pin-compatible) provides full diagnostic access. Key commands:

- `_who` — status + all temperature sensors
- `_temp_rd` — detailed sensor readout
- `_ecc` — ECC error counters
- `_ver` — firmware version for all three flash regions
- `_phyinfo` — PHY link status and speeds
- `set_speed <0-100>` — fan speed override (temporary)

### Thermal Profile

Multiple readings over hours showed consistent pattern:

| Sensor | Description | Temperature | Notes |
|--------|-------------|-------------|-------|
| BP_1, BP_2 | Backplane | 27–32°C | Ambient + slight rise |
| SIM0, SIM1 | ARM processor | 35–38°C | Actively cooled by shelf fans |
| EXP0 | Expander chip 0 | 61–65°C | **Dead zone** |
| EXP1 | Expander chip 1 | 59–63°C | **Dead zone**, slightly cooler |

The 2–4°C delta between EXP0 and EXP1 is consistent — likely positional, one module gets marginally more residual airflow.

### Multipath Verification

Both EMM modules connect to the host controller via separate SAS cables:
- Path0: Controller port → SIM0
- Path1: Controller port → SIM1

All 12 drives show multipath active. This matters because hot-swapping an EMM for modification doesn't take the shelf offline — the surviving EMM maintains all drive paths.

## The Fix: Turbine Fan on EMM Cover

### Concept

If the heatsink is in stagnant air, add airflow. The simplest approach: mount a small fan directly on the EMM module cover, blowing air across the heatsink fins.

### Hardware

- **Fan:** 40x40x10mm turbine fan (e.g., Wathai), 12V, ~80mA (~1W)
- **Power:** [details of your power solution — from the shelf's own 12V or external]
- **Mounting:** On the EMM cover, positioned to blow across the expander heatsink

The heatsink has a bidirectional fin pattern — airflow in any direction is effective.

### Results

Before (stock, no modification):

```
EXP0[4] = 63c
EXP1[5] = 60c
```

After (turbine fan on one EMM, measured after thermal stabilization):

```
EXP0[4] = 63c   ← no fan (control)
EXP1[5] = 53c   ← with fan
```

**10°C reduction at 1W power consumption.** And this was a rough prototype — a cleaner installation could yield 12–15°C.

### What This Means

- Junction temperature delta reduced from 40°C to 30°C (23→53 instead of 23→63)
- BGA thermal stress reduced by roughly half (stress ∝ ΔT²)
- Semiconductor lifetime approximately doubled (Arrhenius)
- 1 watt. Three dollars.

## The Bigger Picture: Why Fans Fight You

Now the shelf's behavior makes sense:

1. Shelf monitors EXP temperature via I²C sensors
2. EXP hits threshold → shelf ramps fans to maximum
3. But main fans don't cool the dead zone effectively — air goes around, not through
4. Fans at 100% barely help EXP — maybe 1–2°C
5. You reduce fan speed for noise → EXP gets even hotter → shelf ramps again
6. Vicious cycle

**The fix isn't louder fans. It's directed airflow on the actual hot spot.**

With the turbine fan mod, you can safely run the main shelf fans at reduced speed for noise control. The expander gets its cooling from the dedicated turbine, not from the shelf fans blowing uselessly past the dead zone.

## Practical Guide

### What You Need

1. 40mm turbine fan (40x40x10mm, 12V, ~80mA)
2. Serial console cable (MN657 or compatible mini-DIN 6-pin)
3. PuTTY or equivalent terminal (38400-8-N-1, XON/XOFF)
4. Thermal paste (optional — MX-4 or similar, if replacing dried stock compound)
5. Spare EMM (recommended — $13–15 on eBay, allows hot-swap modification)

### Procedure

**If you have a spare EMM (recommended):**

1. Prepare spare on workbench: inspect/replace thermal paste, mount turbine fan
2. Hot-swap: pull one production EMM, insert modified spare
3. EMM LED may blink amber (firmware mismatch) — normal, shelf stays operational
4. Connect serial console, verify temperatures with `_who`
5. If firmware mismatch: use `_flashpeer 0` from the other EMM to synchronize
6. Repeat for second EMM

**If modifying in-place:**

1. Power down shelf (coordinate with host — stop I/O first)
2. Remove one EMM, modify, reinstall
3. Power up, verify
4. Repeat for second EMM

### Firmware Notes

- Current latest: **v1.06** (Feb 2016), Dell driver ID 3W1PG, free download
- Changelog: improved I²C handling (temperature sensors), inter-EMM communication, SES fixes
- Your EMMs likely say "MD12XX SERIES SIM MODULE" in FRU regardless of SC200/MD1200 chassis — same hardware
- v1.06 adds persistent fan override via `_shutup` command

### Monitoring

After modification, monitor periodically:

```
_who          — quick status + temps
_ecc          — check for new ECC errors
_temp_rd      — detailed sensor dump
```

Target: EXP temperatures below 55°C under normal operation.

## Applicable Hardware

This issue affects **all** enclosures using the same EMM design:

- Dell PowerVault MD1200 (12× 3.5")
- Dell PowerVault MD1220 (24× 2.5")
- Dell Compellent SC200 (12× 3.5")
- Dell Compellent SC220 (24× 2.5")

All use the same LSI SAS2x36 expander, same passive heatsink, same aerodynamic dead zone.

## Summary

| Parameter | Before | After | Improvement |
|-----------|--------|-------|-------------|
| EXP junction temp | 63°C | 53°C | −10°C |
| Thermal delta (cycle) | 40°C | 30°C | −25% |
| BGA stress (relative) | 1.0× | ~0.56× | −44% |
| Estimated lifetime | 1× | ~2× | +100% |
| Power cost | — | 1W | Negligible |
| Part cost | — | ~$3 | A coffee |

Stop fighting your shelf's fans. Fix the actual problem.

---

*This is Part 2 of a series. Part 1 covers the RAID controller thermal crisis — MegaRAID 9380 at 105°C crashing arrays, the emergency repaste, replacement with 9480, and the channel heatsink + turbine fan mod that brought it down to 45°C. That experience is what led to investigating the shelf.*

*[LINK TO PART 1]*
