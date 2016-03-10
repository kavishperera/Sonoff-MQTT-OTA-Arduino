## Sonoff-MQTT-OTA-Arduino
Provide ESP8266 based [itead Sonoff](https://www.itead.cc/sonoff-wifi-wireless-switch.html) with MQTT and 'Over the Air' or OTA firmware using Arduino IDE.

See [Sonoff-MQTT-OTA](https://github.com/arendst/Sonoff-MQTT-OTA) for the ```esp-open-sdk``` version.
## Prerequisite
Install the ESP8266 Arduino development environment from [esp8266 Arduino](https://github.com/esp8266/Arduino).

- The software is tested on a Windows PC with Arduino IDE version 1.6.5r5 and esp8266 Arduino version 2.1.0
- I prefer a standalone version of the IDE allowing easy ESP8266 file manipulation. This can be achieved by downloading the Arduino IDE ZIP file for non admin install. After unzipping and before executing ```arduino.exe``` add an empty directory called ```portable```
- Follow the procedure to install the ESP8266 Arduino development environment
- Copy the ```sonoff``` directory to your sketchfolder

Install php and a local web server (ie apache) for OTA and copy directory ```api``` in webroot.

If you want to be able to upload the OTA file from the IDE to your webserver perform the following changes to the Arduino IDE environment:

- Copy file ```espupload.py``` to directory ```portable\packages\esp8266\hardware\esp8266\2.1.0\tools```
- Replace files ```boards.txt``` and ```platform.txt``` in directory ```portable\packages\esp8266\hardware\esp8266\2.1.0```

This will provide an additional option ```Tools - Upload Using: OTA_upload```.
## Compile and upload
For sonoff select from ```Tools Board Generic ESP8266 Module``` and set the following options:

- Flash Mode: QIO
- Flash Frequency: 40MHz
- Upload Using: Serial
- CPU Frequency: 80MHz
- Flash Size: 1M (64K SPIFFS)
- Debug Port: Disabled
- Debug Level: None
- Reset Method: ck
- Upload Speed: 115200
- Port: Your COM port connected to sonoff

Update ```sonoff.ino``` with your specific Wifi and MQTT parameters.

Verify source and upload once to sonoff using the serial connection as shown in [Peter Scargill's blog](http://tech.scargill.net/itead-slampher-and-sonoff). **Do not connect AC power during the flash cable connection**. 

Verify and upload an OTA image to your web server with option ```Upload Using: OTA_upload```.
## Usage
The button on sonoff provides the following features:

- a short press toggles the relay. This will blink the LED twice and sends a MQTT status message like ```stat/sonoff/POWER on```
- three short presses start Wifi smartconfig which allows for SSID and Password configuration using an Android mobile phone with the [ESP8266 SmartConfig](https://play.google.com/store/apps/details?id=com.cmmakerclub.iot.esptouch) app. The green LED will blink during the smartconfig period for 100 seconds. The MQTT server still needs to be configured using the ```user_config.h``` file
- four short presses start OTA download of firmware. The green LED is lit during the update
- pressing the button for over four seconds resets settings to defaults as defined in ```user_config.h``` and reboots  sonoff

Sonoff responds to the following MQTT commands:

- the relay can be controlled by ```cmnd/sonoff/power on```, ```cmns/sonoff/power off``` or ```cmnd/sonoff/power toggle```. The LED will blink twice and sends a MQTT status message like ```stat/sonoff/POWER on```. The same function can be initiated with ```cmnd/sonoff/light on```
- the MQTT topic can be changed with ```cmnd/sonoff/topic sonoff1``` which reboots sonoff and makes it available for MQTT commands like ```cmnd/sonoff1/power on```
- the OTA firmware location can be made known to sonoff by ```cmnd/sonoff/otaurl http://sidnas2:80/api/sonoff/user1.bin``` where sidnas2 is your webserver hosting the firmware. Reset to default with ```cmnd/sonoff/otaurl 1```
- upgrade OTA firmware by ```cmnd/sonoff/upgrade 1```
- show status information by ```cmnd/sonoff/status 1```

Most MQTT commands will result in a status feedback like ```stat/sonoff/POWER On```.
## Commands supported
The firmware supports both a **serial** and a **MQTT** Man Machine interface. The serial interface is set to 115200 bps. The MQTT commands are constructed as ```cmnd/sonoff/<command>```. 

The following commands are recognised by both topic and grouptopic:

Command | Description
------- | -----------
power | Show current power state as On or Off
power on | Turn power On
power off | Turn power Off
power toggle | Toggle power
power 1 | Turn power On
power 0 | Turn power Off
power 2 | Toggle power
light | Show current power state as On or Off
light on | Turn power On
light off | Turn power Off
light toggle | Toggle power
light 1 | Turn power On
light 0 | Turn power Off
light 2 | Toggle power
status | Show abbreviated status information
status 1 | Show all status information
status 2 | Show version information
grouptopic | Show current MQTT group topic
grouptopic 1 | Reset MQTT group topic to ```user_config.h``` value and restart
grouptopic your-grouptopic | Set MQTT group topic and restart
timezone | Show current timezone
timezone -12 .. 12 | Set timezone

The following commands are recognised by topic only:

Command | Description
------- | -----------
restart 1 | Restart sonoff
reset 1 | Reset sonoff parameters to ```user_config.h``` values and restart
ssid | Show current Wifi SSId
ssid 1 | Reset Wifi SSId to ```user_config.h``` value and restart
ssid your-ssid | Set Wifi SSId and restart
password | Show current Wifi password
password 1 | Reset Wifi password to ```user_config.h``` value and restart
password your-password | Set Wifi password and restart
host | Show current MQTT host
host 1 | Reset MQTT host to ```user_config.h``` value and restart
host your-host | Set MQTT host and restart
topic | Show current MQTT topic
topic 1 | Reset MQTT topic to ```user_config.h``` value and restart
topic your-topic | Set MQTT topic and restart
smartconfig 1 | Start smart config
otaurl | Show current otaurl
otaurl 1 | Reset otaurl to ```user_config.h``` value
otaurl your-otaurl | Set otaurl
upgrade 1 | Download ota firmware from your web server and restart

If the same topic has been defined to more than one sonoff an individual sonoff can still be addressed by the fall back topic MQTT_CLIENT_ID as defined in user_config.h. The fall back topic will be ```DVES_<last six characters of MAC address>```.
## Tips
- To aid in finding the IP address of sonoff the network name will be ```ESP-<last six characters of MAC address>-<MQTT topic>```. So the default name is ```ESP-123456-sonoff```
- Use the group topic to address several sonoffs with one (restricted) MQTT command
