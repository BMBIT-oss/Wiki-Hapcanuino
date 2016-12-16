## Description
This example shows the basics of Hapcanuino library usage. Without writing single line of code, Hapcanuino should be visible by [Hapcan programmer](http://hapcan.com/software/hap/) software already.

Hello World device can perform three instructions, LED ON, LED OFF, and TOGGLE LED connected to Arduino's port on PIN7.

## Required hardware
For base required hardware see [Hardware requirements](https://github.com/Onixarts/Hapcanuino/wiki/Hardware-requirements)

- Connected LED with 1kOhm resistor to PIN7 and GND of Arduino Uno.

## Basic setup
After proper MCP CAN module connection to Arduino board and Hapcan bus, load the Sketch into Arduino.
Next, You may want to [configure the device](https://github.com/Onixarts/Hapcanuino/wiki/Configuring-device) with Hapcan programmer. In instruction field (byte 16) put:

Instruction|Description
---|---
1| LED ON
2| LED OFF
3| TOGGLE LED

So for toggle instruction box should looks like below:
```
30 30  03 02  FF FF 07 40 4A FF FF FF | 55 55 55 | 03 FF FF FF
```
where `30 30` is a IR Receiver message (in my case), `03 02` is a sender's module node and group, `FF FF 07 40 4A FF FF FF` is my IR remote controller button pushed code. This part is up to You. 
Next `55 55 55` means that all received message bytes must be equal to this configured in BOX, and finally `03 FF FF FF` means instruction 3 with empty parameters.

## First run
In Arduino's Serial port monitor You should see some debug messages and CAN traffic. 
```
Opening port
Port open
Hapcanuino device starting...
Entering Configuration Mode Successful!
Setting Baudrate Successful!
Frame: 0x303  	Node (3,2)		data: FF FF 07 40 07 FF FF FF 
Frame: 0x302  	Node (2,1)		data: FF FF 05 00 FF FF 10 00 
Frame: 0x303  	Node (3,2)		data: FF FF 87 40 07 FF FF FF 
Frame: 0x303  	Node (3,2)		data: FF FF 07 40 07 FF FF FF 
Frame: 0x303  	Node (3,2)		data: FF FF 07 40 4A FF FF FF   <- this message triggers box 1
> Accepted box: 1 instr: 3
Frame: 0x333  	Node (3,3)		data: FF FF 07 01 FF FF FF FF   <- module response with current LED status
Frame: 0x303  	Node (3,2)		data: FF FF 87 40 21 FF FF FF 
```
In this particular example You would see message frames received by device, and one accepted message configured in box 1 to fire instruction number 3.

## Code explanation

First You need to include Onixarts_Hapcanuino library.
```C++
#include "HapcanDevice.h"
```
Hapcanuino library uses C++ namespaces to arrange code, so declare using a `Onixarts::HomeAutomationCore` namespace. This line will tell compiler to look into this namespace to find classes and other types.
```C++
using namespace Onixarts::HomeAutomationCore;
```
Next, declare a HapcanDevice class object
```C++
Hapcan::HapcanDevice hapcanDevice;
```
Now, it's time to configure Your device. We need to provide some device describing parameters to distinguish different devices from each other in Hapcan's network. This parameters are stored in constants grouped in namespaces declared in Hapcanuino library. Just put valid values for each parameter. 

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

const byte Hapcan::Config::Firmware::ApplicationType = 51;		// application (hardware) type (such as button, relay, dimmer) 1-10 Hapcan modules, 102 - ethernet, 51 - Hapcanuino Hellow World 1 LED device
const byte Hapcan::Config::Firmware::ApplicationVersion = 0;	// application (hardware) version, change it with device hardware changes
const byte Hapcan::Config::Firmware::FirmwareVersion = 1;		// firmware version
const int Hapcan::Config::Firmware::FirmwareRevision = 0; // firmware revision
```
**MCP::InterruptPin** - is the Arduino external interrupt pin number where MCP module INT line is connected. You can read more about external interrupts and find valid pin for Your Arduino board [here](https://www.arduino.cc/en/Reference/AttachInterrupt)

**MCP::CSPin** - is the Arduino SPI Chip Select pin connected to MCP module CS pin (10 in most Arduino boards, and 53 on Arduino Mega).

**MCP::OscillatorFrequency** - set this parameter to match You MCP module oscillator (quartz resonator) frequency. In most cases it will be MCP_8MHZ or MCP_16MHZ.

**Hardware::DeviceIdX** - Two bytes of device ID. This ID is returned on Device ID request (0x10F)

**Node::SerialNumberX** - Four bytes with device serial number. This number is presented in Hapcan programmer. Note, that SerialNumber2 and SerialNumber3 also define default node and group. Default values are used on first start of device and when Change to default group and node values message is sent to device. So choose your values carefully, to not override another device in Hapcan's network. 

**Firmware::ApplicationType** - it is Your hardware application type. For example Hapcan's original relay module application type is 2 (1-10 are occupied). If You create a new hardware, like analog inputs, temperature sensors collector or aquarium controller You assign a new application type.

**Firmware::ApplicationVersion** - it is Your hardware application version. If You make some hardware changes in Your existing module (for example, add more inputs to analog input module) assign new number here.

**Firmware::FirmwareVersion** - When changing code for Your existing device that delivers new functions make sure You assign a new version here.

**Firmware::FirmwareRevision** - When fixing some bugs for Your existing device make sure You assign a new revision here. You can also put here a new build number.

In the simplest way of implementing device all your specific code goes into sketch .ino file. So You want to know when Hapcanuino receives a message that meet criteria defined in boxes or a direct control message is received. To do so, You first declare a callback function, that will be called by internal Hapcanuino code.
```C++
void ExecuteInstruction(byte instruction, byte param1, byte param2, byte param3, Hapcan::HapcanMessage& message);
```
You can put the function body above `setup()` function, but in this example `setup()` function is localized in very top, so compiler must know what `ExecuteInstruction(..)` function is before use it.

There is one more important callback function which You probably need to implement (but You don't have to). The `OnStatusRequest` function is called when status request message is received (frame type 0x109). I will talk about this function later. Now just put:

```C++
void OnStatusRequest(byte requestType, bool isAnswer);
```
There is also a `LED7Info` constant definition in `StatusRequestType` namespace.

Next, the `setup()` function itself. In this function You call `hapcanDevice.Begin()` method, which configures MCP CAN module to work with. You also need to pass pointer to the callback `ExecuteInstruction(..)` function by `hapcanDevice.SetExecuteInstructionDelegate(ExecuteInstruction)` and `SetStatusRequestDelegate(OnStatusRequest)` methods if You want Your module to handle Your own messages.

```C++
void setup()
{
	Serial.begin(115200);
	Serial.println("Hapcanuino device starting...");

	// initializing Hapcanuino device
	hapcanDevice.Begin();

	//set callback function to be called, when received message match box criteria or direct control message is received
	hapcanDevice.SetExecuteInstructionDelegate(ExecuteInstruction);

	// Callback function to be called, when received status request message
	void OnStatusRequest(byte requestType, bool isAnswer);

	// demo example, set pin7 as output
	pinMode(PIN7, OUTPUT);
}
```
There is also a line to set PIN7 as an output to drive LED, but this is example specific code. You should configure the rest of Your device here as all Arduino Sketches do.

`loop()` function requires only one line to make Hapcanuino works.
```C++
void loop()
{
	hapcanDevice.Update();
}
```
The `Update()` method processes the received messages from RX buffer, handle system messages, checks normal messages if any of them match box criteria and if so, calls the `ExecuteInstruction(...)` function.

Simplest implementation of `ExecuteInstruction(..)` function is to put `switch` code and perform operations according to instruction parameter.
```C++
void ExecuteInstruction(byte instruction, byte param1, byte param2, byte param3, Hapcan::HapcanMessage& message)
{
	bool ledStateChanged = false;

	switch (instruction)
	{
	case 1: // turn LED ON
		digitalWrite(PIN7, HIGH);
		ledStateChanged = true;
		break;
	case 2: // turn LED OFF
		digitalWrite(PIN7, LOW);
		ledStateChanged = true;
		break;
	case 3: // toggle LED
		digitalWrite(PIN7, digitalRead(PIN7) == LOW);
		ledStateChanged = true;
		break;
	//case 4: // put other instructions here; break;
	}

	// check, if LED change instruction was executed and send message to Hapcan
	if (ledStateChanged)
	{
		// send status frame after change of LED7 to notify other Hapcan modules. 
		// Notice second (isAnswer) parameter is set to false, because we call it directly after status change
		OnStatusRequest(StatusRequestType::LED7Info, false);
	}
}
```
In this case, when box configuration calls instruction 3 it toggles LED in PIN7. It also calls `OnStatusRequest` function which is also a callback function being called when status request message is received. We pass additional information to this function to send only LED7Info status, and set response flag to 0 in sended message (isAnswer = false).

Callback function receive instruction and three parameters that are defined in box that meets message criteria. Parameters can also be send from PC via direct control message. For example, to direct toggle LED, PC should send this message:
```
10 A0  F0 F0  03 FF 20 09 FF FF FF FF
```
where `10 A0` is the type of direct control frame, `F0 F0` is the PC node and group, `03 FF` is the instruction number 3 and first parameter, `20 09` is the Hapcanuino's node and group, `FF FF` are parameters 2 and 3, and last `FF FF` are unused in Hapcanuino. Instruction with parameters can occupy 4 bytes only, same as in box configuration.

Callback function receives also a pointer to `HapcanMessage` which is the whole message receives from CAN bus. You can use its data in your code, for example to get current time send by [Ethernet module](http://hapcan.com/devices/universal/univ_3/univ_3-102-0-x/index.htm) each minute. In case of direct control message, You can pass more parameters to instruction using last bytes.

### Handling status request and status messages

Function `OnStatusRequest(...)` can be called directly from Your code - when one of the device state is changed, in this example it is called from `ExecuteInstruction(..)` function after LED status changed. Then You call it with `isAnswer` parameter set to false, because it is not an answer, it is just a status changed information.
Function is also called by the `HapcanDevice` class when status request message is received. Then the `isAnswer` parameter is set to true. You should use `isAnswer` when creating new `HapcanMessage`.
Other parameter is the `requestType`. You can use it to determine which status should be send. For example if Your device has 8 LED diodes instead of 1 as in this example, You may want to send only one status change message that match LED which has changed. You can point which LED status will be send here. `HapcanDevice` will call this method with `requestType` set to `StatusRequestType::SendAll` which is value of 0. So Your other LED channels can be 1,2,3 and so on.

```C++
void OnStatusRequest(byte requestType, bool isAnswer)
{
	// check if we should send informations about all the functions in module
	bool sendAll = requestType == Hapcan::Message::System::StatusRequestType::SendAll;

	// if we need send all info or just LED7Info status
	if (sendAll || requestType == StatusRequestType::LED7Info)
	{
		// send message status of frame type 0x333 (custom). 
		// Use isAnswer variable here, because it will be set to true when it is a response for StatusRequest message (0x109)
		Hapcan::HapcanMessage statusMessage(0x333, isAnswer);
		statusMessage.m_data[2] = 7;	// set up byte 3 as 7
		statusMessage.m_data[3] = digitalRead(PIN7) == LOW ? 0x00 : 0x01; // set byte 4, 1 = LED ON, 0 = LED OFF
		hapcanDevice.Send(statusMessage);
	}

	// Add another status messages here...
	// if (sendAll || requestType == StatusRequestType::Your_another_defined_status)
	//{
	//}
}
```
The `sendAll` variable is set to true if we need to send whole module status (for example 8 LED channels).

In current example only one status info is send on `StatusRequestType::LED7Info` type or `SendAll` type.
New `HapcanMessage` with current status of LED with frame type of 0x333 is send to the CAN bus `FF FF 07 0X FF FF FF FF`, where X = 1 when LED in ON, and 0 when LED is OFF. This way other Hapcan devices know that LED state has changed and can process this information or status asking device (PC) receives whole status info.