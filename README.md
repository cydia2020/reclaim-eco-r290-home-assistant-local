# Reclaim ECO R290 Heat Pump Water Heater — Local Convert (ESPHome)
This repository documents my findings during my teardown of the Reclaim Eco R290 300L Wi-Fi connected heat pump water heater.

It contains the following information:
1. Teardown instructions;
2. Controller internal photos;
3. Controller components guides;
4. Tuya datapoint IDs and their functions of the MCU;
5. Converting the hot water tank from Tuya to ESPHome.

## Note before you start
The [Australian Consumer Guarantee](https://www.accc.gov.au/media-release/broken-but-out-of-warranty-your-consumer-guarantee-rights-may-still-apply) dictates that the manufacturers or suppliers cannot "void" a product's warranty, simply because the consumer opened or modified the product in some way.

However, this is not legal advice, and you should always backup your firmware before flashing anything else. I might upload a backup in the future if my time allows just in case.

## Access to Controller Instructions
1. Switch off power, wait until everything spins down (compressor, fan, etc) before unplugging the heater;
2. Remove the top 4 screws on the front of the tank;
3. Lift up the top cover panel from the waterproof flange at around 45 deg;
4. Pull panel towards ground when lifted up;
5. Press latch on connector, disconnect controller connector (4-pin);
6. Remove the 4 holding down the fixing bracket;
7. Open the controller door, remove the controller from the front of the cover.

## Controller Internal Photos

## Controller Components Guides

I have identified a few well-known components on this specific board. Please see picture below.

The RS485/Modbus transceiver is significant here as it's quite likely that we can tap into Modbus directly with a Modbus RTU master device without needing to flashing the Tuya Wi-Fi chip. Might look into that later, again, if my time allows it.

![Identified Components](/Teardown%20Pictures/Components/Overview.jpg?raw=true "Identified Components")

## Tuya Datapoints Explained
*Warning: This section is AI generated based on my digging, datapoints are accurate*

This section details the Tuya data point mapping for the hot water heat pump controller. Use these definitions to configure custom firmware, Home Assistant entities, or local integration scripts.

Some are in Chinese as besides the tank the Reclaim Eco R290 use off-the-shelf Chinese parts. (Which is good because replacement parts are very easy to find.)

DP 1: Power (`ON/OFF`)
* Type: Boolean
* Direction: Send and report
* Values: `true` (on), `false` (off)
* Description: Controls and reports the primary operational power state of the unit.
* Side note: This switch does not control the compressor directly, the compressor and fan will only run if the tank temperature drops below 45 °C when using this switch alone; use dpId `101` if you need the unit to run immediately.


DP 2: Target Temperature (`Set Temp`)
* Type: Integer
* Direction: Send and report
* Range: 10 to 65 °C (Step: 1, Scale: 0)
* Description: Configures the target hot water setpoint temperature.


DP 3: Current Water Temperature (`CURRENT TEMPERATURE`)
*(DOES NOT AUTO UPDATE)*
* Type: Integer
* Direction: Report only
* Range: -500 to 1500 (Step: 1, Scale: 0)
* Description: Reports the current water temperature. Verify if host integration requires a scale divisor of 10 for true decimal display.


DP 4: Operating Mode (`Working Mode`)
* Type: Enum
* Direction: Send and report
* Values:
* `0`: ECO Mode (热水模式 (lit. hot water mode) - standard heat pump heating)
* `1`: Boost Mode (速热模式 (lit. quick heat mode) - simultaneous heat pump and auxiliary element)
* `2`: Holiday Mode (假日模式 (lit. holiday mode) - low-temperature frost protection standby)
* `3`: Rescue Mode (补救模式 (lit. rescue mode) - auxiliary element only during heat pump failure)
* Description: Selects the active operational profile.


DP 9: Fault Warning (`Failure warning`)
* Type: Bitmap
* Direction: Report only
* Bit Length: 9
* Available Codes: `E01`, `E02`, `E03`, `E04`, `E05`, `E06`, `E07`, `E08`, `E11`
* Description: Bitmask reporting active diagnostic fault codes and safety trips.


DP 101: Timer Switch (`定时专用开关`)
* Type: Boolean
* Direction: Send only
* Values: `true`, `false`
* Description: Dedicated trigger for the internal hardware timer schedule. MCU provides no state confirmation.
* Side note: Toggling this switch will also toggle dpId `1`. Turning on this switch will command the heat pump to run immediately.
* Side note 2: "定时专用开关" means "Switch specifically for the timer function" in Chinese.


DP 102: Holiday Mode Duration (`Remaining holiday`)
* Type: Integer
* Direction: Send and report
* Range: 2 to 199 Days
* Description: Sets and reports remaining duration for holiday standby mode.


DP 103: Compressor State (`Compressor`)
* Type: Boolean
* Direction: Report only
* Values: `true` (running), `false` (idle)
* Description: Binary feedback of compressor operational status.


DP 104: Fan State (`Fan`)
* Type: Boolean
* Direction: Report only
* Values: `true` (running), `false` (idle)
* Description: Binary feedback of evaporator fan operational status.


DP 105: Auxiliary Heating Element (`Heating Element`)
* Type: Boolean
* Direction: Report only
* Values: `true` (energised), `false` (idle)
* Description: Binary feedback of backup electric resistive element status.


DP 106: Electronic Expansion Valve (`EEV Position`)
* Type: Integer
* Direction: Report only
* Range: 0 to 600 Steps
* Description: Reports stepper motor position of the electronic expansion valve.


DP 107: Outlet / Tank Temperature (`Tank`) (**Auto-updates**)
* Type: Integer
* Direction: Report only
* Range: -500 to 2000 °C
* Description: Reports water cylinder outlet/storage temperature sensor readings.


DP 108: Ambient Temperature (`Ambient`)
* Type: Integer
* Direction: Report only
* Range: -50 to 100 °C
* Description: Reports outdoor ambient air intake temperature.


DP 109: Discharge Temperature (`Discharge`)
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C
* Description: Reports compressor discharge refrigerant pipe temperature.


DP 110: Evaporator Coil Temperature (`Coil`)
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C
* Description: Reports evaporator fin coil temperature for defrost monitoring.


DP 111: Suction Temperature (`Suction`)
* Type: Integer
* Direction: Report only
* Range: -50 to 200 °C (Step: 1)
* Description: Reports compressor refrigerant suction line temperature.


DP 112: Defrost Solenoid Valve (`Defrost Valve`)
* Type: Boolean
* Direction: Report only
* Values: `true` (energised/open), `false` (idle/closed)
* Description: Binary indicator of the reverse-cycle defrost four-way valve.


DP 113: Disinfection Cycle Counter (`P39 cycle` / `xd_cs`)
* Type: Integer
* Direction: Report only
* Range: 0 to 20000 Cycles (Step: 1)
* Description: Total counter of completed high-temperature anti-legionella sanitisation cycles
* Side note: xd_cs is Chinese Pinyin for Xiāodú Cìshù, which literally means "Disinfection Counter".

DP 114: Next Disinfection Countdown (`P39` / `xd_tm`)
* Type: Integer
* Direction: Report only
* Range: 0 to 9999 Minutes (Step: 1)
* Description: Countdown timer until the next scheduled periodic disinfection cycle runs when the tank temperature is lower than 60 °C. Resets to 8640 minutes when tank water reaches 60 °C.


DP 115: Heating Fault Timeout / Watchdog (`P40` / `heat_tm`)
* Type: Integer
* Direction: Report only
* Range: 0 to 9999 Minutes (Step: 1)
* Description: Counts down elapsed heating duration; resets to 4320 minutes when the heat pump and element turn off, and raises a thermal fault if the timer reaches 0. Likely used to track whether the relaies are stuck or if the heat pump is not heating.


DP 116: High-Temperature Disinfection Active (`高温消毒`)
* Type: Boolean
* Direction: Report only
* Values: `true` (active), `false` (idle)
* Description: Binary indicator denoting active execution of the anti-legionella high-temperature heating cycle
* Side note: 高温消毒 means high-temperature disinfection in Chinese. When this `bool` is on, attempting to run the compressor will cause error E07.

## ESPHome Example Config

See [Example Config](/ESPHome%20config/reclaim-heatpump.yaml)
