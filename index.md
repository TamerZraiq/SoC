---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---
Tamer Zraiq

The following document consists of defining the process of designing and implementing a VGA driver on an FPGA. The project demonstrates some of the concepts of digital design using Verilog language and the Vivado environment. 
Starting with provided templates (colour stripes and colour cycle), I modified these templates to enhance the design, including simulation and synthesis to ensure functionality. The document outlines the process of how a VGA controller operates and how to implement various designs.


## **Template VGA Design : Given ColourStripes Template**
### **Project Set-Up**
The project is set up in Vivado to use the Artix-7 FPGA (xc7a35tcpg236-1). VGATop is the top module that integrates various submodules for clocking, VGA synchronization to manage synchronization signals and the ColourStripes module for generating changing color patterns. The design sources include VGATop.v, VGASync.v, and VGAColourStripes.v, while the simulation source has a testbench module. The synthesis, simulation and implementation results confirm that the bitstream was successfully generated, as shown in the Project Summary screenshot.

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

### **Template Code**
The template code generates VGA signals that are compatible with a standard VGA display. These modules outline the form of a simple VGA controller. A VGA interface operates by driving red, green, and blue signals along with synchronization signals to control the display of pixels. The VGATop.v module ties these components together, providing a clocking mechanism through clk_wiz_0 to ensure proper timing.

#### **VGASync.v Code**
The VGA synchronization module (VGASync.v) is used to generate horizontal (hsync) and vertical sync (vsync) signals based on a timing specification. It also calculates the active video region (vid_on), which determines the pixel output.

##### **Parameters**:
The following parameters specify the timing values required for VGA synchronization. HDISP and VDISP define the resolution of the display area. HFP/VFP, HPW/VPW, and HLIM/VLIM represent front porch, pulse width, and total line/frame limits.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

##### **Horizontal and Vertical Counter**:
The following code increments hcount and vcount in an always block to track pixel positions in horizontal and vertical frames. The counter goes through the total line(HLIM)/frame limits(VLIM). hcount resets after reaching the horizontal limit, then triggers vcount to increment vertical synchronization. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

##### **Sync Signal Generation and Active Video Logic**:
hsync signal is active low during the horizontal pulse width, while vsync is active low during vertical pulse width. These signals represent the sync intervals for the display. The following code is used to synchronzie the display hardware process.
The active video logic determines the region where the pixels are outputed. vid_on is high when the counters are in the active display area whcich is defined by the HDISP and VDISP.
Assign statements are used to map row and col to the vcount and hcount. These outputs are used for pixel calculations for the other modules (ColourStripes.v).
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

#### **VGAColourStripes.v Code**
The VGAColourStripes.v module generates RGB color patterns during the active video period to display on the screen.

##### **Parameters**:
The following parameters configure how the counter behaves. COUNTER_WIDTH: Defines the bit width of the counter used in the module, default is 32 bits. COUNT_FROM: Defines the starting value of the counter. COUNT_TO: Defines the maximum count value of the counter (2^26 in this case). COUNT_RESET: Specifies the reset value for the counter (2^27 in this case).
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

##### **Horizontal and Vertical Counter**:

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

##### **Sync Signal Generation and Active Video Logic**:

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">

### **Simulation**
Explain the simulation process. Reference any important details, include a well-selected screenshot of the simulation. Guideline: 1/2 short paragraphs.
### **Synthesis**
Describe the synthesis and implementation processes. Consider including 1/2 useful screenshot(s). Guideline: 1/2 short paragraphs.
### **Demonstration**
Perhaps add a picture of your demo. Guideline: 1/2 sentences.

## **My VGA Design Edit**
Introduce your own design idea. Consider how complex/achievabble this might be or otherwise. Reference any research you do online (use hyperlinks).
### **Code Adaptation**
Briefly show how you changed the template code to display a different image. Demonstrate your understanding. Guideline: 1-2 short paragraphs.
### **Simulation**
Show how you simulated your own design. Are there any things to note? Demonstrate your understanding. Add a screenshot. Guideline: 1-2 short paragraphs.
### **Synthesis**
Describe the synthesis & implementation outputs for your design, are there any differences to that of the original design? Guideline 1-2 short paragraphs.
### **Demonstration**
If you get your own design working on the Basys3 board, take a picture! Guideline: 1-2 sentences.

## **More Markdown Basics**
This is a paragraph. Add an empty line to start a new paragraph.

Font can be emphasised as *Italic* or **Bold**.

Code can be highlighted by using `backticks`.

Hyperlinks look like this: [GitHub Help](https://help.github.com/).

A bullet list can be rendered as follows:
- vectors
- algorithms
- iterators

Images can be added by uploading them to the repository in a /docs/assets/images folder, and then rendering using HTML via githubusercontent.com as shown in the example below.

<img src="https://raw.githubusercontent.com/melgineer/fpga-vga-verilog/main/docs/assets/images/VGAPrjSrcs.png">
