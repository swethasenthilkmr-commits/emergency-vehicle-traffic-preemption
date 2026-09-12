Emergency Vehicle Traffic Preemption System – Manual Override Control

A centralized IoT-based Intelligent Transportation System (ITS) designed to give active emergency vehicles (ambulances, fire engines, police) immediate green-light priority at traffic intersections. Using dual ESP32 microcontrollers, cloud/server REST APIs, GPS tracking, and a real-time web dashboard, the system automates traffic preemption while enabling manual dispatcher overrides.

--------------------------------------------------------------------------------

Key Features

* Dynamic Traffic Preemption: Overrides default 2-lane traffic cycles to hold a continuous green light for approaching emergency vehicles.
* Safety Hold Interval: Executes an automatic 2-second "All-Red" phase prior to lane preemption to clear active intersection traffic safely.
* Real-Time Web Dashboard: Built with HTML5, CSS3, JavaScript, and Leaflet.js to map live vehicle GPS coordinates and monitor signal states via 2-second asynchronous AJAX polling.
* Manual Dispatcher Override: Allows control center operators to manually trigger or release lane preemption, reset system counters, or purge event logs directly from the UI.
* Auditable Event Logging & Analytics: Categorizes live operations by severity levels (Info, Warning, Danger, Error) and aggregates event logs inside a MySQL database for historical analysis.
* Portable Vehicle Power Unit: Mobile emergency node powered via a 2200 mAh 18650 Li-ion battery with integrated TP4056 charging and protection.

--------------------------------------------------------------------------------

Hardware Architecture & Pinout

Component Inventory:
- ESP32 Microcontroller (38-Pin) (Qty: 2) - Primary Traffic Light Controller & Emergency Vehicle Unit
- NEO-6M GPS Module (Qty: 2) - Real-time coordinate acquisition
- 5mm LEDs (2 Red, 2 Yellow, 2 Green) (Qty: 6) - 2-Lane Intersection Signal Simulation
- 220Ω Resistors (Qty: 6) - Current regulation for LEDs
- TP4056 Charging Module (Qty: 1) - Battery charge & over-discharge protection
- 18650 Li-ion Battery (2200 mAh) (Qty: 1) - Portable power for Emergency Vehicle unit
- GL12 Breadboard (840 Points) (Qty: 1) - Intersection circuit assembly
- MB-102 Breadboard (Qty: 1) - Emergency vehicle circuit assembly
- Push Button (Qty: 1) - Hardware manual preemption trigger

ESP32 GPIO Assignments:

Traffic Signal LEDs (Intersection Node):
        +-----------------------------------+
        |            ESP32 MCU              |
        |                                   |
        |  [GPIO 26] ---> 220Ω ---> Red    (Lane 1 - Main St)
        |  [GPIO 27] ---> 220Ω ---> Yellow (Lane 1 - Main St)
        |  [GPIO 14] ---> 220Ω ---> Green  (Lane 1 - Main St)
        |                                   |
        |  [GPIO 25] ---> 220Ω ---> Red    (Lane 2 - Cross St)
        |  [GPIO 33] ---> 220Ω ---> Yellow (Lane 2 - Cross St)
        |  [GPIO 32] ---> 220Ω ---> Green  (Lane 2 - Cross St)
        +-----------------------------------+

NEO-6M GPS Module:
- VCC: 3.3V
- GND: GND
- TX: RXD (GPIO 17)
- RX: TXD (GPIO 16)

--------------------------------------------------------------------------------

Software & Tech Stack

- Firmware: C++ / Arduino Framework (ESP32 WebServer, ArduinoJson libraries)
- Backend: PHP 8.x, Apache Server (XAMPP Environment)
- Database: MySQL / MariaDB (Event logging & spatial data aggregation)
- Frontend: HTML5, CSS3 (Flexbox/Grid), JavaScript (ES6+ AJAX/Fetch API)
- Mapping API: Leaflet.js with OpenStreetMap tile layer

--------------------------------------------------------------------------------

System Workflow & Signal Logic

+---------------------+      HTTP POST (JSON)      +----------------------+
|  Emergency Vehicle  | -------------------------> |   Central Web Server |
|  (GPS / PushBtn)    |                            |     (PHP / MySQL)    |
+---------------------+                            +----------------------+
                                                              |
                                                    HTTP GET  | Polling
                                                    (JSON)    v
