# Lumi MQTT

Translated version (readme and logging messages) of [Beetle-II/lumi](https://github.com/Beetle-II/lumi).

MQTT agent for Xiaomi gateway DGNWG05LM or ZHWG11LM with [OpenWRT 19.07.7](https://github.com/openlumi/openwrt/tags) firmware.  
Allows you to interact with the gateway via MQTT.

Interaction | MQTT topic, getting | MQTT topic, control
--- | --- | ---
Built in light sensor | lumi/illumination
Illumination | lumi/lamp | lumi/lamp/set
lumi/illumination | | lumi/alarm/set
Button | lumi/button/action
Playable url, volume | lumi/audio/play | lumi/audio/play/set
Volume | lumi/audio/volume | lumi/audio/volume/set
Voice memo | | lumi/say/set
BLE devices | lumi/{MAC} |

{MAC} is the address of the Bluetooth device.

---

### Problems noticed:

Some gateways have problems with WiFi operation when BLE device scanning is enabled.  
The connection to the gateway becomes unstable.

---
You need node.js, git, mpc packages to download and work

Add a repository with the latest versions of Node and install the necessary packages:

```
wget https://openlumi.github.io/openwrt-packages/public.key -O /tmp/public.key
opkg-key add /tmp/public.key
echo 'src/gz openlumi https://openlumi.github.io/openwrt-packages/packages/19.07/arm_cortex-a9_neon' >> /etc/opkg/customfeeds.conf

opkg update && opkg install node git-http mpc mpd-full

/etc/init.d/mpd start
```

Download:

```
mkdir /opt
cd /opt
git clone https://github.com/danielk117/lumi.git
cd lumi
cp config_example.json config.json
```

Change the config.json configuration file Specify your server address, login and password

```json
{
  "sensor_debounce_period": 300,
  "sensor_treshhold": 50,
  "button_click_duration": 300,
          
  "homeassistant": true,
  "use_ble": false,
  "tts_cache": true,
  "sound_channel": "Master",
  "sound_volume": 50,
  "mqtt_url": "mqtt://адрес вашего сервера",
  "mqtt_topic": "lumi",
  "use_mac_in_mqtt_topic": false,
  "mqtt_options": {
    "port": 1883,
    "username": "логин сюда",
    "password": "пароль сюда",
    "keepalive": 60,
    "reconnectPeriod": 1000,
    "clean": true,
    "encoding": "utf8",
    "will": {
      "topic": "lumi/state",
      "payload": "offline",
      "qos": 1,
      "retain": true
    }
  }
}
```

Parameter | Description
--- | ---
"homeassistant": true | notify MQTT broker of gateway devices. Helps to add devices to HomeAssistant
"use_ble": false | enable scanning and sending data from BLE devices
||
"tts_cache": true | cache TTS files after playback
||
"sound_channel": "master" | channel for sound output
"sound_volume": 50 | default volume
||
"sensor_debounce_period": 300 | period of sending data about the state of the devices (in seconds)
"sensor_treshhold": 50 | threshold for changing the state of the sensor, to send data instantly
"button_click_duration": 300 | time in ms between button clicks.
||
"use_mac_in_mqtt_topic": true | add MAC gateway to MQTT topics

Run:

```
node /opt/lumi/lumi.js
```

Check if there is data from the sensors and add to the autorun:

```
cd /opt/lumi
chmod +x lumi
cp lumi /etc/init.d/lumi
/etc/init.d/lumi enable
/etc/init.d/lumi start
```

---

### Update to the current version:

```
/etc/init.d/lumi stop
cd /opt/lumi
git pull
/etc/init.d/lumi start
```

---

### Example commands:

Topic | Value | Description
---|---|---
lumi/light/set | {"state": "ON"} | Turn on the backlight
lumi/light/set | {"state": "ON", "color":{"r":50, "g":50, "b":50}} | Turn on the backlight with a specified color
lumi/light/set | {"state": "ON", "timeout": 30} | Switch the light on and off after a specified time (sec)
lumi/light/set | {"state": "OFF"} | Turn off the backlight
||
lumi/audio/play/set |"https://stream.url/radio" | Play stream
lumi/audio/play/set | "/tmp/test.mp3" | Play local audio file
lumi/audio/play/set | {"url": "https://stream.url/radio", "volume": 50} | Play stream at volume 50
lumi/audio/play/set | "STOP" | Turn off playback
||
lumi/audio/volume/set | 30 | Change the volume to 30
||
lumi/say/set | "Hi" | Say 'Hi'
lumi/say/set | {"text": "Hi", "volume": 80} | Say 'Hi' with volume 80
lumi/say/set | {"text": "Hello", "lang": "en"} | Say 'Hello'
||
lumi/alarm/set | {"state": "ON"} | Turn on flashing light
lumi/alarm/set | {"state": "ON", "color":{"r":50, "g":50, "b":50}} | Turn on the flashing lamp with the color you specified
lumi/alarm/set | {"state": "ON", "time": 1} | Turn on the flashing of the lamp at a frequency of 1 sec
lumi/alarm/set | {"state": "ON", "count": 5} | Switch on blinking of the lamp 5 times, then switch off
lumi/alarm/set | {"state": "OFF"} | Switch off the flashing lamp
