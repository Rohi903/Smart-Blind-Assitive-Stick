Smart Assistive Blind Stick - Laptop ML Engine
Author: Waxi
Date: 2026

Features:
- Live camera feed from ESP32-CAM
- YOLOv8 object detection
- Voice output for detected objects
- Distance alerts from HC-SR04 sensors
- Scene description via Gemini Vision API (optional)
- Face recognition for known people (coming soon)

Usage:
    python blind_stick.py
"""

import cv2
import urllib.request
import numpy as np
from ultralytics import YOLO
import pyttsx3
import threading
import time
import queue
import socket

# ─── CONFIGURATION ───────────────────────────────────────────────
ESP32_CAM_IP  = "10.50.174.100"   # Change to your ESP32-CAM IP
LAPTOP_IP     = "10.50.174.216"   # Change to your laptop IP
CAM_URL       = f"http://{ESP32_CAM_IP}/cam.jpg"
SENSOR_PORT   = 5002              # UDP port for sensor data
COMMAND_PORT  = 5003              # UDP port for commands to ESP32

# Gemini API (optional - for scene description)
USE_GEMINI    = False             # Set True if you have API key
GEMINI_KEY    = "your-key-here"  # Paste your Gemini key here

# ─── MODEL SETUP ─────────────────────────────────────────────────
print("[INIT] Loading YOLOv8 model...")
model = YOLO("yolov8n.pt")
print("[INIT] YOLO ready!")

# ─── GEMINI SETUP (optional) ─────────────────────────────────────
if USE_GEMINI:
    try:
        import PIL.Image
        from google import genai
        client_gemini = genai.Client(api_key=GEMINI_KEY)
        print("[INIT] Gemini Vision ready!")
    except Exception as e:
        print(f"[INIT] Gemini setup failed: {e}")
        USE_GEMINI = False

# ─── VOICE ENGINE ────────────────────────────────────────────────
speech_queue = queue.Queue()
last_spoken   = {}

def speech_worker():
    """Runs in background thread — speaks queued text."""
    engine = pyttsx3.init()
    engine.setProperty('rate', 150)
    while True:
        text = speech_queue.get()
        if text is None:
            break
        engine.say(text)
        engine.runAndWait()

speech_thread = threading.Thread(target=speech_worker, daemon=True)
speech_thread.start()

def speak(text):
    """Speak text — same phrase won't repeat within 5 seconds."""
    now = time.time()
    if last_spoken.get(text, 0) + 5 < now:
        last_spoken[text] = now
        speech_queue.put(text)
        print(f"[VOICE] {text}")

# ─── GEMINI SCENE DESCRIPTION ────────────────────────────────────
def describe_scene(frame):
    """Send frame to Gemini and get scene description."""
    if not USE_GEMINI:
        return None
    try:
        rgb     = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        pil_img = PIL.Image.fromarray(rgb)
        response = client_gemini.models.generate_content(
            model="gemini-2.0-flash",
            contents=[
                "You are helping a blind person. Describe what is directly in front in one short sentence. Focus on obstacles and people only.",
                pil_img
            ]
        )
        return response.text
    except Exception as e:
        print(f"[GEMINI] Error: {e}")
        return None

# ─── NETWORK SETUP ───────────────────────────────────────────────
# Receive sensor data from ESP32
sensor_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sensor_sock.bind(('0.0.0.0', SENSOR_PORT))
sensor_sock.setblocking(False)

# Send commands to ESP32
cmd_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

def send_command(cmd):
    cmd_sock.sendto(cmd.encode(), (ESP32_CAM_IP, COMMAND_PORT))

# ─── MAIN LOOP ───────────────────────────────────────────────────
print("[MAIN] Starting Smart Blind Stick...")
print(f"[MAIN] Camera: {CAM_URL}")
print("[MAIN] Press Q to quit")

last_gemini_time = 0
front_distance   = 999
ground_distance  = 999

while True:
    try:
        # ── Grab frame from ESP32-CAM ──
        img_resp = urllib.request.urlopen(CAM_URL, timeout=5)
        img_np   = np.frombuffer(img_resp.read(), dtype=np.uint8)
        frame    = cv2.imdecode(img_np, cv2.IMREAD_COLOR)

        if frame is None:
            continue

        # ── YOLO Object Detection ──
        results = model(frame, verbose=False, conf=0.5)
        for result in results:
            for box in result.boxes:
                label = model.names[int(box.cls[0])]
                conf  = float(box.conf[0])
                speak(f"{label} detected")

        # ── Read sensor data from ESP32 ──
        try:
            data, _ = sensor_sock.recvfrom(1024)
            parts = data.decode().strip().split(',')
            if len(parts) == 2:
                front_distance  = int(parts[0])
                ground_distance = int(parts[1])

                # Distance voice alerts
                if front_distance < 50:
                    speak("Obstacle very close")
                    send_command("VIB_STRONG")
                elif front_distance < 100:
                    speak("Obstacle ahead")

                # Step / drop detection
                if ground_distance > 80:
                    speak("Warning, step or drop ahead")
                    send_command("VIB_STRONG")

        except BlockingIOError:
            pass

        # ── Gemini Scene Description (every 10 seconds) ──
        now = time.time()
        if USE_GEMINI and now - last_gemini_time > 10:
            last_gemini_time = now
            captured_frame   = frame.copy()
            def gemini_thread():
                desc = describe_scene(captured_frame)
                if desc:
                    print(f"[GEMINI] {desc}")
                    speech_queue.put(desc)
            threading.Thread(target=gemini_thread, daemon=True).start()

        # ── Display ──
        annotated = results[0].plot()
        cv2.putText(annotated, f"Front: {front_distance}cm",  (10, 25),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)
        cv2.putText(annotated, f"Ground: {ground_distance}cm", (10, 55),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

        cv2.imshow("Smart Blind Stick", annotated)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    except Exception as e:
        print(f"[ERROR] {e}")
        time.sleep(1)

# ─── CLEANUP ─────────────────────────────────────────────────────
speech_queue.put(None)
cv2.destroyAllWindows()
print("[MAIN] Shutdown complete."
