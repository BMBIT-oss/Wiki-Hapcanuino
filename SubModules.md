## Overview

Original Hapcan devices are designed to perform very specific tasks, for example 8 relay outputs, 14 input buttons, blind controller. These are separate, different devices, which has it own PCB, firmware.

Hapcanuino is designed for Arduino which is a simple PCB board created for designers. Here, You are designer :). So You are responsible for all the connections between electronic parts.
That also means, You are responsible for creating a firmware which will handle connected elements.

In [[Examples]] topic You can learn step by step how to use Hapcanuino from very simple examples, where You have to write all the firmware from scratch, to advanced... but simple and fast way way of creating new modules.

### Example

Let's say You want to create module with 4 relay outputs. You may want to configure 4 Arduino pins as an output 
and set LOW or HIGH state on these pins analyzing instructions received from CAN bus by Hapcanuino module. You should also send a proper change state information to the bus, so other devices
can be notified of Your module change. How about implementing delay function for relays as the original Hapcan device's got? You can add some libraries that will schedule tasks for later execution...
but the code becomes illegible. You also have to know precise Hapcan message structure to handle communication.

### Submodules

To make it easier to implement device's firmware let me introduce a **SubModule** concept. You can think about SubModule as an functional block that can perform some specified tasks. You can simply add
4 instances of `HapcanRelay` SubModule to Your device, configure it by very few lines of code and it's done. It will handle Hapcan's messages automatically for You, it has all the original Hapcan's relay module features (even more) and it will send Hapcan relay status change messages to the CAN bus. No Hapcan message structure knowledge required!

You can even add different types of SubModules to one device, and create 4 relay outputs, 10 digital inputs and 3 thermometers inside one device. How cool is that? ;)

## Details

SubModules are available as separate classes stored in /Hapcanuino/SubModules folder. SubModules that will act like an original Hapcan's devices started with Hapcan prefix (HapcanRelay for example).

In Your sketch You can include required SubModules. Using (and creating) submodules will be cover in [[Examples]] topic.
