# 🚀 Vitis Toolchain Setup

**Author:** Julien Posso (ONERA)  
**Date:** 2026-03  
**Tags:** `Vivado` `Vitis` `Installation` `Linux` `License`

Deploying the 95GB+ AMD Unified FPGAs & Adaptive SoCs toolchain in a shared research lab comes with specific challenges: corporate proxies corrupting web installers, legacy dependencies causing silent crashes, and strict `sudo` policies breaking user permissions. This guide provides a proxy-proof, multi-user deployment strategy in `/opt` and clarifies the licensing maze for advanced boards like the VEK280.

## 🔗 Useful Resources
- 📖 [AMD Vitis/Vivado Download Page](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html)
- 📖 [Vitis Software Platform Installation](https://docs.amd.com/r/en-US/ug1400-vitis-embedded/Vitis-Software-Platform-Installation)
- 📖 [Vivado Design Suite User Guide: Release Notes, Installation, and Licensing](https://docs.amd.com/r/en-US/ug973-vivado-release-notes-install-license)
- 🛠️ [Related Field Note](../notes/vitis-installer-freeze.md)

## 🧠 System Overview
Instead of fighting the corporate firewall with the lightweight web installer, use the massive Single-File Download (SFD) archive. To respect system administration best practices, the tools are installed in `/opt` without running the monolithic installer as `root`, preventing permission pollution in user directories. The Vitis/Vivado installer natively decouples the hardware drivers (which strictly require `root`) from the software stack.

---

## 🏁 Step-by-Step Instructions

### 🛠️ Preparation & Downloads

1. Download the Vitis Core Development Kit Full Product **SFD (Single-File Download)** archive (e.g., `FPGAs_AdaptiveSoCs_Unified_*.tar`) from the [AMD portal](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html). Do not use the Web Installer (`.bin`), as intermediate firewalls may corrupt the downloaded chunks.
2. Verify the md5 sum of the downloaded archive against the value on the AMD website.
```bash
md5sum FPGAs_AdaptiveSoCs_Unified_*.tar
```
3. Copy and extract the archive to a local drive with at least 200GB of free space.
```bash
tar -xvf FPGAs_AdaptiveSoCs_Unified_*.tar
```

### 🔌 Environment Setup & Installation
Follow these steps to safely deploy the tools in `/opt` without breaking user permissions or triggering silent crashes.

**1\. Prepare the target directory** Temporarily give your standard user ownership of the target directory to avoid using `sudo` during the main installation.
```bash
sudo mkdir -p /opt/Xilinx
sudo chown -R $USER:$USER /opt/Xilinx
```

**2\. Pre-install dependencies** Navigate to the extracted archive folder. Run the provided script to install necessary OS libraries, which prevents the installer GUI from freezing indefinitely at 99%.
```bash
cd <untarred_location>/FPGAs_AdaptiveSoCs_Unified_202x.x_*/
sudo apt update
sudo ./installLibs.sh
```

**3\. Run the installer** Execute the setup script **without** `sudo`.
```bash
./xsetup
```
After ~10 seconds the internet connection will fail: click *Change Proxy Settings* and set the manual proxy settings of your institution (server and port). When prompted, choose the following and leave the rest as default settings:
- Vitis: it is the full software package that includes both Vitis and Vivado.
- Tools and devices: select Vitis IP Cache for faster on-boarding and your target hardware device (e.g., SoCs/Zynq UltraScale+ MPSoCs).
- Agree to all terms and conditions.
- Installation path: `/opt/Xilinx`.
- Dismiss proposed downloads (e.g., installLibs.sh, or Vitis 2025.2).
- Leave the Vivado License Manager open for now.

**4\. Secure the installation** Once the GUI confirms the installation is complete, lock the directory to prevent accidental modifications by other users.

```bash
sudo chown -R root:root /opt/Xilinx
sudo chmod -R 755 /opt/Xilinx
```

### 💻 Hardware Drivers & Licensing

**1\. Install Cable Drivers (JTAG/UART)** This is the only post-installation step that requires root privileges. It injects the `udev` rules so your OS can detect connected boards like the VEK280.

```bash
cd /opt/Xilinx/202x.x/Vivado/data/xicom/cable_drivers/lin64/install_script/install_drivers/
sudo ./install_drivers
# Optional but recommended: Add yourself to the dialout group for UART serial access
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
```

*(Note: You must log out and log back into your Ubuntu session for the group changes to take effect).*

**2\. Understanding AMD Licenses** AMD's licensing model has evolved, which often causes confusion:

- **Vivado ML Standard (Free):** Does **not** require a `.lic` file anymore. The authorization to synthesize and route for entry/mid-range chips (Zynq-7000, standard MPSoCs) is hardcoded into the binaries.
- **Vivado ML Enterprise (Paid/Floating):** Required for synthesis targeting large architectures like Virtex or Versal AI Edge ([full list in the UG973](https://docs.amd.com/r/en-US/ug973-vivado-release-notes-install-license/Supported-Devices), e.g., the **XCVE2802** of the **VEK280** evaluation board).
- **AIE-ML NPU IP License (Free):** Required `.lic` to use Vitis-AI 5.1 and above that includes pre-compiled AMD Neural Processing Units (NPU) exploiting both Versal AI engines and programmable logic.
- **AI Engine Tools License (Free):** Required `.lic` to target custom implementations on Versal AI engines.

**3.\. Generate your licenses**:
- Go to the [AMD Product Licensing website](https://amd.entitlenow.com/AcrossUser/main.gsp?). 
- Free licenses are not shown in the default list. Click on *Search the Evaluation and No Charge cores* to add them. Select your required certificates, then click *Generate Node-Locked License*. The host ID information of your computer is available in the Vivado License Manager under *View Host Information*. If you closed it after installation, you can reopen it from your applications.
- Download your `.lic` license and import it in the Vivado License Manager under the *Load License* section.

**Remark: In a lab environment, you may also connect to your IT department's FlexLM server using an environment variable.**

**4\. Terminal Integration (The Smart Way)** **⚠️ Warning:** Do *not* directly source the Xilinx `settings64.sh` in your global `~/.bashrc`. While modern versions no longer pollute the `LD_LIBRARY_PATH` globally, the script heavily modifies the `PATH` variable by prepending its own directories. Because Xilinx bundles its own legacy versions of standard tools (like Python, gmake, or tclsh), sourcing it globally will shadow your system's binaries and silently break your Conda environments or ROS workspaces.

Instead, create an **alias** in your `~/.bashrc` to activate the Xilinx environment and point to your licenses only when needed.

Open your `~/.bashrc` and add the following lines at the end:

```
# Xilinx / Vitis Environment Activation Alias
# Use the Vitis script if you installed the full package (it includes Vivado)
alias load_vitis='source /opt/Xilinx/2025.2/Vitis/settings64.sh && export XILINXD_LICENSE_FILE="~/.Xilinx/:2100@license-server.your-lab.fr"'
```

Reload your terminal (`source ~/.bashrc`). Now, whenever you need to compile a hardware design or run FINN/TVM backends, simply type `load_vitis` in your terminal.

??? info "Deep Dive: The XILINXD\_LICENSE\_FILE variable" 
    While launching Vitis from the Ubuntu app menu requires using the graphical Vivado License Manager (VLM) to import `.lic` files, using the `XILINXD_LICENSE_FILE` environment variable is the most robust method for headless/CLI workflows. As shown in the alias above, you can chain multiple license sources by separating them with a colon (`:`). The compiler will check your local `.lic` files first, and if a feature is missing (like the Enterprise synthesis for the VEK280), it will fall back to the lab's floating FlexLM server.

---

## 🚀 Next Steps

- [Setup Your Evaluation Board](vek280-setup.md)
