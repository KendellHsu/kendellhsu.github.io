---
layout: page
title: Tatsujin
description: A Taiko no Tatsujin clone game implemented on PYNQ-Z2 FPGA with LED matrix display
img: assets/img/Tatsujin/start_screen.png
importance: 2
category: school
---
*蘇凡誠 (Fan-Cheng Su)*
、*張哲維 (Che-Wei Chang)*
、*許評詔 (Ping-Chao Hsu)*、*陳柏元 (Po-Yuan Chen)*


**Tatsujin** is a FPGA-based rhythm game project that recreates the classic Taiko no Tatsujin (太鼓達人) gameplay experience. It's a final project for Logic Design course, this system uses **Verilog** to implement game logic on a **PYNQ-Z2 FPGA board**, with three large buttons simulating drum inputs and a full-color LED matrix for visual display. The game features multiple songs, scoring system, and animations across different game states.

### Project Goals
- **Level Selection Screen**: Use red and blue buttons to switch between challenge levels, and yellow button to confirm and start the game. The screen displays the level name and operation instructions.

- **Gameplay Screen**: Divided into two sections. The upper section displays the current accumulated score, while the lower section shows the note display area. Players must press the corresponding button when red or blue notes approach the judgment line. The closer to the judgment line, the higher the score, and consecutive correct hits provide score multipliers.

- **Score Summary Screen**: Displays the player's final score on the screen and prompts to press the yellow button to return to the level selection screen for replay.

### Implementation Details

The circuit is designed using Verilog and implemented on **PYNQ-Z2**, three large buttons, a full-color LED matrix, and breadboard to realize the Tatsujin game.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/Tatsujin_Device.JPEG" title="Tatsujin Game" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Device Overview
</div>

Detailed implementation:

1. **State Machine**: A Finite State Machine (FSM) controls the overall game flow. The `state_button` module manages state transitions between `START`, `MENU`, `PLAY`, and `FINISH` states, and broadcasts the current state to other modules for coordinated operation.

2. **Screen Display (Matrix Rendering)**: The system utilizes `case` statements to store output data for each screen state and renders them on the LED matrix accordingly. Screen data is connected based on the current game state. When notes are present, the system reads note images from memory, positions them correctly using offset calculations, and then writes the screen data into the buffer frame by frame according to the display panel specifications.

3. **Note Display (Music Score Recording)**: Each note is encoded using two bits—00 represents no note, 01 represents red, and 10 represents blue. Offset values are applied to create smooth movement effects as notes travel across the display.

4. **Scoring Mechanism**: The scoring system calculates points based on the player's hit accuracy and combo streaks, providing real-time feedback. Upon button press, the system checks if a note exists at the current detection position, calculates the score based on timing accuracy (determined by offset), and removes the note upon successful hit.

### Architecture

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
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/architecture.png" title="System Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    System Architecture Diagram
</div>

### Usage Instructions

#### Starting the Game
Press any button from the start screen to enter the song selection menu.
Three songs are available: Rick Roll, Last Night, and Good Night Miss.
* **Red button:** selects previous song
* **Blue button:** selects next song
* **Yellow button:** confirms the song selection


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

#### In Game
After starting the game, you will see the interface below:
  * **Score Interface**: Displays the current accumulated score
  * **Note Interface**: Shows the position of Taiko notes

Follow the prompts to hit the corresponding keys. Use blue and red buttons to hit corresponding notes. Accurately hit notes in rhythm with the music to achieve high scores.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/gameplay_screen.png" title="Gameplay Screen" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Gameplay Screen
</div>

#### End Game
After the game ends, the system displays your score. Press the yellow button to restart the game.

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/finish_screen.png" title="Finish Screen" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Score Summary Screen
</div>


### Vivado Simulation Waveforms Results

<div class="row"> 
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/Vivado_1.png" title="Shift_Load Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Tatsujin/Vivado_2.png" title="Output_Panel Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Resources

* [Source Code on GitHub](https://github.com/KendellHsu/Tatsujin/tree/TATSUJIN)
* [Demo Video](https://youtube.com/shorts/DisX86UEK40)
