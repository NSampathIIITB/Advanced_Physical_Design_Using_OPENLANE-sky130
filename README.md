# Advanced_Physical_Design_Using_OpenLANE-sky130

[Day-1 :  Introduction to open-source EDA, OpenLANE and Sky130 PDK](#day-1)

[Day-2 : Good floorplan vs bad floorplan and Introduction to library cells](#day-2)

[Day-3 : Design library cell using Magic Layout and ngspice characterization](#day-3)

[Day-4 : Pre-layout timing analysis and importance of good clock tree](#day-4)

[Day-5 : Final steps for RTL2GDS using tritonRoute and openSTA](#day-5)

[Acknowledgement](#acknowledgement)

[Reference](#reference)

## Day-1

### How to talk to computers ?

The RISC-V Instruction Set Architecture (ISA) is a language used to talk to computers whose hardware is based on RISC-V core. If a user wishes to run a certain application software on a computer, its corresponding C/C++/Java program must be converted into assembly language instructions by the compliler. The ouput of the compiler is hardware dependent. These instructions go as inputs to the assembler whose output is binary language (0's and 1's) that the hardware logic in the chip layout can make sense of. According to the bits received, the digital logic consisting of gates performs the function required by the user of the application software.

![Screenshot from 2023-08-17 20-24-09](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/9e260e3c-f400-4dc2-b6f5-cc224ed4c202)

![Screenshot from 2023-08-17 20-27-14](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/7f4ef63e-2f83-4ea7-baff-58c2157edcba)

### SoC Design & OpenLANE

#### Components of opensource digital ASIC design

The design of digital Application Specific Integrated Circuit (ASIC) requires three enablers or elements - Resistor Transistor Logic Intellectual Property (RTL IPs), Electronic Design Automation (EDA) Tools and Process Design Kit (PDK) data.

#### RTL IP'S
RTL IP, or Register-Transfer Level Intellectual Property, refers to pre-designed and pre-verified hardware components or blocks that are used in digital circuit design at the RTL level. These IP blocks are typically represented in RTL hardware description languages like VHDL or Verilog and can be integrated into larger digital designs.
<br>
RTL IPs are created to be reusable building blocks. They can include various types of components, such as processors, memory controllers, interfaces (e.g., USB, PCIe), DSP blocks, and more. Designers can integrate these IPs into their larger digital designs to save time and effort.

#### EDA Tools
Electronic Design Automation (EDA) tools are software applications that are used by electronics engineers and designers to create, analyze, simulate, and verify electronic systems and integrated circuits. These tools are crucial for designing and testing complex electronic hardware and are widely used in various industries, including semiconductor design, printed circuit board (PCB) design, and FPGA/ASIC development.

#### PDK'S
PDK, which stands for Process Design Kit, is a set of files, documentation, and tools provided by semiconductor foundries and integrated circuit (IC) manufacturers to enable designers to create custom integrated circuits using their manufacturing processes. PDKs are essential for designing and manufacturing integrated circuits, as they provide the necessary information and resources to ensure that the designed ICs can be fabricated accurately and reliably on the foundry's equipment.
<br>
Each foundry provides its own PDKs, and PDKs are specific to the foundry's manufacturing processes. Different foundries may use different PDK formats and tools.

Designers and semiconductor companies rely on PDKs to bridge the gap between concept and physical implementation, ensuring that IC designs can be fabricated successfully and meet performance specifications. Access to a PDK is typically a prerequisite for working with a semiconductor foundry to manufacture custom integrated circuits.


![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/f883c69f-33b4-456c-a4a3-1879a7e9402f)

- Opensource RTL Designs: github, librecores, opencores
- Opensource EDA tools: QFlow, OpenROAD, OpenLANE
- Opensource PDK data: Google Skywater130 PDK

The ASIC flow objective is to convert RTL design to GDSII format used for final layout. The flow is essentially a software also known as automated PnR (Place & route) or Physical implementation.

#### Simplified RTL to GDSII flow

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/de25a868-1c85-458a-9210-c3952c278a67)

The RTL to GDSII flow basically involves :
1. **RTL Design** -  The process begins with the RTL design phase, where the digital circuit is described using a hardware description language (HDL) like VHDL or Verilog. The RTL description captures the functional behavior of the circuit, specifying its logic and data paths.

2. **RTL Synthesis** - RTL synthesis converts the high-level RTL description into a gate-level netlist. This stage involves mapping the RTL code to a library of standard cells (pre-designed logic elements) and optimizing the resulting gate-level representation for area, power, and timing. The output of RTL synthesis is typically in a format called the gate-level netlist.

3. **Floor and Power Planning** - is a crucial step in the digital design flow that involves partitioning the chip's area and determining the placement of major components and functional blocks. It establishes an initial high-level layout and defines the overall chip dimensions, locations of critical modules, power grid distribution, and I/O placement.The primary goals of floor planning are: Area Partitioning, Power Distribution, Signal Flow and Interconnect Planning, Placement of Key Components, Design Constraints and Optimization.

4. **Placement** - Placement involves assigning the physical coordinates to each gate-level cell on the chip's layout. The placement process aims to minimize wirelength, optimize signal delay, and satisfy design rules and constraints. Modern placement algorithms use techniques like global placement and detailed placement to achieve an optimal placement solution.
   
    Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap.
    Detailed Placement - After Global placement is done minimal alterations are done to correct the issues.
   
![Screenshot from 2023-09-09 09-56-20](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/911f6f6f-6ae8-4fcb-a843-b498cf328772)


5. **Clock Tree Synthesis** - Clock tree synthesis (CTS) is a crucial step in the digital design flow that involves constructing an optimized clock distribution network within an integrated circuit (IC). The primary goal of CTS is to ensure balanced and efficient clock signal distribution to all sequential elements (flip-flops, registers) within the design, minimizing clock skew and achieving timing closure.These are some of the types of clock trees
   
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6dfc1944-95f8-4fac-84be-3a7a30a086de)

6. **Fake Antenna and diode swapping**-Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges.

   - OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.</br>
   - OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode
OpenLane allows the user to chose either of the above approaches
         

7. **Routing** - Routing connects the gates and interconnects on the chip based on the placement information. It involves determining the optimal paths for the wires and vias that carry signals between different components. The routing process needs to adhere to design rules, avoid congestion, and optimize for factors like signal integrity, power, and manufacturability.This step is used to implement the interconnect using the different metal layers specified in the PDK.

   There are two steps:

    Global Routing - This is done inside the OpenROAD flow (FastRoute)<br>
    
    Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the "Logic Equivalence Check" is performed by Yosys, since OpenROAD does some optimisations the circuit.

8. **Sign-off** - Sign-off analysis refers to the final stage of the electronic design process, where comprehensive verification and analysis are performed to ensure that the design meets all the necessary requirements and specifications. It involves a series of checks and simulations to confirm that the design is ready for fabrication and meets the desired functionality, performance, power, and reliability targets. 

9. **GDSII File Generation** - Once the layout is verified and passes all checks, the final step is to generate the GDSII file format, which represents the complete physical layout of the chip. The GDSII file contains the geometric information necessary for fabrication, including the shapes, layers, masks, and other relevant details.

### Introduction to OpenLANE

OpenLANE is an opensource tool or flow used for opensource tape-outs. The OpenLANE flow comprises a variety of tools such as Yosys, ABC, OpenSTA, Fault, OpenROAD app, Netgen and Magic which are used to harden chips and macros, i.e. generate final GDSII from the design RTL. The primary goal of OpenLANE is to produce clean GDSII with no human intervention. OpenLANE has been tuned to function for the Google-Skywater130 Opensource Process Design Kit.


![Screenshot from 2023-09-09 10-26-39](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/ffb45e36-01d7-40c0-9651-2fb99b4f1b2f)


OpenLane flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLane can also be run interactively as shown [here][25].

1. **Synthesis**
    1. `yosys` - Performs RTL synthesis
    2. `abc` - Performs technology mapping
    3. `OpenSTA` - Performs static timing analysis on the resulting netlist to generate timing reports
2. **Floorplan and PDN**
    1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
    2. `ioplacer` - Places the macro input and output ports
    3. `pdn` - Generates the power distribution network
    4. `tapcell` - Inserts welltap and decap cells in the floorplan
3. **Placement**
    1. `RePLace` - Performs global placement
    2. `Resizer` - Performs optional optimizations on the design
    3. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. **CTS**
    1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. **Routing**
    1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
    2. `CU-GR` - Another option for performing global routing.
    3. `TritonRoute` - Performs detailed routing
    4. `SPEF-Extractor` - Performs SPEF extraction
6. **GDSII Generation**
    1. `Magic` - Streams out the final GDSII layout file from the routed def
    2. `Klayout` - Streams out the final GDSII layout file from the routed def as a back-up
7. **Checks**
    1. `Magic` - Performs DRC Checks & Antenna Checks
    2. `Klayout` - Performs DRC Checks
    3. `Netgen` - Performs LVS Checks
    4. `CVC` - Performs Circuit Validity Checks
       
### OpenLane Installation

**Prior to the installation of the OpenLane install the dependencies and packages using the command shown below:** </br>
``` 
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```
**Docker Installation :** </br>
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world

sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot 


# Check for installation of docker
sudo docker run hello-world
```

**Steps to install OpenLane, PDKs and Tools** </br>
```
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane --recurse-submodules 
cd OpenLane
make
make test
cd /home/kanish/OpenLane/designs/ci
cp -r * ../
```

**Steps to run synthesis in OpenLane:**


```
cd ~/OpenLane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
![Screenshot from 2023-09-09 12-45-02](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/ddc2a1bd-082f-4805-bdaa-5065fc640abf)

**To view the synthesized nelist:**
```
cd /home/nsaisampath/OpenLane/designs/picorv32a/runs/RUN_2023.09.09_07.11.58/results/synthesis
vim picorv32.v
```
![Screenshot from 2023-09-09 12-58-37](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/eeed499d-ef4e-45cd-a6d2-5670a6733d70)

**To view the report after synthesis:**
```
cd /home/nsaisampath/OpenLane/designs/picorv32a/runs/RUN_2023.09.09_07.11.58/reports/synthesis
vim 1-synthesis.AREA_0.stat.rpt
```
![Screenshot from 2023-09-09 13-00-53](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/0016c6b7-7314-4a8c-8cec-9337c70e8a4f)

```
Flop ratio = Number of D Flip flops = 1596  = 0.1579
             ______________________   _____
             Total Number of cells    10104
```

    

## Day-2

### Floorplanning considerations

#### Utilization Factor & Aspect Ratio  

Two parameters are of importance when it comes to floorplanning namely, Utilisation Factor and Aspect Ratio. They are defined as follows:

```
Utilisation Factor =  Area occupied by netlist
                     __________________________
                        Total area of core
```

```
Aspect Ratio =  Height
               ________
                Width
```
                                  
A Utilisation Factor of 1 signifies 100% utilisation leaving no space for extra cells such as buffer. However, practically, the Utilisation Factor is 0.5-0.6. Likewise, an Aspect ratio of 1 implies that the chip is square shaped. Any value other than 1 implies rectanglular chip.

#### Pre-placed cells

Once the Utilisation Factor and Aspect Ratio has been decided, the locations of pre-placed cells need to be defined. Pre-placed cells are IPs comprising large combinational logic which once placed maintain a fixed position. Since they are placed before placement and routing, the are known as pre-placed cells.

#### Decoupling capacitors

Pre-placed cells must then be surrounded with decoupling capacitors (decaps). The resistances and capacitances associated with long wire lengths can cause the power supply voltage to drop significantly before reaching the logic circuits. This can lead to the signal value entering into the undefined region, outside the noise margin range. Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.

![Screenshot from 2023-09-10 14-17-45](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/470468dc-8a14-47ff-8ede-8b745813e5cc)

#### Power Planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macros. Therefore, a good power planning ensures that each block has its own VDD and VSS pads connected to the horizontal and vertical power and GND lines which form a power mesh.

![Screenshot from 2023-09-10 14-29-38](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/41e51e13-a642-4401-bef5-a86bfd50a82a)

#### Pin Placement

The netlist defines connectivity between logic gates. The place between the core and die is utilised for placing pins i.e core marg. The connectivity information coded in either VHDL or Verilog is used to determine the position of I/O pads of various pins. Then, logical placement blocking of pre-placed macros is performed so as to differentiate that area from that of the pin area.

![Screenshot from 2023-09-10 16-23-03](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/96ed217d-7d9b-4178-b0c4-9d32a81e599e)


#### Floorplan run on OpenLANE & view in Magic

* Importance files in increasing priority order:

1. ```floorplan.tcl``` - System default envrionment variables
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```

* Floorplan envrionment variables or switches:

1. ```FP_CORE_UTIL``` - floorplan core utilisation
2. ```FP_ASPECT_RATIO``` - floorplan aspect ratio
3. ```FP_CORE_MARGIN``` - Core to die margin area
4. ```FP_IO_MODE``` - defines pin configurations (1 = equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer
6. ```FP_CORE_HMETAL``` - horizontal metal layer

***Note: Usually, vertical metal layer and horizontal metal layer values will be 1 more than that specified in the files***
 
 **To run the picorv32a floorplan in openLANE:**
 ```
 run_floorplan
 
 ```
![Screenshot from 2023-09-10 15-07-20](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/582e2e79-dd38-4c3b-ab2d-1b9b2044db18)

Post the floorplan run, a .def (design exchange format) file will have been created within the ```results/floorplan``` directory. 
We may review floorplan files by checking the ```floorplan.tcl```. 
The system defaults will have been overriden by switches set in ```conifg.tcl``` and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl```.
 
To view the floorplan, **Magic** is invoked after moving to the ```results/floorplan``` directory:

```
magic -T /home/nsaisampath/.volare/sky130A/libs.tech/magic/sky130A.tech lef read tmp/merged.nom.lef def read results/floorplan/picorv32.def &

```
![Screenshot from 2023-09-10 16-01-57](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/4b55a88f-7b17-4bd1-a8a5-4bd80923ec0c)

![Screenshot from 2023-09-10 15-56-40](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6c090a54-c767-4c75-804c-e95e34515340)

* One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key. 
* Various components can be identified by using the ```what``` command in tkcon window after making a selection on the component.
* Zooming in also provides a view of decaps present in picorv32a chip.
* The standard cell can be found at the bottom left corner.
* By clicking s,v we can move the die to the centre.
* we can also observe tapcells,they are placed to avoid latchup conditions.
 
![Screenshot from 2023-09-10 16-13-55](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/d3aa1761-1659-4a96-9f47-43e42beb0ced)

![Screenshot from 2023-09-10 16-14-06](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/912ca50b-5ca0-45bb-b4c5-16f956871bbd)

### Placement 

#### Placement Optimization

The next step in the OpenLANE ASIC flow is placement. The synthesized netlist is to be placed on the floorplan.

![Screenshot from 2023-09-10 16-41-29](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/5280430f-a170-467f-a36f-e5e123f5d20e)

![Screenshot from 2023-09-10 19-33-18](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/bc4a774b-b3d3-422a-bc0f-e17bee615072)

#### Optimised placement

![Screenshot from 2023-09-10 19-50-46](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6f96e907-95b2-466e-8e6b-228ba8bfbeef)

Placement is perfomed in 2 stages:

1. Global Placement:
* Global placement, also known as initial placement, is the first step in the physical design of an IC. The primary objective of global placement is to place all the logical components (gates, cells, etc.) of the circuit on the chip's silicon substrate in a way that minimizes the overall chip area and optimizes certain performance metrics.
* Global placement often involves iterative optimization algorithms that try to balance conflicting objectives, such as minimizing wirelength while avoiding congestion in certain regions of the chip. 
2. Detailed Placement:
  * After global placement, the detailed placement step aims to refine the initial placement and optimize it further. The primary goals include reducing wirelength, minimizing power consumption, improving signal integrity, and meeting various design constraints.
  * Similar to global placement, detailed placement involves optimization algorithms. However, these algorithms operate at a finer granularity, adjusting the positions of individual cells or groups of cells. The goal is to improve the overall chip layout in terms of both performance and manufacturability.
  * Legalisation of cells is important from timing point of view. 

#### Placement run on OpenLANE & view in Magic

Command:

```
run_placement
```
![Screenshot from 2023-09-10 21-31-01](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6a447482-80f7-4ed5-8096-c3ee1ad72858)

![Screenshot from 2023-09-10 20-24-10](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/30a57a71-e877-4a33-8caf-73e67a53ccdb)


The objective of placement is the convergence of overflow value. If overflow value progressively reduces during the placement run it implies that the design will converge and placement will be successful. Post placement, the design can be viewed on magic within ```results/placement``` directory:

```
magic -T /home/nsaisampath/.volare/sky130A/libs.tech/magic/sky130A.tech lef read tmp/merged.nom.lef def read results/placement/picorv32.def &

```
![Screenshot from 2023-09-10 20-28-30](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/051498c9-e425-4953-94c2-96ef401133d7)

![Screenshot from 2023-09-10 20-27-38](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/17ac711f-a4a1-40be-a46c-519bd9a624dd)

***Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN***

### Need for libraries and characterization

As we know, From logic synthesis to routing and STA, each and every stage has one thing in common i.e. logic gates/ logic cells. In order for the tool understand these logic gates  and their timing, we need to characterize these cells.The characterisation of cells is made using cell design flow.

![Screenshot from 2023-09-10 21-41-14](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/1d836e2a-4264-4535-b7b9-19d4c995d756)

### Standard Cell Design Flow

Standard cell design flow involves the following:

1. Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications. 
2. Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow

A typical standard cell characterization flow includes the following steps:

1. Read in the models and tech files
2. Read extracted spice netlist
3. Recognise behaviour of the cell
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide necessary output capacitance loads
8. Provide necessary simulation commands

The opensource software called GUNA can be used for characterization. Steps 1-8 are fed into the GUNA software which generates timing, noise and power models.These .libs are classified as Timing characterization, power characterization and noise characterization.

![Screenshot from 2023-09-10 22-07-50](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/cdc9e5ac-9d5f-40f8-b20d-fdb99f3ec65d)

### Timing characterisation

In standard cell characterisation, One of the classification of libs is timing characterisation.

Timing defintion | Value
------------ | -------------
slew_low_rise_thr  | 20% value
slew_high_rise_thr |  80% value
slew_low_fall_thr | 20% value
slew_high_fall_thr | 80% value
in_rise_thr | 50% value
in_fall_thr | 50% value
out_rise_thr | 50% value
out_fall_thr | 50% value

### Propagation Delay and Transition Time

#### Propagation Delay:
The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value. Poor choice of threshold values lead to negative delay values. Even thought you have taken good threshold values, sometimes depending upon how good or bad the slew, the dealy might be still +ve or -ve.

```
Propagation delay = time(out_thr) - time(in_thr)
```
#### Transition Time:

The time it takes the signal to move between states is the transition time , where the time is measured between 10% and 90% or 20% to 80% of the signal levels.

```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)

Low transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```

***Note:***
1. A poor choice of threshold points leads to negative delay value. Therefore a correct choice of thresholds is very important.
2. Huge wire delays also leads to negative delay value even when proper thresholds points are taken.

  
## Day-3

### Design a Library Cell using Magic Layout and Ngspice Characterization

#### IO Placer revision:
 - PnR is a iterative flow and hence, we can make changes to the environment variables in the fly to observe the changes in our design. 
 - Let us say If I want to change my pin configuration along the core from equvi distance randomly placed to someother placement, we just set that IO mode variable on command prompt as shown below.
 
  For example, to change IO_mode to be not equidistant, use `% set ::env(FP_IO_MODE) 2;` on OpenLANE. The IO pins will not be equidistant on mode 2 (default of 1). Run floorplan again via `% run_floorplan` and view the def layout on magic. However, changing the configuration on the fly will not change the `runs/config.tcl`, the configuration will only be available on the current session. To echo current value of variable: `echo $::env(FP_IO_MODE)`


### Designing a Library Cell:
1. SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.
2. SPICE deck values = value for W/L (0.375u/0.25u means width is 375nm and lengthis 250nm). PMOS should be wider in width(2x or 3x) than NMOS. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)  
3. Add nodes to surround each component and name it. This will be used in SPICE to identify a component.    

**Notes:**
 - Width is the length of source and drain. Length is the distance between source and drain
 - PMOS' hole carrier is slower than NMOS' electron carrier mobility, so to match the rise and fall time PMOS must be thicker (less resistance thus higher mobility) than NMOS  
 

#### SPICE Deck Netlist Description:  

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/5003b310-088d-4f5a-809b-2685952a10b2)

**Notes:**
 - Syntax for the PMOS and NMOS descriptiom:
     - `[component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]`
 - All components are described based on nodes and its values
 - `.op` is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
 - `tsmc_025um_model.mod` is the model file containing the technological parameters for the 0.25um NMOS and PMOS
   
The steps to simulate in SPICE:
```
source [filename].cir
run
setplot 
dc1 
plot out vs in 
```  

### SPICE Analysis for Switching Threshold and Propagation Delay:

CMOS robustness depends on:  

1. Switching threshold (Vm) = Voltage at which Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current( current directly flowing from Vdd to Gnd ) is high. If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.
    - In physical design, the switching threshold Vm is like a critical voltage level for a component called a CMOS inverter. It's the point at which this inverter switches between sending out a "0" or a "1" in a computer chip. This Vm is super important because it decides how well the CMOS inverter works.
    - Now, when we want to see how this CMOS inverter behaves, we do two types of tests.
    - First, we have the static test, where we check how it acts when everything's stable. We look at things like how fast it can send a signal, how much power it uses, and how safe it is against errors.
    - Then, there's the dynamic test, where we see what happens when it's switching on and off. This helps us figure out how quickly it can change from "0" to "1" and back, how strong the signals are, and if there are any weird issues like sudden changes or stuck states.
    - Both these tests are crucial in making sure CMOS inverters work well in computer chips. They help us make sure the chip does its job correctly and efficiently.
   

2. Propagation delay = rise or fall delay

DC transfer analysis is used for finding switching threshold. SPICE DC analysis below uses DC input of 2.5V. Simulation operation is DC sweep from 0V to 2.5V by 0.05V steps:
```
Vin in 0 2.5
*** Simulation Command ***
.op
.dc Vin 0 2.5 0.05
```  
Below is the result of SPICE simulation for DC analysis, the line intersection is the switching threshold:  

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/c13b0482-8593-4494-bc54-3d320d9934ce)


Meanwhile, transient analysis is used for finding propagation delay. SPICE transient analysis uses pulse input: 
1. starts at 0V
2. ends at 2.5V
3. starts at time 0
4. rise time of 10ps
5. fall time of 10ps
6. pulse-width of 1ns
7. period of 2ns  

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6d6cfd9d-09fe-4275-a87b-ce20ebdddd0f)

The simulation operation has 10ps step and ends at 4ns:  

```
Vin in 0 0 pulse 0 2.5 0 10p 10p 1n 2n 
*** Simulation Command ***
.op
.tran 10p 4n
```  
Below is the result of SPICE simulation for transient analysis:

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/cb3dfac6-12a7-4e94-9a77-0d10c4a7ecaf)

