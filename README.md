# reclaim-eco-r290-home-assistant-local
This repository documents my findings when I torn down my new Reclaim Eco R290 300L Wi-Fi connected heat pump water heater.

It contains the following information:
1. Access to Controller Instructions
2. Controller Internal Photos
3. Controller components guides
4. Tuya datapoint IDs and their functions of the MCU
5. Converting the hot water tank from Tuya to ESPHome

## Access to Controller Instructions
1. Switch off power, wait until everything spins down (compressor, fan, etc) before unplugging the heater
2. Remove the top 4 screws on the front of the tank
3. Lift up the top cover panel from the waterproof flange at around 45 deg
4. Pull panel towards ground when lifted up
5. Press latch on connector, disconnect controller connector (4 pin)
6. Remove the 4 holding down the fixing bracket
7. Open the controller door, remove the controller from the front of the cover

## Controller Internal Photos

## Controller Components Guides

## Tuya Datapoints Explained
*Warning: This section is AI generated based on my digging, datapoints are accurate*

This section details the Tuya data point mapping for the hot water heat pump controller. Use these definitions to configure custom firmware, Home Assistant entities, or local integration scripts.

Some are in Chinese as besides the tank the Reclaim Eco R290 use off-the-shelf Chinese parts. (Which is good because replacement parts are very easy to find.)

* **DP 1: Power (`ON/OFF`)**
* Type: Boolean
* Direction: Send and report
* Values: `true` (on), `false` (off)
* Description: Controls and reports the primary operational power state of the unit.


* **DP 2: Target Temperature (`Set Temp`)**
* Type: Integer
* Direction: Send and report
* Range: 10 to 65 °C (Step: 1, Scale: 0)
* Description: Configures the target hot water setpoint temperature.


* **DP 3: Current Water Temperature (`CURRENT TEMPERATURE`)**
**DOES NOT AUTO UPDATE, USE DP 107**
* Type: Integer
* Direction: Report only
* Range: -500 to 1500 (Step: 1, Scale: 0)
* Description: Reports the current water temperature. Verify if host integration requires a scale divisor of 10 for true decimal display.


* **DP 4: Operating Mode (`Working Mode`)**
* Type: Enum
* Direction: Send and report
* Values:
* `0`: ECO Mode (热水模式 - standard heat pump heating)
* `1`: Boost Mode (速热模式 - simultaneous heat pump and auxiliary element)
* `2`: Holiday Mode (假日模式 - low-temperature frost protection standby)
* `3`: Rescue Mode (补救模式 - auxiliary element only during heat pump failure)


* Description: Selects the active operational profile.


* **DP 9: Fault Warning (`Failure warning`)**
* Type: Bitmap
* Direction: Report only
* Bit Length: 9
* Available Codes: `E01`, `E02`, `E03`, `E04`, `E05`, `E06`, `E07`, `E08`, `E11`
* Description: Bitmask reporting active diagnostic fault codes and safety trips.


* **DP 101: Hardware Timer Switch (`定时专用开关`)**
* Type: Boolean
* Direction: Send only
* Values: `true`, `false`
* Description: Dedicated trigger for the internal hardware timer schedule. MCU provides no state confirmation.


* **DP 102: Holiday Mode Duration (`Remaining holiday`)**
* Type: Integer
* Direction: Send and report
* Range: 2 to 199 Days
* Description: Sets and reports remaining duration for holiday standby mode.


* **DP 103: Compressor State (`Compressor`)**
* Type: Boolean
* Direction: Report only
* Values: `true` (running), `false` (idle)
* Description: Binary feedback of compressor operational status.


* **DP 104: Fan State (`Fan`)**
* Type: Boolean
* Direction: Report only
* Values: `true` (running), `false` (idle)
* Description: Binary feedback of evaporator fan operational status.


* **DP 105: Auxiliary Heating Element (`Heating Element`)**
* Type: Boolean
* Direction: Report only
* Values: `true` (energised), `false` (idle)
* Description: Binary feedback of backup electric resistive element status.


* **DP 106: Electronic Expansion Valve (`EEV Position`)**
* Type: Integer
* Direction: Report only
* Range: 0 to 600 Steps
* Description: Reports stepper motor position of the electronic expansion valve.


* **DP 107: Outlet / Tank Temperature (`Tank`)**
* Type: Integer
* Direction: Report only
* Range: -500 to 2000 °C
* Description: Reports water cylinder outlet/storage temperature sensor readings.


