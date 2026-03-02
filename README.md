# Storage Thermal Mods

Practical thermal fixes for Dell SAS enclosures and Broadcom MegaRAID controllers in homelab / workstation environments.

## The Problem

Passive heatsinks in tight spaces with no directed airflow. Works fine for years — then thermal paste degrades, temperatures creep up silently, and hardware starts failing. By the time you notice, you're losing data.

## Articles

### Part 1: MegaRAID 9380/9480 — From 105°C Crashes to 45°C Under Any Load

A MegaRAID 9380 that worked perfectly for 5 years started crashing arrays. Root cause: factory thermal paste degraded to dust, junction temperature hit 105°C. Emergency repaste brought it to 70°C. Replacement with 9480 + channel heatsink conversion + turbine fan with plenum brought it to **45°C max under any load.**

**[Read full article →](01_controller_full.md)**

Quick version for forums: **[Short post](02_controller_short_post.md)**

---

### Part 2: Dell MD1200/SC200 EMM Expander — The Silent Killer

The SAS expander chips (LSI SAS2x36) inside MD1200/SC200 EMMs run at 63°C due to an aerodynamic dead zone. This slowly kills BGA solder joints. A 40mm turbine fan ($3, 1W) drops temperature by 10°C, roughly doubling chip lifespan.

**[Read full article →](03_shelf_full.md)**

Quick version for forums: **[Short post](04_shelf_short_post.md)**

---

### Part 3: Fan Controller With Temperature Monitoring *(coming soon)*

A channel heatsink without monitoring is a trap — if the fan dies, sealed channels are worse than open fins. This part covers a dedicated controller that watches temperature and fan health.

---

## Applicable Hardware

| Component | Models |
|-----------|--------|
| RAID Controllers | MegaRAID 9380-series (SAS3108), 9480-series (SAS3516), Fujitsu PRAID EP420i/EP540e |
| SAS Enclosures | Dell PowerVault MD1200, MD1220, Compellent SC200, SC220 |
| EMM Modules | Any with LSI SAS2x36 expander (part 00TW47A00 or equivalent) |

## Key Findings

- **MegaRAID 9380:** 105°C → array crashes. Repaste → 70°C. 9480 + channel heatsink + turbine → **45°C**
- **MD1200/SC200 EMM:** 63°C in dead zone. Turbine fan on EMM cover → **53°C** (−10°C at 1W)
- **Thermal paste shelf life:** ~5 years before degradation causes problems. Check proactively.
- **Arrhenius rule:** −10°C junction temp ≈ 2× semiconductor lifespan

## License

MIT — do whatever you want with this. If it saves your data, that's payment enough.
