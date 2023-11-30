# PROBLEM STATEMENT 

(a.) As a first step, read images stored in a Block RAM into a buffer, and display it on a VGA monitor at different corners of the VGA after a certain delay. You should be able 
to find the VGA interfacing code. 

(b.) Apply image transformation techniques (these techniques can use the bit serial multiplier and bit parallel multiplier) to the captured image, display it on VGA

# INTRODUCTION

* The VGA (Video Graphics Array) signal is a widely used standard for transmitting video information using analog signals. It involves three primary color channels—Red, Green, and 
Blue (RGB)—to convey color information, along with Horizontal Sync (HSync) and Vertical Sync (VSync) signals for timing interpretation by monitors. The VGA connector, typically a 
DE-15 with 15 pins organized into three rows, plays a pivotal role in this process.
* The VGA connector's pinout includes RGB channels (pins 1-5), Ground (pin 6), HSync (pins 7-8), Ground (pin 9), and VSync, Ground, and Reserved (pins 10-15). Timing requirements are 
crucial for VGA signals, supporting resolutions like 640x480, 800x600, and 1024x768 pixels. Timing parameters include refresh rates, porch durations, and sync pulse widths.

![Picture1](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/6e16ce2b-cb08-454f-ba92-183d519f38f2)

## Generating VGA Signals in FPGA:
FPGAs are responsible for VGA signal generation. Hardware description languages like Verilog or VHDL describe the logic for VGA signal generation. 
Digital-to-analog converters (DACs) often convert RGB values from digital to analog.
* Synchronization Signal Generation:
Horizontal and vertical synchronization signals (HSync and VSync) mark the beginning of each line and frame, ensuring proper timing to prevent visual artifacts.
* Color Generation: 
RGB values, typically 8 bits per color channel, need conversion to analog voltages for the RGB channels. 
* Connection to VGA Monitor: 
Connecting VGA signals to the corresponding pins on the VGA connector is crucial. Adhering to VGA signal voltage levels ensures a secure electrical connection. 
* Programming and Configuration: 
Writing code, whether firmware for microcontrollers or HDL for FPGAs, is essential to generate VGA signals. Configuring microcontrollers or FPGAs to output signals
through designated pins is part of the process. 
* Testing and Debugging: 
Connecting the VGA interface to a monitor and using tools like oscilloscopes or logic analyzers helps debug timing-related issues. Code adjustments may be necessary
for fine-tuning parameters. 
* Display Content: 
Once the VGA interface functions correctly, graphics or text can be displayed on the monitor. RGB values can be updated to change colors, and synchronization signals ensure accurate 
content positioning. 
* Signal Integrity: 
Ensuring good signal integrity is critical to preventing issues like noise or distortion. This involves proper impedance matching and minimizing signal reflections. 
* Compliance with VGA Standards: 
Adhering to VGA standards is essential for compatibility with a wide range of monitors. Referring to VGA signal timing specifications and voltage levels is crucial.


## Steps for Executing Image through VGA

* VGA Interface Code: Develop Verilog code defining VGA timings and signals (h_sync,v_sync, red, green, and blue). 
* Image Format: Determine the image format, considering color depth and pixel mapping. Modern displays may support 24-bit or 32-bit color. 
* Color Depth & Pixel Mapping: Decide on color depth, use a Color Lookup Table (CLUT), and map image pixels to VGA pixels. 
* Image Loading in BlockRAM: Load images into BlockRAM, potentially using Memory Initialization Files (MIFs) for preloading. 
* Image Buffer: Create an image buffer (dual-port BRAM) to synchronize image data with VGA timing. 
* Display Control: Implement logic for controlling image transitions, position changes, and delays using counters and state machines. 
* Color Generation: Generate RGB values for each pixel based on image data and color depth. 
* VGA Signal Generation: Utilize VGA interface code to generate h_sync, v_sync, RGB color signals, and the pixel clock. 
* Positioning: Adjust h_sync and v_sync signals to change image positions on the VGA screen. 
* Timing and Delays: Control image transitions and delays using counters or timers. 
* Programming the FPGA: Program the synthesized bitstream onto the FPGA board. 
* Monitor Connection: Connect the FPGA board's VGA output to a VGA monitor or display device.

  
## Techniques Used for Image Transformation: 
* Bit-Serial Multiplier: Processes binary numbers one bit at a time, suitable for pixel-wise operations like brightness adjustment. 
* Bit-Parallel Multiplier: Performs parallel multiplication of binary numbers, efficient for complex image processing operations like convolution. 
* Integration with VGA: Integrating bit-serial or bit-parallel multipliers with VGA involves interfacing with VGA timing and color signal generation. Considerations include
  memory management, synchronization, timing, and testing for accurate image transformation and display it.