### Lab steps to git clone vsdstdcelldesign

- First, clone the required mag files and spicemodels of inverter,pmos and nmos sky130. The command to clone files from github link is:
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```
once I run this command, it will create ``vsdstdcelldesign`` folder in openlane directory.

To invoke magic to view the sky130_inv.mag file, the sky130A.tech file must be included in the command along with its path. To ease up the complexity of this command, the tech file can be copied from the magic folder to the vsdstdcelldesign folder.

The sky130_inv.mag file can then be invoked in Magic very easily:

For layout we run magic command

``` magic -T sky130A.tech sky130_inv.mag & ```

Ampersand at the end makes the next prompt line free, otherwise magic keeps the prompt line busy. Once we run the magic command we get the layout of the inverter in the magic window

![Screenshot from 2023-09-11 11-12-28](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/af78a05a-fc9e-47f0-87a1-254a224d53d7)

## 16-Mask CMOS Fabrication Process:

 **1. Selecting a substrate** = Layer where the IC is fabricated. Most commonly used is P-type substrate 
 
 **2. Creating active region for transistor** = Separate the transistor regions using SiO2 as isolation
  - Mask 1 = Covers the photoresist layer that must not be etched away (protects the two transistor active regions)
  - Photoresist layer = Can be etched away via UV light  
  - Si3N4 layer = Protection layer to prevent SiO2 layer to grow during oxidation (oxidation furnace)  
  - SiO2 layer = Grows during oxidation (LOCOS = Local Oxidation of Silicon) and will act as isolation regions between transistors or active regions  
  
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/4da7dbcf-270d-46ec-ab9f-b43cdbcedca4)

 **3. N-Well and P-Well Fabrication** = Fabricate the substrate needed by PMOS (N-Well) and NMOS (P-Well)  
  - Phosporus (5 valence electron) is used to form N-well  
  - Boron (3 valence electron) is used to form P-Well.  
  - Mask 2 protects the N-Well (PMOS side) while P-Well (NMOS side) is being fabricated then Mask 3 while N-Well (PMOS side) is being fabricated
   
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/3680920d-a2fd-4f55-bee9-4a1316920efc) 

 **4. Formation of Gate** = Gate fabrication affects threshold voltage. Factors affecting threshold voltage includes:    
 
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/24087ef0-01c8-44bc-8dcc-e78655eea76d)

Main parameters are:
  - Doping Concentration = Controlled by ion implantation (Mask 4 for Boron implantation in NMOS P-Well and Mask 5 for Arsenic implantation in PMOS N-Well)
  - Oxide capacitance = Controlled by oxide thickness  (SiO2 layer is removed then rebuilt to the desire thickness)  
  
 Mask 6 is for gate formation using polysilicon layer.
 
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/de2b69c5-657e-4e76-b833-e106ff9438ca)

**5. Lightly Doped Drain formation** = Before forming the source and drain layer, lightly doped impurity is added: 
 - Mask 7 for N- implantation (lightly doped N-type) for NMOS 
 - Mask 8 for P- implantation (lightly doped P-type) for PMOS.  
Heavily doped impurity (N+ for NMOS and P+ for PMOS) is for the actual source and drain but the lightly doped impurity will help maintain spacing between the source and drain and prevent hot electron effect and short channel effect. 

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/3ec8f45d-bbf9-44e2-9263-7923a507a4b4)

**6. Source and Drain Formation** = Mask 9 is for N+ implantation and Mask 10 for P+ implantation  
 - Channeling is when implantations dig too deep into substrate so add screen oxide before implantation
 - The side-wall spacers maintains the N-/P- while implanting the N+/P+    
 
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/3f42d459-31f6-4bb2-a966-7a9519b52c37)

**7. Form Contacts and Interconnects** =  TiN is for local interconnections and also for bringing contacts to the top. TiS2 is for the contact to the actual Drain-Gate-Source. Mask 11 is for etching off the TiN interconnect for the first layer contact. 

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/ce98e2e7-2cc6-4705-b08b-09ccf418ec1a)

**8. Higher Level Metal Formation** = We need to planarize first the layer via CMP before adding a metal interconnect. Aluminum contact is used to connect the lower contact to higher metal layer. Process is repeated until the contact reached the outermost layer.
 - Mask 12 is for first contact hole
 - Mask 13 is for first Aluminum contact layer
 - Mask 14 is for second contact hole
 - Mask 15 is for second Aluminum contact layer. Mask 16 is for making contact to topmost layer. 
 
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/8ea3f0fd-c401-407e-a4b3-9ad584ff2f3d)

**Final fabricated CMOS**

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/45e4d946-ca82-4e43-97b8-ee3ca04faf13)

### Layout and Metal Layers:

When polysilicon crosses N-diffusion/P-diffusion (diffusion is also called implantation), then an NMOS/PMOS is created. [Explained here](https://electronics.stackexchange.com/questions/223973/why-diffusions-in-cmos-cad-tool-magic-is-continuous) is the reason why the diffusion layer of source and drain "seems" to be connected under the polysilicon (diffusion layer for source and drain supposedly be separated).

The first layer is local-interconnect layer or local-i then metal 1 to 5. [Here is the process stack diagram](https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html) of sky130nm PDK. Metal 1 is for Power and Ground lines. `Nsubstratecontact` connects the N-well to locali. `licon` connects the locali to metal1.Locali is for local connections of cells. 

The layer hierarchy for NMOS is: Psubstrate -> Psubstrate Diffusion (psd) -> Psubstrate Contact (psc) -> Local-interconnect (li) -> Mcon -> Metal1. For poly: Poly -> Polycontact -> Locali. P-substrate diffusion an N-substrate diffusion is also referred to as P-tap and N-tap. 

The output of the layout is the LEF file. [LEF (Library Exchange Format)](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) is used by the router tool in PnR design to get the location of standard cells pins to route them properly. So it is basically the abstract form of layout of a standard cell. `picorv32a/runs/[DATE]/tmp` contains the merged lef files (cell LEF and tech LEF). Notice how metal layer directon (horizontal or vertical) is alternating. Also, metal layer width and thickness is increasing. 

### Magic Commands:  
[Here is a great video guide](https://www.youtube.com/watch?v=RPppaGdjbj0) on layout using Magic. And [here is the Magic website](http://opencircuitdesign.com/magic/) with tutorials.
- Left click = lower-left corner of box  
- Right click = upper-right corner of box  
- "z" = zoom in, "Z" = zoom out, "ctrl + z" = zoom into the box 
- Middle click on empty area will turn the box into empty (similar to erasing it)
- "s" three times will select all geometries electrically connected to each other
  
 ***Some Commands used in tkcon***  
- `:box` = display parameters of selected box  
- `:grid` 0.5um 0.5um = turn on/off and set grid   
- `:snap user` = snap based on current grid  
- `:help snap` = display help for command  
- `:drc style drc(full)` = use all DRC when doing DRC checking
- `:paint poly` = paint "poly" to current box
- `:drc why` = show drc violation inside selected area (white dots are DRC violations )
- `:erase poly` = delete poly inside the box
- `:select area` = select all geometries inside the box
- `:copy n 30` = copy selected geometries to North by 30 grid steps
- `:move n 1` = move selected geometries to North by 1 step ("." to move more, "u" to undo)  
- `: select cell _08555_` = select a particular cell instance (e.g. cell \_08555_ which can be searched in the DEF file)
- `:cellname allcells` = list all cells in the layout
- `:cellname exists sky130_fd_sc_hd__xor3_4` = check if a cell exists 
- `:drc why` = show DRC violation and also the DRC name which can be referenced from [Sky130 PDK Periphery Rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root).




### Slew Rate and Propagation Delay Characterization:

The task is to characterize a sample inverter cell by its slew rate and propagation delay.  

1. View the mag file using magic `magic -T sky130A.tech sky130_inv.mag &`:  

![Screenshot from 2023-09-11 11-12-28](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/af78a05a-fc9e-47f0-87a1-254a224d53d7)

2. Make an extract file `.ext` by typing `extract all` in the tckon terminal. 
3. Extract the `.spice` file from this ext file by typing `ext2spice cthresh 0 rthresh 0` and then `ext2spice` in the tckon terminal.

 ![Screenshot from 2023-09-11 14-57-39](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/ef052699-8b45-4036-941b-598b6495a056)

![Screenshot from 2023-09-11 14-58-20](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/628b785a-8cfe-4749-a960-3b29e6b8d88d)    

We then modify the spice file to be able to plot a transient response:

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

//.subckt sky130_inv A Y VPWR VGND
M1000 Y A VGND VGND nshort_model.0 w=35 l=23
+  ad=1.44n pd=0.152m as=1.37n ps=0.148m
M1001 Y A VPWR VPWR pshort_model.0 w=37 l=23
+  ad=1.44n pd=0.152m as=1.52n ps=0.156m

VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

C0 A VPWR 0.0774f
C1 VPWR Y 0.117f
C2 A Y 0.0754f
C3 Y VGND 2f
C4 A VGND 0.45f
C5 VPWR VGND 0.781f
//.ends

.tran 1n 20n
.control
run
.endc
.end
```

