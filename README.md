
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
cd /home/nsaisampath/OpenLane/designs/ci
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

``` magic -T .libs/sky130A.tech sky130_inv.mag & ```

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

1. View the mag file using magic `magic -T .libs/sky130A.tech sky130_inv.mag &`:  

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

### LAB exercises and DRC Challenges

### Intrdocution of Magic and Skywater DRC's

  - In-depth overview of Magic's DRC engine
  - Introduction to Google/Skywater DRC rules
  - Lab : Warm-up exercise : Fixing a simple rule error
  - Lab : Main exercie : Fixing or create a complex error

 #### Sky130s PDK Intro and Steps to download labs:
  
  - setup to view the layouts
  - For extracting and generating views, Google/skywater repo files were built with Magic
  - Technology file dependency is more for any layout. hence, this file is created first.
  - Since, Pdk is still under development, there are some unfinished tech files and these are packaged for magic along with lab exercise layout and bunch of stuff into the tar ball
    
Read through [this site about tech file](http://opencircuitdesign.com/magic/techref/maint2.html). All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types, design rules, rules for mask generation, and rules for extracting netlists for circuit simulation. 
Read through also [this site on the DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)
    
 
We can download the packaged files from web using ``wget `` command. wget stands for web get, a non-interactive file downloader command.
  
  ``` wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz```
  
The archive file drc_tests.tgz is downloaded into our user directory 

 ![Screenshot from 2023-09-11 20-23-51](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/57d90d98-c819-4058-b66e-ad35e51a1885) 

once extraction is done, drc_tests file is created and you will have all the information about magic layout for this lab exercise

Now run MAGIC

For better graphics use command ``magic -d XR ``

Now, lets see an example of simple failing set of rules of metal 3 layer.  you can either run this by magic command line `` magic -d XR met3.mag `` or from the magic console window, `` menu - file - open -load file9here, met3.mag) ``

![Screenshot from 2023-09-11 20-51-16](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6c9d5dd3-dc2a-4080-9453-94e5b6926422)

We use following commands to see metal cut as shown.
```
cif see VIA2

```
![Screenshot from 2023-09-11 20-58-05](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/07dee56b-c86b-4976-90bd-0e296afa1c2a)

### Fix Tech File DRC via Magic: 

1. Open magic with `poly.mag` as input: `magic poly.mag`. Focus on `Incorrect poly.9` layout. As described on the poly.9 [design rule of SKY130 PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly), the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using `:box`, we can see that the distance is 0.200um YET there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.

![Screenshot from 2023-09-11 22-10-43](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/37924203-45b9-4617-8cd4-3757044122c3)   

2. We should go to sky130A.tech file and modify as follows to detect this error.
   
![Screenshot from 2023-09-11 21-57-23](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/aa799d54-fd91-478c-a891-3111fa38330a)

In line

```
spacing npres *nsd 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
change to

```
spacing npres allpolynonres 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
Also,
```
spacing xhrpoly,uhrpoly,xpc alldiff 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"
```

change to 

```
spacing xhrpoly,uhrpoly,xpc allpolynonres 480 touching_illegal \

	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"

```
![Screenshot from 2023-09-11 21-55-38](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/d81d4f8e-b42b-4b75-a1b4-f995e584bfca)


## Day-4

### Timing Analysis and Clock Tree Synthesis (CTS)

### Standard Cell LEF generation

During Placement, entire mag information is not necessary. Only the PR boundary, I/O ports, Power and ground rails of the cell is required. This information is defined in LEF file.
The main objective is to extract lef from the mag file and plug into our design flow.

#### Grid into Track info

 **Track** :A path or a line on which metal layers are drawn for routing. Track is used to define the height of the standard cell. 

To implement our own stdcell, few guidelines must be followed 
 - I/O ports must lie on the intersection of Horizontal and vertical tracks
 - Width and Height of standard cell are odd mutliples of Horizontal track pitch and Vertical track pitch

This information is defined in ``tracks.info``. 

```
li1 X 0.23 0.46 
li1 Y 0.17 0.34
met1 X 0.17 0.34 
met1 Y 0.17 0.34
met2 X 0.23 0.46 
met2 Y 0.23 0.46
met3 X 0.34 0.68 
met3 Y 0.34 0.68
met4 X 0.46 0.92 
met4 Y 0.46 0.92
met5 X 1.70 3.40 
met5 Y 1.70 3.40

```

before grid on:

![Screenshot from 2023-09-12 00-16-53](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/e668806c-3b23-4ea0-b7aa-74210c406ad1)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. After providing the command, we have following:

