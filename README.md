Project: Simon Says Game on FPGA (Verilog)

Description
-----------
This project implements the classic "Simon Says" memory game on an FPGA board using Verilog HDL.
The design uses a VGA monitor to display four colored quadrants (red, green, blue, white) and a PS/2 keyboard as the input device.
The core logic generates random color sequences, shows them to the player, and then checks whether the player repeats the sequence correctly.

Main Features
-------------
- Implemented entirely in Verilog
- Top-level module: SimonSAys
- VGA output (640x480 @ 60 Hz timing)
  - hsync and vsync signals
  - 4-bit red, green, blue outputs
- PS/2 keyboard interface
  - kclk and kData inputs
  - Edge detection on PS/2 clock
  - Shift register used to decode specific key patterns
- Random sequence generator
  - 24-bit free-running register
  - XOR-based logic to produce random 2-bit values (0–3)
  - Stores up to 6 steps of the PC sequence
- Game state machine
  - Separate FSMs for:
    - Rounds (IDLE, RND1–RND6, NXTRND1–NXTRND6)
    - Game flow (PC, FIND, PLAYR, PLAYG, PLAYB, PLAYW, MAN, CTRL, CORRECT, FALSE, etc.)
  - Player input memory and comparison with PC sequence

Top-Level Module (I/O)
----------------------
Module name: SimonSAys

Inputs:
- clk   : System clock input (FPGA board clock)
- rst   : Asynchronous active-low reset
- kData : PS/2 keyboard data
- kclk  : PS/2 keyboard clock

Outputs:
- hsync : VGA horizontal sync
- vsync : VGA vertical sync
- red   : 4-bit red color bus
- green : 4-bit green color bus
- blue  : 4-bit blue color bus
- mitsos: 2-bit debug/auxiliary output
- Q     : 11-bit shift register (recent PS/2 bits for key decoding)

How It Works
------------
1) VGA Timing
   - Horizontal counter (Hcnt) and vertical counter (Vcnt) generate correct VGA timing.
   - The active video region is divided into four quadrants:
     - Each quadrant corresponds to one color: red, green, blue, white.
   - The design drives the RGB outputs depending on the current Hcnt/Vcnt region.

2) PS/2 Keyboard Interface
   - The PS/2 clock is sampled and filtered.
   - A falling-edge detector is used to safely capture bits from kData.
   - The incoming bits are shifted into register Q.
   - Specific bit patterns in Q are mapped to four "color keys".
   - Each color key is encoded as a 2-bit value (0–3).

3) Random Sequence Generator
   - A 24-bit register runs continuously and its bits are XOR-ed in pairs.
   - The XOR results are grouped into 2-bit values (val1–val6).
   - Each 2-bit value represents one color (0–3).
   - These values are stored into the PC memory array (mempc[0..5]) depending on the round.

4) Game Flow
   - PC phase:
     - The design shows the stored color sequence step-by-step on the screen.
     - Each color is displayed for a fixed time using separate counters.
   - Player (MAN) phase:
     - The module waits for keyboard input and stores each color choice in memman.
   - CTRL phase:
     - The PC sequence (mempc) is compared with the player sequence (memman).
     - If all elements match and the length is correct, the game goes to CORRECT.
     - If there is any mismatch, the game goes to FALSE and returns to IDLE.

Project Structure
-----------------
Suggested folder structure:

SimonSays_FPGA/
  src/
    SimonSAys.v
  README.txt


Hardware (Example)
--------------------------------
- FPGA Board: any board with VGA and PS/2 connectors (e.g. Xilinx Spartan / Artix, Intel Cyclone)
- Tools: vendor tools such as Xilinx Vivado/ISE or Intel Quartus
- Peripherals:
  - VGA monitor
  - PS/2 keyboard
  - Reset button for rst input

Basic Synthesis Steps
---------------------
1) Create a new FPGA project in your tool.
2) Add src/SimonSAys.v to the project.
3) Assign pins:
   - clk, rst
   - hsync, vsync
   - red[3:0], green[3:0], blue[3:0]
   - kclk, kData
4) Run synthesis, implementation/place-and-route, and bitstream generation.
5) Program the FPGA with the generated bitstream.

How to Play
-----------
1) Power on the FPGA board and connect:
   - VGA monitor
   - PS/2 keyboard
2) Use the defined start key (or key sequence) to begin the game.
3) Watch the color sequence displayed on the monitor.
4) Repeat the sequence using the corresponding keyboard keys.
5) If the input sequence is correct:
   - The game advances to the next round with a longer sequence.
6) If the input is wrong:
   - The FSM goes to the FALSE path and the game resets back to IDLE.

Possible Extensions
-----------------
- Add score or round indicator on screen.
- Add simple sound output (buzzer or audio PWM).
- Add a timeout if the player is too slow.
- Add a "WIN" screen after completing all rounds.
- Refactor the design into smaller modules (VGA, PS/2, FSM).

