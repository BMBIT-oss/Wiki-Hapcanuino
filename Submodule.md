# Step 5

## Description
This example will show You another, much faster approach to create firmware for Your custom devices.

## Required hardware
For base required hardware see [Hardware requirements](https://github.com/Onixarts/Hapcanuino/wiki/Hardware-requirements)

## Setup
This example use the same circuit as [[Direct Control]] example.

## What is submodule?

Submodule is a class (code) that You can just attach to Your custom device firmware code with just few lines of code. Submodule brings all the functions that author implements. For example if You want Your device has an one controlled LED You can just use Relay submodule and it's done. Your device will support standard Hapcan relay module instructions and will send its state notifications automatically. 

# Implement own HapcanDevice class

Before we get into the submodule itself, let's talk few words about object oriented programming. If You familiar with classes You can inherit custom class from `HapcanDevice` class. These way You don't have to register `ExecuteInstruction` and `SendStatus`, but just override virtual methods declared in `HapcanDevice` base class. These methods are listed below. 

```C++
virtual void OnInit() {};
virtual void OnReadEEPROMConfig() {};
virtual void OnUpdate() {};
virtual void OnExecuteInstruction(InstructionStruct& exec, Hapcan::HapcanMessage& message) {};
virtual void OnStatusRequest(byte requestType, bool isAnswer) {};
```

As You can see, these methods have the same parameters as described previously `ExecuteInstruction` and `SendStatus` callback functions. Hapcanuino will call `OnExecuteInstruction` and `OnStatusRequest` methods before callback functions (if You also register the second ones).

Take a look below. This code is equivalent to previous example (I've removed function bodies for clarity).

```C++
//Hapcan::HapcanDevice hapcanDevice; - don't use this class
// define You own custom derived class instead
class MyLED7Device : public Hapcan::HapcanDevice
{
protected:
    virtual void OnInit()
    {
        // init LED here, not in setup()
        pinMode(PIN7, OUTPUT);
    }

    virtual void OnExecuteInstruction(Hapcan::InstructionStruct& exec, Hapcan::HapcanMessage& message)
    {
        // put ExecuteInstruction() code here
    }
    
    virtual void OnStatusRequest(byte requestType, bool isAnswer)
    {
        // put StatusRequest() code here
    }
};

// and now, instantiate custom MyLED7Device class
MyLED7Device hapcanDevice;

void setup()
{
    Serial.begin(115200);
    Serial.println("Hapcanuino device starting...");

    // initializing Hapcanuino device as usual
    hapcanDevice.Begin();

    // no need to register callbacks, no LED initialization
    // as it is done in OnInit() class method
}

void loop()
{
    hapcanDevice.Update();
}
```
You can use this method or the previous one - it's up to You. I prefer the class based solution. But if You want to implement more functions let's try something even better :).

# HapcanDeviceSubModuleHost

Ohh. That's one, long, ugly... class :). In fact, this is a class template that will automate the process.

Continuing our LED7 example, let's define `MyLED7Device` class again, but in this case the class will inherit from `HapcanDeviceSubModuleHost` template class (note <1> suffix). The whole example is below.

```C++
class MyLED7Device : public Hapcan::HapcanDeviceSubModuleHost<1>
{
    // submodules declaration
    Hapcan::SubModule::HapcanRelay::Module LED7Output;

public:
    MyLED7Device()
    	: LED7Output(*this, 1, PIN7, 0x00)
    {   
        // add SubModule to the host
        m_subModules[0] = &LED7Output;
    }
};

// and now, instantiate custom MyLED7Device class
MyLED7Device hapcanDevice;

void setup()
{
    Serial.begin(115200);
    Serial.println("Hapcanuino device starting...");

    // initializing Hapcanuino device as usual
    hapcanDevice.Begin();
}

void loop()
{
    hapcanDevice.Update();
}

```

And that's it. The `HapcanDeviceSubModuleHost` class will handle `ExecuteInstruction()` and `SendStatus()` for Hapcan relay submodule attached to the host. Submodule will react for the direct and indirect control messages and it will automatically send status change messages.

If You wan't to know more about specific submodule take a look inside submodule folder in source. Each submodule has its own documentation file.

## Next step

To be continued...