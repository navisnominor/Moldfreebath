# README:
# This blueprint requires the mold risk index sensor provided by the mold_risk_index integration.
# See: https://github.com/Strixx76/mold_risk_index
#
# The blueprint implements a two-stage control strategy based on the mold risk index:
#  - Fan(s) turn on immediately when the risk exceeds the Fan On Threshold.
#  - If dehumidifier switches are provided and the risk remains above the Dehumidifier On Threshold
#    after a delay, the dehumidifier(s) are turned on.
#  - When the risk drops below the Off Threshold, the dehumidifier(s) are turned off first,
#    then after an additional delay the fan(s) are turned off.
#  - During a specified nighttime period, if the risk remains below the Nighttime Off Threshold,
#    only the dehumidifier(s) are turned off (leaving the fan(s) running).
#  - Optionally, if dehumidifier cycling is enabled, after turning off the dehumidifier(s)
#    the automation will wait a configurable delay and then turn them back on if the risk rises again.
#
blueprint:
  name: Two-Stage Mold Risk Control with Optional Dehumidifier, Cycling, and Staged Off (Risk Index Only)
  description: >
    Two-stage control based solely on a mold risk index from the mold_risk_index integration.
    The fan(s) turn on immediately when the risk exceeds the Fan On Threshold.
    If dehumidifier switches are provided and the risk exceeds the Dehumidifier On Threshold for a set delay,
    the dehumidifier(s) will be turned on.
    When the risk drops, the dehumidifier(s) are turned off first, then after an additional delay the fan(s) are turned off.
    Optionally, if dehumidifier cycling is enabled, the dehumidifier(s) will be turned back on after a delay if the risk rises again.
    During the defined nighttime period, only the dehumidifier(s) are turned off if the risk is low.
  domain: automation
  input:
    sensor_mold_risk_index:
      name: Mold Risk Index Sensor
      description: The sensor that provides the mold risk index value.
      selector:
        entity:
          domain: sensor
    fan_switches:
      name: Fan Switches
      description: The switch(es) controlling the fan(s) (e.g., Shelly devices).
      selector:
        entity:
          domain: switch
          multiple: true
    dehumidifier_switches:
      name: Dehumidifier Switches (Optional)
      description: The switch(es) controlling the dehumidifier(s). Leave empty if not used.
      selector:
        entity:
          domain: switch
          multiple: true
      default: []
    fan_on_threshold:
      name: Fan On Threshold
      description: Risk threshold at which the fan(s) turn on.
      default: 1.0
      selector:
        number:
          min: 0
          max: 10
          unit_of_measurement: ""
    dehumidifier_on_threshold:
      name: Dehumidifier On Threshold
      description: Risk threshold at which the dehumidifier(s) turn on.
      default: 1.5
      selector:
        number:
          min: 0
          max: 10
          unit_of_measurement: ""
    off_threshold:
      name: Off Threshold
      description: Risk threshold below which devices will eventually be turned off.
      default: 1.0
      selector:
        number:
          min: 0
          max: 10
          unit_of_measurement: ""
    nighttime_off_threshold:
      name: Nighttime Off Threshold
