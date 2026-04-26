# iFlight Nazgul DC5 V2 — Betaflight Tune Project

**Date started:** 2026-04-25  
**Betaflight version:** 4.5.0  
**FC:** iFlight BLITZ F7 (STM32F7X2)

---

## Hardware

| Component | Spec |
|-----------|------|
| Frame | iFlight Nazgul DC5 V2 |
| FC | iFlight BLITZ F722 |
| ESC | iFlight BLITZ E55S 55A 4-in-1 |
| Motors | ~1800 KV (5" freestyle) |
| Gyro | ICM42688P |
| Props | Tri-blade 5" |
| RX | ELRS 2.4GHz @ 250Hz (RadioMaster GX12) |
| GPS | Yes (for GPS Rescue over water) |
| Video | Analog (FPV cam) |

---

## Profiles

| Profile | Use case | Rate profile |
|---------|----------|-------------|
| **0** | Freestyle | 0 — high stick feel, punchy |
| **1** | Cinematic / Long Range | 1 — smooth, slow expo |

Switch profile via **AUX6**:
- AUX6 **low** (900–1300) → Profile 0 (Freestyle)
- AUX6 **high** (1700–2100) → Profile 1 (Cinematic)

Setup: flash `tune/profile_switch_aux6.txt` after both profiles are flashed.

---

## Failsafe

- **GPS Rescue** always active (flies over water)
- GPS Rescue also mapped to **AUX 3** as manual switch (1825–2100)
- `failsafe_delay = 15` (1.5 seconds)
- Strict GPS sanity checks enforced

---

## Repository Structure

```
.
├── README.md                    # This file
├── original/
│   └── BTFL_cli_20260424_173744_IFLIGHT_BLITZ_F722.txt  # Factory dump
├── tune/
│   ├── block_A_safety.txt       # Block A: Safety fixes (flash first)
│   ├── block_B_filters.txt      # Block B: Filter optimization (after blackbox)
│   ├── block_C_pids.txt         # Block C: PID fine-tune (after blackbox)
│   ├── profile_freestyle.txt    # Profile 0: Freestyle PIDs + rates
│   ├── profile_cinematic.txt    # Profile 1: Cinematic/LR PIDs + rates
│   └── profile_switch_aux6.txt  # AUX6 toggle between profile 0/1
├── blackbox/                    # Flight logs go here
│   └── .gitkeep
├── docs/
│   ├── flight_protocol.md       # How to do the tuning flight
│   ├── filter_analysis.md       # How to analyze blackbox for filters
│   └── pid_analysis.md          # How to analyze blackbox for PIDs
└── .gitignore
```

---

## Tuning Workflow

1. **Flash Block A** — safety fixes (failsafe, GPS rescue, RPM filter)
2. **Flash Profile 0** (`profile_freestyle.txt`)
3. **Flash Profile 1** (`profile_cinematic.txt`)
4. **Flash Profile Switch** (`profile_switch_aux6.txt`)
5. **Verify in Configurator** — toggle AUX6, confirm profile changes
6. **Do tuning flight** — follow `docs/flight_protocol.md`
7. **Analyze blackbox** — follow `docs/filter_analysis.md`
8. **Flash Block B** — filter settings based on analysis
9. **Repeat flight** — confirm clean gyro trace
10. **Flash Block C** — PID fine-tune
11. **Iterate** — one variable at a time per flight

---

## Quick Reference — CLI Flash Procedure

1. Connect FC via USB
2. Open Betaflight Configurator → CLI tab
3. Paste contents of the relevant `.txt` file
4. Type `save` and press Enter
5. FC reboots automatically
