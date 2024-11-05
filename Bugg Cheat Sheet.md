
# Testing doc 
https://docs.google.com/spreadsheets/d/1cMmZYa-TVD4BYe5KFDFw56bmfUa1-lQCvIrKGAzqqnk/edit#gid=1712861135

This shows lots of details for setup and test.

# Turn on/off wifi
``
```
nmcli radio wifi off
```

# Modem connection

Create connection profile:
```
sudo nmcli con add type gsm ifname '*' con-name 'ee' apn 'everywhere' gsm.username 'eesecure' gsm.password 'secure'
```

Turn on modem
`en_modem.py`

Connect
`nmcli con up ee`

# Record some audio

Build and install bugg-soundcard-driver codec and DT overlay

1) Check it's properly installed:
```
bugg@bugg-v3:~$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: soundcard [soundcard], device 0: bcm2835-i2s-ad400x-hifi ad400x-hifi-0 [bcm2835-i2s-ad400x-hifi ad400x-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

2) Set the gain and phantom mode:
```
soundcardctl power external on
soundcardctl gain 10
soundcardctl phantom none
```

3) Record:
```
arecord -D hw:0,0 --channels=2  --format=S16_LE  --rate=48000 --duration=5 r4-1kHz-48ksps-gain6dB.wav
```


## 48V Phantom-powered shotgun condenser

Tested with ECM999 mic
Standard 3-pin XLR pinout
P48 phantom setting
Works well with about 45dB of gain.

## Plug In Powered Lavalier type

Tested with Rode SmartLav+ Pinout and connections as follows:

| SmartLav | Signal | Bugg |
| ---- | ---- | ---- |
| T | none | none |
| R | none | none |
| R | Ground | AGND |
| S | Signal, bias voltage | MIC_IN_P |
|  | none | MIC_IN_N |

## Ultrasonic mic

Tested with US-D from Titley Scientific.
3V3 power mode.

## Internal mic

Run the pcmd3180_i2c_init.sh script
(Updated script for SHDNZ on GPIO0)

It's pretty quiet - set max volume with
`sudo i2cset -y 1 0x4c 0x3e 0xff`

## Recording audio for self-test

I want the self-test to detect that the internal or external mic's aren't working.

We can dump two streams like this:
```
arecord --separate-channels --device plughw:0,0 --channels 2 --format S16_LE --rate 48000 --duration 1 out.wav
```

The output files aren't wav - they're raw data, because arecord can't create two separate files from one stream of interleaved L/R samples. That makes some kind of sense when you think about it.

So can we detect a defective channel?
We get two outputs files of raw data:
out.wav.0 <-- Internal mic
out.wav.1 <-- External mic

With P48 phantom selected, and even with 0dB gain, there's a detectable background hiss (obviously).

### Removing the internal mic:



# syslog (journal)

To get output from a systemd unit, you can say
`journalctl -u first-boot.service`

To get output from a particular identifier (or "tag" as the "logger" command calls it
`journalctl SYSLOG_IDENTIFIER=first-boot.sh`

To get the output of buggd, run:
```journalctl -f -u buggd.service ```
