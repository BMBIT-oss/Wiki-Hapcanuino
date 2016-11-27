You can use [Hapcan programmer](http://hapcan.com/software/hap/) software to set some parameters of Hapcanuino device, but because of Arduino's architecture, not all features are available. **Hapcan programmer** don't know Your device, so it can't control it directly (You can still send direct commands to Your device using), it can't display any specific window for configure Your device either. (Solution for that will be available in a while).

First, search Hapcan network for available devices. 

[[img/hapcan-programmer-search.png|Hapcan programmer found Hapcanuino device]]

Hapcanuino device should have Hardware version of `TYPE:4F41`, which means OA (OnixArts) in ASCII.

## General settings
In General Settings You can change node number, group and device description. No Firmware update is currently available.

[[img/hapcan-programmer-general-settings.png|Hapcan programmer general settings]]

## Using EEPROM HEX editor
You can open processor memory window for Hapcanuino device. First, switch to EEPROM tab (1), then click Read EEPROM button (2). After process is done, You should see something like this:

[[img/hapcan-programmer-eeprom.png|Hapcan programmer EEPROM editor]]

Memory purpose from 0xF00000 to 0xF0004F is the same as in original Hardware. According to limited box count to 32 only first 4 bytes of Enable Box Bits are used by Hapcanuino. This memory addresses are translated by Hapcanuino to Arduino addresses, so if Hapcan programmer addressing bytes from 0xF00000, Hapcanuino accessing EEPROM.

At the 0xF00080 address the first box configuration is located. Each box has 19 Bytes length to reduce EEPROM usage, so it is a little bit hard to identify boxes. You can calculate starting memory address like this: address = 128+(boxNumber-1)*19, where boxNumber is 1-32 and convert it to HEX.

### Box structure

Byte|meaning
--- | --- 
1|Message type High byte
2|Message type Low byte
3|Sender node
4|Sender group
5|Data 0
6|Data 1
7|Data 2
8|Data 3
9|Data 4
10|Data 5
11|Data 6
12|Data 7
13|Operator for bytes 1-4
14|Operator for bytes 5-8
15|Operator for bytes 9-12 
16|Instruction
17|Instruction param 1
18|Instruction param 2
19|Instruction param 3


