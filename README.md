# PhysicalDesign_using_OpenLane_Sky130
Advanced Physical Design project - A complete P&R flow was carried out on Picorv32 SoC using OpenLane SkyWater-130nm Pdk on riscV, as part of the course "Advanced Physical Design using OpenLANE/Sky130" conducted by VLSI System Design Corporation. PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set which is used as an example in this course.

# Contents
1. Introduction to OpenLane
2. OpenLane-invoking and Design Preparation
3. Synthesis
4. Floorplaning
5. Placement
6. Std. Cell Design


# Introduction to OpenLane
OpenLane is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, CVC, SPEF-Extractor, KLayout and a number of custom scripts for design exploration and optimization. The flow performs all ASIC implementation steps from RTL all the way down to GDSII. 
You can check out the documentation, including in-depth guides and reference manuals at ReadTheDocs.

# OpenLane Architecture
![flow_v1](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/33d0363c-80df-4450-a0e1-eaa5c4fd88c8)

A diagram showing the general stages of the OpenLane flow as a series of blocks

# OpenLane Design Stages
OpenLane flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLane can also be run interactively as shown here.
  - Synthesis
      * yosys/abc - Perform RTL synthesis and technology mapping.
      * OpenSTA - Performs static timing analysis on the resulting netlist to generate timing reports
  - Floorplaning
      * init_fp - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
      * ioplacer - Places the macro input and output ports
      * pdngen - Generates the power distribution network
      * tapcell - Inserts welltap and decap cells in the floorplan
  - Placement
      * RePLace - Performs global placement
      * Resizer - Performs optional optimizations on the design
      * OpenDP - Performs detailed placement to legalize the globally placed components
  - CTS
      * TritonCTS - Synthesizes the clock distribution network (the clock tree)
  - Routing
      * FastRoute - Performs global routing to generate a guide file for the detailed route
      * TritonRoute - Performs detailed routing
      * OpenRCX - Performs SPEF extraction
  - Tapeout
      * Magic - Streams out the final GDSII layout file from the routed def
      * KLayout - Streams out the final GDSII layout file from the routed def as a back-up
  - Signoff
      * Magic - Performs DRC Checks & Antenna Checks
      * KLayout - Performs DRC Checks
      * Netgen - Performs LVS Checks
      * CVC - Performs Circuit Validity Checks

# Invoking OpenLANE and Design Preparation

  ![Invoking_Openlane](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/643b2588-3d0e-4d05-bc12-1dcef19ea74c)

Preparation step basically sets up the directory structure, merges the technology LEF (.tlef) and cell LEF(.lef) into one. Tech LEF contains the layer informations and cell LEF contains the cell informations. All the designs are placed under the designs directory for openLANE flow. 
Directory structure of picrorv32a before and after executing prep command.
src: contains verilog files and constraints file 
config.tcl: contains the configurations used by openLANE

There are three configuration files:

* Each phase used in the process flow has a configuration tcl file under openlane_working_dir/openlane/configuration/<phase_name>.tcl
* Each design will have its own config.tcl file
* Each design will have its own pdk specific tcl file, sky130A_sky130_fd_sc_hd_config.tcl which has the highest precedence.

# Synthesis
To synthesize the design
```
% run_synthesis
```
  - Running Synthesis
  ![RunningSynthesis](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/2d66c092-2e9a-40a5-97e1-7047fc8a2ae0)

  -Calculating Flip Flop ratio
  ![No_of_flip-flops](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/7d8633c4-076d-4a9b-8b0f-ad1912e25056)

dfxtp_2 = 1613,
Number of cells = 14876,
Flop ratio = 1613/14876 = 0.1084 = 10.84%

Synthesis logs and report will be captured under runs directory.
All the configuration parameters related to synthesis phase are available in synthesis.tcl

# Floorplaning

Floorplanning phase deals with setting die area, core area, core utilization factor, aspect ratio, placing of macros, decoupling capacitors, power distribution networks and placement of IO pins.

To run floorplanning phase
```
% run_floorplan
```
  - Running Floorplan
    

Floor planning phase generate DEF file which contains core area and placement details of standard cells.

   - To see Floorplan in Magic   
   DEF file generated by floorplan phase can be utilized by magic tool to get the floorplan view which requires 3 configuration files:
    * Magic technology file (sky130A.tech)
    * DEF file from floorplan phase
    * Merged LEF file from preparation phase


# Placement

Placement determine the locations of standard cells or logic elements within each block.Some circuit elements may have fixed locations while others are movable.

Global placement:
Global placement assigns general locations to movable objects. Some overlaps are allowed between placed objects.

Detailed placement:
Detailed placement refines object locations to legal cell sites and enforces non-overlapping constraints.
Detailed placement determines the achievable quality of the subsequent routing stages.

To run placement phase
```
% run_placement
```
  - Running Placement


  - To see Floorplan in Magic   
    

# Standard Cell Design

Standard cell design flow consists of 3 stages
* Inputs: PDKs, DRC and LVS rules, SPICE models, library & user-defined specs.
* Design Steps:  Involves circuit design, layout design, characterization using GUNA tool. Characterization involves timing, power and noise characterizations.
*Outputs:  CDL (Circuit Description Language), GDSII, LEF(Library Exchange Format), Spice extracted netlist, timing, noise, power libs.


Standard cell characterization refers to gathering data about the behaviour of standard cells. To build the circuit knowledge of logic function of cell alone is not sufficient.
Standard cell library has cells with different drive strength and functionalities.These cells are characterized by using tool like GUNA from https://www.paripath.com/home[`Paripath`].

The standard cell characterization flow involves

* Read the model files
* Read the extracted spice netlist
* Recognize function or behaviour of the cell
* Apply stimulus and characterization setup
* Vary the output load capacitance and observe the different characterization behaviours
* Provide necessary simulation commands

Apply the entire flow to GUNA tool to generate timing, noise and power models.

=== Design and characterize library cell CMOS inverter

Magic layout view to cmos inverter::
To get the cell files refer https://github.com/nickson-jose/vsdstdcelldesign[`standard cell characterization`]

Magic layout view to cmos inverter:



To extract the parasitics and characterize the cell design use below commands in tkcon window:


Extracted spice deck file from the layout:



Few modifications needs to be done in spice deck file:


To run the simulation in ngspice, invoke the ngspice tool with the modified extracted spice file as input:



To plot transient analysis output, where y - output node and a - input node
```
plot y vs time a
```








