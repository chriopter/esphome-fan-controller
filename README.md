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

```mermaid
graph TD
    %% Define styles for clearer visualization
    classDef power_hv fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef power_lv fill:#ff9,stroke:#333,stroke-width:2px,color:black;
    classDef ground fill:#eee,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5;
    classDef component fill:#fff,stroke:#333,stroke-width:2px;

    %% --- Power Source ---
    subgraph Power_Supply [Power Source]
        USBC[USB-C PD Charger] ==>|USB Cable| TRIGGER[PD Trigger Module<br/>Set to: 12V Output]
    end

    %% --- Voltage Regulation ---
    TRIGGER ==>|12V V+| BUCK[Step-Down Converter<br/>12V → 5V]

    %% --- Main Controller ---
    subgraph ESP32 [ESP32-C3]
        ESP_5V[5V Pin]
        ESP_3V3[3.3V Pin]
        ESP_GND[GND Pin]
        GPIO20[GPIO20<br/>PWM Out]
        GPIO21[GPIO21<br/>Tach In]
        GPIO0[GPIO0<br/>Data]
    end

    %% --- Logic Level Converter ---
    subgraph LLC [Level Shifter]
        HV_REF[HV Ref 5V]
        LV_REF[LV Ref 3.3V]
        LLC_GND[GND]

        LV1_IN[LV1] --> HV1_OUT[HV1]
        HV2_IN[HV2] --> LV2_OUT[LV2]
    end

    %% --- Peripherals ---
    subgraph FAN [12V 4-Pin Fan]
        FAN_RED[Red Wire<br/>12V]
        FAN_BLK[Black Wire<br/>GND]
        FAN_BLUE[Blue Wire<br/>PWM Signal]
        FAN_YELL[Yellow Wire<br/>Tach Signal]
    end

    subgraph SENSOR [DHT22 Sensor]
        DHT_VCC[VCC Pin]
        DHT_GND[GND Pin]
        DHT_DAT[DATA Pin]
    end

    %% --- Common Ground Node ---
    COM_GND((COMMON<br/>GROUND)):::ground

    %% ================= WIRING CONNECTIONS =================

    %% --- Positive Power Rails (Thick Lines) ---
    TRIGGER ==>|12V Major| FAN_RED
    BUCK ==>|5V| ESP_5V
    BUCK ==>|5V| HV_REF
    ESP_3V3 ==>|3.3V| LV_REF
    ESP_3V3 ==>|3.3V| DHT_VCC

    %% --- Signal Paths (Standard Lines) ---
    %% PWM Path (ESP -> Fan)
    GPIO20 --> LV1_IN
    HV1_OUT -->|PWM 5V| FAN_BLUE

    %% Tach Path (Fan -> ESP)
    FAN_YELL -->|Tach 5V| HV2_IN
    LV2_OUT -->|Tach 3.3V| GPIO21

    %% Sensor Path (Bidirectional)
    GPIO0 <--> DHT_DAT

    %% --- Ground Connections (Dotted Lines) ---
    TRIGGER -.->|GND| COM_GND
    BUCK -.->|GND| COM_GND
    ESP_GND -.-> COM_GND
    LLC_GND -.-> COM_GND
    FAN_BLK -.-> COM_GND
    DHT_GND -.-> COM_GND

    %% Apply styles to power nodes for visual clarity
    class TRIGGER,FAN_RED power_hv;
    class BUCK,ESP_5V,HV_REF,ESP_3V3,LV_REF,DHT_VCC power_lv;
    class USBC,ESP32,LLC,FAN,SENSOR component;
```

| ESP32-C3 | Level Converter | Fan Wire | Description |
|----------|-----------------|----------|-------------|
| 3.3V     | LV              | -        | Low voltage reference |
| 5V       | HV              | -        | High voltage reference (from step-down) |
| GND      | GND             | Black    | Common ground |
| GPIO20   | LV1 → HV1       | Blue     | PWM speed control |
| GPIO21   | LV2 → HV2       | Yellow   | Tachometer RPM signal |
| GPIO0    | -               | -        | DHT22 data pin |

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