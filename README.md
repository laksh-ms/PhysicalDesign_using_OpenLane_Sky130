# PhysicalDesign_using_OpenLane_Sky130

Advanced Physical Design project - A complete P&R flow was carried out on Picorv32 SoC using OpenLane SkyWater-130nm Pdk on riscV, as part of the course "Advanced Physical Design using OpenLANE/Sky130" conducted by VLSI System Design Corporation. PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set which is used as an example in this course.

# Author

_Lakshmi M Satyananda_, M.Tech VLSI Design, Amrita Vishwa Vidyapeetham


# Contents

1. [Introduction to OpenLane](#introduction-to-openlane)
2. [OpenLane-invoking and Design Preparation](#invoking-openlane-and-design-preparation)
3. [Synthesis](#synthesis)
4. [Floorplaning](#floorplaning)
5. [Placement](#placement)
6. [Std. Cell Design](#standard-cell-design)
7. [Timing analysis and Clock tree synthesis](#timing-analysis-and-clock-tree-synthesis)
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

- Covert grid to tracks of the std. cell:
  
Tracks are used in routing stages. Routes are metal traces which can go over the tracks. The information of horizontal and vertical tracks present in each layer is given in tracks.info file.

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. Convergence of grid and tracks can be achieved using the following command:
```
grid 0.46um 0.34um 0.23um 0.17um
```

![tracks_info](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/6fc70e6b-b4b0-4ca6-aa22-c77cdfcf1151)


- Convert Magic Layout to LEF:

  LEF exposes only the necessary things need for the PnR tool and protecting the logic or intellectual property.
  
When extracting LEF file, these ports are what are defined as pins of the macro. These are done in magic tool by adding text with enabling port.

  A and Y is attached to locali layer and Vdd and Gnd attached to metal1 layer. To set port class and port attribute refer standard cell characterization. Saved it as sky130_inv_lak.mag
  
![Screenshot from 2023-08-13 20-57-31](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/bba9513c-11ea-4c50-b904-ded7514c4923)

![Screenshot from 2023-08-13 10-15-04](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/f2c09006-08e5-4f0b-b53d-4ab0614b04a2)

![Screenshot from 2023-08-13 10-21-00](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/1998329b-27d7-42cd-83d7-a2781c71b4d5)

To extact LEF file as sky130_inv_lak.lef
```
lef write
```
![Screenshot from 2023-08-13 21-00-08](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/24168bf7-f9f0-4ed6-83e2-6ac3f7f66431)


- Custom inverter incorporated in picorv32a

  o include the custom inverter cell into the openLANE flow

Copy the extracted LEF file from layout into designs\picorv32a\src directory along with sky130_fd_sc_hd_*.lib from the reference repository. Custom cell inverter characterization information is included in above mentioned libs.

![Screenshot from 2023-08-13 10-50-16](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/de75d437-7eb7-4f04-9cec-195631e3ae53)

![Screenshot from 2023-08-13 21-22-38](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/a3e71232-b9dc-462c-b7d9-f93c5a5ea1cb)

- Modify design/picorv32a/config.tcl

![Screenshot from 2023-08-13 12-52-41](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/9803fa06-1f85-4566-a344-10e000f752dd)


- Now perform openLANE design flow
  
```
docker
./flow.tcl - interactive
package require openLANE 0.9
 prep -design picorv32a -tag 12-08_20-54 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```

![Screenshot from 2023-08-13 21-43-15](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/edd3e695-4d24-40d5-be6c-5f1d91d63fae)
 

```
% run_floorplan

% run_placement
```
If floorplan throws error then follow below steps post run_synthesis:
```
init_floorplan
place_io
global_placement_or
tap_decap_or
run_placement
```
Check if Merged.lef has the custom inverter macro int it.:

![Screenshot from 2023-08-14 01-00-47](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/7913c6e2-9974-4d11-928c-c7409ee047bb)

![Screenshot from 2023-08-13 22-04-07](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/cba163ca-0ac7-429a-9d9b-25f124f4fd11)

![Screenshot from 2023-08-13 22-02-05](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/9be1077b-91ab-4a41-855f-e82e52a5ea1f)


- Timing analysis and CTS

WNS (worst negative slack)and TNS (total negative slack) are referred as the worst path delay and total path delay in regards to setup timing constraint. Fixing slack violations and analyzing is done using OpenSTA tool. These analysis are performed out of the openLANE flow.
For this, a new file `pre_sta.conf` and `my_base.sdc` files are created. 

![Screenshot from 2023-08-14 05-17-40](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/3fa51f22-e32b-4fe5-8f39-da766fd3ff4a)

![Screenshot from 2023-08-14 05-20-27](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/2299ddd8-893d-4190-af60-cbb77967b253)


Invoke OpenSTA outside the openLANE flow under openlane directory as follows:
```
sta pre_sta.conf
```
![Screenshot from 2023-08-14 05-33-26](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/772825cd-ae9e-469f-8f99-083f3eb4eec6)


- Fixing slack violations
  
  For the design to be complete, the worst negative slack needs to be above or equal to 0. If the slack is not within the range:

      * Review synthesis strategy in OpenLANE
      * Enable cell buffering, and synthesis sizing values
      * Review maximum fanout of cells
      * Perform manual cell replacement using the OpenSTA tool (This step then alters the netlist by cell replacement techniques. Therefore, the verilog file needs to be modified using the "write_verilog" command in openSTA. Then, perform floorplan and placement  again, WITHOUT running the synthesis again. )
  Note: All openLANE configuration parameters are mentioned in $OPENLANE_ROOT/configuration/README.md

```
set ::env(SYNTH_SIZING) 1
1
% set ::env(SYNTH_MAX_FANOUT) 4
4
% echo $::env(SYNTH_MAX_FANOUT)
```
  
  ![Screenshot from 2023-08-15 07-08-49](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/021f876b-21ef-4bef-ab9f-6cb05ee0f9bb)

```
replace_cell _14481_ sky130_fd_sc_hd__or4_4 
replace_cell _14506_ sky130_fd_sc_hd__or4_4
replace_cell _14510_ sky130_fd_sc_hd__or3_4
```

![write_verilog_in_opensta](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/de9754e0-cd66-42f0-a16e-c745210dd857)


- Next perform CTS:
  The main concern in generation of clock tree is the clock skew, difference in arrival times of the clock for sequential elements across the design.To ensure timing constraints CTS will add buffers throughout the clock tree which will modify our netlist. This will generate new def file which is used in routing.
  
```
run_cts
```
![running_cts](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/617a26a6-53b0-4cde-9659-0dab02566f24)

![cts_complete](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/b24a61e6-5ca7-4f4b-8508-b41e1a8fb032)

- Further analysis of CTS in done in openROAD which is integrated in openLANE flow using openSTA tool. As the CTS run adds clock buffers in therefore buffer delays come into picture and our analysis from here on deals with real clocks. Setup and hold time slacks may now be analysed in the post-CTS STA anlysis in OpenROAD within the openLANE flow:
  Note: All variables of openroad can be accessed in openlane flow.
```
% echo $::env(LIB_CTS)
/openLANE_flow/designs/picorv32a/runs/12-08_20-54/tmp/cts.lib
% echo $::env(CURRENT_DEF)
/openLANE_flow/designs/picorv32a/runs/12-08_20-54/results/cts/picorv32a.cts.def
% echo $::env(LIB_TYPICAL)
/openLANE_flow/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib
% echo $::env(CTS_MAX_CAP)
1.53169
% 
% openroad
```

Following steps was performed in openRoad for post_cts timming analysis:

```
read_lef /openLANE_flow/designs/picorv32a/runs/12-08_20-54/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/12-08_20-54/results/cts/picorv32a.cts.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/12-08_20-54/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```
Note: TritonCTS is designed for only Typical corner (i.e. it cannot create optimized CTS for multiple corners)

![slack_met_after_cts_openroad_typical_corner](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/1544a4f4-ba71-4f73-98d9-1b821e47ddf1)


Try removing sky130_fd_sc_hd__clkbuf_1 from clock tree and do post cts timing analysis
```
% set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]
sky130_fd_sc_hd__clkbuf_2 sky130_fd_sc_hd__clkbuf_4 sky130_fd_sc_hd__clkbuf_8
% echo $::env(CTS_CLK_BUFFER_LIST)
sky130_fd_sc_hd__clkbuf_2 sky130_fd_sc_hd__clkbuf_4 sky130_fd_sc_hd__clkbuf_8
% echo $::env(CURRENT_DEF)
/openLANE_flow/designs/picorv32a/runs/03-07_16-12/results/cts/picorv32a.cts.def
% set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/03-07_16-12/results/placement/picorv32a.placement.def
```
Now run openROAD and do a timing analysis as mentioned above.
Note: Including large size clock buffers in clock path improves slack but area increases.


# Final steps in RTL to GDSII

- PDN (Power distribution network)
  Typically in ASIC P&R, the Power planning is done with floorplanning in which power grid network is created to distribute power to each part of the design equally. In openLANE flow it is done before routing. It is done with the following command and This generates new def file in /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/12-08_20-54/tmp/floorplan/17-pdn.def:
  
  ```
  gen_pdn
  ```
![gen_pdn](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/4eb739d0-ec9f-49fc-ac74-44150f9f1638)

![pdn_success](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/b754f257-56cc-4543-9b5c-2ee3ea8b1d3a)

  
- Routing

Routing is the stage where the interconnnections. This includes interconnections of standard cells, the macro pins, the pins of the block boundary or pads of the chip boundary. Logical connectivity is defined by netlist and design rules are defined in technology file are available to routing tool. In routing stage, metal and vias are used to create the electrical connections.
To perform routing:
```
  run_routing
```
![run_routing](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/fcb6dcb8-85f4-4119-ad7a-c44bf4e5ca3f)

![routing_2](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/d2451238-3c62-4d42-a7a8-7c63a211ad80)


After routing magic tool can be used to get routing view from runs directory:
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read 12-08_20-54/tmp/merged.lef def read 12-08_20-54/results/routing/picorv32a.def &
```
![final_layout](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/78b00252-ec5d-499e-b445-bc23c40b580b)

![zoomed_picorv32a](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/e0b18bd9-9b6b-46ea-8cf1-74dc925a06b7)


Custom inverter in the final Routed view:

![inv_lak_final](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/dcd677a1-7373-44ea-84ce-28daa8d65ac4)

  
- SPEF extraction

  In old versions of Openlane, after routing has been completed interconnect parasitics can be extracted to perform sign-off post-route STA analysis. The parasitics are extracted into a SPEF file using SPEF-Extractor.

The latest version of openlane supports spef generation. The spef file will be generated after run_routing command at location:  /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/12-08_20-54/results/routing/picorv32a.spef

- GDSII

  GDSII files are usually the final output product of the IC design cycle and are given to silicon foundries for IC fabrication.It is a binary file format representing planar geometric shapes, text labels, and other information about the layout in hierarchical form.

To generate GDSII file:
```
run_magic
```
![gds_write_complete](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/2e1077a2-76b3-4753-9016-e91ed4974f2d)

No DRC errors are found.

![final_gds](https://github.com/laksh-ms/PhysicalDesign_using_OpenLane_Sky130/assets/109785515/9654083e-8fbc-4245-b228-3f9032a19f99)


# Conclusion

Clean `gds` file with No DRC errors was generated at location 
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/12-08_20-54/results/magic/picorv32a.gds


# Acknowledgements 

- [Kunal Ghosh](https://github.com/kunalg123)
- [The OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenLane)
- [Nickson Jose](https://github.com/nickson-jose/vsdstdcelldesign)
  






