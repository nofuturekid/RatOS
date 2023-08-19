# Troubleshooting

## My steppers are running backwards!

Refer to the stepper section at the top of printer.cfg, you can add or remove `!` in front of the dir_pin to reverse the direction of that particular stepper.

## Everytime i update my changes are gone.

You're not supposed to change _any_ files inside the managed RatOS config/ folder. You should _only_ change your printer.cfg, if you need to change settings refer to the [Inlcudes & Overrides Documentation](/docs/configuration/includes-and-overrides) section.
![Hands off the RatOS Managed Config Folder!](/img/config-folder.png)

## Klipper says the MCU is unable to connect

Double check your USB connection, try another cable (the one that comes with the board usually works), and check that your firmware was flashed correctly (refer to the guide for your board).
If you need to flash new firmware (for example autoflashing will not work if you upgrade klipper before flashing your board and it's properly connected), use the `COMPILE_BINARIES` macro to generate new firmware binaries for all supported boards. Then download the binary for your board from the `firmware_binaries` folder in the Machine tab, and flash it via SD card.

## I updated klipper and now i get an error!

When you update klipper you might see an error that looks like this:

![Firmware version mismatch between host and guest](/img/firmware_version_mismatch.png)

This is because klipper made changes to a part of the MCU firmware that we use. Klipper is telling us that the version of klipper running on the Pi is newer than the version running on the MCU. To fix this, we have to flash the board with a new version of the firmware. For convenience, and ease of use, the newest firmware is compiled and put in the `firmware_binaries` folder which you can find in the `MACHINE` tab in Mainsail. You can use this to flash your MCU the same way you did initially, via SD Card. RatOS attempts to flash supported boards automatically when klipper is updated. If you're getting this error you're probably using a board which does not support automatic flashing via USB. It's also possible that you updated klipper without having your board flashed and connected. In that case, use the `COMPILE_BINARIES` macro to generate new firmware binaries for all supported boards. Then download the binary for your board from the `firmware_binaries` folder in the Machine tab, and flash that via SD card.

## Unparsed config option 'config_path: ~/klipper_config' detected in section \[server\]
![Unparsed config option](/img/moonraker_unparsed_config.png)
This happens because moonraker has moved `config_path` and `log_path` from the `[server]` section to the `[file_manager]` section. You can fix this by moving those to options into the `[file_manager]` (create it if it doesn't already exist) in moonraker.conf.

Example:
```properties
[server]
host = 0.0.0.0
port = 7125
enable_debug_logging = False
config_path = ~/klipper_config
log_path = ~/klipper_logs
```

becomes
```properties
[server]
host = 0.0.0.0
port = 7125
enable_debug_logging = False

[file_manager]
config_path = ~/klipper_config
log_path = ~/klipper_logs
```

You may have to ssh into your raspberry pi and edit the file with nano:

```
ssh pi@ratos.local
nano ~/klipper_config/moonraker.conf
```

Use ctrl+o to write your changes to moonraker.conf and then ctrl+x to exit nano. Then run:

```
sudo systemctl restart moonraker
```

And you should be back in action.

## Section 'gcode_shell_command generate_shaper_graph_x' (or others) is not a valid config section
This happens if you hard reset klipper through mainsail, to relink the extensions go to the configurator at [http://ratos.local/configure](http://ratos.local/configure), click the "Actions" dropdown menu in the top right and pick "Symlink klippy extensions". Then go back to mainsail and restart the klipper service by clicking on the power icon in the top right corner, then the refresh icon next to "Klipper" under "Service controls". You should now be back in action. 

## Connection to moonraker failed
If you see the mainsail interface but you get an error about the connecting to moonraker, the connection to the pi is fine, but you're using a non standard IPv4 or IPv6 range that is not whitelisted in moonraker (only standard local ip ranges are whitelisted for security reasons). Try using the ip address of your pi (look it up in your routers admin interface) instead of ratos.local, or fix it by adding your IP range in CIDR notation to the `[authorization]` section in ~/klipper_config/moonraker.conf on the pi. You can do that through SSH, by running: 
```
ssh ratos.local
nano ~/klipper_config/moonraker.conf
sudo systemctl restart moonraker
```

Alternatively you can delete the entire `[authorization]` section, which will allow anyone to connect to moonraker (this is insecure, so do not do this if your network is open to - our your pi is reachable by - the outside world)

## Get help

For further support check out the RatOS-support and klipper channels on Discord. Use the invite link below to join.

<a href="https://discord.gg/ratrig" class="button button--primary">Join the Unnofficial RatRig Discord Community</a>
