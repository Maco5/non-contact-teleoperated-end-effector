# Non-Contact Teleoperated End-Effector (Proportional Control)

**Author:** Maximilian Comsia  
**Institution:** University of British Columbia, Electrical Engineering (Biomedical Option)  
**Status:** Prototype V1.0 (Completed Fall 2025)  
**Tech Stack:** Embedded C++, PWM Actuation, Signal Processing, COTS Integration

![Project Banner]([LINK TO YOUR PHOTO])

## 1. Project Abstract
This project prototypes a **sterile, non-contact teleoperation interface** for a robotic manipulator. Designed to mimic the master-slave architecture of surgical systems, the interface uses ultrasonic time-of-flight sensing to translate the operator's hand distance into proportional finger flexion.

The core engineering focus was overcoming the limitations of low-cost hardware through firmware. I developed a custom C++ control loop that implements **signal debouncing**, **slew-rate limiting** (motion smoothing), and **sequential power management** to ensure stable operation on a standard USB bus.

## 2. System Architecture

### Hardware Implementation
* **Microcontroller:** ATmega328P (Arduino Uno R3)
* **Perception:** HC-SR04 Ultrasonic Transceiver (40kHz) for distance measurement.
* **Actuation:** 5x SG90 Micro-Servos (Tendon-driven).
* **End-Effector:** Integrated a COTS (Commercial Off-The-Shelf) InMoov-derivative hand kit.
* **Mechanical Mods:** Manually routed/tensioned nylon tendons and added elastomeric traction pads to fingertips to increase friction coefficient for grasping.

### Electrical Interface & Pin Mapping

| Component | Signal Pin (Arduino) | Power Source | Notes |
| :--- | :--- | :--- | :--- |
| **Ultrasonic Trigger** | D11 | 5V Bus | Sends 10µs pulse |
| **Ultrasonic Echo** | D12 | - | Returns pulse duration |
| **Thumb Servo** | ~D3 (PWM) | 5V Bus | Range: 90° - 175° |
| **Index Servo** | ~D5 (PWM) | 5V Bus | Range: 90° - 180° |
| **Middle Servo** | ~D6 (PWM) | 5V Bus | Range: 90° - 0° |
| **Ring Servo** | ~D9 (PWM) | 5V Bus | Range: 90° - 0° |
| **Pinky Servo** | ~D10 (PWM) | 5V Bus | Range: 90° - 0° |

*> **Note:** All servos share a common Ground (GND) to prevent floating voltage references.*

## 3. Firmware Engineering (C++)

The firmware (`main.ino`) executes a **20Hz control loop** featuring three distinct signal processing stages:

### A. Signal Conditioning (Glitch Filter)
The HC-SR04 sensor is prone to acoustic noise and packet loss (returning `0`).
* **Debounce Algorithm:** The system rejects invalid `0` readings and waits for **5 consecutive missing frames** before defaulting to the "Open" state. This prevents the hand from snapping open due to momentary sensor occlusion.
* **Input Clamping:** Raw data is constrained to a 5cm–20cm active window to filter out environmental background noise.

### B. Motion Control (Velocity Smoothing)
Direct mapping of sensor data to servos causes erratic, jerky movement.
* **Slew Rate Limiter:** Implemented a "Hydraulic" damping model where servo position is updated in fixed increments (`speedLimit = 0.8`) rather than instantaneous jumps.
* **Result:** This enforces a constant-velocity profile, allowing precise, non-destructive grasping of deformable objects (e.g., paper balls).

### C. Power Load Management
Driving 5 servos simultaneously causes current spikes >1.2A, risking USB brownouts.
* **Sequential Actuation:** The firmware updates finger positions in a staggered cascade (Thumb → 50ms Delay → Index...).
* **Result:** Distributes peak current draw over time, stabilizing the 5V rail.

## 4. Performance Validation
The system was validated through "Pick-and-Place" trials to test mechanical compliance and grip stability:

| Test Object | Morphology | Success Rate (n=10) | Engineering Notes |
| :--- | :--- | :--- | :--- |
| **Cardboard Roll** | Rigid Cylinder | 10/10 | High stability; sequential grip effective. |
| **Paper Ball** | Deformable Sphere | 10/10 | Velocity smoothing prevented crushing. |
| **Plastic Cup** | Low-Friction Cone | 9/10 | Required careful approach; traction pads essential. |

## 5. Setup & Calibration
1.  **Mount Sensor:** Position HC-SR04 facing the operator's workspace.
2.  **Flash Firmware:** Upload `main.ino`.
3.  **Calibrate:** Check Serial Monitor for `Target: [cm] | Robot: [pos]` output.
    * **5cm:** Hand Fully Closed (Grip).
    * **20cm:** Hand Fully Open (Release).

---
*Developed as an iterative engineering prototype to explore feedback control systems and electromechanical integration.*
