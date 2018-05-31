## Radio configs

"radio configs" consist of the following pieces of information:

1. Channel set: every remote we've encountered listens on three distinct 2.4 GHz channels
2. Syncwords: a 4 byte configuration of radio chips used by milight.  More detail on [LT8900 datasheet](https://wenku.baidu.com/view/12a4cdd381c758f5f61f67ae.html)
3. Packet size: number of bytes in the payload

[This spreadsheet](https://docs.google.com/spreadsheets/d/1eMNn5HmMzNc0LgRBQlcnDYeE7KKCKBPMd15dMbIpqfM/edit#gid=0) lists the radio configs for all Milight remote types supported by the iBox 2 wifi gateway.  These values were sniffed from the SPI bus.