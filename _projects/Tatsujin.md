---
layout: page
title: Tatsujin
description: A Tatsujin (Taiko no Tatsujin) clone game implemented on PYNQ-Z2 FPGA with LED matrix display
img: assets/img/Tatsujin/Tatsujin_cover.png
importance: 2
category: school
---



**Tatsujin** is a FPGA-based rhythm game project that recreates the classic Taiko no Tatsujin (太鼓達人) gameplay experience. Developed as a final project for Logic Design course, this system uses **Verilog HDL** to implement game logic on a **PYNQ-Z2 FPGA board**, with three large buttons simulating drum inputs and a full-color LED matrix for visual display. The game features multiple songs, scoring system, and smooth animations across different game states.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/Tatsujin_cover.png" title="Tatsujin Game" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Tatsujin Game on PYNQ-Z2 FPGA
</div>

### 1. Project Overview

This project was inspired by the desire to bring the arcade Taiko no Tatsujin experience home. The team developed a simplified RGB LED matrix version of the game, using buttons to simulate drum hits and implementing three songs for gameplay.

**Team Members:**
* E94116148 蘇凡誠 (Fan-Cheng Su)
* E94114073 張哲維 (Che-Wei Chang)
* E94111059 許評詔 (Ping-Chao Hsu)
* E94116180 陳柏元 (Po-Yuan Chen)

### 2. Project Goals

The game is implemented in Verilog to mimic Taiko no Tatsujin gameplay, using three external large buttons to simulate drum surfaces as inputs. The game consists of three main screens: level selection, gameplay, and score summary, each with its own animations.

1. **Level Selection Screen**: Use red and blue buttons to switch between challenge levels, and yellow button to confirm and start the game. The screen displays the level name and operation instructions.

2. **Gameplay Screen**: Divided into two sections. The upper section displays the current accumulated score, while the lower section shows the note display area. Players must press the corresponding button when red or blue notes approach the judgment line. The closer to the judgment line, the higher the score, and consecutive correct hits provide score multipliers.

3. **Score Summary Screen**: Displays the player's final score on the screen and prompts to press the yellow button to return to the level selection screen for replay.

### 3. Design Approach

* **Logic Design**: Uses FSM (Finite State Machine) to control game states (START, MENU, PLAY, FINISH).
* **Matrix Rendering**: Utilizes case statements to store output data for each screen and render them on the LED matrix based on different states.
* **Note Display**: Uses offset values to create smooth movement effects.
* **Scoring System**: Calculates scores based on player's hit accuracy and combos, providing real-time feedback.

### 4. Implementation Details

The circuit is designed using Verilog and implemented on **PYNQ-Z2**, three large buttons, a full-color LED matrix, and breadboard to realize the Tatsujin game.

Detailed implementation:

1. **Music Score Recording**: Uses two bits to record a note—00 represents no note, 01 represents red, and 10 represents blue.

2. **Screen Display**: Connects screen data based on the current screen. If there are notes, it reads the note images from memory and draws them to the correct position based on the offset, then inputs the screen data into the buffer frame by frame according to the display panel specifications.

3. **Scoring Mechanism**: When a button is pressed, it determines if there is a note at the current detection position and calculates the score based on the offset, then removes that note.

4. **State Machine**: Uses a `state_button` module to control state transitions between START, MENU, PLAY, and FINISH, and shares the state with other modules.

### 5. Usage Instructions

#### Starting the Game
* Press any button from the start screen to enter the song selection menu
* Red button selects previous song; Blue button selects next song; Yellow button confirms the song selection
* Three songs are available: Rick Roll, Last Night, Good Night Miss

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/start_screen.png" title="Start Screen" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/menu_screen.png" title="Menu Screen" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Start Screen and Song Selection Menu
</div>

#### Game Interface
* After entering the game, you will see the following interface:
  * **Score Interface**: Displays the current accumulated score
  * **Note Interface**: Shows the position of Taiko notes

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/gameplay_screen.jpg" title="Gameplay Screen" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Gameplay Screen
</div>

#### Game Controls
* Follow the prompts to hit the corresponding keys
  * Use blue and red buttons to hit corresponding notes
  * Accurately hit notes in rhythm with the music to achieve high scores

#### Ending the Game
* After the game ends, the system displays your score
* Press the yellow button to restart the game

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/finish_screen.jpg" title="Finish Screen" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Score Summary Screen
</div>

### 6. Architecture

The system consists of the following submodules:

* **Shift_Load**: Handles data shifting and loading operations
* **state_button**: Controls game state transitions
* **Draw_main_scene**: Renders main game scenes
* **Draw_node**: Draws note graphics
* **Draw_score**: Displays score information
* **Score_Counter**: Manages score calculation
* **Button_judge**: Processes button input and judgment
* **clk_div**: Clock divider for timing control
* **Output_penel**: Output panel controller

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/architecture.jpg" title="System Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    System Architecture Diagram
</div>

### 7. Vivado Simulation Results

The following are partial simulation waveform screenshots:

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/shift_load.png" title="Shift_Load Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/output_panel.png" title="Output_Panel Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Vivado Simulation Waveforms
</div>

### 8. Project Summary

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/summary.png" title="Project Summary" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Project Summary
</div>

### 9. Future Improvements

* Add more music track selections
* Optimize graphical interface for better visual effects, e.g., using PWM to enhance rendering colors and score summary screen
* Connect speakers to integrate music playback and hit sound effects

---
## Resources

* [Source Code on GitHub](https://github.com/KendellHsu/Tatsujin/tree/TATSUJIN)
* [Demo Video](https://youtube.com/shorts/DisX86UEK40)

> We hope this project allows you to experience the fun of Taiko no Tatsujin gameplay, and we look forward to your valuable feedback to improve our design!
> 
> Email: kendellhsu@gmail.com
