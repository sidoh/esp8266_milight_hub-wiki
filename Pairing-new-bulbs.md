### Introduction

"Pairing" is the process of associating bulbs or other lighting devices with a given Device ID.  With stock Milight remotes and wifi gateways, this Device ID is a random hardcoded number.  With espMH, you can choose whichever Device ID you want (between 0x0000 and 0xFFFF = 65,535).

Each Device ID can be accompanied by anywhere between 1 and 8 Group IDs depending on the protocol being used (for example, RGB+CCT supports 4 Group IDs).  Each (Device ID, Group ID) pair represents a unique address which your lighting device(s) can be associated with.

It's important to note that for protocols with multiple groups, Group ID "0" is a special group.  All bulbs paired with the same Device ID will respond to commands addressed to Group ID = 0 for that Device ID.  In other words, Group ID = 0 is the "all" group for a given Device ID.

### Instructions

Choose a Device ID arbitrarily and pair your device with it following these instructions:

1. Power off the bulb or lighting device.  If it's a bulb, it's best to remove it from the socket entirely.
1. Enter your chosen Device ID into the "Device ID" field in the UI.
1. Choose a Group ID to pair with.
1. Supply power to the bulb or lighting device.
1. Within five seconds, click on the "Pair" button in the UI.
1. The light should flash on and off a few times to indicate success.