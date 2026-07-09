# ABU-Robocon-Work
This repository marks the contribution made by me in the overall journey of Robocon and how did we implemented different ideas into hardware reality. It contain all the methodology implemented by me and how did that helped in the overall dedvelopment of the autonomous bot.

<p align="center">
  <img src="assets/banner.png" alt="Team Banner" width="850"/>
</p>

<h1 align="center">Team Dinobots(KIET Deemed To Be University) — Robocon Multi-MCU Autonomous Robot</h1>
<p align="center"><i>Built for DD Robocon | Mecanum-drive, FSM-based mission robot</i></p>

<p align="center">
  <img src="https://img.shields.io/badge/Event-DD%20Robocon-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/MCU-Arduino%20Due%20%7C%20STM32H7-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/Compute-Jetson%20Orin%20Nano-76B900?style=flat-square" />
  <img src="https://img.shields.io/badge/RTOS-FreeRTOS-green?style=flat-square" />
  <img src="https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square" />
</p>

---

## 📑 Table of Contents
- [About the Team](#about-the-team)
- [About DD Robocon](#about-dd-robocon)
- [Quick Start](#quick-start)
- [System Architecture](#system-architecture)
- [Hardware Setup](#hardware-setup)
- [Software / Firmware Build & Flash](#software--firmware-build--flash)
- [Running the Robot](#running-the-robot)
- [Missions](#missions)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [License](#license)

---

## About the Team

<p align="center">
  <img src="assets/team-with-bot.jpg" alt="Team member with the robot" width="480"/>
</p>

<!-- TODO: Replace with your team's story -->
We are a student robotics team competing in **DD Robocon**, building a fully autonomous, multi-MCU robot from the ground up — mechanical design, embedded firmware, control systems, and computer vision. This repository documents our robot's architecture, our mission logic, and everything needed to build, flash, and run the system.

> *We are Team Dinobots! we are fascinated by robots because they are reflection of ourselves. Bots shows us and our character and we foster on the reflection of ours in them*

---

## About DD Robocon

<p align="center">
  <img src="assets/dd-robocon-1.jpg" width="220"/>
  <img src="assets/dd-robocon-2.jpg" width="220"/>
  <img src="assets/dd-robocon-3.jpg" width="220"/>
  <img src="assets/dd-robocon-4.jpg" width="220"/>
</p>

<!-- TODO: 3-4 sentence intro to the event -->
**DD Robocon** is India's premier national-level robotics competition, serving as the official qualifier for the international ABU Robocon. For 2026, teams face the **"Kung Fu Quest"** theme, which challenges engineering students to build robots capable of martial arts-inspired agility and strategy. Competitors must operate a manual (R1) and an autonomous (R2) robot to assemble weapons in the Martial Club, independently navigate the booby-trapped Meihua Forest to collect Kung Fu Scrolls, and seamlessly combine physically to secure a winning line on a massive 3x3 Tic-Tac-Toe rack, achieving the ultimate "Kung Fu Master" victory.

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# See build instructions per MCU below
```

| Component        | Board              | Role                                   |
|-------------------|---------------------|-----------------------------------------|
| Due A             | Arduino Due         | Primary FSM / mission logic controller, Outer loop motor control and Inner loop motor control(rear motors)        |
| Due B             | Arduino Due         | Inner Loop motor control, slave of DUE A(front motor control)                                                     |
| Due C             | Arduino Due         | Sensor Integration, SPI communication with DUE A, 4X1D distance Lidar integration                                 |
| Vision Unit       | Jetson Orin Super Nano    | CV pipeline, object detection, Meihua UI                                                                          |

---

## System Architecture

<!-- TODO: embed or link an architecture diagram, e.g. assets/architecture.png -->
<p align="center">
  <img src="assets/architecture-diagram.png" alt="System architecture diagram" width="700"/>
</p>

The robot runs a **multi-MCU architecture**:
- **Due A** — top-level FSM mission controller, coordinates mission sequencing (`WEAPON`, `MEIHUA`, `TICTAC`, `LIFITNG`), cascade PID motor control (FreeRTOS), mecanum wheel kinematics
- **Due B** — Inner PID loop(fron motor control), Slave architect of DUE A
- **Due C** — Handles SPI comms with DUE A, 4X1D distance lidar inegration, BN085 IMU
- **Jetson Orin Nano** — vision pipeline (YOLO / DepthAI), feeds detections upstream

**Inter-MCU communication:** UART and SPI links between Due A, Due B and C, and the vision unit.

---

## Hardware Setup

1. **Chassis assembly** — mecanum wheel base, motor mounts
2. **Wiring** — motor drivers, encoders, IMU, proximity sensors, 1D distance lidars, Relays, Pneumatics (pressure)
3. **MCU interconnects** — UART/SPI wiring between Due A ↔ Due B/C ↔ Jetson
4. **Power distribution** — battery, regulators, fusing, Diodes, Capacitors

### Hardware Used

#### Arduino Due (32-bit ARM Cortex-M3)
<p align="center">
  <img src="assets/arduino-due.jpg" alt="Arduino Due Board" width="400"/>
</p>

* **Why we used it:** The DD Robocon arena requires micro-second precision for motor control and multiple fast serial ports for inter-board communication. The Due's 84 MHz clock speed and multiple hardware UART/SPI buses made it the perfect backbone over standard 8-bit microcontrollers.
* **How we used it:** We deployed three Dues in a master-slave architecture. **Due A** acts as the brain (FSM and cascade PID master), **Due B** handles the inner PID loops for the front motors, and **Due C** manages high-speed SPI polling for the IMU and LiDAR arrays.

#### NVIDIA Jetson Orin Nano
<p align="center">
  <img src="assets/jetson-orin-nano.jpg" alt="Jetson Orin Nano" width="400"/>
</p>

* **Why we used it:** The "Kung Fu Quest" theme requires detecting complex objects like the Meihua Forest poles and weapon racks on the fly. The Orin Nano provides massive edge AI computing power in a lightweight footprint, allowing us to run deep learning models without frame drops.
* **How we used it:** It runs our ROS2 (Humble) environment and YOLO vision pipelines. It takes feeds from the DepthAI cameras, processes bounding boxes and depth data, and streams alignment coordinates down to Due A via a dedicated UART link.

#### Mecanum Wheels & Drive System
<p align="center">
  <img src="assets/mecanum-wheels.jpg" alt="Mecanum Wheels Setup" width="400"/>
</p>

* **Why we used it:** Agility is everything when maneuvering through the booby-trapped Meihua Forest. Mecanum wheels provide holonomic, omnidirectional movement, allowing the bot to strafe sideways or rotate on its axis without needing space for traditional steering arcs.
* **How we used it:** Integrated into a custom-machined chassis and driven by high-torque DC motors. The kinematics equations are solved in real-time on FreeRTOS, mapping the X, Y, and Theta joystick/autonomous commands to individual wheel RPMs. 

#### BNO085 IMU & 4x1D Distance LiDARs
<p align="center">
  <img src="assets/sensors-setup.jpg" alt="LiDAR and IMU Sensors" width="400"/>
</p>

* **Why we used it:** Pure wheel encoder odometry suffers from slip, especially on standard arena carpets. The BNO085 provides a highly stable, drift-free heading (using its internal sensor fusion), while the 1D LiDARs give absolute, millimeter-accurate distances to the arena walls and game elements.
* **How we used it:** Mounted on the perimeter of the bot and wired to Due C. The IMU ensures the bot stays perfectly straight during the `WEAPON` sequences, and the 1D LiDARs are the core of our `meihuaIrAlign()` positioning logic for scoring on the Tic-Tac-Toe rack.

### Bill of Materials (BOM)

| Part | Qty | Notes |
|------|-----|-------|
| Arduino Due | 3 | Due A (Master) / Due B / Due C |
| Jetson Orin Nano | 1 | Vision/compute node |
| Mecanum wheels | 4 | Holonomic drive base |
| High-Torque DC Motors | 4 | With quadrature encoders |
| BNO085 IMU | 1 | For rotational odometry / heading |
| 1D Distance LiDAR | 4 | For absolute positioning against walls |
| Motor Drivers | 4 | High-current H-Bridges |

---

## Software / Firmware Build & Flash

<!-- TODO: fill in exact toolchain/commands you use -->
```bash
# Example — adjust to your actual build system
# Arduino Due (Due A / mission FSM)
arduino-cli compile --fqbn arduino:sam:arduino_due_x due_a/
arduino-cli upload  -p <PORT> --fqbn arduino:sam:arduino_due_x due_a/

# STM32H7 (motor control)
# Build via STM32CubeIDE / Makefile, flash via ST-Link
```

| Board | Toolchain | Flash Method |
|-------|-----------|--------------|
| Due A/B | Arduino IDE / arduino-cli | USB (native port) |
| STM32H7 | STM32CubeIDE | ST-Link |
| Jetson Orin Nano | ROS2 (Humble) / Python | SSH / direct |

---

## Running the Robot

<!-- TODO: describe your actual startup sequence -->
1. Power on chassis and confirm all MCU heartbeat LEDs
2. Launch vision pipeline on Jetson: `ros2 launch <package> vision.launch.py`
3. Verify UART/SPI link status between Due A and Due B/C
4. Trigger mission start (button / switch / remote command)
5. Monitor FSM state over serial (baud: `<rate>`)

---

## Missions

| Mission | Description |
|---------|--------------|
| **WEAPON** | [Brief mission description] |
| **MEIHUA** | IR-sensor alignment and positioning mission (`meihuaIrAlign()`) |
| **TICTAC** | [Brief mission description] |

---

## Tech Stack

**Languages:** C, C++, Python
**Frameworks/RTOS:** FreeRTOS, ROS2 (Humble)
**Vision:** YOLO (v5/v7/v8), DepthAI, OpenCV
**Control:** Cascade PID, mecanum wheel kinematics, FSM mission logic
**Hardware:** Arduino Due, STM32H7, Jetson Orin Nano

---

## Repository Structure

```
.
├── due_a/            # FSM mission controller firmware
├── due_b/            # Motor control / STM32H7 firmware
├── vision/           # Jetson CV pipeline (ROS2)
├── assets/           # Images used in this README
└── README.md
```

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

<p align="center"><i>Built with ⚙️ for DD Robocon</i></p>