* **DP 108: Ambient Temperature (`Ambient`)**
* Type: Integer
* Direction: Report only
* Range: -50 to 100 °C
* Description: Reports outdoor ambient air intake temperature.


* **DP 109: Discharge Temperature (`Discharge`)**
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C
* Description: Reports compressor discharge refrigerant pipe temperature.


* **DP 110: Evaporator Coil Temperature (`Coil`)**
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C
* Description: Reports evaporator fin coil temperature for defrost monitoring.


* **DP 111: Suction Temperature (`Suction`)**
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C (Step: 1)
* Description: Reports compressor refrigerant suction line temperature.


* **DP 112: Defrost Solenoid Valve (`Defrost Valve`)**
* Type: Boolean
* Direction: Report only
* Values: `true` (energised/open), `false` (idle/closed)
* Description: Binary indicator of the reverse-cycle defrost four-way valve.


* **DP 113: Disinfection Cycle Counter (`P39 cycle` / `xd_cs`)**
* Type: Integer
* Direction: Report only
* Range: 0 to 20000 Cycles (Step: 1)
* Description: Total counter of completed high-temperature anti-legionella sanitisation cycles.


* **DP 114: Next Disinfection Countdown (`P39` / `xd_tm`)**
* Type: Integer
* Direction: Report only
* Range: 0 to 9999 Minutes (Step: 1)
* Description: Countdown timer until the next scheduled periodic disinfection cycle runs.


* **DP 115: Heating Fault Timeout / Watchdog (`P40` / `heat_tm`)**
* Type: Integer
* Direction: Report only
* Range: 0 to 9999 Minutes (Step: 1)
* Description: Thermal watchdog timer (default reset value: `4320` minutes / 72 hours). Tracks elapsed heating duration or window between valid heating cycles; resets whenever the target cylinder temperature condition is fulfilled, and raises a thermal fault if the timer reaches 0.


* **DP 116: High-Temperature Disinfection Active (`高温消毒`)**
* Type: Boolean
* Direction: Report only
* Values: `true` (active), `false` (idle)
* Description: Binary indicator denoting active execution of the anti-legionella high-temperature heating cycle.

## ESPHome Example Config

Below is the config I use for my ESPHome project

