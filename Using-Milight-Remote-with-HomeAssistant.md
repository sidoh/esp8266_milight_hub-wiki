## Purpose

Milight makes 2.4GHz remotes to control their bulbs directly:

[[ http://imgur.com/AQKkJSM.png | height = 200px ]]

These work quite well, but there's not a nice way to integrate them with a home automation platform like HomeAssistant (HASS). Fortunately, esp8266_milight_hub (ESPMH) makes it fairly easy to do just that:

[[[ http://i.imgur.com/vK0FM3M.gif | height = 200px ]]](http://i.imgur.com/vK0FM3M.gifv)
[[[ http://i.imgur.com/sCOOIMm.gif | height = 200px ]]](http://i.imgur.com/sCOOIMm.gifv)

## High-level setup

The goal here is to use a Milight remote to control Milight bulbs, but have the state kept in sync with HASS. The remote can just as easily be used to control anything else managed by HASS.

MQTT is used as the communication layer between HASS and the ESP. The following steps are in chronological order of occurrence:

1. A button is pressed on the Milight remote.
2. The ESP captures the packet, and pushes a corresponding message to a configurable MQTT topic.
3. HASS receives the message, and forwards it to the MQTT topic used by the ESP to control Milight bulbs.

## Setup instructions

#### Make sure passive listening is enabled

Set the config setting `listen_repeats` to 3. If you have problems with reliability, you can increase this value.

#### Configure MQTT on the ESP

Specify `mqtt_server`, and `mqtt_username`/`mqtt_password` if your MQTT server is password protected. If your server runs on a non-default port, put in the server field (e.g., `mymqtt.com:1234`).

Configuring which MQTT topics are used is a little tricky, but not bad when you get the feel for it. There are two categories of topics:

1. Topics for messages to the ESP. Called "command" topics.
2. Topics for the ESP to publish state changes to. Called "update" topics.

Set the command topic pattern (`mqtt_topic_pattern`) to something like `milight/commands/:device_id/:device_type/:group_id`. You would then publish a message to the topic `milight/commands/1/rgb_cct/1` in order to control the `rgb_cct` bulbs paired with device ID `1` and group ID `1`.

If you define an update topic pattern (`mqtt_update_topic_pattern`) `milight/updates/:device_id/:device_type/:group_id`, both state changes sent by ESPMH, and captured packets sent from other devices will cause state updates to be published to the appropriate topic.

It's easiest to verify things are working by fiddling around in a REPL (I'm using `irb` here):

```ruby
irb(main):019:0> client = MQTT::Client.new('mymqttserver',username:'username',password:'hunter2')
irb(main):020:0> client.connect
irb(main):021:0> client.subscribe('milight/#')
irb(main):022:0> client.publish('milight/commands/0x118D/rgb_cct/3', '{"state":"ON"}')
irb(main):023:0> while true; puts client.get.inspect; end
["milight/0x118D/rgb_cct/3", "{\"state\":\"ON\"}"]
["milight/updates/0x118D/rgb_cct/3", "{\"device_id\":4493,\"group_id\":3,\"device_type\":\"rgb_cct\",\"state\":\"ON\"}"]
```

#### Configure HASS

Of course since we're using MQTT, you'll want to [add your broker to your HASS config](https://home-assistant.io/components/mqtt/).

An easy way to get HASS to redirect MQTT traffic triggered by use of a remote to Milight bulbs is to forward messages received on the remote's update topic to the bulb's command topic. This can be done with an automation:

```yaml
automation:
- alias: MiLight Forwarder
  initial_state: True
  trigger:
    platform: mqtt
    topic: milight/updates/0x1111/rgb_cct/+
  action:
    - service: mqtt.publish
      data_template:
        topic: "milight/commands/0x2222/rgb_cct/{{ trigger.topic.split('/')[4] }}"
        payload_template: >
          {{ trigger.payload }}
```

Note the use of `+` in the updates topic. This is a [single-level wildcard](https://mosquitto.org/man/mqtt-7.html), and matches any number of non-`/` characters.

The command topic template extracts the value of the group ID from the update topic and forwards it to the matching command topic.