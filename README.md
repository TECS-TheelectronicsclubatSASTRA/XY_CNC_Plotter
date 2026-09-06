# XY_CNC_Plotter

✍️ 2D Writing Machine — CoreXY Pen Plotter (ESP32 + FluidNC)
A CoreXY pen plotter that reproduces handwriting and line art on paper, built from 3D-printed parts, machined steel rod, and a GT2 belt drive, controlled by an ESP32 running FluidNC and streamed G-code from Universal Gcode Sender.

     


📖 Table of Contents
Abstract
Key Features
System Architecture
Hardware Components
CoreXY Belt Kinematics
Mechanical Build
Software Stack
Getting Started
Firmware Overview
Challenges Encountered
What We Learned
Results
Project Team Workflow
Known Limitations & Future Work
Abbreviations
References
License


🧾 Abstract
This project designs and builds a 2D writing machine (pen plotter) that translates digital designs and text into physical drawings. A pen mounted on a CoreXY carriage moves across an X–Y plane in the same style as a 3D-printer gantry, capable of rendering both handwriting and complex line art. Compared to a traditional printer, the machine uses no ink cartridges, can write on almost any flat canvas, and reproduces vector artwork at high fidelity.

The system was originally prototyped with custom Arduino C++ firmware for manual/serial control, then migrated to a dedicated CNC workflow: the ESP32 is flashed with FluidNC, letting a computer talk to the machine as a standard CNC device through Universal Gcode Sender (UGS), which natively handles X/Y motion planning from G-code exported out of Inkscape.


✨ Key Features
Feature
Description
🧭 CoreXY "+" Belt Drive
A single open-loop GT2 belt routed in a "+" pattern drives both axes from two fixed motors, minimizing moving mass on the carriage
🖥️ G-code CNC Workflow
FluidNC turns the ESP32 into a standard GRBL-style CNC controller, recognized natively by Universal Gcode Sender
✏️ Servo-Actuated Pen Lift
An SG90 servo raises/lowers the pen between strokes for clean line breaks
🖨️ Fully 3D-Printed Structure
Custom SOLIDWORKS-designed sliders, motor mounts, and pen holder printed on Ultimaker 3 Extended printers
🔩 Hand-Machined Linear Rods
8 mm steel rod cut and faced down to the exact 300 mm lengths needed for the frame
🎨 Inkscape → G-code Pipeline
Any vector design or handwriting-style font can be exported from Inkscape and streamed directly to the machine
🔋 Isolated 12 V Motor Power
Steppers and drivers run from a dedicated 12 V 2 A supply, separate from ESP32 logic power



🏗️ System Architecture
┌──────────────────┐     Design / Text      ┌──────────────────┐

│     Computer       │ ─────────────────────▶ │     Inkscape       │

│ (Universal Gcode    │                        │ (vector → G-code)  │

│  Sender - UGS)       │ ◀───────────────────── └──────────────────┘

└─────────┬────────┘        G-code file

          │ USB Serial (G-code stream)

          ▼

┌──────────────────────┐

│        ESP32           │

│   (FluidNC firmware)    │

└─────────┬────────────┘

          │ STEP / DIR

          ▼

┌──────────────────────────────┐

│   2 × A4988 Stepper Drivers     │

│   → Nema 17 Motor A  (STEP/DIR)  │

│   → Nema 17 Motor B  (STEP/DIR)  │

└─────────┬─────────────────┘

          │ GT2 "+" belt (CoreXY)

          ▼

┌──────────────────────────────┐

│  X–Y Carriage + Pen Holder      │

│  SG90 Servo → Pen Up / Pen Down │

└──────────────────────────────┘

Data flow summary:

A design or piece of text is drawn/typed in Inkscape and exported as a G-code file.
Universal Gcode Sender streams that G-code to the ESP32 over USB serial.
FluidNC on the ESP32 interprets the G-code and generates coordinated STEP/DIR pulses for the two A4988 drivers.
The two Nema 17 motors drive the CoreXY "+" belt, moving the carriage to the target X/Y position.
The SG90 servo lifts or lowers the pen between strokes, reproducing the design on paper.


🔩 Hardware Components
Electrical

Component
Quantity
Purpose
ESP32 Development Board
1
Central controller — runs FluidNC, interprets G-code
A4988 Stepper Motor Driver
2
Drives the X/Y Nema 17 motors
Nema 17 Stepper Motor
2
CoreXY belt actuation
SG90 Servo Motor
1
Pen up/down (Z-axis equivalent)
Custom Control Wiring Board
1
Voltage splitter transistor, capacitors, driver/motor wiring
12 V 2 A Adapter
1
Main power for motors and drivers


Mechanical

