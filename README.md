# Physical Design Using Openlane/Sky130

This repository contains all the information needed on SoC design planning in Openlane flow using the latest Google-SkyWater 130nm process node. One can design and characterize their own standard cell and generate a full GDSII from a RTL netlist.

# Table of Contents
- [Introduction to Openlane and sky130 PDK](#introduction-to-openlane-and-sky130-pdk)
- [Overview of Physical Design Flow](#overview-of-physical-design-flow)
- [Introduction to Openlane Flow](#introduction-to-openlane-flow)
  - [Opensource Tools at each step](#opensource-tools-at-each-step)
- [Build and Invoke Openlane](#build-and-invoke-openlane)
- [openlane Directory structure](#openlane-directory-structure)
- [Build your Design in openlane ](#build-your-design-in-openlane)
  - [Preparation](#preparation)
  - [Synthesis](#synthesis)
- [Acknowledgements](#acknowledgements)

# Introduction to Openlane and sky130 PDK

  * OpenLANE is an automated RTL to GDSII flow which includes in it several open source tools such as OpenROAD, Yosys, Magic, Netgen, Fault, OpenPhySyn, SPEF-Extractor and custom methodology scripts for design exploration and optimization.Full ASIC implementation steps from RTL to GDSII can be performed using this flow.

More details on openLANE can be obtained [here](https://github.com/efabless/openlane).
  * The SkyWater Open Source PDK is a collaboration between Google and SkyWater Technology Foundry to provide a fully open source Process Design Kit and related resources, which can be used to create manufacturable designs at SkyWater’s facility.

More details on sky130 PDK can be obtained [here](https://github.com/google/skywater-pdk).
 
# Overview of Physical Design Flow

The Backend flow is transforms the RTL circuit description into a physical design,composed by gates and its interconnections.
The below flow chart gives a better picture of physical design flow.

<div align="center">
 <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/asic_flow.png' alt='ASIC Flow'/>
</div>

  * #### Synthesis  
    * During synthesis the RTL description is converted into a structural gate level based netlist which instantiates standard cells and macros that compose the circuit and its connections.In this step,Translation + Optimization + Mapping are performed.
 
  * #### Floor/Power Planning 
    * During this step, 1) Width and height of core, Die are defined
                                             2) Location of preplaced cells is defined and these cells are surrounded with Decoupling capacitors
                                             3) Multiple VDD ,VSS lines are defined
                                             4) Pin placement, logical cell placement blockage is done
                                             
  * #### Placement 
    * During this step, netlist is binded with physical cells and these are placed on floorplan rows aligned with the sites.Standard cell placement happens here.
  
  * #### Clock Tree Synthesis 
    * During this step, a clock distribution network is created to deliver clock to all the sequential elements with minimum skew.
  
  * #### Routing 
    * During this step, interconnects are implemented using available metal layers , routing grid is formed using metal tracks.Routing is done in two stages                                       1) Global routing - Generates Routing guides 
                                             2) Detailed routing - This routes within the proprocessed route guides provided from Global routing step.
    So routing basically finds out the best possible connection between two end points ,one being the source and the other being target. 
  
  * #### Sign Off 
    * During this step, Physical verifications like Design Rule Check(DRC), Layout Versus Schematic(LVS) and Timing verification like Static Timing Analysis(STA) are done.
 
# Introduction to Openlane Flow
  
The below flow chart provides a better picture of Openlane flow as a whole.[(Image source)](https://github.com/efabless/openlane)
  
<div align="center">
 <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/openlane_flow.png' alt='Openlane Flow'/>
</div>

## Opensource Tools at each step

Below are the stages and respective tools called by openlane during the flow :

* #### Synthesis
  * Generates gate-level netlist `yosys`
  * Performs Cell mapping `abc`
  * Performs pre-layout STA `OpenSTA`
* #### Floorplanning
  * Defines the core area for the macro as well as the rows and the tracks `init_fp`
  * Places the macro input and output ports `ioplacer`
  * Generates the power distribution network `pdn`
* #### Placement
  * Performs Global placement `RePlace`
  * Performs detailed placement to legalize the globally placed components `OpenDP`
* #### Clock Tree Synthesis
  * Synthesizes the clock distribution network `TritonCTS`
* #### Routing
  * Performs Global routing to generate guide file `FastRoute`
  * Performs Detailed routing `TritonRoute`
  * Performs SPEF extraction `SPEF-Extractor`
* #### GDSII Generation
  * Streams out the final GDSII layout file from the routed def `Magic` 

# Build and Invoke Openlane 

Detailed description on how to build and invoke openlane is given [here](https://github.com/nickson-jose/openlane_build_script).

# openlane Directory structure
 * Process Design kits generally contains information of timing libraries for different process corners, cell lef,tech lef,basic building blocks description etc., The below picture describes the PDK directory flow.

<div align="center">
 <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/pdk_flow.png' alt='PDK directory structure'/>
</div>

 * libs.ref file contents are specific to technology whereas libs.tech file contents are specific to tool.While implementing this physical design flow, `Sky130 nm` technology node ,`sky130_fd_sc_hd` standard cell library are used.

 * Any output run data is placed by default under ./designs/design_name/runs folder. Each flow cycle will output timestamp-marked folder which contains the following files.The below picture lists all the files.

<div align="center">
 <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/output_file.png' alt='Output files structure'/>
</div>

# Build your Design in openlane

 * Invoke openlane and run the script file in an intercative (step by step)mode.
   `./flow.tcl -interactive`
 * Input the packages needed to run this flow
   `package require openlane 0.9
 * Below image shows the terminal after openlane being invoked.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/invoke_openlane.png' alt='Invoking openlane'/>
 </div>
 
## Preparation

 * openlane has close to 30-40 designs, of which one can select any design and run it. 'picorv32a' design is chosen here for example.
 * Any design comprises of files such as 'src' which contains verilog netlist(.v file) , constraints(.sdc) file and 'config.tcl' which is specific to design. 
 * Design specific config.tcl file overrides the parameter values which are from default openlane flow configuration file.
 * Now design setup stage is to be done so that a seperate filesystem is created specific to the flow
   `prep -design picorv32a`
 * After preparation step is done:
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/preparation.png' alt='preparation step'/>
 </div>
