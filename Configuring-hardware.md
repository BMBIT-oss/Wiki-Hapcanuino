Each device You will build need to be configured in code side, so it will be recognizable in Hapcan system. In the main ino file You should put these configuration lines and fill up the defined constants to the proper values. Note, there are some constants that must be different in any device connected to the Hapcan bus.

```C++
const byte Hapcan::Config::MCP::InterruptPin = 2;				// CAN module interrupt is connected to this pin (see https://www.arduino.cc/en/Reference/AttachInterrupt)
const byte Hapcan::Config::MCP::CSPin = 10;						// SPI CS pin
const byte Hapcan::Config::MCP::OscillatorFrequency = MCP_8MHZ;	// MCP oscillator frequency on MCP CAN module (or MCP_16MHz)

const byte Hapcan::Config::Hardware::DeviceId1 = 0x12;			// unique device identifier 1, change it
const byte Hapcan::Config::Hardware::DeviceId2 = 0x34;			// unique device identifier 2, change it

const byte Hapcan::Config::Node::SerialNumber0 = 9;				// ID0 serial number MSB
const byte Hapcan::Config::Node::SerialNumber1 = 9;
const byte Hapcan::Config::Node::SerialNumber2 = 32;			// this is also a default node
const byte Hapcan::Config::Node::SerialNumber3 = 9;				// this is also a default group

const byte Hapcan::Config::Firmware::ApplicationType = 50;		// application (hardware) type (such as button, relay, dimmer) 1-10 Hapcan modules, 102 - ethernet, 50 - Hapcanuino default device
const byte Hapcan::Config::Firmware::ApplicationVersion = 0;	// application (hardware) version, change this value each time You make some changes in device hardware
const byte Hapcan::Config::Firmware::FirmwareVersion = 1;		// firmware version
const int Hapcan::Config::Firmware::FirmwareRevision = 0; // firmware revision
```

For more information about using this code see [Hello World example](https://github.com/Onixarts/Hapcanuino/blob/master/examples/HelloWorld/HelloWorld.ino).

## Description

**MCP::InterruptPin** - is the Arduino external interrupt pin number where MCP module INT line is connected. You can read more about external interrupts and find valid pin for Your Arduino board [here](https://www.arduino.cc/en/Reference/AttachInterrupt)

**MCP::CSPin** - is the Arduino SPI Chip Select pin connected to MCP module CS pin (10 in most Arduino boards, and 53 on Arduino Mega).

**MCP::OscillatorFrequency** - set this parameter to match You MCP module oscillator (quartz resonator) frequency. In most cases it will be MCP_8MHZ or MCP_16MHZ.

**Hardware::DeviceIdX** - Two bytes of device ID. This ID is returned on Device ID request (0x10F)

**Node::SerialNumberX** - Four bytes with device serial number. This number is presented in Hapcan programmer. Note, that SerialNumber2 and SerialNumber3 also define default node and group. Default values are used on first start of device and when Change to default group and node values message is sent to device. So choose your values carefully, to not override another device in Hapcan's network. 

**Firmware::ApplicationType** - it is Your hardware application type. For example Hapcan's original relay module application type is 2 (1-10 are occupied). If You create a new hardware, like analog inputs, temperature sensors collector or aquarium controller You should assign a new hardware application type.

**Firmware::ApplicationVersion** - it is Your hardware application version. If You make few different variants of the hardware (for example 6 analog inputs and 12 analog inputs) or change hardware capabilities (for example, add new input type) assign new number here.

**Firmware::FirmwareVersion** - When changing code for Your existing device that delivers new functions make sure You assign a new version here, so any external programming tool can handle different version of EEPROM structure for example.

**Firmware::FirmwareRevision** - You can put code revision here. This value is not returned for any system message. You can use it internally for new firmware releases.
