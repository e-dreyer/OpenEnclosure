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

### ESP Home

A standard config file is used for the [ESP Home](https://esphome.io/) setup. This configuration requires the implementation of a `secrets.yaml` file which allows sensitive information to be hidden in the config file. The following sections are included in the configuration:

### esphome

[ESPHome Core config](https://esphome.io/components/esphome)

```yaml
esphome:
  name: <unique name of your node>
  comment: <descriptive comment for front-ends and your own reference>
  friendly_name: <friendly name to be used in front-ends>

  # Automation executed at boot time
  on_boot:
    priority: 600
    then:
      - logger.log: "Booting"
      - light.turn_off: enclosure_heating_lights
      # This section can be commented out to change the OTA password over Wi-Fi after setup
      # - lambda: |-
      #   id(my_ota).set_auth_password("New password");

  # Automation executed when shutting down
  on_shutdown:
    priority: 600
    then:
      - logger.log: "Shuting down"
      - light.turn_off: enclosure_heating_lights
```

### board specific configuration

The following section specifies the board used by the config and should be changed according to your board:

- [ESP8266](https://esphome.io/components/esp8266)
- [ESP32](https://esphome.io/components/esp32)
- [RP2040](https://esphome.io/components/rp2040)

```yaml
# ESPHome Processor settings
esp32:
  board: esp32doit-devkit-v1
  framework:
    type: arduino
```

### OTA

[ESPHome OTA](https://esphome.io/components/ota)

Over-the-air or OTA allows for updates over a Wi-Fi connection. The initial installation requires a `serial` connection to a host PC but future updates can be performed over Wi-Fi:

```yaml
ota:
  password: !secret ota_password

  # Automation on start
  on_begin:
    then:
      - logger.log: "OTA start"

  # Automation during upload
  on_progress:
    then:
      - logger.log:
          format: "OTA progress %0.1f%%"
          args: ["x"]

  # Automation on end
  on_end:
    then:
      - logger.log: "OTA end"

  # Automation on error
  on_error:
    then:
      - logger.log:
          format: "OTA update error %d"
          args: ["x"]
```

### Wi-Fi

[ESPHome Wi-Fi](https://esphome.io/components/wifi)

The following section defines the Wi-Fi configuration for the `ESPHome` node. It also specifies a fall-back access-point which is used when Wi-Fi isn't available or credentials have incorrectly been configured. The user can then connect to the network of the node and configure the network credentials.

This also includes the `ESPHome` captive portal which allows for a useful and attractive UI over Wi-Fi and the access-point.

```yaml
# Wifi settings
wifi:
  # Credentials
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # IP settings
  manual_ip:
    static_ip: !secret wifi_ip
    gateway: !secret wifi_gateway
    subnet: !secret wifi_subnet

  # Wifi-fallback settings for access point
  ap:
    ssid: !secret ap_ssid
    password: !secret ap_password

# ESPHome Captive Portal settings
captive_portal:
```

### MQTT

[ESPHome MQTT](https://esphome.io/components/mqtt)

This section defines the setup for the MQTT functionality of the enclosure:

```yaml
# ESPHome MQTT settings
mqtt:
  # First the MQTT credentials are specified
  broker: !secret mqtt_broker # Broker IP
  username: !secret mqtt_username # Broker username
  password: !secret mqtt_password # Broker password

  # A birth message which will be posted on-connect can then be defined
  birth_message:
    topic: enclosure/status
    payload: online

  # As well as a last will message. This allows for detection of the state of the ESPHome Node over MQTT
  will_message:
    topic: enclosure/status
    payload: offline

  # Then initial topics to subscribe to are specified
  on_json_message:

    # In my case I configured it to detect the temperature from a sensor connected to my Raspberry pi
    - topic: cr6semainsail/klipper/state/temperature_sensor enclosure_temperature/temperature # Topic from Moonraker Config
      then:
      # Set the template sensor, this acts as a virtual sensor
      - sensor.template.publish:
          id: enclosure_temperature_sensor

          # Get the value from the JSON format
          state: !lambda |-
            return x["value"];
    
    # It also receives the temperature target as provided by Klipper and Moonraker, which allows for the usage of G-code commands
    - topic: cr6semainsail/klipper/state/temperature_sensor enclosure_temperature/target # Topic from Moonraker Config
      then:
      # Set the PID control
      - climate.control:
          id: enclosure_heating_lights_climate # Climate controller
          mode: HEAT

          target_temperature: !lambda |-
            return x["value"];
```

### Climate Control

[ESPHome Climate](https://esphome.io/components/climate/)

The enclosure is configured as an `ESPHome` `climate`, which allows for the configuration of a PID. This PID is then fed back to the AC dimmer and lights

```yaml
# ESPHome Climate PID control
climate:
  - platform: pid
    id: enclosure_heating_lights_climate # ID of the PID controller
    name: "Enclosure Heating Lights Climate"
    sensor: enclosure_temperature_sensor # Sensor to use as the input
    default_target_temperature: 21°C

    heat_output: enclosure_heating_lights_dimmer # The ESPHome output

    # Control parameters calculated by autotune
    control_parameters:
      kp: 1.01859 #0.29030
      ki: 0.00328 #0.00087
      kd: 0.0 #79.04271

      output_averaging_samples: 20      # smooth the output over 10 samples
      derivative_averaging_samples: 20  # smooth the derivative value over 10 samples

    # Set deadband to + or - 1.0°C
    deadband_parameters:
      threshold_high: 2.0°C
      threshold_low: -2.0°C

    # These settings are required for correct behavior in HomeAssistant
    visual:
      min_temperature: 0 °C
      max_temperature: 65 °C
      temperature_step: 1 °C
```

### Sensors

[ESPHome PID](https://esphome.io/components/climate/pid)

With `ESPHome` most outputs can be configured as sensors, which allows for many different use cases. It can also be used to publish the state of the PID:

```yaml
# ESPHome sensors
sensor:

  # This sensor acts as a virtual sensor which is updated over MQTT from Moonraker
  - platform: template
    name: "Enclosure Temperature Sensor"
    id: enclosure_temperature_sensor
    device_class: "temperature"
    state_class: "measurement"
    accuracy_decimals: 2

    filters:
      - heartbeat: 1.0s

  # Show PID parameters on MQTT
  - platform: pid
    name: "Enclosure Heating Lights PID Result"
    id: enclosure_heating_lights_pid_result
    climate_id: enclosure_heating_lights_climate

    type: RESULT
    state_class: "measurement"
    accuracy_decimals: 2

    filters:
      # Help scale the PID output to a value between 0 and 255
      - calibrate_linear:
          - 0.0 -> 0.0
          - 100.0 -> 255.0
      # Send a heartbeat value to smooth PID output
      - heartbeat: 1.0s
      # Sliding window to smooth the temperature
      - sliding_window_moving_average:
          window_size: 20
          send_every: 20
      # Return 0 for negative values
      - lambda: !lambda |-
          if (x < 0) {
            return 0;
          }
          else {
            return x;
          }

#   Used for debugging purposes
  - platform: pid
    name: "Enclosure Heating Lights PID Error"
    id: enclosure_heating_lights_pid_error
    climate_id: enclosure_heating_lights_climate
    type: ERROR
    state_class: "measurement"
    accuracy_decimals: 2
    filters:
      - heartbeat: 1.0s

  - platform: pid
    name: "Enclosure Heating Lights PID Proportional"
    id: enclosure_heating_lights_pid_proportional
    climate_id: enclosure_heating_lights_climate
    type: PROPORTIONAL
    state_class: "measurement"
    accuracy_decimals: 2

  - platform: pid
    name: "Enclosure Heating Lights PID Integral"
    id: enclosure_heating_lights_pid_integral
    climate_id: enclosure_heating_lights_climate
    type: INTEGRAL
    state_class: "measurement"
    accuracy_decimals: 2
    
  - platform: pid
    name: "Enclosure Heating Lights PID Derivative"
    id: enclosure_heating_lights_pid_derivative
    climate_id: enclosure_heating_lights_climate
    type: DERIVATIVE
    state_class: "measurement"
    accuracy_decimals: 2
```

### Output

[ESPHome AC Dimmer](https://esphome.io/components/output/ac_dimmer)

The dimmer is used as the output to control the lights

```yaml
# ESPHome Output
output:

  # Triac AC dimmer used to control the heating lights in the enclosure
  - platform: ac_dimmer
    id: enclosure_heating_lights_dimmer

    gate_pin: GPIO12 # This is the PWM Output
    
    zero_cross_pin:
      number: GPIO5 # The is the ZC input
      mode:
        input: true

    method: leading

    min_power: 30%
    max_power: 100%
    zero_means_zero: true # A value of zero turns the PID off
```

### Lights

[ESPHome Light](https://esphome.io/components/light/)
[ESPHome Monochromatic Light](https://esphome.io/components/light/monochromatic)

The lights are configured to be controlled by the dimmer, which is in turn controlled by the PID

```yaml
# ESPHome Lights
light:
  # This is the heating lights used in the enclosure
  - platform: monochromatic
    name: "Enclosure Heating Lights"
    id: enclosure_heating_lights

    output: enclosure_heating_lights_dimmer # Is controlled by the dimmer
```

### Button

[ESPHome Button](https://esphome.io/components/button/)

A simple virtual button is set-up to trigger the auto-tune functionality of the PID system. This can be triggered through `Home Assistant` or via a physical button.

```yaml
# ESPHome Button
button:
  # Autotune functionality for the PID in HomeAssistant
  - platform: template
    name: "Enclosure Heating Lights PID Autotune"
    id: enclosure_heating_lights_pid_autotune

    on_press:
      - climate.pid.autotune: enclosure_heating_lights_climate
```
