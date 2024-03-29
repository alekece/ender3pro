# Ender 3 Pro backup 💾 
Klipper backup script for manual or automated GitHub backups 

This backup is provided by [klipper-backup](https://github.com/Staubgeborener/klipper-backup).

## Installation ⚙️
1. Install [KIAUH]:
   ```sh
   sudo apt update && sudo apt-get install git -y && \
   git config --global http.postBuffer 1048576000 && \
   cd && git clone https://github.com/dw-0/kiauh.git
   ```
   Then install [Klipper], [Moonraker], [Fluidd], [PrettyGCode], [Obico], [Crowsnest] and [G-Code Shell Command] by running `./kiauh/kiauh.sh`

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
   
5. Install [Klipper-Backup]:
   ```sh
   cd && git clone https://github.com/Staubgeborener/klipper-backup.git && \
   chmod +x ./klipper-backup/script.sh && \
   cp ./klipper-backup/.env.example ./klipper-backup/.env
   ```
   Replace *.env* content by:
   ```sh
   github_token=<token>
   github_username=alekece
   github_repository=ender3pro
   branch_name=main
   commit_username="pi"
   commit_email="alexis.leprovost@outlook.com"
   
   path_klipperconfig=printer_data/config/*
   path_klipperdatabase=printer_data/database/data.mdb
   path_klipperledstrip=Klipper-WS281x_LED_Status/settings.conf
   path_spoolman=.local/share/spoolman/spoolman.db
   
   exclude=( \
   "*.swp" \
   "*.tmp" \
   "printer-[0-9]*_[0-9]*.cfg" \
   "*.bak" \
   "*.bkp" \
   "*.csv" \
   "*.zip" \
   )
   ```
   Do not forget to set the `github_token` value with fine-grained token at https://github.com/settings/tokens?type=beta.  
   Create */etc/systemd/system/klipper-backup-on-boot.service* file:
   ```sh
   [Unit]
   Description=Klipper Backup On-boot Service
   After=network-online.target
   Wants=network-online.target
   
   [Service]
   User=pi
   Type=oneshot
   ExecStart=/bin/bash -c 'bash $HOME/klipper-backup/script.sh "New Backup on boot $(date +%%D)"'
   
   [Install]
   WantedBy=default.target
   ```
   Then enable the service:
   ```sh
   sudo systemctl daemon-reload && \
   sudo systemctl enable klipper-backup-on-boot.service
   ```
   
6. Install [Klipper-WS281x_LED_Status]:
   ```sh
   cd && ./klippy-env/bin/pip install requests PyYAML RPi.GPIO rpi_ws281x adafruit-circuitpython-neopixel && \
   git clone https://github.com/11chrisadams11/Klipper-WS281x_LED_Status.git && \
   chmod +x ./Klipper-WS281x_LED_Status/klipper_ledstrip.py
   ```
   Enable the SPI communication by running `sudo raspi-config`. In the menu, select "3 Interface Options", then "I3 SPI" and set "Yes".  
   Edit */boot/cmdline.txt*:
   ```sh
   spidev.bufsiz=32768
   ```
      Edit */boot/config.txt*:
   ```sh
   core_freq=500
   core_freq_min=500
   dtoverlay=gpio-shutdown
   ```
   
7. Install [Klipper-DHT]:
   ```sh
   cd && git clone https://github.com/alekece/klipper-dht.git && \
   ~/klippy-env/bin/pip install adafruit-circuitpython-dht && \
   sudo cp klipper-dht/klipper-dht.service /etc/systemd/system && \
   sudo systemctl daemon-reload && \
   sudo systemctl enable klipper-dht.service && \
   sudo systemctl start klipper-dht.service
   ```

8. Set the host as a secondary MCU:
   ```sh
   cd ~/klipper/ && \
   sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
   sudo systemctl enable klipper-mcu.service
   make menuconfig
   ```
   In the menu, set "Microcontroller Architecture" to "Linux process," then save and exit.  
   To build and install the new micro-controller code, run:
   ```sh
   sudo usermod -a -G tty pi && \
   sudo service klipper stop && \
   make flash && \
   sudo service klipper start
   ```
   
9. Install this repository:
   ```sh
   cd && git clone https://github.com/alekece/ender3pro.git && \
   cd ender3pro && cp -r !(README.md) ~/
   ```

10. In order to shutting the host down gracefully, first run `sudo visudo` then append:
    ```sh
    pi ALL=(ALL) NOPASSWD: /sbin/poweroff, /sbin/reboot, /sbin/shutdown
    ```
    Second, install `uhubctl` to handle USB powers:
    ```sh
    sudo apt install uhubctl -y
    ```
    Third, Create */etc/systemd/system/shutdown.service* file:
    ```sh
    [Unit]
    Description= Graceful shutdown or reboot service
   
    [Service]
    Type=oneshot
    RemainAfterExit=true
    ExecStop=sudo uhubctl -a off -l 1-1
    ExecStop=/home/pi/klippy-env/bin/python /home/pi/Klipper-WS281x_LED_Status/klipper_ledstrip.py 0 0 0
   
    [Install]
    WantedBy=multi-user.target
    ```
    Finally enable the service:
    ```sh
    sudo systemctl daemon-reload && \
    sudo systemctl enable shutdown.service && \
    sudo systemctl start shutdown.service
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
[Klipper-Backup]: https://github.com/Staubgeborener/klipper-backup
[Klipper-DHT]: https://github.com/alekece/klipper-dht
