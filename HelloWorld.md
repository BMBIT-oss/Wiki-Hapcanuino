## Description
This example shows the basics of Hapcanuino library usage. Without writing single line of code, Hapcanuino should be visible by [Hapcan programmer](http://hapcan.com/software/hap/) software already.

Hello World device can perform only one instruction, toggle LED connected to Arduino's port.

## Required hardware
For base required hardware see [Hardware requirements](https://github.com/Onixarts/Hapcanuino/wiki/Hardware-requirements)

- Connected LED with 1kOhm resistor to PIN7 and GND of Arduino Uno.

## Basic setup
After proper MCP CAN module connection to Arduino board and Hapcan bus, load the Sketch into Arduino.

TODO: To Be continued...

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