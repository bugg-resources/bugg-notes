* Consider adding a reset button - handy for dev, resetting after flashing, etc., to reduce the amount of power cycling faff (particularly since the board is often simulatenously powered from USB and bench supply)
* Testpoint for SHDNZ
* Testpoints on every input pin signal (J10)
* U36, Q30, R80 circuit is unnecessary. It serves two purposes. 1) Preventing 5V USB power reaching a 3.3V powered, e.g., ultrasonic mic. This can never happen, because the 3.3V powered device and 5V USB input *share the same pin*, so by physics can never be both connected at the same time. 2) Preventing 5V USB power from flowing into the PIP power supply rail (+3.3VA) if the P3V3 mode is enabled when the USB cable is connected. This is not a problem, because the PIP specification gives the supply range as +1.5V to +5V, so 5V is legal. +3.3VA sources nothing else in the design. When the device is powered from IN_VIN, +3.3VA is produced by regulator U29, whose output is rated up to 6V absolute max, so that also is safe. Conclusion: remove this circuit.
* We should monitor VGPIO to ensure safe power down of the modem, as well as SAFE_PWR_REMOVE. At present, we're just cutting power dead, totally abusing the modem!
* Sack off the ALWAYS-RED power led, it's annoying. Connect it to the User LED on GPIO13. Actually maybe not - power leds are good...
* Maybe add the Activity LED somewhere - that tells you the CM4 is booting, which you can't otherwise tell if the case is closed and buggd is dead
* 12 pin M12??
* Fix borderline undervolt condition when USB powered - DCDC boost?
* Consider resetting the PCF8574 state on boot so the LED's reset with the CM4
* Replace nano SIM connector - major yield problems due to soldering fail (see email "Invoice IN-0837..."
* When recording from a PIP lav mic with the final enclosure wiring, etc. (Rode Lav Go) there is quite a lot of noise around 8kHz - a kind of whistle.