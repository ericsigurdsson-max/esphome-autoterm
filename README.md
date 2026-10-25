This [ESPHome](https://esphome.io/) External Component allows to monitor and control Autoterm/Planar Diesel Air Heaters from within [Home Assistant](https://www.home-assistant.io/).

## Features
- Heater On/Off
- Operating mode
  - By Heater temperature sensor
  - By Control Panel temperature sensor
  - By External temperature sensor
  - By Power
- Power level
- Temperature setpoint
- Ventilation On/Off
- Temperature sensors
  - Heater
  - Control Panel
  - External
- Battery voltage
- Experimental extended status telemetry: requested fan RPM, actual fan RPM, and fuel-pump pulse frequency (Hz)

## Supported Devices
- Autoterm Air 2D & 4D (Planar 2D & 44D)

## Wiring
> [!CAUTION]
> Logic level shifters required. Heater uses 5V logic level, ESP32 3.3V !  

<details><summary>Prototype with WT32-ETH01 & ADUM1201 as level shifters</summary>

### Control Panel
```
Panel Green (RX)  → ADUM1201 VOA → VIA → ESP32 GPIO17 (TX)
Panel White (TX)  → ADUM1201 VIB → VOB → ESP32 GPIO05 (RX)
```

### Heater
```
Heater Green (TX) → ADUM1201 VIB → VOB → ESP32 GPIO33 (RX)
Heater White (RX) → ADUM1201 VOA → VIA → ESP32 GPIO32 (TX)
```
</details>

## Home Assistant Integration with Native API
This component exposes:
- Heater, panel, external temperature sensors
- Battery voltage sensor
- Temperature setpoint (Number)
- Power level (Number)
- Operating mode (Select)
- Ventilation (Switch)
- Heater power (Switch)
- Operating state (Text sensor)
- Fan speed target and actual (RPM), fuel pump pulse frequency (Hz; optional diagnostic sensors)

## Experimental live telemetry

The following ESPHome configuration under `autoterm:` enables diagnostic data from a **19-byte** heater status (`0x0F`) response:

```yaml
  fan_speed_target:
    name: "Fan Speed Target"
  fan_speed_actual:
    name: "Fan Speed Actual"
  fuel_pump_frequency:
    name: "Fuel Pump Frequency"
```

The byte offsets and conversions were reverse-engineered for **Air 2D**, and have **not yet been validated on Air 4D**. The status parser only publishes these values when it receives a CRC-validated response with the expected 19-byte payload. A heater that uses a different format will leave these entities unavailable. Values are published on every valid status message (rather than the normal periodic refresh) to capture transient startup behavior.

These are diagnostic readings, **not a direct measurement of thermal output or the current 1–10 power stage**. Fuel pump frequency may help estimate thermal output, but needs validation/calibration for the specific heater before being converted to kW or a displayed power stage. No new commands are sent to the heater.

Protocol reference: [Air 2D serial-control notes](https://github.com/schroeder-robert/autoterm-air-2d-serial-control#readme).

## Optional MQTT
Optionally enable MQTT for basic integration.

<details><summary>Optional MQTT JSON Interface</summary>

### State Codes
```
0: off
1: starting
4: running
5: shutting down
6: testing environment
8: ventilation
```

### Mode Codes
```
1: by heater temp sensor
2: by panel temp sensor
3: by external temp sensor
4: by power level
```

### Values
```
power: 0–9
vent: 1=on, 2=off
temp_set: 1–30 °C
```

### Published Topic
```
<node>/autoterm/state
```
Example:
```
{"state":0,"mode":4,"temp_set":23,"power":3,"vent":2,"temp_heater":0,"temp_ext":8,"temp_panel":-2,"batt":26.9}
```

### Command Topic
```
<node>/autoterm/command
```
Commands:
```
1: start
2: apply settings
3: shutdown
```

Examples:
```
{"cmd":1,"mode":4,"temp_set":23,"power":3,"vent":2}
{"cmd":1}
{"cmd":2,"mode":3,"temp_set":22}
{"cmd":3}
```
</details>

## TODO
- Replace magic numbers with enums


## Disclaimer
> [!WARNING]
> This project is provided “AS IS” without any warranty of any kind.  
> You use this software at your own risk.  
> The authors and contributors are not responsible for any damage, malfunction,  
> injury, or legal issues resulting from the use of this code.
>
> For full legal terms, see the Apache-2.0 license included in this repository.

## Thanks to
- https://grimoire314.wordpress.com/2018/08/22/planar-diesel-heater-controller-reverse-engineering/ 
- https://grimoire314.wordpress.com/2019/03/21/autoterm-planar-diesel-heater-controller-reverse-engineering-part-2/
- https://helloworld.schlussdienst.net/blog/hacking-autoterm-planar-2d
- https://github.com/csreades/AutothermDieselRepeater
