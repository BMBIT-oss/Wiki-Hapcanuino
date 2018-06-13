You can use [Hapcan programmer](http://hapcan.com/software/hap/) software to set some parameters of Hapcanuino device, but because of Arduino's architecture, not all features are available. **Hapcan programmer** don't know Your device, so it can't control it directly (You can still send direct commands to Your device using Monitor window), it can't display any specific window for configure Your device either. (Solution for that will be available in a while. Currently work is in progress on new [Hapcan Programmer 2](https://github.com/Onixarts/HapcanProgrammer) that will solve custom devices programming).

First, search Hapcan network for available devices. 

[[img/hapcan-programmer-search.png|Hapcan programmer found Hapcanuino device]]

Hapcanuino device will have `Hapcanuino` in hardware version column. In programmer software older than 3.50 the hardware version will be shown as `TYPE:4F41`, which means OA (OnixArts) in ASCII.

## General settings
In **General Settings** You can change node number, group and device description. No Firmware update is currently available.

[[img/hapcan-programmer-general-settings.png|Hapcan programmer general settings]]

## Using EEPROM HEX editor

You can open processor memory window for Hapcanuino device. First, switch to EEPROM tab (1), then click Read EEPROM button (2). After process is done, You should see something like this:

[[img/hapcan-programmer-eeprom.png|Hapcan programmer EEPROM editor]]

Memory purpose from 0xF00000 to 0xF0004F is the same as in original Hardware. According to limited box count to 28 only first 4 bytes of Enable Box Bits are used by Hapcanuino. This memory addresses are translated by Hapcanuino to Arduino addresses, so if Hapcan programmer addressing bytes from 0xF00000, Hapcanuino accessing EEPROM.

At the 0xF00080 address the first box configuration is located. Each box has 32 Bytes length (as original Hapcan modules). You can calculate box starting memory address like this: address = 128+(boxNumber-1)*32, where boxNumber is 1-28 and convert it to HEX. You can also use calculator in [Excel memory file](https://github.com/Onixarts/Hapcanuino/blob/master/docs/Hapcanuino_1-50-0-0-memory.xlsx).

After box memory is being edited, click **Save EEPROM** button to store data in Hapcanuino.

### Box structure
0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
F1|F2|F3|F4|F5|F6|F7|F8|F9|F10|F11|F12|Op1|Op2|Op3|Op4
Op5|Op6|Op7|Op8|Op9|Op10|Op11|Op12|Inst1|Inst2|Inst3|Inst4|Inst5|Inst6|Inst7|Inst8

Bytes 0-11 (F1-F12) describes filter values for received Hapcan Message. Operator bytes (Op1-Op12) determines matching functions for each byte. 
Each received message byte is compared with F1-F12 values using operators Op1-Op12. If all matches are satisfied then Hapcanuino calls ExecuteInstruction delegate function passing Inst1-Inst8 parameters. 
This functon should execute specific module instruction then.

#### Operators
Available operators:

Value|meaning
---|---
0x00|CAN byte don't need checking. It always matched any value (ignores the byte)
0x01|CAN byte must be equal to coresponding filter byte
0x02|CAN byte must be different than coresponding filter byte
0x03|CAN byte must be less or equal to coresponding filter byte
0x04|CAN byte must be greater or equal to coresponding filter byte

#### Operator Example
Assuming we want call instruction number 4 on our device, each day on 8:00 AM, we want to match Hapcan's time frame sent by [Ethernet module](http://hapcan.com/devices/universal/univ_3/univ_3-102-0-x/index.htm) which is node 1 in group 1. The message sent by this module will looks like this:

30|00|01|01|FF|16|11|27|07|08|00|00
---|---|---|---|---|---|---|---|---|---|---|---
msg type H|msg type L|node|group|FF|year|month|date|day|hour|min|sec

So in first 12 filters box bytes we can put this message, and then in operators bytes store values to match first 4 bytes (time frame 0x3000, node and group must match, so put (0x01=must equal) `01 01 01 01` in Op1-Op4 bytes. Next there are 8 data bytes in CAN message, and we want to ignore everything that is not matter, leaving hour and minute bytes that must be equal to 8:00 AM. So we have (0x00=accept anything) `00 00 00 00 00 01 01 00` in Op5-Op12. In Instr1 we have to put our instruction so put `0x04`. Params in this case are irrelevant so we left default `0xFF` values.

The whole box config should look like this:
```
30 00  01 01  FF 16 11 27 07 08 00 00 | 01 01 01 01 00 00 00 00 00 01 01 00 | 04 FF FF FF FF FF FF FF
```
In case we are ignoring some bytes we could also configure box like below:
```
30 00  01 01  FF FF FF FF FF 08 00 FF | 01 01 01 01 00 00 00 00 00 01 01 00 | 04 FF FF FF FF FF FF FF
```
and leave only these bytes in F1-F12 filters which are important (using by comparison function other than 0x00 - match anything).

