# OpenEnclosure

This is my implementation for my 3D printing enclosure. Most configuration files use the extra `secret.yaml` file functionality to define
credentials used in the configuration

## Enclosure

The enclosure is controlled by an ESP32 and has the following features:

- A temperature sensor `enclosure_temperature_sensor` for measuring the temperature of the enclosure.
- A set of heating lamps `enclosure_heating_lights` which are used to heat the enclosure.
- A configured climate and PID `enclosure_heating_lights_climate` to control the brightness of `enclosure_heating_lights` to establish a stable and controlled temperature. The temperature is measured by `enclosure_temperature_sensor`
- A virtual button `enclosure_heating_lights_pid_autotune` to be used in `Home Assistant` to tune the PID `enclosure_heating_lights_climate`
- The dimming of the `enclosure_heating_lights` are controlled by `enclosure_heating_lights_climate` and is implemented using a Triac AC Dimmer `enclosure_heating_lights_dimmer`
