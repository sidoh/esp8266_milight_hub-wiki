## Integrating with HomeAssistant

The easiest and probably best way to integrate with HomeAssistant is to use the builtin [`mqtt_json`](https://home-assistant.io/components/light.mqtt_json/) component. The advantages of this approach are:

* Very minimal HASS config.
* Super low latency because all communication is happening over pre-established TCP connections. You can expect lights to react to changes in HASS within 10-100ms.

### Prerequisites

* MQTT server. HASS' embedded MQTT server probably works just fine.

### Configuration

#### esp8266_milight_hub

You will need to set the following properties in the UI:

* mqtt_server - can be an IP or hostname. Specify the port here (e.g., `mymqttserver.com:1234`) if it's not the default (1883).
* mqtt_username / mqtt_password - if you have auth enabled on your MQTT server.
* mqtt_topic_pattern - set this to `milight/:device_id/:device_type/:group_id`.
* mqtt_update_topic_pattern - set this to `milight/updates/:device_id/:device_type/:group_id`.

#### HASS

You'll need a HASS config for each device ID / group you want to control. Fortunately we can leverage YAML anchors to avoid repeating ourselves too much.

Note that device IDs should be specified in hex (e.g., `0xF` instead of `15`).

```yaml
light: 
  - name: "Milight Lamp #1"
    # The command topic is derived by substituting real values for the
    # tokens in the setting mqtt_topic_pattern (e.g., "rgb_cct" for
    # ":device_type"):
    #                        ______________ Device ID
    #                       |      ________ Device Type
    #                       |     |     ___ Group ID
    #                       |     |    |
    #                       v     v    v
    command_topic: milight/0x1/rgb_cct/1

    # This is the same structure as above, but for the setting
    # mqtt_update_topic_pattern. esp8266_milight_hub sends state updates
    # to this topic.
    state_topic: milight/updates/0x1/rgb_cct/1

    # Use a YAML anchor for common settings so we can just reference
    # them for other lights.
    <<: &MILIGHT_PARAMS
      platform: mqtt_json
      color_temp: true
      rgb: true
      brightness: true

  - name: "Milight Lamp #2"
    # Only difference here is we're using the group ID "2" instead of "1".
    command_topic: milight/0x1/rgb_cct/2
    state_topic: milight/updates/0x1/rgb_cct/2

    # Reference YAML anchor defined above.
    <<: *MILIGHT_PARAMS

  - name: "Milight RGBW Bulbs"
    # Note here that we're using the "rgbw" bulb type instead of "rgb_cct".
    command_topic: milight/0x1/rgbw/1
    state_topic: milight/updates/0x1/rgbw/1
    <<: *MILIGHT_PARAMS
```