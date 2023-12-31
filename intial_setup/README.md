# Initial Setup

## Printer construction

For the initial setup and construction of the printer, the setup is quite simple. The original `manual` has been included in the `manual/` folder. In this `manual` the user will see that 4 screws need to be used to mount the gantry to the base of the printer. A top handle is also included, which can be added and the printer will still fit in the enclosure. However, for some reason a different `allen key` is used for this handle, which was never included in the original toolkit.

![Gantry screws](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/gantry_screws.jpg?raw=true)

In the top of the packaging, a drawer is included and the tools have been removed and placed in a zip lock bag in the printer's box. This drawer can be inserted in the front of the printer.

Then, the two motors should be connected, using the `white` connectors on the base of the printer. These connectors will be close to the mounts of the motors and the user will easily see how to connect them.

Then the hotend and motor wire needs to be connected. This is the long cord in the sleave and is connected to the base and motherboard of the printer. This wire needs to be connected to the socket, just below the red filament sensor. The cord should then be connected to the bowden tube and be connected to the daughterboard of the hotend.

![Connector below sensor](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/connector_below_sensor.jpg?raw=true)

This cable can also be stabilized using the included printed support arm:

![Cable feed arm](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/cable_feed_arm.jpg?raw=true)

Finally, the z-stop sensor needs to be connected. It is positioned on the front left of the printer and has a black connector. These connectors are also close to each other and the user will easily notice them.

## Placing the printer in the enclosure

First start by inserting the two included `light bulbs` in the back left and right-hand corners. Although the bulbs are not identical, there connectors are and order does not matter.

In order to place the printer in the enclosure, the top `LED` bar needs to be disconnected, by removing the two screws. The `LED` bar stays connected with its wire, but can be placed on the top of the enclosure, to keep it out of the way. The printer can then be inserted into the enclosure. 

![Disconnecting LED bar](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/disconnecting_led_bar.jpg?raw=true)

![LED bar rest position](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/LED_rest_position.jpg?raw=true)

Note, that there are two L-shaped blocks at the back. The printer should be moved to the back, that the feet of the printer are against these blocks.

![L-shaped blcoks](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/positioning_blocks.jpg?raw=true)

The printer can then be connected on the left-hand side to connect its AC power.

Then the black plastic enclosure can be removed on the left-hand side of the enclosure, to reveal the bottom module containing the `Raspberry Pi` and `ESP32`. A `webcam` and `2 USB` cables have been included, as well as a `WiFi dongle`. The user can then plug in `1 USB` cable into the `Raspberry Pi`. Please note there is a small piece of `black isolation tape` on one of the cables. This is to isolate the printer's power from the `Raspberry Pi` to ensure that they do not attempt to power each other. `DO NOT REMOVE THIS TAPE` and plug it in as is.

![Cable connection](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/cable_connection.jpg?raw=true)

At this point, the `ESP32` can also be carefully removed, by holding the plate it is mounted on and pulling it out in a straight line. We will be setting the `WiFi` settings on the `ESP32` in the next section.

The `webcam` can then also be connected, and the cables fed through the side of the enclosure and the printer can be connected on its front left-hand side.

![Web cam and printer wiring](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/web_cam_wiring.jpg?raw=true)

The `LED` bar can then be connected to the enclosure again.

Keep the black enclosure on the side open for now as we still need access to the `ESP32`.

## Wifi setup

This guide is intended to isolate the setup for the `WiFi` functionality and the printer. If the both the `ESP32` for the enclosure and `Raspberry Pi` is already setup and pre-installed, this guide needs to be followed to connect the printer to a new network. The process only needs to be completed once to connect to a new network

### Raspberry Pi

In order to connect the `Raspberry Pi` to a new network, an `HDMI` cable needs to be plugged into the `Raspberry Pi` and a `keyboard` needs to be connect. This is required to gain access to the terminal of the `Raspberry Pi`. Once both the `keyboard` and monitor are connected, ensure that a `WiFi dongle` is plugged into one of the `USB` ports of the `Raspberry Pi`, you can power it on by connecting a `micro USB` cable to the power connector. Wait for the `Raspberry Pi` to boot until you are prompted for a `password`.

The default `password` for the `Raspberry Pi` is `root`. While entering the `password` no text will be displayed on the screen. Simply press `enter` after entering the `password` and you will be logged in.

