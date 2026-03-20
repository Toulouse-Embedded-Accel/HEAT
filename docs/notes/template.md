# 🛠️ Vivado/Vitis Installer Freeze

**Author:** Julien Posso (ONERA)

**Date:** 2026-03

**Tags:** `Vivado` `Installation` `Linux` `libtinfo5` `Troubleshooting`

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
Do not reinstall or delete the downloaded files. The physical installation is already complete. You simply need to kill the zombie process to unblock the GUI, and install the missing legacy dependencies on your host OS.

!!! success "The Fix: Install the missing dependencies"
    1. Identify and kill the frozen unwrapped `vivado` process. This will force the GUI to gracefully finish the installation sequence.
    2. Install `libtinfo5` and `libncurses5` via your package manager.
    3. Verify the installation by running the failing TCL script manually.

```bash
# 1. Find the PID of the frozen unwrapped vivado binary and kill it
ps auxf | grep -E "vivado.*xlpartinfo"
# Look for the deepest child process (e.g., .../unwrapped/lnx64.o/vivado)
kill -9 <PID>
# The GUI installer should now unfreeze. You can safely click "Finish".
# If that's not the case, forcefully close the installer (WARNING: This kills all Java apps like Eclipse/IntelliJ)
killall -9 java

# 2. Install the missing legacy libraries
sudo apt update
sudo apt install libtinfo5 libncurses5

# 3. Verify the fix by manually running the background task
# Note: Adjust the /opt/Xilinx/2025.2/ path if your installation directory or tool version differs.
/opt/Xilinx/2025.2/Vivado/bin/vivado -nolog -nojournal -mode batch -source /opt/Xilinx/2025.2/Vivado/scripts/sysgen/tcl/xlpartinfo.tcl -tclargs /opt/Xilinx/2025.2/Vivado/data/parts/installed_devices.txt

# If successful, the command will exit cleanly with:
# INFO: [Common 17-206] Exiting Vivado...
```

## 🔗 References

* [Official documentation page](https://adaptivesupport.amd.com/s/article/76616)
