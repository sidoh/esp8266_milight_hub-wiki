## Integrating with HomeAssistant (>= 0.84)

The easiest and probably best way to integrate with HomeAssistant is to use the builtin [`mqtt`](https://home-assistant.io/components/light.mqtt/) component. The advantages of this approach are:

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
* mqtt_state_topic_pattern - set this to `milight/states/:device_id/:device_type/:group_id`.
* Optionally: set mqtt_client_status_topic to `milight/client_status`, and `simple_mqtt_client_status` to `true` (in the UI, set "Client Status Message Mode" to "Simple.")

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
    # mqtt_state_topic_pattern. esp8266_milight_hub sends state updates
    # to this topic.
    state_topic: milight/states/0x1/rgb_cct/1

    # Use a YAML anchor for common settings so we can just reference
    # them for other lights.
    <<: &MILIGHT_PARAMS
      platform: mqtt
      schema: json
      color_temp: true
      rgb: true
      brightness: true
      
      # If you enabled a client status topic, you can set these to show when
      # the hub is disconnected in the HASS UI.
      availability_topic: milight/client_status
      payload_available: 'connected'
      payload_not_available: 'disconnected'

  - name: "Milight Lamp #2"
    # Only difference here is we're using the group ID "2" instead of "1".
    command_topic: milight/0x1/rgb_cct/2
    state_topic: milight/states/0x1/rgb_cct/2

    # Reference YAML anchor defined above.
    <<: *MILIGHT_PARAMS

  - name: "Milight RGBW Bulbs"
    # Note here that we're using the "rgbw" bulb type instead of "rgb_cct".
    command_topic: milight/0x1/rgbw/1
    state_topic: milight/states/0x1/rgbw/1
    <<: *MILIGHT_PARAMS
```

#### Effects

You can add support for effects using the `effects_list` parameter.  For example:

```yaml
bedroom_light:
  platform: mqtt
  schema: json
  color_temp: true
  rgb: true
  brightness: true
  effect: true
  effect_list:
    - white_mode
    - night_mode
    # Most milight bulbs have effects 0-8.
    - 0
    - 1
    # ... and so on
```

## For older versions of HomeAssistant (< 0.84)

Version 0.84 merged several MQTT light components into one (see [home-assistant#18227](https://github.com/home-assistant/home-assistant/pull/18227))

Use these shared parameters instead:

```yaml
    <<: &MILIGHT_PARAMS
      platform: mqtt_json
      color_temp: true
      rgb: true
      brightness: true
```