#### Adding your network credentials

Enter the following commands line by line separately in the terminal, pressing `enter` after every line. Be sure to replace `ssid` with your `WiFi network name` and `password` with your `password`. They should be enclosed by the single quotes.

```bash
sudo rfkill unblock wifi
sudo sh -c "wpa_passphrase 'ssid' 'password' >> /etc/wpa_supplicant/wpa_supplicant.conf"
sudo wpa_cli -i wlan0 reconfigure
```

`Reboot` the `Raspberry Pi` by typing `reboot` into the terminal. Wait for the `Raspberry Pi` to reboot and login again.

Once the system is booted, enter the command `ifconfig` and you'll be presented by an output such as the following:

```bash
pi@mainsail:~ $ ifconfig
eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether b8:27:eb:3c:98:b5  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 12  bytes 1698 (1.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12  bytes 1698 (1.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.59  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::b087:ae52:2982:2133  prefixlen 64  scopeid 0x20<link>
        ether c8:3a:35:c3:d9:3e  txqueuelen 1000  (Ethernet)
        RX packets 655  bytes 145261 (141.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 344  bytes 156595 (152.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Note the output for `wlan0`, this is the `Wireless dongle`. The section `inet 192.168.1.59` is the `ip address` of the `Raspberry Pi` on your network. `ether c8:3a:35:c3:d9:3e` is the `MAC` address of the dongle. The `ip address` in your output may differ as this is provided by your network and router. The `MAC` address might be the same depending on the dongle you used. 

### Static IP settings

The `ip address` displayed in the previous section is very important. This is the address of the printer on the network. Although the `ip address` is supplied by your router, it might not remain constant and may change every 48 hours or so, which is not desired. For this reason, we want to supply it with a `static ip address`, this will ensure that the `Raspberry Pi` will always have the same address, which is required for you to easily connect to your printer as well as connect the `ESP32` which controls the enclosure to your printer.

First, we need to find the `ip address` of your `router`. Run the following command to get this:

```bash
ip r | grep default
```

The output should look something like this:

```bash
default via 192.168.1.1 dev wlan0 proto dhcp src 192.168.1.59 metric 303 
```

Note the part `192.168.1.1`, which might differ for you, this is the `ip address` of your router. We will call it `<router ip>` for now, that you can use it in the following sections.

Next, we need to find your `DNS` server. You can find this using the following command. This command will prompt you for your `password` of your `Raspberry Pi` and open the terminal text editor afterwards.

```bash
sudo nano /etc/resolv.conf
```

The output should look something like this:

```bash
# Generated by resolvconf
domain RT-AC58U_V3-3C28
nameserver 192.168.1.1
```

Note the `ip address` after `nameserver`. We will call this `<dns ip>`. Press `ctrl+x` to exit this editor and return to the terminal.

Now we are ready to set the `static ip address`. Start by entering the following command. This will once again open a terminal text-editor.

```bash
sudo nano /etc/dhcpcd.conf
```

The output in your terminal will then possibly be an empty text file or the might be some other configurations. If settings already exist, update the information, other simply enter the following:

- `NETWORK` should be replaced with your interface that you are using, in the case of a `WiFi dongle` you need to enter `wlan0`
- `STATIC_IP/24` should be replaced with your desired `ip address`. For this guide we will use `192.168.1.59/24`. If you choose your own `ip address`, be sure that the third digit marked by an x `192.168.x.y` needs to be the same as the third digit in your `<router ip>` we found earlier and y should always be less than `255` and never numbers like `0`, `1`, `10`, `99` or `100` as they are often used by your network or router, but your could use them if you know what you are doing.
- `ROUTER_IP` should be replaced by your `<router ip>` we found in the previous section.
- `DNS_IP` should be replaced by your `<dns ip>` found in the previous section

```bash
interface NETWORK 
static ip_address=STATIC_IP/24
static routers=ROUTER_IP 
static domain_name_servers=DNS_IP
```

Thus, my implementation looked something like this. Important, if your file contains a lot of other information, do not change any of these settings. Also, if the lines already exist in the file, but they start with a `#`, remove it. The `#` acts as a comment line and these lines are thus commented out and not active, as seen below in my configuration.

```bash
interface wlan0
static ip_address=192.168.1.59/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8 fd51:42f8:caae:d92e::1
```

