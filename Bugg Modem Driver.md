When the application firmware runs:
* The state of the modem power rail (3V7_EN GPIO pin configuration) is unknown
* The modem's power state is unknown - it could be off, on or, possibly, booting up. Booting can take as long as 30 seconds between the POWER_ON_N signal being asserted and USB enumeration completing

We can check the rail status by checking the 3V7_EN pin. 

We can check the modem is powered by whether it's enumerated on USB. That's the only way we have, really.

The modem may be on but :
* Missing a SIM card
* Missing an antenna
* Unable to connect to the GSM network
* Unable to see any cell towers.

Didn't bother with a state machine in the end - we don't want to just implement a shit version of ModemManager. 

I implemented some serial control methods using pyserial, to check the SIM is present, and to safely power down the modem by issuing the soft power off command. 

Wrote a standalone tool modemctl to interact with the modem from the CLI.
