# Tuning Flight Protocol
# Nazgul DC5 V2 — Betaflight 4.5.0

This is the flight pattern for capturing blackbox data for filter and PID analysis.  
**Total flight time:** ~75 seconds  
**Blackbox settings required before flight (CLI):**

```
set debug_mode = GYRO_SCALED
set blackbox_sample_rate = 1/2
set blackbox_device = SDCARD
save
```

---

## Pre-Flight Checklist

- [ ] Block A flashed (safety fixes)
- [ ] Blackbox settings applied (debug_mode = GYRO_SCALED)
- [ ] SD card inserted and formatted (FAT32)
- [ ] Profile 0 active (Freestyle for the tuning flight)
- [ ] GPS fix acquired (green LED) — min 8 satellites
- [ ] Home point set (arm and disarm once on ground)
- [ ] Props on and tight
- [ ] Battery fully charged

---

## Flight Pattern (75 seconds)

Fly in a clear, open area. No wind if possible.

| Time | Maneuver | Purpose |
|------|----------|---------|
| 0–5s | Hover at 1.5m | Baseline noise floor |
| 5–15s | Slow forward flight, gentle turns | Mid-throttle gyro noise |
| 15–25s | **Hard throttle punches** (5× throttle 0→100→0) | Motor noise harmonics |
| 25–35s | **Snap rolls left and right** (full stick, fast) | P/D response check |
| 35–45s | **Snap flips forward and back** | Pitch P/D response |
| 45–55s | **Consecutive fast rolls** (3-4 in a row) | Propwash detection |
| 55–65s | **Power dives + pull out** (2×) | High-throttle D-term |
| 65–75s | Slow hover, land gently | End baseline |

> **Important:** Do all maneuvers at mid-field. Keep throttle punches vertical (straight up/down) to avoid flyaway. Stay in visual range.

---

## After Flight

1. Disarm quad
2. Wait 10 seconds (blackbox write finishes)
3. Remove SD card
4. Copy `.bbl` log file to `blackbox/` folder in this repo
5. Follow `docs/filter_analysis.md` for analysis steps

---

## Blackbox Explorer Install (Mac)

Download from: https://github.com/betaflight/blackbox-log-viewer/releases  
Current version: latest release for macOS (`.dmg`)

---

## Notes

- Fly with **Profile 0 (Freestyle)** for all tuning flights
- Use **default filters first** — Block B only after seeing the noise floor
- If motors get hot mid-flight, land immediately (D too high or filter issue)
- Temperature affects gyro noise — fly in similar conditions each time
