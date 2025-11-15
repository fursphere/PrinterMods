


## Linux APT sources

The VentureXL ships with a list of Chinese update/apt sources which have been known to interfere with updates/installs. To change
this just run:

`sudo nano /etc/apt/sources.list`

and remove or comment out (with a #) all the ones that are there (they all point to mirrors.ustc.edu.cn). 

Then add:

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

Now to add the new ADAPTIVE_BED_HEAT control, simply download the adaptive_bed_heat.cfg file from [here](./macros/adaptive_bed_heat.cfg) and copy it to your printer 
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


## Safer Quad Gantry Level

The VentureXL has a large bed, so even a slight deviation in the corner levels can cause a multi-mm drop from one corner to the other. This 
is most likely to happen either after a print failure crash (due to a bad setting or command) or after doing work on the gantry/belts.

In order to *lessen* the chance of gouging your bed if the gantry is really out of tram simple add this macro to your printer.cfg somewhere

```
[gcode_macro QUAD_GANTRY_LEVEL]
rename_existing: _QUAD_GANTRY_LEVEL
gcode:
  RESPOND MSG="Custom QGL"
  {% if not printer.quad_gantry_level.applied %} 
    _QUAD_GANTRY_LEVEL horizontal_move_z=20 retries=1 retry_tolerance=1.000
  {% endif %}
  _QUAD_GANTRY_LEVEL horizontal_move_z=5
  G90
  G1 Z20 F500
  M400
  G28 Z
  ```

  It will take the stock QUAD_GANTRY_LEVEL command and split it into two sections. The first sweep is a rough QGL but done at a much higher
  Z moving position (20mm). This is to account for any potential drop the nozzle might go on.
  Once this has been roughly trammed in it will do a normal QGL to fine tune it.
  
## PRINT_START

The stock VentureXL PRINT_START macro is a bit lacking. This version is based off of it but with fixes. It uses the ADAPTIVE_BED_HEAT macro
from above as well as rearranging some things around.

```
[gcode_macro PRINT_START]
#   Use PRINT_START for the slicer starting script - please customise for your slicer of choice
gcode:
    {% set DEFAULT_EXTRUDER_TEMP = 150 %}
    {% set BED_TEMP = params.BED_TEMP|default(0)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(0)|float %}
	
	  ## Setup and initialisation ##
  	CLEAR_PAUSE
  	BED_MESH_CLEAR
  	M220 S100 ;Set Speed to 100%
  	M221 S100 #Set extrude factor to 100%
  	G92 E0 ; Reset Extruder
  	G90 ; Use absolute coordinates	
  	SET_GCODE_OFFSET Z=0 ;Remove any previous Gcode Offsets
  	
  	G28 #Home all axes
  	G1 Z10 #Just a safety rise
    STATUS_HEATING
    G90
    ADAPTIVE_BED_HEAT BED_TEMP={BED_TEMP}
    G28 Z #To account for any bed thermal expansion
  	QUAD_GANTRY_LEVEL RETRIES=10
  	G28 Z #Always Z home after QGL
  	M400
  	G90
  	G1 Z10
  	STATUS_MESHING
  	BED_MESH_CALIBRATE ADAPTIVE=1
    M109 S{DEFAULT_EXTRUDER_TEMP}
    STATUS_PART_READY
  
    #CARTOGRAPHER_TOUCH_HOME
  
    G90
    G1 Z10  
    M109 S{EXTRUDER_TEMP}
    LINE_PURGE 
    STATUS_PRINTING
```

It is recommeneded (but not mandatory) to use the Touch feature of the Cartographer probe installed on the VentureXL. This allows for more
accurate Z=0 homing using the nozzle tip itself.

To enable Cartograper Touch you should first update the Cartographer software as the latest software is likely different to what shipped
with the VentureXL. Simply go to <https://docs.cartographer3d.com/cartographer-probe/installation-and-setup/software-configuration/klipper-setup> 
and follow the steps. You will also need to change some of the settings in the printer.cfg to match the new Cartographer names/settings.

Then go through with the Calibration steps as per the Cartographer docs.

Once it is all calibrated you can just remove the `#` from the `#CARTOGRAPHER_TOUCH_HOME` line in the PRINT_START macro and it will now 
do a Cartographer Touch action during your print start routine.


## Homing Override

I have developed a "universal" HOMING_OVERRIDE config that should be able to be used with most printers. Using this instead of the stock
safe_z_home isn't strictly necessary but it *will* help avoid situations where you try to home and your Y endstop arm collides with the Z 
cable chain and breaks.

Changing over is simple. Just go into your printer.cfg and delete or comment out the `[safe_z_home]` section to disable it.

<img width="548" height="242" alt="image" src="https://github.com/user-attachments/assets/61cf65ae-add4-4292-a892-e01f6d2a7330" />

Then download the homing_override.cfg file from [here](./macros/homing_override.cfg) and copy it to your printer 
config directory. Then open your printer.cfg and add in the line

```
[include homing_override.cfg]
```

<img width="433" height="269" alt="image" src="https://github.com/user-attachments/assets/b23d99fa-1515-41a7-acab-4572125a16bc" />

Everything is preconfigured for the VentureXL. If you would like to tweak the behaviour (because you change or mod things in the future)
just open the homing_override.cfg file and you can change the X/Y/Z hop values, the homing axis order, or set a specific z homing
location (this would only be used if you removed the Cartographer and used a dedicated Z endstop switch).

<img width="1352" height="489" alt="image" src="https://github.com/user-attachments/assets/4d31215d-c287-43e9-bca4-7e04466aa319" />




## Hardware mods

when I get around to it....
