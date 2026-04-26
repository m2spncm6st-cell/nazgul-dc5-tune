# Filter Analysis Guide
# Nazgul DC5 V2 — Betaflight 4.5.0
# Based on Chris Rosser BF 4.5 Filter Tuning Masterclass

---

## Tools needed

- **Blackbox Explorer** — https://github.com/betaflight/blackbox-log-viewer/releases
- Log file from tuning flight (`.bbl` in `blackbox/` folder)

---

## Blackbox Settings (set before filter flight)

```
# BF 4.5: raw gyro is ALWAYS logged regardless of debug mode
# Ideal logging rate: 1.6K or 2K — fills flash slowly, enough detail
set blackbox_sample_rate = 1/2
set blackbox_device = SDCARD
save
```

> Rosser: "If you set it to 4K or 8K you don't get much extra useful info but you fill up the log really fast."

---

## Step 1: Open the Frequency vs Throttle View

1. Open Blackbox Explorer
2. Load your `.bbl` file
3. Click **Overlay** → enable **Analyzer**
4. Select: **Gyro Scaled** (pre-filter raw gyro)
5. Click **Frequency** → switch to **Frequency vs Throttle**
6. Adjust **Luminosity** slider so you can see all features without blowout

You'll see a red/black 2D plot: X = frequency (0–1000 Hz), Y = throttle (0–100%)

---

## Step 2: Identify Motor Noise

Motor noise shows as **diagonal lines** — starts low at low throttle, climbs with throttle.

For **tri-blade props** (your setup):
- **Strong noise at 1× RPM** (fundamental)
- **Almost nothing at 2× RPM** ← why we set weight to 0
- **Some noise at 3× RPM**

> Rosser: "Three blades, three times the RPM — every time a blade passes over an arm it creates vibration."

**Action:** Confirm RPM filter weights `100,0,80` in Block A are suppressing this correctly.

---

## Step 3: Identify Frame Resonances

Frame resonances show as **vertical stripes** — fixed frequency, visible across all throttle levels.

| What you see | Action |
|-------------|--------|
| No vertical stripes | **Disable dynamic notch** (`dyn_notch_count = 0`) — it's adding delay for nothing |
| 1 vertical stripe | Set `dyn_notch_count = 1`, tune min/max Hz around the stripe |
| 2-3 stripes | Set count = number of stripes |

> Rosser: "If your log is clean and there are no bright vertical stripes — disable the dynamic notch. It's sitting there doing nothing and causing delay."

**Setting the dynamic notch range:**
- `dyn_notch_min_hz` = stripe frequency minus ~25 Hz (never below 150 Hz)
- `dyn_notch_max_hz` = just above the stripe (or leave at 600 Hz default)
- `dyn_notch_q` = push toward 1000, back off if noise escapes either side

---

## Step 4: Check RPM Filter Effectiveness

Compare **pre-filter gyro** vs **post-filter gyro** on the same axis.

- Motor noise diagonal lines should be **suppressed** in post-filter
- If noise still visible: lower `gyro_rpm_notch_q` slightly
- If noise fully gone and log looks clean: try increasing Q to 600–1000 for less delay

---

## Step 5: Check D-term Noise

Switch to **D-term (unfiltered)** and **D-term (filtered)** traces.

> Rosser: "The D-term amplifies high frequency noise — twice the frequency = twice the gain. You can't leave high frequency noise unfiltered."

- If D-term unfiltered looks very noisy above 150 Hz: lower `dterm_lpf1_dyn_max_hz`
- If D-term filtered looks clean and you have headroom: try raising max cutoff for less latency

---

## Step 6: Gyro LPF2 Check

With BLITZ F722 (ICM42688P):
- Check Betaflight Motors tab → what does it show for gyro/PID loop rate?
- If **8K/8K** → disable LPF2 (`set gyro_lpf2_static_hz = 0`) — no aliasing possible
- If **8K/4K** → keep LPF2 at 1000 Hz (anti-aliasing needed)
- When in doubt: 1000 Hz is safe and adds very little delay

---

## Step 7: Fill in Block B

Based on your analysis, update `tune/block_B_filters.txt`:

| Setting | Default start | Adjust if... |
|---------|--------------|-------------|
| `rpm_filter_weights` | 100,0,80 | 2nd harmonic noise visible → increase second value |
| `gyro_rpm_notch_q` | 500 | Motor noise getting through → lower; clean log → raise toward 1000 |
| `dyn_notch_count` | 1 | No stripes → set 0; more stripes → match count |
| `dyn_notch_min_hz` | 150 | Set 25 Hz below your frame resonance stripe |
| `dterm_lpf1_dyn_max_hz` | 150 | Motors warm/rough → lower; motors cool, clean → raise |

---

## Quick Reference — What to Look For

| Symptom in log | Cause | Fix |
|---------------|-------|-----|
| Diagonal stripes not suppressed | RPM filter Q too high or weights wrong | Lower Q or fix weights |
| Vertical stripe at fixed frequency | Frame resonance | Enable/tune dynamic notch |
| Broad noise floor 100–300 Hz | LPF2 too high, or frame resonance | Check frame screws, tune filters |
| Clean log above 300 Hz | ✅ Excellent — push Q higher for less delay | |
| Motors hot after 2 min | D-term filter too high OR D gain too high | Lower dterm max cutoff first |