Once done, press `ctrl+x` and you will be prompted at the bottom of the screen to save the file. Press `shift+y` to select `yes` and press `enter` to confirm to overwrite the file contents. You will then be returned to the terminal screen. Finally, we can `reboot` the system again, by running the command `reboot` in the terminal.

## ESP32 and ESP Home

First start by going to the `root` directory with the following command

```bash
cd ~
```

Then we want to go to the directory of the `ESP32` configuration

```bash
cd OpenEnclosure
```

Then we need to activate the `virtual environment`

```bash
source venv/bin/activate
```

Please not that after activating the `virtual environment` your terminal will change and look like this:

```bash
(venv) pi@mainsail:~/OpenEnclosure $ 
```

The reason for this is we are using `Python virtual environments` which allows for isolated control over dependencies.

```bash
cd enclosure
```

In this folder we have a few files, the two important ones are `enclosure.yaml` and `secrets.yaml`. The `enclosure.yaml` file contains the entire configuration for the `firmware`. This file allows us to program the `firmware` with a simple configuration file. If you look at this file, you will see that it defines different `pins` and `behaviors` for the `ESP32`. Finally, we have the `secrets.yaml` file, which contains our credentials and other parameters. Let's open this file to edit it:

```bash
nano secrets.yaml
```

You should see something like this:

```bash
wifi_ssid: YOUR_WIFI_SSID
wifi_password: YOUR_WIFI_PASSWORD
wifi_ip: 192.168.1.74 # Your static IP for your ESP32
wifi_gateway: 192.168.1.1 # Your default gateway, this is the <router ip> we got in a previous section
wifi_subnet: 255.255.255.0 # Your subnet mask, this is almost always 255.255.255.0

ap_ssid: enclosure 
ap_password: password 

ota_password: password 

mqtt_broker: 192.168.1.59 # This is the IP of your Raspberry Pi
mqtt_username: mainsail # This is your MQTT username
mqtt_password: root # This is your MQTT password
```

Here you need to change a few things:

- You need to change `YOUR_WIFI_SSID` to the name of your `WiFi` network, similar to how we connected the `Raspberry Pi` to your `WiFi`.
- You also need to change and set the `YOUR_WIFI_PASSWORD` to the `password` of your `WiFi` network.
- `wifi_ip` will be the `ip address` that you want for the `ESP32`, similar to how we set the `static ip` of the `Raspberry Pi`, you need to chose an `ip address` for the `ESP32`.
- `wifi_gateway` will be the same as the `<router ip>` we found in the previous section
- `wifi_subnet` is the `subnet mask` of your router. For most users this will always be `255.255.255.0`.

- `ap_ssid` is the name for a `hotspot` the `ESP32` can create. If the `ESP32` is unable to connect to a `WiFi` network, it will create its own `access point` and you will be able to connect to it and set the `ssid` and `password` in your browser.
- `ap_password` will be the `password` of this `hotspot` which the `ESP32` creates.

- `ota_password` is the password which `esp_home` on the `Raspberry Pi` can use to update the `ESP32` over the network, thus eliminating the need for a `USB` cable in the future.

- `mqtt_broker` needs to be the `ip address` of the `Raspberry Pi`. You can find this by using the `ifconfig` command in the terminal of the `Raspberry Pi`, similar to how we did it in previous sections of the guide.
- `mqtt_username` is the username you used for your `MQTT Broker` from the previous section.
- Finally, `mqtt_password` is the `password` for your `MQTT Broker` from the previous section.

Now we are ready to flash the `firmware` to the `ESP32`. First start by opening the bottom panel on the side of the enclosure. The panel should look like this.

![Closed enclosure side panel](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/enclosure_case_closed.jpg?raw=true)

Open the enclosure by pulling of the top panel, it should look something like this:

![Opened enclosure side panel](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/enclosure_powered_on.jpg?raw=true)

Ensure that the enclosure is not powered on, this can be notices by the LCD display and lights not be powered on. You can achieve this by disconnecting the main AC plug of the enclosure

![Powered off enclosure side panel](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/enclosure_powered_off.jpg?raw=true)

Note the orientation of the `ESP32` in the enclosure. Carefully disconnect it by pulling it out of the enclosure. Be sure to hold on to the back panel which the `ESP32` is mounted on, as it can also be removed to reveal the wiring of the `ESP32`:

