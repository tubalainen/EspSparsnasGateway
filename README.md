# EspSparsnasGateway

This is a Mqtt Gateway for Ikeas energy monitor Sparsnas. The monitor 
sends encoded data by radio to a control panel. This device collects the data 
and sends it to Mqtt-enabled receivers in Json-format. Thus you need a Mqtt broker 
in your network and adjust the settings  in the ino-file:

```
// Settings for the Mqtt broker:
#define MQTT_USERNAME "<username>"    // If used by the broker     
#define MQTT_PASSWORD "<password>"     //    -   *   -
const char* mqtt_server = "192.168.1.79";  // Mqtt brokers IP
```

The data is also printed to the seriaĺ port. If the reception is bad, the received data can be bad. 
This gives a CRC-error, the data is in this case not sent via Mqtt but printed via the serial port. 

The data sent via Mqtt is in Json format and looks like this:

```
{"seq":13688,"power":2024,"total":1821.344,"battery":100}
```

## Hardware
The hardware used is a Esp8266-based wifi-enabled Mcu. You can use different devices like a Wemos Mini or a Nodemcu, but take care of the Gpio labels that can differ. The receiver is a RFM69B radio transciever. I use a 868MHz device, but a 900MHz should work as well. To this a simple antenna is connected, I use a straight wire, 86 millimeters long connected to the RFM's Ant-connection. The wire shall be vertical, standing up. You can also add a similar wire to the gnd-connection next to the antenna connection, pointing down, opposite to the first wire. 

The connection for the RFM69 is hardcoded. This is standard Spi connections set in the spi-library that can't be changed. See https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/using-the-arduino-addon. 

The schematic shows a Nodemcu, but you can use another ESP8266-based device if you want (except the Esp-01). Use these pin mappings:

```
NodeMcu - Esp12
D1	- Gpio5
D5	- Gpio14
D6	- Gpio12
D7	- Gpio13
D8	- Gpio15
```

## Hardware hacks to ensure good RF performance.
Also add two capacitors, 330-470uF and 100nF, to Vin in and Gnd for stability.
Keep the wires between the RFM69 module and the NodeMCU as short as possible and DO NOT make them 8 cm long hence that calculates into 1/4 wavelenth of 868 MHz.
You will experiance interference and very poor performance if the above is not applied and followed.

If you want to learn more about the Rfm69 and get some tips & tricks, look at https://learn.sparkfun.com/tutorials/rfm69hcw-hookup-guide.

## If it doesn't work
As usual, check your connections one more time. If possible, solder the connections. Also make sure to use a good power supply, both the Esp and the Rfm69 want's that. 

### Connect to computer
You can use the device with a simple USB power supply and get data via Mqtt. The device also puts out more information via the serial port. You can connect it to a computer and look at the messages with a serial monitor, for example the one in the Arduino IDE or Minicom. The baudrate is 115200.

### Enable debug
Further more information is given by the device if debug is activated:

```
#define DEBUG 1
```

### Change the channel filter width
With some RFM's a software adjustment can be tested if the code doesn't work. Line 191 looks like this:

```
/* 0x19 */ {REG_RXBW, RF_RXBW_DCCFREQ_010 | RF_RXBW_MANT_16 | RF_RXBW_EXP_4}, // p26 in datasheet, filters out noise
```

You can try to change this to:

```
/* 0x19 */ {REG_RXBW, RF_RXBW_DCCFREQ_010 | RF_RXBW_MANT_16 | RF_RXBW_EXP_3}, // p26 in datasheet, filters out noise
```

This makes the channel filter wider, 62.5khz instead of 31.3khz.
 

## Home Assistant integration
The Mqtt data can be used anywhere, here's an example for the Home Automation software Home Assistant.
In Home Assistant the sensors can look like this:

```
\#Sparnas energy monitor
  - platform: mqtt
    state_topic: "EspSparsnasGateway/values"
    name: "House energy usage"
    unit_of_measurement: "W"
    value_template: '{{ float(value_json.power) | round(0)  }}'
    
  - platform: mqtt
    state_topic: "EspSparsnasGateway/values"
    name: "House energy meter batt"
    unit_of_measurement: "%"
    value_template: '{{ float(value_json.battery) }}'
```

We then get these sensors: 

```
-sensor.house_energy_meter_batt
-sensor.house_energy_usage
```

The result can be seen in SparsnasHass.png. 

![alt text](https://github.com/bphermansson/EspSparsnasGateway/blob/master/SparsnasHass.png "Sparsnas in Home Assistant")

## Thanks!
The code is based on Sommarlovs version of Ludvig Strigeus code:
http://elektronikforumet.com/forum/viewtopic.php?f=2&t=85006&start=255
Strigeus original code for use with a DVB-T Usb dongle:
https://github.com/strigeus/sparsnas_decoder

