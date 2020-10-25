# Physical Design Using Openlane/Sky130

This repository contains all the information needed on SoC design planning in Openlane flow using the latest Google-SkyWater 130nm process node. One can design and characterize their own standard cell and generate a full GDSII from a RTL netlist.

# Table of Contents
- [Introduction to Openlane and sky130 PDK](#introduction-to-openlane-and-sky130-pdk)
- [Overview of Physical Design Flow](#overview-of-physical-design-flow)
- [Introduction to Openlane Flow](#introduction-to-openlane-flow)
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

  * #### Synthesis - 
    * During synthesis the RTL description is converted into a structural gate level based netlist which instantiates standard cells and macros that compose the circuit and its connections.In this step,Translation + Optimization + Mapping are performed.
 
  * Floor/Power Planning - During this step, 1) Width and height of core, Die are defined
                                             2) Location of preplaced cells is defined and these cells are surrounded with Decoupling capacitors
                                             3) Multiple VDD ,VSS lines are defined
                                             4) Pin placement, logical cell placement blockage is done
                                             
  * Placement - During this step, netlist is binded with physical cells and these are placed on floorplan rows aligned with the sites.
  
  * Clock Tree Synthesis - During this step, a clock distribution network is created to deliver clock to all the sequential elements with minimum skew.
  
  * Routing - During this step, interconnects are implemented using available metal layers , routing grid is formed using metal tracks.Routing is done in two stages                                       1) Global routing - Generates Routing guides 
                                             2) Detailed routing - This routes within the proprocessed route guides provided from Global routing step
    So routing basically finds out the best possible connection between two end points ,one being the source and the other being target. 
  
  * Sign Off - During this step, Physical verifications like Design Rule Check(DRC), Layout Versus Schematic(LVS) and Timing verification like Static Timing Analysis(STA) are done.
 
# Introduction to Openlane Flow
  
The below flow chart provides a better picture of Openlane flow as a whole.
  
<div align="center">
 <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/openlane_flow.png' alt='Openlane Flow'/>
</div>
