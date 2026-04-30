# 🛰️ DAUST 2U CubeSat — Software & Ground Station Dashboard

> **SAT-2U-001** · LEO 500 km · Senegalese student-built spacecraft  
> Developed at DAUST (Dakar American University of Science & Technology)

---

## Project Overview

This repository contains the web-based ground station software for the DAUST 2U CubeSat prototype. It includes:

- **Mission Animation** — A cinematic 3D fly-through showcasing the satellite's deployment sequence and subsystem activation
- **Interactive Dashboard** — A multi-page mission control interface with 3D model viewer, component status, telemetry streaming, and command terminal
- **3D Model Assets** — GLTF/GLB models of the 2U CubeSat (10 × 10 × 22.7 cm)

All interfaces run entirely in the browser — no server, build step, or installation required.

---

## File Structure

| File | Description |
|------|-------------|
| `animation.html` | Cinematic mission animation with 7-scene storyboard |
| `cubesat_3d_viewer.html` | Full dashboard — Overview, Components, Telemetry, Terminal |
| `cubesat-dashboard-3d.html` | Alternate dashboard layout focused on 3D model |
| `dashboard.html` | Lightweight dashboard variant |
| `cubesat-light.glb` | Compressed 3D model (GLB, ~6 MB) |
| `2U_CUBESAT_DP_DAUST-compressed.gltf` | GLTF model (references `buffer.bin`) |
| `buffer.bin` | Geometry buffer for GLTF model (~211 MB) |
| `stm32.png` / `BBB.png` / `dht11.png` | Component reference images |
| `SOFTWARE_WEBSITE.md` | Software architecture documentation |
| `cubesat_pages_spec_v2.md` | Frontend specification for planned React migration |

---

## Mission Animation (`animation.html`)

A cinematic, auto-playing 3D animation that walks through the CubeSat's mission lifecycle in 7 scenes:

| Scene | Title | Duration | Description |
|-------|-------|----------|-------------|
| 01 | **Deployment** | 7s | Ejected from dispenser into LEO |
| 02 | **Power Up** | 6s | Solar panels generate energy, EPS distributes power |
| 03 | **OBC Online** | 6s | STM32 F446RE real-time controller awakens |
| 04 | **Data Handling** | 9s | CAN bus link — STM32 ⇄ BeagleBone Black |
| 05 | **Telemetry** | 8s | Sensors stream data, MicroSD persistence, DHT11 |
| 06 | **Ground Link** | 9s | LoRa downlink, 10-minute pass over Dakar |
| 07 | **Mission Ready** | 7s | Full system overview |

### Controls

| Key | Action |
|-----|--------|
| `Space` | Pause / Resume |
| `←` / `→` | Previous / Next scene |
| `R` | Restart animation |

### Features

- Animated camera transitions with cubic easing
- Component markers with pulsing halos and labels
- Data flow visualization (animated packets between subsystems)
- LoRa ground station downlink beam
- Procedural starfield and Earth backdrop
- Cinematic letterbox framing with crosshairs

---

## Dashboard Pages

### 1. Overview (3D Model Viewer)

Interactive 3D viewer using Three.js. Users can rotate, zoom, and click component markers on the model. A side panel displays component details including firmware version, interface pins, and signal strength.

### 2. Components

Card grid showing the 8 onboard hardware subsystems with status badges:

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

### 3. Telemetry Stream

Simulated real-time telemetry with rolling line charts (Temperature, Humidity, RSSI), live values panel, and raw packet log. Data is generated client-side every 2 seconds.

### 4. Command Terminal

Mission control console with predefined commands (Ping, Request Telemetry, LED On/Off, Safe Mode, Reboot, etc.). Simulates TX/RX with latency and occasional timeouts.

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| 3D Rendering | Three.js r160 (ES modules via import map) |
| 3D Models | GLTF / GLB format |
| Styling | Vanilla CSS with custom properties (dark/light theming) |
| Logic | Vanilla JavaScript (no build step) |
| Typography | JetBrains Mono (data) · Space Grotesk (headings) |
| Data | Hardcoded mock data + client-side random generation |

### Planned Migration

| Layer | Technology |
|-------|------------|
| Framework | React 19 + Vite |
| Backend/DB | Convex |
| Styling | Tailwind CSS v4 |
| Charts | Recharts |

---

## How to Run

### Option 1 — Local HTTP Server (recommended)

The 3D models require HTTP to load due to browser CORS restrictions on `file://` URLs.

```bash
# Navigate to the software directory
cd software

# Start a local server
python3 -m http.server 8080

# Open in browser
open http://localhost:8080/animation.html        # Mission animation
open http://localhost:8080/cubesat_3d_viewer.html # Full dashboard
```

### Option 2 — Direct File Open (limited)

Opening the HTML files directly in a browser will work for the dashboard UI, but the 3D GLTF model will fall back to a procedural placeholder geometry.

```bash
open animation.html
open cubesat_3d_viewer.html
```

---

## Design System

### Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| Background | `#050810` | Dark theme base |
| Foreground | `#e5e7eb` | Primary text |
| Dim | `#6b7280` | Secondary text, timestamps |
| Teal | `#14b8a6` | Active indicators, accents |
| Blue | `#3b82f6` | Links, selected states |
| Green | `#10b981` | Online status, ACK |
| Amber | `#f59e0b` | Standby, LoRa |
| Red | `#ef4444` | Offline, timeout, danger |
| Purple | `#a855f7` | NRF radio |

### Status Badges

| Status | Color | Meaning |
|--------|-------|---------|
| `ONLINE` | Green | Component active and communicating |
| `READING` | Teal | Sensor actively taking measurements |
| `STANDBY` | Amber | Powered but idle |
| `OFFLINE` | Red | Not responding |
| `UNKNOWN` | Gray | State undetermined |

---

## Notes

- All data is **simulated** — no real hardware or satellite connection is required.
- The 3D model files (`cubesat-light.glb`, `2U_CUBESAT_DP_DAUST-compressed.gltf`, `buffer.bin`) must remain in the same directory as the HTML files.
- Optimized for desktop screens (~1400px width). Tested on Chrome, Firefox, Safari, and Edge.
- The `cubesat-light.glb` (6 MB) is the preferred model file. The GLTF variant references `buffer.bin` (211 MB) and is significantly larger.

---

## Authors

Mbaye Ndiaye Diouf
DAUST · Dakar American University of Science & Technology  
Junior Year · Design Project
