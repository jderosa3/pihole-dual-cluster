---

# 📘 **Pi-hole Fail-Over Cluster — OLED Dashboard System**

A complete Raspberry Pi OLED dashboard designed for a **dual-node Pi-hole failover cluster**.
Includes advanced animations, statistics visualization, hardware indicators, and safe reboot/shutdown control.

**Current Version: v1.6 — November 2025**
Author: **James DeRosa**

---

# 🧭 **Project Overview**

This repository contains the complete code and assets for an **SSD1306 128×32 OLED dashboard** used on each node of a dual-Pi Pi-hole failover setup.

Each Raspberry Pi provides:

* Real-time Pi-hole v6 statistics
* Real-time CPU, RAM, DISK, and temperature displays
* Title screen with noise-disintegration fade
* Screen rotation with button control
* Screensaver with animated vortex
* Safe reboot & shutdown with dual-stage hold
* GPIO LED indicators for button and network activity
* SSH session activity indicator
* Clean Pi-hole v6 API authentication

This is the **full production-ready OLED system** used in the failover cluster.

---

# 🖥️ **OLED Dashboard: Feature Summary**

## **📊 Dashboard Screens**

The dashboard rotates through the following screens:

### **1. Status Screen**

* Uptime
* Current date/time
* Pi-hole versions (Core / Web / FTL) with pixel bounce scrolling
* Hobbyist-style CRT terminal SSH icon (blinks when active)

### **2. System Stats**

* CPU usage bar
* RAM usage bar
* Disk usage bar
* CPU temperature thermometer
* Wave tick marks
* Pixel-aligned labels and spacing
* Timestamp

### **3. Network Info**

* eth0 IP with network jack icon
* wlan0 IP with Wi-Fi icon
* Hostname with device icon
* All icons vertically aligned with pixel precision

### **4. Blocklist Domains**

* Total/active domain counts
* 32×32 blocklist icon

### **5. Pi-hole Stats (v6 API)**

* DNS Queries
* Ads Blocked
* Percent Blocked
* Gravity Domain Count
* 32×32 funnel icon

---

# 🏁 **Title Screen + Fade-Out**

The startup title screen:

* Loads ASCII logo from
  `/opt/oled/pi-hole-logo.txt`
* Renders pixel-accurate graphic
* Shows a centered loading bar with rounded ends
* Performs a **fast random single-pixel dither fade-out**

  * No I²C race conditions
  * No OLED crashes
  * Smooth digital dissolve effect

---

# 🌙 **Screensaver — Starfield Vortex**

After **5 seconds of inactivity**, a vortex animation plays:

* ~80 stars
* Accelerate into center
* Spiral motion
* Trail lines
* Optional ASCII overlay mask
* Exits instantly on button press
* Returns to **System Statistics** (skips title screen)

---

# 🔄 **Dual-Stage Reboot / Shutdown System (v1.6)**

Powered by GPIO button on pin BCM16.

### **Short Press**

Cycles to next screen.

### **3-Second Hold → Reboot Mode**

Enters the circular reboot sweep:

```
Rebooting...
(hold for
 power down)
```

Circle fills over 5 seconds.

### **Release Early**

Shows inverted CRT reboot countdown:

```
Rebooting... (5)
Rebooting... (4)
...
```

Automatically reboots afterward.

---

### **5-Second Hold → Shutdown Mode**

If the button is still held after the first circle:

* Enters **Shutdown Circle**
* Shows text:
  `Powering Off...`

### **Release During Shutdown Circle**

Triggers final shutdown sequence.

### **Final CRT Shutdown Screen**

White background with black bold text:

```
It's safe to
turn off Pi
```

* Classic Macintosh-style rounded corners
* LED1 pulses slowly **3 times**
* System runs `sync` then `halt`
* OLED remains lit in halted safe state

---

# 🔌 **GPIO LED Behavior**

This version uses **five** LEDs:

| LED  | Purpose          | Interface | Direction     | GPIO (BCM) | Pin |
| ---- | ---------------- | --------- | ------------- | ---------- | --- |
| LED1 | Button indicator | —         | ON while held | 26         | 37  |
| LED2 | Network RX       | eth0      | Receive       | 5          | 29  |
| LED3 | Network TX       | eth0      | Transmit      | 7          | 26  |
| LED4 | Network RX       | wlan0     | Receive       | 10         | 19  |
| LED5 | Network TX       | wlan0     | Transmit      | 27         | 13  |

