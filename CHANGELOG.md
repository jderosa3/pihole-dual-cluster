# Pi-hole Fail-Over Cluster Project

## **CHANGELOG**

---

# **Version 1.4 — Pi-hole v6 API Migration & Stats Engine Overhaul**

### **Date: 2025-11-15**

---

## 🚀 **Pi-hole v6 Compatibility**

### **Full Migration to New API**

* Removed all deprecated Pi-hole v5 API calls:

  * `/admin/api.php?summaryRaw`
  * `/admin/api.php?summary`
  * Legacy token-based endpoints
* Added complete Pi-hole v6.2.2 session-based authentication workflow:

  * `POST /api/auth`
  * Correct handling of returned **SID cookie**
  * Correct extraction of **CSRF token**
  * Automatic header injection for authenticated requests

### **Automatic Session Handling**

* Implemented **`pihole_login()`**:

  * Manages CSRF headers
  * Manages SID cookies
  * Tracks session validity window
  * Performs automatic re-login before expiration
  * Recovers gracefully if Pi-hole restarts

### **New Stats Retrieval**

Replaced all old v5 logic with `get_pihole_stats_v6()` to fetch:

* Total DNS queries
* Ads blocked
* Percent blocked
* Gravity domain count (correct v6 field)

### **Zero Stats Bug Resolved**

Older versions always showed:

```
Queries: 0
Blocked: 0
Blocked%: 0.0
```

Root cause:

* v6 endpoints returned 401 Unauthorized due to missing SID + CSRF handling.

Resolved by:

* Using session-based authentication
* Using `/api/stats/summary` endpoint
* Full error fallback handling

OLED now shows **real-time, correct Pi-hole statistics**.

### **Improved Error Stability**

* JSON parsing resilience
* Fallback values on fetch failures
* No screen crashes
* Auto-recovery if Pi-hole web service restarts

---

# **Version 1.3 — Thermometer, UI Polish, Title Screen Rewrite, New Screensaver, New Reboot Animation**

### **Date: 2025-11-14**

---

## 🔥 **Thermometer & Gauge Enhancements**

* Completely rebuilt CPU temperature thermometer:

  * Thickened tube walls (6 px outer, 4 px inner)
  * Rounded top corners
  * Merged bulb + tube geometry
  * Bulb overlaps tube for solid continuous look
  * Smoother temperature scaling
* Added **wave-style tick marks**:

  * 4-column wave pattern (`X··X / ·X·X`)
  * Offset bottom row by 1px for wave effect
  * Crisp, non-blocky shape
* Bar graphs improved:

  * CPU/RAM/DISK labels realigned
  * 1-px spacing between numeric values and bar
  * Bars shortened to prevent overlap with thermometer
  * Updated margins for perfect layout

---

## 🎨 **General Layout Refinements**

* Dozens of micro-adjustments (1px shifts) for perfect spacing
* Consistent vertical rhythm across all three hardware bars
* Thermometer width/height tuned to avoid pixel clipping
* Icons and text spacing updated for OLED clarity

---

## 🏁 **Title Screen Overhaul**

* Removed old scrolling text animation

* Now loads ASCII logo from:

  ```
  /opt/oled/pi-hole-logo.txt
  ```

* New bottom loading bar:

  * 2px height
  * Rounded corners (corner pixel removal)
  * Smooth sweep animation
  * Centered horizontally

* Added micro fade-out transition

* Simplified code, removed old unused font logic

---

## 🌙 **Idle Screensaver System**

* Added full idle detection (now **5 seconds** in newest code)
* Replaced old “star bounce” with **vortex drain animation**:

  * 80 stars
  * Accelerate toward center
  * Clockwise swirl
  * Full screen coverage
  * Trail-line rendering for fluid effect
* Added overlay mask support (centered, 27×21 ASCII):

  ```
  1 = force white
  0 = force black
  X = transparent
  ```
* Screensaver exit behavior:

  * Pressing button immediately returns to **System Stats**
  * Title screen is *not* replayed

---

## 🔄 **Reboot Animation — Full Redesign**

* 5-second smooth circular sweep
* Inner + outer border rings
* Middle donut “fill ring”
* Black inner/outer separators for recessed effect
* Countdown text:

  ```
  Reboot in 5…
  Reboot in 4…
  Reboot in 3…
  Reboot in 2…
  Reboot in 1…
  Rebooting…
  ```
* Ends with clean wipe + actual system reboot

---

# **Version 1.2 — Status Screen, Version Scroll, Icons, Long-Press Reboot**

### **Date: 2025-11-14**

* Added **Status Screen**:

  * Uptime counter
  * Current date/time
  * Pi-hole versions (Core/Web/FTL)
* Implemented **pixel-bounce scrolling** for version line
* New icons:

  * Hourglass (uptime)
  * Analog clock
* Improved spacing and clipping prevention
* Updated navigation order:

  ```
  Status → System → IP Info → Domains → Pi-hole Stats
  ```
* Implemented **3-second long-press** reboot trigger
* Added reboot semaphore event for safe thread control
* Greatly improved button responsiveness and debounce accuracy

---

# **Version 1.1 — Early Status Screen & Icon Improvements**

### **Date: 2025-11-12**

* Added initial uptime/time/version screen
* Added first-generation horizontal scrolling for long text
* Adjusted icon spacing, alignment, and readability
* Improved timing consistency across all screens
* Added switch behavior:

  * Short press = next screen
  * Long press = reboot (legacy version)

---

# **Version 1.0 — Initial OLED Dashboard Release**

### **Date: 2025-11-11**

* Original title screen with Pi-hole logo
* Added following screens:

  * System Stats
  * Pi-hole Stats (v5 API at that time)
  * Blocklist Domains
  * Network Info
* Implemented SQLite gravity.db queries
* Aligned all icons & bar graphs for SSD1306 clarity
* Added debounced button handling
* Added LED output tied to button state
* Added first systemd service for auto-start
* Established foundation for full OLED control system

---

# **End of Changelog**

---
