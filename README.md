# STM32 Self-Balancing Robot

![Project Status](https://img.shields.io/badge/status-in--development-orange)
![Platform](https://img.shields.io/badge/platform-STM32-blue)
![Language](https://img.shields.io/badge/language-C-green)

A two-wheeled self-balancing inverted pendulum robot built using an **STM32** microcontroller. The robot utilizes an **MPU6050** Inertial Measurement Unit (IMU) to sense its orientation, processes this data through a PID loop, and drives two **DC shaft motors** via an **L298N** motor driver to maintain perfect equilibrium.

This project is split into two distinct execution phases: prototyping with off-the-shelf modules and migrating to a custom-designed Printed Circuit Board (PCB).

---

## 📌 Project Roadmap

- [x] **Phase 1: Breadboard Prototype** 
  - Wire STM32, MPU6050, and L298N on a chassis.
  - Implement basic I2C communication and PWM motor control.
- [ ] **Phase 2: Firmware Optimization**
  - Tune the PID parameters ($K_p$, $K_i$, $K_d$).
  - Implement encoder feedback for position tracking and drift prevention.
- [ ] **Phase 3: Custom PCB Layout**
  - Design a compact, integrated schematic in KiCad
  

---

## 🛠️ Hardware Architecture

### Phase 1: Prototype Components
* **Microcontroller:** STM32 (e.g., STM32F103C8T6 "Blue Pill") — Handles high-speed sensor parsing, PID calculation, and PWM generation.
* **IMU Sensor:** MPU6050 — Configured via I2C to read accelerometer and gyroscope raw data.
* **Motor Driver:** L298N Dual H-Bridge — Regulates power and direction to the motors based on logic signals from the STM32.
* **Actuators:** 2x DC Gear Shaft Motors (equipped with quadrature encoders for speed feedback).
* **Power Supply:** 3S LiPo battery (11.1V) to power the motors, stepped down via buck converter to 5V/3.3V for logic.

### Phase 2: Custom Board Upgrades
To reduce wiring complexity, eliminate inductive noise, and minimize the physical footprint:
* **Integrated MCU:** STM32 chip placed directly on-board with dedicated decoupling capacitors and an external crystal oscillator.
* **On-board Sensors:** Hardwired MPU6050 footprint to prevent I2C signal degradation over jumper wires.

---

## 💻 System Topology

The firmware relies on a real-time hardware feedback loop to maintain stability:

```text
  +-------------+       I2C       +-------------------+
  |   MPU6050   | --------------->|   STM32 (MCU)     |
  | (IMU Sensor)|                 |                   |
  +-------------+                 |                   |
                                  | 1.  PIID Control    |
  +-------------+      PWM/Dir    |    Calculation    |
  |    L298N    | <---------------|                   |
  | (Driver IC) |                 +-------------------+
  +-------------+
         |
         v
  +-------------+
  | Shaft Motors|
  +-------------+

Sensor Fusion: Merges raw accelerometer and gyroscope data from the MPU6050
               using a Kalman Filter to calculate a stable, precise tilt angle.

PID Control: Compares the actual tilt angle against the target upright angle
             (setpoint) to calculate corrections using three terms.

P (Kp): Corrects based on immediate tilt severity.
I (Ki): Corrects long-term drifting or leaning.
D (Kd): Dampens rapid movements to prevent over-correcting.

Motor Actuation: Converts the combined PID output into specific PWM speed
                 signals and direction commands sent directly to the L298N driver.
