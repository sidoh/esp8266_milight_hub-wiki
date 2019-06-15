## Install the plugin

Goto your Domoticz Server/setup

```
cd ~/domoticz/plugin/
git clone https://github.com/poudenes/espmilighthub-domoticz ESPMilight
systemctl restart domoticz.service
```

## Configuring Domoticz

Goto your Domoticz Frontend page

setup > hardware > scroll down in Type and select: ESP8266 Milight Hub
* Give this hardware a name
* Fill in all the info that is asked
* For MQTT settings, enter
  * mqtt_topic_pattern: `milight/:device_id/:device_type/:group_id`
  * mqtt_state_topic_pattern: `milight/states/:hex_device_id/:device_type/:group_id`
  * mqtt_update_topic_pattern: `milight/updates/:hex_device_id/:device_type/:group_id`
* Hit Add

## Configuring esp8266_milight_hub

* Go to your Milight Hub then Settings > MQTT.  Enter these settings to match the ones above:
  * MQTT topic pattern: `milight/:device_id/:device_type/:group_id`
  * MQTT state topic pattern: `milight/states/:hex_device_id/:device_type/:group_id`
  * MQTT update Pattern: `milight/updates/:hex_device_id/:device_type/:group_id`
* Hit Submit

## Getting bulbs to show up in Domoticz

* Now go back to Domoticz and then Setup > Settings 
* Below hit Allow for 5 minutes 
* Hit "Apply Settings" upper right corner

After this...

* Go back to Milight Hub and switch on/off every bulb you have configured.
* You will see that every bulb will add as device inside Domoticz

## Further Reading

**Plugin Repo**:

https://github.com/galinette2000/espmilighthub-domoticz

Here you find some info when you use dzVents System inside domoticz.
Some set options are working some not:
http://www.domoticz.com/forum/viewtopic.php?f=65&t=27771&start=20#p213498

## Thanks

* Big thank you to [@galinette2000](https://github.com/galinette2000) for writing the Domoticz plugin!
* Thanks you to [@poudenes](https://github.com/poudenes) for writing these instructions!  (Reference: [#452](https://github.com/sidoh/esp8266_milight_hub/issues/452))