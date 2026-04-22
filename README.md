Installation instructions

MicroPython

The software is developed in micropython. After the first tests, I was amazed of how good and powerful the microPython.org platform is.
However, the software did not run with the latest stable kernel from July 2022 (among other things, the bytearray.hex was not implemented there yet). The latest kernels for various ports can be found in the download section. It is quite possible that the software can also run on other ports. If you have had this experience, please let us know. We can then amend the readme accordingly.
Alternative 1: OTA-Installation with mip

If you just want to get the inetbox2mqtt running on the RP2 pico w, this is the way to go.
For the simple 4M types of the ESP32-S this way does not work anymore. Here I ask you to use alternative 2.
The entire installation process should not take longer than 10 minutes. The installation does not have to take place in the final WLAN in which the inetbox2mqtt is to run later.
To do this, you first have to install an up to date microPython version, to be found at micropython/download. My tests were done with upython-version > 19.1-608.
You must enter the commands from the console line by line in the REPL interface. The last import command reloads the entire installation.
import network
st = network.WLAN(network.STA_IF)
st.active(True)
st.connect('<yourSSID>','<YourWifiPW>')
import mip
mip.install('github:mc0110/inetbox2mqtt/bootloader/main.py','/')
import main

Alternative 2: With esptool - only works with the ESP32

The ESP32 with 4M memory does not have enough main storage in the standard micropython firmware to have all the software in memory. For this reason, some of the python modules have been precompiled and are already included in the firmware. Therefore, it is recommended to use the .bin file. Of course, all source files of the project are included, so that anyone can create the micropython firmware himself.
The .bin file contains both the python and the .py files. This allows the whole project to be flashed onto the ESP32 in one go. For this, you can use the esptool. In my case, it finds the serial port of the ESP32 automatically, but the port can also be specified. The ESP32 must be in programming mode (GPIO0 to GND at startup). The command to flash the complete .bin file to the ESP32 is:
esptool.py write_flash 0 flash_esp32_inetbox2mqtt_v265_4M.bin

Alternatively, you can also use the Adafruit online tool (of course only running with Chrome or Edge):
https://adafruit.github.io/Adafruit_WebSerial_ESPTool/

The offset 0x0 must then be entered here.
This is not a partition but the full image for the ESP32. The address 0x0 is not a typo.
Releasing and updating

There are two release numbers that must match, one in main.py and one in release.py. The update process looks at this and if the numbers are different, then the software is updated during the update.
We are very keen to support the application in the best possible way. Most of the changes were necessary to enable the realisation of a web frontend and the initial installation, the entry of the login data for Wifi connection and mqtt-broker and to test this connection and, from version 2.1.x, also to be able to test the LIN connection to CPplus. It is recommended as best practice to reinstall the application in the process if updating is also possible.
Web frontend

After rebooting the port (ESP32, RP2 pico w), an access point (ESP or PICO) is opened first. For the RP2 pico w, the password "password" is required. Please first establish a Wifi connection with the access point. Then you can access the chip in the browser at http://192.168.4.1 and enter the credentials. For details of the Wifimanager, please refer to mc0110/wifimanager.
We call this the OS mode -> i.e. an operating mode that is not the normal run mode but is necessary for tests and for entering the login data.
￼
You have access to the entire file system with up, download and delete functions via a simple file manager.
You can also test the connectivity to the MQTT broker.
As of version 2.1.x, the LIN module - responsible for the TRUMA communication - is already fully executable in this mode. This is very helpful for debugging the electrical connection or for any adjustments.
In this mode, the TRUMA status elements can be queried about the overall status (see button STATUS) and the set command for the water heater (ON / OFF) can be set via buttons.
You can now also carry out the INIT process in this mode. Details are described below.
Credentials formular

￼
Switching to Normal-Run mode

After entering the login credentials, the boot mode needs to be switched from "OS-Run" to "Normal-Run" to start full communication with the MQTT broker. Press on the "OS mode after reboot" button to toggle to the "Normal mode after reboot" state. (The button toggles between those two states and shows which mode will be started after the next reboot.)
After rebooting in "normal-run" mode, inetbox2mqtt is ready for use. If everything is correctly set up and the port is rebooted, it should connect to the MQTT broker with two confirmation messages.
For placing the files and creating the credentials on the port, it does not need to be connected to the CPplus. You can also swap between 2 different credential-files, e.g. you are working on your computer at home for configuring and then swap to the RV-credentials in your motorhome.
INIT - RESET process

