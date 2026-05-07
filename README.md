<div align="center">

# ⚡ Intelligent Power Source Identification System Demo

<p align="center">
  <img src="https://img.shields.io/badge/Status-Delivered-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Client-Private%20%7C%20Confidential-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Type-Full--Stack%20IoT-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Built-From%20Scratch-red?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Flask-000000?style=flat-square&logo=flask&logoColor=white" />
  <img src="https://img.shields.io/badge/SQLite-07405E?style=flat-square&logo=sqlite&logoColor=white" />
  <img src="https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat-square&logo=raspberry-pi&logoColor=white" />
  <img src="https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white" />
</p>

> A hardware + software system that **automatically identifies in real time** whether electrical power is being supplied from the **grid, solar, or generator** — built entirely from scratch: hardware selection, circuit design, firmware, and live web dashboard.

</div>

---

## 📌 The Problem

In areas with multiple power sources (grid, solar, generator), knowing which source is active at any moment is critical for:

- 🔍 **Energy auditing** — track which source is used and for how long
- ⚖️ **Load management** — make smart switching decisions
- 💰 **Billing** — attribute consumption accurately per source
- 🚨 **Fault detection** — catch unexpected source changes instantly

---

## 💡 The Solution

Current sensors placed on each power line feed analog readings into an **ADS1115 ADC over I2C**. Python firmware takes **500-sample RMS readings** per channel, applies **margin-based classification logic** to determine the dominant active source, logs every change to SQLite, and serves the result to a **live Flask dashboard** that auto-refreshes every 2 seconds.

---

## 🚀 Key Features

| Feature | Details |
|---------|---------|
| 🔌 Multi-Source Detection | Simultaneously monitors Grid, Solar, and Generator lines |
| 📐 RMS Current Measurement | 500-sample RMS with DC offset removal — accurate for AC waveforms |
| 🎯 Margin-Based Classification | Dominant source must lead by 10% margin — prevents flicker between close readings |
| 🔬 ADS1115 ADC | 16-bit precision analog-to-digital conversion over I2C |
| 📝 Change-Detection Logging | New DB row only written when source changes or current/power shifts >5% — no log spam |
| 🌐 Flask REST API | Three endpoints for live status, 24h history, and per-source statistics |
| 📊 Live Dashboard | Animated power-flow visualisation, real-time metrics, scrollable event history |
| 🔁 Auto-Start on Boot | `start.sh` launches the system automatically when the Pi powers on |
| 🛠️ Built From Scratch | Full-stack ownership: hardware selection → wiring → firmware → dashboard |

---

## 🏗️ System Architecture

```
Power Lines (Grid / Solar / Generator)
        │
        ▼
Current Sensors (ACS712 / SCT-013) — one per line
        │
        ▼
ADS1115 16-bit ADC  ──  I2C  ──  Raspberry Pi 4B
        │
        ▼
Python: RMS Calculation + Margin-Based Classification
        │
        ▼
SQLite Database (change-detection logging)
        │
        ▼
Flask REST API
(/api/current-status · /api/power-history · /api/source-stats)
        │
        ▼
Live Web Dashboard (auto-refreshes every 2 seconds)
```

---

## 🧰 Hardware Components

| Component | Purpose |
|-----------|---------|
| Raspberry Pi 4B | Main controller |
| ACS712 | AC/DC current sensor |
| SCT-013 | Non-invasive AC clamp sensor |
| ADS1115 | 16-bit ADC over I2C |

---

## 🧠 Classification Logic

A source must exceed `ON_THRESHOLD (0.3A)` **and** lead the next-highest reading by `MARGIN (10%)` to be declared active. Only one source is ever reported as active at a time.

```python
active_source = None
max_current   = 0.0

for name, current in readings.items():
    if current > ON_THRESHOLD and current > max_current * (1 + MARGIN):
        active_source = name
        max_current   = current
```

The RMS reading removes DC offset across 500 samples before classification:

```python
avg_voltage = sum(voltages) / len(voltages)
squared_sum = sum(((v - avg_voltage) / CT_SENSITIVITY) ** 2 for v in voltages)
rms = (squared_sum / SAMPLES) ** 0.5
```

---

## 🌐 API Endpoints

| Endpoint | Method | Returns |
|----------|--------|---------|
| `/api/current-status` | GET | Latest source, status, current (A), power (W), timestamp |
| `/api/power-history` | GET | Last 100 log entries from the past 24 hours |
| `/api/source-stats` | GET | Per-source: active periods, average power, total energy (kWh) |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Hardware | Raspberry Pi 4B, ACS712, SCT-013, ADS1115 |
| Firmware | Python 3, adafruit-circuitpython-ads1x15, busio |
| Database | SQLite via SQLAlchemy ORM |
| Backend | Flask |
| Frontend | HTML · CSS (glassmorphism) · Vanilla JS (async fetch, 2s polling) |

---

## 📁 File Structure

```
power-source-identifier/
├── app.py              # Firmware + Flask backend
├── start.sh            # Auto-start script (runs on Pi boot)
├── templates/
│   └── index.html      # Live web dashboard
└── power_logs.db       # Auto-created SQLite database
```

---

## ⚙️ Setup & Deployment

**1. Install dependencies**
```bash
pip install adafruit-circuitpython-ads1x15 sqlalchemy flask
```

**2. Enable I2C on the Pi**
```bash
sudo raspi-config
# Interface Options → I2C → Enable
```

**3. Run**
```bash
python app.py
# Dashboard available at http://<your-pi-ip>:5000
```

**4. Auto-start on boot**
```bash
bash start.sh
```

---

## 🔒 Note on Source Code

> Source code is private under a **client confidentiality agreement** — company name withheld.
> This repository documents the system architecture, engineering decisions, and hardware design.
> The full codebase is available upon request for verified employers and clients.

---

## 👤 Author

<div align="center">

**Aftab Ahmad Lodhi**
*Electrical Engineer | Embedded Systems & IoT*

[![GitHub](https://img.shields.io/badge/GitHub-AftabAhmad5-181717?style=flat-square&logo=github)](https://github.com/AftabAhmad5)
[![Email](https://img.shields.io/badge/Email-aftabahmadlodhi%40gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:aftabahmadlodhi@gmail.com)

</div>
