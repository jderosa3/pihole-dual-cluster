# pihole-dual-cluster
Redundant dual-Pi Pi-hole cluster with automatic sync, DNS waterfall failover, and full OLED display monitoring.

✅ Pi-hole v6 API login + CSRF session handling
✅ New title screen + loading bar implementation
✅ New system stats layout (bar changes, thermometer, wave ticks)
✅ Improved IP screen with icons + alignment
✅ All 32×32 icons (funnel, block list)
✅ Fully rebuilt reboot animation
✅ Vortex screensaver with ASCII overlay
✅ Button + LED threaded logic
✅ Full screen loop behavior + idle handling
✅ Updated GPIO pin assignments (2 LEDs!)
✅ New simplified long-press logic
✅ New startup behavior (title → status → system …)

This is ready to paste directly into GitHub **as your official README**.

---

# 📘 Pi-hole Fail-Over Cluster Project

### **OLED Dashboard System — Current Version (as of November 2025)**

**Author:** James DeRosa

---

# 🧭 Project Overview

This repository contains the **complete OLED dashboard system** used in the Pi-hole Fail-Over Cluster Project.

Two Raspberry Pi 4 units (Primary + Fail-Over) each run:

* **Pi-hole v6**
* **A custom OLED dashboard**
* **A button + LED interface**
* **A systemd service for auto-start**

The OLED dashboard (`oled_full.py`) provides:

⭐ Real-time hardware and Pi-hole statistics
⭐ Custom pixel icons and animations
⭐ Elegant title screen
⭐ A powerful idle screensaver
⭐ A fully custom reboot animation
⭐ Button-driven UI with threaded LED mirroring
⭐ Pi-hole v6 API login + session management

This README documents the **project setup**, **code behavior**, and **hardware requirements** as the code exists today.

---

# 🖥️ OLED Dashboard — Feature Summary

## 📊 **Dashboard Screens**

The dashboard rotates through the following screens:

1. **Status Screen**

   * Uptime (days/hours/minutes)
   * Current date & time
   * Pi-hole versions (Core / Web / FTL) with *bounce scrolling*
   * Hourglass + clock icons

2. **System Stats**

   * CPU usage bar
   * RAM usage bar
   * Disk usage bar
   * CPU temperature thermometer
   * Wave-style temperature tick marks
   * Timestamp
   * Smooth bar animations with pixel-perfect edges

3. **Network Info**

   * eth0 IP with Ethernet jack icon
   * wlan0 IP with WiFi icon
   * Hostname with device icon
   * Icon vertical offsets for perfect alignment

4. **Blocklist Domains**

   * Total domains
   * Active (enabled) domains
   * 32×32 blocklist icon

5. **Pi-hole Stats** (v6 API)

   * Queries
   * Ads blocked
   * Percent blocked
   * Gravity domain count
   * 32×32 funnel icon

---

# 🏁 Title Screen

A clean graphical startup animation:

* Loads **ASCII bitmap** from:

  ```
  /opt/oled/pi-hole-logo.txt
  ```
* Renders a pixel-accurate logo (25×116 forced size)
* Bottom loading bar:

  * 2 px tall
  * 50% screen width
  * Rounded edges (corner pixels removed)
  * Micro-sweep animation with smooth fill

After completion, the title screen fades into the dashboard.

---

# 🌙 Idle Screensaver (Starfield Vortex)

After **5 seconds** of inactivity:

* A particle vortex animates across the full 128×32 OLED
* Stars spiral inward with:

  * Acceleration near center
  * Angular swirl
  * Trail lines
  * Automatic respawn
* Runs until button is pressed
* Returns **directly to System Stats screen**, *not* the title
* Fully supports an ASCII overlay mask:

```
1 = force white pixel
0 = force black pixel
X = transparent
```

Location:

```python
SCREENSAVER_OVERLAY = [...]
```

Overlay is perfectly centered every frame.

---

# 🔄 Reboot Animation (5-Second Circular Sweep)

Triggered via:

* **Button long-press (3 seconds)**
* Internal `reboot_event`

Animation details:

* Inner + outer circular borders (1-pixel crisp rings)
* Middle “donut band” fills clockwise over 5 seconds
* Black 1-pixel separators around fill to create a recessed effect
* Text displays on the right:

