# PhysicalDesign_using_OpenLane_Sky130
Advanced Physical Design project - A complete P&R flow was carried out on Picorv32 SoC using OpenLane SkyWater-130nm Pdk on riscV, as part of the course "Advanced Physical Design using OpenLANE/Sky130" conducted by VLSI System Design Corporation. PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set which is used as an example in this course.

# Author

# Contents
1. [Introduction to OpenLane](#introduction-to-openlane)
2. [OpenLane-invoking and Design Preparation](#invoking-openlane-and-design-preparation)
3. [Synthesis](#synthesis)
4. [Floorplaning](#floorplaning)
5. [Placement](#placement)
6. [Std. Cell Design](#standard-cell-design)
7. [Timing analysis and Clock tree synthesis](#timing-analysis-and-clock-design)
8. [Final steps in RTL to GDSII](#final-steps-in-rtl-to-gdsii)
9. [Conclusion](#conclusion)
10. [Acknowledgements](#acknowledgements)


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
![Floorplan_successful](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/ec6cf76d-e4fa-4e1d-9b63-f45a68241593)

Floor planning phase generate DEF file which contains core area and placement details of standard cells.

   - To see Floorplan in Magic   
   DEF file generated by floorplan phase can be utilized by magic tool to get the floorplan view which requires 3 configuration files:
    * Magic technology file (sky130A.tech)
    * DEF file from floorplan phase
    * Merged LEF file from preparation phase
![Screenshot from 2023-08-13 02-31-32](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/84e719f2-5fe4-4491-970e-6c3887719c69)


![Screenshot from 2023-08-13 02-32-48](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/c2c26bfa-32dd-4df8-ae4a-cc6e5c0aecd0)


# Placement

Placement determine the locations of standard cells or logic elements within each block.Some circuit elements may have fixed locations while others are movable.

  * Global placement: Global placement assigns general locations to movable objects. Some overlaps are allowed between placed objects.

  * Detailed placement: Detailed placement refines object locations to legal cell sites and enforces non-overlapping constraints. Detailed placement determines the achievable quality of the subsequent routing stages.

To run placement phase
```
% run_placement
```
  - Running Placement in Magic   
![Screenshot from 2023-08-13 03-14-00](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/33a0722d-31e9-48f4-94cf-f6e6538ac025)

![Screenshot from 2023-08-13 03-14-22](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/981921f8-563f-44c9-8a79-e5891e521792)


# Standard Cell Design

Standard cell design flow consists of 3 stages
  * Inputs: PDKs, DRC and LVS rules, SPICE models, library & user-defined specs.
  * Design Steps:  Involves circuit design, layout design, characterization using GUNA tool. Characterization involves timing, power and noise characterizations.
  * Outputs:  CDL (Circuit Description Language), GDSII, LEF(Library Exchange Format), Spice extracted netlist, timing, noise, power libs.


Standard cell characterization refers to gathering data about the behaviour of standard cells. To build the circuit knowledge of logic function of cell alone is not sufficient.
Standard cell library has cells with different drive strength and functionalities.These cells are characterized by using tool like GUNA from [`Paripath`](#https://www.paripath.com/home).

The standard cell characterization flow involves

  * Read the model files
  * Read the extracted spice netlist
  * Recognize function or behaviour of the cell
  * Apply stimulus and characterization setup
  * Vary the output load capacitance and observe the different characterization behaviours
  * Provide necessary simulation commands
    
Apply the entire flow to GUNA tool to generate timing, noise and power models.

# Design and characterize library cell CMOS inverter
  
To get the cell files refer [`standard cell characterization`](#https://github.com/nickson-jose/vsdstdcelldesign).

- Magic layout view to cmos inverter:
![Screenshot from 2023-08-13 03-28-09](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/0699795b-b7c6-4e70-b63c-3a3409d65b6f)

- To extract the parasitics and characterize the cell design use below commands in tkcon window:
![Screenshot from 2023-08-13 03-37-58](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/c6b86088-6c21-499a-98ff-7693ed08e487)

- Extracted spice deck file from the layout:
![Screenshot from 2023-08-13 03-39-52](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/38914095-2a72-47a5-8c6d-b3c52a5baeea)

- Few modifications needs to be done in spice deck file and then run the simulation in ngspice,
    * invoke the ngspice tool with the modified extracted spice file as input.
    * To plot transient analysis output, where y - output node and a - input node
      ```
      plot y vs time a
      ```
![Screenshot from 2023-08-13 08-15-28](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/48a3ac65-bb46-413f-a270-0bb947d9f96c)


- Inverter Standard cell characterization
Four timing parameters are used to characterize the inverter standard cell:

  * Rise transition: Time taken for the output to rise from 20% of max value to 80% of max value
  * Fall transition: Time taken for the output to fall from 80% of max value to 20% of max value
  * Cell rise delay = time(50% output rise) - time(50% input fall)
  * Cell fall delay = time(50% output fall) - time(50% input rise)
    
The above timing parameters can be computed by noting down various values from the ngspice waveform.

![Screenshot from 2023-08-13 08-23-00](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/1b9678d1-6c7c-40ed-be0f-b100cb15e590)
![Screenshot from 2023-08-13 08-27-10](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/4b1ab738-294b-4c26-8f5a-ee2b1c3e5d93)


# Timing analysis and Clock tree synthesis



# Final steps in RTL to GDSII



# Conclusion




# Acknowledgements 

- [The OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenLane)
- [Nickson Jose](https://github.com/nickson-jose/vsdstdcelldesign)
- [Kunal Ghosh](https://github.com/kunalg123)
  






