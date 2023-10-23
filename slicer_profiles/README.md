# Importing Slicer profiles into PrusaSlicer

## Importing profiles

Importing slicer profiles are quite simple. In the included `.ini` file, profiles for `ABS` and `TPU` have been included. To import these profiles into `PrusaSlicer`, simply click `File > Import > Import Config Bundle` and select the file. This will import the `printer` profiles for `0.4mm` and `0.6mm` nozzle configurations, profiles for `ABS` and `TPU` as well as the default slicer profiles which can be modified by the user

## Starting G-code

For correct operations of the printer, it is required to include specific starting `G-code`, this `G-code` is provided below and can be imported into other slicers. However, this `G-code` uses `variables` from `PrusaSlicer` and might require changes to be made to be used in other slicers:

```gcode
_PRINT_START_PHASE_INIT EXTRUDER={first_layer_temperature[initial_tool]} BED=[first_layer_bed_temperature] MESH_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} MESH_MAX={first_layer_print_max[0]},{first_layer_print_max[1]} LAYERS={total_layer_count} NOZZLE_SIZE={nozzle_diameter[0]}

; Load bed
BED_MESH_PROFILE LOAD="default"
SET_SURFACE_OFFSET OFFSET=0.149

; Insert custom gcode here.
_PRINT_START_PHASE_PREHEAT

; Insert custom gcode here.
_PRINT_START_PHASE_PROBING

; Insert custom gcode here.
_PRINT_START_PHASE_EXTRUDER

; Insert custom gcode here.
_PRINT_START_PHASE_PURGE
```

In this `G-code` the most important part is:

```gcode
_PRINT_START_PHASE_INIT EXTRUDER={first_layer_temperature[initial_tool]} BED=[first_layer_bed_temperature] MESH_MIN={first_layer_print_min[0]},{first_layer_print_min[1]} MESH_MAX={first_layer_print_max[0]},{first_layer_print_max[1]} LAYERS={total_layer_count} NOZZLE_SIZE={nozzle_diameter[0]}
```

This `G-code` initializes the print. It sets the temperature for the `EXTRUDER` and `BED`. Furthermore, it also specifies the `MESH_MIN` and `MESH_MAX`. This allows the slicer to tell the printer what the `min` and `max` dimensions of the print are. This is usefull for `adaptive mesh leveling`, which only levels the print for the area of the print. This greatly reduces leveling time and also allows the user to increase the resolution of the leveling, without requiring to `probe` the entire bed. `LAYERS` tells the printer how many layers there are and is used for `time estimates` and other fun features such as `GCODE AT LAYER` and other more advanced features such as color changes. Finally `NOZZLE_SIZE` tells `Klipper` what the size of the nozzle is and allows `Klipper` to perform more advanced compensation and flow control.

`SET_ENCLOSURE_TEMPERATURE TARGET=50.0` could also be added to set the temperature for the enclosure. Where `50.0` can be changed to any other temperature. The max temperature for the enclosure is however restricted to `60.0` in the firmware to prevent overheating and for safety.

The following section is used to specify the `printing surface` for `Klipper`:

```gcode
; Load bed
BED_MESH_PROFILE LOAD="default"
SET_SURFACE_OFFSET OFFSET=0.149
```

This tells `Klipper` that it should use the printing surface with the name `default` and is useful if the user has many different build plates which have different thicknesses and leveling meshes. It also specifies the `surface z-offset`, which is useful for different filaments requiring different amounts of squish for the first layer.

```gcode
; Insert custom gcode here.
_PRINT_START_PHASE_PREHEAT

; Insert custom gcode here.
_PRINT_START_PHASE_PROBING

; Insert custom gcode here.
_PRINT_START_PHASE_EXTRUDER

; Insert custom gcode here.
_PRINT_START_PHASE_PURGE
```

The last segment is implement using an open-source library called `KlipperMacros`, users can read more about it [here](https://github.com/jschuh/klipper-macros). This is used to level the bed, probe and perform an initial purge, which is automatically calculated based of the aforementioned `MIN` and `MAX` mesh parameters.

Finally, the following end `G-code` should be added in the `END Gcode` section:

```gcode
PRINT_END
SET_ENCLOSURE_TEMPERATURE TARGET=0.0
; total layers count = [total_layer_count]
```

Here `PRINT_END` tells the printer that the end of the print has been reached and `SET_ENCLOSURE_TEMPERATURE TARGET=0.0` tells the printer to ensure that the enclosure heater turns off.

