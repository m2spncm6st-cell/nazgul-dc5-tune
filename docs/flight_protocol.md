# Tuning Flight Protocol
# Nazgul DC5 V2 — Betaflight 4.5.0
# Based on Chris Rosser BF 4.5 PID Tuning Masterclass

---

## Blackbox Settings (set once via CLI before any tuning flight)

```
set debug_mode = GYRO_SCALED
set blackbox_sample_rate = 1/2
set blackbox_device = SDCARD
save
```

For PD Balance analysis (Block C), change to:
```
set debug_mode = D_MIN
save
```

---

## Tools needed

- **Blackbox Explorer** — https://github.com/betaflight/blackbox-log-viewer/releases
- **PID Toolbox** — https://github.com/bw1129/PIDtoolbox/releases
  → Rosser uses PID Toolbox step response to find correct PD balance visually

---

## Tuning Stage Overview (Rosser Method)

| Stage | What you tune | Tool |
|-------|--------------|------|
| **Filter Flight** | Noise floor, RPM filter | Blackbox Explorer — Spectrum |
| **Master Multiplier** | Overall PID volume | Feel + motor temp |
| **PD Balance** | Spring (P) vs shock absorber (D) | PID Toolbox — Step Response |
| **I Term** | Slow wobble / attitude hold | Feel + blackbox |
| **Feedforward** | Stick tracking / delay | Blackbox — Gyro vs Setpoint |
| **Dynamic Damping** | D boost on sharp moves | Blackbox — D_MIN debug |

---

## Flight 1 — Filter Flight

**Blackbox debug:** `GYRO_SCALED`

| Time | Maneuver | Purpose |
|------|----------|---------|
| 0–5s | Hover at 1.5m | Baseline noise floor |
| 5–15s | Slow forward flight, gentle turns | Mid-throttle gyro noise |
| 15–25s | **5× Throttle punches** (0→100→0, senkrecht) | Motor noise harmonics |
| 25–35s | **Snap rolls L/R** (Vollausschlag, festhalten!) | P/D response check |
| 35–45s | **Snap flips vor/zurück** | Pitch P/D |
| 45–55s | **3–4 Rolls hintereinander** | Propwash |
| 55–65s | **2× Sturzflug + Abfangen** | High-throttle D |
| 65–75s | Hover, landen | End baseline |

> ⚠️ **Rosser Regel:** Stick beim Snap-Move FESTHALTEN — nie loslassen und zurückschnappen lassen. Das verfälscht alle PID-Daten.

> ✅ **Tipp von Rosser:** PID-Tuning-Flüge gerne im **Angle Mode** fliegen — einfacher zu kontrollieren, trotzdem scharfe Inputs für den PID-Controller.

---

## Flight 2 — Master Multiplier

**Dynamic Damping vorher auf 0 setzen** (in Betaflight PID-Tab → Expert Mode → Dynamic Damping Slider → 0)

1. Master Multiplier auf **0.5** starten
2. Fliegen, nach Flug Motortemperatur fühlen
3. Master um **0.1** erhöhen, wiederholen
4. Optimum = kurz bevor Motoren warm werden oder Trilling-Geräusche hörbar werden
5. Dort lassen — das ist dein Master

> **Rosser:** "Der Master Multiplier ist wie die Lautstärke. Alles andere klingt falsch wenn die Lautstärke nicht stimmt."

---

## Flight 3 — PD Balance (mit PID Toolbox auswerten)

**FF und I auf 0 setzen vorher:**
```
profile 0
set f_roll = 0
set f_pitch = 0
set f_yaw = 0
set i_roll = 0
set i_pitch = 0
set i_yaw = 0
save
```

1. P:I Slider auf **0.5** starten
2. Mehrere Logs bei 0.5 / 0.75 / 1.0 / 1.1 / 1.25 aufnehmen
3. Alle Logs in PID Toolbox laden → **Step Response** Tool
4. Ziel: rote Kurve — schnell auf 1.0, minimales Überschwingen, keine Oszillation
5. Grün = unter-gedämpft (zu viel Oszillation), Blau = über-gedämpft (zu langsam)

---

## Flight 4 — I Term

1. FF weiter auf 0 lassen
2. EY Gain (I-Slider) auf **0.5** starten
3. Fliegen: Quad fühlt sich lose/ungenau an → I zu niedrig
4. Slider erhöhen bis langsame Wobbles auftreten → I zu hoch
5. Zurück auf Wert vor den Wobbles

---

## Flight 5 — Feedforward

**ELRS 250Hz Preset anwenden** (in Betaflight → Presets → "ExpressLRS 250Hz" → HD Freestyle → Apply)

1. Stick Response Slider auf **0.5**
2. Schnelle Snap-Moves machen, Logs anschauen:
   - Gyro-Trace (hellblau) vs Setpoint (grün)
   - Gyro hängt hinter Setpoint = FF zu niedrig
   - Gyro überschießt Setpoint = FF zu hoch
3. Optimum: Gyro folgt Setpoint eng, kein Überschwingen am Ende der Bewegung

---

## Nach jedem Flug

1. Disarmen
2. **10 Sekunden warten** (Blackbox schreibt noch)
3. SD-Karte raus
4. `.bbl` Datei in `blackbox/` Ordner kopieren (mit Datum im Namen)
5. Auswerten, dann nächste Stage

---

## Pre-Flight Checklist

- [ ] Block A geflasht (Failsafe-Fix)
- [ ] Blackbox-Einstellungen gesetzt
- [ ] SD-Karte drin (FAT32)
- [ ] GPS-Fix (min. 8 Satelliten, grüne LED)
- [ ] Homepoint gesetzt (einmal armen + disarmen)
- [ ] Props fest
- [ ] Akku voll
- [ ] **Props mit denen du IMMER fliegst** — nicht neue Props nur fürs Tuning!
