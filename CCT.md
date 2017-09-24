## Structure

* Unidirectional. Bulbs never send packets.
* Always 7 bytes.

The packets are structured as follows:


| Field           | Length  | Notes                                                                                     |
|-----------------|---------|-------------------------------------------------------------------------------------------|
| 1 | 1 byte | Always 0x5A |
| 2 | 2 bytes | Device ID |
| 3 | 1 byte | Group ID |
| 4 | 1 byte | Command (listed in code [here](https://github.com/sidoh/esp8266_milight_hub/blob/1.5.0/lib/MiLight/CctPacketFormatter.h#L9)) |
| 5 | 1 byte | Sequence num (unused?) |
| 6 | 1 byte | Sequence num (usually ≠ to field #5) |

## Notes

* Brightness and temperature can only be controlled by increase and decrease commands.