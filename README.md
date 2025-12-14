# ESPHome Fan Controller

Temperature-controlled fan for cooling cabinets. Inspired by [Patrick's project](https://github.com/patrickcollins12/esphome-fan-controller).

## Parts
- ESP32-C3-DevKitM-1
- DHT22 temperature sensor
- 12V 4-pin PWM PC fan
- 12V power supply
- Some wires

## Wiring (DO NOT CHANGE - PCB IS FIXED)
- GPIO0 → DHT22 data
- GPIO20 → Fan PWM (blue wire)
- GPIO21 → Fan tachometer (yellow wire)
- Fan red → 12V
- Fan black → GND

### Known Limitation: Reduced Max Speed
The ESP32 outputs 3.3V PWM signals, but PC fans expect 5V PWM. This means:
- Fan interprets 3.3V as ~66% signal
- Max speed limited to ~750-1000 RPM instead of full speed
- This is a hardware limitation of the fixed PCB

For most cabinet cooling this is sufficient. If you need 100% speed, temporarily disconnect the PWM wire (blue) - the fan will run at full speed without PWM control.

## How It Works
Set target temperature. Fan adjusts speed automatically.
- Too hot → Fan speeds up
- Just right → Fan slows/stops
- Set to 0°C → Fan runs 100% (manual mode)

## Setup
1. Install ESPHome
2. Edit WiFi credentials in `fan-control.yaml`
3. Flash ESP32
4. Add to Home Assistant

## Tuning
Default values work fine (optimized for quiet). If needed:
- **Response Speed (P)**: How aggressive (0.3 = quiet)
- **Drift Correction (I)**: Fixes offset (0.0015 = stable)

All settings (P, I, and temperature) save to flash and survive reboots.

### Auto-Tune Button
Automatically finds "perfect" PID values by cycling the fan on/off repeatedly for ~30 minutes. Fan will be noisy during tuning and likely noisier after (optimizes for accuracy, not quiet). Only use if default values don't work for you.

That's it.