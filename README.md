<div align="center">

# 💧 Smart Water Tank
### IoT-Based Ultrasonic Water Level Monitoring & Automatic Alert System

*Built with ESP32 · HC-SR04 · Blynk IoT*

![Platform](https://img.shields.io/badge/Platform-ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![IoT](https://img.shields.io/badge/IoT-Blynk-1BA0D7?style=for-the-badge&logo=blynk&logoColor=white)
![Language](https://img.shields.io/badge/Language-C++-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![Status](https://img.shields.io/badge/Status-Working%20Prototype-2ECC71?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

<br>

**Non-contact water level sensing → live cloud dashboard → local RGB + buzzer alerts**
<br>*No manual checking. No submerged probes. Just glance at your phone.*

</div>

---

## 📖 Overview

The **Smart Water Tank** continuously measures water level with a non-contact ultrasonic sensor, classifies it into **LOW 🔴 / MEDIUM 🔵 / FULL 🟢**, and pushes it live to a **Blynk IoT dashboard** — while a local RGB LED and buzzer give instant on-site feedback. Built around the ESP32, it's compact, low-cost, and needs zero display hardware.

<div align="center">

```
┌──────────────┐      40 kHz pulse      ┌──────────────┐
│   HC-SR04    │ ─────────────────────▶ │ Water Surface │
│  Ultrasonic  │ ◀───────────────────── │              │
└──────┬───────┘        echo            └──────────────┘
       │ distance (cm)
       ▼
┌──────────────────────────────────────────────┐
│                    ESP32                       │
│   Distance ──▶ Water Level %  ──▶  Classify     │
│                                    (LOW/MED/FULL)│
└───┬─────────────────┬─────────────────┬────────┘
    │                 │                 │
    ▼                 ▼                 ▼
🔴🔵🟢 RGB LED     🔊 Buzzer        ☁️ Blynk Cloud
(local status)   (audible alert)   (V0 · V1 · V2)
                                        │
                                        ▼
                              📱 Live Dashboard
                         Gauge · Label · LED widget
```

</div>

---

## ✨ Features

| | |
|---|---|
| 📡 | **Non-contact ultrasonic sensing** — no corrosion, no electrical risk |
| ☁️ | **Live Blynk dashboard** — monitor from anywhere over Wi-Fi |
| 🚦 | **3-state RGB indication** — Red (Low) · Blue (Medium) · Green (Full) |
| 🔊 | **Audible double-beep alert** — on both critical-low and full states |
| ⚙️ | **Derived motor-status flag** — shows when refilling is implied |
| 🧩 | **Minimal wiring** — no display hardware required |

---

## 🧰 Hardware Used

| Component | Qty | Notes |
|---|:---:|---|
| ESP32 Dev Board (WROOM-32) | 1 | Main controller + Wi-Fi |
| HC-SR04 Ultrasonic Sensor | 1 | Mounted above tank, facing water surface |
| RGB LED (Common Cathode) | 1 | Local status indicator |
| Active Buzzer (3.3V/5V) | 1 | Audible alerts |
| 220Ω Resistors | 3 | RGB LED channels |
| Breadboard + Jumper Wires | — | Prototyping |
| USB-C Cable | 1 | Programming / power |
| 2.4 GHz Wi-Fi Network | — | ESP32 connectivity (5 GHz unsupported) |
| Blynk Account | — | Free tier is enough |

---

## 🔌 Circuit Connections

| Component Pin | ESP32 Pin |
|---|---|
| HC-SR04 VCC | 5V (VIN) |
| HC-SR04 GND | GND |
| HC-SR04 TRIG | GPIO 5 |
| HC-SR04 ECHO | GPIO 18 *(via voltage divider — ECHO is 5V, GPIO is 3.3V-tolerant only)* |
| RGB LED — Red | GPIO 25 (via 220Ω) |
| RGB LED — Green | GPIO 26 (via 220Ω) |
| RGB LED — Blue | GPIO 27 (via 220Ω) |
| RGB LED — Common Cathode | GND |
| Buzzer (+) | GPIO 14 |
| Buzzer (−) | GND |

> ⚠️ **Safety note:** The HC-SR04 ECHO pin outputs 5V. Use a resistor voltage divider (e.g. 1kΩ / 2kΩ) on the ECHO line to protect the ESP32's 3.3V-tolerant GPIO.

---

## 🧮 How It Works

**1. Distance measurement**
```
Distance (cm) = (echo_duration × 0.0343) / 2
```

**2. Convert to water level %**
```
Water Level (%) = ((Tank Height − Measured Distance) / Tank Height) × 100
```

**3. Classify & react**

| Water Level | Status | RGB LED | Buzzer | Motor Status (V2) |
|:---:|:---:|:---:|:---:|:---:|
| ≤ 30% *(critical/low)* | 🔴 **LOW** | Red — solid | 2 beeps | ON, steady |
| 31 – 70% | 🔵 **MEDIUM** | Blue — solid | OFF | ON, steady |
| 71 – 94% | 🟢 **FULL** | Green — solid | 2 beeps | OFF |

Level, status label, and motor-status flag are written every second to Blynk virtual pins **V0**, **V1**, and **V2**, driving the live dashboard.

---

## 📊 Blynk Dashboard

| Widget | Virtual Pin | Purpose |
|---|:---:|---|
| **Gauge** | V0 | Live water level (0–100%) |
| **Label** | V1 | Tank status text — LOW / MEDIUM / FULL |
| **LED** | V2 | Motor status — lit while refilling is needed |

<div align="center">

| 🔴 LOW — 19% | 🔵 MEDIUM — 61% | 🟢 FULL — 91% |
|:---:|:---:|:---:|
| Critical alert, 2 beeps | Steady monitoring | Full alert, 2 beeps |

</div>

---

## 🛠️ Getting Started

```bash
# 1. Install libraries (Arduino IDE Library Manager)
Blynk        → by Volodymyr Shymanskyy
esp32        → by Espressif Systems (Boards Manager)

# 2. Configure Blynk
- Create a Template: "Automated Water Tank" (ESP32 / Wi-Fi)
- Add datastreams:
    V0 → Integer (0-100) → "Water Level" → Gauge widget
    V1 → String           → "Tank Status" → Label widget
    V2 → Integer (0/1)    → "Motor Status" → LED widget
- Copy Template ID, Template Name & Auth Token into the sketch

# 3. Flash & power on
- Wire the circuit per the table above
- Upload the sketch to the ESP32
- Open the Blynk dashboard and watch it go live 🚀
```

---

## ✅ Advantages

- 🚫 Non-contact sensing — avoids corrosion & electrical risk from submerged probes
- 🌍 Remote monitoring from anywhere with Wi-Fi
- 👀 Intuitive 3-colour status, locally and on the dashboard
- 🔔 Audible confirmation for critical states
- 💸 Compact, low-cost — no display hardware needed

## ⚠️ Limitations

- Motor status is currently **derived**, not driving a physical pump yet
- Requires a stable **2.4 GHz** Wi-Fi network (5 GHz unsupported)
- Ultrasonic accuracy can be affected by turbulence, foam, or vapour
- Wi-Fi credentials & auth token are hardcoded — secure before deployment

## 🌱 Applications

- 🏠 Domestic overhead / underground tank monitoring
- 🌾 Remote farm or agricultural reservoir monitoring
- 🏭 Industrial storage tank tracking with cloud visibility
- 🎓 Educational demo of ESP32 + Blynk IoT integration

---

## 🚀 Future Scope

- [ ] Add a relay + pump driver for true auto-fill
- [ ] Move credentials to a secure config / secrets file
- [ ] Add historical level logging & graphing on Blynk
- [ ] Push notifications on critical-low / full states

---

<div align="center">

## 👤 Author

**P.M Shoaib Khan**
B.Tech, Electronics and Communication Engineering

[![GitHub](https://img.shields.io/badge/GitHub-shoaib725-181717?style=flat-square&logo=github)](https://github.com/shoaib725)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-shoaibkhan725-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/shoaibkhan725)

⭐ *If this project helped you, consider giving it a star!*

</div>
