Goals:
1) Catch PCB assembly bugs
2) Catch FPC bad inserts - can't record from internal mic, can't control LED's
3) Catch M12 connector failure
4) Catch antenna connection failure - check for cell towers?

# PCB Assembly bugs

1) Turn on all of the rails and measure the testpoints
2) Self test: check modem enumerates
3) Self test: SD card
4) Self test: SIM ID (make sure ModemManager / NetworkManager are stopped
5) Self test: I2C responses from all chips
6) LEDS: blink, visual test?
7) Record internal and external, though disconnected. Check for hiss? 

# Scratchpad

signal strength:
at+csq
+csq: 13,99

-- removed antenna here

OK
Aat
OK
at+csq
+csq: 10,99

-- took a few seconds before 99,99, indicating no signal
OK
at+csq
+csq: 99,99
>>> s=serial.Serial("/dev/tty_modem_command_interface",115200,timeout=1);s.write("at+ccid?\r\n".encode());time.sleep(0.5);s.read_all();s.close()
10
b'\r\n+CCID: 8944303513211074926\r\n\r\nOK\r\n'

