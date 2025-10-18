Note: This is a forked repo and a lot of this page is from the original, with slight modifications and some additions. For info on what I've changed from the original, check the detailed descriptions in the commit history on the Heatbed_Soak.cfg file for specific details on what I've changed from the original. 
This macro is less complicated than other ones I found online, so I chose this one to tinker with. What it's missing, I think, is the ability to flex the time of the heat soak if the bed was very recently at temperature. I may or may not add this later.

# Heatbed Soak Macro for Klipper 3D Printers

### Why use a heat soak macro?
It is well known that the print bed on 3D Printers can continue to flex and move even after the thermister reports that the print bed has reached its target temperature. This is due to the thermal expansion of the metal. Some 3D printers need it more than others. I've noticed my Elegoo Neptune 4 Pro performs significantly worse when I don't heat soak the bed, which may be partially due to the design of the frame underneath the bed, but this is not a unique issue to Elegoo printers. Here is an example from a redditor with a Sovol printer:
<img width="3331" height="1563" alt="image" src="https://github.com/user-attachments/assets/143e716a-be73-4059-8c03-65858d8f8391" />

"I used rapid scan with a btt eddy sensor to check the mesh of the bed every minute while setting the bed temp to 65c. each frame is 1 minute apart with the first frame taken when the bed was cold." - u/manatails, source: https://www.reddit.com/r/3Dprinting/comments/1lrn1gb/bed_heat_soaking_timelapse_the_visual_reason_you/ 

This issue becomes more prevalent the larger a print bed is, and the larger your print is. 
This movement can cause any bed levelling mesh routine to measure values that will subsequently change after it was measured, leading to poor first layer print quality and potential print failures. You can test this yourself by generating a new bed mesh before and after heat soaking your printer's bed. 

### What this macro does
This macro takes note of how long it takes for a print bed to reach a target temperature and will then wait for as long again plus two more minutes, to ensure that the print bed's temperature has fully equalised.
If the print-bed was already pre-warmed (as in, your print end gcode doesn't have any cool down functions and your printbed heater remains on at the same temp) then this macro will use the default heat soak DURATION of 5 minutes. 
At given intervals the time spent waiting and the time remaining is reported back to the printer's UI console.

## Dependencies
1. You will get the most use out of this macro if you're using adaptive bed meshing. KAMP works, so does OrcaSlicer's built-in ABM feature. The bed leveling shouldn't begin until after the bed is done heat soaking, obviously. Both KAMP and Orca's ABM have their own dependenceies, like requiring your printer.cfg file to have already defined labeling or excluding objects.
2. "PAUSE" config section must be defined in your firmware, ie within your printer.cfg file (this is different from original, where PAUSE_BASE was used and was incompatible with my firmware)
3. I forget what I was going to put here but there was something else. I will return

## How to install

To use this macro, import the Heatbed_Soak.cfg file from this repository into your printer's configuration directory

Now edit your `printer.cfg` and include the path to the `Heatbed_Soak.cfg` macro file

For example:

```
[include Heatbed_Soak.cfg]
```

The above assumes that the `Heatbed_Soak.cfg` file was placed in the same directory as your `printer.cfg` file.

## How to Use

Inside your slicer there will be a printer configuration panel which defines the Gcode that the printer will
run when starting a print.

You will want add the following `HEAT_SOAK` line, preferably just before the first line that has a `G28` command on it. The original example below uses [first_layer_bed_temperature] as the placeholder for the target temperature, but just use whatever is used in your functioning M190 command. For my N4Pro, my custom start gcode in OrcaSlicer is as follows, but be warned it is flawed:
```
HEAT_SOAK TARGET=[bed_temperature_initial_layer_single]
G28 ;home
;M190 S[bed_temperature_initial_layer_single] ; Set the bed temperature
G92 E0 ;Reset Extruder
; Always pass `ADAPTIVE_MARGIN=0` because Orca has already handled `adaptive_bed_mesh_margin` internally
; Make sure to set ADAPTIVE to 0 otherwise Klipper will use it's own adaptive bed mesh logic
BED_MESH_CALIBRATE mesh_min={adaptive_bed_mesh_min[0]},{adaptive_bed_mesh_min[1]} mesh_max={adaptive_bed_mesh_max[0]},{adaptive_bed_mesh_max[1]} ALGORITHM=[bed_mesh_algo] PROBE_COUNT={bed_mesh_probe_count[0]},{bed_mesh_probe_count[1]} ADAPTIVE=0 ADAPTIVE_MARGIN=0
G1 Z4.0 F3000 ;Move Z Axis up
M109 S[nozzle_temperature_initial_layer] ; Wait until the nozzle reaches the desired temperature
G92 E0 ;Reset Extruder
G1 X1.1 Y20 Z0.28 F5000.0 ;Move to start position
G1 X1.1 Y80.0 Z0.28 F1500.0 E10 ;Draw the first line
G1 X1.4 Y80.0 Z0.28 F5000.0 ;Move to side a little
G1 X1.4 Y20 Z0.28 F1500.0 E20 ;Draw the second line
G92 E0 ;Reset Extruder
G1 Z2.0 F3000 ;Move Z Axis up
G92 E0
G92 E0
G1 F2700 E-0.5
```

Original example:

```
...
HEAT_SOAK TARGET=[first_layer_bed_temperature]
G28
...
```

If you see any `M140` or `M190` commands, these can be safely commented out by placing a `;` at the front of those lines as shown above.

This following image may help to clarify where to edit the printer's start g-code in your slicer software (original example)

![Printer Start Gcode](https://github.com/stew675/heatbed-soak-macro/blob/main/Heat-Soak.jpg)

I also use Orca, so it is in the same place. Make sure to properly configure ABM as well (the values below will only work for the N4Pro)
<img width="1292" height="306" alt="image" src="https://github.com/user-attachments/assets/6d436c90-f341-4556-8b90-cfacd127d06c" />

