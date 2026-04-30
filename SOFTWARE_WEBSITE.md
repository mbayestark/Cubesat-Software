# DAUST CubeSat Mission Control Dashboard — Software Overview

## Project Summary

The CubeSat Mission Control Dashboard is a web-based ground station interface developed for the DAUST 2U CubeSat prototype (designation **SAT-2U-001**). It provides a unified interface for monitoring satellite subsystems, visualizing telemetry data, and sending commands to the spacecraft — all within a browser.

The current implementation uses standalone HTML files with vanilla JavaScript and Three.js for 3D rendering. A future iteration is planned using React 19, Vite, Convex, and Tailwind CSS v4.

---

## Architecture

| File | Purpose |
|------|---------|
| `cubesat_3d_viewer.html` | Full-featured dashboard with all four views (Overview, Components, Telemetry, Terminal), dark/light theme support, and interactive 3D model |
| `cubesat-dashboard-3d.html` | Alternate dashboard layout focused on the 3D model with side panels for system status, radio links, and telemetry |
| `dashboard.html` | Lightweight dashboard variant |
| `cubesat_pages_spec_v2.md` | Frontend specification document for the planned React migration |
| `2U_CUBESAT_DP_DAUST-compressed.gltf` / `cubesat-light.glb` / `buffer.bin` | 3D model assets for the CubeSat (GLTF/GLB format) |
| `*.png` | Component reference images (STM32, sensors, etc.) |

---

## Dashboard Pages

### 1. Overview (3D Model Viewer)

The landing page renders an interactive 3D model of the 2U CubeSat (10 x 10 x 22.7 cm) using **Three.js**. Users can:

- **Rotate** the model by dragging
- **Zoom** with the scroll wheel
- **Toggle auto-rotation** on/off
- **Click interactive markers** placed on the model to inspect individual hardware components

A side panel lists all components with status indicators and, when a component is selected, shows detailed metadata (firmware version, interface pins, signal strength, etc.).

### 2. Components

Displays the eight onboard hardware subsystems as a card grid. Each card shows:

- Component name and role
- Communication interface and pin/address details
- Status badge (Online, Standby, Offline, Reading, Unknown)
- Last-seen timestamp

Clicking a card expands it to reveal detailed metadata (e.g., firmware version, clock speed, RSSI, storage usage).

**Hardware components tracked:**

| Component | Role | Interface |
|-----------|------|-----------|
| STM32 Nucleo F446RE | On-Board Computer (OBC) | USB-Serial |
| BeagleBone Black | Ground Station Computer | SSH |
| NRF24L01 #1 | Satellite TX (2.4 GHz) | SPI |
| NRF24L01 #2 | Ground Station RX (2.4 GHz) | SPI |
| RYLR890 | Long-Range RF Link (915 MHz LoRa) | UART |
| MicroSD + Breakout | Persistent Storage (FAT32) | SPI |
| DHT11 | Temperature + Humidity Sensor | GPIO |
| USB Webcam | Payload Camera | USB |

A summary bar at the top shows counts by status (e.g., "5 online, 1 standby, 1 offline, 1 reading").

### 3. Telemetry Stream

Simulates real-time satellite telemetry using client-side data generation (readings every 2 seconds). The page contains:

- **Three rolling line charts** — Temperature (C), Humidity (%), and RSSI (dBm) — showing the last 60 readings
- **Live values panel** — current Temperature, Humidity, RSSI, SNR, packet count, and packet loss percentage
- **Raw packet log** — scrollable monospace log of timestamped readings, auto-scrolling to the newest entry
- **Controls** — a pulsing LIVE badge, Pause/Resume button, and Clear Log button

All data is mock-generated; no backend or hardware connection is required.

### 4. Command Terminal

A simulated mission control console for sending predefined commands and viewing responses. Features include:

- **Command buttons** grouped into three categories:
  - **Diagnostic** — PING, Request Telemetry, LED On/Off, Dump Log
  - **Control** — Safe Mode, Nominal Mode (require confirmation)
  - **Danger** — Reboot OBC, Factory Reset (require confirmation via modal dialog)
- **Command log** — shows transmitted commands (TX) and received responses (RX) with timestamps and status indicators (SENDING, ACK, TIMEOUT)
- **Simulated latency** — responses appear after a 300-800 ms random delay, with a 10% chance of a timeout
- **Link selector** — displays the active communication link (LoRa) and connection status

---

## UI/UX Design

- **Dark and light themes** — toggleable via a button in the header; the dark theme uses a deep navy/charcoal palette, the light theme uses a clean gray/white palette
- **Monospace typography** — JetBrains Mono for data, labels, and technical content; Space Grotesk for headings
- **Color-coded status system:**
  - Green — Online / ACK / nominal
  - Amber — Standby / warning
  - Red — Offline / timeout / danger
  - Teal — Reading / active sensor
  - Blue — accent / selected / sending
- **Responsive card layouts** using CSS Grid
- **Animated elements** — pulsing live indicators, loading spinners, smooth transitions

---

## Technology Stack (Current)

| Layer | Technology |
|-------|------------|
| Rendering | HTML5 Canvas + Three.js (r128) |
| 3D Models | GLTF / GLB format |
| Styling | Vanilla CSS with CSS custom properties (theming) |
| Logic | Vanilla JavaScript (no build step) |
| Data | Hardcoded mock data and client-side random generation |

### Planned Stack (React Migration)

| Layer | Technology |
|-------|------------|
| Framework | React 19 + Vite |
| Backend/DB | Convex |
| Styling | Tailwind CSS v4 |
| Charts | Recharts |

---

## How to Run

1. Open `cubesat_3d_viewer.html` in a modern web browser (Chrome, Firefox, Safari, or Edge).
2. The 3D model loads automatically from the GLTF/GLB files in the same directory.
3. Use the tab navigation to switch between Overview, Components, Telemetry, and Terminal.
4. No server, build step, or installation is required — it runs entirely in the browser.

---

## Notes

- All data displayed is **simulated** — the dashboard does not connect to real hardware or a satellite.
- The 3D model files (`2U_CUBESAT_DP_DAUST-compressed.gltf`, `cubesat-light.glb`, `buffer.bin`) must remain in the same directory as the HTML files for the viewer to load correctly.
- The dashboard is designed at a fixed width of ~1400px and is optimized for desktop screens.
