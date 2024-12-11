---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---
Tamer Zraiq

The following document consists of defining the process of designing and implementing a VGA driver on an FPGA. The project demonstrates some of the concepts of digital design using Verilog language and the Vivado environment. Starting with provided templates (colour stripes and colour cycle), I modified these templates to enhance the design, including simulation and synthesis to ensure functionality. The document outlines the process of how a VGA controller operates and how to implement various designs.


## **Template VGA Design**
### **Project Set-Up**
The project is set up in Vivado to use the Artix-7 FPGA (xc7a35tcpg236-1). VGATop is the top module that integrates various submodules for clocking, VGA synchronization to manage synchronization signals and the ColourStripes module for generating dynamic color patterns. The design sources include VGATop.v, VGASync.v, and VGAColourStripes.v, while the simulation source has a testbench module. The synthesis, simulation and implementation results confirm that the bitstream was successfully generated, as shown in the Project Summary screenshot.

<img src="https://raw.githubusercontent.com/TamerZraiq/Soc/main/docs/assets/images/projectsummary.png">


### **Template Code**
Outline the structure and design of the Verilog code templates you were given. What do they do? Include reference to how a VGA interface works. Guideline: 2/3 short paragraphs, consider including screenshot(s).
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
