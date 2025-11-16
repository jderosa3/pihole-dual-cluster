---

# **Pi-hole Fail-Over Cluster — CHANGELOG**

---

# **Version 1.6 — Dual-Stage Reboot/Shutdown, CRT UI, SSH Indicator, Faster Fade-Out**

**Date:** 2025-11-16

## 🔄 Dual-Stage Reboot/Shutdown System

* **Short press:** advance to next screen
* **3-second hold:** enter **Reboot Circle** animation
* **Continue holding 5 seconds:** enter **Shutdown Circle**
* **Release during reboot circle:** inverted CRT reboot countdown
* **Release during shutdown circle:** CRT-style shutdown confirmation screen
* **LED1 pulses 3×** before system halt
* OLED continues displaying safe-to-power-off message after halt
* Safe sequence: `sync → halt → OLED stays lit`

## 🖵 CRT-Style Shutdown Screen

* Bold, centered, double-drawn text
* “It’s safe to turn off Pi”
* White screen with **6-pixel Macintosh-style rounded corners**
* Screen remains active after CPU halt

## 🔥 Shutdown Circle Improvements

* Added “**Powering Off…**” text
* Layout matches reboot circle
* Smooth 5-second circular fill animation

## 🎨 Title Screen Fade-Out

* Replaced ordered dither with **fast randomized single-pixel dissolve**
* 100% safe for SSD1306 (draw-before-show pipeline)
* No race conditions or last-frame crashes
* Smooth 250 ms digital dissolve effect

## 🖥️ SSH Session Indicator

* New **33×9 CRT-style terminal icon**
* Appears at **top-right corner**
* Blinks at 0.5 s intervals
* Uses reliable `who` / `pts/` session detection
* Zero false positives
* Fully integrated into Status screen rendering

## 🔌 Network LED Enhancements

* **LED1:** button-only indicator
* **LED2–LED5:** real-time network activity LEDs

  * eth0 RX (LED2)
  * eth0 TX (LED3)
  * wlan0 RX (LED4)
  * wlan0 TX (LED5)
* Four threaded monitors (one per direction/interface)
* Reads `/sys/class/net/.../rx_bytes` and `tx_bytes`
* Very low CPU overhead (~0.3%)
* Works even if Wi-Fi missing or interfaces down

## 🛠 Stability & Code Fixes

* Fixed indentation issue preventing shutdown stage
* Eliminated draw-during-I²C-update crashes
* Ensured all OLED writes occur **after** draw operations
* Improved reboot/shutdown variable state handling
* Fixed reboot text overflow
* Centered final CRT text with proper vertical offsets
* Cleaned stage transitions and early-release logic

---

# **Version 1.5 — API Cleanup, Code Organization & Commenting Overhaul**

**Date:** 2025-11-15

## 🔖 Summary

Complete migration to a unified Pi-hole v6 stats engine.
Removed all deprecated v5 endpoints, variables, and fallback structures.

## 🔌 Network Activity LEDs

* Added LEDs 2–5 for per-interface RX/TX
* GPIO assignments:

  * LED2 → eth0 RX (BCM 5)
  * LED3 → eth0 TX (BCM 7)
  * LED4 → wlan0 RX (BCM 10)
  * LED5 → wlan0 TX (BCM 27)
* Fully threaded
* No interference with button LED
* Independent LED pulses for each traffic direction

## 🧠 Monitoring Engine

* 4 threads:
  `_net_activity_rx("eth0")`
  `_net_activity_tx("eth0")`
  `_net_activity_rx("wlan0")`
  `_net_activity_tx("wlan0")`
* Short LED pulses (20–30 ms)
* No kernel hooks
* Zero disk writes
* Auto-detects absent interfaces

## 🧼 API Cleanup

* Removed `get_pihole_stats()` (v5 summary API)
* Removed `ads_blocked_today`, `dns_queries_today`, `ads_percentage_today`
* Removed legacy environment variable references
* Standardized on `get_pihole_stats_v6()`
* Correct v6 endpoint: `/api/stats/summary`

## 🧹 Code Structure Refinement

* Updated all docstrings
* Removed old “fallback” / “legacy” comments
* Reorganized file into clear sections:

  1. Imports
  2. Pi-hole v6 Authentication
  3. OLED Setup
  4. GPIO Setup
  5. Icons
  6. Utilities
  7. Screens
  8. Screensaver
  9. Reboot/Shutdown Animation
  10. Main Loop

---

# **Version 1.4 — Pi-hole v6 API Migration & Stats Engine Overhaul**

**Date:** 2025-11-15

## 🚀 Full Pi-hole v6 Integration

* Removed all deprecated v5 endpoints
* Added v6 session-auth login
* SID + CSRF token extraction
* Automatic session refresh

## 🆕 Stats Retrieval

* New function: `get_pihole_stats_v6()`
* Retrieves:

  * DNS queries
  * Ads blocked
  * Percent blocked
  * Gravity domain count

## 🐞 Bug Fixes

* Fixed “all zeros” stats bug caused by missing v6 SID
* Added robust JSON parsing
* Added auto-recovery if Pi-hole restarts

---

# **Version 1.3 — Thermometer, UI Polish, Title Rewrite, Screensaver, Reboot Animation**

**Date:** 2025-11-14

## 🔥 Thermometer Enhancements

* Thickened walls (6px outer / 4px inner)
* Rounded tops
* Smoother scaling
* Wave tick marks (4-column offset pattern)

## 🎨 UI Layout Refinements

* Dozens of micro-shifts for pixel-perfect alignment
* Reworked bar lengths + spacing
* Better text alignment on all screens

## 🏁 Title Screen Rewrite

* Loads ASCII logo from `/opt/oled/pi-hole-logo.txt`
* Centered loading bar with rounded ends
* Initial fade-out improvements

## 🌙 Screensaver Upgrade

* Full vortex starfield animation
* Trail lines
* 80 animated particles
* Overlay mask system (`0/1/X`)
* Exits to System Stats screen

## 🔁 New Reboot Animation

* Circular reboot sweep
* Countdown text
* Smooth donut-style filling

---

# **Version 1.2 — Status Screen, Version Scroll, Icons, Long-Press Reboot**

**Date:** 2025-11-14

* Added Status Screen (uptime, time, Pi-hole versions)
* Added bounce-scrolling version line
* Added hourglass + analog clock icons
* Improved clipping behavior
* Added long-press (3s) reboot

---

# **Version 1.1 — Early Status Screen & Icon Improvements**

**Date:** 2025-11-12

* Initial uptime/time/version screen
* Horizontal scrolling implementation
* Better spacing + readability
* Added early long-press reboot support

---

# **Version 1.0 — Initial OLED Dashboard Release**

**Date:** 2025-11-11

* First title screen implementation
* Added System, Pi-hole, Domains, and IP Info screens
* Added gravity.db SQL queries
* Pixel-accurate SSD1306 rendering
* Debounced button
* LED follower
* systemd autostart service

---

If you want this exported as a **release-diff**, **GitHub Release Notes**, or a **short-form version for the repo front page**, just tell me.
