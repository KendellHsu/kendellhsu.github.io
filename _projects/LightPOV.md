---
layout: page
title: LightPOV
description: Wireless IoT LED Performance System
img: assets/img/LightPOV/LightPOV_cover.JPG
importance: 1
category: maker
---



**LightPOV** is a comprehensive IoT project designed to revive and modernize legacy from the maker space performance props for freshman orientation camps. The system integrates custom hardware, firmware, and software to create synchronized light shows. It features **Persistence of Vision (POV)** , allowing rotating props to display 2D images, all synchronized wirelessly to music.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightPOV_cover.JPG" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The Effect of Persistence of Vision (POV)
</div>

### 1. Electronics
#### Stick & Snake
* **MCU:** Built on **ESP32** for its dual-core architecture and Wi-Fi capabilities.
* **Power Management:** Integrated 18650 Li-ion batteries with IP5306 modules for charging management and AMS1117 as 3.3V/5V power regulator for stable 3.3V/5V rails.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/s_schematic.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LightStick、LightSnake Schematic
</div>

**PCB Layout:** Iterative design process from breadboard prototypes to custom-manufactured PCBs (Version1 to Version3).
We are designing  Version 4 Now !!

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/prototype_front.jpeg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/prototype_back.jpeg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PCB prototype
</div>

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/s_pcb_version1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/s_pcb_soldering.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PCB Version 1
</div>

In version 2, we rearranged the layout of the PCB to decrease the soldering difficulty and switched from 18650 battery to 14500 battery to reduce the tube radius.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/s_version2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/Version2_front.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/Version2_back.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PCB Version 2
</div>

#### Ball
The circuit utilizes an AMS1117-3.3 regulator to step down the input power into a stable 3.3V rail, which supplies both the ESP-12F microcontroller and the RGB LEDs. The MCU manages the lighting effects by driving 2N2222 transistors; these transistors act as high-speed switches, toggling the LEDs' ground connection based on logic signals from the MCU

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightBall Schematic.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LightBall Schematic
</div>


<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightBall_pcbLayout.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightBall_pcb_soldering.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LightBall PCB
</div>

### 2. Hardware Design

Light Stick and Snake use PC transparent plastic tubes for impact resistance.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightStick.png" title="example image" class="img-fluid rounded z-depth-1" %}
        <div class="caption">
            LightStick
        </div>
    </div>
</div>

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightSnake_core.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/LightSnake.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LightSnake
</div>

Light Ball uses a semi-transparent PE medicine container for better light effects.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/LightPOV/LightBall.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/LightPOV/LightBall_1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    LightBall
</div>


### 3. Firmware (Embedded System)
* **Dual-Core Processing:** Leveraged ESP32's dual cores—**Core 0** handles Wi-Fi stacks/networking, while **Core 1** is dedicated to LED timing to prevent flickering.
* **Waveform Algorithms:** Implemented custom math functions (`ramp()`, `tri()`, `pulse()`, `step()`) to map HSV color spaces dynamically onto the LED strip coordinates.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/Arduino_function.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Arduino costom math function for Light Effect
</div>

* **Wireless Group Control:** Synchronizes multiple devices via Wi-Fi with low-latency signal broadcasting. The server sends a queue of light effect to each device to ensure that the light effects between devices synchronize, compensating for Wi-Fi signal latency and instability.

## 4. Effect Editing GUI & IOT monitoring System

* **Backend Server:** Developed with **Node.js** (`server.js`) to handle device registration, heartbeats, and command broadcasting.

* **Effect Editor:** Built a C# **Windows Forms** application featuring a timeline-based editor. It exports JSON configuration files that the server parses to control the devices.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid 
           loading="eager" 
           path="assets/img/LightPOV/effect_editting_GUI.png" 
           title="Effect Editor" 
           class="img-fluid rounded z-depth-1" %}
        <div class="caption">
            Timeline-based Light Effect Editor with Audio Waveform
        </div>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid 
           loading="eager" 
           path="assets/img/LightPOV/control_UI.png" 
           title="Control UI" 
           class="img-fluid rounded z-depth-1" %}
        <div class="caption">
            Device control and monitoring interface
        </div>
    </div>
</div>

---
## 🎬 Live Performance
*(Click to watch on YouTube)*

<div class="caption" style="margin-bottom: 1rem;">
    <strong>117th Freshman Orientation Camp</strong>
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <a href="https://www.youtube.com/watch?v=YyiI2fg6KxA">
            <img src="https://img.youtube.com/vi/YyiI2fg6KxA/0.jpg" alt="117th Stick" class="img-fluid rounded z-depth-1">
        </a>
        <div class="caption">
            <strong>Props:</strong> Stick
        </div>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <a href="https://youtu.be/KIUfSW9J6ho">
            <img src="https://img.youtube.com/vi/KIUfSW9J6ho/0.jpg" alt="117th LightBall" class="img-fluid rounded z-depth-1">
        </a>
        <div class="caption">
            <strong>Props:</strong> Ball
        </div>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <a href="https://www.youtube.com/watch?v=Cy09c8EzSV8">
            <img src="https://img.youtube.com/vi/Cy09c8EzSV8/0.jpg" alt="117th LightSnake" class="img-fluid rounded z-depth-1">
        </a>
        <div class="caption">
            <strong>Props:</strong> Snake
        </div>
    </div>
</div>

<br>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <div class="caption" style="margin-bottom: 0.5rem;">
            <strong>27th ESCamp</strong>
        </div>
        <a href="https://www.youtube.com/watch?v=DXz8Qr7GCnU">
            <img src="https://img.youtube.com/vi/DXz8Qr7GCnU/0.jpg" alt="27th" class="img-fluid rounded z-depth-1">
        </a>
        <div class="caption">
            <strong>Props:</strong> Stick、Ball、Snake
        </div>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <div class="caption" style="margin-bottom: 0.5rem;">
            <strong>118th Freshman Orientation Camp</strong>
        </div>
        <a href="https://www.youtube.com/watch?v=Ix1kZmECrI4">
            <img src="https://img.youtube.com/vi/Ix1kZmECrI4/0.jpg" alt="118th" class="img-fluid rounded z-depth-1">
        </a>
        <div class="caption">
            <strong>Props:</strong> LightStick
        </div>
    </div>
</div>

---
## Resources
* [Source Code and PCB files on GitHub](https://github.com/ivan125126/light_light_light)

> Stay Tune!! The project is updating!!!
> New hardware、PCB、Effect Editing GUI is coming!

Replace the module with mounted electronics to reduce the core radius.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/Version3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PCB Version 3
</div>

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/LightPOV/new_hardware.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    New Device Design in Fusion360
</div>