## VERILOG CODE OF VGA EXTRACTING RESULTS FROM BIT PARALLEL TECHNIQUE

```

`timescale 1ns / 1ps 
module vga_interfacing( 
clock,reset,mode_1,mode_2,mode_3,mode_4,val, // mode_1 to mode_4 used for all four 
corner images operations 
hsync,vsync, // hsync and vsync for the working of monitor 
red, green, blue // red, green and blue output pixels 
);
 input clock;
 input reset;
 input [7:0] val = 0; // intialize value to zero
 input mode_1;
 input mode_2;
 input mode_3;
 input mode_4; 
 reg[7:0] red_o, blue_o, green_o; // variables used during calcultion
 reg [15:0] r, b, g,r1,r2; // variables used during calcultion
 wire [3:0] result;
 wire [7:0] red_result;
 wire [7:0] blue_result;
 reg clk;
 initial begin 
 clk =0;
 end 
 always@(posedge clock) 
 begin 
 clk<=~clk;
 end 
 
 output reg hsync;
 output reg vsync;
 reg [7:0] red_pixel,green_pixel,blue_pixel;
output reg [3:0] red,green;
output reg [3:0] blue;
 integer i, j;
// reg [7:0] reg_value [0:7]; // 2D array to store intermediate results
 
reg read = 0;
reg [14:0] addra = 0;
reg [95:0] in1 = 0;
wire [95:0] out2;
 
 
image inst1( 
 .clka(clk), // input clka 
 .wea(read), // input [0 : 0] wea 
 .addra(addra), // input [14 : 0] addra 
 .dina(in1), // input [95 : 0] dina 
 .douta(out2) // output [95 : 0] douta 
);
 wire pixel_clk;
 reg pcount = 0;
 wire ec = (pcount == 0);
 always @ (posedge clk) pcount <= ~pcount;
 assign pixel_clk = ec;
 
 reg hblank=0,vblank=0;
 initial begin 
 hsync =0;
 vsync=0;
 end 
 reg [9:0] horizontal_counter=0; 
 reg [9:0] vertical_counter=0; 
 
 wire horizontal_syncon,horizontal_syncoff,hreset,horizontal_blankon;
 assign horizontal_blankon = ec & (horizontal_counter == 690); 
 assign horizontal_syncon = ec & (horizontal_counter == 721);
 assign horizontal_syncoff = ec & (horizontal_counter == 751);
 assign hreset = ec & (horizontal_counter == 799);
 wire blank = (vblank | (hblank & ~hreset)); 
 
 wire vsyncon,vsyncoff,vreset,vblankon;
 assign vblankon = hreset & (vertical_counter == 479); 
 assign vsyncon = hreset & (vertical_counter == 490);
 assign vsyncoff = hreset & (vertical_counter == 492);
 assign vreset = hreset & (vertical_counter == 523);
 always @(posedge clk) begin 
 horizontal_counter <= ec ? (hreset ? 0 : horizontal_counter + 1) : horizontal_counter;
 hblank <= hreset ? 0 : horizontal_blankon ? 1 : hblank;
 hsync <= horizontal_syncon ? 0 : horizontal_syncoff ? 1 : hsync; 
 
 vertical_counter <= hreset ? (vreset ? 0 : vertical_counter + 1) : vertical_counter;
 vblank <= vreset ? 0 : vblankon ? 1 : vblank;
 vsync <= vsyncon ? 0 : vsyncoff ? 1 : vsync;
end 
(*DONT_OPTIMIZE = "YES"*) 
bit_parallel_multiplier multiplier_inst ( 
 .red(red_pixel), 
 .green(green_pixel), 
 .blue(blue_pixel), 
 //.b(val), // Assuming val is the multiplier 
 .result(result) 
 );
 
 red_filter red_inst( 
 .clk(pixel_clk), 
 .red_color(red_pixel), 
 .blue_color(blue_pixel), 
 .value(val), 
 .red_val(red_result), 
 .blue_val(blue_result) 
 ); 
 
 always @ (posedge pixel_clk) 
 begin 
 blue_pixel = {out2[23], out2[22], out2[21], out2[20], out2[19], out2[18], out2[17], 
out2[16]};
 green_pixel = {out2[15], out2[14], out2[13], out2[12], out2[11], out2[10], out2[9], 
out2[8]};
 red_pixel = {out2[7], out2[6], out2[5], out2[4], out2[3], out2[2], out2[1], out2[0]};
 
 
 if(blank == 0 && (horizontal_counter >= 0 && horizontal_counter < 160) && 
(vertical_counter >= 0 && vertical_counter < 115)) begin 
 begin 
 