Open the spice file by typing `ngspice sky130A_inv.spice`. Generate a graph using `plot y vs time a` :

![Screenshot from 2023-09-11 18-08-41](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6807150b-95d3-441d-9a1c-d12db74e2519)

![Screenshot from 2023-09-11 17-57-05](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/b31085e3-096d-4b6d-8938-db24d93c8035)


Using this transient response, we will now characterize the cell's slew rate and propagation delay:

![Screenshot from 2023-09-11 18-24-09](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/42372926-62f2-4f71-8402-b4ee11f1fee9)

characterization of the inverter standard cell depends on Four timing parameters
 
 **Rise Transition**: Time taken for the output to rise from 20% to 80% of max value

 **Fall Transition**: Time taken for the output to fall from 80% to 20% of max value

 **Cell Rise delay**: difference in time(50% output rise) to time(50% input fall)
 
 **Cell Fall delay**: difference in time(50% output fall) to time(50% input rise)
 
 The above timing parameters can be computed by noting down various values from the ngspice waveform.

 ![Screenshot from 2023-09-11 18-22-06](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/f0b45712-ca4c-4963-bd0e-93fb54cbe2f7)
 
 ``` 
 Rise Transition : 2.23871ns - 2.18198ns = 0.005673ns / 56.73ps
```
![Screenshot from 2023-09-11 18-23-42](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/e5319734-9646-473f-ad69-441390bce79d)
```
 Fall Transition : 4.09355ns - 4.04872ns = 0.04483ns  / 44.83ps 
 ```
![Screenshot from 2023-09-11 18-19-01](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/964bc942-326f-4265-b845-74bcc2f00ffa)

 ```
 Cell Rise Delay : 2.20684ns - 2.15214ns = 0.0547ns / 54.7ps 
 ```
![Screenshot from 2023-09-11 18-20-06](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/4d249cc6-90cf-45dd-9522-8e143be7a2a3)
 ```
 Cell Fall Delay : 4.07564ns - 4.0525ns = 0.02314ns / 23.14ps 
 ```




## Day-4








## Day-5






## Acknowledgement
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Chatgpt
- Kanish R,Colleague,IIIT B
- Alwin Shaju, Colleague IIITB
  
## Reference
- https://www.vsdiat.com
- https://github.com/nickson-jose/vsdstdcelldesign
- https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop
- https://github.com/riscv/riscv-gnu-toolchain
- https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130
