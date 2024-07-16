---
title: "Smart Whole House Fan"
date: 2024-07-15T12:00:00-07:00
categories:
  - house
tags:
  - home automation
  - zwave
  - home assistant
draft: false
description: Install a whole house fan and make it smart using Home Assistant and ZWave.
---

We added a whole house fan to our house to bring in fresh air and aid in cooling down the house in the evening. The fan is fully controllable with Home Assistant using a couple ZWave relays.

## System Design

We went with a slightly oversized unit than our square footage and climate zone need. The unit we chose was the Quiet Cool QC ES-4700 which provides two speeds, high at 4,195 CFM and low at 2,304 CFM. The fan has two control lines that determine speed. Normally, these lines are hooked up to Quiet Cool's own smart fan control system to control the speed or a timer switch and control switch.

{{< img src="images/official_wiring.webp" alt="Quiet Cool Wiring Diagram" >}}

I prefer to keep things controlled locally and with ZWave or Zigbee if possible. I already had a ZWave mutli relay in the attic for controllling an exterior light and I had the necessary relays left to control the system. This also means I didn't need to run additional wires anywhere in my house. I only needed to run two 12-2 wires from the junction box housing the relay to the fan.

{{< img src="images/wiring.webp" alt="Wiring Diagram" >}}

### Parts list

- [Quiet Cool QC ES-4700](https://norell.link/qcfan)
- [Zooz ZEN16 Multi Relay](https://norell.link/zen16)
- 12-2 wire

### Tools required

- Impact Driver
- Screwdriver
- Wire cutter and strippers
- Drywall knife

### Additional tools

These are tools that I needed to cut out the floor and add some framing to my attic to mount the fan.

- Reciprocating saw
- Jigsaw
- Drywall Rotary Cutter

## Installation

Installing a whole home fan is exactly as easy as the manufacturer claims in their marketing.

Here are the steps.

1. Identify a place to put the fan
2. Cut a hole in the ceiling
3. Install the damper in the ceiling
4. Hook the tube up to the damper
5. Hang the fan from a rafter
6. Hook the tube up to the fan
7. Wire the fan up

In total it took 4 hours to install from opening up the box to turning on the fan. My install was a bit longer due to needing to cut through an unused attic floor and building something to hang the fan from.

## Home Assistant

I modified posts from [this thread](https://norell.link/ItGwC) in the Home Assistant community. The main changes were around having two speeds and having a separate script to handle the speed changes. I also renamed the entities for each relay to reflect what they are doing high and low speed.

***`configuration.yaml`***

```yaml
fan:
  - platform: template
    fans:
      house_fan:
        friendly_name: "House Fan"
        value_template: >
          {% if is_state('switch.house_fan_high', 'on') or is_state('switch.house_fan_low', 'on') %}
            on
          {% else %}
            off
          {% endif %}
        percentage_template: >
          {% if is_state('switch.house_fan_high', 'on') %}
            100
          {% elif is_state('switch.house_fan_low', 'on') %}
            50
          {% else %}
            0
          {% endif %}
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.house_fan_low
        turn_off:
          service: switch.turn_off
          data:
            entity_id:
              - switch.house_fan_high
              - switch.house_fan_low
        set_percentage:
          service: script.set_house_fan_speed
          data:
            percentage: "{{ percentage }}"
        speed_count: 2
```

***`scripts.yaml`***

```yaml
set_house_fan_speed:
  alias: Set House Fan Speed
  sequence:
    - service: switch.turn_off
      data:
        entity_id:
          - switch.house_fan_high
          - switch.house_fan_low
    - delay: "00:00:01"
    - choose:
        - conditions:
            - condition: template
              value_template: "{{ percentage == 100 }}"
          sequence:
            - service: switch.turn_on
              data:
                entity_id: switch.house_fan_high
        - conditions:
            - condition: template
              value_template: "{{ percentage == 50 }}"
          sequence:
            - service: switch.turn_on
              data:
                entity_id: switch.house_fan_low
```

### Control

With the fan entity created, a tile can be added that has the fan speeds embedded.

```yaml
type: tile
entity: fan.house_fan
features:
  - type: fan-speed
```

{{< img src="images/house_fan.webp" alt="House fan control entity in Home Assistant" >}}
