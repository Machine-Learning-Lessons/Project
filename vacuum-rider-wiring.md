# Vacuum IMU Rider — Wiring & Parts Reference

## Wiring Diagram

```
                         ESP32 Dev Board
                    ┌──────────────────────┐
                    │                      │
           3V3 ────┤ 3V3            GPIO2  ├──── (onboard LED)
           GND ────┤ GND                   │
                    │                      │
  I2C Bus ─────────┤ GPIO21 (SDA)          │
  (shared) ────────┤ GPIO22 (SCL)          │
                    │                      │
  VL53L0X #1 ──────┤ GPIO16   (XSHUT fwd) │
  VL53L0X #2 ──────┤ GPIO17   (XSHUT lft) │
  VL53L0X #3 ──────┤ GPIO18   (XSHUT rgt) │
                    │                      │
                    │              USB/5V  ├──── Power in
                    └──────────────────────┘

  I2C Bus Wiring (all 4 sensors in parallel):
  ────────────────────────────────────────────

  ESP32 GPIO21 (SDA) ──┬── BNO055 SDA
                       ├── VL53L0X #1 SDA
                       ├── VL53L0X #2 SDA
                       └── VL53L0X #3 SDA

  ESP32 GPIO22 (SCL) ──┬── BNO055 SCL
                       ├── VL53L0X #1 SCL
                       ├── VL53L0X #2 SCL
                       └── VL53L0X #3 SCL

  ESP32 3V3 ───────────┬── BNO055 VIN
                       ├── VL53L0X #1 VIN
                       ├── VL53L0X #2 VIN
                       └── VL53L0X #3 VIN

  ESP32 GND ───────────┬── BNO055 GND
                       ├── VL53L0X #1 GND
                       ├── VL53L0X #2 GND
                       └── VL53L0X #3 GND

  XSHUT wiring (one GPIO per VL53L0X):

  ESP32 GPIO16 ──── VL53L0X #1 XSHUT  (forward)
  ESP32 GPIO17 ──── VL53L0X #2 XSHUT  (left 45°)
  ESP32 GPIO18 ──── VL53L0X #3 XSHUT  (right 45°)

  Optional pullups (recommended):

  3V3 ───┬── 3.3K resistor ──── SCL bus
         └── 2.2K resistor ──── SDA bus
```

## Sensor Mounting Layout (top view of vacuum)

```
                    FORWARD
                      ▲
                      │
              ╱───────┼───────╲
           ╱    L45°  │  R45°   ╲
         ╱    ◄       │       ►   ╲
        │    VL53L0X  │  VL53L0X   │
        │  #2         │         #3 │
        │             │            │
        │      ┌──────┴──────┐     │
        │      │  VL53L0X #1 │     │
        │      │   (forward) │     │
        │      │      ▲      │     │
        │      └──────┼──────┘     │
        │             │            │
        │     ┌───────┴────────┐   │
        │     │    BNO055      │   │
        │     │  X-axis → fwd  │   │
        │     │  (away from    │   │
        │     │    motor!)     │   │
        │     └────────────────┘   │
        │                          │
        │     ┌────────────────┐   │
        │     │     ESP32      │   │
        │     │   + battery    │   │
        │     └────────────────┘   │
        │            ●             │  ← vacuum center
         ╲                        ╱
           ╲                    ╱
              ╲────────────╱

```

## Parts List

| # | Part | Search Term (AliExpress) | Alt (Amazon) | ~Cost |
|---|------|--------------------------|-------------|-------|
| 1 | ESP32 dev board | "ESP32 WROOM 32 development board" | Any ESP32 DevKit | $4-6 |
| 1 | BNO055 IMU | "GY-BNO055 9DOF sensor module" | "GY-BNO055 breakout" | $4-8 |
| 3 | VL53L0X ToF sensor | "VL53L0X GY-530 laser ranging" | "VL53L0X breakout" | $2-3 ea |
| 1 | Small LiPo battery | "3.7V 500mAh LiPo JST" | - | $3-5 |
| 1 | TP4056 charger | "TP4056 USB-C charge module" | - | $1-2 |
| 1 | Hookup wire | "Dupont jumper wire female" | - | $2 |
| 1 | Velcro strips | "Velcro adhesive strips" | Any hardware store | $2-3 |
| opt | Pullup resistors | "3.3K 2.2K resistor kit" | - | $1 |
|   | **TOTAL** | | | **~$25-35** |

## I2C Address Map (after XSHUT sequencing)

| Device | Address | Notes |
|--------|---------|-------|
| BNO055 | 0x28 | Default (0x29 if ADR pin HIGH) |
| VL53L0X #1 (fwd) | 0x30 | Reassigned at boot via XSHUT |
| VL53L0X #2 (left) | 0x31 | Reassigned at boot via XSHUT |
| VL53L0X #3 (right) | 0x32 | Reassigned at boot via XSHUT |

No address conflicts. BNO055 is at 0x28, VL53L0X sensors are
reassigned to 0x30-0x32 (away from default 0x29 to avoid any
chance of collision with BNO055 alternate address).

## First Boot Checklist

1. Flash the YAML + header to the ESP32 via USB
2. Check the ESPHome logs for:
   - "BNO055 initialized — NDOF fusion mode" (IMU working)
   - VL53L0X sensors reporting distances (not NaN)
   - I2C scan showing 4 devices found
3. In HA, the device should appear with these entities:
   - sensor.vacuum_heading
   - sensor.vacuum_accel_x / y / magnitude
   - sensor.imu_calibration
   - sensor.distance_forward / left / right
   - binary_sensor.vacuum_bump_detected
   - binary_sensor.obstacle_close_forward / left / right
4. Wave your hand in front of each VL53L0X — distance should change
5. Rotate the board — heading should change
6. Smack the board — bump_detected should trigger

## Tuning Notes

- Bump threshold (3.0 m/s²): start here, lower if bumps aren't
  detected, raise if normal vibration triggers false positives.
  The vacuum motor vibration might require raising to 4-5.

- Obstacle close threshold (0.15m = 15cm): adjust based on how
  early you want to know about obstacles. For scripted paths,
  you probably want warning at 20-30cm to have time to stop.

- If heading drifts wildly, the vacuum motor's magnetic field is
  interfering with the magnetometer. Switch to IMUPLUS mode by
  uncommenting the line in bno055_component.h. You lose absolute
  north but relative heading (turns) still works fine.

- The telemetry interval (200ms) fires HA events at 5Hz. If your
  HA instance struggles, increase to 500ms. For accurate dead
  reckoning, don't go above 500ms.
