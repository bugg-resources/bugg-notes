# Scratchpad

Apr 03 18:55:21 bugg-v3 sudo[1031]:     root : PWD=/ ; USER=root ; COMMAND=/usr/bin/timeout 180s ntpdate ntp.ubuntu.com
Apr 03 18:55:21 bugg-v3 python[665]: RPiID-10000000b1bab292 - Updating time from internet before GCS sync
Apr 03 18:55:21 bugg-v3 python[665]: INFO:root:Updating time from internet before GCS sync
Apr 03 18:55:21 bugg-v3 python[665]: RPiID-10000000b1bab292 - Connected to the Internet
Apr 03 18:55:21 bugg-v3 python[665]: INFO:root:Connected to the Internet
Apr 03 18:55:21 bugg-v3 python[665]: RPiID-10000000b1bab292 - Waiting for internet connection...
Apr 03 18:55:21 bugg-v3 python[665]: INFO:root:Waiting for internet connection...
Apr 03 18:55:21 bugg-v3 python[665]: RPiID-10000000b1bab292 - Modem is enumerated.
Apr 03 18:55:21 bugg-v3 python[665]: INFO:buggd.drivers.modem:Modem is enumerated.
Apr 03 18:55:21 bugg-v3 python[665]: RPiID-10000000b1bab292 - Checking if modem is enumerated...
Apr 03 18:55:21 bugg-v3 python[665]: INFO:buggd.drivers.modem:Checking if modem is enumerated...
Apr 03 18:55:19 bugg-v3 python[665]: RPiID-10000000b1bab292 - Checking if modem is enumerated...
Apr 03 18:55:19 bugg-v3 python[665]: INFO:buggd.drivers.modem:Checking if modem is enumerated...
Apr 03 18:55:17 bugg-v3 python[665]: RPiID-10000000b1bab292 - Checking if modem is enumerated...
Apr 03 18:55:17 bugg-v3 python[665]: INFO:buggd.drivers.modem:Checking if modem is enumerated...
Apr 03 18:55:15 bugg-v3 python[665]: RPiID-10000000b1bab292 - Checking if modem is enumerated...
Apr 03 18:55:15 bugg-v3 python[665]: INFO:buggd.drivers.modem:Checking if modem is enumerated...
Apr 03 18:55:13 bugg-v3 python[665]: RPiID-10000000b1bab292 - POWER_ON_N asserted, waiting for modem to boot up...
Apr 03 18:55:13 bugg-v3 python[665]: INFO:buggd.drivers.modem:POWER_ON_N asserted, waiting for modem to boot up...
Apr 03 18:55:11 bugg-v3 python[665]: RPiID-10000000b1bab292 - Turning on 3.7V rail.
Apr 03 18:55:11 bugg-v3 python[665]: INFO:buggd.drivers.modem:Turning on 3.7V rail.
Apr 03 18:53:45 bugg-v3 python[665]: RPiID-10000000b1bab292 - 2024-04-03T18_50_12.850Z - Finished audio compression
Apr 03 18:53:45 bugg-v3 python[665]: INFO:buggd.sensors.externalmic:2024-04-03T18_50_12.850Z - Finished audio compression
Apr 03 18:53:34 bugg-v3 python[971]: Recording WAVE '/tmp/rpi-ecosystem-monitoring_tmp/currentlyRecording.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Stereo
Apr 03 18:53:34 bugg-v3 sudo[970]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=0)
Apr 03 18:53:34 bugg-v3 sudo[970]:     root : PWD=/ ; USER=root ; COMMAND=/usr/bin/arecord --device plughw:0,0 --channels 2 --rate 48000 --format S16_LE --duration 201>
Apr 03 18:53:34 bugg-v3 python[665]: RPiID-10000000b1bab292 - Started recording stereo from internal and external microphones at 2024-04-03T18_53_35.877Z for 200s
Apr 03 18:53:34 bugg-v3 python[665]: INFO:buggd.sensors.externalmic:Started recording stereo from internal and external microphones at 2024-04-03T18_53_35.877Z for 200s
Apr 03 18:53:34 bugg-v3 python[665]: RPiID-10000000b1bab292 - Capturing data from sensor
Apr 03 18:53:34 bugg-v3 python[665]: INFO:root:Capturing data from sensor
Apr 03 18:53:34 bugg-v3 python[665]: RPiID-10000000b1bab292 - GLOB_no_sd_mode: False, GLOB_is_connected: True, GLOB_offline_mode: False
Apr 03 18:53:34 bugg-v3 python[665]: INFO:root:GLOB_no_sd_mode: False, GLOB_is_connected: True, GLOB_offline_mode: False

1) Test: Running buggd manually, not from systemd: Result: same - it's not systemd.
2) Observation: the first couple of prints are normal, until logging to disk begins
3) Test: Installing buggd myself, rather than in the buggOS build. Source venv, pip remove, pip install from git, run again. Result: same
4) Conclusion, it's not packaging, its our codebase. 
5) Test: do our logs actually get written to disk - maybe they're failing over to stdout. Result: yes, logging to disk is working
6) Refactor: stop using the default logger