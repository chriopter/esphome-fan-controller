# ESPHome Fan Controller

Temperature-controlled fan for cooling cabinets. Inspired by [Patrick's project](https://github.com/patrickcollins12/esphome-fan-controller).

## Parts
- ESP32-C3-DevKitM-1
- DHT22 temperature sensor
- 12V 4-pin PWM PC fan
- USB-C PD power supply (12V capable)
- USB-C PD trigger module (set to 12V output)
- DC step-down converter (12V → 5V) for ESP32
- Logic level converter (3.3V ↔ 5V) for PWM output and tachometer input
- Some wires

## Wiring

### Power (12V)
- USB-C PD charger → PD trigger module (set to 12V)
- PD trigger 12V output → Step-down converter input
- PD trigger 12V output → Fan red wire

### Power (5V)
- Step-down converter output → ESP32 5V pin
- Step-down converter output → Level shifter HV pin

### Power (3.3V)
- ESP32 3.3V pin → Level shifter LV pin
- ESP32 3.3V pin → DHT22 VCC pin

### Signals
- ESP32 GPIO20 → Level shifter LV1 → HV1 → Fan blue wire (PWM)
- Fan yellow wire → Level shifter HV2 → LV2 → ESP32 GPIO21 (Tach)
- ESP32 GPIO0 → DHT22 DATA pin

### Ground
All GND pins connected together: PD trigger, step-down, ESP32, level shifter, fan black wire, DHT22

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