```
grid 0.46um 0.34um 0.23um 0.17um

```
![Screenshot from 2023-09-12 00-17-40](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/99cc7267-49d2-40db-88ca-30707f6cc3c5)

![Screenshot from 2023-09-12 00-18-01](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/5cd630e2-0293-431d-811d-633bd2870cd1)


### Create port definition

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 

The easiest way to define a port is through Magic Layout window and following are the steps:

- In Magic Layout window, first source the .mag file for the design (here inverter). Then **Edit >> Text** which opens up a dialogue box.
  
![Screenshot from 2023-09-12 00-42-22](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/be5b90e7-fa48-4d5b-997e-69facf9e5c93)

- For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:

![Screenshot from 2023-09-12 00-51-59](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/b2491aba-03dc-447c-9f83-12e70950578b)

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

- For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Notice the sticky label).

![Screenshot from 2023-09-12 00-54-10](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/09fa9a2e-8138-47f2-a89a-e10cff6ef698)

![Screenshot from 2023-09-12 00-54-40](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/2c93f555-b98e-4a59-bd9a-7b55dfa11094)

### Standard Cell LEF generation

Before the CMOS Inverter standard cell LEF is extracted, the purpose of ports must be defined:

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port use signal
```
Select VPWR area
```
port class inout
port use power
```

Select VGND area
```
port class inout
port use ground
```

LEF extraction can be carried out in tkcon as follows:

```
save sky130_vsdinv.mag
lef write
```
![Screenshot from 2023-09-12 02-12-33](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/f08e5eb3-f59d-46f8-a9f1-82914404fe16)

This generates ```sky130_vsdinv.lef``` file.

![Screenshot from 2023-09-12 02-13-57](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/040ff0e8-20f8-4cd4-adc2-c99ef90af36b)

```
VERSION 5.7 ;
  NOWIREEXTENSIONATPIN ON ;
  DIVIDERCHAR "/" ;
  BUSBITCHARS "[]" ;
MACRO sky130_vsdinv
  CLASS CORE ;
  FOREIGN sky130_vsdinv ;
  ORIGIN 0.000 0.000 ;
  SIZE 1.380 BY 2.720 ;
  SITE unithd ;
  PIN A
    DIRECTION INPUT ;
    USE SIGNAL ;
    ANTENNAGATEAREA 0.165600 ;
    PORT
      LAYER li1 ;
        RECT 0.060 1.180 0.510 1.690 ;
    END
  END A
  PIN Y
    DIRECTION OUTPUT ;
    USE SIGNAL ;
    ANTENNADIFFAREA 0.287800 ;
    PORT
      LAYER li1 ;
        RECT 0.760 1.960 1.100 2.330 ;
        RECT 0.880 1.690 1.050 1.960 ;
        RECT 0.880 1.180 1.330 1.690 ;
        RECT 0.880 0.760 1.050 1.180 ;
        RECT 0.780 0.410 1.130 0.760 ;
    END
  END Y
  PIN VPWR
    DIRECTION INOUT ;
    USE POWER ;
    PORT
      LAYER nwell ;
        RECT -0.200 1.140 1.570 3.040 ;
      LAYER li1 ;
        RECT -0.200 2.580 1.430 2.900 ;
        RECT 0.180 2.330 0.350 2.580 ;
        RECT 0.100 1.970 0.440 2.330 ;
      LAYER mcon ;
        RECT 0.230 2.640 0.400 2.810 ;
        RECT 1.000 2.650 1.170 2.820 ;
      LAYER met1 ;
        RECT -0.200 2.480 1.570 2.960 ;
    END
  END VPWR
  PIN VGND
    DIRECTION INOUT ;
    USE GROUND ;
    PORT
      LAYER li1 ;
        RECT 0.100 0.410 0.450 0.760 ;
        RECT 0.150 0.210 0.380 0.410 ;
        RECT 0.000 -0.150 1.460 0.210 ;
      LAYER mcon ;
        RECT 0.210 -0.090 0.380 0.080 ;
        RECT 1.050 -0.090 1.220 0.080 ;
      LAYER met1 ;
        RECT -0.110 -0.240 1.570 0.240 ;
    END
  END VGND
