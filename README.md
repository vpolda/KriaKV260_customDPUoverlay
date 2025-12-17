# KriaKV260 Custom DPU Overlay

The goal of this project was to compare the performance of different Xilinx FPGA DPU architectures while running a CNN. 

## Description

Resnet50 was used as the CNN for comparision of the DPUs. The images used are found under /Target/images.

The following two DPU architectures of the DPUCZDX8G were compared: the b4096 and the b1600. The primary metrics gathered being inference time, classes and their respective confidences, and ...

## Getting Started

### Dependencies

Host
Linux - not WSL, full distro!
Kria Platform and overlay base - https://github.com/Xilinx/kria-vitis-platforms
Vivado & Vitis 2023.1 - For building Kria Platforms and overlays

Target
KriaKV260 - Running default Ubuntu image
Vitis AI v2.5

### Installing 

Host


Target


### Building

Host
Refer to the Host/kria-vitis-platforms folder and repo for how to build the overlay and associated platform. The dpu_conf.vh file describes the DPU configuration used in the overlay when it is generated.
Build the smaller_benchmark overlay. 

The only files that are needed by the Target are:
* Application bitstream converted to *.bit.bin format. ex. testapp.bit.bin
* Application bitstream device tree overlay *.dtbo. ex. testapp.dtbo
* If its a Vitis based PL design using XRT, a metadata file in .xclbin format. ex. testapp.xclbin
* shell.json with metadata about the PL design. ex. shell.json

The applicaiton bitstream and device tree overlay both need separate commands to change the generate files into their correct types. The shell.json is a simple file and can be copied from the other overlays in this repo. See the sources (x1) and (x2) below for more details.

The bitstream, device tree overlay, and the metadata file must have the same base name (different extensions of course). After this, these files need to be copied to the Kria and put under ```lib/firmware/xilinx/<app_name>```. 


Target
After copying the overlay files over from the step above, verify the custom application shows up via ```sudo xmutil listapps```. If you do not see it here, something is wrong with the generated files, their location, or the configuration.

To get the actual model file

Building executable

### Executing program

Host

Target
Run the following on boot:
```
sudo xmutil unloadapp 
sudo xmutil loadapp kv260-benchmark-b4096 
sudo xdputil query 
```
"kv260-benchmark-b4096" Can be replaced with any of the apps listed by ```sudo xmutil listapps ```
Verify that ```sudo xdputil query ``` returns no error and the DPU fingerprint is somewhere in the data it outputs.

## Help

Highly recommend sticking with the versions of software listed here. Different versions of Vivado or Vitis can brick the Kria Build Platform and overlay. 2023.1 was the only version reported to work on Github. 
Vitis AI v3.5 is very different from v2.5, and these files here will not work with v3.5

## References

Vitis AI v2.5 and KriaKV260 tutorials, really good tutorials
* https://www.daiphys.com/portal/fpga/xilinx/tools/vitis-ai.html 
* https://www.daiphys.com/portal/fpga/xilinx/board/kv260.html 

Xilinx docs
* (x1) https://xilinx.github.io/kria-apps-docs/creating_applications/2022.1/build/html/docs/target.html
* (x2) https://xilinx.github.io/kria-apps-docs/creating_applications/2022.1/build/html/docs/dtsi_dtbo_generation.html

Images from: https://image-net.org/

## Authors
Victoria Polda
