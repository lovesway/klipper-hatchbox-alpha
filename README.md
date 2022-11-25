# Klipper Hatchbox Alpha

This repo contains files related to updating the Hatchbox Alpha 3D printer so that it will work with Klipper platforms. To accomplish this the firmware on the 3D printer must be overwritten which is dangerous and can leave the printer in an unusable state; I do not recommend doing this, but I also realize that you are an infinite creator! If anyone attempts doing this, they should be capable of restoring the machine back to the original firmware. You should also double check all of the pin settings in the printer.cfg and verify that your microcontroller hardware is the same as mine (I have no idea whether or not all Hatchbox Alphas use the same microcontrollers).

I had an older copy of the firmware which I gotten from 
https://github.com/mikekirby/Repetier-Firmware
This is what I had been using as the basis for my Hatchbox Alpha's firmware before upgrading to Klipper. I had also modded my Hatchbox Alpha by adding a v6 hot end which required adding an additional fan and designing a new effector; I will not be describing these mods in this repo.

The impetus of my upgrade was that of getting a new 3D printer. I got an FLSUN v400 and realized that the Speeder pad that it used could be set up to run multiple printers using Klipper; https://github.com/Guilouz/Klipper-Flsun-Speeder-Pad
The FLSUN Speeder Pad platform uses the following components;
-   [Klipper (Operating System)](https://www.klipper3d.org/)
-   [Moonraker (API Web Server)](https://moonraker.readthedocs.io/)
-   [Mainsail (Web Interface)](https://docs.mainsail.xyz/)

You should be able to port these ideas to other platforms like [Octoprint](https://octoprint.org/) which also support Klipper.

Since upgrading to Klipper I've experienced a whole new printer, everything works and it's been amazing for me! I'm sure that there are improvements to be had, please reach out to me if you've spotted anything that needs changed or if you've found your own improvements; what is here represents my first attempt.

## Updating the Firmware on the Hatchbox Alpha
In order for any of this to work, you need to update your current firmware so that it uses the Klipper firmware. Although daunting, this task is actually fast and relatively easy.

You will need a linux server with USB to update the firmware. A good suggestion is simply to use the device you are using to run your web interface which should be a Raspberry Pi or the likes. In my case I am using my Speeder pad. It's good to reference and follow this document as you proceed;
https://www.klipper3d.org/Installation.html

Don't worry about the configuration file at this point, we cover that next. For now we simply need to compile the firmware and upload it.

The following commands presume that you are already logged into your linux server via ssh. For example, I access my server via the following command; `ssh pi@speeder-pad`

Checkout the Klipper repo:

```
git clone https://github.com/Klipper3d/klipper klipper-hatchbox_alpha
cd klipper-hatchbox_alpha/
```

Configure Klipper to build for the Hatchbox Alpha. My Hatchbox Alpha is using an MKS Ramps 1.4 board with an Atmega2560. 

```
make menuconfig
```

Running the menuconfig command will present a menu allowing you to set your build options.
Set these options:
- Enable extra low-level configuration options
-- Processor model (atmega2560)
-- Processor speed (16Mhz)
-- Communication interface (UART0)
- (115200) Baud rate for serial port

After setting your options select `Q` and save. Then build your firmware using the make command.

```
make
```

After  the build completes you can upload it to your Hatchbox Alpha!
Note the `Atmega2560` section in https://www.klipper3d.org/Bootloaders.html

Find the right USB serial port to use with `ls /dev/serial/by-id/*`
Write the firmware, replacing `/dev/ttyACM0` with the appropriate USB port.

```
avrdude -cwiring -patmega2560 -P/dev/ttyACM0 -b115200 -D -Uflash:w:out/klipper.elf.hex:i
```

This was almost instant for me. After the firmware is updated, your printer is ready and running the Klipper OS! Next you need to configure your web interface, in my case I am using Mainsail.

## Configuration Files

This is the printer.cfg and macros.cfg that I used in the Mainsail web interface to set up the Hatchbox Alpha printer. These were ported from the FLSUN v400 implementation, hopefully I've not carried over any artifacts which are not relative to the Hatchbox Alpha.

- [printer.cfg](printer.cfg)
- [macros.cfg](macros.cfg)

The printer.cfg should be pretty vanilla apart from my addition of the secondary fan pin. If you have not updated your extruder/fan and you only have 1 fan then you will likely need to remove the section described as `[heater_fan Hotend]`.

Find your printers USB port with `ls /dev/serial/by-id/*` and update it in the `printer.cfg`.


The macros were copied over from the FLSUN v400, they seem to work well. Note that I use the `Delta Calibration` macro in lieu of Auto Leveling. There may be better, more extensive, ways to do leveling, but Delta Calibration will work with the existing Z-Probe and and seems to be an improvement on the old Auto Leveling behavior.

### Start Print GCODE
I borrowed the Start G-code that the FLSUN v400 used in Cura. It works well and I've adjusted only the arc radius so that it doesn't hit the outer clips on the print bed. This is not at all required, it's just a thing I've enjoyed.
```
G21
G90
M82
M107 T0
M140 S{material_bed_temperature}
M104 S{material_print_temperature} T0
M190 S{material_bed_temperature}
M109 S{material_print_temperature} T0
G28
G1 F3000 Z1
G1 X-140 Y0 Z0.4
G92 E0
G3 X0 Y-130 I140 Z0.3 E30 F2000
G92 E0
```