END sky130_vsdinv
END LIBRARY
```

### Integrating custom cell in OpenLANE

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.json``` must be modified:
```
"PL_RANDOM_GLB_PLACEMENT": 1,
"PL_TARGET_DENSITY": 0.5,
"FP_SIZING": "relative",
"LIB_SYNTH":"dir::src/sky130_fd_sc_hd__typical.lib",
"LIB_FASTEST":"dir::src/sky130_fd_sc_hd__fast.lib",
"LIB_SLOWEST":"dir::src/sky130_fd_sc_hd__slow.lib",
"LIB_TYPICAL":"dir::src/sky130_fd_sc_hd__typical.lib",
"TEST_EXTERNAL_GLOB":"dir::../picorv32a/src/*",
"SYNTH_DRIVING_CELL":"sky130_vsdinv"

```



In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a 
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
![Screenshot from 2023-09-13 19-15-12](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/2f786b33-9554-4a8d-875d-a566345e9872)

![Screenshot from 2023-09-13 19-20-48](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/1f925aad-96b3-412d-af85-78ad7c5eb415)

![Screenshot from 2023-09-13 19-21-51](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/a87becdd-a964-4d8b-8baa-503111c93d06)


Next floorplan is run, followed by placement:

```
run_floorplan
run_placement
```
![Screenshot from 2023-09-14 14-27-32](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/38e5dbe7-d3b3-4e50-a05a-c61949e2c860)

To check the layout invoke magic from the ```runs/RUN_2023.09.14_08.55.14 ``` directory:

```
magic -T /home/nsaisampath/.volare/sky130A/libs.tech/magic/sky130A.tech lef read tmp/merged.nom.lef def read results/placement/picorv32.def &

```
![Screenshot from 2023-09-14 14-47-00](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/7008f5e3-bd62-4012-be96-b369ec78c092)

![Screenshot from 2023-09-14 14-50-05](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/77eb43cb-dbd0-43a6-95e4-0698e2bb1d64)

![Screenshot from 2023-09-14 14-50-16](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/104a5a74-3fff-4c0a-8ffc-f1a983261e82)

Since the custom standard cell has been plugged into the openLANE flow, it would be visible in the layout.

### Delay Tables

Basically, Delay is a parameter that has huge impact on our cells in the design. Delay decides each and every other factor in timing. 
For a cell with different size, threshold voltages, delay model table is created where we can it as timing table.
```Delay of a cell depends on input transition and out load```. 
Lets say two scenarios, 
we have long wire and the cell(X1) is sitting at the end of the wire : the delay of this cell will be different because of the bad transition that caused due to the resistance and capcitances on the long wire.
we have the same cell sitting at the end of the short wire: the delay of this will be different since the tarn is not that bad comapred to the earlier scenario.
Eventhough both are same cells, depending upon the input tran, the delay got chaned. Same goes with o/p load also.

VLSI engineers have identified specific constraints when inserting buffers to preserve signal integrity. They've noticed that each buffer level must maintain consistent sizing, but their delays can vary depending on the load they drive. To address this, they introduced the concept of "delay tables," which essentially consist of 2D arrays containing values for input slew and load capacitance, each associated with different buffer sizes. These tables serve as timing models for the design.

When the algorithm works with these delay tables, it utilizes the provided input slew and load capacitance values to compute the corresponding delay values for the buffers. In cases where the precise delay data is not readily available, the algorithm employs a technique of interpolation to determine the closest available data points and extrapolates from them to estimate the required delay values.

