# PhysicalDesign_using_OpenLane_Sky130
Advanced Physical Design project - A complete P&R flow was carried out on Picorv32 SoC using OpenLane SkyWater-130nm Pdk on riscV, as part of the course "Advanced Physical Design using OpenLANE/Sky130" conducted by VLSI System Design Corporation. PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set which is used as an example in this course.

# Contents
1. Introduction to OpenLane
2. OpenLane-invoking and Design Preparation
3. Synthesis
4. Floorplaning
5. Placement
6. Std. Cell


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

OpenSTA - Performs static timing analysis on the resulting netlist to generate timing reports

  - Floorplaning

init_fp - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)

ioplacer - Places the macro input and output ports

pdngen - Generates the power distribution network

tapcell - Inserts welltap and decap cells in the floorplan

  - Placement

RePLace - Performs global placement

Resizer - Performs optional optimizations on the design

OpenDP - Performs detailed placement to legalize the globally placed components

  - CTS

TritonCTS - Synthesizes the clock distribution network (the clock tree)

  - Routing
      * FastRoute - Performs global routing to generate a guide file for the detailed router

TritonRoute - Performs detailed routing

OpenRCX - Performs SPEF extraction

  - Tapeout
      * Magic - Streams out the final GDSII layout file from the routed def

KLayout - Streams out the final GDSII layout file from the routed def as a back-up

  - Signoff
      * Magic - Performs DRC Checks & Antenna Checks
      * KLayout - Performs DRC Checks
      * Netgen - Performs LVS Checks
      * CVC - Performs Circuit Validity Checks

# Invoking OpenLANE and Design Preparation