![ESP32 mounting orientation](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/esp_mounting_orientation.jpg?raw=true)

Connect the `ESP32` with a `USB` cable to the `Raspberry Pi`. Preferably not the `USB` cable used to connect the printer, as it does not provide power to the `ESP32` and the red `LED` on the `ESP32` will not turn on when plugged in. Rather use the extra included `USB` cable.

![ESP32 connected to printer](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/esp_connected_to_usb.jpg?raw=true)

Power on the enclosure, by connecting the AC plug again and wait for the `Raspberry Pi` to boot. After a few minutes you can `SSH` into the `Raspberry Pi` again, or use an externally connected monitor, this is your choice.

Once back in the terminal of the `Raspberry Pi`, let's go back to the directory of the `ESP Home` config, with the following commands

```bash
cd ~
cd OpenEnclosure
```

Now we first need to activate the `Python virtual environment` again with the following command:

```bash
source venv/bin/activate
```

Please not that after activating the `virtual environment` your terminal will change and look like this:

```bash
(venv) pi@mainsail:~/OpenEnclosure $ 
```

Now we can move to the directory of the `.yaml` config file:

```bash
cd enclosure
```

Now we can upload the `firmware` to the `ESP32` with the following command:

```bash
esphome run enclosure.yaml
```

This will run for a while as it is possible that you might not have all the required dependencies the first time you run this command. This can easily take about `10` to `15` minutes the first time you run it.

After this command has run for a while, you will be presented with the following prompt:

```bash
INFO Successfully compiled program.
Found multiple options for uploading, please choose one:
  [1] /dev/ttyUSB0 (CP2102 USB to UART Bridge Controller - CP2102 USB to UART Bridge Controller)
  [2] Over The Air (192.168.1.74)
(number): 
```

This gives us 1 of 2 options. The first option is to flash the `firmware` over the `USB` cable, while second option is for `OTA` or `Over-the-air updates` which allows us to update the `ESP32` over `WiFi`. As the `ESP32` is not yet connected to the `WiFi` network, we will press `1` and then `enter`.

The output should look something like this:

```bash
INFO Successfully compiled program.
Found multiple options for uploading, please choose one:
  [1] /dev/ttyUSB0 (CP2102 USB to UART Bridge Controller - CP2102 USB to UART Bridge Controller)
  [2] Over The Air (192.168.1.74)
(number): 1
esptool.py v4.6.2
Serial port /dev/ttyUSB0
Connecting....
Chip is ESP32-D0WD (revision v1.0)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 8c:aa:b5:a2:a0:04
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Flash will be erased from 0x00010000 to 0x0011dfff...
Flash will be erased from 0x00001000 to 0x00005fff...
Flash will be erased from 0x00008000 to 0x00008fff...
Flash will be erased from 0x0000e000 to 0x0000ffff...
Compressed 1103120 bytes to 718365...
Wrote 1103120 bytes (718365 compressed) at 0x00010000 in 17.3 seconds (effective 510.6 kbit/s)...
Hash of data verified.
Compressed 17440 bytes to 12128...
Wrote 17440 bytes (12128 compressed) at 0x00001000 in 0.6 seconds (effective 235.6 kbit/s)...
Hash of data verified.
Compressed 3072 bytes to 144...
Wrote 3072 bytes (144 compressed) at 0x00008000 in 0.1 seconds (effective 330.8 kbit/s)...
Hash of data verified.
Compressed 8192 bytes to 47...
Wrote 8192 bytes (47 compressed) at 0x0000e000 in 0.1 seconds (effective 463.2 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
INFO Successfully uploaded program.
INFO Starting log output from /dev/ttyUSB0 with baud rate 115200
```

After this process has been completed, you will start to see `logging` and other information from the `ESP32`, which verifies that flashing the `firmware` was successful and it has successfully connected to the `MQTT Broker` on the `Raspberry Pi`. You might however notice some errors, this will be due to the `ESP32` not being connected to the sensors in the enclosure, so you can ignore them for now:

