# Applespi Fix for Ubuntu Kernels

This repository contains modifications to the Applespi driver that fix compilation errors on newer Ubuntu kernels (tested on Ubuntu 24.04.2 LTS with Linux kernel 6.8.0-41-generic). This fix is useful for users running Ubuntu on older Apple hardware (e.g., MacBook8,1) who require working keyboard, touchpad and other Apple-specific input devices and wish to upgrade Ubuntu. This fix has not been tested on other devices of similar vintage but seems to be working OK. 

Note: the trackpad was not initially recognised on reboot (keyboard ok), though the file has worked fine since. I haven't been able to replicate the problem.

## Problem Description

When upgrading Ubuntu, the Applespi driver fails to compile due to several kernel API changes. Some of the errors encountered include:

- Removal of the `delay_usecs` field in `struct spi_transfer`.
- Incomplete or undefined structures (e.g. `struct efi_variable`).
- Mismatched function prototypes and type conflicts (e.g. with `appleib_detach_and_free_hid_driver` and the HID driver structure).

Example error messages:

- `error: invalid use of undefined type ‘struct efi_variable’`
- `conflicting types for ‘appleib_detach_and_free_hid_driver’`
- `error: ‘drv_info’ undeclared`

## Solution Overview

The fix applies the following changes:

- **applespi.c**  
  - Replaces usage of the incomplete `struct efi_variable` with a fixed-size buffer or generic allocation.
  - Comments out unused variables (such as `efi_var_data`) and initialises variables (like `sts` in `applespi_save_bl_level`).
- **apple-ibridge.c**  
  - Corrects the function prototype and definition of `appleib_detach_and_free_hid_driver` so that both use the same type (`struct apple_hid_drv_info`) and parameter name.
  - Provides a minimal definition for `struct apple_hid_drv_info`
  - Fixes list iteration calls to pass actual variables instead of type names.

## Build and Installation Instructions

This fix is distributed as source code intended for use with DKMS. To build and install the module:

1. **Clone the Repository:**
   
   ```bash
   git clone https://github.com/yourusername/applespi-fix-ubuntu.git
   cd applespi-fix-ubuntu
   ```

2. **Copy the Source Files:**
   
   Place the modified source files (contained in the `src/` folder) to your DKMS source directory. For example:
   
   ```
   sudo cp -r src /usr/src/applespi-0.1
   ```

3. **Add the Module to DKMS:**    
   
   ```
   sudo dkms add -m applespi -v 0.1
   ```

4. **Build the Module:**
   
   ```
   sudo dkms install -m applespi -v 0.1
   ```
   
   And then reboot

## Legal and Licensing

This project is based on the Applespi driver, which is licensed under the GNU General Public License version 2 (GPLv2). All modifications in this repository are distributed under the GPLv2. Please see the LICENSE file for the full text.

**Disclaimer:**  
I am not a lawyer. This repository is provided "as-is" without any warranty. Ensure that your use of this code complies with the original driver's licensing terms and your local laws.

## Acknowledgments

- The original Applespi driver authors.
