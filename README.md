# Two-Stage Mold Risk Control with Optional Dehumidifier and Cycling

## Overview
This Home Assistant blueprint provides an automated two-stage mold risk control system. The system activates fans and dehumidifiers based on a mold risk index. The automation ensures efficient moisture control by turning devices on and off based on set thresholds, delays, and nighttime preferences.

## Features
- **Two-Stage Activation:**
  - Fans turn on immediately when the mold risk exceeds a defined threshold.
  - Dehumidifiers activate if the mold risk remains high for a specified delay.
- **Two-Stage Deactivation:**
  - Dehumidifiers turn off first when risk drops.
  - Fans turn off after an additional delay.
- **Nighttime Mode:**
  - Only dehumidifiers are deactivated if the risk is low.
- **Dehumidifier Cycling (Optional):**
  - If enabled, after the dehumidifier turns off, the automation checks the risk after a delay and reactivates if necessary.

## Requirements
- A mold risk index sensor (`sensor.mold_risk_index`).
- At least one of:
  - Fan switches (`switch.fan_x`)
  - Dehumidifier switches (`switch.dehumidifier_x`)

## Input Parameters
| Parameter | Description | Default |
|-----------|-------------|---------|
| `sensor_mold_risk_index` | The sensor providing the mold risk index. | Required |
| `fan_switches` | Fan switch(es) controlling the fan(s). | Optional |
| `dehumidifier_switches` | Dehumidifier switch(es). | Optional |
| `fan_or_dehumidifier_required` | Ensures that at least one of fan or dehumidifier is required. | True |
| `fan_on_threshold` | Mold risk index at which the fan(s) turn on. | 1.0 |
| `dehumidifier_on_threshold` | Mold risk index at which the dehumidifier(s) turn on. | 1.5 |
| `off_threshold` | Mold risk index below which devices will turn off. | 1.0 |
| `nighttime_off_threshold` | Mold risk index below which the dehumidifier(s) turn off at night. | 2.0 |
| `delay_dehumidifier_on` | Delay before turning on the dehumidifier after reaching threshold. | 10 min |
| `delay_dehumidifier_off` | Delay before turning off the dehumidifier after risk drops. | 5 min |
| `delay_fan_off` | Delay after turning off the dehumidifier before turning off the fan. | 3 min |
| `delay_night` | Delay before turning off dehumidifiers at night. | 5 min |
| `nighttime_start` | Start time of nighttime mode. | 20:00 |
| `nighttime_end` | End time of nighttime mode. | 08:30 |
| `enable_dehumidifier_cycle` | Enable dehumidifier cycling to check if risk remains high after a delay. | False |
| `cycle_dehumidifier_on_delay` | Delay before turning dehumidifier back on during cycling. | 5 min |

## Triggers
- Time-based checks.
- Mold risk threshold crossings.
- Nighttime start and end events.

## Usage
1. Add this blueprint to your Home Assistant automation.
2. Configure the required sensors and switches.
3. Adjust thresholds and delays to suit your environment.
4. Enable nighttime mode and cycling if desired.
5. Save and activate the automation.

## Notes
- Ensure that the mold risk index sensor provides accurate readings.
- Adjust thresholds based on local climate and humidity conditions.

This blueprint helps maintain an optimal indoor environment by preventing mold growth through automated moisture control. 🚀