```yaml
# Board: Generic ESP8266 Board (Generic)
# Definition: definitions/boards/generic-esp8266/manifest.yaml

esphome:
  name: water-heater
  friendly_name: Water Heater
  project:
    name: Reclaim.300REC25P
    version: ESPHome

esp8266:
  board: esp01_1m

logger:
  baud_rate: 0

api:
  reboot_timeout: 6h
  encryption:
    key: !secret esphome_api_key

ota:
  - platform: esphome
    encryption:

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  ap:
    ssid: Water Heater Fallback Hotspot
    password: !secret wifi_ap_password
  reboot_timeout: 6h

captive_portal:

uart:
  - id: uart_1
    baud_rate: 9600
    rx_pin: 3
    tx_pin: 1
    rx_buffer_size: 1024B
  
tuya:
  time_id: time_sntp
  uart_id: uart_1
  on_datapoint_update:
    - sensor_datapoint: 9
      datapoint_type: bitmask
      then:
        - lambda: |-
            std::string errors = "";
            if (x & (1 << 0)) errors += "E04 ";
            if (x & (1 << 1)) errors += "E06 ";
            if (x & (1 << 2)) errors += "E05 ";
            if (x & (1 << 3)) errors += "E02 ";
            if (x & (1 << 4)) errors += "E07 ";
            if (x & (1 << 5)) errors += "E01 ";
            if (x & (1 << 6)) errors += "E08 ";
            if (x & (1 << 7)) errors += "E03 ";
            if (x & (1 << 8)) errors += "E11 ";

            if (errors.empty()) {
              id(heat_pump_fault_string).publish_state("Clear");
              id(heat_pump_fault_binary).publish_state(false);
            } else {
              id(heat_pump_fault_string).publish_state(errors);
              id(heat_pump_fault_binary).publish_state(true);
            }

text_sensor:
  - platform: template
    name: "Active Fault Codes"
    id: heat_pump_fault_string
    icon: "mdi:alert-circle-outline"
    entity_category: "diagnostic"

time:
  - platform: sntp
    id: time_sntp
    on_time:
      - seconds: 0
        minutes: /15
        hours: 10-17
        then:
          - switch.turn_on: main_switch
      - seconds: 0
        minutes: /15
        hours: 0-9
        then:
          - switch.turn_off: main_switch
      - seconds: 0
        minutes: /15
        hours: 18-23
        then:
          - switch.turn_off: main_switch
water_heater:
  - platform: tuya
    id: tuya_water_heater
    name: "Hot Water Heat Pump"
    icon: "mdi:heat-pump"
    switch_datapoint: 1
    target_temperature_datapoint: 2
    current_temperature_datapoint: 107 # dpId 3 doesn't auto update
    visual:
      min_temperature: 10
      max_temperature: 65
      target_temperature_step: 1
    eco_value: 0
    high_demand_value: 3
    performance_value: 1
    mode_datapoint: 4
    supported_modes:
      - "OFF"
      - ECO
      - PERFORMANCE
      - HIGH_DEMAND

switch:
  - platform: tuya
    id: main_switch
    name: "Hot Water Switch"
    switch_datapoint: 1
    
select:
  - platform: tuya
    id: mode_select
    name: "Working Mode"
    enum_datapoint: 4
    options: {"0":"ECO","1":"Performance","2":"Holiday","3":"Electric / Rescue"}
    entity_category: ""
    icon: "mdi:cog"

number:
  - platform: tuya
    name: "Holiday Remaining"
    number_datapoint: 102
    min_value: 2
    max_value: 199
    step: 1
    unit_of_measurement: "d"
    icon: "mdi:calendar-clock"
    entity_category: "config"

binary_sensor:
  - platform: tuya
    name: "Compressor"
    sensor_datapoint: 103
    device_class: running
    icon: "mdi:heat-pump"
    entity_category: "diagnostic"

  - platform: tuya
    name: "Fan"
    sensor_datapoint: 104
    device_class: running
    icon: "mdi:fan"
    entity_category: "diagnostic"

  - platform: tuya
    name: "Heating Element"
    sensor_datapoint: 105
    device_class: heat
    icon: "mdi:water-boiler"
    entity_category: "diagnostic"

  - platform: tuya
    name: "Defrost Valve"
    sensor_datapoint: 112
    device_class: opening
    icon: "mdi:snowflake-melt"
    entity_category: "diagnostic"

  - platform: tuya
    name: "Disinfection Active"
    sensor_datapoint: 116
    device_class: heat
    icon: "mdi:bacteria-outline"
    entity_category: "diagnostic"

  - platform: template
    name: "System Fault Active"
    id: heat_pump_fault_binary
    device_class: problem
    entity_category: "diagnostic"

sensor:
  - platform: tuya
    name: "Tank Water Temperature"
    sensor_datapoint: 3
    unit_of_measurement: "°C"
    accuracy_decimals: 0
    id: temp_tank
    device_class: temperature
# dpId 3 doesn't auto update, so this sensor is disabled by default. Use dpId 107 instead.
    disabled_by_default: true

  - platform: tuya
    name: "EEV Position"
    sensor_datapoint: 106
    unit_of_measurement: "steps"
    accuracy_decimals: 0
    icon: "mdi:valve"
    entity_category: "diagnostic"

  - platform: tuya
    name: "Tank Outlet Temperature"
    sensor_datapoint: 107
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 0
    icon: "mdi:faucet"

  - platform: tuya
    name: "Ambient Temperature"
    sensor_datapoint: 108
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 0
    icon: "mdi:thermometer-lines"
    entity_category: ""

  - platform: tuya
    name: "Discharge Temperature"
    sensor_datapoint: 109
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 0
    entity_category: ""
    icon: "mdi:thermometer-water"

  - platform: tuya
    name: "Coil Temperature"
    sensor_datapoint: 110
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 0
    icon: "mdi:heating-coil"

  - platform: tuya
    name: "Air Intake Temperature"
    sensor_datapoint: 111
    unit_of_measurement: "°C"
    device_class: temperature
    state_class: measurement
    accuracy_decimals: 0
    entity_category: ""
    icon: "mdi:air-filter"

# how many disinfection cycles were run
  - platform: tuya
    name: "Disinfection Count"
    sensor_datapoint: 113
    state_class: total_increasing
    icon: "mdi:counter"
    entity_category: "diagnostic"
# when to run next disinfection cycle
  - platform: tuya
    name: "Next Disinfection Countdown"
    sensor_datapoint: 114
    unit_of_measurement: "min"
    device_class: duration
    icon: "mdi:timer-outline"
    entity_category: "diagnostic"
# if counts to 0, error E11 is thrown in dpId 9
  - platform: tuya
    name: "Relay Stuck Timeout"
    sensor_datapoint: 115
    unit_of_measurement: "min"
    device_class: duration
    icon: "mdi:timer-alert-outline"
    entity_category: "diagnostic"
```