# PID Analysis Guide
# Nazgul DC5 V2 — Betaflight Blackbox

Use this after Block B filters are confirmed to determine PID adjustments for Block C.

---

## Step 1: Open Log in Blackbox Explorer

Select the flight log from a post-Block-B flight.  
Use the **Graph view** (not spectrum analyser).

Show these traces:
- `gyroADC[0]` (Roll gyro — actual rotation)
- `axisP[0]` (P term contribution — roll)
- `axisD[0]` (D term contribution — roll)
- `setpoint[0]` (your stick input)

---

## Step 2: Evaluate P Gain

**Find a fast roll section** (the snap rolls from the flight protocol).

**Good P:** Gyro trace follows setpoint closely, no ringing after the move ends.

| Problem | Symptom | Fix |
|---------|---------|-----|
| P too high | Ringing / oscillation at end of flip | Reduce P by 10% |
| P too low | Gyro lags behind setpoint | Increase P by 10% |

---

## Step 3: Evaluate D Gain

**Find a throttle punch section** (throttle punches from flight protocol).

**Good D:** Damps P oscillation without adding noise. Motor audio should sound clean (no high-pitched whine = D too high).

| Problem | Symptom | Fix |
|---------|---------|-----|
| D too high | High-frequency oscillation on gyro trace, motors hot | Reduce D by 15% |
| D too low | Propwash on dive exit, P ringing not damped | Increase D by 10% |

---

## Step 4: Evaluate Feedforward

**Find fast directional changes** (the consecutive rolls section).

**Good F:** Gyro trace leads setpoint slightly on fast inputs (predictive). No spikes at the ends of moves.

| Problem | Symptom | Fix |
|---------|---------|-----|
| F too high | Sharp spike/overshoot at start of move | Reduce F by 15% |
| F too low | Feels sluggish on fast inputs | Increase F by 15% |

---

## Step 5: Evaluate I Gain

**Find a steady hover or forward flight section.**

**Good I:** Quad holds attitude in wind without bobbing or weaving.

| Problem | Symptom | Fix |
|---------|---------|-----|
| I too high | Low-frequency rolling oscillation in hover | Reduce I by 10% |
| I too low | Drifts with wind, can't hold roll/pitch attitude | Increase I by 10% |

---

## Step 6: Iterate

- Change **one gain at a time**, 10–15% increments
- Fly, analyze, repeat
- Lock in Roll axis first, then Pitch (usually similar), then Yaw
- Yaw D is almost always left at 0

---

## Reference Starting Points (Profile 0 — Freestyle)

```
P Roll: 47   P Pitch: 51   P Yaw: 45
I Roll: 84   I Pitch: 90   I Yaw: 90
D Roll: 46   D Pitch: 48
F Roll: 120  F Pitch: 125  F Yaw: 120
```

See `tune/profile_freestyle.txt` for full baseline.
