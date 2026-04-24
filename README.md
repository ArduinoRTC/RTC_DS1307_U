# DS1307
This library is part of an initiative to create a unified API for handling RTCs,
and it serves as a library (device driver) for the [DS1307][DS1307],
which is also equipped with [Grove RTC][GroveRTC].

Previously, it was implemented as a wrapper for the [https://github.com/Seeed-Studio/RTC_DS1307][github] library for Grove RTC, but I’ve now made it completely custom.


## operation verification
I used Akizuki Denshi's [DS1307 Real-Time Clock (RTC) Module Kit] [https://akizukidenshi.com/catalog/g/gK-15488/].

I have confirmed that this works on the models listed in the table below.
| CPU | Model | Support Status |
|---|---|---|
| AVR | Arduino Mega | ✓ |
| SAMD | Arduino MKR WiFi1010 | ✓ |
| SAM | Arduino Due | ✓ |
| ESP32 | Switch Science ESP Developer32 | ✓ |


## External links
- DS1307 - [https://www.maximintegrated.com/jp/products/analog/real-time-clocks/DS1307.html][DS1307]
- Grove RTC - [https://www.seeedstudio.com/Grove-RTC.html][GroveRTC]
- Seeed studio RTC_DS1307 library [https://github.com/Seeed-Studio/RTC_DS1307][github]


## Precautions for Use

The `RTC_DS1307_U.h` file contains flags that enable features used for debugging and performance testing, as shown below.
Enable or disable these flags as needed.
```
#define DEBUG
```


## Sample Program
The sample programs are designed to verify whether the RTC registers are set correctly when each function of this driver is executed.

Please enable the `DEBUG` definition in `RTC_DS1307_U.h`
before compiling and installing.

Disabling the following line will cause the system to display detailed messages and check the contents of the registers.
```
#undef DEBUG 
```

If you enable the following line, the contents of all registers will be dumped after each test has written to the registers.
```
#define DUMP_REGISTER  // Enable this if you want to view a dump of the register values after they have been modified (also enable DEBUG)
```


## API
Please also refer to the maxim datasheet.
## Initialization
### Object creation
```
RTC_DS1307_U(TwoWire *theWire, int32_t rtcID = -1)
```
Create an object by specifying the I2C I/F and ID used by RTC.
|Argument|Description|
|---|---|
|theWire|I2C I/F|
|rtCID|Used to assign an ID to the RTC. (Default value is -1)|


### Initialization
```
bool  begin(bool init, uint32_t addr=RTC_DS1307_DEFAULT_I2C_ADDR)
```
The first argument is a flag indicating whether to configure the time, timer, or alarm; if it is set to ``false``, only I2C initialization is performed.

If you are using a module with an I2C address different from default of DS1307 , specify it as the second argument. If not specified, initialization will be performed using the default address.

| Return Value | Meaning |
|---|---|
|true|Initialization successful|
|false|Initialization failed|

## Retrieving RTC Information
A member function that retrieves information about the type and features of the RTC chip.
```
void  getRtcInfo(rtc_u_info_t *info)
```

## Time-related matters
### Time Settings
```
bool  setTime(date_t* time)
```
Sets the RTC to the time specified in the argument.
| Return Value | Meaning |
|---|---|
|true|Set successfully|
|false|Set failed|

### Getting the time
```
bool  getTime(date_t* time)
```
Stores the time information obtained from the RTC in a structure passed as an argument.
| Return Value | Meaning |
|---|---|
|true|Retrieval successful|
|false|Retrieval failed|

## Frequency signal output 
Since the DS1307 has only one terminal/function that outputs a frequency signal, the value of the first argument in the following function must be 0.
### Output Settings
```
int   setClockOut(uint8_t num, uint8_t freq, int8_t pin=-1)
```

Since the DS1307 does not have the capability to control the output of the periodic signal based on an external input signal, the third argument is ignored.

The values for the second argument are shown in the table below.

| Value of `freq` | Clock frequency |
|---|---|
|0|1 Hz|
|1|4 kHz|
|2|8 kHz|
|3|32 kHz|

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_UNSUPPORTED | Unsupported parameter configuration, etc. |

### Clock Frequency Settings
```
int   setClockOutMode(uint8_t num, uint8_t freq)
```
It is identical to `setClockOut()` except for the third argument, `pin`; since `pin` is ignored in `setClockOut()`,
it behaves the same as `setClockOut()`.

The second argument is the same as for `setClockOut()`, as shown in the table below.
|``freq`` value|Clock frequency|
|---|---|
|0|1 Hz|
|1|4 kHz|
|2|8 kHz|
|3|32 kHz|

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_UNSUPPORTED | Unsupported parameter configuration, etc. |


### Clock Output Control
```
int   controlClockOut(uint8_t num, uint8_t mode)
```
|“Mode” Value|Meaning|
|---|---|
|0|Clock output stopped|
|1|Clock output started|

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_UNSUPPORTED | Unsupported parameter configuration, etc. |

## Pause/Resume the timer
This RTC has a function to pause and resume the time count-up in order to reduce power consumption. (The most significant bit “CH” of register number 00h)

### reference of condition of the clock
```
int   clockHaltStatus(void)
```
You can check whether the power supply voltage has dropped (power failure d.) or whether the timer has been manually stopped.

|Return Value|Meaning|
|---|---|
|0|The clock is running|
|1|The clock has stopped|
|RTC_U_FAILURE|Registry read failure|

### Control of the timing clock
```
int   controlClockHalt(uint8_t mode)
```
A function to stop/resume the clock's operation.

| “Mode” Value | Meaning |
|---|---|
|0 | Clock stopped |
|1 | Clock resumed |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |

### Access to SRAM Area
On the DS1307, the 56 registers starting from register number ``0x08`` are available as SRAM area.
If the first argument ``addr`` of the following two functions is 0, data is read from register number ``0x08``. Additionally, an error occurs if the sum of the first argument ``addr`` and the third argument ``len`` exceeds the number of registers.
 
### Reading from SRAM Area
```
int getSRAM(uint8_t addr, uint8_t *array, uint16_t len)
```
Reads consecutively `len`  data starting from the first argument `addr`.
 
### Writing to SRAM Area
```
int setSRAM(uint8_t addr, uint8_t *array, uint16_t len)
```
 
Writes `len` data items starting from the first argument, `addr`, consecutively to registers that can be used as SRAM area.


[DS1307]:https://www.maximintegrated.com/jp/products/analog/real-time-clocks/DS1307.html
[GroveRTC]:https://www.seeedstudio.com/Grove-RTC.html
[github]:https://github.com/Seeed-Studio/RTC_DS1307
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811/
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/

<!--- コメント

## 動作検証
5V専用

|CPU| 機種 |ベンダ| 結果 | 備考 |
| :--- | :--- | :--- | :---: | :--- |
|AVR| [Uno R3][Uno]  |[Arduino][Arduino]|   ○   |      |
|       | [Mega2560 R3][Mega] |[Arduino][Arduino] |  ○    |      |
|       | [Leonardo Ethernet][LeonardoEth] |[Arduino][Arduino] |   ○  |      |
|       | [Uno WiFi][UnoWiFi] |[Arduino][Arduino] |    ○  | |
|       | [Pro mini 3.3V][ProMini] | [Sparkfun][Sparkfun] |      |      |
| ARM/M0+ | [M0 Pro][M0Pro] |[Arduino][Arduino] |||
|ESP8266|[ESPr developer][ESPrDev]| [スイッチサイエンス][SwitchScience] |||
|ESP32 | [ESPr one 32][ESPrOne32] | [スイッチサイエンス][SwitchScience] ||　|

## 外部リンク

- Seeed Studio技術Wiki [http://wiki.seeedstudio.com/Grove-3-Axis_Digital_Accelerometer-1.5g/][SeedWiki]
- センサ商品ページ [https://www.seeedstudio.com/Grove-3-Axis-Digital-Accelerometer-1-5-p-765.html][ProductPage]
- ソースリポジトリ [https://github.com/Seeed-Studio/Accelerometer_MMA7660][github]
- Adafruit Unified Sensor Driver - [https://github.com/adafruit/Adafruit_Sensor][AdafruitUSD]
- Groveシールド - [https://www.seeedstudio.com/Base-Shield-V2-p-1378.html][shield]
- Arduino M0 Pro - [https://store.arduino.cc/usa/arduino-m0-pro][M0Pro]
- Arduino Due - [https://store.arduino.cc/usa/arduino-due][Due]
- Arduino Uno R3 - [https://store.arduino.cc/usa/arduino-uno-rev3][Uno]
- Arduino Uno WiFi - [https://store.arduino.cc/usa/arduino-uno-wifi-rev2][UnoWiFi]
- Arduino Leonardo Ethernet - [https://store.arduino.cc/usa/arduino-leonardo-eth][LeonardoEth]
- Arduino Mega2560 R3 - [https://store.arduino.cc/usa/arduino-mega-2560-rev3][Mega]
- Arduino Pro mini 328 - 3.3V/8MHz - [https://www.sparkfun.com/products/11114][ProMini]
- ESPr developer - [https://www.switch-science.com/catalog/2652/][ESPrDev]
- ESPr Developer用GROVEシールド - [https://www.switch-science.com/catalog/2811/][ESPrDevShield]
- ESpr one - [https://www.switch-science.com/catalog/2620/][ESPrOne]
- ESPr one 32 - [https://www.switch-science.com/catalog/3555/][ESPrOne32]
- Grove - [https://www.seeedstudio.io/category/Grove-c-1003.html][Grove]
- Seed Studio - [https://www.seeedstudio.io/][SeedStudio]
- Sparkfun Electronics - [https://www.sparkfun.com/][Sparkfun]
- スイッチサイエンス - [https://www.switch-science.com/][SwitchScience]


[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[ProductPage]:https://www.seeedstudio.com/Grove-3-Axis-Digital-Accelerometer-1-5-p-765.html
[SeedWiki]:http://wiki.seeedstudio.com/Grove-3-Axis_Digital_Accelerometer-1.5g/
[github]:https://github.com/Seeed-Studio/Accelerometer_MMA7660
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/


[Adafruit Unified Sensor Driver][AdafruitUSD]
[Groveシールド][shield]
[Arduino M0 Pro][M0Pro]
[Arduino Due][Due]
[Arduino Uno R3][Uno]
[Arduino Mega2560 R3][Mega]
[Arduino Leonardo Ethernet][LeonardoEth]
[Arduino Pro mini 328 - 3.3V/8MHz][ProMini]
[ESpr one][ESPrOne]
[ESPr one 32][ESPrOne32]
[Grove][Grove]
[Seed Studio][SeedStudio]
[Arduino][Arduino]
[Sparkfun][Sparkfun]
[スイッチサイエンス][SwitchScience]
--->
