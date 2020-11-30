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
  * A library used in Openlane is `sky130_fd_sc_hd_typical.lib`. This has all the cells and its characteristics like temperature, voltage, timing definitions. An updated lib file also needs to be included as
  ```
  set ::env(LIB_SYNTH) "<Location to *__typical.lib>"
  set ::env(LIB_MIN) "<Location to *__fast.lib>"
  set ::env(LIB_MAX) "<Location to *__slow.lib>"
  set ::env(LIB_SYNTH) "<Location to *__typical.lib>"
  ```
  * The new cell needs to be included in OpenLANE flow libraries so that *abc* can pick it for synthesis. This requires `set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]` added to config.tcl and `set lefs [glob $::env(DESIGN_DIR)/src/*.lef]` and `add_lefs -src $lefs` to interactive flow. `runs/*/tmp/merged.lef` should have the new cell.
  * `run_synthesis` should pick the new cell. Update the script at `.../openlane/scripts/sta.tcl` as below after `read_sdc` statement. This process ends by providing timing information.
  ```
  report_checks -path_delay min_max -fields {input_pin slew net cap}
  report_checks > $::env(opensta_report_file_tag)_main.timing.rpt
  ```
  
  * OpenLANE switch information is present in `.../openlane/configuration/README.md`. Switches can be set as `set $::env(SYNTH_STRATERGY) 1`. Some switches updated to improve timing were SYNTH_STRATERGY=1, SYNTH_BUFFERING=1, SYNTH_SIZING=1, SYNTH_DRIVING_CELL=sky130_fd_sc_hd__inv_8, SYNTH_MAX_FANOUT=4
  * some abbreviations in the report are wns=worst network slack, tns=total network slack.
  * `run_floorplan` should converge on `OVFL`.The results would be in `runs/*/results/placement/*.placement.def`. .def files can be opened in Magic with `magic -T <link to tech file>/aky130A.tech lef read <link to lef file>/merged.lef def read <link to def file>/*.placement.def`
  * Magic `% expand` will show content of subcells.
  
* Delay tables TODO

## Timing Analysis with ideal clocks using OpenSTA
* Set-up timing analysis
* Clock jitter and uncertainty
* OpenSTA for post-synth timing analysis.
  * We can run OpenSTA outside openLane
  * A configuration file can be prepared as below and can be named as sta.conf
  ```
  set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
  read_liberty -min </location/to/min_lib>
  read_liberty -max </location/to/max_lib>
  read_verilog </location/to/verilog_file>
  link_design <Top-Module-name>
  read_sdc </location/to/sdc_file>
  report_checks -path_delay min_max -fields {slew trans net cap input_pin}
  report_tns
  report_wns
  ```
  * A SDC file needs to be prepared. It needs to have all variables defined as it is running outside of OpenLane. This is similar to file in `.../openlane/scripts/base.sdc`
  ```
  set ::env(CLOCK_PORT) clk
  set ::env(CLOCK_PERIOD) 12.000
  set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv_8
  set ::env(SYNTH_DRIVING_CELL_PIN) Y
  set ::env(SYNTH_CAP_LOAD) 17.65
  create_clock [get_ports $::env(CLOCK_PORT)]  -name $::env(CLOCK_PORT)  -period $::env(CLOCK_PERIOD)
  set IO_PCT  0.2
  set input_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
  set output_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
  puts "\[INFO\]: Setting output delay to: $output_delay_value"
  puts "\[INFO\]: Setting input delay to: $input_delay_value"

  set clk_indx [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
  #set rst_indx [lsearch [all_inputs] [get_port resetn]]
  set all_inputs_wo_clk [lreplace [all_inputs] $clk_indx $clk_indx]
  #set all_inputs_wo_clk_rst [lreplace $all_inputs_wo_clk $rst_indx $rst_indx]
  set all_inputs_wo_clk_rst $all_inputs_wo_clk

  # correct resetn
  set_input_delay $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
  #set_input_delay 0.0 -clock [get_clocks $::env(CLOCK_PORT)] {resetn}
  set_output_delay $output_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

  set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) -pin $::env(SYNTH_DRIVING_CELL_PIN) [all_inputs]
  set cap_load [expr $::env(SYNTH_CAP_LOAD) / 1000.0]
  puts "\[INFO\]: Setting load to: $cap_load"
  set_load  $cap_load [all_outputs]
  ```
  * Then STA can be run as `$ sta 
  * Hold-time is significant only after Clock Tree Synthesis(CTS). Before CTS, timing analysis can be considered optimistic. We can measure slew and delay before CTS. Slew is rate of rise or fall of signal. Delay of cell is funtion of input slew and output load. So, more the slew, more would be the delay. More the output capacitance, more would be the delay.
  * Place and Route (PNR) is an iterative process. We keep checking timing and DRC and rerun with a different settings.
  * We can analyze details in Openlane after running `run_synthesis` with following commands
    * `report_net -connections _*****_`
    * `replace_cell _*****_ sky130_fd_sc_*`
    * 
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
