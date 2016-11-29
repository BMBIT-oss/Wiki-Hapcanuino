## Description
This example shows the basics of Hapcanuino library usage. Without writing single line of code, Hapcanuino should be visible by [Hapcan programmer](http://hapcan.com/software/hap/) software already.

Hello World device can perform only one instruction, toggle LED connected to Arduino's port.

## Required hardware
For base required hardware see [Hardware requirements](https://github.com/Onixarts/Hapcanuino/wiki/Hardware-requirements)

- Connected LED with 1kOhm resistor to PIN7 and GND of Arduino Uno.

## Basic setup
After proper MCP CAN module connection to Arduino board and Hapcan bus, load the Sketch into Arduino.
Next, You may want to [configure the device](https://github.com/Onixarts/Hapcanuino/wiki/Configuring-device) with Hapcan programmer.

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
Frame: 0x303  	Node (3,2)		data: FF FF 07 40 21 FF FF FF 
> Accepted box: 1 instr: 1
Frame: 0x303  	Node (3,2)		data: FF FF 87 40 21 FF FF FF 
```
In this particular example You would see message frames received by device, and one accepted message configured in box 1 to fire instruction number 1.

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
In the simplest way of implementing device all your specific code goes into sketch .ino file. So You want to know when Hapcanuino receives a message that meet criteria defined in boxes. To do so, You first declare a callback function, that will be called by internal Hapcanuino code.
```C++
void DoInstruction(Hapcan::HapcanMessage* message, byte instruction, byte param1, byte param2, byte param3);
```
You can put the function body above `setup()` function, but in this example `setup()` function is localized in very top, so compiler must know what `DoInstruction(..)` function is before use it.

Next, the `setup()` function itself. In this function You call `hapcanDevice.Begin()` method, which configures MCP CAN module to work with. You also need to pass pointer to the callback `DoInstruction(..)` function by `hapcanDevice.OnMessageAcceptedEvent(DoInstruction)` method.
```C++
void setup()
{
	Serial.begin(115200);
	Serial.println("Hapcanuino device starting...");

	// initializing Hapcanuino device
	hapcanDevice.Begin();

	//set callback function to be called, when received message match box criteria
	hapcanDevice.OnMessageAcceptedEvent(DoInstruction);

	// demo example, set pin7 as output
	pinMode(PIN7, OUTPUT);
}
```
There is also a line to set PIN7 as an output to drive LED, but this is example specific code. You should configure the rest of Your device here as all Arduino Sketches do.

`loop()` function requires only one line to make Hapcanuino work.
```C++
void loop()
{
	hapcanDevice.Update();
}
```
The `Update()` method processes the received messages from RX buffer, handle system messages, checks normal messages if any of them match box criteria and if so, calls the `DoInstruction(...)` function.

Simplest implementation of `DoInstruction(..)` function is to put `switch` code and perform operations according to instruction parameter.
```C++
void DoInstruction(Hapcan::HapcanMessage* message, byte instruction, byte param1, byte param2, byte param3)
{
	switch (instruction)
	{
	case 1: digitalWrite(PIN7, digitalRead(PIN7) == LOW);
		break;
		// TODO: place other instructions here
	}
}
```
In this case, when box configuration calls instruction 1 it toggle LED in PIN7.

Callback function receives pointer to `HapcanMessage` which is the whole message receives from CAN bus. You can use its data in your code, for example to get current time send by [Ethernet module](http://hapcan.com/devices/universal/univ_3/univ_3-102-0-x/index.htm) each minute.
Instruction and three params are defined in box that meets message criteria. These params are stored in EEPROM and can be configured using Hapcan programmer's EEPROM HEX editor. Dedicated solution will be available asap.