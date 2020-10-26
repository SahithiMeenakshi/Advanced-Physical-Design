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
  - [Floorplan](#floorplan)
  - [Placement](#placement)
- [Standard cell design & characterization](#[standard-cell-design-&-characterization)
  - [Extracting lef file from .mag file](#extracting-lef-file-from-.mag-file)
  - [Plugging Custom LEF to openlane flow](#plugging-custom-lef-to-openlane-flow)
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

 * Invoke openlane and run the script file in an interactive (step by step)mode.
 
   `./flow.tcl -interactive`

* Input the packages needed to run this flow.
 
   `package require openlane 0.9`
 
 * Below image shows the terminal after openlane being invoked.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/invoke_openlane.png' alt='Invoking openlane'/>
 </div>
 
## Preparation

 * openlane has close to 30-40 designs, of which one can select any design and run it. 'picorv32a' design is chosen here for example.
 * Any design comprises of files such as 'src' which contains verilog netlist(.v file) , constraints(.sdc) file and 'config.tcl' which is specific to design. 
 * Design specific config.tcl file overrides the parameter values which are from default openlane flow configuration file.
 * Now design setup stage is to be done so that a seperate filesystem is created specific to the flow.
  
   `prep -design picorv32a`
 
 * After preparation step is done:
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/preparation.png' alt='preparation step'/>
 </div>
 
 * Now time-stamp or tag folder is created under 'runs' directory which contains results,reports,logs of each stage.
  
## Synthesis

 * Now run the synthesis step using the command:
 
   `run_synthesis`
   
 * Below image shows the statistics before mapping.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/before_mapping.png' alt='Before Mapping stage'/>
 </div>
 
 * Below images show the statistics after mapping and synthesis is done.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/after_mapping.png' alt='After Mapping stage'/>
 </div>
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/area_after_synthesis.png' alt='After synthesis stage'/>
 </div>
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/synthesis.png' alt='After synthesis stage'/>
 </div>
 
 * Below image shows the reports that are generated after synthesis step.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/reports_after_synthesis.png' alt='Reports after synthesis stage'/>
 </div>

## Floorplan

 * Before proceeding , lets see the changes in config files when Floorplan switches are modified in the flow. To know more about switches at each stage, go to openlane_working_dir-->openLANE_flow-->configuration-->README.md file.
 * Below image shows the floorplan switches from openlane flow configuration file(default) which has less priority.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/floorplan_defaults.png' alt='Default floorplan switches'/>
 </div>
 
 * Now set the horizontal,vertical metal layer switches to some values (3,4 respectively in example taken here) using the following commands in the design specific config file.
 
   `set ::env(FP_IO_HMETAL) 3`
   `set ::env(FP_IO_VMETAL) 4`
 
 * Now run the floorplan step using the following command.
 
   `run_floorplan`
   
 * Now check whether the changes are incorporated in the flow by observing the design specific floorplan log file.Below is the snapshot indicating changes in the vertical,horizontal metal layer information.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/override_defaults.png' alt='Overriding Default floorplan switches'/>
 </div>
 
 * So above indicates that default openlane configuration parameters have been overridden by parameters that are included in design specific config file.
 * Below snapshot shows the die area after the floorplan step in which the coordinates can be read as(Lower left X value, Lower left Y value)(Upper right X value, Upper right Y value).
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/die_area.png' alt='Die area after floorplan'/>
 </div>
 
 * To see the Layout after Floorplan stage ,launch Magic from designs/picorv32a/runs/timestamp of recent run/results/floorplan location. Magic tech file, lef file,def file are read to display the layout using the following command.
 
   `magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &`
  
 * If Input ouput pin mode is set as '1' ,pins are placed at random but at equidistant points.
 * If I/O mode is set to 2 using `set ::env(FP_IO_MODE) 2` and then floorplan step is run again, now in the layout after Floorplan stage pins are seen not being equidistant rather they are stacked upon one another. Below snapshot shows this scenario.
 
 <div align="center">
  <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/io_mode2.png' alt='I/O mode as 2'/>
 </div>
 
## Placement

 * Run the placement step using the following command.
  
    `run_placement`
    
 * For the design to converge, as number of iterations increases overflow(OVFL) should decrease.Below is the snapshot after the Placement stage.
  
  <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/placement.png' alt='placement stage'/>
  </div>
  
 * To see the Layout after Placement stage ,launch Magic from designs/picorv32a/runs/timestamp of recent run/results/placement location. Magic tech file, lef file,def file are read to display the layout using the following command.
  
   `magic -T ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &`
 
 * Below image shows the layout after placement stage.
 
  <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/placement_layout.png' alt='Layout after Placement'/>
  </div>

# Standard cell design & characterization

 * Clone the following [link](https://github.com/nickson-jose/vsdstdcelldesign) into openLANE_flow directory which contains custom made .mag inverter file and also pmos,nmos sky130 spice models.
 * Launch Magic from vsdstdcelldesign location using the command `magic -T sky130.tech sky130_inv.mag &`
 * To extract parasitic capacitances, first do `extract all` and then `ext2spice cthresh 0 rthresh 0` and finally `ext2spice`.Below is the snapshot of all these steps.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/inverter_layout.png' alt='Inverter Layout'/>
 </div>
 
 * In the extracted spice file, change the scale according to layout grid measurements and include libraries,power supplies,input pulse and specify analysis to run.Below snapshot shows the above steps in detail.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/spice_file.png' alt='changes in spice file'/>
 </div>
 
 * Invoke ngspice using command `ngspice sky130_inv.spice` . Below snapshot shows plots in ngspice.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/ngspice.png' alt='ngspice plots'/>
 </div>
 
 ## Extracting lef file from .mag file
 
 * Before extracting lef file, certain definitions need to be set to the pins of the cell.As the ports are the declared as pins of the macro in LEF files, defining port and setting correct class and use attributes to each port is the first step.
 * In Magic Layout window, first source the .mag file of inverter and then goto `Edit >> Text` which opens up a dialogue box. Below image is a snapshot of this.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/define_port.png' alt='Defining ports of inverter'/>
 </div>
 
 * To create port definition and setting 'port class' and 'port use' attributes follow the steps from [here](https://github.com/nickson-jose/vsdstdcelldesign).
 * Convert grids to tracks from the information availbale in tracks.info file at the loaction specified in the terminal. Below are the images of tracks info file and command to convert grids to tracks.Finally save this file using command `save sky130_vsdinv.mag`.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/tracks_file.png' alt='Tracks file'/>
 </div>
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/grid_conversion.png' alt='Grids to Tracks'/>
 </div>
 
 * Now invoke magic tool using command `magic -T sky130A.tech sky130_vsdinv.mag &`.
 * Now to extract lef file , use command `lef write` in magic layout command window. Below is the image of LEF file contents.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/lef_file.png' alt='LEF file'/>
 </div>

 ## Plugging Custom LEF to openlane flow
 
 * Copy LEF file, library files to `src` folder location as shown in figures below.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/copy_lef.png' alt='LEF file copying'/>
 </div>
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/copy_lib.png' alt='LIB files copying'/>
 </div>
 
 * In the design's config.tcl file add the libraries and also add the below line to point to the lef location which is required during spice extraction.
   `set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]`.
 * Below is the snapshot of config.tcl file after modifications.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/config_file.png' alt='design config file'/>
 </div>
 
 * Now invoke the openlane in interactive mode, import the packages required and go through design preparation stage using commands one by one.
  
  `./flow.tcl -interactive`
  
  `package require openlane 0.9`
  
  `prep -design picorv32a`
 * Add the below commands to include LEF into the openlane flow.
   `set lefs [glob $::env(DESIGN_DIR)/src/*.lef]`
  
   `add_lefs -src $lefs`
   
 * Below is the snapshot of these steps.
 
 <div align="center">
   <img src='https://github.com/SahithiMeenakshi/Advanced-Physiscal-Design/blob/main/Images/lef_preparation.png' alt='preparation stage after LEF in flow'/>
 </div>
 
 
# Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Nickson Jose](https://github.com/nickson-jose)
