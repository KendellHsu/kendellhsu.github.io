---
layout: page
title: LightPOV
description: Wireless IoT LED Performance System
img: assets/img/LightPOV/LightPOV_cover.JPG
importance: 1
category: maker
---

> Stay Tune!! The project is updating!!!
> New hardware、PCB、Effect Editing GUI is coming!

**LightPOV** is a comprehensive IoT project designed to revive and modernize legacy performance props for freshman orientation camps. The system integrates custom hardware, firmware, and software to create synchronized light shows. It features **Persistence of Vision (POV)** , allowing rotating props to display 2D images, all synchronized wirelessly to music.

## 🚀 Key Features
* **Wireless Group Control:** Synchronizes multiple devices via Wi-Fi with low-latency signal broadcasting.
* **POV Visual Effects:** High-speed LED refreshing creates stable 2D patterns when the props are spun.
* **Custom Choreography Software:** A dedicated Windows GUI allows users to visually edit light patterns, colors, and timing aligned with audio waveforms.
* **Robust Industrial Design:** Combines 3D-printed internal structures with high-impact acrylic tubes to withstand vigorous stage use.

## 🛠️ Technical Implementation

### 1. Hardware & Electronics
* **MCU:** Built on **ESP32** for its dual-core architecture and Wi-Fi capabilities.
* **Power Management:** Integrated 18650 Li-ion batteries with IP5306 modules for charging/discharging management and LDO regulators for stable 3.3V/5V rails.
* **PCB Design:** Iterative design process from breadboard prototypes to custom-manufactured PCBs (Version 1 & 2) to ensure signal integrity and durability.

#### Custom PCB Evolution
*From Prototype to Final Product*


### 2. Firmware (Embedded System)
* **Dual-Core Processing:** Leveraged ESP32's dual cores—**Core 0** handles Wi-Fi stacks/networking, while **Core 1** is dedicated to LED timing to prevent flickering.
* **Waveform Algorithms:** Implemented custom math functions (`ramp()`, `tri()`, `pulse()`, `step()`) to map HSV color spaces dynamically onto the LED strip coordinates.

### 3. Software Ecosystem
* **Backend Server:** Developed with **Node.js** (`server.js`) to handle device registration, heartbeats, and command broadcasting.
* **Effect Editor:** Built a C# **Windows Forms** application featuring a timeline-based editor. It exports JSON configuration files that the server parses to control the devices.
- Custom Software Interface
*Timeline-based Light Editor with Audio Waveform Integration*

---

## 🎬 Live Performance
*(Click to watch on YouTube)*

### 117th Freshman Orientation Camp


| 117th Orientation Camp | 27th ESCamp | 118th Orientation Camp |
| :---: | :---: | :---: |
| [![117th](https://img.youtube.com/vi/YyiI2fg6KxA/0.jpg)](https://www.youtube.com/watch?v=YyiI2fg6KxA) | [![27th](https://img.youtube.com/vi/DXz8Qr7GCnU/0.jpg)](https://www.youtube.com/watch?v=DXz8Qr7GCnU) | [![118th](https://img.youtube.com/vi/Ix1kZmECrI4/0.jpg)](https://www.youtube.com/watch?v=Ix1kZmECrI4) |
| **Props:** Stick, Snake, Ball | **Props:** Ball, Snake | **Props:** LightStick |

---

## Resources
* [Source Code on GitHub](https://github.com/ivan125126/light_light_light)

---