```bash
INFO Successfully uploaded program.
INFO Starting log output from /dev/ttyUSB0 with baud rate 115200
[11:17:03][I][logger:271]: Log initialized
[11:17:03][C][ota:473]: There have been 0 suspected unsuccessful boot attempts.
[11:17:03][D][esp32.preferences:114]: Saving 1 preferences to flash...
[11:17:03][D][esp32.preferences:143]: Saving 1 preferences to flash: 0 cached, 1 written, 0 failed
[11:17:03][I][app:029]: Running through setup()...
[11:17:03][I][i2c.arduino:183]: Performing I2C bus recovery
[11:17:03][C][switch.gpio:011]: Setting up GPIO Switch 'Printer Power'...
[11:17:03][D][switch:016]: 'Printer Power' Turning OFF.
[11:17:03][D][switch:055]: 'Printer Power': Sending state OFF
[11:17:03][D][switch:016]: 'Printer Power' Turning OFF.
[11:17:03][C][switch.gpio:011]: Setting up GPIO Switch 'LED Lights Power'...
[11:17:03][D][switch:016]: 'LED Lights Power' Turning OFF.
[11:17:03][D][switch:055]: 'LED Lights Power': Sending state OFF
[11:17:03][D][switch:016]: 'LED Lights Power' Turning OFF.
[11:17:03][C][light:035]: Setting up light 'Enclosure Heating Lights'...
[11:17:03][D][light:036]: 'Enclosure Heating Lights' Setting:
[11:17:03][D][light:041]:   Color mode: 
[11:17:03][D][light:085]:   Transition length: 1.0s
[11:17:03][D][main:409]: Booting
[11:17:03][D][light:036]: 'Enclosure Heating Lights' Setting:
[11:17:03][D][light:085]:   Transition length: 1.0s
[11:17:03][D][climate:011]: 'Enclosure Heating Lights Climate' - Setting
[11:17:03][D][climate:015]:   Mode: HEAT
[11:17:03][D][climate:040]:   Target Temperature: 0.00
[11:17:03][D][climate:378]: 'Enclosure Heating Lights Climate' - Sending state:
[11:17:03][D][climate:381]:   Mode: HEAT
[11:17:03][D][climate:383]:   Action: OFF
[11:17:03][D][climate:401]:   Current Temperature: nan°C
[11:17:03][D][climate:407]:   Target Temperature: 0.00°C
[11:17:03][C][sgp4x:012]: Setting up SGP4x...
[11:17:03][E][sensirion_i2c:085]: Failed to write i2c register=0x3682 (2) err=2,
[11:17:03][E][sgp4x:017]: Failed to read serial number
[11:17:03][E][component:113]: Component sgp4x.sensor was marked as failed.
[11:17:03][C][dht:011]: Setting up DHT...
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Proportional': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Integral': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Derivative': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][C][wifi:038]: Setting up WiFi...
[11:17:03][C][wifi:051]: Starting WiFi...
[11:17:03][C][wifi:052]:   Local MAC: 8C:AA:B5:A2:A0:04
[11:17:03][D][wifi:428]: Starting scan...
[11:17:03][W][dht:169]: Requesting data from DHT failed!
[11:17:03][W][dht:060]: Invalid readings! Please check your wiring (pull-up resistor, pin number).
[11:17:03][D][esp32.preferences:114]: Saving 3 preferences to flash...
[11:17:03][D][esp32.preferences:143]: Saving 3 preferences to flash: 3 cached, 0 written, 0 failed
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Error': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Temperature': Sending state nan °C with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Proportional': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Integral': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][sensor:094]: 'Enclosure Heating Lights PID Derivative': Sending state 0.00000 % with 1 decimals of accuracy
[11:17:03][D][climate:378]: 'Enclosure Heating Lights Climate' - Sending state:
[11:17:03][D][climate:381]:   Mode: HEAT
[11:17:03][D][climate:383]:   Action: IDLE
[11:17:03][D][climate:401]:   Current Temperature: nan°C
[11:17:03][D][climate:407]:   Target Temperature: 0.00°C
```

Once you received this output. You can power off the `Raspberry Pi` and unplug the `AC` plug of the enclosure and install the `ESP32` back into the enclosure. Please note the orientation is EXTREMELY important and that the pins are correctly aligned. They should fit perfectly as there is the correct amount of pins.

![ESP32 mounting orientation](https://github.com/e-dreyer/OpenEnclosure/blob/main/Images/esp_mounting_orientation.jpg?raw=true)