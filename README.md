# Elevator Control System – Project Report

**Authors:** Lorenzo Barbieri  and Edoardo Saturno 

---

## Overview

This project presents the design and implementation of an **elevator control system** developed in the Codesys environment using generalized actuators and well-defined control policies.  
The system handles door operations, vertical movement, call management, and emergency/fault conditions to ensure safe and efficient operation.

---

## Generalized Actuators

### 1. `Doors_GA` (Door Management)
A generalized **DO/DONE** actuator that manages door opening and closing, including **presence detection**.

**Behavior:**
- If an obstacle is detected (via presence sensor), the system **prevents door closure** or **interrupts** the process and **reopens** the doors.
- The doors remain open until the obstruction is cleared.

**Inputs / Outputs:**

| Input | Meaning |
|--------|----------|
| `Do_` | Command signal |
| `DoWhat` | Action (Open / Close) |
| `Presence_sensor` | Detects obstacles |
| `State` | Internal state (Init, Busy, Ready) |

| Output | Meaning |
|---------|----------|
| `Done` | Operation completed |
| `Door_closed` / `Door_open` | Door position |
| `Opening_port` / `Closing_port` | Motor control signals |

---

### 2. `MovimentoVerticale_GA` (Vertical Movement)
A generalized **Start/Stop** actuator that controls the elevator’s **vertical motion**.

**Behavior:**
- Once activated, it moves the elevator **up or down** as directed.
- Movement stops when the actuator receives the appropriate stop condition.

**Inputs / Outputs:**

| Input | Meaning |
|--------|----------|
| `Start` | Command to begin movement |
| `StartWhat` | Direction (Go_up / Go_down) |
| `Stop` | Stop signal |

| Output | Meaning |
|---------|----------|
| `MotorOn_port` | Motor control |
| `Direction_port` | Motion direction |
| `State` | Internal state (Init, Busy, Ready) |

---

## Nominal Operation Policy

1. **Homing Phase:**  
   - On startup, the elevator closes its doors and moves slowly down to the **lower limit switch**, then rises to the **ground floor**.  
   - At the ground floor, it **opens the doors** and waits for a call.

2. **Call Handling:**  
   - Calls can be made from **internal** or **external** panels.  
   - Once a button is pressed, the corresponding **indicator light** is turned on.  
   - After **2 seconds**, the doors close, and the elevator starts moving.

3. **Movement Control:**  
   - The system uses **ramp sensors** to adjust speed dynamically.  
   - The cabin accelerates after the first ramp sensor and slows down before reaching the target floor.

4. **Arrival and Next Call:**  
   - Upon arrival, the system waits **2 seconds** before opening the doors.  
   - After **5 seconds**, it checks for any new pending calls.

---

## Call Management Policy

The system supports **on-the-fly calls** and **call memory** for those that cannot be served immediately.

**Key rules:**
1. **Internal calls** have **priority** over external calls.  
2. If moving **up**, the elevator serves the **lowest pending upward call** first.  
3. If moving **down**, it serves the **highest pending downward call** first.  

If an immediate stop is possible (e.g., the request comes from a floor ahead of the current position), the elevator performs an **intermediate stop**; otherwise, the call is stored and served later.

---

## Emergency Management Policy

When the **emergency button** inside the cabin is pressed:
- The elevator **slows down** and **stops at the next floor**.  
- If the button is pressed near a floor (between sensors), it stops at the **next available floor** to ensure sufficient braking distance.
- Doors open to allow passengers to exit.
- No further operations are allowed until the emergency button is **pressed again** to reset the system.
- Calls made during the emergency state are **stored** and will be processed once normal operation resumes.

---

## Fault Management Policy

### 1. Floor Sensor Faults
- Detected by monitoring discrepancies in **ramp and floor sensor counts**.
- When a fault is detected, the elevator **slows down** and **stops at the next floor**.  
- If the **ground floor** or **top floor** sensors fail, the elevator moves slowly to the corresponding **limit switch** before reversing direction.
- After stopping, **no further calls** are served.

### 2. Ramp Sensor Faults
- Detected in a similar way through counting inconsistencies.
- Upon detection, the elevator **slows down** and **stops at the next floor**.
- Once stopped, the elevator **does not serve additional requests**.

---

## Summary

This project defines:
- Two generalized actuators (`Doors_GA` and `MovimentoVerticale_GA`)
- Structured policies for:
  - Normal operation
  - Call handling
  - Emergency and fault conditions  

The result is a robust and safety-oriented control system for an elevator capable of handling multiple situations autonomously and efficiently.
