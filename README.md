# Line-Follower Robots: Arduino Bang-Bang to ESP32 PID with Bluetooth Tuning

Three stages of line-following robot firmware, from a simple two-sensor controller to a 5-sensor PID controller whose gains are tuned live over Bluetooth.

| Sketch | Board | Sensors | Control |
|---|---|---|---|
| `lf_basic/` | Arduino Uno | 2 × digital IR | Bang-bang; L293D motor driver |
| `ir_line_follower_uno/` | Arduino Uno | 2 × digital IR | Bang-bang with a **Timer0 prescaler change (~7.8 kHz PWM)** so TT gear motors run controllably at low speed; L298N |
| `esp32_pid_line_follower/` | ESP32 | 5-channel analog reflectance array (QTR) | **PID** with Kp/Ki/Kd set over **Bluetooth Classic**; L298N |

## How the PID version works
1. **Calibration:** at power-up the robot samples the sensors 400 times (about 10 s) to record per-channel minimum and maximum values. Midpoint thresholds are then computed.
2. **Position:** `qtr.readLineBlack()` returns 0–4000, and the error is `2000 − position`.
3. **PID:** `P = e`, `I += e`, `D = e − e_prev`. Each gain is sent as a value plus a power-of-ten divisor, for fine tuning from a phone. Motor commands are `base ± PID`, clamped to ±255.
4. **Line lost:** if every sensor reads white, the robot spins toward the side where the line was last seen.
5. **Bluetooth protocol:** 2-byte commands `[id, value]`. Ids 1/3/5 set Kp/Ki/Kd, 2/4/6 set their decimal multipliers, and 7 is run/stop.

## Hardware connections (ESP32 version)
| Function | ESP32 pins |
|---|---|
| Motor A (PWM, IN1, IN2) | 23, 21, 22 |
| Motor B (PWM, IN1, IN2) | 32, 25, 33 |
| Driver standby | 19 |
| QTR sensors 1–5 | 26, 27, 14, 12, 13 |

## Setup
- Arduino IDE with the ESP32 board package. Libraries: `QTRSensors` (Pololu) and `L298N`.
- Flash the sketch, keep the robot over the line during the calibration blink, then pair a Bluetooth terminal app and send gain commands.

## Limitations and future improvements
- The integral term has no anti-windup, and the gains are stored only in RAM. Add clamping and save the gains to NVS.
- Add wheel encoders for speed control, and log the error to Bluetooth so tuning can be plotted.

## Acknowledgements
The motor-driver and sensor wiring patterns are adapted from public hobbyist line-follower examples and the Pololu QTRSensors library examples.

## Technologies
`Arduino` `ESP32` `PID control` `Bluetooth Classic` `PWM` `Timer registers` `IR / reflectance sensors` `L298N` `Robotics`
