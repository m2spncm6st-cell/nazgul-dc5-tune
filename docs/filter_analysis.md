# Filter Analysis Guide
# Nazgul DC5 V2 — Betaflight Blackbox

Use this after the tuning flight to determine correct filter settings for Block B.

---

## Tools Needed

- **Blackbox Explorer**: https://github.com/betaflight/blackbox-log-viewer/releases
- Your `.bbl` log file in the `blackbox/` folder

---

## Step 1: Open Log

1. Open Blackbox Explorer
2. Drag `.bbl` file onto the window
3. Click the **Spectrum Analyser** button (graph icon, top right)

---

## Step 2: Check Gyro Noise (Pre-filter)

Select **Gyro Scaled (pre-filter)** from the dropdown.

Look for:
- **Noise floor**: the base level of noise across all frequencies
- **Motor noise spikes**: peaks at motor frequency and harmonics
  - With tri-blade props, you'll see spikes at 1×, 2×, 3× motor frequency
- **Structural resonance**: any fixed-frequency spike (frame, prop resonance)

**ICM42688P is a clean gyro** — the noise floor should be low.  
If RPM filter is working, motor harmonics should be well-suppressed.

**Target:** 
- Gyro LPF1 cutoff = ~2× the highest significant noise peak (usually 250–400 Hz)
- If noise floor is clean above 300 Hz → you can push LPF1 higher = less latency

---

## Step 3: Check D-term Noise

Select **D-term (filtered)** and **D-term (unfiltered)** and compare.

Look for:
- How much noise is in unfiltered D-term
- Whether filtered D-term is clean without killing too much signal

**Target:**
- D-term LPF1 cutoff: typically 75–120 Hz
- D-term LPF2: can be disabled if LPF1 + RPM filter is sufficient

---

## Step 4: Dynamic Notch Check

Select **Gyro (post-filter)**. Look for any remaining spikes after RPM filter.

If there are no significant spikes:
- Set `dyn_notch_count = 1` (minimal) or even `= 0` (off)
- This removes latency

If there are still spikes:
- Keep dynamic notch active, tune `dyn_notch_q` higher for narrower notches

---

## Step 5: Fill in Block B

Based on your analysis, fill in the `[PLACEHOLDER]` values in `tune/block_B_filters.txt`.

Typical values for a clean ICM42688P quad with good RPM filter:

```
gyro_lpf1_dyn_min_hz = 250
gyro_lpf1_dyn_max_hz = 500
gyro_lpf2_hz = 0          # Disable — RPM filter does the work
dyn_notch_count = 1
dterm_lpf1_dyn_min_hz = 75
dterm_lpf1_dyn_max_hz = 150
dterm_lpf2_hz = 0          # Disable if D-term is clean
```

---

## What to Watch Out For

| Symptom | Cause | Fix |
|---------|-------|-----|
| Spiky noise at RPM harmonics | RPM filter not working | Check BiDir DSHOT in Motors tab (R: value changes) |
| Broad noise floor 200–500 Hz | LPF1 too high / gyro noise | Lower LPF1 cutoff |
| Motors hot after flight | D-term too high or LPF too low | Raise D-term LPF or lower D gains |
| Oscillation at stick input | P too high (see pid_analysis.md) | |
| Clean spectrum everywhere | ✅ Great — push filters higher for less latency | |
