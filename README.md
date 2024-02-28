# Ender 3 Pro backup 💾 
Klipper backup script for manual or automated GitHub backups 

This backup is provided by [klipper-backup](https://github.com/Staubgeborener/klipper-backup).

## Installation ⚙️
1. Install [KIAUH]:
```sh
sudo apt update && sudo apt-get install git -y && \
cd && git clone https://github.com/dw-0/kiauh.git && \
./kiauh/kiauh.sh
```
then install [Klipper], [Moonraker], [Fluidd], [PrettyGCode], [Obico], [Crowsnest] and [G-Code Shell Command].

2. Install [Spoolman]:
```sh
sudo apt update && \
sudo apt install -y curl jq && \
mkdir -p ./Spoolman && \
source_url=$(curl -s https://api.github.com/repos/Donkie/Spoolman/releases/latest | jq -r '.assets[] | select(.name == "spoolman.zip").browser_download_url') && \
curl -sSL $source_url -o temp.zip && unzip temp.zip -d ./Spoolman && rm temp.zip && \
cd ./Spoolman && \
bash ./scripts/install_debian.sh
```

3. Install [KAMP]:
```sh
 cd && git clone https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging.git && \
 ln -s ~/Klipper-Adaptive-Meshing-Purging/Configuration printer_data/config/KAMP
```

4. Install [Moonraker-Timelapse]:
```sh
cd && git clone https://github.com/mainsail-crew/moonraker-timelapse.git && \
cd ~/moonraker-timelapse && \
make install
```

5. Install [Klipper-WS281x_LED_Status]:
```sh
cd && ./klippy-env/bin/pip install requests PyYAML RPi.GPIO rpi_ws281x adafruit-circuitpython-neopixel && \
git clone https://github.com/11chrisadams11/Klipper-WS281x_LED_Status.git && \
chmod 744 ./Klipper-WS281x_LED_Status/klipper_ledstrip.py
```
  5.1 Enable the SPI communication by running `sudo raspi-config`. In the menu, select "3 Interface Options", then "I3 SPI" and set "Yes".
  5.2 Edit */boot/firmware/cmdline.txt*:
```sh
spidev.bufsiz=32768
```
  5.3 Then */boot/firmware/config.txt*:
```sh
core_freq=500
core_freq_min=500
```

6. Set the host as a secondary MCU:
```sh
cd ~/klipper/ && \
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service
make menuconfig
```
   6.1 In the menu, set "Microcontroller Architecture" to "Linux process," then save and exit.
   6.2 To build and install the new micro-controller code, run:
```sh
sudo usermod -a -G tty pi
sudo service klipper stop
make flash
sudo service klipper start
```

[KIAUH]: https://github.com/dw-0/kiauh
[Klipper]: https://www.klipper3d.org/
[Moonraker]: https://github.com/Arksine/moonraker
[Fluidd]: https://docs.fluidd.xyz/
[PrettyGCode]: https://github.com/Kragrathea/pgcode
[Obico]: https://www.obico.io/klipper.html
[Crowsnest]: https://github.com/mainsail-crew/crowsnest
[G-Code Shell Command]: https://github.com/dw-0/kiauh/blob/master/docs/gcode_shell_command.md
[Spoolman]: https://github.com/Donkie/Spoolman
[KAMP]: https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging
[Moonraker-Timelapse]: https://github.com/mainsail-crew/moonraker-timelapse
[Klipper-WS281x_LED_Status]: https://github.com/11chrisadams11/Klipper-WS281x_LED_Status
