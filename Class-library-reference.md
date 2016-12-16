##HapcanDevice class
Main class that handle HAPCAN messages receiving and sending. It also handles base system and programming messages. In short, this class makes Your sketch a Hapcan compatible device. There should be only one instance of this class declared.

Method|Description
---|---
HapcanDevice|Default Constructor
Begin|Initiate CAN module, sets up external interrupt, reads config from EEPROM. Call this method in ino `setup()` function.
Update|Read RX buffer and processes message. Call this method in ino `loop()` function. Method should be called as often as possible.
Send|Send `HapcanMessage` passed as argument. Node and group bytes are set from device's current configuration.
ReceiveAnswerMessages|Set if device should process messages marked as answer. Default is not.
SetExecuteInstructionDelegate|Set `ExecuteInstructionDelegate` callback function to be called when received HAPCAN message meet box criteria or direct control message is received (0x10A). This callback is used in ino file (simple Hapcanuino device implementation).
SetStatusRequestDelegate|Set `StatusRequestDelegate` callback function to be called when status request message is received (0x109). You can call this function directly to send status change information. In this case pass `isAnswer` as false.
OnCanReceived|**Don't use this method directly**. It's for internal use only.
OnCanReceivedDispatcher|**Don't use this static method directly**. It's for internal use only.

##HapcanMessage class

Method|Description
---|---
HapcanMessage()|Default Constructor
HapcanMessage(unsigned long id, byte* buffer)|Constructor used when new message is received. It takes CAN message ID and 8 bytes buffer array and constructs `HapcanMessage` object.
HapcanMessage(unsigned int frameType, bool isAnswer, byte node, byte group)| Constructor used internally from Hapcan Device class to create response messages.
HapcanMessage(unsigned int frameType, bool isAnswer)|Constructor used from user code (ino file). Same as previous but no node and group parameters are available. These parameters will be filled automatically when sending message.
void Prepare(byte node, byte group)|Fill node and group if wasn't specified. 
byte GetFrameTypeCategory()|Returns message category. Used by HapcanDevice to process system messages or normal messages.
unsigned int GetFrameType()|Returns frame type. This method is used by HapcanDevice to recognize current message.
bool IsAnswer()|Returns true if message is an answer for request.
void SetAnswer()|Set answer bit in message.
byte GetNode()|Returns node number from message
byte GetGroup()|Returns node group from message
void PrintToSerial()|Send Message to Serial port. Used for debug purpose.