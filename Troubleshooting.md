This guide will walk you through common ways in which an ESP8266 running esp8266_milight_hub can fail or perform poorly.

## Hardware

A dead ESP8266 is usually pretty obvious -- if you can't flash the firmware or it won't boot, you should try replacing the module.

A dead or miswired radio module is harder to spot.  Some suggestions:

#### Wiring

* Double-check wiring guide.  [This blog post](http://blog.christophermullins.com/2017/02/11/milight-wifi-gateway-emulator-on-an-esp8266/) details the hookup.
* Use pin and headers or soldered joints instead of jumper cables wherever possible.  They're far more reliable.

Bad wiring can cause crashes or hangs when sending commands.

#### Bad radio module

Knockoff NRF24L01s are [very common, and have relatively poor build quality](https://forum.mysensors.org/topic/1153/we-are-mostly-using-fake-nrf24l01-s-but-worse-fakes-are-emerging).

If you can flash the firmware and load the webpage, but not send or receive packets, and you're sure the nRF is wired correctly, it's likely you have a faulty module.  It's possible that you'll be able to receive, but not send packets.

Several users have reported that switching to a new radio module solves their problems.

#### Mislabeled pins

Sometimes ESP8266 dev boards have mislabeled pins.  Usually the pinouts you find online will have the correct order.

#### Range

Try sending commands when the module is right next to the device you're trying to control.  Radio modules have a fair amount of variation in range capabilities.  If the one you're using sucks, try another one.

## Make sends work

If you're sure hardware is working, but your devices aren't responding, the next step is to make sure that the ESP8266 is actually sending packets.  To do this:

* Enable sniffing in the UI.  Packets should show up.  If they do, then the ESP at least thinks it sent a packet.  However, sent packets will show up regardless of whether the radio actually processed them.
* Set up another ESP8266 with an nRF and flash esp8266_milight_hub on it as well.  Enable sniffing on that second ESP while sending commands with the first.

## If all else fails...

Please [open an issue](https://github.com/sidoh/esp8266_milight_hub/issues/new) and include the following information:

* Mention that you've run through this troubleshooting guide.
* Which ESP8266 board you're using (nodemcu, esp12f, etc.)
* Which version of the firmware you're using.
* Whether you built the firmware yourself, or used a pre-compiled binary.
* The model number of the device you're trying to control (see [this product catalog](https://github.com/sidoh/esp8266_milight_hub/files/1379131/MiLight-ProductCatalog-2017.pdf)).
* Which radio type you're using (RGBW, RGB+CCT, etc.)
* The output of http://milight-hub.local/settings and http://milight-hub.local/about. You might need to substitute `milight-hub.local` for the IP of your ESP, and **Make sure to redact any passwords**!