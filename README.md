# Hand Gesture Recognition for Embedded Device Control

Camera-based hand gesture control of ESP32 outputs. MediaPipe Hands runs in the browser, a deterministic rule table turns the 21 landmarks into a gesture, and a WebSocket link over Wi-Fi drives GPIO on the ESP32.

```
web-controller/                    Vite + React + TypeScript + Tailwind console
  src/config.ts                    gesture table, debounce N, pinch calibration (edit here during demo)
  src/vision/fingerState.ts        landmark geometry -> [T, I, M, R, P]
  src/vision/classifier.ts         pattern lookup + pinch rule
  src/vision/debounce.ts           N-frame consistency filter
  src/vision/commandPolicy.ts      edge-triggered packet policy + 10 Hz PWM stream
  src/vision/handTracker.ts        camera + MediaPipe Hands bridge
  src/net/esp32Link.ts             WebSocket client, reconnect, heartbeat
esp32-firmware/
  esp32_gesture_control/esp32_gesture_control.ino
docs/ORAL_DEFENSE_GUIDE.md         rubric-mapped study guide and live-edit cheat sheet
```

## Run the web console

### Prerequisite: install Node.js

1. Download the **LTS** Windows Installer (`.msi`) for Node.js 20 or newer from [nodejs.org](https://nodejs.org/).
2. Run the installer and keep the option to add Node.js to `PATH` enabled.
3. Open a new PowerShell or Command Prompt window and verify the installation:

   ```powershell
   node --version
   npm --version
   ```

   Both commands should print a version number. If either command is not recognized, restart your terminal (or VS Code) and try again.

### Start the web console

```powershell
cd web-controller
npm install
npm run dev        # http://localhost:5173
npm test           # classifier, debounce and packet-policy tests
npm run build      # production bundle in dist/
```

`npm run dev` and `npm run build` first copy the MediaPipe runtime from `node_modules` into `public/mediapipe`, so the console works with no internet connection (for example, when the laptop is joined to the ESP32's own access point).

Camera access requires `http://localhost` or `https`. Opening the dev server from a phone over plain `http://<laptop-ip>:5173` will block the camera.

## Flash the ESP32

1. Arduino IDE, Boards Manager: install "esp32 by Espressif Systems".
2. Library Manager: install "WebSockets" by Markus Sattler and "ArduinoJson" by Benoit Blanchon (v7).
3. Open `esp32-firmware/esp32_gesture_control/esp32_gesture_control.ino`. Set `WIFI_SSID` and `WIFI_PASSWORD`, or leave them as-is to use the fallback access point `GestureCtrl-ESP32` (password `gesture123`, address `192.168.4.1`).
4. Select "ESP32 Dev Module", upload, open Serial Monitor at 115200 baud. The board prints the `ws://` address to enter in the console.

## Wiring

| Output     | GPIO | Notes |
|------------|------|-------|
| Actuator 1 | 25   | LED + 220 ohm resistor to GND |
| Actuator 2 | 26   | LED + 220 ohm resistor to GND |
| Actuator 3 | 27   | LED + 220 ohm resistor to GND |
| Relay      | 14   | Relay module IN; firmware assumes active-low (`RELAY_ACTIVE_LOW`) |
| Buzzer     | 12   | Active buzzer via NPN transistor. GPIO 12 is a strapping pin; move to 13 if boot fails |
| PWM output | 33   | LED + resistor, or a MOSFET / motor driver input |

## Gestures

| Pattern [T,I,M,R,P] | Gesture | Effect |
|---|---|---|
| `[0,0,0,0,0]` | FIST | All outputs off |
| `[1,1,1,1,1]` | OPEN_PALM | All outputs on, PWM 255 |
| `[0,1,0,0,0]` | INDEX_UP | Actuator 1 only |
| `[0,1,1,0,0]` | PEACE | Actuator 2 only |
| `[0,1,1,1,0]` | THREE_FINGERS | Actuator 3 only |
| thumb-index tips close, middle/ring/pinky up | PINCH_PWM | PWM 0-255 from tip distance |

<img width="1912" height="1076" alt="act8maes" src="https://github.com/user-attachments/assets/955d0396-3189-407a-ba40-8d27df7522a7" />



