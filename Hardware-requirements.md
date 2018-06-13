In order to make [Hapcan](http://hapcan.com) module using Arduino You need to connect CAN module to the Arduino board.

It could be simple and cheap MCP2515 module. 
This module is connected to Arduino board by SPI interface using ICSP connector. Hapcanuino takes advantage of external interrupt (PIN2 on Arduino Uno), so this pin is also required to connect MCP module.

Other CAN modules wasn't tested.

## HAPCAN Bus
HAPCAN bus is connected with two data wires (CAN L, CAN H) and GND to the MCP CAN module. You can also use a power supply wires from HAPCAN bus to power Arduino but, You should take care of reducing voltage from Hapcan's 24V to 5-9V with external voltage regulator. Arduino's onboard voltage regulator can (and will) overheat at 24V.
There is also posibility to power Arduino from external power supply, which should have galvanic isolation from AC power.

See more information about Hapcan devices [here](http://hapcan.com/project/basis/).

When connecting Arduino to PC USB port while CAN module connected to Hapcan bus it is safe to use an USB galvanic isolator with isolated DC-DC power lines, like this one: USB-4620 isolator by YN Tech.

