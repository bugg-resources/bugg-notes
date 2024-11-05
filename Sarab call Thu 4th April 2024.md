Making hay while the sun shines 

Slight delay in the hardware. First units arrived at GroupGets, they're prodding Vital.

I've done an insane quantity of development on both 

# Software Status
Intro - buggOS and buggd (and also bugg-soundcard-driver)

## buggOS
### Talking points
- [ ] Github Actions CI/CD automation is complete, for now at least. 
- [ ] It was an incredible slog, vertical learning curve, nightmarish debugging cycle, but it's done. 
- [ ] Very happy with it.
- [ ] Pipeline has two jobs - build and release
- [ ] Build creates the OS image artefact
- [ ] Release applies a filename with version number and uploads the artefact out of the runner to the surrounding buggOS repo for download by humans
- [ ] Version numbers are defined by Semantic Versioning
- [ ] Version numbers are set with tags in the repo
- [ ] Build passes - artefact is created
- [ ] Release passes - artefact is uploaded

buggOS build system (Github Actions)
	Build pass

## buggd

EVEN MORE PROGRESS MADE HERE!

### Talking points
- [ ] buggd is the recording daemon
- [ ] It's a Git repo
- [ ] It defines a Python package
- [ ] Grown, complexified and simplified into 27 python modules and counting. 17 classes.
- [ ] Contains the original business logic - recording, sensors, threads, etc
- [ ] More robust:
- [ ] Discovered some bugs, handled with some better quality engineering
	- [ ] modem power state properly managed - no pulling the plug
	- [ ] log file rotation (now handled explicitly), etc
	- [ ] ... more
- [ ] Extends it massively with new driver modules
	- [ ] modem driver
	- [ ] soundcard driver
	- [ ] leds
	- [ ] pcmd3180
	- [ ] userled
	- [ ] resource locking
- [ ] The package provides a number of applications:
	- [ ] buggd - the recording daemon
	- [ ] soundcardctl - an interactive app for controlling the soundcard directly from the CLI
	- [ ] modemctl - an interactive app for controlling the soundcard
- [ ] Factory Self-test happens in two levels
	- [ ] Triggered by files on SD card
	- [ ] Board level (turn on rails, operator measures a load of test points - bed-of-nails in future)
	- [ ] Product level (fully assembled unit)
- [ ] Factory test results are handled
	- [ ] 9 Tests performed:
		- [ ] SD mountable, readable (implicit)
		- [ ] Modem enumerates
		- [ ] Modem responds to AT commands
		- [ ] Modem SIM is readable
		- [ ] Modem can see cell towers, gets valid RSSI
		- [ ] I2S bridge (pcmd3180) responds
		- [ ] RTC responds
		- [ ] LED controller responds
		- [ ] Internal microphone can record a signal (variance test)
		- [ ] External microphone can record a signal (variance test)
	- [ ] Results written permanently to disk
	- [ ] Results displayed on console on login
	- [ ] Results logged to log
	- [ ] Results displayed on LED's as test is performed
	- [ ] Results summary displayed on LEDs on boot 
- [ ] New ExternalMic(SensorBase) class
	- [ ] new config file options
		- [ ] phantom
		- [ ] gain
		- [ ] dual channel mode
		- [ ] stereo/mono file recording management


# What next
- [ ] One weird bug remaining in buggd
- [ ] Griffin hasn't got all units, so I'll try to fix today
- [ ] If not,worst case, flash twice
- [ ] Thumbscrews ready
- [ ] 