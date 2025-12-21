# ESPHome Fan Controller

Temperature-controlled fan for cooling cabinets. Inspired by [Patrick's project](https://github.com/patrickcollins12/esphome-fan-controller).

## Parts
- ESP32-C3-DevKitM-1
- DHT22 temperature sensor
- 12V 4-pin PWM PC fan
- 12V power supply
- Logic level converter (3.3V ↔ 5V) for PWM output and tachometer input
- Some wires

## Wiring (DO NOT CHANGE - PCB IS FIXED)
- GPIO0 → DHT22 data
- GPIO20 → Fan PWM (blue wire)
- GPIO21 → Fan tachometer (yellow wire)
- Fan red → 12V
- Fan black → GND

### Known Limitation: Reduced Max Speed
The ESP32 outputs 3.3V PWM signals, but PC fans expect 5V PWM.
- Fan limited to ~750 RPM (about 66% of max speed)
- Usually sufficient for quiet cabinet cooling

**TODO - Fix if more speed needed:**
Add a 4-channel I2C logic level converter (€2.89 on eBay) between GPIO20 and fan PWM:
- Example: "3x Pegelwandler 4 Kanal I2C IIC Logic Level Converter 5V~3.3V"
- Connections:
  - LV (Low Voltage) → ESP 3.3V
  - HV (High Voltage) → ESP 5V
  - GND → Common ground
  - LV1 → GPIO20
  - HV1 → Fan blue PWM wire
- Result: Fan reaches full speed (2000+ RPM instead of 750 RPM)

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