+---------------------+      GPIO Signals          +----------------------+
|  Physical Traffic   | <------------------------- |   Intersection Node  |
|  Lights (LED Array) |                            |       (ESP32)        |
+---------------------+                            +----------------------+

1. Normal Cycle: Rotates between Lane 1 (Main St) and Lane 2 (Cross St).
   - Green: 5 seconds
   - Yellow: 2 seconds
   - All-Red Transition: 0.5 seconds
2. Preemption Phase:
   - Upon trigger, the active lane transitions from Green to Yellow (2s).
   - Both lanes switch to All-Red for 2 seconds to clear the intersection.
   - Priority lane switches to Green (Held) indefinitely until a release command is received.
3. Release Phase: Upon dispatcher or system release signal, the system resumes normal cyclic operation starting at Lane 1.

--------------------------------------------------------------------------------

REST API Endpoints

- GET  /              Serves the interactive dispatcher web dashboard
- GET  /api/status    Returns current signal states, preemption mode, uptime, and total triggers (JSON)
- GET  /api/history   Fetches the recent timestamped event logs (JSON)
- GET  /api/stats     Retrieves aggregate operation counters
- POST /api/ambulancel Triggers preemption hold for Lane 1 (Main St)
- POST /api/ambulance2 Triggers preemption hold for Lane 2 (Cross St)
- POST /api/release   Clears preemption state and restores normal traffic cycling
- POST /api/reset     Resets system stats and counters
- POST /api/clear-log Flushes internal event log buffer
- GET  /api/test      Health-check endpoint verifying ESP32 connectivity

--------------------------------------------------------------------------------

Project Structure

├── firmware/
│   └── traffic_controller.ino   # Complete ESP32 control & embedded WebServer code
├── backend/
│   ├── config.php               # MySQL database connections
│   ├── api.php                  # REST API endpoints for JSON payload processing
│   └── schema.sql               # Database structure for event logging
├── frontend/
│   ├── index.html               # Dispatcher dashboard structural template
│   ├── style.css                # CSS3 UI layout, badges, and warning banners
│   └── app.js                   # AJAX polling logic & Leaflet.js map integration
└── README.md                    # Project documentation

--------------------------------------------------------------------------------

Installation & Setup

1. Hardware Assembly:
   - Mount ESP32 modules onto breadboards.
   - Connect LEDs with 220Ω series resistors to dedicated GPIO pins (Lane 1: 26, 27, 14; Lane 2: 25, 33, 32).
   - Connect NEO-6M GPS modules via UART pins (GPIO 16 & 17).

2. Backend Setup:
   - Install XAMPP and launch Apache and MySQL services.
   - Import schema.sql into MySQL via phpMyAdmin.
   - Place backend PHP files into xampp/htdocs/traffic-system/.

3. Firmware Deployment:
   - Open firmware/traffic_controller.ino in Arduino IDE.
   - Ensure WiFi.h, WebServer.h, and ArduinoJson.h libraries are installed.
   - Configure Wi-Fi credentials:
     const char* ssid = "YOUR_WIFI_SSID";
     const char* password = "YOUR_WIFI_PASSWORD";
   - Flash the sketch to your ESP32 boards.
   - Monitor Serial Output (115200 baud) to obtain the assigned local IP address.

4. Accessing the Dashboard:
   - Open any browser on the network and navigate to http://<ESP32_IP_ADDRESS>/ or your local XAMPP web server address.

--------------------------------------------------------------------------------

Project Contributors

- Rohan Menon (25BCE5764) – Computer Science & Engineering
- Swetha Senthil Kumar Sini (25BCE5765) – Computer Science & Engineering
- Shiva S (25BCE5763) – Computer Science & Engineering
- S Yuvan Shankar (25BVD1085) – VLSI / Electronics Engineering

Academic Supervision: Dr. Usha Rani S.
Institution: School of Computer Science and Engineering (SCOPE), Vellore Institute of Technology (VIT), Chennai.
Alignment: UN Sustainable Development Goal 11 (Target 11.2: Sustainable Transport Systems & Smart Urban Infrastructure).