Component
Quantity
Notes
3D-Printed Structural Parts
—
Sliders, motor mounts, pen holder — designed in SOLIDWORKS
8 mm Plain Steel Rod, 300 mm
4
Machined down from two 1000 mm rods (standard 300 mm stock was unavailable)
8 mm Threaded Rod, 300 mm
2
Cut from longer stock to match frame
GT2 6 mm Belt
~1 m+
Spliced/extended to reach around the custom "+" frame
LM8UU Linear Bearing
8
Slider motion on the 8 mm rods
LM6UU Linear Bearing
2
Slider motion on smaller-diameter rod sections
623ZZ Bearing (10×4×3 mm)
10
Belt idlers/rollers
GT2 Pulley
2
Mounted on Nema 17 shafts
Misc. Nuts, Bolts, Springs
—
Frame and tensioning hardware



🧮 CoreXY Belt Kinematics
The plotter uses a CoreXY ("+") belt layout, where a single belt path drives both axes from two stationary motors. Diagonal motor motion combinations were verified experimentally during hardware bring-up:

Direction
Motor A
Motor B
+X
Clockwise (+)
Counter-clockwise (−)
−X
Counter-clockwise (−)
Clockwise (+)
+Y
Clockwise (+)
Clockwise (+)
−Y
Counter-clockwise (−)
Counter-clockwise (−)


Which reduces to the standard CoreXY transform used throughout the firmware:

motorA_steps =  dx + dy

motorB_steps = -dx + dy

This relationship was first validated with a manual serial-jog test sketch before being handed off to FluidNC, which handles the same math internally once the machine is configured as a CoreXY (or standard Cartesian, depending on final YAML config) device.


🛠️ Mechanical Build
 
3D printing: All structural parts — sliders, motor mounts, pen holder, belt-joining piece — were modeled in SOLIDWORKS Student and sliced in Ultimaker Cura for printing on Ultimaker 3 Extended machines.
Rod machining: Standard 300 mm stock wasn't available locally, so two 1000 mm 8 mm steel rods were clamped in a bench vice and cut down to four 300 mm sections; the threaded rods were similarly trimmed to 300 mm.
Bearing assembly: LM8UU/LM6UU linear bearings were pressed into the printed slider bodies, with 623ZZ bearings used as belt idlers/rollers.
Belt routing: The GT2 belt is routed in an open-loop "+" pattern across the frame; because the custom frame dimensions exceeded a standard 1 m belt, the belt was spliced and extended to achieve correct tension.
Control board: A custom wiring board integrates both A4988 drivers, a voltage-splitter transistor, decoupling capacitors, and the harness connecting the ESP32 to the Nema 17 motors and SG90 servo.
  
 

🖥️ Software Stack
Layer
Technology
CAD / Part Design
SOLIDWORKS Student Edition
Slicing
Ultimaker Cura
Main Firmware
FluidNC (flashed onto the ESP32)
G-code Streaming
Universal Gcode Sender (UGS)
Vector Art / G-code Export
Inkscape
Hardware Bring-Up Only
Arduino IDE (manual serial-jog diagnostic sketch)



🚀 Getting Started
1. Prerequisites
SOLIDWORKS (or the provided STL/part files) to review or reprint structural parts
Ultimaker Cura for slicing
Arduino IDE with ESP32 board support (bring-up/testing only)
FluidNC firmware binaries + a FluidNC YAML config for this machine's pinout
Universal Gcode Sender installed on the controlling computer
Inkscape (v0.92 or compatible) for generating G-code from designs
2. Assembly
Follow the Mechanical Build section: print and bearing-fit the structural parts, machine the rods to length, assemble the X/Y sliders and pen holder, then route and tension the GT2 belt in the "+" pattern.
3. Wiring
Wire the two A4988 drivers to the ESP32 STEP/DIR/ENABLE pins, connect the Nema 17 motors to their respective drivers, and wire the SG90 servo signal pin plus 12 V motor power through the custom control board. Tie all grounds (ESP32, driver board, 12 V supply) together.
4. Flashing FluidNC
Flash the ESP32 with FluidNC (replacing any earlier Arduino test sketch).
Upload a FluidNC YAML configuration matching this machine's pin map, steps/mm, and CoreXY kinematics.
Connect to the board's config/console over USB or Wi-Fi to verify homing and jog commands.
5. Drawing
Create or import a design in Inkscape and export it as G-code.
Open Universal Gcode Sender, connect to the ESP32's serial port, and load the G-code file.
Press Start — the pen lifts, positions, and begins tracing the design on paper.