Now you can establish the connection between the port and the LIN bus.
The inetbox2mqtt must be registered once with the CPplus. This initialisation process is very important and without it the connection will not be established successfully.
Inetbox2mqtt must be in normal-run mode when you initialise the CPplus.
Let's have one further look at the INIT process:
Without inetbox2mqtt you find 2 entries in the INIT menu:
* TRUMA: Hx.00.nn
* CPplus: Cy.0z.00
Start INIT process

To do this you have to select and confirm the menu item RESET in the CPplus menu and then also confirm the PR SET that appears. The display then shows a flickering INIT... -> example given.
If the INIT process was successful, then a third entry appears in the INIT menu for the inetbox
* TRUMA: Hx.00.nn
* CPplus: Cy.0z.00
* inetbox: T23.70.0
This process has to be carried out once, after which the connection can be terminated at any time and then resumed as long as no further INIT (RESET at CPplus) takes place.
Very, very important:
If you have already connected an inetbox to the CPplus, it is essential to carry out the INIT once without the inetbox. After that, there should only 2 entries be displayed. Then you can connect the inetbox2mqtt to perform another INIT.
MQTT topics

The service/truma/control_status/# topics can be received. They include the current status of CPplus and TRUMA If your heater is off and you start with a set-command or with an input at the CPplus there is a delay of about 30sec before you'll see the first values. This is a normal behavior.
Status Topic	Payload	Function
service/truma/control_status/#		subcribing all status-entries
service/truma/control_status/alive	on/off	connection control
service/truma/control_status/clock	hh:mm	CPplus time
service/truma/control_status/release	x.y.z	release no
service/truma/control_status/current_temp_room	temperature in °C (0, 5-30°C)	show current room temperature
service/truma/control_status/target_temp_room	temperature in °C (0, 5-30°C)	show target room temperature
service/truma/control_status/current_temp_water	temperature in °C (0-70°C)	show current water temperature
service/truma/control_status/target_temp_water	temperature in °C (0-70°C)	show target water temperature
service/truma/control_status/energy_mix	gas, mix, electricity	heater mode of operation
service/truma/control_status/el_power_level	0, 900, 1800	heater electrical max. consumption
service/truma/control_status/heating_mode	off, eco, high	fan state
service/truma/control_status/aircon_operating_mode	off, vent, cool, hot, auto	aircon mode of operating
service/truma/control_status/aircon_vent_mode	low, mid, high, night, auto	aircon ventilator mode
service/truma/control_status/target_temp_aircon	temperature in °C (20 - 32°C)	show target aircon temp
service/truma/control_status/operating_status	0 - 7	TRUMA heater operation-mode (0,1 = off / 7 = running)
service/truma/control_status/error_code	0-xx	TRUMA error codes
service/truma/control_status/release	xx.xx.xx	Software-Release-No
If you want to set values, then you must use the corresponding set-topics. The list of set-topics and valid payloads can be found here.
Set Topic	Payload	Function
service/truma/set/target_temp_room	temperature in °C (0, 5-30°C)	set target room temperature
service/truma/set/target_temp_water	0, 40, 60, 200	set target water temperature (= off, eco, high, boost)
service/truma/set/energy_mix	gas, mix, electricity	set mode of operation
service/truma/set/el_power_level	0, 900, 1800	set electrical max. consumption
service/truma/set/heating_mode	off, eco, high	set fan state (off only accepted, if room heater off)
service/truma/set/aircon_operating_mode	off, vent, cool, hot, auto	set aircon operating mode
service/truma/set/aircon_vent_mode	low, mid, high, night, auto	set aircon ventilation
service/truma/set/target_temp_aircon	temperature in °C (20-30°C)	set target aircon temperature
System commands		
service/truma/set/reboot	1	reboot the port
service/truma/set/os_run	1	set mode OS-RUN
service/truma/set/ota_update	1	download current version from GITHUB, only recommended for RPI
To switch on the room heating, target_temp_room > 4 and heating_mode = eco must be set together. For this purpose, the respective commands should be sent immediately after each other.
The same applies to energy_mix and el_power_level, which should be set together. Of course, only certain constellations of energy_mix and el_power_level make sense here if the heating has an electric and a combustion unit. The commands are the same for diesel and gas burners, by the way. Valid combinations are (gas-0, mix-900, mix-1800, electricity-900, electricity-1800).
For the Aventa aircon, only certain pairs of aircon_operating_mode and aircon_vent_mode payloads are possible (off-low, auto-auto, vent/cool/hot-low/mid/high).
The system commands are an extension of the command scope and allow restarting the port, changing the operating mode and updating the software status with the GITHUB release.
An example for a complete control system of a Smart Home solution can be found in the Docs - it shows the capability of bidirectional operation of Home Assistant. Bidirectional means that the values can be set in the CPplus display as well as in the Home Assistant frontend and are passed through in each case.
Status LEDs are showing the operating mode

