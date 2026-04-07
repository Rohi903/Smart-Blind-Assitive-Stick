# Smart Assistive Blind Stick

A smart assistive device for visually impaired people using ESP32, computer vision, and AI.

## Features
- Real-time object detection using YOLOv8
- Haptic feedback via vibration motors based on obstacle distance
- Voice output announcing detected objects and distance alerts
- Live camera streaming from ESP32-CAM over WiFi
- Scene description using Gemini Vision API (optional)
- Face recognition for identifying known people (coming soon)

## Architecture
```
ESP32-CAM ──WiFi──► Laptop (YOLO + Voice + Gemini)
ESP32 DevKit ──WiFi──► Laptop (sensor data)
HC-SR04 x2 ──► ESP32 DevKit (distance sensing)
Vibration Motors x2 ◄── ESP32 DevKit (haptic feedback)
```

## Hardware Required
| Component | Quantity | Purpose |
|---|---|---|
| ESP32 DevKit WROOM-32 | 1 | Sensor + motor control |
| ESP32-CAM AI Thinker | 1 | Camera streaming |
| HC-SR04 Ultrasonic Sensor | 2 | Distance detection |
| Coin Vibration Motor | 2 | Haptic feedback |
| Breadboard | 1 | Prototyping |
| Jumper Wires | 1 set | Connections |
| Resistors (1kΩ, 2kΩ) | 2 each | Voltage divider |

## Wiring
### ESP32 DevKit
```
HC-SR04 #1 (Front):
  VCC  → VIN
  GND  → GND
  TRIG → GPIO 5
  ECHO → GPIO 18 (via 1kΩ resistor)

Vibration Motor #1:
  Signal → GPIO 22
  GND    → GND

Vibration Motor #2:
  Signal → GPIO 23
  GND    → GND
```

### ESP32-CAM
```
5V  → Breadboard red rail
GND → Breadboard blue rail
```

## Software Setup

### Laptop
```bash
pip install ultralytics opencv-python pyttsx3 google-genai Pillow
```

### Arduino IDE
- Install ESP32 board support
- Board: AI Thinker ESP32-CAM (for camera)
- Board: ESP32 Dev Module (for sensor board)

## How to Run

1. Upload `esp32_sensor/esp32_sensor.ino` to ESP32 DevKit
2. Upload `esp32_cam/esp32_cam.ino` to ESP32-CAM
3. Run on laptop:
```bash
python blind_stick.py
```

## Project Structure
```
smart-blind-stick/
├── esp32_sensor/
│   └── esp32_sensor.ino    # Sensor + motor code for ESP32 DevKit
├── esp32_cam/
│   └── esp32_cam.ino       # Camera streaming code for ESP32-CAM
├── blind_stick.py           # Main laptop ML code
└── README.md
```

## Demo
- Point stick at any object → voice says "chair detected"
- Bring stick close to obstacle → rapid vibration + "obstacle very close"
- Every 10 seconds → Gemini describes full scene

## Built With
- YOLOv8 (Ultralytics)
- Google Gemini Vision API
- ESP32 Arduino framework
- OpenCV
- pyttsx3

## Author
Rohith — College Project 2026
