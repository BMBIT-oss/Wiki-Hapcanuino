##HapcanDevice class
Main class that handle HAPCAN messages receiving and sending. It also handles base system and programming messages. In short, this class makes Your sketch a Hapcan compatible device. There should be only one instance of this class declared.

Method|Description
---|---
HapcanDevice|Default Constructor
Begin|Initiate CAN module, sets up external interrupt, reads config from EEPROM. Call this method in ino `setup()` function.
Update|Read RX buffer and processes message. Call this method in ino `loop()` function. Method should be called as often as possible.
Send|Send `HapcanMessage` passed as argument. Node and group bytes are set from device's current configuration.
ReceiveAnswerMessages|Set if device should process messages marked as answer. Default is not.
OnMessageAcceptedEvent|Set `MessageAcceptedEventDelegate` callback function to be called when received HAPCAN message meet box criteria. This callback is used in ino file (simple Hapcanuino device implementation).
OnControlMessageEvent|Set `ControlMessageEventDelegate` callback function to be called when control frame is received. This message contains instruction for module to execute (direct control).
OnCanReceived|**Don't use this method directly**. It's for internal use only.
OnCanReceivedDispatcher|**Don't use this static method directly**. It's for internal use only.

##HapcanMessage class