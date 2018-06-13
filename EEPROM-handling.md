## EEPROM handling

Hapcanuino stores some important data in EEPROM, just like the original Hapcan device does. Data structure stored in EEPROM is very important, because EEPROM memory can be programmed with Hapcan programmer software. The programmer knows this structure and using it in many ways. 
There are also box configuration stored in EEPROM so **You shouldn't access EEPROM directly** and store data - it may be dangerous if You overwrite some important data. 

Memory structure is described in [[Memory.xlsx|https://github.com/Onixarts/Hapcanuino/tree/master/docs]] file.

To make it easier to store and reading configuration data `HapcanDevice` class provide this methods:

```C++
bool GetConfigByte(byte configBank, byte byteNumber, byte& value);
bool SetConfigByte(byte configBank, byte byteNumber, byte value);
```
You can use this method for safely get and set EEPROM values from one of config banks. You can't use it for accessing box configurations - but You really don't have to, as Hapcanuino will handle incomming CAN messages for You automatically.

### Config banks

There are 2 config banks:

**Node Config** - is similar to original Hapcan device and is located at the beginning of the memory starting at byte 0x08. There are 24 bytes available to store and read device configuration. It is probably best not to write any data from device itself. This bank should be configured by external programmer, Your device should only read settings from this bank. But there is no save restrictions on this bank.

**Extended Config** - there are 47 byte available and You can store here some data that does not fit in Node Config bank or save any runtime settings - for example current relay status. 

#### Important note

EEPROM memory has limited save cycles to about 100 000, so use this memory carefully because overwriting memory location very often will cause memory damage! Hapcanuino optimizes save operations by not overwriting the EEPROM cells if there is already equal byte stored.

### Accessing data

Reading byte 17 from Node Config bank is simple as:

```C++
byte value = 0;
if( GetConfigByte(Hapcan::ConfigBank::NodeConfig, 17, value)
{
    Serial.println(value);
}
```
`GetConfigByte(..)` method guards the memory access and if You cross the bank memory boundaries it will returns false. You can always check each bank capacity by accessing `Hapcan::ConfigBank::NodeConfigCapacity` constant etc.

### Best practices

It is not a good practise to use any constant values in code directly, instead use constants declaration or old school #define directive to point to Your data. Lets say we stored default OLED display brightness connected to our device in Node Config at byte 17. We can organize our memory config in the top of ino file like this:

```C++
namespace NodeConfigBank
{
   //const byte MyData1 = 16;
   const byte DefaultOLEDBrightness = 17;
   //const byte MyData2 = 18;
}

//...

byte oledBrightness = 0;
if( GetConfigByte(Hapcan::ConfigBank::NodeConfig,NodeConfigBank::DefaultOLEDBrightness, oledBrightness))
{
  // Set OLED brigthness here at startup
}
```

This way it is very easy to recognize each config byte meaning, but You can use `#define DEFAULT_OLED_BRIGHTNESS 17` if You like - it's up to You. I'm using namespace convention for clarity. 
You probably want to fill this information also in docs/memory.xls file and place it with Your device code for better understanding.