Crude Oil Heater Simulator
This repository contains the code for a crude oil heater simulator developed using assembly language for the PIC16F877A microcontroller. The simulation controls various inputs and outputs related to a crude oil heating system, ensuring the proper functioning of burners, pumps, valves, and safety mechanisms.

Features

Input Sensors:
Flame Detectors (4 channels)
Temperature Sensor (TT_101)
Pressure Sensors (PT_101, PT_102, PT_103)
Tank Level Sensors (T_201_NA, T_201_NB, T_202_NA, T_202_NB)
Gas Line Presence Sensor

Output Controls:
Ignition Sparkers (4 channels)
Line Pumps (3 channels)
Flow Control Valves (3 channels)
Burners (4 channels)
Solenoid Valves (4 channels)
Vent Controller

Operation

Initialization:
Configures input and output ports.
Clears all output registers.

Main Process:

Continuously monitors input conditions.
Controls the activation of pumps, burners, and valves based on sensor readings.
Ensures the safety protocols are followed, halting operations if unsafe conditions are detected.

Safety Procedures:
Monitors various burner combinations to prevent unsafe operations.
Emergency shutdown procedure (PARO_ACCION) engages when unsafe conditions are detected.

Subroutines:
VACIADO1: Handles tank emptying process.
SIN_GAS: Pauses operation if gas is absent.
T_20MIN: Implements a 20-minute delay using nested loops.
NIVEL: Manages tank level adjustments.
PRUEBA: Conducts a test cycle to ensure system readiness.
