## HomeAssistant

There are two supported ways to integrate with HomeAssistant:

* **Discovery**: if [MQTT discovery](https://www.home-assistant.io/docs/mqtt/discovery/) is enabled with HomeAssistant, espMH can register named devices with HomeAssistant automatically.
* **Manual**: configure components using the [`light.mqtt`](https://home-assistant.io/components/light.mqtt/) component.

Both of these methods require that you have an MQTT broker and have configured both HomeAssistant and espMH with it.

For espMH, update your configuration as follows:

1. `mqtt_server` - your MQTT broker. can be an IP or hostname. Specify the port here (e.g., `mymqttserver.com:1234`) if it's not the default (1883).
1. `mqtt_username` / `mqtt_password` - if you have auth enabled on your MQTT server.
1. `mqtt_topic_pattern` - set this to `milight/:device_id/:device_type/:group_id`.
1. `mqtt_state_topic_pattern` - set this to `milight/states/:device_id/:device_type/:group_id`.
1. Optionally: set `mqtt_client_status_topic` to `milight/client_status`, and `simple_mqtt_client_status` to `true` (in the UI, set "Client Status Message Mode" to "Simple.")
1. Make sure you've got the appropriate `group_state_fields` selected.  The default values should work: state, brightness, computed_color, mode, color_temp, bulb_mode.

### MQTT Discovery

#### Enable MQTT discovery

For this to work, [MQTT discovery](https://www.home-assistant.io/docs/mqtt/discovery/) must be enabled.  In your HomeAssistant `configuration.yaml`, this would look like:

```yaml
mqtt:
  discovery: true
  # Note that HASS does not support a multi-level prefix 
  # (so homeassistant/discovery) will _not_ work.
  discovery_prefix: homeassistant
```

#### Configure espMH

Assuming you followed the MQTT configuration instructions in the previous step, all that you need to do is set `home_assistant_discovery_prefix` to `homeassistant`.

You're then ready to add some devices.  Select the appropriate device configurations (ID, group ID, remote type), and then add an alias for it:

![Adding an alias](https://user-images.githubusercontent.com/589893/59980513-dced5b80-95ab-11e9-9e60-3b22ef5bde9d.png)

The specified alias should then show up in HomeAssistant:

<img width="348" alt="image" src="https://user-images.githubusercontent.com/589893/59980544-36ee2100-95ac-11e9-878f-f5ec2d83f25f.png">

### Manually (HASS >= 0.84)

If you wish to integrate with HomeAssistant manually, it is easiest to use the builtin [`mqtt`](https://home-assistant.io/components/light.mqtt/) component. 

### Prerequisites

* MQTT server. HASS' embedded MQTT server probably works just fine.

### Configuration

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

### Manually, older versions of HASS (< 0.84)

Version 0.84 merged several MQTT light components into one (see [home-assistant#18227](https://github.com/home-assistant/home-assistant/pull/18227))

Use these shared parameters instead:

```yaml
    <<: &MILIGHT_PARAMS
      platform: mqtt_json
      color_temp: true
      rgb: true
      brightness: true
```