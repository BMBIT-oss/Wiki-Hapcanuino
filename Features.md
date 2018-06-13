## Hapcan vs Hapcanuino

[Hapcan](http://hapcan.com) module is based on PIC microcontroller and it has it's own CAN aware bootlader which give an ability to configure, program and upload firmware to a device using CAN network from [Hapcan programmer](http://hapcan.com/software/hap/) software. Hapcan programmer knows each device's FLASH memory and can modify it.

Hapcanuino imitates Hapcan's device but it still has an Arduino bootloader. Data are stored in different FLASH locations, so internally it is very different from original Hapcan hardware. That cause some lacks in full compatibility with original Hapcan programmer software.

## Using Hapcan programmer
When Hapcanuino device is connected to Hapcan network, it can be found by Hapcan programmer. You can use this software to configure Hapcanuino device.

[[img/hapcan-programmer-search.png|Hapcan programmer with Hapcanuino device]]

### What Hapcanuino can do
- Handling all [system messages](http://hapcan.com/devices/universal/univ_3/index.htm)
- EEPROM programming (in programming mode Hapcan's addresses are translated to Arduino's)
- Reseting device
- Changing node and group number (through node general settings)
- Restoring default node and group
- Changing node description
- Box programming via EEPROM Hex editor (box configuration is stored in EEPROM unlike FLASH in original device)

### What Hapcanuino can't do
- No firmware update via CAN network (must use USB connection)
- No FLASH memory reading and writing
- No power supply correct information returned (requires some additional hardware on Arduino board)
- No module text information support (EEPROM is used for box config storage)

## Other features
- 28 boxes available in Arduino with 1kB EEPROM (128 boxes in original device)
