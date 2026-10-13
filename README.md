# REVO - Interactive Desktop Robot

**REVO** is a compact, expressive desktop companion powered by an ESP32, a 128 x 64 OLED display, and an MPU6050 motion sensor. It combines animated eyes and emotions with motion-responsive interactions, weather updates, and clock screens.

The project is built with the Arduino framework and is designed to run directly on an ESP32-compatible board.

## Features

- **Animated robotic face:** Two eyes with smooth, spring-like movement, blinking, pupils, and a small expressive mouth.
- **Motion-responsive eyes:** Tilt the device to change its gaze and eye position using the MPU6050.
- **Shake reactions:** Shaking triggers a dizzy expression followed by an angry expression before returning to the previous weather-based mood.
- **Multiple emotions:** Normal, happy, surprised, sleepy, angry, sad, excited, love, suspicious, and dizzy are defined in the code. Some modes share visual details.
- **Touch navigation:** A touch input switches between the face, clock, and weather views. Long presses provide additional controls.
- **Clock and date:** Gets network time via NTP and displays a 12-hour clock and date.
- **World clock:** Shows three labeled locations (JPR, LDN, and NYC).
- **Live weather:** Displays temperature, apparent temperature, humidity, and weather condition from OpenWeather.
- **Three-slot forecast:** Displays upcoming forecast entries with icons and temperatures.
- **Boot animation:** Animated startup sequence with a customizable project name.

## Hardware

| Component | Purpose |
| --- | --- |
| ESP32-compatible development board | Main controller, Wi-Fi connectivity, and display processing |
| SH1106 128 x 64 I2C OLED | Face animations, time, and weather interface |
| MPU6050 accelerometer/gyroscope | Tilt and shake detection |
| Digital touch sensor/module | Navigation and long-press input |
| USB cable and suitable power source | Programming and power |
| Jumper wires / breadboard | Connections |

### Wiring (as defined in the sketch)

| Module | Pin | ESP32 GPIO |
| --- | --- | --- |
| OLED SH1106 | SDA | GPIO 6 |
| OLED SH1106 | SCL | GPIO 7 |
| MPU6050 | SDA | GPIO 6 |
| MPU6050 | SCL | GPIO 7 |
| Touch module | Digital OUT | GPIO 4 |
| OLED + MPU6050 | VCC / GND | Appropriate power rail / GND |
| Touch module | VCC / GND | Appropriate power rail / GND |

Both the OLED and MPU6050 share the I2C bus. The sketch initializes the OLED at address `0x3C`; the MPU6050 typically uses `0x68`.

> **Board compatibility:** GPIO 6 and GPIO 7 are not available for general-purpose I2C on every ESP32 variant. On many original ESP32 boards, these GPIOs are connected to flash memory. Verify your exact board's pinout before wiring, and update `SDA_PIN` and `SCL_PIN` if necessary. Use voltage levels supported by your specific modules.

## Software and Libraries

Developed using the **Arduino IDE** with an ESP32 board package. Install these libraries using the Arduino Library Manager:

- `Adafruit GFX Library`
- `Adafruit SH110X`
- `MPU6050_tockn`
- `Arduino_JSON`

The sketch also uses ESP32/Arduino-provided libraries including `WiFi.h`, `HTTPClient.h`, `Wire.h`, `WebServer.h`, `Preferences.h`, and standard time/math headers.

## Getting Started

1. Install the Arduino IDE and the **ESP32 by Espressif Systems** board package.
2. Install the libraries listed above.
3. Connect the OLED, MPU6050, and touch module according to your board's supported pins.
4. Open the REVO sketch in Arduino IDE.
5. Replace the placeholder values in the configuration section with your own Wi-Fi credentials, OpenWeather API key, location, and time zone:

   ```cpp
   const char* WIFI_SSID = "YOUR_WIFI_NAME";
   const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
   const char* OPENWEATHER_API_KEY = "YOUR_OPENWEATHER_API_KEY";
   const char* CITY = "YOUR_CITY";
   const char* COUNTRY_CODE = "YOUR_COUNTRY_CODE";
   const char* TIMEZONE = "IST-5:30"; // Example: India Standard Time
   ```

