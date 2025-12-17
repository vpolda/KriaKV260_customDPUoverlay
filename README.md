# KriaKV260 Custom DPU Overlay

The goal of this project was to compare the performance of different Xilinx FPGA DPU architectures on a Kria KV260, while running a common CNN. 

## Description

The following two DPU architectures of the DPUCZDX8G were used: the b4096 and the b1600. The primary metrics gathered being inference time, class, and confidence per image.

Resnet50 was used as the CNN for comparision of the DPUs as there were plenty of examples and resources available. The images used are found under /Target/images.

## Results

## Setup

This section is not a complete tutorial, but instead includes highlights or key details. Please see the references section below for complete tutorials.

### Dependencies

Host
Linux - Will not work on WSL or without significant workarounds on Windows
Kria Platform and overlay base - https://github.com/Xilinx/kria-vitis-platforms
Vivado & Vitis **2023.1** - For building Kria Platforms and overlays
Serial port monitor - like Putty, minicom, Tera Term, etc

Target
KriaKV260 board - Running default Ubuntu image
Vitis AI **v2.5** - https://github.com/Xilinx/Vitis-AI.git

### Building

Host
Refer to the Host/kria-vitis-platforms folder and repo for how to build the overlay and associated platform. Then build the smaller_benchmark overlay. This is a variation of the kv260-benchmark-b4096 that instead implements a b1600 DPU. 

The only files that are needed by the Target are:
* Application bitstream converted to *.bit.bin format. ex. testapp.bit.bin
* Application bitstream device tree overlay *.dtbo. ex. testapp.dtbo
* If its a Vitis based PL design using XRT, a metadata file in .xclbin format. ex. testapp.xclbin
* shell.json with metadata about the PL design. ex. shell.json

The applicaiton bitstream and device tree overlay both need separate commands to change the generated files into their correct types. The shell.json is a simple file and can be copied from the other overlays in this repo. See the sources (x1) and (x2) below for more details.

The bitstream, device tree overlay, and the metadata file must have the same base name (different extensions of course). After this, these files need to be copied to the Kria and put under ```lib/firmware/xilinx/<app_name>```. 


Target
After copying the overlay files over from the step above, verify the custom application shows up via ```sudo xmutil listapps```. If you do not see it here, something is wrong with the generated files, their location, or the configuration.

To get the actual model file of a certain CNN, we recommend finding and downloading it via the Vitis AI Model Zoo. This gives an xmodel file along with which DPU architecture the model is compiled for, stored in the "meta.json" file for the model. We used one of the pytorch resnet50 model available in downloader.

TO DO: If the "target" in the json file does not match that of the overlay there will be a non-descriptive error! TBD

If the model does not match the overlay DPU, the model must be reconfigured via Vitis-AI and the appropiate Vitis AI compiler (ie. vai_c_xir, vai_c_tensorflow, or vai_c_tensorflow2). Pytorch xmodel files can be recompiled via vai_c_xir! While the tensorflow models must have a quantized model, either a .pb or .h5 file (x3).


Finally, the model executable that handles running the model, sending inputs, and reading outputs, must be built. These files can be grabbed from the Vitis AI/examples or the Vitis AI/src/Library. In either case, there will be a build.sh file that must be ran to generate these executables. In this case, we focused on building the ```examples/VART/resnet50```. 

### Executing on Target

Run the following on boot:
```
sudo xmutil unloadapp 
sudo xmutil loadapp kv260-benchmark-b4096 
sudo xdputil query 
```
"kv260-benchmark-b4096" Can be replaced with any of the apps listed by ```sudo xmutil listapps ```
Verify that ```sudo xdputil query ``` returns no error and the DPU fingerprint is somewhere in the data it outputs. 
The DPU target of the model you desire to run **MUST** match the architecture of the overlay, shown via query command above and in the meta.json file.

In this repo, the build files for the resnet50 executable have been modified to write the output data to a csv file along with the terminal.

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
* (x3) https://docs.amd.com/r/2.5-English/ug1414-vitis-ai/Compiling-for-DPU

Images from: https://image-net.org/

## Authors
Victoria Polda