```
Reboot in 5...
Reboot in 4...
Reboot in 3...
Reboot in 2...
Reboot in 1...
Rebooting...
```

At the end, the Pi reboots.

---

# 🔘 Button + LED Behavior

The dashboard uses a **threaded, software-debounced input system**.

## Button (GPIO 16)

* **Short press** → Advance to next screen
* **Long press (3s)** → Trigger reboot animation
* Fully debounced
* Sampled at 200 Hz
* Zero CPU spike behavior

## LEDs

Two LEDs mirror button state:

| LED   | GPIO | Physical Pin |
| ----- | ---- | ------------ |
| LED 1 | 26   | Pin 37       |
| LED 2 | 5    | Pin 29       |

Thread `_led_follower()` polls switch at 100 Hz and updates LEDs instantly.

LED behavior:

* **On only while button is physically pressed**
* **Off otherwise**
* Independent of screen animation timing

---

# 🛠️ Hardware Setup

## Raspberry Pi Hardware

* **2 × Raspberry Pi 4B**
* USB-C power supplies
* Pi-hole installed on both
* SSD1306 OLED (128×32 I²C)
* Momentary push button
* 2 LEDs (button-follow behavior)

## I²C Wiring

* SDA → Pin 3
* SCL → Pin 5
* 3.3V → Pin 1
* GND → Pin 6

## GPIO Wiring

| Component | GPIO | Pin | Notes                          |
| --------- | ---- | --- | ------------------------------ |
| Button    | 16   | 36  | Pull-up enabled, GND to Pin 34 |
| LED 1     | 26   | 37  | Follows button                 |
| LED 2     | 5    | 29  | Follows button                 |
| GND       | —    | 39  | LED ground                     |

---

# 📦 Software Requirements

### System

```
Raspberry Pi OS 64-bit
Pi-hole v6 (required for API login)
Python 3.11+
```

### APT Packages

```
python3-pip
python3-smbus
i2c-tools
```

### Python Libraries

```
adafruit-circuitpython-ssd1306
psutil
requests
netifaces
Pillow
```

---

# 🚀 Installation

## Enable I²C

```
sudo raspi-config
Interface Options → I2C → Enable
```

## Install Pi-hole v6

```
curl -sSL https://install.pi-hole.net | bash
```

## Create Virtual Environment (Recommended)

```
python3 -m venv ~/oled-venv
source ~/oled-venv/bin/activate
pip install adafruit-circuitpython-ssd1306 psutil netifaces requests pillow
```

## File Structure

```
/opt/oled/oled_full.py
/opt/oled/pi-hole-logo.txt
/etc/systemd/system/oled-dashboard.service
/home/piadmin/oled-venv/
```

---

# 🔧 systemd Auto-Start

Create:

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

# 🔌 Pi-hole v6 API Authentication

`oled_full.py` includes a fully functional login to Pi-hole v6:

* Handles session (`sid`)
* Handles CSRF token
* Auto-refreshes validity before expiry
* Accesses `/api/stats/summary`
* Handles JSON conversion and error fallback
* Automatically relogins if expired

This replaces all older Pi-hole v5 API calls.

---

# ▶️ Runtime Behavior Summary

* Boot → Title Screen → Dashboard Loop
* Button taps rotate through screens
* After 5s idle → Screensaver
* Exit screensaver → System screen
* Long-press (3s) → Reboot animation → System reboot
* LED always mirrors button
* Threaded input ensures perfect responsiveness

---

# 🧩 Next Steps (Future Repo Additions)

*(Not included yet, but to be added later when you're ready)*

* Full SD-card image creation workflow
* Cluster fail-over management
* DNS waterfall configuration using Asus Merlin + dnsmasq
* Sync daemon between primary and failover Pi
* Gravity sync + version sync
* Auto-update propagation
* Alerts to ASUS router or external service

---

# 👤 Credits

Developed by **James DeRosa**
Part of the **Pi-hole Fail-Over Cluster Project**

All pixel graphics, animations, and UI elements custom built.
* Cluster monitoring agent
* SD-card imaging workflow

I can generate all of these.
