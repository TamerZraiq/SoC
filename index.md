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

#### **Parameters**:
The following parameters specify the timing values required for VGA synchronization. HDISP and VDISP define the resolution of the display area. HFP/VFP, HPW/VPW, and HLIM/VLIM represent front porch, pulse width, and total line/frame limits.

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/SyncParam.png">

#### **Horizontal and Vertical Counter**:
The following code increments hcount and vcount in an always block to track pixel positions in horizontal and vertical frames. The counter goes through the total line(HLIM)/frame limits(VLIM). hcount resets after reaching the horizontal limit, then triggers vcount to increment vertical synchronization. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Sync1.png">

#### **Sync Signal Generation and Active Video Logic**:
hsync signal is active low during the horizontal pulse width, while vsync is active low during vertical pulse width. These signals represent the sync intervals for the display. The following code is used to synchronzie the display hardware process.
The active video logic determines the region where the pixels are outputed. vid_on is high when the counters are in the active display area whcich is defined by the HDISP and VDISP.
Assign statements are used to map row and col to the vcount and hcount. These outputs are used for pixel calculations for the other modules (ColourStripes.v).

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Sync2.png">

#### **VGAColourStripes.v Code**
The VGAColourStripes.v module generates RGB color patterns during the active video period to display on the screen based on column positions, while also updating RGB values using a state machine.

#### **Parameters**:
The following parameters configure how the counter behaves. **COUNTER_WIDTH** defines the bit width of the counter (32 bits). **COUNT_FROM** defines the initial value (0 bits) of the counter. **COUNT_TO** is the maximum count value of the counter (2^26 bits). **COUNT_RESET** is the reset value for the counter (2^27 bits).
The inputs include **clk** which is the clock signal driving the application, the **rst** signal to reset the RGB output, and **row** and **col** represent the coordinates of the current pixel processed. The outputs are red green and blue which are the RGB values for the current pixel at row and column.

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/StripesParam.png">

#### **Generating Colour Stripes**:
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

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/simStrip1.png">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/sim1Strip1.png">

### **Synthesis**
Synthesis operation is used to reflect the code into a hardware implementation. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/OGsynth.png">

The diagram shows the connection between components of the design. It represents the entire signal generation pipeline. The clk_wiz_0 is a clock generation component that adjusts the clock signal to the required frequency (25MHz). The VGASync component generates hsync and vsync signals to control synchronization signals. Each of the VGA RGB colors are connected to 4 bit outut buffers.

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/synthsch.png">

### **Implementation**
The schematic below represents the process for generating a VGA signal with color outputs and synchronization. The clk_wiz_0 takes an input clock signal (clk) and outputs 25 MHz clock. The VGA Sync module uses the 25 MHz clock and reset signal to generate hsync and vsync. The ColourStripes module produces the RGB values.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/OGschematic.png">

### **Demonstration**
The final result is a generated VGA signal that displays vertical coloured lines on the screen.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/StrResult.jpg">

## **My VGA Design Edit**
The brainstorming process consisted of trying to find a fitting project that would incorprate what I learned in the labs as well as trying to understand new concepts. During the 4 sessions for this project I have tried multiple designs to understand how to manipulate the design. 
My first attempt was maniulating rows and columns to get smaller squares with different colors. However, I couldnt fully comprehend how to control rows and columns.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att1.jpg">

My second attempt was more successful. I used chatgpt to explain certain concets such as a moving design. I was able to maniulate square size as well as adding a vertical line that would move left to right. Every cycle the line does it would change colors and the color of the squares.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att2.1.jpg">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att2.2.jpg">

My third attempt consisted of trying a similar design to the start screen design. It proved more challenging and with little time I was not able to fulfil what I wanted to do. 
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att3.jpg">

On the final session I settled on building on what I had in the second attempt, which is my final VGA design seen in the demonstration section below.

### **Code Adaptation**
I combined both templates (color cycle and color stripes) and with the help of chatgt I was able to manipulate the code so that I control square size as well as changing color speed. 
The counter is used to determine the timing of color changes. The counter[30:25] is used to speed up or slow down color changes by changing the bits of the counter. If I used counter[15:10], it uses lower bits so the color changes will be faster because they toggle more frequently. This applies vice versa for slower color change time. 
So the following `always @*` block uses case statement to assign colors and change the colors (as a cycle) based on their position (square_row, square_col) and color cycle. The %6 is to make sure that all 6 colors are displayed. If I keep all the other variables as they were and change %6 to %10, some squares will be black (default) since theres only 6 colors. If it becomes less than the number of colors present (%4) it will dislay only 4 colors since the colors are already predefined.
The RGB values registration and VGAsync is the same as the given templates. The change in VGATop is adding the row and col parameters for my ColourStripes module (I did not change the name).

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/counter1.png">

To manipulate square size, I divided row and col values according to the square size I desired.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/square1.1.png">

* For bigger squares, I divided the rows and col by a bigger number (Examle 60x60 ixels).
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.2.jpg">

* For smaller squares, I implemented more divisions thus a smaller number (Examle: divide by 10).
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.1.jpg">

* The default square size divider I settled on was divide by 20.
  The design I created changes colors deending on the counter values, in my case I chose counter[30:25].
  <img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.3.1.jpg">
  <img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.3.2.jpg">

### **Project Summary**
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/ps.png">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/Schematic.png">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/colourStripesSch.png">

### **Simulation**
Similar to the simulation process described in the Given Template section, the clock drives the logic while reset intializes the counters and register, pixels are put into squares based on row and col divisions, hsync and vsync control screen refresh and timing while the RGB values are displayed for current pixel. Unlike the ColourStries module where the vertical colors are stationary, in this case they are changing. As seen in the simulation figure below, the RGB outputs are 4 bit outputs that define the color for each pixel at their respective time. The RGB are changing periodically depending on the counter[30:25], the RGB signals remain constant for some duration before toggling a new value. This changes whenever the counter value is changed for slower or faster times. 

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/MySim.png">

### **Synthesis**
The synthesis schematic shows the components used to generate the VGA signals to display and to manage clock signals. The colorstripes module takes clk and rst as inputs for synchronization and outputs generated RGB signals. hsync and vsync similarly to the Synthesis section above; used as synchronization signals and outputs the vid_on to enable display. hcount and vcount are used to control the scan lines.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/MySynth.png">

The following visualizes the layout of logic blocks and routing on the FPGA after synthesis and implementation. The labels X0Y2, X1Y2, X0Y1, X1Y1, X0Y0, X1Y0 are FPGA regions where the logic is. The colored blocks are the modules in these regions.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/MySynth2.png">

### **Demonstration**
My final design are squares moving across the screen by changing colors at defined speeds and square size.
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.3.1.jpg">
<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/att4.3.2.jpg">

## **Resources Used**
* ChatGPT for clarification and code modifications.
* [What is VGA? A Comprehensive Guide to Video Graphics Array](https://www.hp.com/us-en/shop/tech-takes/what-is-vga-comprehensive-guide-video-graphics-array)
* [VGA Basics](https://faculty-web.msoe.edu/johnsontimoj/EE3921/files3921/vga_basics.pdf)
* M. Lynch, “System On Chip Design and Verification”, Labs, ATU, Galway, 2024.
* [Case Statement - Nandland](https://nandland.com/case-statement-2/)
