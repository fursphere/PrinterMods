


## Linux APT sources

`sudo nano /etc/apt/sources.list`

Remove or comment out (with a #) all the ones that are there (they all point to mirrors.ustc.edu.cn) and instead add:

```
deb http://deb.debian.org/debian bullseye main contrib
deb-src http://deb.debian.org/debian bullseye main contrib

deb http://deb.debian.org/debian bullseye-updates main contrib
deb-src http://deb.debian.org/debian bullseye-updates main contrib

deb http://deb.debian.org/debian bullseye-backports main contrib
deb-src http://deb.debian.org/debian bullseye-backports main contrib

deb http://security.debian.org/debian-security/ bullseye-security main contrib
deb-src http://security.debian.org/debian-security/ bullseye-security main contrib
```

<img width="847" height="413" alt="image" src="https://github.com/user-attachments/assets/48523502-7edb-4d7c-8de8-a04f0f509d3d" />


then ctrl-x to save and quit

## Adaptive Bed Heat

I have created a macro that will optimise bed heating for the Venture XL. For starters, it will only heat up the beds that are actually
in use (unless you have a chamber temperature sensor and a desired chamber temp in which case it will always heat all four beds).
Also, it will only heat the beds one at a time in 10 degree increments so in order to minimise the amount of power your beds will draw
all at once.

Before installing this new macro, you want to disable the KAMP-based functionality that tries to do the same thing but worse (and doesn't 
work properly anyway on the VentureXL).

Go into your printer config directory (where your printer.cfg lives) and open the KAMP_Settings.cfg file. **DO NOT** open the KAMP_Settings.cfg
file that lives inside the KAMP folder. There are two copies of the file and only the one in your **main** config directory is actually used.

In the KAMP_Settings.cfg simply comment out (put a `#` in front) of the `[include ./KAMP/Adaptive_Meshing.cfg]` line. 

<img width="1259" height="195" alt="image" src="https://github.com/user-attachments/assets/385aef6e-923d-4b39-a0d0-b8cd331a8ef9" />

This will disable the KAMP-based adaptive meshing and the KAMP bed heating. We don't need KAMP meshing anymore because Klipper can do it 
natively.

Now to add the new ADAPTIVE_BED_HEAT control, simply download the adaptive_bed_heat.cfg file from **HERE** and copy it to your printer 
config directory. Then open your printer.cfg and add in the line

```
[include adaptive_bed_heat.cfg]
```

and do a save and restart.

<img width="579" height="216" alt="image" src="https://github.com/user-attachments/assets/f75cf09a-48c8-4efc-b0b0-0ede822cb607" />

This will now allow you to use the `ADAPTIVE_BED_HEAT` macro either in your PRINT_START macro or just manually in your printer console
(or any other macros you care to write).

ADAPTIVE_BED_HEAT accepts two parameters. `BED_TEMP=` and `CHAMBER_TEMP=`

The BED_TEMP parameter is simply the bed temperature you wish to reach. CHAMBER_TEMP is if you use a chamber temperature sensor and
have a target chamber temperature you would like to reach. The ADAPTIVE_BED_MESH macro doesn't actually *do* much for you to reach your 
chamber temp, but if you have a CHAMBER_TEMP= parameter then it will always heat all 4 beds under the assumption you want as much heating
power available in order to heat up the chamber.

If you are printing something like PLA that doesn't need a warm chamber simply run:

`ADAPTIVE_BED_HEAT BED_TEMP=60` (or whatever bed temp you want for your filament).

If you have a chamber temp sensor and are printing a filament such as ABS and have a chamber temperature sensor then you could do something
like:

`ADAPTIVE_BED_HEAT BED_TEMP=100 CHAMBER_TEMP=60`

Note that the adaptive_bed_heat.cfg also includes a "redirect" for the M190 command. So if you run a simple `M190 S60` it will be the same
as running `ADAPTIVE_BED_HEAT BED_TEMP=60` (note that the M190 redirect can't handle the CHAMBER_TEMP option).

Instructions on how to integrate this macro into your PRINT_START are below.


## PRINT_START


## Homing Override


## Hardware mods