In order to avoid large skew between endpoints of a clock tree (signal arrives at different point in time):
 - Buffers on the same level must have same capacitive load to ensure same timing delay or latency on the same level. 
 - Buffers on the same level must also be the same size (different buffer sizes -> different W/L ratio -> different resistance -> different RC constant -> different delay).    
 
 ![image](https://user-images.githubusercontent.com/87559347/188773408-e503023f-0288-4993-a68a-5f20bccb886c.png)


Buffers on different level will have different capacitive load and buffer size but as long as they are the same load and size on the same level, the total delay for each clock tree path will be the same thus skew will remain zero. **This means different levels will have varying input transition and output capacitive load and thus varying delay.** 

Delay tables are used to capture the timing model of each cell and is included inside the liberty file. The main factor in delay is the output slew. The output slew in turn depends on **capacitive load** and **input slew**. The input slew is a function of previous buffer's output cap load and input slew and it also has its own transition delay table.

![image](https://user-images.githubusercontent.com/87559347/188783693-423bd170-dd0b-4f2f-9652-8fae9418df31.png)

Notice how skew is zero since delay for both clock path is x9'+y15.

### Post-synthesis timing analysis Using OpenSTA 

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, ```pre_sta.conf``` is required to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:
 
```
sta pre_sta.conf
```
Since clock tree synthesis has not been performed yet, the analysis is with respect to ideal clocks and only setup time slack is taken into consideration. The slack value is the difference between data required time and data arrival time. The worst slack value must be greater than or equal to zero. If a negative slack is obtained, following steps may be followed:
1. Change synthesis strategy, synthesis buffering and synthesis sizing values 
2. Review maximum fanout of cells and replace cells with high fanout
sdc file for OpenSTA is modified like this:

base.sdc is located in vsdstdcelldesigns/extras directory.
So, I copied it into our design folder using

``` cp my_base.sdc /home/nsaisampath/OpenLane/designs/picorv32a/src/ ```

![Screenshot from 2023-09-16 10-22-08](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/f42621e4-3c0d-4086-ab6e-25251579a7b7)


Since I have no Violations I skipped this, but have hands on experience on timing analysis using OpenSTA.

Since clock is propagated only once we do CTS, In placement stage, clock is considered to be ideal. So only setup slack is taken into consideration before CTS.


clock is generated from PLL which has inbuilt circuit which cells and some logic. There might variations in the clock generation depending upon the ckt. These variations are collectivity known as clock uncertainity. In that jitter is one of the parameter. It is uncertain that clock might come at that exact time withought any deviation. That is why it is called clock_uncertainity
Skew, Jitter and Margin comes into clock_uncertainity

```  Clock Jitter : deviation of clock edge from its original position. ```



### Clock Tree Synthesis :

There are three parameters that we need to consider when building a clock tree:
- Clock Skew = In order to have minimum skew between clock endpoints, clock tree is used. This results in equal wirelength (thus equal latency/delay) for every path of the clock. 
- Clock Slew = Due to wire resistance and capacitance of the clock nets, there will be slew in signal at the clock endpoint where signal is not the same with the original input clock signal anymore. This can be solved by clock buffers. Clock buffer differs in regular cell buffers since clock buffers has equal rise and fall time. 


![image](https://user-images.githubusercontent.com/87559347/190031283-3bc25c79-f622-4b58-a448-95982d32612d.png)

Clock tree synthesis (CTS) can be implemented in various ways, and the choice of the specific technique depends on the design requirements, constraints, and goals. Here are some different types or approaches to clock tree synthesis:

Balanced Tree CTS:
In a balanced tree CTS, the clock signal is distributed in a balanced manner, often resembling a binary tree structure.
This approach aims to provide roughly equal path lengths to all clock sinks (flip-flops) to minimize clock skew.
It's relatively straightforward to implement and analyze but may not be the most power-efficient solution.

H-tree CTS:
An H-tree CTS uses a hierarchical tree structure, resembling the letter "H."
It is particularly effective for distributing clock signals across large chip areas.
The hierarchical structure can help reduce clock skew and optimize power consumption.

Star CTS:
In a star CTS, the clock signal is distributed from a single central point (like a star) to all the flip-flops.
This approach simplifies clock distribution and minimizes clock skew but may require a higher number of buffers near the source.

Global-Local CTS:
Global-Local CTS is a hybrid approach that combines elements of both star and tree topologies.
The global clock tree distributes the clock signal to major clock domains, while local trees within each domain further distribute the clock.
This approach balances between global and local optimization, addressing both chip-wide and domain-specific clocking requirements.

Mesh CTS:
In a mesh CTS, clock wires are arranged in a mesh-like grid pattern, and each flip-flop is connected to the nearest available clock wire.
It is often used in highly regular and structured designs, such as memory arrays.
Mesh CTS can offer a balance between simplicity and skew minimization.

Adaptive CTS:
Adaptive CTS techniques adjust the clock tree structure dynamically based on the timing and congestion constraints of the design.
This approach allows for greater flexibility and adaptability in meeting design goals but may be more complex to implement.

#### Crosstalk in VLSI:

Impact: Crosstalk is a significant concern in VLSI design due to the high integration density of components on a chip. Uncontrolled crosstalk can lead to data corruption, timing violations, and increased power consumption.
Mitigation: VLSI designers employ various techniques to mitigate crosstalk, such as optimizing layout and routing, using appropriate shielding, implementing proper clock distribution strategies, and utilizing clock gating to reduce dynamic power consumption when logic is idle.Clock shielding prevents crosstalk to nearby nets by breaking the coupling capacitance between the victim (clock net) and aggresor (nets near the clock net), the shield might be connected to VDD or ground since those will not switch. Shileding can also be done on critical data nets.

#### Clock Net Shielding in VLSI:

Purpose: In VLSI circuits, the clock distribution network is crucial for synchronous operation. Clock signals must reach all parts of the chip while minimizing skew and maintaining signal integrity.
Shielding Techniques: VLSI designers may use shielding techniques to isolate the clock network from other signals, reducing the risk of interference. This can include dedicated clock routing layers, clock tree synthesis algorithms, and buffer insertion to manage clock distribution more effectively.
Clock Domain Isolation: VLSI designs often have multiple clock domains. Shielding and proper clock gating help ensure that clock signals do not propagate between domains, avoiding metastability issues and maintaining synchronization.


#### CTS

In this stage clock is propagated and make sure that clock reaches each and every clock pin from clock source with minimum skew and insertion delay. Inorder to do this, we implement H-tree using mid point strategy. For balancing the skews, we use clock inverters or buffers in the clock path. Before attempting to run CTS in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. 

To run CTS use the below command:

```
run_cts
```
![Screenshot from 2023-09-16 10-43-50](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/c7f9f2e8-79c3-43f3-a812-62dee4813ff1)

![Screenshot from 2023-09-16 12-10-00](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/9464e672-c6e9-402e-b341-383e51312366)

After CTS run, my slack values are
``` setup:12.97, Hold:0.36 ```

Here the both values are not violating 

#### Setup Timing Analysis using real clocks

Setup time analysis is a critical aspect of digital circuit design, particularly in synchronous digital systems. It refers to the amount of time a signal must be stable and valid before the clock edge arrives. Ensuring that setup time requirements are met is essential to prevent data corruption and ensure the proper operation of the digital circuit.

![Screenshot from 2023-09-16 10-58-28](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/9dc1fb93-111b-41ca-9ea6-227b1c9d238e)

![Screenshot from 2023-09-16 10-59-40](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/54126cbd-5012-4302-820d-96395ee6ada2)

![Screenshot from 2023-09-16 11-01-12](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/2d27d960-7771-4ddb-a41a-bc3dbab2d717)

![Screenshot from 2023-09-16 11-02-33](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/30e1e9d4-fc8e-477c-b1cf-acdddd138b39)

To ensure setup time requirements are met, you need to:

   1. Select Proper Flip-Flops/Latches: Choose flip-flops or latches with appropriate clock-to-Q delays for your design.

   2. Optimize Combinational Logic: Minimize the propagation delay through the combinational logic by optimizing the logic gates used and the routing paths.

   3. Clock Skew Analysis: Consider clock skew, which is the variation in arrival times of the clock signal at different flip-flops. Ensure that clock skew does not cause setup time violations.

   4. Timing Constraints: Use tools like static timing analysis (STA) to analyze the entire design and verify that setup time requirements are met under various conditions, including process variations and corner cases.

Meeting setup time requirements is crucial for reliable and robust digital circuit operation. Failing to do so can result in data errors and malfunctioning of the circuit. Therefore, careful setup time analysis and design considerations are essential in digital circuit design.

#### Hold Timing Analysis using real clocks

Hold time analysis is another critical aspect of digital circuit design, particularly in synchronous systems. It refers to the minimum amount of time a data input (D) must remain stable and valid after the clock edge before it can change. Ensuring that hold time requirements are met is essential to prevent data corruption and ensure the proper operation of digital circuits.

![Screenshot from 2023-09-16 11-04-57](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/a781da96-8b34-4295-8b84-79682c36efb3)

![Screenshot from 2023-09-16 11-07-27](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/01e685d6-4219-44ab-a3a4-e0e4c231f8d4)

![Screenshot from 2023-09-16 11-07-50](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/60f00d94-0d75-4b77-9c43-967186be5ad4)

![Screenshot from 2023-09-16 11-14-46](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/c2047be2-e90c-4b24-9e91-c204099337b2)

![Screenshot from 2023-09-16 11-15-02](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/64631fb4-6272-4026-a231-2d3cb9e197ba)

![Screenshot from 2023-09-16 11-15-51](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/916d8ae8-5e6c-47e3-bf33-d292871f80e7)

To ensure hold time requirements are met, you need to:

   1. Select Proper Flip-Flops/Latches: Choose flip-flops or latches with appropriate clock-to-Q delays for your design.

   2. Optimize Combinational Logic: Minimize the propagation delay through the combinational logic by optimizing the logic gates used and the routing paths.

   3. Clock Skew Analysis: Similar to setup time analysis, consider clock skew to ensure that it doesn't cause hold time violations.

   4. Timing Constraints: Use tools like static timing analysis (STA) to analyze the entire design and verify that hold time requirements are met under various conditions, including process variations and corner cases.

Meeting hold time requirements is as crucial as meeting setup time requirements to ensure reliable and robust digital circuit operation. Failing to do so can lead to data errors and circuit malfunctions. Therefore, thorough hold time analysis and design considerations are essential in digital circuit design.<br>

#### Timing analysis using real clocks

![Screenshot from 2023-09-16 11-17-05](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/0493b8a1-2eb5-448e-b04c-35a3c3fe674e)

![Screenshot from 2023-09-16 11-18-33](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/aef76655-f5b8-4924-aeeb-c4eea6ea1994)

![Screenshot from 2023-09-16 11-19-39](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/d1eb0ea7-c37d-4b20-9adb-c0909aec6a2e)

![Screenshot from 2023-09-16 11-19-54](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/16927fd4-9d1d-4792-abb0-6a7c7f4b129a)

![Screenshot from 2023-09-16 11-20-18](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/e1c35b06-8f85-42db-8923-2ccf59884059)

![Screenshot from 2023-09-16 11-20-37](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/45d0c419-c064-4679-8197-d8463f060b97)


Since, clock is propagated, from this stage, we do timing analysis with real clocks. From now post cts analysis is performed by operoad within the openlane flow 


```
openroad
read_lef <path of merge.nom.lef>
read_def <path of def>
write_db pico_cts.db
read_db pico_cts.db
read_verilog /home/nsaisampath/OpenLane/designs/picorv32a/runs/RUN_2023.09.16_05.10.04/results/synthesis/picorv32.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
read_sdc /home/nsaisampath/OpenLane/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

Hold slack:

![Screenshot from 2023-09-16 12-28-31](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/6fba7bcc-ec87-431f-926a-f9917351a1ee)


setup slack:

![Screenshot from 2023-09-16 12-28-15](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/9f622390-e662-4a22-ba87-944722a50fae)


#### Test:

Try this in openlane
```
echo $::env(CTS_CLK_BUFFER_LIST)
set $::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]
echo $::env(CTS_CLK_BUFFER_LIST)
```
After changing the files, load the placement stage def file and run cts again. 
Now, again run OpenROAD and create another db and everything else is same.


## Day-5

### Maze Routing and Lee's algorithm

Routing is the process of establishing a physical connection between two pins. Algorithms designed for routing take source and target pins and aim to find the most efficient path between them, ensuring a valid connection exists.

The Maze Routing algorithm, such as the Lee algorithm, is one approach for solving routing problems. In this method, a grid similar to the one created during cell customization is utilized for routing purposes. The Lee algorithm starts with two designated points, the source and target, and leverages the routing grid to identify the shortest or optimal route between them.

The algorithm assigns labels to neighboring grid cells around the source, incrementing them from 1 until it reaches the target (for instance, from 1 to 7). Various paths may emerge during this process, including L-shaped and zigzag-shaped routes. The Lee algorithm prioritizes selecting the best path, typically favoring L-shaped routes over zigzags. If no L-shaped paths are available, it may resort to zigzag routes. This approach is particularly valuable for global routing tasks.

However, the Lee algorithm has limitations. It essentially constructs a maze and then numbers its cells from the source to the target. While effective for routing between two pins, it can be time-consuming when dealing with millions of pins. There are alternative algorithms that address similar routing challenges.

Here in this case he shortest path is one that follows a steady increment of one (1-to-9 on the example below). There might be multiple path like this but the best path that the tool will choose is one with less bends. The route should not be diagonal and must not overlap an obstruction such as macros.

This algorithm however has high run time and consume a lot of memory thus more optimized routing algorithm is preferred (but the principles stays the same where route with shortest path and less bends is preferred)

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/c30e0be8-7bd4-4f05-bc79-334006e2858a)

### Design Rules Check (DRC)

Design rule checks are nothing but physical checks of metal width, pitch and spacing requirement for the different layers which depend on different technology nodes. We need to clean up the DRC of the design because there is a logical connection of various components, and if they are physically connected, then it will fail the functionality of the chips, and chips won’t be able to perform a specific task.

The layout of a design must be in accordance with a set of predefined technology rules given by the foundry for manufacturability. After completion of the layout and its physical connection, an automatic program will check each and every polygon in the design against these design rules and report any violations. This whole process is called Design Rule Checking (DRC). There are many design rules at different technology nodes, a few of which are mentioned below.

Types of DRCs:

   - Minimum width and spacing for metal
   - Minimum width and spacing for via
   - Fat wire Via keep out Enclosure
   - End of Line spacing
   - Minimum area
   - Over Max stack level
   - Wide metal jog
   - Misaligned Via wire
   - Different net spacing
   - Special notch spacing
   - Shorts violation
   - Different net Via cut spacing
   - Less than min edge length

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/0b72e0e0-ddc9-4c67-96b2-c09d4539473b)
 
![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/72931273-1da5-4031-a712-766239e59516)

### Power Distribution Network Generation

Unlike the general ASIC flow, Power Distribution Network generation is not a part of floorplan run in OpenLANE. PDN must be generated after CTS and post-CTS STA analyses:

we can check whether PDN has been created or no by check the current def environment variable: ``` echo $::env(CURRENT_DEF)```

```
gen_pdn

```
We can confirm the success of PDN by checking the current def environment variable: ``` echo $::env(CURRENT_DEF) ```

![Screenshot from 2023-09-16 21-50-35](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/d32eed9c-d0ca-445b-9c81-58c012378b71)

![Screenshot from 2023-09-16 21-48-45](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/4c252a16-9113-4eff-a3b2-406f7b23b8a4)


- `gen_pdn` - Generates the Power Distribution network
- The power distribution network has to take the `design_cts.def` as the input def file.
- This will create the grid and the straps for the Vdd and the ground. These are placed around the standard cells.
- The standard cells are designed such that it's height is multiples of the space between the Vdd and the ground rails. Here, the pitch is `2.72`. Only if the above conditions are adhered it is possible to power the standard cells.
- The power to the chip, enters through the `power pads`. There is each for Vdd and Gnd
- From the pads, the power enters the `rings`, through the `via`
- The `straps` are connected to the ring. Vdd straps are connected to the Vdd ring and the Gnd Straps are connected to the Gnd ring. There are horizontal and the vertical straps
- Now the power has to be supplied from the straps to the standard cells. The straps are connected to the `rails` of the standard cells
- If macros are present then the straps attach to the `rings` of the macros via the `macro pads` and the pdn for the macro is pre-done.
- There are definitions for the straps and the railss. In this design straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.

### Power Distribution Network

This is just a review on PDN. The power and ground rails has a pitch of 2.72um thus the reason why the customized inverter cell has a height of 2.72 or else the power and ground rails will not be able to power up the cell. Looking at the LEF file runs/[date]/tmp/merged.nom.lef, you will notice that all cells are of height 2.72um and only width differs.

As shown below, power and ground flows from power/ground pads -> power/ground ring-> power/ground straps -> power/ground rails.

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/634bca91-d895-457b-b0ca-267809695fa6)

### Routing

In the realm of routing within Electronic Design Automation (EDA) tools, such as both OpenLANE and commercial EDA tools, the routing process is exceptionally intricate due to the vast design space. To simplify this complexity, the routing procedure is typically divided into two distinct stages: Global Routing and Detailed Routing.

The two routing engines responsible for handling these two stages are as follows:

  1. Global Routing: In this stage, the routing region is subdivided into rectangular grid cells and represented as a coarse 3D routing graph. This task is accomplished by the "FASTE ROUTE" engine.

  2. Detailed Routing: Here, finer grid granularity and routing guides are employed to implement the physical wiring. The "tritonRoute" engine comes into play at this stage. "Fast Route" generates initial routing guides, while "Triton Route" utilizes the Global Route information and further refines the routing, employing various strategies and optimizations to determine the most optimal path for connecting the pins.

**Triton Route**

 - Performs detailed routing and honors the pre-processed route guides (made by global route) and uses MILP based (Mixed  Integer Linear Programming algorithm) panel routing scheme(uses panel as the grid guide for routing) with intra-layer parallel routing (routing happens simultaneously in a single layer) and inter-layer sequential layer (routing starts from bottom metal layer to top metal layer sequentially and not simultaneously).
    
 - Honors preferred direction of a layer. Metal layer direction is alternating (metal layer direction is specified in the LEF file e.g. met1 Horizontal, met2 Vertical, etc.) to reduce overlapping wires between layer and reduce potential capacitance which can degrade the signal.  

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/0a04cd46-e387-4f1c-90e2-cce87d0c861e)
 
 ![image](https://user-images.githubusercontent.com/87559347/190557016-163a2d31-b650-4924-b937-69e775a21213.png)
Best reference for this the [Triton Route paper](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiHkP7pnZj6AhUFHqYKHcBlC3UQFnoECBEQAQ&url=https%3A%2F%2Fvlsicad.ucsd.edu%2FPublications%2FConferences%2F363%2Fc363.pdf&usg=AOvVaw0ywnaeyGqzqAjI6TaJnamd).

### Key Features of TritonRoute

- **Initial Detail Routing**: TritonRoute initiates the detailed routing process, providing the foundation for the subsequent routing steps.

- **Adherence to Pre-Processed Route Guides**: TritonRoute places significant emphasis on following pre-processed route guides. This involves several actions:

   - **Initial Route Guide Analysis**: TritonRoute analyzes the directions specified in the preferred route guides. If any non-directional routing guides are identified, it breaks them down into unit widths.

   - **Guide Splitting**: In cases where non-directional routing guides are encountered, TritonRoute divides them into unit widths to facilitate routing.

   - **Guide Merging**: TritonRoute merges guides that are orthogonal (touching guides) to the preferred guides, streamlining the routing process.

   - **Guide Bridging**: When it encounters guides that run parallel to the preferred routing guides, TritonRoute employs an additional layer to bridge them, ensuring efficient routing within the preprocessed guides.
   - Assumes route guide for each net satisfy inter guide connectivity Same metal layer with touching guides or neighbouring metal layers with nonzero  vertically overlapped area( via are placed ).each unconnected termial i.e., pin of a standard cell instance should have its pin shape overlapped by a routing guide( a black dot(pin) with purple box(metal1 layer))
     
In summary, TritonRoute is a sophisticated tool that not only performs initial detail routing but also places a strong emphasis on optimizing routing within pre-processed route guides by breaking down, merging, and bridging them as needed to achieve efficient and effective routing results.


### TritonRoute problem statement

```
Inputs : LEF, DEF, Preprocessed route guides
Output : Detailed routing solution with optimized wire length and via count
Constraints : Route guide honoring, connectivity constraints and design rules.

```
The space where the detailed route takes place has been defined. Now TritonRoute handles the connectivity in two ways.

Access Point(AP) : An on-grid point on the metal of the route guide, and is used to connect to lower-layer segments, pins or IO ports,upper-layer segments.<br>
Access Point Cluster(APC) : A union of all the Aps derived from same lower-layer segment, a pin or an IO port, upper-layer guide.

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/1d1c631e-8261-4ae8-893c-5517c0404c59)

![image](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/b1069fc3-8285-40a5-9421-c876bb0286eb)

**TritonRoute run for routing**

Make sure the CURRENT_DEF is set to pdn.def

Start routing by using

```
run_routing
```
![Screenshot from 2023-09-17 15-10-57](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/5baccc63-e1d9-471d-b984-def6788cf5f3)

![Screenshot from 2023-09-17 15-33-19](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/56b4ee19-9115-456b-8b4f-7a12c98ec184)

The optimisations in routing can also be done by specifying the routing strategy to use different version of TritonRoute Engine. There is a tradeoff between the optimised route and the runtime for routing.

For the default setting picorv32a takes approximately 30 minutes according to the current version of TritonRoute.

Here drc violation is zero.

### Layout in magic tool post routing: 

The design can be viewed on magic within results/routing directory. Run the follwing command in that directory:

```
magic -T /home/nsaisampath/.volare/sky130A/libs.tech/magic/sky130A.tech lef read tmp/merged.nom.lef def read picorv32.def &

```

![WhatsApp Image 2023-09-17 at 18 11 07](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/e07e39f3-a7f6-4250-b112-45992dba9a6c)

### Slack report post routing:

![Screenshot from 2023-09-17 16-22-51](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/95e838de-8ec9-44d7-92bd-86e75caec2f9)

### Post-synthesis flip flop to standard cell ratio

flip-flop to standard cell ratio = 1596/9819 = 0.16

![Screenshot from 2023-09-17 16-27-41](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/d9948c08-ad58-417b-a08d-1b28168b7627)

### Post-synthesis Gate count:-

![Screenshot from 2023-09-17 16-27-07](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/7688f5cb-64c9-4fe4-b728-108eee317f69)

## Openlane Interactive flow:

```
cd OpenLane

./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
detailed_placement
run_cts
gen_pdn
run_routing

```
## OpenLANE non-interactive flow

```
cd Desktop/OpenLane 
make mount
./flow.tcl -design picorv32a

```


## Acknowledgement
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Chatgpt
- Kanish R,Colleague,IIIT B
- Alwin Shaju, Colleague IIIT B
  
## Reference
- https://www.vsdiat.com
- https://github.com/nickson-jose/vsdstdcelldesign
- https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop
- https://github.com/riscv/riscv-gnu-toolchain
- https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130