6. Select your board and port, then upload the sketch.
7. Keep the device **still** while the MPU6050 calibrates on startup.
8. After calibration and the boot animation, REVO attempts to connect to Wi-Fi and fetch the time and weather.

> **Important security note:** Do not publish a sketch containing your real Wi-Fi password or API key. If you have already pushed them to a public repository, rotate the password/key as appropriate and remove secrets from the tracked code and commit history. Consider a local `secrets.h` file excluded using `.gitignore`.

## Touch Controls

| Action | Result |
| --- | --- |
| Single tap | Cycle through face -> clock -> weather -> face |
| Long press on face (over 800 ms) | Cycle through selectable moods |
| Long press on clock | Toggle regular clock / world clock |
| Long press on weather | Toggle current weather / forecast |
| Single tap while viewing a subpage | Return to the parent page |
| Shake the device on the face page | Dizzy animation -> angry animation -> weather-based mood |

**Note:** Although the sketch tracks taps using a short delay, it does not currently assign an action to double-tapping.

## How It Works

```text
                 +-------------------+
                 |       ESP32       |
                 +---------+---------+
                           |
          +----------------+----------------+
          |                |                |
     MPU6050 (I2C)   Touch input       Wi-Fi / NTP
          |                |                |
   Tilt and shake   Page and mood    Time + OpenWeather
          |                |                |
          +----------------+----------------+
                           |
                    Display logic
                           |
                   SH1106 OLED (I2C)
                           |
                Face / Clock / Weather
```

The main loop reads touch and motion events, refreshes weather periodically when connected to Wi-Fi, and redraws the display about every 20 ms. Eye movement uses a damped, spring-like interpolation for smoother animation.

## Customize REVO

### Change the startup name

The provided sketch currently uses the startup text `N I M O`. In `playBootAnimation()`, change:

```cpp
String bootText = "N I M O";
```

to:

```cpp
String bootText = "R E V O";
```

You can also update the introductory code comment to use the name **REVO**.

### Change sensor pins

If your ESP32 board needs different I2C pins, edit:

```cpp
#define SDA_PIN 6
#define SCL_PIN 7
#define TOUCH_PIN 4
```

### Adjust reactions

The sketch includes these settings:

- `SHAKE_DURATION` - dizzy reaction duration (currently 1,500 ms)
- `ANGRY_DURATION` - angry reaction duration (currently 2,000 ms)
- `LONG_PRESS_TIME` - touch hold threshold (currently 800 ms)
- `SMOOTH_FACTOR` - pupil smoothing setting (currently 0.4)

## Current Limitations

- Weather and NTP time need Wi-Fi access; live weather also needs a working OpenWeather API key.
- The forecast view samples fixed positions from OpenWeather's forecast list. It is not a daily high/low forecast.
- The world-clock implementation uses fixed offsets and should be improved for daylight-saving changes and accurate per-city time-zone handling.
- The sketch currently initializes a digital touch input with `INPUT_PULLUP` but treats `HIGH` as active. Verify your touch module's output polarity if presses behave unexpectedly.
- Some defined mood modes do not yet have completely distinct face animations.

## Future Improvements

- On-device Wi-Fi setup instead of credentials in source code
- Improved world clock with time-zone-aware conversions
- More expressive faces and transitions
- More robust tap/debounce handling
- Battery level monitoring and sleep modes
- A small enclosure or 3D-printed body for REVO

## Project Status

**Working prototype / ongoing development.** Features and behavior described here reflect the current shared sketch; exact wiring and performance depend on the chosen ESP32 board and modules.

## License

No license has been specified yet. Add a `LICENSE` file if you want to define permissions for others to use, modify, or share this project.
