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

Switch profile via **SA** (AUX channel) on transmitter.

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
│   └── profile_cinematic.txt    # Profile 1: Cinematic/LR PIDs + rates
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

1. **Flash Block A** — safety fixes (failsafe, GPS rescue, motor KV)
2. **Do tuning flight** — follow `docs/flight_protocol.md`
3. **Analyze blackbox** — follow `docs/filter_analysis.md`
4. **Flash Block B** — filter settings based on analysis
5. **Do another flight** — confirm clean gyro trace
6. **Flash Block C** — PID fine-tune
7. **Repeat flights** — iterate until perfect

---

## Quick Reference — CLI Flash Procedure

1. Connect FC via USB
2. Open Betaflight Configurator → CLI tab
3. Paste contents of the relevant `.txt` file
4. Type `save` and press Enter
5. FC reboots automatically
