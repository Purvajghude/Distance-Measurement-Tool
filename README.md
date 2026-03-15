# Distance-Measurement-Tool

<div align="center">

```
· · · · · · · · ·
d i s t a n c e .
· · · · · · · · ·
```

# distance.

**A dual-sensor distance measurement system with a live web dashboard**

*HC-SR04 Ultrasonic · Arduino Uno · Python Flask · Nothing OS aesthetic*

---

![Arduino](https://img.shields.io/badge/Arduino-Uno-00979D?style=flat-square&logo=arduino&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-2.x-000000?style=flat-square&logo=flask&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-CC0000?style=flat-square)

</div>

---

## What is this?

**distance.** is a physics project that measures real-time distance using an HC-SR04 ultrasonic sensor connected to an Arduino Uno. Readings stream live to a web dashboard built in the aesthetic of [Nothing OS](https://nothing.tech) — warm off-white cards, Space Mono typography, and red accent dots.

The project started as a simple Arduino Serial Monitor experiment and grew into a full-stack system with:

- Live distance readings at 5Hz
- Smoothed average, velocity, and zone classification
- A beautiful responsive web dashboard
- CSV data export
- A circuit wiring reference page
- A Theremin-style generative art canvas driven by your hand movement

> **Physics context:** The HC-SR04 uses the echo principle — it fires 40kHz ultrasonic pulses and measures the time for the echo to return. Distance = (speed of sound × time) ÷ 2. At room temperature (~25°C), sound travels at approximately 346 m/s, or 0.0343 cm/μs.

---

## Dashboard Preview

```
┌─────────────────────────────────────────────────────────────────┐
│  · · · · · · ·                                    ● ● ● ○ ○ ○ ○ │
│  distance.                              ● LIVE      21:14:32     │
│  HC-SR04 · Arduino Uno                                           │
├──────────────────────────────────────────────────────────────────┤
│ ▶ Start  ■ Stop  ↓ Export  ✕ Clear                               │
├────────────────────┬────────────────────────┬────────────────────┤
│                    │  Distance · Time Series │  Range Gauge       │
│   74.3             │                         │                    │
│         cm  743mm  │  ▂▃▄▃▂▃▄▅▄▃▂▃▄▃▂       │    ◐  74.3         │
│                    │                         │                    │
│  ┌─ MID RANGE ──┐  ├────────────────────────┤  ├────────────────┤ │
│  │ ● approaching│  │ MIN    MAX   STATUS  %  │  │  Event Log     │ │
│  └──────────────┘  │ 72.1  80.4  ONLINE  19% │  │ 21:14:31 ...  │ │
└────────────────────┴────────────────────────┴────────────────────┘
```

---

## Hardware Required

| Component | Purpose | Approx. Cost |
|-----------|---------|-------------|
| Arduino Uno (R3 or clone) | Microcontroller | ₹400–600 |
| HC-SR04 Ultrasonic Sensor | Distance measurement | ₹50–80 |
| GY-53 (VL53L0X module) | Optional — decorative in circuit | ₹300–500 |
| Breadboard (full size) | Prototyping | ₹80–120 |
| Jumper wires (M-M) | Connections | ₹50 |
| USB-A to USB-B cable | Arduino ↔ PC | Included with Uno |

**Minimum setup:** Arduino Uno + HC-SR04 + breadboard + jumper wires. Everything else is optional.

---

## Wiring

### HC-SR04 → Arduino Uno

| HC-SR04 | Arduino | Wire colour |
|---------|---------|------------|
| VCC | 5V | 🔴 Red |
| TRIG | D9 | 🟠 Orange |
| ECHO | D10 | 🔵 Blue |
| GND | GND | ⚫ Black |

### GY-53 → Arduino Uno (if included — use RIGHT side pins only)

| GY-53 Pin | Arduino | Wire colour | Note |
|-----------|---------|------------|------|
| VCC | 5V | 🔴 Red | |
| GND | GND | ⚫ Black | |
| TX | D3 | 🟢 Green | |
| RX | — | — | Leave unconnected |
| PWM | GND | ⚫ Black | Required |
| PS | GND | ⚫ Black | **Critical — forces UART mode** |

> ⚠️ **Important:** The GY-53 module has an onboard MCU. It does **not** work with standard VL53L0X I2C libraries. PS must be connected to GND to enable UART output. The left side pins (SCL, SDA, GPIO1, XSHUT) should be left completely unconnected.

Visit `localhost:5000/wiring` after setup for the interactive circuit diagram.

---

## Software Setup

### 1. Install Arduino IDE

1. Download from [arduino.cc/en/software](https://www.arduino.cc/en/software)
2. Install for your operating system
3. Connect your Arduino Uno via USB
4. Open **Tools → Board** and select **Arduino Uno**
5. Open **Tools → Port** and select your COM port (e.g. `COM9` on Windows, `/dev/ttyUSB0` on Linux)

### 2. Upload the Arduino sketch

1. Open `arduino/sketch.ino` in Arduino IDE
2. Click **Upload** (→ button)
3. Open **Tools → Serial Monitor**, set baud rate to **9600**
4. You should see distance readings printing every 200ms

```
==============================
   HC-SR04 Distance Sensor
==============================
  Distance: 74.3 cm
  Distance: 74.1 cm
  Distance: 73.9 cm
```

### 3. Set up Python environment

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/distance.git
cd distance

# Install dependencies
pip install pyserial flask
```

### 4. Configure your COM port

Open `app.py` and update line 8:

```python
PORT = "COM9"   # Windows example
# PORT = "/dev/ttyUSB0"   # Linux
# PORT = "/dev/cu.usbmodem14201"   # macOS
```

To find your port: Arduino IDE → Tools → Port — use whatever is listed there.

### 5. Run the dashboard

```bash
# Close Arduino Serial Monitor first (it holds the COM port)
python app.py
```

Open your browser at **[http://localhost:5000](http://localhost:5000)**

---

## Project Structure

```
distance/
│
├── app.py                  # Flask backend — serial reader + API routes
├── dashboard.html          # Main live dashboard (Nothing OS aesthetic)
├── wiring.html             # Circuit diagram reference page
├── canvas.html             # Theremin generative art canvas
│
├── arduino/
│   └── sketch.ino          # Arduino sketch — reads HC-SR04, prints to Serial
│
├── distance_log.csv        # Auto-generated data log (created on first run)
└── README.md
```

---

## How It Works

### Arduino side

The sketch runs a loop every 200ms:

1. Sends a 10μs HIGH pulse on `TRIG` (D9)
2. HC-SR04 fires 8 ultrasonic bursts at 40kHz
3. `ECHO` pin (D10) goes HIGH for the duration of the return
4. `pulseIn()` measures that duration in microseconds
5. Distance = `(duration × 0.0343) / 2` cm
6. Prints the result over Serial at 9600 baud

### Python side

`app.py` runs two things simultaneously:

- A **background thread** that reads the serial port continuously, parses distance values, computes smoothed average + velocity, classifies zone and movement, and logs everything to CSV
- A **Flask web server** that serves the HTML pages and exposes a `/data` JSON endpoint

### Browser side

The dashboard polls `/data` every 200ms and updates:

- Live distance display + mm equivalent
- Smoothed 5-sample rolling average
- Velocity (cm/s) with direction indicator
- Zone classification (VERY CLOSE / NEAR / MID RANGE / FAR)
- Session min/max
- Live Chart.js time series graph
- Semicircle range gauge with needle
- Scrollable event log

---

## Pages

| URL | Description |
|-----|-------------|
| `localhost:5000/` | Live dashboard |
| `localhost:5000/wiring` | Circuit diagram + pin tables |
| `localhost:5000/canvas` | Theremin-style generative art canvas |
| `localhost:5000/data` | Raw JSON data endpoint |
| `localhost:5000/download` | Download CSV log |

---

## Physics Background

### Ultrasonic sensing

The HC-SR04 uses the same principle as a bat or submarine sonar. A burst of 40kHz sound (inaudible to humans) is emitted, reflects off the nearest surface, and the sensor measures the round-trip time.

```
Distance (cm) = (Time in μs × 0.0343) / 2
```

The speed of sound varies with temperature:

```
v = 331.4 + (0.6 × T)  m/s
```

where T is temperature in °C. At 25°C: v ≈ 346 m/s = 0.0346 cm/μs.

### Sensor characteristics

| Property | HC-SR04 |
|----------|---------|
| Range | 2 cm – 400 cm |
| Accuracy | ±3 mm at close range |
| Beam angle | ~15° cone |
| Frequency | 40 kHz |
| Works on | Hard flat surfaces |
| Struggles with | Soft/angled/transparent surfaces |

### Known limitations

- Soft materials (fabric, foam) absorb sound — echo weakens or disappears
- Angled surfaces deflect the beam away from the sensor
- Temperature changes shift the speed of sound slightly
- Minimum range of 2cm — below that the echo overlaps with the pulse

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---------|-------------|-----|
| Upload error: `COM port access denied` | Serial Monitor is open | Close Serial Monitor, then upload |
| `Out of range` on both sensors | Wiring loose | Recheck all connections |
| No output in browser | Wrong COM port in `app.py` | Match the port shown in Arduino IDE |
| Dashboard shows stale data | Flask not running | Run `python app.py` first |
| GY-53 sends no data | PS pin not grounded | Connect PS → GND |
| Garbage characters in Serial Monitor | Wrong baud rate | Set to 9600 in Serial Monitor |

---

## Built With

- **Arduino Uno** + C++ (Arduino framework)
- **Python 3** + `pyserial` + `Flask`
- **HTML5 Canvas API** + **Chart.js**
- **Space Mono** (Google Fonts)
- Design inspired by [Nothing OS](https://nothing.tech)

---

## License

MIT — do whatever you want with it.

---

<div align="center">

```
· · · · · · · · ·
```

*Built as a first-year engineering physics project*
*Navi Mumbai, 2026*

</div>
