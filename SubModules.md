## Overview

Original Hapcan devices are designed to perform very specific tasks, for example 8 relay outputs, 14 input buttons, blind controller. These are separate, different devices, which has it own PCB, firware. 

Hapcanuino is designed for Arduino which is a simple PCB board created for designers. Here, You are designer :). So You are responsible for all the connections between electronic parts.
That also means, You are responsible for creating a firmware which will handle connected elements.

In HelloWorld example You can find a basic Hapcanuino usage as handling `StatusRequest` and `ExecuteInstruction` events. In this case You must write all the firmware from scratch.

Let's say You want to create module with 4 relay outputs. You may want to configure 4 Arduino pins as an output 
and set LOW or HIGH state on these pins analyzing instructions received from CAN bus by Hapcanuino module. You should also send a proper change state information to the bus, so other devices
can be notified by Your module. How about implementing delay function for relays as the original Hapcan device's got? You can add some libraries that will schedule tasks for later execution...
but the code becomes illegible.

To make it easier to implement device's firmware let me introduce a **SubModule** concept. You can think about SubModule as an functional block that can perform a task. You can simply add
4 instances of `HapcanRelay` SubModule to Your device, configure it by very few lines of code and it's done. It will handle Hapcan's messages automatically, it has all the features (even more)
and it will send Hapcan relay status change messages to the CAN bus.

You can even add different types of SubModules to one device, and create 4 relay outputs, 10 digital inputs and 3 thermometers inside one device.

## Details

SubModules are available as separate classes stored in /Hapcanuino/SubModules folder. SubModules that will act like an original Hapcan's devices started with Hapcan prefix (HapcanRelay for example).

In Your sketch You can include required SubModules. There are two aproaches for creating device firmware described in [[Writing firmware]] topic.