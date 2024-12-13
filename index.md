---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---
Tamer Zraiq

The following document consists of defining the process of designing and implementing a VGA driver on an FPGA. The project demonstrates some of the concepts of digital design using Verilog language and the Vivado environment. 
Starting with provided templates (colour stripes and colour cycle) and with the help of ChatGPT to explain certain aspects of the application and error solve, I modified these templates to enhance the design, including simulation and synthesis to ensure functionality. The document outlines the process of how a VGA controller operates and how to implement various designs.


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

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/SyncParam.png">

##### **Horizontal and Vertical Counter**:
The following code increments hcount and vcount in an always block to track pixel positions in horizontal and vertical frames. The counter goes through the total line(HLIM)/frame limits(VLIM). hcount resets after reaching the horizontal limit, then triggers vcount to increment vertical synchronization. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Sync1.png">

##### **Sync Signal Generation and Active Video Logic**:
hsync signal is active low during the horizontal pulse width, while vsync is active low during vertical pulse width. These signals represent the sync intervals for the display. The following code is used to synchronzie the display hardware process.
The active video logic determines the region where the pixels are outputed. vid_on is high when the counters are in the active display area whcich is defined by the HDISP and VDISP.
Assign statements are used to map row and col to the vcount and hcount. These outputs are used for pixel calculations for the other modules (ColourStripes.v).
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Sync2.png">

#### **VGAColourStripes.v Code**
The VGAColourStripes.v module generates RGB color patterns during the active video period to display on the screen based on column positions, while also updating RGB values using a state machine.

##### **Parameters**:
The following parameters configure how the counter behaves. **COUNTER_WIDTH** defines the bit width of the counter (32 bits). **COUNT_FROM** defines the initial value (0 bits) of the counter. **COUNT_TO** is the maximum count value of the counter (2^26 bits). **COUNT_RESET** is the reset value for the counter (2^27 bits).
The inputs include **clk** which is the clock signal driving the application, the **rst** signal to reset the RGB output, and **row** and **col** represent the coordinates of the current pixel processed. The outputs are red green and blue which are the RGB values for the current pixel at row and column.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/StripesParam.png">

##### **Generating Colour Stripes**:
In the always functoin, RGB values are assigned based on the col value which maps column ranges for specific colors. This is done using if else statements to assing different colors  based on the column number. Each time col becomes in a certain range, the red_next, green_next, and blue_next assign the correct color to the pixel. 
* 0 - 79: black
* 80 - 159: blue
* 160 - 239: green
* 240 - 319: cyan, which is blue + green
* 320 - 399: red
* 400 - 479: magents, which is red + blue
* 480 - 559: yellow, which is green + red
* 560 - 639: white, which is the combination of RGB. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Stripes1.png">

##### **RGB Output State Machine**:
To update RGB values, we implement a state machine. The red_reg, green_reg, and blue_reg (RGB values) are updated based on the next color value (red_next, green_next, blue_next). So the `always@*` above calculates the next RGB value based on column position. While `always@(posedge clk, posedge rst)` updates RGB values based on rising edge of the clock, and when reset signal is high it clears the color. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Stripes2.png">

### **ColourStripes Simulation**
Generating a simulation was successful. A simulation validates how the module operates ina VGA environment, ensuring that the color signals (red, green, blue) are generated for each column pixel accordingly. The major signals in the simulation were clk, rst, hsync, vsync, row, col, vid_on, and the color outputs RGB. The clock (_clk_) drives the system, each rising edge represents a VGA controller cycle, and with each rising edge it increments the col counter, as seen in the second simulation figure. The _rst_ signal initialises the system by clearing all outputs, it is then deasserted to allow VGA signal generation. The synchronization signals synchronize the display. _hsync_ is triggered at the end of each row to signal to next row. _vsync_ is triggered at the end of a frame signaling transition to the top left corner of the screen. _vid_on_ indictaes if the current pixel is on the visible screen (active display area), when it's high the RGBs are displayed. _col_ increments for each pixel in the row and resets when hsync is toggled. _row_ increments for each row and resets when vsync is toggled. RGB values are determined by col value to display the vertical stripes in the image. 

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Stripes2.png">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Stripes2.png">

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