🧠 Firmware Overview
Hardware Verification Sketch (Arduino IDE — bring-up only)
Before moving to FluidNC, a manual-control Arduino sketch was used to verify CoreXY belt kinematics, motor directions, and stepper driver current limits over the serial monitor. It exposed simple jog keys (w a s d, diagonals), absolute/relative G/M-style move commands, pen up/down toggles, and a handful of self-test routines (axis isolation, perimeter, diagonals, centre cross).
Rejected G-code-Style Prototype
An intermediate ESP32 sketch (kept in the repo for reference, not used in the final build) implemented a lightweight subset of G-code parsing directly in Arduino C++ — G0/G1 moves, G28 homing, M3/M5 pen down/up, and M114 position reporting — using the same CoreXY step-splitting (Bresenham-style) approach. This was superseded by flashing FluidNC directly, which provides a complete, industrial-grade G-code interpreter instead of a hand-rolled one.
Final Architecture (FluidNC)
Stage
Role
Inkscape
Converts vector art/handwriting fonts into G-code
Universal Gcode Sender
Streams G-code over serial to the ESP32
FluidNC
Interprets G-code, plans motion, drives STEP/DIR signals via YAML-configured pins
A4988 Drivers + Nema 17
Execute coordinated CoreXY belt motion
SG90 Servo
Executes pen-up/pen-down commands between strokes



🧗 Challenges Encountered
ESP32 ≠ plug-and-play GRBL: Running on an ESP32 instead of an 8-bit Arduino meant the usual open-source GRBL G-code interpreter wasn't directly usable. This required flashing FluidNC, an industrial ESP32-targeted G-code/CNC firmware, in place of a standard Arduino IDE sketch.
Custom CoreXY frame needed custom parts: Using a "+" belt drive with an open-loop belt meant no off-the-shelf plotter kit parts would fit, so every slider, motor mount, and pen holder had to be designed from scratch and 3D printed.
Rod stock unavailability: 300 mm rods weren't available in the local market, so 1000 mm rod stock had to be hand-machined down to the required 300 mm lengths with a bandsaw and bench vice.


📚 What We Learned
SOLIDWORKS part design and hands-on 3D printing workflow, from CAD model to sliced, printed part.
Machining steel rod to precise length by hand (bandsaw + vice work).
Operating FluidNC and Universal Gcode Sender together as a CNC control pipeline, including producing usable G-code documents from Inkscape.


📊 Results
 
✅ Successfully 3D printed and assembled the full CoreXY structure — X/Y axes, middle sliders, and pen holder.
✅ Machined and fitted custom-length steel rod where standard stock was unavailable.
✅ Spliced and tensioned the GT2 belt to fit the custom "+" frame dimensions.
✅ Verified CoreXY belt kinematics, motor directions, and driver current limits via a manual serial-jog test sketch.
✅ Built a custom control board integrating both A4988 drivers, voltage regulation, and motor/servo wiring.
✅ Migrated from Arduino-based manual control to a full FluidNC + Universal Gcode Sender CNC workflow.

(Full G-code-driven writing-on-paper demonstration and final calibration are the last milestones — see Next Steps.)


👥 Project Team Workflow
Member
Focus
Visrudh Pranav Hariharan
Mechanical design / assembly
Dip Mondal
Electronics & control wiring
Mukundh Shrivatsa K S
Firmware & CNC software pipeline


The project was carried out end-to-end as a team across three tracks — mechanical fabrication, electronics/control wiring, and firmware/G-code tooling — integrated into a single working plotter.


🔮 Known Limitations & Future Work
FluidNC YAML settings still need final calibration for this specific machine's steps/mm and kinematics.
End-to-end testing of complex, multi-stroke G-code files generated in Inkscape is still pending.
A full demonstration of automated handwriting on paper is the next major milestone.
Planned: optimize the motion loop with a better PID controller for smoother, more accurate motion.
Planned: move toward autonomous operation, reducing the need for manual G-code streaming per job.
Future software idea: a pipeline to convert a user's own handwriting into a custom font, so the machine can "write" arbitrary text in that handwriting style.


🔤 Abbreviations
Abbreviation
Full Form
CNC
Computer Numerical Control
UGS
Universal Gcode Sender
MCU
Micro-Controller Unit
CAD
Computer-Aided Design
STL
Stereolithography (3D model file format)
PID
Proportional–Integral–Derivative (control loop)



📚 References
ESP32 Datasheet
FluidNC — G-code CNC firmware for ESP32
Universal Gcode Sender — G-code streaming host software
Inkscape — vector design and G-code export
Project reference notes: https://share.google/jwl8KGwCiCEOgCwME


📝 License
This project is released under the MIT License. Feel free to fork, modify, and build upon it — attribution appreciated.


Built as a mechanical + electronics + firmware integration project — combining 3D-printed hardware, machined rod stock, and a CoreXY belt drive into a CNC-controlled writing machine.