### Behavior:

* LED1 → mirrors button state
* LEDs 2–5 → pulse on real RX/TX activity
* LED threads read `/sys/class/net/.../rx_bytes` and `tx_bytes`
* Extremely low CPU usage
* Safe, no polling on disks
* Automatically disables on missing interfaces

---

# 🖥️ **SSH Activity Indicator**

A custom ASCII-art mini terminal icon appears in the **top-right corner** when an SSH session is active.

Features:

* 33×9 pixel CRT terminal icon
* Blinks at 0.5-second intervals
* Detection based on `who` (`pts/`)
* Works instantly for SSH and SFTP
* Zero false positives
* Displays on **Status Screen**

---

# 🛠 **Pi-hole v6 API Integration**

The system uses a complete Pi-hole v6 login workflow:

* `POST /api/auth` session login
* SID cookie management
* CSRF header management
* Auto-session refresh
* Accesses:
  `/api/stats/summary`
* Fully removed Pi-hole v5 legacy code
* Correctly parses gravity count from v6 schema

No deprecated endpoints remain.

---

# 🧱 **Hardware Requirements**

* Raspberry Pi 4B (two units recommended for cluster)
* SSD1306 OLED 128×32 I²C
* Momentary push button
* Five LEDs
* Jumper wires
* 3.3V/GND rails

### **I²C Wiring**

* SDA → Pin 3
* SCL → Pin 5
* 3.3V → Pin 1
* GND → Pin 6

### **GPIO Assignments**

| Component     | GPIO | Pin |
| ------------- | ---- | --- |
| Button        | 16   | 36  |
| LED1 Button   | 26   | 37  |
| LED2 eth0 RX  | 5    | 29  |
| LED3 eth0 TX  | 7    | 26  |
| LED4 wlan0 RX | 10   | 19  |
| LED5 wlan0 TX | 27   | 13  |

---

# 📦 **Software Requirements**

### System:

* Raspberry Pi OS 64-bit
* Pi-hole v6
* Python 3.11+

### Python Libraries:

```
adafruit-circuitpython-ssd1306
psutil
requests
pillow
netifaces
```

---

# 🚀 Installation Guide

## Enable I²C

```
sudo raspi-config
Interface Options → I2C → Enable
```

## Install Pi-hole v6

```
curl -sSL https://install.pi-hole.net | bash
```

## Create Virtual Environment

```
python3 -m venv ~/oled-venv
source ~/oled-venv/bin/activate
pip install adafruit-circuitpython-ssd1306 psutil netifaces requests pillow
```

## Files & Paths

```
/opt/oled/oled_full.py
/opt/oled/pi-hole-logo.txt
/etc/systemd/system/oled-dashboard.service
/home/piadmin/oled-venv/
```

---

# 🔧 **Systemd Auto-Start**

Create service:

```
/etc/systemd/system/oled-dashboard.service
```

Add:

```ini
[Unit]
Description=OLED Dashboard for Pi-hole Fail-Over Node
After=network.target

[Service]
ExecStart=/home/piadmin/oled-venv/bin/python /opt/oled/oled_full.py
User=piadmin
WorkingDirectory=/opt/oled
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable:

```
sudo systemctl daemon-reload
sudo systemctl enable oled-dashboard
sudo systemctl start oled-dashboard
```

---

# ▶️ **Runtime Logic Summary**

* Boot → Title Screen → Status Screen → System Stats
* Button tap → next screen
* 5s idle → Screensaver
* Screensaver exit → System screen
* 3s hold → Reboot Circle
* 5s hold → Shutdown Circle
* Release during Stage 2 → CRT shutdown sequence
* LED1 pulses 3× then halt
* OLED stays lit indefinitely (safe to remove power)

---

# 🧩 **Future Enhancements (Not Yet Implemented)**

* pi-hole sync daemon (primary ↔ failover)
* DNS waterfall automation
* Cluster heartbeat service
* Full SD-card imaging workflow
* Push notifications on failover events

---

# 👤 **Credits**

Developed by **James DeRosa**
All OLED graphics, animations, and UI built by hand.

---
