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

## Basic Concepts
Terms
* Package
* chip
* pads
* core
* die
* IP

## SOC Design and OpenLANE



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

Characterize synthesis results

# Day 2: Chip Floor Planning and Placement, Cell Design and characterization

## Chip Floor planning considerations
* Utilization factor and aspect ratio
* Preplaced Cells
* De-coupling Capacitors
* Power Planning
* Pin Placement and logical cell placement blockage
* Run floorplan using OpenLANE
* Review Floorplan layout in MAGIC

## Library Binding and Placement
* Netlist binding and initial place design
* Optimize placement optimization
* Final placement optimization
* libraries and characterization
* Congestion awareness

## Cell design and characterizzation flows
* Inputs for cell design flow
* Circuit design step
* Layout design step
* Typical characterization flow

## General timining characterization parameters
* Timing threshold functions
* Propagation delay and transition time

# Day 3: 16-Mask CMOS fabrication process, ngspice simulations, MAGIC Tech File update for DRC

## CMOS inverter ngspice simulations
* IO placer revision
* Spice deck creation and simulation
* Switching threshold Vm
* Static and dynamic simulation
* Standard Cell design

## CMOS Fabrication process
* Create Active regions
* Formation and N-well and P-well
* Formation of gate terminal
* Lightly Doped drain (LDD) formation
* Source - drain formation
* Local interconnect formation
* Higher level metal formation

## Using Sky130 tech file
* Sky130 layers and LEF using inverter
* Create standard cell layout and extract spice netlist
* Create Spice deck using Sky130 tech
* Characterrize inverter using sky130 model


## MAGIC DRC Engine
* Different DRC styles
* Basic rules type
* Complicated rule types
* Rules handled separatly from interactive DRC Engine

### Finding problems in DRC section of MAGIC tech file for SKY130 PDK

**Important links**
* [Using MAGIC](http://opencircuitdesign.com/magic/userguide.html)
* [MAGIC Tech Files](http://opencircuitdesign.com/magic/techref/maint2.html)
* [PDK Documentation](https://skywater-pdk.readthedocs.io/en/latest/)

# Day 4: Timing Analysis

## Timing modelling using delay tables
* Converting MAGIC layout to standard cell LEF and including custom standard cell in Openlane synthesis and fix slack
  * Guidelines
    * Input and output port must lie in the intersection of the vertical and horizontal tracks
    * Width of the standard cell = odd multiples of horizontal track pitch
    * Height of the standard cell = odd multiples of horizontal track pitch. Sky130 track information is present in `.../sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info` in formatted in columns layer, direction, offset, pitch
  * pressing "g" in Magic activates grid. This can be verified by a black dot towards bottom left corner.
  * Magic `% grid` command draws the grid. `% help grid` will show the format as `grid xSpacing ySpacing xOrigin yOrigin`This can be set as per track definition and to review the positioning of ports and standard cell width and hieght.
  * Port definition is required before LEF can be extracted from MAGIC aas per instruction [here](https://github.com/nickson-jose/vsdstdcelldesign)
  * LEF can be extracted by `% lef write` in Magic
  * A library used in Openlane is `sky130_fd_sc_hd_typical.lib`. This has all the cells and its characteristics like temperature, voltage, timing definitions.
  * The new cell needs to be included in OpenLANE flow libraries so that *abc* can pick it for synthesis. This requires `set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]` added to config.tcl and `set lefs [glob $::env(DESIGN_DIR)/src/*.lef]` and `add_lefs -src $lefs` to interactive flow. `runs/*/tmp/merged.lef` should have the new cell.
  * `run_synthesis` should pick the new cell. This process ends by providing timing information.
  * OpenLANE switch information is present in `.../openlane/configuration/README.md`. Switches can be set as `set $::env(SYNTH_STRATERGY) 1`. Some switches updated to improve timing were SYNTH_STRATERGY=1, SYNTH_BUFFERING=1, SYNTH_SIZING=1, SYNTH_DRIVING_CELL=sky130_fd_sc_hd__inv_8
  * some abbreviations in the report are wns=worst network slack, tns=total network slack.
  * `run_floorplan` should converge on `OVFL`.The results would be in `runs/*/results/placement/*.placement.def`. .def files can be opened in Magic with `magic -T <link to tech file>/aky130A.tech lef read <link to lef file>/merged.lef def read <link to def file>/*.placement.def`
  * Magic `% expand` will show content of subcells.
  
* Delay tables TODO

## Timing Analysis with ideal clocks using OpenSTA
* Set-up timing analysis
* Clock jitter and uncertainty
* OpenSTA for post-synth timing analysis
* optimizing synthesis to reduce set-up violations
* Baisc timing ECO

## Clock tree systesis using TritonCTS
* Clock tree routing and buffering using H-tree algorithm
* Crosstalk and clock net shielding
* using TritonCTS

## Timing Analysis with real clocks using OpenSTA
* Setup timing analysis using real clocks
* Hold timing analysis using real clocks
* Analyze timing with real clocks using OpenSTA
* OpenSTA with right timing libraries and CTS
* Impact of bigger CTS buffers on set-up and hold timing

# Day 5: Place and Route

## Routing and design rule check (DRC)
* Introduction to Maze Routing - Lee's algorithm
* Design Rule Check

## Power distribution network and routing
* Building power distribution network
* power straps to standard cell power
* gobal and detail routing and configuring TritonRoute

## TritonRoute features
* Honors pre-processed route guides
* Inter-guide connectivity and intra & inter-layer routing
* Routing topology ad final files list post route