// RGB image to gray scale image 
 if(mode_1 == 1'b0)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 
 red_o = (red_pixel >> 2) + (red_pixel >> 5) + (green_pixel >> 1) + 
(green_pixel >> 4)+ (blue_pixel >> 4) + (blue_pixel >> 5);
 green_o = (red_pixel >> 2) + (red_pixel >> 5) + (green_pixel >> 1) + 
(green_pixel >> 4)+ (blue_pixel >> 4) + (blue_pixel >> 5);
 blue_o = (red_pixel >> 2) + (red_pixel >> 5) + (green_pixel >> 1) + 
(green_pixel >> 4)+ (blue_pixel >> 4) + (blue_pixel >> 5);
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {result[3],result[2], result[1], result[0]};
 green = {result[3],result[2], result[1], result[0]};
 blue = {result[3],result[2], result[1], result[0]};
 
end 
 
 // Increase brightness 
 end 
 else if(mode_1 == 1'b1)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 
 red_o = red_pixel;
 green_o = green_pixel;
 blue_o = blue_pixel;
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 end 
 end 
 
 if(addra <18399) 
 addra = addra + 1;
 else 
 addra = 0;
 end 
 end 
 else if ( horizontal_counter >= 0 && horizontal_counter < 160 && vertical_counter 
>= 364 && vertical_counter < 479) 
 begin 
// RGB image to gray scale image 
 if(mode_2 == 1'b0)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 red_o = red_pixel; 
 green_o = green_pixel;
 blue_o = blue_pixel;
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]}; 
 
 end 
 
 // Increase brightness 
 end 
 
 else if(mode_2 == 1'b1)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 
 red_o = red_pixel;
 green_o = green_pixel;
 blue_o = blue_pixel;
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 end 
 end 
 
 
 if(addra <18399) 
 addra = addra + 1;
 else 
 addra = 0;
 end 
 
 
 else if (horizontal_counter >= 540 && horizontal_counter < 700 && vertical_counter 
>= 0 && vertical_counter < 115) 
 begin 
 
// RED COLOUR FILTER 
 if(mode_3 == 1'b0)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 r2 <= blue_result;
 if(r2 >100)begin 
 blue_o = 0;
 end else begin 
 blue_o = r2/16;
 end 
 
 red_o = red_pixel/16;
 green_o = green_pixel/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 
 end 
 
 // Increase brightness 
 end 
 
 else if(mode_3 == 1'b1)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 
 red_o = red_pixel;
 green_o = green_pixel;
 blue_o = blue_pixel;
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 end 
 end 
 
 
 if(addra <18399) 
 addra = addra + 1;
 else 
 addra = 0;
 end 
 
 else if ( horizontal_counter >= 540 && horizontal_counter < 700 && vertical_counter 
>= 364 && vertical_counter < 479) 
 begin 
 
// RGB image to gray scale image 
 if(mode_4 == 1'b0)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 r1 = red_result;
 
 if(r1 >100)begin 
 red_o = 0;
 end else begin 
 red_o = red_pixel/16;
 end 
 
 blue_o = blue_pixel/16;
 green_o = green_pixel/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 
 end 
 
 
 end 
 
 else if(mode_4 == 1'b1)begin 
 
 if(reset) begin 
 red = 0;
 green = 0;
 blue = 0;
 end else begin 
 
 red_o = red_pixel;
 green_o = green_pixel;
 blue_o = blue_pixel;
 
 red_o = red_o/16;
 blue_o = blue_o/16;
 green_o = green_o/16;
 red = {red_o[3],red_o[2], red_o[1], red_o[0]};
 green = {green_o[3],green_o[2], green_o[1], green_o[0]};
 blue = {blue_o[3],blue_o[2], blue_o[1], blue_o[0]};
 end 
 end 
 
 
 if(addra <18399) 
 addra = addra + 1;
 else 
 addra = 0; 
 
 end 
  
 else 
 begin 
 
 red = 0;
 green = 0;
 blue = 0;
 
 end 
  
 end 
  endmodule

```

# PARALLEL BIT MULTIPLIER VERILOG CODE

## Submodule1: Edge Detection at Threshold=200
  
```
module bit_parallel_multiplier( 
 input [7:0] red,green,blue, 
 output [3:0] result 
);
 wire [7:0] result_red;
 // Bit-wise AND operation 
 assign result_red[4] = (red[7]*blue[7]*green[7]);
 assign result_red[3] = (red[6] * blue[6] * green[6]);
 assign result_red[2] = (red[5] * blue[5] * green[5]);
 assign result_red[1] = (red[4] * blue[4] * green[4]);
 
 
 
 assign result = (result_red);
 // Concatenate the results 
 // assign result = {result[0], result[1], and_result[2], and_result[3]};
endmodule
```

##  Submodule2: Red Filter and Blue Filter Verilog Code
  
```
module red_filter( 
input clk, 
 input [7:0] red_color, 
 input [7:0] blue_color, 
 input [7:0] value, 
 output reg [7:0] red_val, 
 output reg [7:0] blue_val 
);
 integer i, j;
 reg [7:0] reg_value [0:7]; // 2D array to store intermediate results
 reg [7:0] reg_blue_value [0:7];
 always @(posedge clk) begin 
 // Initialize red_val and reg_value 
 red_val = 8'b0;
 for (i = 0; i < 8; i = i + 1) begin 
 reg_value[i] = 8'b0;
 end 
 // Perform row-wise multiplication and accumulate the results 
 for (i = 0; i < 8; i = i + 1) begin
 reg_value[i][0] = (red_color[0] * value[i]);
 reg_value[i][1] = (red_color[1] * value[i]);
 reg_value[i][2] = (red_color[2] * value[i]);
 reg_value[i][3] = (red_color[3] * value[i]);
 reg_value[i][4] = (red_color[4] * value[i]);
 reg_value[i][5] = (red_color[5] * value[i]);
 reg_value[i][6] = (red_color[6] * value[i]);
 reg_value[i][7] = (red_color[7] * value[i]);
 
 reg_blue_value[i][0] = (blue_color[0] * value[i]);
 reg_blue_value[i][1] = (blue_color[1] * value[i]);
 reg_blue_value[i][2] = (blue_color[2] * value[i]);
 reg_blue_value[i][3] = (blue_color[3] * value[i]);
 reg_blue_value[i][4] = (blue_color[4] * value[i]);
 reg_blue_value[i][5] = (blue_color[5] * value[i]);
 reg_blue_value[i][6] = (blue_color[6] * value[i]);
 reg_blue_value[i][7] = (blue_color[7] * value[i]);
 
 end 
 // Sum the intermediate results to obtain the final red_val 
 for (i = 0; i < 8; i = i + 1) begin
 red_val = red_val + reg_value[i];
 blue_val = blue_val + reg_blue_value[i];
 end 
 end 
endmodule
```

## CONSTRAINT FILE

```
# Clock signal 
set_property PACKAGE_PIN W5 [get_ports clock] 
set_property IOSTANDARD LVCMOS33 [get_ports clock] 
create_clock -period 10.000 -name sys_clock_pin -waveform {0.000 5.000} -add [get_ports 
clock] 
## Switches 
## Switches 
## sel_modules 
#####
set_property PACKAGE_PIN V17 [get_ports reset] 
set_property IOSTANDARD LVCMOS33 [get_ports reset] 
#set_property PACKAGE_PIN V15 [get_ports {B[1]}] 
#set_property IOSTANDARD LVCMOS33 [get_ports {B[1]}] 
set_property PACKAGE_PIN W14 [get_ports {val[0]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[0]}] 
set_property PACKAGE_PIN W13 [get_ports {val[1]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[1]}] 
set_property PACKAGE_PIN V2 [get_ports {val[2]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[2]}] 
set_property PACKAGE_PIN T3 [get_ports {val[3]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[3]}] 
set_property PACKAGE_PIN T2 [get_ports {val[4]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[4]}] 
set_property PACKAGE_PIN R3 [get_ports {val[5]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[5]}] 
set_property PACKAGE_PIN W2 [get_ports {val[6]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[6]}] 
set_property PACKAGE_PIN U1 [get_ports {val[7]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {val[7]}] 
##VGA Connector 
set_property PACKAGE_PIN G19 [get_ports {red[0]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {red[0]}] 
set_property PACKAGE_PIN H19 [get_ports {red[1]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {red[1]}] 
set_property PACKAGE_PIN J19 [get_ports {red[2]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {red[2]}] 
set_property PACKAGE_PIN N19 [get_ports {red[3]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {red[3]}] 
set_property PACKAGE_PIN N18 [get_ports {blue[0]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {blue[0]}] 
set_property PACKAGE_PIN L18 [get_ports {blue[1]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {blue[1]}] 
set_property PACKAGE_PIN K18 [get_ports {blue[2]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {blue[2]}] 
set_property PACKAGE_PIN J18 [get_ports {blue[3]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {blue[3]}] 
set_property PACKAGE_PIN J17 [get_ports {green[0]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {green[0]}] 
set_property PACKAGE_PIN H17 [get_ports {green[1]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {green[1]}] 
set_property PACKAGE_PIN G17 [get_ports {green[2]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {green[2]}] 
set_property PACKAGE_PIN D17 [get_ports {green[3]}] 
set_property IOSTANDARD LVCMOS33 [get_ports {green[3]}] 
set_property PACKAGE_PIN P19 [get_ports horizontal_sync] 
set_property IOSTANDARD LVCMOS33 [get_ports horizontal_sync] 
set_property PACKAGE_PIN R19 [get_ports vertical_sync] 
set_property IOSTANDARD LVCMOS33 [get_ports vertical_sync] 
#set_property IOSTANDARD LVCMOS33 [get_ports QspiCSn] 
set_property BITSTREAM.GENERAL.COMPRESS TRUE [current_design] 
set_property BITSTREAM.CONFIG.CONFIGRATE 33 [current_design] 
set_property CONFIG_MODE SPIx4 [current_design] 
set_property PACKAGE_PIN P19 [get_ports hsync] 
set_property PACKAGE_PIN V16 [get_ports mode_1] 
set_property IOSTANDARD LVCMOS33 [get_ports hsync] 
set_property IOSTANDARD LVCMOS33 [get_ports mode_1] 
set_property IOSTANDARD LVCMOS33 [get_ports mode_2] 
set_property IOSTANDARD LVCMOS33 [get_ports mode_3] 
set_property PACKAGE_PIN W16 [get_ports mode_2] 
set_property PACKAGE_PIN W17 [get_ports mode_3] 
set_property PACKAGE_PIN W15 [get_ports mode_4] 
set_property IOSTANDARD LVCMOS33 [get_ports mode_4] 
set_property IOSTANDARD LVCMOS33 [get_ports vsync] 
set_property PACKAGE_PIN R19 [get_ports vsync] 
set_property IOSTANDARD LVCMOS33 [get_ports mode] 
set_property PACKAGE_PIN R2 [get_ports mode]
```

## Post Implementation Schematic

## Power Utilisation

## Resource Utilisation

## Timing Report

## Critical Path

## Floorplan


## Image Output Explanation

The Verilog module vga_syncIndex orchestrates a complex yet versatile image display on a VGA monitor, creating a visually captivating experience for the viewer.
This implementation employs a block RAM module (image module) to store and retrieve pixel data, contributing to the dynamic nature of the displayed images. The VGA screen is 
intelligently divided into four distinct corners, each corner offering a unique visual presentation based on the selected mode. 

Starting with the top-left corner, the module presents edge detection version of the image. Upon changing the mode to 1, the corner seamlessly transitions to showcasing the original 
pixel image, providing viewers with the option to appreciate both the artistic and technical aspects of the image. 
Moving to the top-right corner, the visual experience takes an artistic turn. In mode 0, the image is displayed with a blue color filter applied, creating a distinct and visually 
appealing effect. Switching to mode 1 restores the top-right corner to display the unaltered, original image, allowing users to toggle between artistic interpretations and 
true representations. The bottom-left corner introduces an intriguing dynamic. In mode 0, viewers observe the unfiltered original image, while
switching to mode 1 transforms the corner into edge detection rendition, adding a layer of sophistication to the visual presentation. 

The bottom-right corner incorporates a red color filter in mode 0, infusing warmth and artistic expression into the image. Changing to mode 1 in this corner restores the display to 
the original, unfiltered image, providing a balance between artistic interpretation and faithful representation. 

Throughout this visual journey, the continuous retrieval and display of pixel data from the block RAM contribute to the fluidity of the image presentation. The module not only 
demonstrates technical proficiency in interfacing with VGA signals but also offers a creative platform for experimenting with various image processing effects. Users can 
explore diverse visual interpretations at each corner of the VGA display, making this module a powerful tool for both technical and artistic exploration.



# Output Images By Bit Parallel Multiplication Techniques

## Original Image on Bottom-Right Corner of Screen

![right_bottom_original_imag](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/dce6f2f8-0814-4992-ba1f-7278cd2068d8)

## Red Filtered Transition Image on Bottom-Right Corner of Screen

![right_bottom_corner](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/5c2cbb27-d8a2-4d0d-a3da-ebafacafd3db)

## Original Image on Top-Right Corner of Screen

![right_top_original_tansistion image](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/4a32dd89-c484-4a22-8912-450877b2c9fe)

## Blue Filtered Transition Image on Top-Right Corner of Screen

![_right_top_image](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/d851f2ed-1fbb-4539-8c5b-35ee26cf12d6)

## Displayed Original Image on Top-Left Corner of Screen

![original_image](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/f68218e4-c5b1-4f5f-bec9-f861511df961)

## Edge Detection Transition Image on Bottom-Left Corner of Screen

![edge_detection_left_bottom](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/eb5f8efe-6be5-47ea-b584-4b9116089219)

## Complete Final Output Image

![complete_final_images](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/b40d916b-6ef9-44bd-a134-cad4350d19d4)

 

# Simulation Output Video


## Result of VGA Interfacing using Bit Serial Multiplication Technique 

* Bit Serial Multiplier Power Report

![serial_POWER](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/b700b2bb-4b1a-43c5-9fca-178479b1cba9)

* Bit Serial Multiplier Schematics
  
![seria_schematics](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/e2ac98ba-c7cc-4024-b193-2459f2c58050)
  
* Bit Serial Multiplier Timing, Floorplan, Leaf Cells
  
![serial_Timing_LeAF_Floorplan](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/d6dcc838-301e-4e58-ba7a-b782e751c6bf)

* Bit Serial Multiplier Resource Utilisation

![SERIAL_LUTS](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/3b044f19-03e2-46a7-8d87-0067720ec241)


# Result Comparision Table 

| Parameters                    | Serial Bit Multiplier | Parallel Bit Multiplier  |
|------------------------------ |-----------------------|--------------------------|
| Total On-Chip Power (mW)      | 130                   | 76                       |
| Worst Negative Slack (ns)     | 8.855                 | 6.74                     |
| Worst Negative Hold Slack (ps)| 97                    | 1051                     |
| Number of LUTs used           | 1969                  | 222                      |
| LEAF CELL used                | 950                   | 206                      |

![original_image](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/7fa31642-74e2-4377-b1a4-0039bda2d8db)

![comparision_chart_fpga](https://github.com/SolankiPratikkumar/FPGA_PROJECT_VGA_BIT_PARALLEL_MULTIPLIER/assets/140999250/2edf7115-291d-4493-ab78-14936a700c9f)



# Acknowledgement

* Dr. Nanditha Rao, Course Professor
* Jay Shah, TA for Project
* Akhil Asati, Partner for this Project
* Nancy and Shivangi, IIITB Colleagues provided Bit_Serial_Technique_Results

# References

* https://www.researchgate.net/profile/Sivanantham-S/publication/286921324_Image_edge_detection_in_FPGA/links/566fa69b08ae486986b71137/Image-edge-detection-in-FPGA.pdf?origin=publication_detail&_tp=eyJjb250ZXh0Ijp7ImZpcnN0UGFnZSI6InB1YmxpY2F0aW9uIiwicGFnZSI6InB1YmxpY2F0aW9uRG93bmxvYWQiLCJwcmV2aW91c1BhZ2UiOiJwdWJsaWNhdGlvbiJ9fQ
* https://stackoverflow.com/questions/
* https://github.com/Shubhayu-Das/VL504-project#video-over-vga-using-the-ov7670
* https://embeddedthoughts.com/2016/07/29/driving-a-vga-monitor-using-an-fpga/
* https://github.com/FPGADude/Digital-Design/tree/main/FPGA%20Projects/VGA%20Projects/VGA%20Bouncing%20Square
  

