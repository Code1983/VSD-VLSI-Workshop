# Advanced Physical Design using OpenLANE/Sky130 Workshop
VSD VLSI Workshop 25-29 Nov 2020

# Workshop Structure
Day1 – Inception of open-source EDA, OpenLANE and Sky130 PDK

* How to talk to computers
* SoC design and OpenLANE
* Starting RISC-V SoC Reference design
* Get familiar to open-source EDA tools

Day 2 - Understand importance of good floorplan vs bad floorplan and introduction to library cells

* Chip Floor planning considerations
* Library Binding and Placement
* Cell design and characterization flows
* General timing characterization parameters

Day 3 - Design and characterize one library cell using Magic Layout tool and ngspice

* Labs for CMOS inverter ngspice simulations
* Inception of Layout – CMOS fabrication process
* Sky130 Tech File Labs

Day 4 - Pre-layout timing analysis and importance of good clock tree

* Timing modelling using delay tables
* Timing analysis with ideal clocks using openSTA
* Timing analysis with real clocks using openSTA

Day 5 - Final steps for RTL2GDS

* Routing and design rule check (DRC)
* PNR interactive flow tutorial

# Day 1: Intro to EDA tools, Openlane, RISC-V

Terms
* Package
* chip
* pads
* core
* die
* IP

Components of open-source digital ASIC design

RTL to GDS Flow

## OpenLANE
Openlane detailed ASIC design flow
![OpenLANE FLow](https://github.com/efabless/openlane/blob/master/doc/openlane.flow.1.png)
*Image from OpenLANE git*

OpenLANE interactive commands

* `./flow.tcl -interactive` on terminal at OpenLANE root folder
* `% package require openlane 0.9` openlane package sourced on tcl shell.
* `prep -design <design> -tag <tag> -config <config> -init_design_config -overwrite` similar to the command line arguments, design is required and the rest is optional
* `run_synthesis` 
* `run_floorplan`
* `run_placement`
* `run_cts`
* `gen_pdn`
* `run_routing`
* `run_magic`
* `run_magic_spice_export`
* `run_magic_drc`
* `run_lvs`
* `run_magic_antenna_check`

More details are in [Openlane Git](https://github.com/efabless/openlane)
# Day 2: Chip Floor Planning and Placement, Cell Design and characterization

# Day 3: 16-Mask CMOS fabrication process, ngspice simulations, MAGIC Tech File update for DRC

## MAGIC DRC Engine
Different DRC styles
Basic rules type
Complicated rule types
Rules handled separatly from interactive DRC Engine

### Finding problems in DRC section of MAGIC tech file for SKY130 PDK

**Important links**
* [Using MAGIC](http://opencircuitdesign.com/magic/userguide.html)
* [MAGIC Tech Files](http://opencircuitdesign.com/magic/techref/maint2.html)
* [PDK Documentation](https://skywater-pdk.readthedocs.io/en/latest/)

# Day 4: Timing Analysis

# Day 5: Place and Route
