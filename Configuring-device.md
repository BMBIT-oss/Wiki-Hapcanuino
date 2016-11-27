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

After box memory is being edited, click **Save EEPROM** button to store data in Hapcanuino.

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

Bytes 1-12 describes Hapcan Message. Operator bytes 13-15 determines conditions for comparing message bytes. Instruction and params are send to the users firmware function if all message bytes satisfy operator conditions.

#### Operators
One operator byte stores 4 operators for subsequent message bytes. There are three operators available:

Bit A|Bit B|meaning
---|---|---
0|0|Ignore byte
0|1|Input message byte must be equal to this in box
1|0|Input message byte must be different from this in box

Operators for each 4 bytes are located as follows:

<7>|<6>|<5>|<4>|<3>|<2>|<1>|<0>
---|---|---|---|---|---|---|---
Byte 4 A|Byte 4 B|Byte 3 A|Byte 3 B|Byte 2 A|Byte 2 B|Byte 1 A|Byte 1 B

#### Operator Example
Assuming we want call instruction number 4 on our device, each day on 8:00 AM, we want to match Hapcan's time frame sent by [Ethernet module](http://hapcan.com/devices/universal/univ_3/univ_3-102-0-x/index.htm) which is node 1 in group 1. The message sent by this module will looks like this:

30|00|01|01|FF|16|11|27|07|08|00|00
---|---|---|---|---|---|---|---|---|---|---|---
msg type H|msg type L|node|group|FF|year|month|date|day|hour|min|sec

So in first 12 box bytes we can put this message, and then in operators bytes store values to match first 4 bytes (time frame 0x3000, node and group must match, so put (01=must equal) `01 01 01 01` = 55 hex in Byte 13 of box config. Next there are 8 data bytes, and we want to ignore everything that is not matter, leaving hour and minute bytes that must be equal to 8:00 AM. So we have (00=ignore byte) `00 00 00 00` = 0 hex into byte 14 and `00 01 01 00` = 14 hex into byte 15. In byte 16 we have to put our instruction so put 04. Params in this case are irrelevant so we left default FF value.

The whole box config should look like this:
```
30 00  01 01  FF 16 11 27 07 08 00 00 | 55 00 14 | 04 FF FF FF
```
In case we are ignoring some bytes we could also configure this like that:
```
30 00  01 01  FF FF FF FF FF 08 00 FF | 55 00 14 | 04 FF FF FF
```

