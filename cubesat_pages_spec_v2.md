# CubeSat Dashboard — Frontend Spec (UI Only)
## Pages: Component Status · Telemetry Stream · Command Terminal

> Stack: React 19 · Vite · Convex · Tailwind v4  
> All data is mocked/static for now. No hardware integration.  
> These are sub-routes under `/projects/cubesat/*`

---

## Shared Notes

- All three pages are new routes inside the existing CubeSat dashboard layout
- Data is hardcoded mock data — no real Convex queries yet, no live connections
- Use the same design language as the existing dashboard (colors, fonts, spacing)
- Navigation between the three pages lives in the CubeSat dashboard header/sidebar

---

## 1. Component Status Page

**Route:** `/projects/cubesat/components`

Shows each hardware component as a card with an image placeholder, name, interface info, and a status badge. Clicking a card expands a detail view.

---

### Components to display (mock data)

```ts
const COMPONENTS = [
  {
    key: "stm32",
    name: "STM32 Nucleo F446RE",
    role: "On-Board Computer",
    interface: "USB-Serial",
    interfaceDetail: "ttyACM0",
    status: "online",        // "online" | "standby" | "offline" | "reading" | "unknown"
    lastSeen: "0.8s ago",
    meta: { Firmware: "v0.4.2", Clock: "180 MHz", State: "NOMINAL" }
  },
  {
    key: "bbb",
    name: "BeagleBone Black",
    role: "Ground Station Computer",
    interface: "SSH",
    interfaceDetail: "192.168.7.2",
    status: "online",
    lastSeen: "1.2s ago",
    meta: { IP: "192.168.7.2", Uptime: "4h 12m", OS: "Debian 11" }
  },
  {
    key: "nrf_sat",
    name: "NRF24L01 #1",
    role: "Satellite TX — 2.4 GHz",
    interface: "SPI",
    interfaceDetail: "CE: PA4 · CSN: PA3",
    status: "standby",
    lastSeen: "8m ago",
    meta: { Role: "SAT TX", Channel: "76", "Data rate": "250 kbps" }
  },
  {
    key: "nrf_gs",
    name: "NRF24L01 #2",
    role: "Ground Station RX — 2.4 GHz",
    interface: "SPI",
    interfaceDetail: "CE: PB6 · CSN: PB7",
    status: "online",
    lastSeen: "0.5s ago",
    meta: { Role: "GS RX", Channel: "76", "Pkts recv": "1,204" }
  },
  {
    key: "lora",
    name: "RYLR890",
    role: "Long-Range RF Link",
    interface: "UART2",
    interfaceDetail: "115200 baud",
    status: "online",
    lastSeen: "0.9s ago",
    meta: { RSSI: "-87 dBm", SNR: "9.5 dB", Band: "915 MHz" }
  },
  {
    key: "sdcard",
    name: "MicroSD + Breakout",
    role: "Persistent Storage",
    interface: "SPI",
    interfaceDetail: "CS: PA8",
    status: "online",
    lastSeen: "2s ago",
    meta: { Capacity: "8 GB", Used: "2.3 MB", FS: "FAT32" }
  },
  {
    key: "dht11",
    name: "DHT11",
    role: "Temp + Humidity Sensor",
    interface: "GPIO",
    interfaceDetail: "PA1",
    status: "reading",
    lastSeen: "0.3s ago",
    meta: { Temp: "27.4 °C", Humidity: "61 %", Reads: "8,441" }
  },
  {
    key: "webcam",
    name: "USB Webcam",
    role: "Payload Camera",
    interface: "USB",
    interfaceDetail: "not detected",
    status: "offline",
    lastSeen: "never",
    meta: { Status: "Not found", Driver: "v4l2", Resolution: "1080p" }
  }
];
```

---

### Status colors

| Status    | Color  |
|-----------|--------|
| `online`  | green  |
| `reading` | teal   |
| `standby` | amber  |
| `offline` | red    |
| `unknown` | gray   |

---

### Layout

Grid of cards, 2–3 columns depending on viewport width.

**Each card contains:**
- A square image placeholder (gray box with a component icon or label — no real photos needed)
- Component name (bold)
- Role (subtitle, muted)
- Interface + detail (monospace, small)
- Status badge (colored pill: ONLINE / STANDBY / etc.)
- "Last seen" timestamp

**On click → expanded detail panel** (either a slide-over or an inline expanded state below the card):
- Same info as card
- Plus the `meta` key-value pairs shown as small labeled tiles

**Summary bar at the top of the page:**
```
8 components   ·   5 online   ·   1 standby   ·   1 offline   ·   1 reading
```

---

---

## 2. Telemetry Stream Page

**Route:** `/projects/cubesat/telemetry`

Shows simulated sensor data as rolling charts and a live value panel. All data is generated client-side with `setInterval` — no backend.

---

### Mock data generation

Generate a new telemetry reading every 2 seconds using `setInterval`:

