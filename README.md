# OpenEnclosure

This is my implementation for my 3D printing enclosure. Most configuration files use the extra `secrets.yaml` file functionality to define
credentials used in the configuration

## System Architecture

- The system is implemented using a `Raspberry Pi` for hosting the `Klipper` instance with `Moonraker API`. The `Raspberry Pi` also hosts an `MQTT Broker` which is used for communication between all of the subsystems.
- A `HomeAssistant` instance is hosted on a different host but is not required, it mostly serves as a dashboard and control panel for the system, however the `PID` of the heating system is automated and controlled through `G-code`.
- The `ESP32` is used to control the enclosure. `ESPHome` is used for the firmware, which allows for an easy setup and simple configuration. The `ESP32` communicates over Wi-Fi with the rest of the system.

A flow-chart of the system architecture is shown below:

![System Architecture](https://github.com/e-dreyer/OpenEnclosure/blob/main/diagrams/SystemArchitecture.drawio.png?raw=true)

## Enclosure

The enclosure is controlled by an ESP32 and has the following features:

- A temperature sensor `enclosure_temperature_sensor` for measuring the temperature of the enclosure.
- A set of heating lamps `enclosure_heating_lights` which are used to heat the enclosure.
- A configured climate and PID `enclosure_heating_lights_climate` to control the brightness of `enclosure_heating_lights` to establish a stable and controlled temperature. The temperature is measured by `enclosure_temperature_sensor`
- A virtual button `enclosure_heating_lights_pid_autotune` to be used in `Home Assistant` to tune the PID `enclosure_heating_lights_climate`
- The dimming of the `enclosure_heating_lights` are controlled by `enclosure_heating_lights_climate` and is implemented using a Triac AC Dimmer `enclosure_heating_lights_dimmer`
