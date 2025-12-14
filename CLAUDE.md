# ESPHome Fan Controller - Critical Constraints

## ⚠️ IMPORTANT: PCB IS FIXED - DO NOT MODIFY GPIO MAPPINGS

**The PCB for this project is already produced and deployed. Under NO circumstances should the following GPIO pin assignments be changed:**

### Fixed GPIO Pin Mappings (DO NOT MODIFY)
- **GPIO21**: Tachometer input (fan RPM reading) - INPUT_PULLUP
- **GPIO20**: PWM output for fan speed control (LEDC)
- **GPIO0**: DHT22 temperature/humidity sensor data pin

### Hardware Configuration (FIXED)
- **Board**: ESP32-C3-DevKitM-1
- **Framework**: Arduino
- **PWM Frequency**: 25KHz (standard PC fan frequency)
- **PWM Range**: 10% min, 100% max power
- **Fan Type**: 4-pin PWM PC fan with tachometer

## System Overview

This is a PID-controlled fan system for a media cabinet ("Medienschrank") that:
- Monitors temperature via DHT22 sensor
- Controls fan speed using PID algorithm
- Reports to Home Assistant via ESPHome API
- Supports both automatic (PID) and manual control modes

## Key Components

### Temperature Control
- PID controller with configurable KP, KI, KD parameters
- Deadband control to prevent oscillation near target temp
- Default target: 30°C

### Fan Control Modes
1. **Automatic (PID)**: Fan speed controlled by temperature
2. **Manual**: Direct percentage control (0-100%)

### Monitoring
- Real-time fan RPM monitoring via tachometer
- Temperature and humidity readings
- PID term visibility (P, I, D, error, output values)
- WiFi signal strength and uptime reporting

## Modification Guidelines

✅ **Safe to modify:**
- PID parameters (KP, KI, KD)
- Temperature targets and thresholds
- Deadband settings
- Fan speed limits (within 10-100% range)
- Update intervals and logging levels
- Home Assistant integration settings

❌ **NEVER modify:**
- GPIO pin assignments
- Board type (esp32-c3-devkitm-1)
- PWM frequency (25KHz)
- Any hardware-related configurations

The hardware is fixed and working - only software behavior can be adjusted.