# Open Source Brushless CNC Machine Project

A custom desktop CNC machine built with affordable Amazon parts and a brushless spindle motor. This project is all about making a capable Desktop CNC Machine that's simple, expandable, and way better than the typical cheap brushed motor setups.

##  What This Is

A desktop CNC controlled by a GRBL board and Raspberry Pi running CNC.js. The main upgrade from budget CNCs is using a real brushless motor with a proper driver (NBD200) instead of the usual weak brushed motors. This means better speed control, more torque, less noise, and it'll actually last.

**Design Goals:**
- Keep it simple and reliable
- Use parts that are easy to find online
- Clean wiring that makes sense
- Room to add cool stuff later

##  Main Parts

### GRBL Controller Board
**RATTMMOTOR 2-Axis / 3-Axis GRBL Controller**
- Runs on 24V power
- Controls X, Y1, Y2, and Z stepper motors
- Has PWM output for the spindle
- Connects to Raspberry Pi via USB-C
- Built-in stepper drivers (no external drivers needed!)
<img width="1080" height="861" alt="Controller" src="https://github.com/user-attachments/assets/0086994e-1b0c-4712-9fe4-e25759799fc5" />

---

### Brushless Spindle
**Motor:** RATTMMOTOR 24V Brushless Spindle
- 104W power
- 10,800 RPM max speed
- ER8 collet for bits

**Driver:** NBD200 BLDC Motor Driver
- Takes 24V power
- Controlled by PWM signal from GRBL board
- Much smoother than brushed motors
<img width="1080" height="760" alt="Spindle" src="https://github.com/user-attachments/assets/ffc08e93-1a26-4fe8-b08e-d65e6007d8a8" />

---

### Power Supply
24V 5A power supply that runs both the GRBL board and spindle driver.
<img width="1080" height="1180" alt="PSU" src="https://github.com/user-attachments/assets/9de95817-accb-4e9b-948f-c190979b3bd0" />

### Raspberry Pi
Runs CNC.js so you can control everything from a web browser on your phone or computer. Gets powered separately with its own 5V USB charger.

<img width="193.5" height="137" alt="Pi 4" src="https://github.com/user-attachments/assets/7645315b-e44b-446e-8f75-30134dde669e" />

##  What It Does

- 3-axis CNC motion (X, Y, Z)
- Dual Y motors for better stability
- Speed control through G-code commands
- Web interface for controlling everything
- Upload G-code files and watch them run
- Tons of room for adding sensors, lights, etc.

##  How It's Wired

### Power
```
24V Power Supply
├── GRBL Board
└── NBD200 Spindle Driver
    └── Share the same ground! (Important!)

Raspberry Pi
└── Its own 5V charger
```

### Motors
Plug directly into the GRBL board:
- X motor → X header
- Left Y motor → Y1 header  
- Right Y motor → Y2 header
- Z motor → Z header

### Spindle Wiring
The brushless motor has 3 wires that go to the NBD200 driver (U, V, W). If it spins backwards, just swap any two wires.

### The Important Part: PWM Control
From GRBL board to NBD200 driver:
- PWM pin → PWM pin
- Ground → Ground

 **The shared ground is super important!** Without it, the PWM signal won't work right.

### Raspberry Pi
Just USB-C from the Pi to the GRBL board. That's it.

##  Software

The Raspberry Pi runs **[CNCjs](https://cnc.js.org/)**, which gives you a web page to:
- See where the machine is
- Move the axes manually
- Control spindle speed
- Upload and run G-code files
- Monitor your cuts in real-time

Just open a browser and go to the Pi's IP address. No complicated software to install on your computer.

##  Mechanical Build

Planning to use:
- Aluminum extrusion for the structure
- 3D printed motor mounts and brackets
- Rack and pinion mechanism for motion
- Drag chains for cable management

##  Future Upgrades I Want To Add

-  RGB LED status lights
-  Automatic tool height probe
-  Door switch for safety
-  Automatic dust collection control
-  Maybe add a laser module for engraving
-  Physical jog controller with buttons
-  Camera to watch jobs remotely
-  **Switch to belt drive or leadscrew system** for different speed/precision tradeoffs

## Contributing

This is open source! If you want to help out or have ideas:
- Share your own builds or modifications
- Suggest improvements
- Help with the frame design
- Make it better however you want

Feel free to fork it and make it your own!

---

**Status:** Work in progress - building and testing as I go!
