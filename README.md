# Advanced_Physical_Design_Using_OpenLANE-sky130

[Day-1 :  Introduction to open-source EDA, OpenLANE and Sky130 PDK](#day-1)

[Day-2 : Good floorplan vs bad floorplan and Introduction to library cells](#day-2)

[Day-3 : Design library cell using Magic Layout and ngspice characterization](#day-3)

[Day-4 : Pre-layout timing analysis and importance of good clock tree](#day-4)

[Day-5 : Final steps for RTL2GDS using tritonRoute and openSTA](#day-5)


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

OpenLane integrated several key open source tools over the execution stages:

1. RTL Synthesis, Technology Mapping, and Formal Verification : yosys + abc
2. Static Timing Analysis: OpenSTA
3. Floor Planning: init_fp, ioPlacer, pdn and tapcell
4. Placement: RePLace (Global), Resizer and OpenPhySyn (formerly), and OpenDP (Detailed)
5. Clock Tree Synthesis: TritonCTS
6. Fill Insertion: OpenDP/filler_placement
7. Routing: FastRoute or CU-GR (formerly) and TritonRoute (Detailed) or DR-CU
8. SPEF Extraction: OpenRCX or SPEF-Extractor (formerly)
9. GDSII Streaming out: Magic and KLayout
10. DRC Checks: Magic and KLayout
11. LVS check: Netgen
12. Antenna Checks: Magic
13. Circuit Validity Checker: CVC

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

Prior to the installation of the OpenLane install the dependencies and packages using the command shown below :</br>
``` 
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```
Docker Installation :</br>
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

**Steps to install OpenLane, PDKs and Tools**</br>
```
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane --recurse-submodules 
cd OpenLane
make
make test
cd /home/kanish/OpenLane/designs/ci
cp -r * ../
```

### Steps to run synthesis in OpenLane:


```
cd ~/OpenLane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
![Screenshot from 2023-09-09 12-45-02](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/ddc2a1bd-082f-4805-bdaa-5065fc640abf)

To view the synthesized nelist:
```
cd /home/nsaisampath/OpenLane/designs/picorv32a/runs/RUN_2023.09.09_07.11.58/results/synthesis
vim picorv32.v
```
![Screenshot from 2023-09-09 12-58-37](https://github.com/NSampathIIITB/Advanced_Physical_Design_Using_OpenLANE-sky130/assets/141038460/eeed499d-ef4e-45cd-a6d2-5670a6733d70)

To view the report after synthesis:
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


## Day-3






## Day-4








## Day-5






[Acknowledgement](#acknowledgement)
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Chatgpt
- Kanish R,Colleague,IIIT B
- Alwin Shaju, Colleague IIITB
- 
[Reference](#reference)
- https://www.vsdiat.com
- https://github.com/riscv/riscv-gnu-toolchain
- https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130