```ts
function generateReading() {
  return {
    timestamp: Date.now(),
    temperature: 26 + Math.random() * 3,        // °C, range 26–29
    humidity: 58 + Math.random() * 6,            // %, range 58–64
    rssi: -90 + Math.random() * 8,               // dBm, range -90 to -82
    snr: 8.5 + Math.random() * 2,                // dB
    seqNum: prevSeq + 1,
  };
}
```

Keep the last 60 readings in state (React `useState` array, newest appended). The charts show this rolling 60-reading window.

---

### Layout

**Left: charts (3 stacked)**
- Temperature (°C) — line chart, green
- Humidity (%) — line chart, blue
- RSSI (dBm) — line chart, amber
- Each chart: ~120px tall, shared x-axis (relative time, newest on right)
- Use Recharts `<LineChart>` with `<ResponsiveContainer>`

**Right: latest values panel**
```
🌡  Temperature     27.4 °C
💧  Humidity        61 %
📡  RSSI            -87 dBm
〰  SNR             9.5 dB
📦  Packets         1,204
❌  Packet loss     0.8 %
```

**Bottom: raw packet log**
- Scrollable list of the last ~30 readings
- Each line: `[HH:MM:SS.mmm]  T=27.4 H=61.0 RSSI=-87 SNR=9.5 SEQ=1204`
- Monospace font
- Auto-scrolls to bottom on new reading

**Header controls:**
- `● LIVE` badge with pulsing dot (always on for now)
- `[Pause]` button — stops the interval, badge becomes `◌ PAUSED`
- `[Clear Log]` button — empties the raw log

---

---

## 3. Command Terminal Page

**Route:** `/projects/cubesat/terminal`

A mock mission control console to send predefined commands and see simulated responses. No actual serial or network calls — responses are simulated with a short delay.

---

### Command definitions

```ts
const COMMANDS = [
  { opcode: "PING",          label: "Ping",          group: "diagnostic", requiresConfirm: false, simulatedResponse: "PONG OK"          },
  { opcode: "REQUEST_TLM",   label: "Request Telemetry", group: "diagnostic", requiresConfirm: false, simulatedResponse: "T=27.4,H=61.0,RSSI=-87,SNR=9.5,SEQ=1205" },
  { opcode: "LED_ON",        label: "LED On",         group: "diagnostic", requiresConfirm: false, simulatedResponse: "ACK LED=ON"       },
  { opcode: "LED_OFF",       label: "LED Off",        group: "diagnostic", requiresConfirm: false, simulatedResponse: "ACK LED=OFF"      },
  { opcode: "LOG_DUMP",      label: "Dump Log",       group: "diagnostic", requiresConfirm: false, simulatedResponse: "100 lines follows..." },
  { opcode: "SET_MODE SAFE", label: "Safe Mode",      group: "control",    requiresConfirm: true,  simulatedResponse: "ACK MODE=SAFE"    },
  { opcode: "SET_MODE NOM",  label: "Nominal Mode",   group: "control",    requiresConfirm: true,  simulatedResponse: "ACK MODE=NOM"     },
  { opcode: "REBOOT",        label: "Reboot OBC",     group: "danger",     requiresConfirm: true,  simulatedResponse: "ACK REBOOTING\nBOOT v0.4.2" },
  { opcode: "FACTORY_RESET", label: "Factory Reset",  group: "danger",     requiresConfirm: true,  simulatedResponse: "ACK FACTORY_RESET DONE" },
];
```

---

### Send flow (simulated)

1. User clicks a command button
2. If `requiresConfirm: true` → show a confirmation modal: `"Send [OPCODE] to satellite?" [Cancel] [Confirm]`
3. Log entry appears immediately: `▶ [HH:MM:SS]  CMD:[OPCODE]` with status `SENDING...`
4. After 300–800ms random delay → response appears: `◀ [HH:MM:SS]  [simulatedResponse]`, status becomes `ACK`
5. Occasionally (10% chance) simulate a timeout: status `TIMEOUT`, response `No response after 3s`

---

### Layout

**Left: command log** (takes ~60% of width)
- Scrollable, newest at bottom, auto-scrolls
- Each TX line: `▶ [timestamp]  CMD:PING` in a muted color
- Each RX line: `◀ [timestamp]  PONG OK` in green (or red for TIMEOUT)
- `[Clear Log]` button at top right of this pane

**Right: command buttons** (takes ~40% of width)
- Grouped by `group`: Diagnostic / Control / Danger
- Danger group has a warning divider and requires confirm
- Each button: `[Label  ▶ TX]` — full width of the right pane, monospace label
- Buttons are disabled while a command is `SENDING`

**Header:**
- Link selector: `Link: LoRa ▾` (dropdown, no functionality yet — just display)
- Connection status: `● Connected` (always, mock)

---

## Navigation

Add a tab/nav bar inside the CubeSat dashboard that links to all three pages:

```
[Overview]   [Components 🔌]   [Telemetry 📡]   [Terminal ⌨️]
```

Active tab is highlighted. The nav lives below the CubeSat dashboard header.
