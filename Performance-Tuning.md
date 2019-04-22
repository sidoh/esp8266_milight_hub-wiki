The ESP8266 has a single CPU core.  This project tries to avoid waste where possible, but optimizes more for feature set and ease of use than packet throughput.  That said, there are some configurations to tweak if you want to get the most out of it.

## Use MQTT or v5 UDP

Other protocols (v6 UDP, REST) include receipt messages, which drastically decrease performance.  v5 UDP is the most lightweight option, but doesn't support all bulb types.

Because UDP has no retransmit or backpressure functionality, MQTT (which is over TCP) is a better choice for both performance and reliability.

## Set state flushing rate limit high

Set `state_flush_interval` as high as you can tolerate it.  This controls how often state is persisted to flash.

#### When using MQTT

* Disable updates by clearing the setting `mqtt_update_topic_pattern`.  
* If you don't need state updates at all, you can also clear `mqtt_state_topic_pattern`.  
* If you do need state, set `mqtt_state_rate_limit` as high as you can tolerate it.  This controls how often  state is flushed to the state topic.

## Disable passive listening

This will prevent the radio from rapidly being reconfigured to support scanning multiple channels at once.  It will also free CPU cycles otherwise spent in code handling listening.  Do this by setting `listen_repeats` to 0.

## Make sure repeat throttling is enabled

Repeat throttling automatically decreases the number of repeated packets down to a configurable minimum value when sending many commands in a short period of time.  This improves throughput at the cost of reliability.

Repeat throttling is off by default.  To enable it, set `packet_repeat_throttle_sensitivity` to something around 10.  Higher values cause throttling to have more effect sooner.

You may also choose to edit these other throttling-related parameters:

* `packet_repeat_throttle_threshold` should be set to no lower than the time it takes to send a command.
* `packet_repeat_minimum` should be set to 1-3.  Use 1 if you don't mind devices missing packets.

## Disable some transmit channels

In settings, under _Radio_, try disabling some of the transmit channels under the setting _nRF24 Send Channels_.  By default, all three channels are enabled.  If your bulbs respond with a single channel enabled, try that.  This will decrease the time it takes to send a single packet, which will improve throughput.

Note that this setting is available as of 1.9.0.