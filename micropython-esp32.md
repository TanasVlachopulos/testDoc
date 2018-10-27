# MicroPython ESP32

## Flash firmware

Latest firmware for ESP32 and ESP8266 boards can be found at official MicroPython website [http://micropython.org/download\#esp32](http://micropython.org/download#esp32)

{% hint style="info" %}
 I am using Ubuntu 18 in Windows Subsystem for Linux, some steps are specific for WSL environment.
{% endhint %}

* Install ESP tools `sudo pip3 install esptool`
* Connect device over USB and check serial port number in Windows device manager, Windows COM ports are mapped to linux serial devices, e.g. COM5 = /dev/ttyS5. On Linux you can list recently connected devices via `dmsg`.
* Check if connected device communicate correctly `sudo esptool.py --port /dev/ttyS5 flash_id`
* You will get similar result:

```bash
esptool.py v2.5.1
Serial port /dev/ttyS5
Connecting........_
Detecting chip type... ESP32
Chip is ESP32D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core
MAC: 30:ae:a4:38:6e:20
Uploading stub...
Running stub...
Stub running...
Manufacturer: c8
Device: 4016
Detected flash size: 4MB
Hard resetting via RTS pin...
```

* Before uploading firmware is good to erase flash memory `sudo esptool.py --port /dev/ttyS5 erase_flash`
* Flash firmware  for **ESP32**: `esptool.py --chip esp32 --port /dev/ttyS5 write_flash -z 0x1000 esp32-20181007-v1.9.4-631-g338635ccc.bin` for ESP8266: `esptool.py --chip esp8266 --port /dev/ttyUSB1 write_flash --flash_size=detect 0 esp8266-20180511-v1.9.4.bin`Specifically for Sonoff switches with ESP8266: `esptool.py --chip esp8266 --port /dev/ttyUSB0 write_flash -fs 1MB -fm dout 0x0 esp8266-20180511-v1.9.4.bin`

Sonoff switches can be recognized by flash manufacturer. As result of flash\_id you will get  _Manufacturer "5e"_   


### Connect to device

MicroPython provide some kind of terminal called REPR \(Read Eval Print Look\) it is basically interactive python shell.

You can connect to shell via any serial terminal, but you are not able to transfer files between computer and ESP. For better interaction there are a few tools. [Adafruit ampy](https://github.com/adafruit/ampy) can upload, download, list and delete files and can run script on device. More advanced tool is [rshell](https://github.com/dhylands/rshell), provide interactive shell with ability to transfer files.

{% hint style="warning" %}
I was not able to run ampy or rshell under Windows. In WSL and PS I am getting error related to access to serial port.
{% endhint %}

* Install rshell `pip3 install rshell`
* Connect to device \(exit with ctrl+D\) `rshell -p /dev/ttyUSB0`
* List available boards `boards`
* Open Python script terminal on board \(exitv with ctrl+X\) `repl`
* You can list content of board storage with  `ls /pyboard`
* You can use also do `cd, cat, cp, edit, mkdir, rm, rsync`
* Alternatively you can connect with minicom `minicom --baudrate 115200 --device /dev/ttyUSB1`



