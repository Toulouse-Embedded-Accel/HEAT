# 🛠️ Vivado/Vitis Installer Freeze

**Author:** Julien Posso (ONERA)

**Date:** 2026-03

**Tags:** `Vivado` `Vitis` `Installation` `Linux` `Troubleshooting`

## 🎯 The Goal
Successfully install the AMD Unified FPGAs & Adaptive SoCs Installer (Vivado/Vitis 2025.2) on a modern Linux distribution.

## ⚙️ Environment
* **Hardware:** Host PC
* **OS:** Ubuntu 22.04
* **Toolchain:** Vitis & Vivado 2025.2

## 💥 The Problem
At the end of the installation process, after all files have been successfully copied to the target directory (e.g. `/opt/Xilinx`), the GUI installer freezes indefinitely. The progress bar halts on the message: "Final Processing... Generating installed device list".

!!! failure "The Root Cause"
    The installer attempts to run a TCL script (`xlpartinfo.tcl`) in the background to index the supported FPGA/SoC hardware. However, the underlying `vivado` unwrapped binary is statically linked against legacy `ncurses` libraries. Modern Linux distributions have migrated to version 6, stripping the legacy version. The binary crashes silently, leaving the parent GUI process waiting for an exit code that never comes.

Manual execution of Vivado in batch mode to run the underlying TCL script reveals the silent failure:
```bash
$ /opt/Xilinx/2025.2/Vivado/bin/vivado -nolog -nojournal -mode batch -source /opt/Xilinx/2025.2/Vivado/scripts/sysgen/tcl/xlpartinfo.tcl -tclargs /opt/Xilinx/2025.2/Vivado/data/parts/installed_devices.txt

application-specific initialization failed: couldn't load file "libxv_commontasks.so": libtinfo.so.5: cannot open shared object file: No such file or directory
```

## 💡 The Workaround / Solution
Since the installer was interrupted before generating crucial environment scripts (like `settings64.sh`) and system registry keys, you cannot keep this broken installation. You must resolve the dependency issue at the OS level, clean up the corrupted directory, and run the installer again.

!!! success "The Fix: Install missing libraries and reinstall"
    **1. Kill the frozen installer**
    Forcefully close the GUI or kill the underlying Java process to unlock your terminal.
    ```bash
    # WARNING: This kills all Java apps (e.g., Eclipse, IntelliJ) running in your session
    killall -9 java
    ```

    **2. Install the missing legacy libraries**
    Update your system and install the required `ncurses` packages.
    ```bash
    sudo apt update
    sudo apt install libtinfo5 libncurses5
    ```
    Alternative: if you are using the Single-File Download (SFD), you can use the AMD script (2023.2 and above).
    ```bash
    cd <untarred_location>/FPGAs_AdaptiveSoCs_Unified_202x.x_*/
    sudo ./installLibs.sh
    ```

    **3. Purge the incomplete installation**
    Wipe the corrupted files so you can start fresh.
    ```bash
    # Example if installed in /opt
    sudo rm -rf /opt/Xilinx/2025.2
    ```

    **4. Re-install**
    [Link to the HEAT Quickstart Guide](../quickstarts/vitis-toolchain-setup.md)

## 🔗 References

* [Official documentation page](https://adaptivesupport.amd.com/s/article/76616)
