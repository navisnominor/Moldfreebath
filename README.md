# README:
# This blueprint requires the mold risk index sensor provided by the mold_risk_index integration.
# See: https://github.com/Strixx76/mold_risk_index
#
# The blueprint implements a two-stage control strategy:
#  - Fan(s) turn on immediately when the mold risk index exceeds the configured fan threshold.
#  - If a dehumidifier is configured and the risk remains above the dehumidifier threshold after a delay,
#    the dehumidifier(s) are turned on.
#  - When the risk drops below the off threshold, the dehumidifier(s) are turned off first, then after an additional
#    delay the fan(s) are turned off.
#  - During the defined nighttime period, if the risk remains below a separate nighttime threshold,
#    only the dehumidifier(s) will be turned off, while the fan(s) continue running.
#  - Optionally, if dehumidifier cycling is enabled, the blueprint will recheck the risk after a configurable delay
#    and turn the dehumidifier(s) back on if the risk rises above the dehumidifier threshold.
#
blueprint:
  name: Two-Stage Mold Risk Control with Optional Dehumidifier, Cycling, and Staged Off (Risk Index Only)
  description: >
    Two-stage control based solely on a mold risk index from the mold_risk_index integration.
    The fan(s) turn on immediately when the risk exceeds the fan threshold.
    If a dehumidifier is configured and the risk exceeds a higher threshold for a set delay,
    the dehumidifier(s) will be turned on.
    When the risk drops, the dehumidifier(s) are turned off first, then after an additional delay the fan(s) are turned off.
    Optionally, if dehumidifier cycling is enabled, the dehumidifier will be turned back on after a delay if the risk rises.
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
      optional: true
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
      description: Risk threshold below which the dehumidifier(s) are turned off at night.
      default: 2.0
      selector:
        number:
          min: 0
          max: 10
          unit_of_measurement: ""
    delay_dehumidifier_on:
      name: Dehumidifier On Delay
      description: Delay to confirm risk remains above the dehumidifier threshold before turning it on.
      default: "00:10:00"
      selector:
        time:
    delay_dehumidifier_off:
      name: Dehumidifier Off Delay
      description: Delay to confirm risk remains below the off threshold before turning off dehumidifier(s).
      default: "00:05:00"
      selector:
        time:
    delay_fan_off:
      name: Fan Off Delay
      description: Delay after turning off dehumidifier(s) before turning off fan(s).
      default: "00:03:00"
      selector:
        time:
    delay_night:
      name: Nighttime Delay
      description: Delay to confirm risk remains below the nighttime off threshold before turning off dehumidifier(s) at night.
      default: "00:05:00"
      selector:
        time:
    nighttime_start:
      name: Nighttime Start
      description: Start time of the nighttime period.
      default: "20:00:00"
      selector:
        time:
    nighttime_end:
      name: Nighttime End
      description: End time of the nighttime period.
      default: "08:30:00"
      selector:
        time:
    enable_dehumidifier_cycle:
      name: Enable Dehumidifier Cycling
      description: >
        If enabled, after the dehumidifier is turned off due to low risk,
        the automation will wait a configurable delay and then, if the risk is high again,
        turn the dehumidifier back on.
      default: false
      selector:
        boolean: {}
    cycle_dehumidifier_on_delay:
      name: Dehumidifier Cycle On Delay
      description: Delay before turning the dehumidifier back on after it has been turned off by cycling.
      default: "00:05:00"
      selector:
        time:
trigger:
  - platform: time_pattern
    minutes: "0"
  - platform: time
    at: !input nighttime_end
  - platform: time
    at: !input nighttime_start
  - platform: numeric_state
    entity_id: !input sensor_mold_risk_index
    above: !input fan_on_threshold
  - platform: numeric_state
    entity_id: !input sensor_mold_risk_index
    below: !input off_threshold
  - platform: numeric_state
    entity_id: !input sensor_mold_risk_index
    above: !input dehumidifier_on_threshold
  - platform: numeric_state
    entity_id: !input sensor_mold_risk_index
    below: !input nighttime_off_threshold
condition: []
action:
  # Stage 1: Turn on the fan(s) immediately if risk exceeds the fan threshold.
  - choose:
      - conditions:
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            above: !input fan_on_threshold
          - condition: state
            entity_id: !input fan_switches
            state: "off"
        sequence:
          - service: switch.turn_on
            target:
              entity_id: !input fan_switches

  # Stage 2: (Optional) Turn on the dehumidifier(s) if risk exceeds the dehumidifier threshold.
  - choose:
      - conditions:
          - condition: template
            value_template: >
              {{ !input.dehumidifier_switches | length > 0 }}
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            above: !input dehumidifier_on_threshold
          - condition: state
            entity_id: !input dehumidifier_switches
            state: "off"
        sequence:
          - delay: !input delay_dehumidifier_on
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            above: !input dehumidifier_on_threshold
          - service: switch.turn_on
            target:
              entity_id: !input dehumidifier_switches

  # Two-Stage Off Process:
  # Stage 1: Turn off dehumidifier(s) if risk remains below the off threshold.
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ !input.dehumidifier_switches | length > 0 }}"
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            below: !input off_threshold
        sequence:
          - delay: !input delay_dehumidifier_off
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            below: !input off_threshold
          - service: switch.turn_off
            target:
              entity_id: !input dehumidifier_switches

  # Stage 2: After dehumidifier(s) are off, turn off the fan(s) after an additional delay.
  - choose:
      - conditions:
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            below: !input off_threshold
          - condition: state
            entity_id: !input fan_switches
            state: "on"
          - condition: template
            value_template: >
              {% if (!input.dehumidifier_switches | length) > 0 %}
                {{ states(!input.dehumidifier_switches[0]) == "off" }}
              {% else %}
                true
              {% endif %}
        sequence:
          - delay: !input delay_fan_off
          - service: switch.turn_off
            target:
              entity_id: !input fan_switches

  # Nighttime Special: During nighttime, only turn off the dehumidifier(s) if risk remains below the nighttime off threshold.
  - choose:
      - conditions:
          - condition: time
            after: !input nighttime_start
            before: !input nighttime_end
          - condition: template
            value_template: "{{ !input.dehumidifier_switches | length > 0 }}"
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            below: !input nighttime_off_threshold
        sequence:
          - delay: !input delay_night
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            below: !input nighttime_off_threshold
          - service: switch.turn_off
            target:
              entity_id: !input dehumidifier_switches

  # Optional Cycle: If dehumidifier cycling is enabled, after the dehumidifier has been turned off,
  # wait for the configured cycle on delay and then, if the risk is again high, turn the dehumidifier back on.
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ not input.enable_dehumidifier_cycle }}"
        sequence: []
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ input.enable_dehumidifier_cycle }}"
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            above: !input dehumidifier_on_threshold
        sequence:
          - delay: !input cycle_dehumidifier_on_delay
          - condition: numeric_state
            entity_id: !input sensor_mold_risk_index
            above: !input dehumidifier_on_threshold
          - service: switch.turn_on
            target:
              entity_id: !input dehumidifier_switches
mode: restart