The operating mode is indicated by 2 GPIO outputs. This gives the option of displaying the current mode with 2 LEDs.
￼
The LEDs are hardware-dependent and can be configured in tools.py.
'mqtt_led' indicates when the MQTT connection is up. 'lin_led' indicates when the connection to the CPplus is established.
The search for connection errors (e.g. missing LIN signal, swapping rx/tx, defective TJA 1020) can be very time-consuming and annoying. Therefore 'lin_led' has been supplemented to the extent that the LED already flickers slightly when signals are registered on the port rx line (equivalent is tx-output on the TJA 1020). This also happens before an CPplus INIT, in other words a registration of the inetbox2mqtt at the CPplus has taken place. In this way, it can be detected very quickly whether there are connection problems on the rx line. If the INIT process has taken place, the LED lights up brightly.
Attention: The CPplus does not transmit continuously, therefore transmission pauses of 15-25s are normal.
Addon: Integration of Truma DuoControl

Another functionality has been added. This is an additional function, at the moment not implemented in INETBOX.
￼
The status changes of two GPIO inputs (GPIO18 and GPIO19) and the GPIO outputs (GPIO22 and GPIO23) are now also published to the broker. The reaction time for status-changes is approx. 10s.
The associated topics are
* service/truma/control_center/duo_ctrl_gas_green
* service/truma/control_center/duo_ctrl_gas_red
* service/truma/control_center/duo_ctrl_i
* service/truma/control_center/duo_ctrl_ii
with the payloads ON/OFF. The outputs can be controlled with the SET commands
* service/truma/set/duo_ctrl_i
* service/truma/set/duo_ctrl_ii
Inputs and outputs are inverted, i.e. the inputs react with ON when connected to GND, the outputs switch to GND level when ON.
Addon: Integration of MPU6050 for spiritlevel-Feature

A second optional feature has been added. For leveling of an RV-car a MPU6050 IMU (inertial measurement unit) can be connected to the I2C bus.
Attention: The original version of this add-on used the GPIO00/01 for the I2C communication. Unfortunately, there is a conflict with the system UART (Tx=GPIO01). As a consequence, no debugging output was possible after initialising the I2C interface. It is important for all those who want to update to the current version that the pins for SDA and SCL have changed!
Different pins are required here:
For ESP32 please use I2C bus with SDA (GPIO26) and SCL (GPIO25). For RP2 pico w please use I2C bus 1 with SDA (GPIO02) and SCL (GPIO03).
Every 500ms the acceleration and gyroscopic values are read and combined and filtered by a Kalman-Filter. For the moment the result (pitch and roll angle) is published via MQTT every 10s.
The associated topics are
* service/spiritlevel/spirit_level_pitch
* service/spiritlevel/spirit_level_roll
Alive topic

Short digression: The CPplus only sends 0x18 (with parity it is 0xD8) requests if an INETBOX is registered. This can be recognised by the third entry in the index menu on the CPplus, among other things. The port answers these requests. Only when it receives 0x18 messages, the connection to the CPplus is established and the registration has taken place. This makes it easy to find out if there is an electrical problem. If the LED (GPIO14, see Status LEDs) is lit, communication with the CPplus is established. The port also outputs this as an "alive" topic via the MQTT connection (approx. every 60 sec): connection OK => payload: ON; connection not OK => payload: OFF.
