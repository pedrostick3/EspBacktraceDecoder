# EspBacktraceDecoder

Python Script to decode ESP8266 and ESP32 Exceptions and Backtraces.


## License

GPL 3.0


## Usage:

```
usage: decoder.py [-h] [-p {ESP8266,ESP32}] [-t TOOL] -e ELF [-f] file

decode ESP Stacktraces.

positional arguments:
  file                  The file to read the exception data from ('-' for
                        STDIN)

optional arguments:
  -h, --help            show this help message and exit
  -p {ESP8266,ESP32}, --platform {ESP8266,ESP32}
                        The platform to decode from
  -t TOOL, --tool TOOL  Path to the xtensa toolchain
  -e ELF, --elf ELF     path to elf file
  -f, --full            Print full stack dump
```

The tool is the path to your xtensa toolchain. If you use [PlatformIO](http://platformio.org/) it should be `~/.platformio/packages/toolchain-xtensa` for the ESP8266 and `~/.platformio/packages/toolchain-xtensa32` for the ESP32.

The elf path is the path to your built elf binary. On PlatformIO it is located at `<project-dir>/.pio/build/<environment-name>/firmware.elf`.


## Dependencies:

* python 3 or 2.7
* The xtensa toolchain for your ESP


## Example:

Given you have the following stacktrace from the ESP:

```
Guru Meditation Error: Core  0 panic'ed (LoadProhibited). Exception was unhandled.
Core 0 register dump:
PC      : 0x400e271a  PS      : 0x00060430  A0      : 0x800de7ca  A1      : 0x3ffd03e0  
A2      : 0x00000110  A3      : 0x3ffd0424  A4      : 0x3ffc48dc  A5      : 0x3ffe891a  
A6      : 0x3ffef488  A7      : 0x00000000  A8      : 0x3ffd0435  A9      : 0x3ffd03d0  
A10     : 0x3ffd042c  A11     : 0x3f401ce5  A12     : 0x3f401cee  A13     : 0x0000ff00  
A14     : 0x00ff0000  A15     : 0xff000000  SAR     : 0x00000008  EXCCAUSE: 0x0000001c  
EXCVADDR: 0x0000014c  LBEG    : 0x4000c349  LEND    : 0x4000c36b  LCOUNT  : 0xffffffff  

ELF file SHA256: 0000000000000000

Backtrace: 0x400e271a:0x3ffd03e0 0x400de7c7:0x3ffd0420 0x400da136:0x3ffd0460 0x400de97d:0x3ffd04b0 0x400e32c5:0x3ffd0760 0x400e1745:0x3ffd0780 0x400e1b89:0x3ffd07d0 0x400e1146:0x3ffd07f0 0x400df8c1:0x3ffd0850 0x40106375:0x3ffd0870 0x400fe02e:0x3ffd08b0 0x40090afe:0x3ffd08e0
```

You can dump it in a file called `backtrace.txt` and run the `decode.py` script:
(don't forget to replace the `<...>` with your file path)

```
$ python3 <...>/decoder.py <...>/backtrace.txt
```

OR if you don't want to use the default platformio path to the `firmware.elf` you can run:

```
$ python3 <...>/decoder.py -e <...>/firmware.elf backtrace.txt
```

This prints the processed stacktrace:

```
stack:
0x400e271a: FreeRTOS::Semaphore::take(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/FreeRTOS.cpp:191
0x400de7c7: BLECharacteristic::setValue(unsigned char*, unsigned int) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLECharacteristic.cpp:781
0x400da136: AssayStatusCallBacks::onWrite(BLECharacteristic*) at C:\Users\pedrostick\Documents\GitKraken\doctorvida-pocket-firmware\Firmware\doctorvida-pocket_esp32/src\4 - Application\BLECallbacks/BLECallbacks_AssayStatus.cpp:44
0x400de97d: BLECharacteristic::handleGATTServerEvent(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLECharacteristic.cpp:781
0x400e32c5: BLECharacteristicMap::handleGATTServerEvent(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLECharacteristicMap.cpp:87
0x400e1745: BLEService::handleGATTServerEvent(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLEService.cpp:390
0x400e1b89: BLEServiceMap::handleGATTServerEvent(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLEServiceMap.cpp:93
0x400e1146: BLEServer::handleGATTServerEvent(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLEServer.cpp:113
0x400df8c1: BLEDevice::gattServerEventHandler(esp_gatts_cb_event_t, unsigned char, esp_ble_gatts_cb_param_t*) at C:\Users\pedrostick\.platformio\packages\framework-arduinoespressif32\libraries\BLE\src/BLEDevice.cpp:591
0x40106375: btc_gatts_cb_to_app at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/bt/bluedroid/btc/profile/std/gatt/btc_gatts.c:54
  \-> inlined by: btc_gatts_cb_handler at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/bt/bluedroid/btc/profile/std/gatt/btc_gatts.c:811
0x400fe02e: btc_task at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/bt/common/btc/core/btc_task.c:163
0x40090afe: vPortTaskWrapper at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port.c:355 (discriminator 1)
```


## Acknowledgement
This is heavily inspired by [EspExceptionDecoder](https://github.com/me-no-dev/EspExceptionDecoder).
