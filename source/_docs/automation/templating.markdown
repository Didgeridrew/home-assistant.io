---
title: "Automation Templating"
description: "Advanced automation documentation using templating."
---

Automations support [templating](/docs/configuration/templating/) in the same way as scripts do. In addition to the [Home Assistant template extensions](/docs/configuration/templating/#home-assistant-template-extensions) available to scripts, the `trigger` template variable is available.

<div class='note'>
  
  Be aware that if you reference a `trigger` state object in templates of an automation' `action` or `condition` sections, attempting to test that automation by calling the `automation.trigger` service or by clicking EXECUTE in the More Info box for the automation will not work. This is because the trigger state object doesn't exist in those contexts. One way to test automations like these is to manually check that the templates work as expected by pasting them in {% my developer_template title="Developer Tools > Template" %} together with your trigger's definition like:

{%raw%}

```yaml
{% set trigger={'to_state':{'state': 'heat'}} %}
{% set option = trigger.to_state.state %}
{{ 'on' if option == 'heat' else 'off' }}
```

{%endraw%}
  
</div>

## Available Trigger Data

The following tables show the available trigger data per platform.

### Event

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `event`.
| `trigger.event` | Event object that matched.
| `trigger.event.event_type` | Event type.
| `trigger.event.data` | Optional event data.

```yaml
trigger:
- platform: event
  event_type: zha_event
  event_data:
    device_ieee: 00:12:4b:11:21:35:ae:22

action:
- choose:
  - conditions:
    - '{{ trigger.event.data.command == "toggle" }}'
    sequence: 
      - service: light.toggle
        target:
          entity_id: light.basement
  - conditions:
    - '{{ trigger.event.data.command == "on" }}'
    sequence:
      - service: notify.alexa_media
          data:
            data:
              type: tts
            target: media_player.kitchen
            message: "Dinner is ready"
  - conditions:
    - '{{ trigger.event.data.command == "off" }}'
    sequence:
      - service: script.nighttime_shutdown
```

### MQTT

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `mqtt`.
| `trigger.topic` | Topic that received payload.
| `trigger.payload` | Payload.
| `trigger.payload_json` | Dictonary of the JSON parsed payload.
| `trigger.qos` | QOS of payload.

```yaml

  trigger:
    - platform: mqtt
      topic: "/notify/+"
  action:
    service: >
      notify.{{ trigger.topic.split('/')[-1] }}
    data:
      message: "{{ trigger.payload }}"
```

### Numeric State

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `numeric_state`
| `trigger.entity_id` | Entity ID that we observe.
| `trigger.below` | The below threshold, if any.
| `trigger.above` | The above threshold, if any.
| `trigger.from_state` | The previous [state object] of the entity.
| `trigger.to_state` | The new [state object] that triggered trigger.
| `trigger.for` | Timedelta object how long state has met above/below criteria, if any.

### State

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `state`
| `trigger.entity_id` | Entity ID that we observe.
| `trigger.from_state` | The previous [state object] of the entity.
| `trigger.to_state` | The new [state object] that triggered trigger.
| `trigger.for` | Timedelta object how long state has been to state, if any.

```yaml
automation:
  trigger:
    - platform: state
      entity_id: device_tracker.paulus
  action:
    - service: notify.notify
      data:
        message: >
          Paulus just changed from {{ trigger.from_state.state }}
          to {{ trigger.to_state.state }}
 
automation_2:
  trigger:
    # Multiple entities for which you want to perform the same action.
    - platform: state
      entity_id:
        - light.bedroom_closet
        - light.kiddos_closet
        - light.linen_closet
      to: "on"
      # Trigger when someone leaves one of those lights on for 10 minutes.
      for: "00:10:00"
  action:
    - service: light.turn_off
      target:
        # Turn off whichever entity triggered the automation.
        entity_id: "{{ trigger.entity_id }}"
```

### Sun

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `sun`
| `trigger.event` | The event that just happened: `sunset` or `sunrise`.
| `trigger.offset` | Timedelta object with offset to the event, if any.

### Template

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `template`
| `trigger.entity_id` | Entity ID that caused change.
| `trigger.from_state` | Previous [state object] of entity that caused change.
| `trigger.to_state` | New [state object] of entity that caused template to change.
| `trigger.for` | Timedelta object how long state has been to state, if any.

### Time

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `time`
| `trigger.now` | DateTime object that triggered the time trigger.

```yaml

trigger:
  - platform: time
    at: '09:00:00'
  - platform: sun
    event: sunset
    offset: '-01:30:00'
condition: []
action:
  - choose:
      - conditions:
          - condition: template
            value_template: '{{(trigger.now.time()|string)[:5] == ''09:00'' }}'
        sequence:
          - service: light.turn_off
            target:
              entity_id: light.living_room_lamps
      - conditions:
          - condition: template
            value_template: '{{ trigger.offset.total_seconds()/60 == -90 }}'
        sequence:
          - service: light.turn_on
            data:
              color_temp: 350
              brightness_pct: 100
            target:
              entity_id: light.living_room_lamps
```

### Time Pattern

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `time_pattern`
| `trigger.now` | DateTime object that triggered the time_pattern trigger.

### Webhook

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `webhook`
| `trigger.webhook_id` | The webhook ID that was triggered.
| `trigger.json` | The JSON data of the request (if it had a JSON content type).
| `trigger.data` | The form data of the request (if it had a form data content type).
| `trigger.query` | The URL query parameters of the request (if provided).

### Zone

| Template variable | Data |
| ---- | ---- |
| `trigger.platform` | Hardcoded: `zone`
| `trigger.entity_id` | Entity ID that we are observing.
| `trigger.from_state` | Previous [state object] of the entity.
| `trigger.to_state` | New [state object] of the entity.
| `trigger.zone` | State object of zone
| `trigger.event` | Event that trigger observed: `enter` or `leave`.


[state object]: /docs/configuration/state_object/
