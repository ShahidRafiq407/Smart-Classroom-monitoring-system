# Smart Classroom AI Energy Management System

> An AI-powered classroom automation system that uses computer vision to detect human occupancy in specific zones and automatically controls fans, lights, and ventilation — saving up to 50% energy compared to traditional always-on systems.

**Built as a Final Year Project (FYP) — Riphah International University, Faisalabad**

---

## Project Demo

| Hardware Prototype | Classroom Model |
|---|---|
| ![Hardware Setup](images/hardware_setup.jpeg) | ![Classroom Model](images/classroom_model.jpeg) |

> 📹 **[Watch Full Demo Video](#)** ←

---

## 🧠 How It Works

```
┌─────────────────────────┐
│   USB Wide-Angle Camera  │  ← Captures live room video
└──────────┬──────────────┘
           │  USB Video
           ▼
┌──────────────────────────────┐      ┌──────────────────────┐
│  Laptop — Python + YOLOv8    │─────▶│  Real-Time Display   │
│  (Master Device)             │      └──────────────────────┘
└──────────┬───────────────────┘
           │  USB Serial Command  e.g.  "LEFT_ON"
           ▼
┌──────────────────────────────┐  ←─(Analog)──  [MQ-135 CO2 Sensor]
│   ESP32 Microcontroller      │
│   (Slave Device)             │  ←─(Wi-Fi)──▶  [Blynk IoT Cloud]
└──────────┬───────────────────┘                 [Mobile App]
           │  5V Trigger Signals
    ┌──────┼──────────┬─────────────┐
    ▼      ▼          ▼             ▼
[LIGHTS] [LEFT Fan] [TEACHER Fan] [RIGHT Fan] [EXHAUST Fan]
```

**The system works in 3 independent layers:**

1. **AI Vision Layer** — YOLOv8 detects humans only (ignores pets/objects), assigns them to Left / Center (Teacher) / Right zones
2. **Hardware Control Layer** — ESP32 receives serial commands, activates relays with debouncing to prevent flickering
3. **Air Quality Layer** — MQ-135 sensor runs completely independently; triggers exhaust fan when CO2 is high — no laptop needed

---

## ✨ Features

- **Zone-based control** — Only the occupied zone's fan/light turns ON
- **Human-only detection** — Ignores pets, moving objects, shadows
- **Teacher zone** — Dedicated center zone with vertical boundary (table area only)
- **Debounce/Hysteresis** — 3-second stability filter prevents relay flickering
- **Independent CO2 control** — Exhaust fan works even if laptop is off
- **Real-time mobile monitoring** — Blynk IoT dashboard shows live zone status and air quality
- **~47% estimated energy savings** vs traditional always-on systems

---

## 🛒 Hardware Required

| Component | Quantity | Approx. Price (PKR) |
|---|---|---|
| ESP32 Development Board | 1 | 1,200 |
| USB Wide-Angle Camera (1080p, 120°) | 1 | 2,500 |
| MQ-135 Air Quality Sensor | 1 | 350 |
| 4-Channel Relay Module (5V) | 1 | 450 |
| DC Fans 5V (for zones) | 3 | 840 |
| Exhaust Fan 5V | 1 | 350 |
| LED Lights / Bulbs | 3 | 400 |
| Breadboard + Jumper Wires | 1 set | 450 |
| USB Type-C Cable (1.5m) | 1 | 200 |
| 5V 3A Power Adapter | 1 | 400 |
| **Total** | | **~7,140 PKR** |

---

## 💻 Software Required

| Software | Version | Download |
|---|---|---|
| Python | 3.9 or 3.10 | [python.org](https://www.python.org/downloads/) |
| Arduino IDE | 2.2.1+ | [arduino.cc](https://www.arduino.cc/en/software) |
| Blynk Mobile App | Latest | [iOS](https://apps.apple.com/app/blynk-iot/id1559317868) / [Android](https://play.google.com/store/apps/details?id=cloud.blynk) |

---

## 📁 Repository Structure

```
smart-classroom-ai/
│
├── python/
│   └── main.py                  ← Main AI detection script (run this on laptop)
│
├── esp32/
│   └── smart_classroom.ino      ← Arduino firmware (upload this to ESP32)
│
├── images/
│   ├── hardware_setup.jpg       ← Hardware photo
│   └── classroom_model.jpg      ← Classroom model photo
│
├── docs/
│   └── FYP_Report.pdf           ← Full project report (optional)
│
└── README.md                    ← This file
```

---

## ⚙️ Installation Guide — Step by Step

### STEP 1 — Clone the Repository

```bash
git clone https://github.com/ShahidRafiq407/Smart-Classroom-monitoring-system.git
cd Smart-Classroom-monitoring-system
```

> If you don't have git, click the green **Code** button on GitHub → **Download ZIP** → extract it

---

### STEP 2 — Install Python Libraries

Open **Command Prompt** (Windows) or **Terminal** (Mac/Linux) and run:

```bash
pip install ultralytics opencv-python pyserial
```

This installs:
- `ultralytics` — YOLOv8 model
- `opencv-python` — camera and video processing
- `pyserial` — USB communication with ESP32

> ⚠️ **First run will automatically download** `yolov8n.pt` (~6 MB). Make sure you have internet.

---

### STEP 3 — Set Up Blynk IoT (for mobile monitoring)

1. Go to [blynk.cloud](https://blynk.cloud) → Create a free account
2. Click **+ New Template** → Name it `Smart Classroom` → Hardware: `ESP32` → Connection: `WiFi`
3. Go to **Datastreams** → Add these virtual pins:

| Virtual Pin | Name | Type |
|---|---|---|
| V0 | Air Quality | String |
| V1 | Exhaust Fan | Integer (0/1) |
| V2 | Lights | Integer (0/1) |
| V3 | Left Fan | Integer (0/1) |
| V4 | Teacher Fan | Integer (0/1) |
| V5 | Right Fan | Integer (0/1) |

4. Go to **Web Dashboard** → Add widgets for each pin
5. Copy your **Auth Token** from Template Settings → you will need it in Step 4

---

### STEP 4 — Configure and Upload ESP32 Firmware

1. Open **Arduino IDE**
2. Install ESP32 board support:
   - Go to `File` → `Preferences`
   - Add this URL to "Additional Board Manager URLs":
     ```
     https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
     ```
   - Go to `Tools` → `Board` → `Board Manager` → search `esp32` → Install **esp32 by Espressif**

3. Install Blynk library:
   - Go to `Tools` → `Manage Libraries` → search `Blynk` → Install **Blynk by Volodymyr Shymanskyy**

4. Open `esp32/smart_classroom.ino`

5. Fill in YOUR credentials at the top of the file:

```cpp
#define BLYNK_TEMPLATE_ID   "YOUR_TEMPLATE_ID"    // from blynk.cloud
#define BLYNK_TEMPLATE_NAME "Smart Classroom"
#define BLYNK_AUTH_TOKEN    "YOUR_AUTH_TOKEN"      // from blynk.cloud

char ssid[] = "YOUR_WIFI_NAME";       // your Wi-Fi name
char pass[] = "YOUR_WIFI_PASSWORD";   // your Wi-Fi password
```

6. Connect ESP32 to laptop via USB
7. Go to `Tools` → `Board` → select **ESP32 Dev Module**
8. Go to `Tools` → `Port` → select the correct COM port (e.g., COM3, COM5)
9. Click **Upload** (→ arrow button)

> ✅ You should see "System Ready. Waiting for Python Commands..." in Serial Monitor

---

### STEP 5 — Wire the Hardware

```
ESP32 Pin    →    Connected To
─────────────────────────────────
GPIO 14      →    Relay 1 (LIGHTS)
GPIO 26      →    Relay 2 (LEFT FAN)
GPIO 25      →    Relay 3 (TEACHER FAN)
GPIO 27      →    Relay 4 (RIGHT FAN)
GPIO 32      →    Relay 5 (EXHAUST FAN)
GPIO 33      →    MQ-135 AOUT (analog output)
3.3V         →    MQ-135 VCC
GND          →    MQ-135 GND + Relay GND
5V (adapter) →    Relay VCC + All Fans/Lights positive wire
```

> ⚠️ **Relay wiring:** All relays are **Active LOW** — HIGH = OFF, LOW = ON
> ⚠️ Fans and lights should be powered from **separate 5V adapter**, NOT from ESP32 directly

---

### STEP 6 — Configure and Run Python Script

1. Open `python/main.py`

2. Find and update the configuration section at the top:

```python
COM_PORT = 'COM3'     # Change to your ESP32 COM port (check Device Manager on Windows)
CAMERA_INDEX = 1      # 0 = laptop webcam, 1 = external USB camera
TEST_MODE = False     # Set True to test camera without ESP32 connected
```

**How to find your COM port:**
- Windows: Open Device Manager → Ports (COM & LPT) → look for "Silicon Labs" or "CH340"
- Mac/Linux: Run `ls /dev/tty.*` in terminal

3. Run the script:

```bash
cd python
python main.py
```

4. A window will open showing the camera feed with zone lines drawn. You should see:
   - **White vertical lines** — dividing Left / Center (Teacher) / Right zones
   - **Red horizontal line** — teacher zone upper boundary
   - **Blue box** — Region of Interest (ROI)
   - **Green boxes** — detected humans with zone labels

5. Press **Q** to quit

---

## 🎮 How to Use

| Action | Result |
|---|---|
| Person sits in **Left** zone | Left fan turns ON after 3 seconds |
| Person sits in **Right** zone | Right fan turns ON after 3 seconds |
| Person sits at **Teacher table** (center, top area) | Teacher fan turns ON |
| Room is **empty** | All fans/lights turn OFF after 3 seconds |
| CO2 level exceeds threshold | Exhaust fan turns ON automatically |
| Open Blynk app | See live zone status and air quality |

---

## 🔧 Troubleshooting

| Problem | Solution |
|---|---|
| `Serial port COM3 connect nahi ho raha` | Check Device Manager for correct COM port |
| Camera not opening | Change `CAMERA_INDEX` from 1 to 0 (or try 2) |
| YOLOv8 not detecting people | Lower `CONFIDENCE_THRESHOLD` from 0.35 to 0.25 |
| ESP32 not connecting to WiFi | Check ssid/password, ensure 2.4GHz network |
| Blynk not updating | Check Auth Token is correct |
| Relay flickering rapidly | Increase `FAN_ON_DELAY` and `FAN_OFF_DELAY` to 5.0 |
| MQ-135 always showing high values | Sensor needs 24-48 hour burn-in after first use |

---

## 📊 Performance Results

| Test | Result |
|---|---|
| Human detection accuracy (good light) | 96.7% |
| Human detection accuracy (low light) | 86.7% |
| Pet correctly ignored | 100% |
| Average response time (detect → relay) | 165 ms |
| Estimated energy saving vs always-on | ~47% |

---

## 🔌 Serial Commands Reference

| Command | Action |
|---|---|
| `LIGHTS_ON` / `LIGHTS_OFF` | Control room lights |
| `LEFT_ON` / `LEFT_OFF` | Control left zone fan |
| `TEACHER_ON` / `TEACHER_OFF` | Control teacher zone fan |
| `RIGHT_ON` / `RIGHT_OFF` | Control right zone fan |

Exhaust fan is controlled **autonomously** by ESP32 based on MQ-135 readings.

---

## 🛠️ Customization

```python
# Change zone boundary (main.py)
TEACHER_Y_LIMIT_PCT = 50    # Increase to expand teacher zone downward

# Change detection sensitivity (main.py)
CONFIDENCE_THRESHOLD = 0.35    # Lower = more sensitive

# Change CO2 threshold (smart_classroom.ino)
#define AIR_QUALITY_THRESHOLD 300    # Lower = triggers fan sooner
```

---

## 📄 License

This project is open-source under the [MIT License](LICENSE).
Feel free to use, modify, and build upon it — credit appreciated.

---

## 🤝 Want a Custom Project Built?

We build custom **FYP projects** for CS, SE, EE, and Robotics students:

- AI + Computer Vision systems
- IoT + ESP32 automation
- Smart energy management systems
- Complete hardware prototypes with full documentation

📱 **WhatsApp:** +92 3435219710
📧 **Email:** siteosia@gmail.com
---

*If this helped you, please ⭐ star the repository!*
